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
    maxLength: {
      type: Number,
    },
    type: {
      type: String,
      default: "text",
    },
    initalValue: {
      type: String,
    },
    mask: {
      type: String,
    },
    placeholder: {
      type: String,
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
      showPassword: false,
      errors: "",
      errorMsg: "",
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
        this.$emit("input", newValue);
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

      if (this.type === "email") {
        let valid =
          /^[A-Za-z0-9!#$%&'*+-/=?^_`{|}~]+@[A-Za-z0-9-]+(\.[A-Za-z0-9-]+)+$/.test(
            this.value
          );

        if (!valid) {
          this.error = true;
          this.errorMsg = this.errortext["invalid"];
          return false;
        }
      } else if (this.type === "tel") {
        // return this.value.replace(/[^0-9]/g, "").length === 11;
        let valid = this.value.replace(/[^0-9]/g, "");

        if (!valid) {
          this.error = true;
          this.errorMsg = this.errortext["invalid"];
          return false;
        }
      } else if (this.type === "password" || this.showPassword === true) {
        // return this.value.length > 5;
        let valid = this.value.length > 5;

        if (!valid) {
          this.error = true;
          this.errorMsg = this.errortext["invalid"];
          return false;
        }
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
    <span class="label unselectable" v-if="label">{{ label }}</span>
    <input
      :type="showPassword ? 'text' : type"
      ref="input"
      class="textbox"
      v-model="valueData"
      @focus="focus"
      @blur="blur"
      :placeholder="isActive ? placeholder : ''"
    />

    <div
      class="show-pass"
      v-if="type == 'password'"
      @click="showPassword = !showPassword"
    >
      <svg-icon :icon="showPassword ? 'show_pass' : 'pass'" />
    </div>

    <span class="error-msg" v-if="error"> {{ errorMsg }} </span>
  </label>
</template>
