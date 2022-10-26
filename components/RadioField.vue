<script>
export default {
  emits: ["input"],
  props: {
    label: {
      type: String,
      default: "",
    },
    action: {
      type: null,
    },
    type: {
      type: String,
      default: "radio",
    },
    name: {
      type: String,
    },
    value: {
      type: null,
    },
    options: {
      type: String,
    },
    defaultValue: {},
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
      errorMsg: "",
      valueData: this.defaultValue,
    };
  },
  watch: {
    valueData(value) {
      this.$emit("input", value);
    },
  },
  methods: {
    /*
    checkValue(option) {
      this.valueData = option.id;
      this.$emit("input", option.id);
    },
    */
    validate() {
      if (this.required && !this.valueData) {
        this.error = true;
        this.errorMsg = this.errortext["required"];
        return false;
      } else if (!this.valueData) {
        return true;
      }

      this.error = false;
      return (this.success = true);
    },
  },
  mounted() {
    this.$emit("input", this.valueData);
  },
};
</script>

<template>
  <div class="radio-title unselectable" v-if="label">{{ label }}</div>
  <div
    class="input-wrap"
    :class="{
      error: error,
      success: success,
    }"
  >
    <label
      class="radio-label"
      :class="{ checked: valueData == option.id }"
      v-for="option in JSON.parse(options)"
      :key="option.id"
    >
      <span
        ><input
          :type="type"
          :name="name"
          :value="option.id"
          v-model="valueData"
      /></span>
      {{ option.title }}
    </label>
  </div>

  <div class="error-msg" v-if="error">{{ errorMsg }}</div>
</template>
