<script>
export default {
  emits: ["input"],
  props: {
    label: {
      type: String,
      default: "",
    },
    value: {},
    keyField: {
      type: String,
      default: "name",
    },
    titleField: {
      type: String,
      default: "value",
    },
    options: {
      type: String,
      default: "",
    },
    defaultoption: {
      type: Number,
    },
    required: {
      type: Boolean,
      default: false,
    },
    errortext: {
      type: Object,
      default: function () {
        return {};
      },
    },
  },
  data() {
    return {
      error: false,
      success: false,
      isFocused: false,
      valueData: this.value,
      showOptions: false,
      errorMsg: "",
      title: "",
      optionsList: [],
    };
  },
  watch: {
    value(value) {
      this.valueData = value;
      this.title = this.optionsList[value] ? this.optionsList[value].title : "";
    },
    valueData(newValue) {
      this.$emit("input", newValue);
    },
    options(value) {
      const optionsList = JSON.parse(value);

      if (!optionsList[this.valueData]) {
        const keys = Object.keys(optionsList);
        this.valueData = keys[0];
        this.title = optionsList[keys[0]].title;
      }

      this.optionsList = optionsList;
      this.selectValue(optionsList[this.valueData]);
    },
  },
  methods: {
    validate() {
      if (!this.valueData && this.required) {
        this.error = true;
        this.errorMsg = this.errortext["required"];
        return false;
      } else if (!this.valueData) {
        return true;
      }

      this.error = false;
      return (this.success = true);
    },
    selectValue(option) {
      this.valueData = option.id;
      this.title = option.title;
      this.showOptions = false;
    },
    changeOptions() {
      this.showOptions = !this.showOptions;
    },
  },
  computed: {
    isActive() {
      return this.isFocused || this.valueData;
    },
  },
  mounted() {
    if (this.defaultoption) {
      this.valueData = this.defaultoption;
    }

    if (this.options) {
      this.optionsList = JSON.parse(this.options);
    }

    if (this.valueData && this.optionsList) {
      this.title = this.optionsList[this.valueData].title;
    }
  },
};
</script>

<template>
  <div v-if="showOptions" class="select-overlay" @click="changeOptions()"></div>

  <label
    class="input-wrap"
    :class="{
      error: error,
      success: success,
      active: isActive,
    }"
  >
    <span class="label unselectable" v-if="label">{{ label }}</span>

    <span class="textbox select-wrap" @click="changeOptions()">
      <span class="selectbox-value">
        <span class="value-title">{{ title }}</span>
      </span>

      <svg-icon :icon="showOptions ? 'select_top' : 'select'" />

      <input type="hidden" ref="input" v-model="valueData" />
    </span>

    <span class="error-msg" v-if="error"> {{ errorMsg }} </span>
  </label>

  <div class="selectbox-wrap" v-if="showOptions">
    <div class="scorll-wrap">
      <ul class="selectbox-list">
        <li
          v-for="option in optionsList"
          :key="option.id"
          :class="{ selected: option.id === valueData }"
          @click="selectValue(option)"
        >
          <span class="option">{{ option.title }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>
