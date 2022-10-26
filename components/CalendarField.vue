<script>
import { DatePicker } from "v-calendar";

export default {
  emits: ["input"],
  components: {
    DatePicker,
  },
  props: {
    label: {
      type: String,
      default: "",
    },
    value: {
      type: String,
      default: "",
    },
    type: {
      type: String,
      default: "calendar",
    },
    mode: {
      type: String,
      default: "date",
    },
    range: {
      type: Boolean,
      default: false,
    },
    initalValue: {
      type: String,
    },
    initStart: {
      type: String,
      default: "",
    },
    initEnd: {
      type: String,
      default: "",
    },
    mask: {
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
    notValid: {
      type: Boolean,
      default: false,
    },
    notValidText: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      error: false,
      success: false,
      isFocused: false,
      dateValue: "",
      errorMsg: "",
      dateTimestamp: "",
      rangeTimestamp: {},
      date: this.initalValue ? this.initalValue : "",
      dateRange: {
        start: this.initStart ? this.initStart : "",
        end: this.initEnd ? this.initEnd : "",
      },
      modelConfig: {
        type: "string",
        mask: "DD.MM.YYYY HH:mm",
      },
    };
  },
  watch: {
    date(date) {
      this.dateValue = date;
    },
    dateRange(dateRange) {
      this.dateValue = dateRange.start + " - " + dateRange.end;
    },
    dateValue(value) {
      this.$emit("input", value);
    },
  },
  mounted() {
    if (this.range) {
      this.dateValue = this.dateRange.start + " - " + this.dateRange.end;
    } else {
      this.dateValue = this.date;
    }
  },
  methods: {
    isDate(string) {
      let dateTime = string.split(" ");
      let date = dateTime[0].split(".");
      let dateFormat;

      if (dateTime[1] && dateTime[1].length < 8) {
        dateTime[1] += ":00";
      }

      if (date.length == 3) {
        dateFormat = date[2] + "-" + date[1] + "-" + date[0];
      }

      return Date.parse(dateFormat + "T" + dateTime[1]);
    },
    validate() {
      if (!this.dateValue && this.required) {
        this.error = true;
        this.errorMsg = this.errortext["required"];
        return false;
      } else if (!this.dateValue) {
        return true;
      }

      if (this.range) {
        this.rangeTimestamp.start = this.isDate(this.dateRange.start);
        this.rangeTimestamp.end = this.isDate(this.dateRange.end);

        if (!this.rangeTimestamp) {
          this.error = true;
          this.errorMsg = "";
          return false;
        }
      } else {
        this.dateTimestamp = this.isDate(this.dateValue);

        if (!this.dateTimestamp) {
          this.error = true;
          this.errorMsg = "";
          return false;
        }
      }

      this.error = false;
      return (this.success = true);
    },
  },
};
</script>

<template>
  <div
    class="input-wrap"
    :class="{
      error: error || notValid,
      success: success && !notValid && !error,
      active: date || dateRange.start,
    }"
  >
    <span class="label unselectable" v-if="label">{{ label }}</span>

    <DatePicker
      v-model="dateRange"
      :model-config="modelConfig"
      :mode="mode"
      is-range
      v-if="range"
    >
      <template v-slot="{ inputValue, inputEvents }">
        <input
          ref="input"
          class="bg-white px-2 py-1"
          :value="
            inputValue.start ? inputValue.start + ' - ' + inputValue.end : ''
          "
          v-on="inputEvents.start"
        />
      </template>
    </DatePicker>

    <DatePicker
      v-model="date"
      :model-config="modelConfig"
      :mode="mode"
      is24hr
      v-else
    >
      <template v-slot="{ inputValue, inputEvents }">
        <input
          ref="input"
          class="bg-white px-2 py-1"
          :value="inputValue"
          v-on="inputEvents"
        />
      </template>
    </DatePicker>

    <svg-icon :icon="'calendar'" />
    <span class="error-msg" v-if="error || notValid">
      {{ errorMsg }} {{ notValidText }}
    </span>
  </div>
</template>
