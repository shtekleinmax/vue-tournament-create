<script>
export default {
  emits: ["input"],
  props: {
    label: {
      type: String,
      default: "",
    },
    value: {
      type: String,
      default: "",
    },
    initalValue: {
      type: String,
      default: "",
    },
    maxLength: {
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
    };
  },
  watch: {
    value(value) {
      this.valueData = value;
    },
    valueData(newValue) {
      let value = newValue;

      if (this.maxLength) {
        value = value.substr(0, this.maxLength);
      }

      if (value !== newValue) {
        this.valueData = value;
      } else {
        this.$emit("input", value);
      }
    },
  },
  computed: {
    isActive() {
      return this.isFocused || this.value;
    },
  },
  methods: {
    focus() {
      this.isFocused = true;
    },
    blur() {
      this.isFocused = false;
    },
    validate() {
      if (!this.value && this.required) {
        this.error = true;
        this.errorMsg = this.errortext["required"];
        return false;
      } else if (!this.value) {
        return true;
      }

      this.error = false;
      return (this.success = true);
    },
  },
  mounted() {
    if (this.initalValue) {
      this.$emit("input", this.initalValue);
    }
  },
};
</script>

<template>
  <label
    class="input-wrap"
    :class="{
      active: isActive,
      error: error,
      success: success,
    }"
  >
    <span class="label" v-if="label">{{ label }}</span>
    <textarea
      class="textarea"
      v-model="valueData"
      @focus="focus"
      @blur="blur"
    ></textarea>
    <span class="error-msg" v-if="error">{{ errorMsg }}</span>
  </label>
</template>
