<script>
export default {
  emits: ["input"],
  props: {
    label: {
      type: String,
      default: "",
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
  methods: {
    checkValue(option) {
      this.valueData = this.valueData !== option ? option : 0;
      this.$emit("input", this.valueData);
    },
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
  },
};
</script>

<template>
  <div class="numbers-title unselectable" v-if="label">{{ label }}</div>
  <div
    class="input-wrap"
    :class="{
      error: error,
      success: success,
    }"
  >
    <label
      class="numbers-label"
      :class="{ checked: valueData == option }"
      v-for="option in JSON.parse(options)"
      :key="option"
      @click="checkValue(option)"
    >
      {{ option }}
    </label>
  </div>

  <div class="error-msg" v-if="error">{{ errorMsg }}</div>
</template>
