<template lang="pug">
	select.form-control(v-model="value", @change="selectChanged" :disabled="disabled", :name="schema.inputName", :id="getFieldID(schema)", :class="schema.fieldClasses", v-attributes="'input'")
		option(v-if="!selectOptions.hideNoneSelectedText", :disabled="schema.required", :value="null") {{ selectOptions.noneSelectedText || "&lt;Nothing selected&gt;" }}

		template(v-for="item in items")
			optgroup(v-if="item.group", :label="getGroupName(item)")
				option(v-if="item.ops", v-for="i in item.ops", :value="getItemValue(i)") {{ getItemName(i) }}

			option(v-if="!item.group", :value="getItemValue(item)") {{ getItemName(item) }}
</template>

<script>
import { isObject, isNil, find } from "lodash";
import abstractField from "../abstractField";

export default {
	mixins: [abstractField],
	data () {
		return {
			getValues: null
		};
	},
	computed: {
		selectOptions() {
			return this.schema.selectOptions || {};
		},
		items() {
			let values = this.schema.values;
			const getterValues = this.getterValues();
			if (getterValues) {
				return this.groupValues(getterValues);
			} else if (typeof values == "function") {
				return this.groupValues(values.apply(this, [this.model, this.schema]));
			} else return this.groupValues(values);
		}
	},
	mounted () {
		this.$nextTick(() => {
			this.selectChanged();
		});
	},
	methods: {
		getterValues () {
			if (!this.getValues || !this.getValues.length) {
				const storeGetter = this.schema.storeGetter;
				const modelGetter = this.schema.modelGetter;
				if (storeGetter) {
					const vals = typeof this.$store.getters[storeGetter] === "function" ?
						this.$store.getters[storeGetter](this.model, this.schema) :
						this.$store.getters[storeGetter];
					const getValues = this.groupValues(vals);
					this.getValues = JSON.parse(JSON.stringify(getValues));
				} else if (modelGetter) {
					const getValues = this.groupValues(this.model[this.schema.modelGetter]);
					this.getValues = JSON.parse(JSON.stringify(getValues));
				}
			}
			return this.getValues;
		},
		selectChanged () {
			const getterValues = this.getterValues();
			const method = this.schema.selectMethod;
			if (getterValues && method && this.model[method]) {
				const obj = getterValues.find(item=>
					+this.getItemValue(item) === +this.value);
				this.model[method](obj);
			}
		},

		formatValueToField(value) {
			if (isNil(value)) {
				return null;
			}
			return value;
		},

		groupValues(values) {
			let array = [];
			let arrayElement = {};

			values.forEach(item => {
				arrayElement = null;

				if (item.group && isObject(item)) {
					// There is in a group.

					// Find element with this group.
					arrayElement = find(array, i => i.group === item.group);

					if (arrayElement) {
						// There is such a group.

						arrayElement.ops.push({
							id: item.id,
							name: item.name
						});
					} else {
						// There is not such a group.

						// Initialising.
						arrayElement = {
							group: "",
							ops: []
						};

						// Set group.
						arrayElement.group = item.group;

						// Set Group element.
						arrayElement.ops.push({
							id: item.id,
							name: item.name
						});

						// Add array.
						array.push(arrayElement);
					}
				} else {
					// There is not in a group.
					array.push(item);
				}
			});

			// With Groups.
			return array;
		},

		getGroupName(item) {
			if (item && item.group) {
				return item.group;
			}

			throw "Group name is missing! https://icebob.gitbooks.io/vueformgenerator/content/fields/select.html#select-field-with-object-items";
		},

		getItemValue(item) {
			if (isObject(item)) {
				if (typeof this.schema["selectOptions"] !== "undefined" && typeof this.schema["selectOptions"]["value"] !== "undefined") {
					return item[this.schema.selectOptions.value];
				} else {
					// Use 'id' instead of 'value' cause of backward compatibility
					if (typeof item["id"] !== "undefined") {
						return item.id;
					} else {
						throw "`id` is not defined. If you want to use another key name, add a `value` property under `selectOptions` in the schema. https://icebob.gitbooks.io/vueformgenerator/content/fields/select.html#select-field-with-object-items";
					}
				}
			} else {
				return item;
			}
		},

		getItemName(item) {
			if (isObject(item)) {
				if (typeof this.schema["selectOptions"] !== "undefined" && typeof this.schema["selectOptions"]["name"] !== "undefined") {
					return item[this.schema.selectOptions.name];
				} else {
					if (typeof item["name"] !== "undefined") {
						return item.name;
					} else {
						throw "`name` is not defined. If you want to use another key name, add a `name` property under `selectOptions` in the schema. https://icebob.gitbooks.io/vueformgenerator/content/fields/select.html#select-field-with-object-items";
					}
				}
			} else {
				return item;
			}
		}
	}
};
</script>


<style lang="sass">
</style>
