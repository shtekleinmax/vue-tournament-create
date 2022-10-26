<script>
import Inputmask from "inputmask";

export default {
  emits: ["input", "code"],
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
      default: "tel",
    },
    mask: {
      type: String,
    },
    placeholder: {
      type: String,
    },
    countries: {
      type: String,
      default: "",
    },
    defaultCountry: {
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
      showPassword: false,
      errors: "",
      showOptions: false,
      selected: false,
      countriesList: [],
      //    country: 0,
      phone: "",
      code: "",
      flag: "",
    };
  },
  watch: {
    value(value) {
      this.valueData = value;
    },
    valueData(newValue) {
      this.$emit("input", newValue);
    },
    code(code) {
      this.$emit("code", code);
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
    setCode(country) {
      this.code = country.code;
      this.flag = country.flag;
      this.selected = country.id;
      this.showOptions = false;
    },
    getCode() {
      return this.code;
    },
    validate() {
      if (!this.value && this.required) {
        this.error = true;
        this.errorMsg = this.errortext["required"];
        return false;
      } else if (!this.value) {
        return true;
      }

      let valid = this.value.replace(/[^0-9]/g, "").length > 8;

      if (!valid) {
        this.error = true;
        this.errorMsg = this.errortext["invalid"];
        return false;
      }

      this.error = false;
      return (this.success = true);
    },
  },
  mounted() {
    if (this.defaultCountry) {
      this.selected = this.defaultCountry;
    } else {
      if (localStorage.location) {
        const location = JSON.parse(localStorage.getItem("location") || "[]");
        this.selected = location.country_id;
      } else {
        this.selected = 1;
      }
    }

    if (this.countries) {
      const countriesList = JSON.parse(this.countries);

      this.countriesList = countriesList;
      this.code = countriesList[this.selected]["code"];
      this.flag = countriesList[this.selected]["flag"];
    }

    if (this.mask) {
      Inputmask({
        mask: this.mask,
        showMaskOnHover: false,
        showMaskOnFocus: true,
      }).mask(this.$refs.input);
    }
  },
};
</script>

<template>
  <div
    v-if="showOptions"
    class="select-overlay"
    @click="showOptions = !showOptions"
  ></div>

  <label
    class="input-wrap"
    :class="{
      active: isActive,
      error: error,
      success: success,
    }"
  >
    <span class="label unselectable" v-if="label">{{ label }}</span>

    <span class="textbox select-number">
      <span class="selectbox-code" @click="showOptions = !showOptions">
        <span class="flag"
          ><img v-if="flag" :src="'/download/files/' + flag"
        /></span>
        <svg-icon :icon="'arrow'" />
        <span class="code">{{ code }}</span>
      </span>

      <input :type="type" ref="input" v-model="valueData" @focus="focus" />
    </span>

    <span class="error-msg" v-if="error">{{ errorMsg }}</span>
  </label>

  <div class="selectbox-wrap" v-if="showOptions">
    <div class="scorll-wrap">
      <ul class="selectbox-list">
        <li
          v-for="country in countriesList"
          :key="country.id"
          :class="{ selected: country.id === selected }"
          @click="setCode(country)"
        >
          <span class="flag">
            <img v-if="country.flag" :src="'/download/files/' + country.flag" />
          </span>
          <span class="code">{{ country.code }}</span>
          <span class="country">{{ country.title }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>
