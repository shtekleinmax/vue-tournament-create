<script>
import axios from "@/init/axios.js";

export default {
  name: "TournamentCreate",
  props: {
    action: {
      type: String,
      default: "",
    },
    errortext: {
      type: Object,
      default: function () {
        return {};
      },
    },
    values: {
      type: Object,
      default: function () {
        return {};
      },
    },
    steptitleone: {
      type: String,
      default: "",
    },
    steptitletwo: {
      type: String,
      default: "",
    },
    types: {
      type: Object,
      default: function () {
        return {};
      },
    },
    options: {
      type: Object,
      default: function () {
        return {};
      },
    },
  },
  data() {
    return {
      loading: false,
      pageTitle: "",
      stepTitle: "",
      defaultFormat: Object.keys(this.options.format)[0],
      format: Object.keys(this.options.format)[0],
      currentStepNumber: 1,
      tournamentType: this.values.type ? this.values.type : 0,
      country_id: this.values.country ? +this.values.country : 1,
      city_id: this.values.city ? +this.values.city : 1,
      countries: JSON.stringify(this.options.countries),
      cities: this.values.country
        ? JSON.stringify(this.options.cities[this.values.country])
        : JSON.stringify(this.options.cities[1]),
      fieldValues: {},
      showBlock: {},
      notValid: {
        drow: false,
        drowText: "",
        dedline: false,
        dedlineText: "",
        range: false,
        rangeText: "",
      },
    };
  },
  watch: {
    stepTitle(title) {
      let pageTitle = document.querySelectorAll(".page-title h1")[0];
      pageTitle.innerHTML = this.pageTitle + " - " + title;
    },
    country_id(country) {
      this.setValue("country_id", country);
      let cities = this.options.cities[country];
      this.cities = JSON.stringify(cities);
      this.city_id = cities[this.city_id]
        ? this.city_id
        : Object.keys(cities)[0];
    },
    city_id(city) {
      this.setValue("city_id", city);
    },
    tournamentType() {
      this.showBlockByType("tournament_form");
      this.showBlockByType("players_numbers");
      this.showBlockByType("pro_am");
    },
  },
  methods: {
    setValue(name, value) {
      this.fieldValues[name] = value;
    },
    getValue(name) {
      return this.fieldValues[name] || null;
    },
    setType(name, value) {
      this.setValue(name, value);
      this.tournamentType = value;
    },
    setFormat(name, value) {
      this.setValue(name, value);
      this.format = value;
    },
    showBlockByType(method) {
      let values = this.options.types[this.tournamentType][method].split(",");

      if (values.length > 1) {
        this.showBlock[method] = true;
        this.setValue(method, 0);
      } else {
        this.showBlock[method] = false;
        this.setValue(method, values[0]);
      }
    },
    setTournamentFrom(name, value) {
      this.showBlock["tournament_cat"] =
        this.options.forms[value].tournaments_categories;
      this.setValue(name, value);
    },
    setCountry(name, value) {
      this.country_id = value;
    },
    uploadClick() {
      this.$refs.file.click();
    },
    uploadFile() {
      this.file = this.$refs.file.files[0];
    },
    goToStepTwo() {
      let result = true;
      this.notValid.drow = false;
      this.notValid.dedline = false;
      this.notValid.range = false;
      this.notValid.drowText = "";
      this.notValid.dedlineText = "";
      this.notValid.rangeText = "";

      if (!this.$refs.title.validate()) result = false;
      if (!this.$refs.description.validate()) result = false;
      if (!this.$refs.country_id.validate()) result = false;
      if (!this.$refs.city_id.validate()) result = false;
      if (!this.$refs.address.validate()) result = false;
      if (!this.$refs.admission_fee.validate()) result = false;

      if (!this.$refs.tournament_range.validate()) {
        result = false;
      } else {
        var rangeTimestamp = this.$refs.tournament_range.rangeTimestamp;
      }

      if (!this.$refs.deadline.validate()) {
        result = false;
      } else {
        var deadlineTimestamp = this.$refs.deadline.dateTimestamp;
      }

      if (!this.$refs.drow.validate()) {
        result = false;
      } else {
        var drowTimestamp = this.$refs.drow.dateTimestamp;
      }

      if (deadlineTimestamp > drowTimestamp) {
        this.notValid.drow = true;
        this.notValid.dedline = true;
        this.notValid.drowText = this.$v("MSG_DROW_BEFORE_DEADLINE");
        this.notValid.dedlineText = this.$v("MSG_DROW_BEFORE_DEADLINE");
        result = false;
      }

      if (deadlineTimestamp > rangeTimestamp.start) {
        this.notValid.range = true;
        this.notValid.dedline = true;
        this.notValid.rangeText = this.$v("MSG_TOURNAMENT_BEFORE_DEADLINE");
        this.notValid.dedlineText = this.$v("MSG_TOURNAMENT_BEFORE_DEADLINE");

        result = false;
      }

      if (drowTimestamp > rangeTimestamp.start + 86399999) {
        this.notValid.range = true;
        this.notValid.drow = true;
        this.notValid.rangeText = this.$v("MSG_TOURNAMENT_BEFORE_DROW");
        this.notValid.drowText = this.$v("MSG_TOURNAMENT_BEFORE_DROW");

        result = false;
      }

      if (result) {
        this.currentStepNumber = 2;
        this.stepTitle = this.steptitletwo;
        this.setValue("currency_id", this.$refs.admission_fee.selected);
      }
    },
    validate() {
      let result = true;

      for (let name in this.$refs) {
        if (name == "document" || name == "rating" || !this.$refs[name]) {
          continue;
        }

        if (!this.$refs[name].validate()) {
          result = false;
        }
      }

      return result;
    },
    submit() {
      if (this.validate()) {
        const sendData = Object.assign({}, window.config, {
          form: this.fieldValues,
        });

        this.loading = true;

        axios.post(this.action, sendData).then((response) => {
          this.loading = false;

          if (response) {
            if (response.result === "success") {
              //   this.clear();
            }

            if (response.msg) {
              this.$root.showModal("modal_message", false, response.msg);
            }

            if (response.error) {
              this.$root.showModal("modal_message", false, response.error);
            }

            if (response.location) {
              window.location = response.location;
            }
          }
        });
      }
    },
  },
  computed: {
    showUpload() {
      return this.fieldValues["players_numbers"] != 17;
    },
  },
  mounted() {
    this.pageTitle = document.querySelectorAll(".page-title h1")[0].innerHTML;
    this.stepTitle = this.steptitleone;

    if (!this.values.country) {
      const location = JSON.parse(localStorage.getItem("location") || "[]");
      this.country_id = location.country_id;
      this.city_id = location.city_id;
    }

    this.setValue("format", this.format);
    this.setValue("country_id", this.country_id);
    this.cities = JSON.stringify(this.options.cities[this.country_id]);

    for (let [name, value] of Object.entries(this.values)) {
      if (name === "tournament_type") {
        this.setType(name, value);
      } else if (name === "format") {
        this.setFormat(name, value);
      } else {
        this.setValue(name, value);
      }
    }
  },
};
</script>

<template>
  <div class="form" v-if="currentStepNumber === 1">
    <transition name="fast-fade">
      <div v-if="loading" class="overlay progress"></div>
    </transition>

    <div class="form-wrap">
      <div class="form-block">
        <div class="form-field">
          <text-field
            :ref="'title'"
            :label="$v('STR_TOURNAMENT_TITLE') + ' *'"
            :type="'text'"
            @input="setValue('title', $event)"
            :value="getValue('title')"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field text-area">
          <text-area
            :ref="'description'"
            :label="$v('STR_TOURNAMENT_DESCRIPTION')"
            @input="setValue('description', $event)"
            :value="getValue('description')"
            :errortext="errortext"
          />
        </div>
      </div>

      <div class="form-block">
        <div class="form-field select">
          <select-field
            :ref="'country_id'"
            :label="$v('STR_COUNTRY') + ' *'"
            @input="setCountry('country_id', $event)"
            :options="countries"
            :value="getValue('country_id')"
            :defaultoption="+country_id"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field select">
          <select-field
            :ref="'city_id'"
            :label="$v('STR_CITY') + ' *'"
            :options="cities"
            @input="setValue('city_id', $event)"
            :value="getValue('city_id')"
            :defaultoption="+city_id"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field">
          <text-field
            :ref="'address'"
            :label="$v('STR_TOURNAMENT_ADDRESS')"
            :type="'text'"
            @input="setValue('address', $event)"
            :value="getValue('address')"
            :errortext="errortext"
          />
        </div>
      </div>

      <div class="form-block">
        <div class="form-field calendar">
          <calendar-field
            :ref="'tournament_range'"
            :label="$v('STR_TOURNAMENT_DATE_RANGE') + ' *'"
            :type="'calendar'"
            :mode="'date'"
            :range="true"
            @input="setValue('tournament_range', $event)"
            :value="getValue('tournament_range')"
            :initStart="values.tournament_start ? values.tournament_start : ''"
            :initEnd="values.tournament_end ? values.tournament_end : ''"
            :notValid="notValid.range"
            :notValidText="notValid.rangeText"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field calendar">
          <calendar-field
            :ref="'deadline'"
            :label="$v('STR_APPLICATION_DEADLINE') + ' *'"
            :type="'calendar'"
            :mode="'dateTime'"
            :range="false"
            @input="setValue('deadline', $event)"
            :value="getValue('deadline')"
            :initalValue="values.deadline ? values.deadline : ''"
            :notValid="notValid.dedline"
            :notValidText="notValid.dedlineText"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field calendar">
          <calendar-field
            :ref="'drow'"
            :label="$v('STR_DROW_TIME') + ' *'"
            :type="'calendar'"
            :mode="'dateTime'"
            :range="false"
            @input="setValue('drow', $event)"
            :value="getValue('drow')"
            :initalValue="values.drow ? values.drow : ''"
            :notValid="notValid.drow"
            :notValidText="notValid.drowText"
            :errortext="errortext"
            :required="true"
          />
        </div>
      </div>

      <div class="form-block">
        <div class="form-field select">
          <money-field
            :ref="'admission_fee'"
            :label="$v('STR_ADMISSION_FEE') + ' *'"
            :type="'text'"
            @input="setValue('admission_fee', $event)"
            :value="getValue('admission_fee')"
            :options="options.currency"
            :errortext="errortext"
            :required="true"
          />
        </div>
      </div>
    </div>

    <div class="form-footer" v-if="currentStepNumber == 1">
      <button type="button" class="button primary" @click="goToStepTwo()">
        {{ $v("STR_NEXT") }}
      </button>

      <div class="step-note">{{ $v("STR_STEP_ONE_NOTE") }}</div>
    </div>
  </div>

  <div class="form" v-if="currentStepNumber === 2">
    <transition name="fast-fade">
      <div v-if="loading" class="overlay progress"></div>
    </transition>

    <div class="form-wrap">
      <div class="form-block">
        <div class="form-field select">
          <select-field
            :ref="'coverage_type'"
            :label="$v('STR_COVERAGE_TYPE')"
            @input="setValue('coverage_type', $event)"
            :value="getValue('coverage_type')"
            :defaultoption="values.coverage ? +values.coverage : 0"
            :options="JSON.stringify(options.court)"
            :errortext="errortext"
            :required="false"
          />
        </div>

        <div class="form-field select">
          <select-field
            :ref="'tournament_type'"
            :label="$v('STR_TOURNAMENT_TYPE') + ' *'"
            @input="setType('tournament_type', $event)"
            :options="JSON.stringify(options.types)"
            :value="getValue('tournament_type')"
            :defaultoption="+tournamentType"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field radio subtitle" v-if="showBlock.pro_am">
          <radio-field
            :ref="'pro_am'"
            :label="$v('STR_PROFESSIONAL') + ' *'"
            :type="'radio'"
            :name="'pro_am'"
            @input="setValue('pro_am', $event)"
            :value="getValue('pro_am')"
            :options="JSON.stringify(options.proAm)"
            :defaultValue="
              values.proAm ? +values.proAm : Object.keys(options.proAm)[0]
            "
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field radio" v-if="showBlock.players_numbers">
          <radio-field
            :ref="'players_numbers'"
            :label="$v('STR_DOUBLE_SINGLE_TOURNAMENT') + ' *'"
            :type="'radio'"
            :name="'players_numbers'"
            @input="setValue('players_numbers', $event)"
            :value="getValue('players_numbers')"
            :options="JSON.stringify(options.numbers)"
            :defaultValue="
              values.numbers ? +values.numbers : Object.keys(options.numbers)[0]
            "
            :errortext="errortext"
            :required="true"
          />
        </div>
      </div>

      <div class="form-block" v-if="showBlock.tournament_form">
        <div class="form-field select">
          <select-field
            :ref="'tournament_form'"
            :label="$v('STR_TOURNAMENT_FORM') + ' *'"
            @input="setTournamentFrom('tournament_form', $event)"
            :options="JSON.stringify(options.forms)"
            :value="getValue('tournament_form')"
            :defaultoption="values.form ? +values.form : 0"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field select" v-if="showBlock.tournament_cat > 0">
          <select-field
            :ref="'categories'"
            :label="$v('STR_TOURNAMENT_CATEGORY') + ' *'"
            @input="setValue('tournament_cat', $event)"
            :options="JSON.stringify(options.categories)"
            :value="getValue('tournament_cat')"
            :defaultoption="values.categories ? +values.categories : 0"
            :errortext="errortext"
            :required="true"
          />
        </div>
      </div>

      <div class="form-block">
        <div class="form-field radio">
          <radio-field
            :ref="'format'"
            :label="$v('STR_TOURNAMENT_FORMAT') + ' *'"
            :type="'radio'"
            :name="'format'"
            @input="setFormat('format', $event)"
            :value="getValue('format')"
            :options="JSON.stringify(options.format)"
            :defaultValue="values.format ? +values.format : format"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field numbers subtitle" v-if="format == defaultFormat">
          <numbers-field
            :ref="'groups'"
            :label="$v('STR_TOURNAMENT_GROUPS') + ' *'"
            :type="'radio'"
            :name="'groups'"
            @input="setValue('groups', $event)"
            :value="getValue('groups')"
            :options="JSON.stringify(options.groups)"
            :defaultValue="values.group ? +values.group : 0"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field numbers subtitle" v-if="format == defaultFormat">
          <numbers-field
            :ref="'players'"
            :label="$v('STR_TOURNAMENT_PLAYERS') + ' *'"
            :type="'radio'"
            :name="'players'"
            @input="setValue('players', $event)"
            :value="getValue('players')"
            :options="JSON.stringify(options.players)"
            :defaultValue="values.player ? +values.player : 0"
            :errortext="errortext"
            :required="true"
          />
        </div>

        <div class="form-field numbers subtitle" v-if="format != defaultFormat">
          <numbers-field
            :ref="'grid'"
            :label="$v('STR_PLAYERS') + ' *'"
            :type="'radio'"
            :name="'grid'"
            @input="setValue('grid', $event)"
            :value="getValue('grid')"
            :options="JSON.stringify(options.grid)"
            :defaultValue="values.grid ? +values.grid : 0"
            :errortext="errortext"
            :required="true"
          />
        </div>
      </div>

      <div class="form-block">
        <div class="form-field numbers">
          <numbers-field
            :ref="'wild'"
            :label="$v('STR_WILD_CARD')"
            :type="'radio'"
            :name="'wild'"
            @input="setValue('wild', $event)"
            :value="getValue('wild')"
            :options="JSON.stringify(options.wildCard)"
            :defaultValue="values.wild ? +values.wild : 0"
            :errortext="errortext"
            :required="false"
          />
        </div>

        <div class="form-field numbers">
          <numbers-field
            :ref="'qualifier'"
            :label="$v('STR_QUALIFICATION')"
            :type="'radio'"
            :name="'qualifier'"
            @input="setValue('qualifier', $event)"
            :value="getValue('qualifier')"
            :options="JSON.stringify(options.qualifier)"
            :defaultValue="values.qualifier ? +values.qualifier : 0"
            :errortext="errortext"
            :required="false"
          />
        </div>
      </div>

      <div class="form-block files-block">
        <upload-file
          :ref="'rating'"
          :label="$v('STR_UPLOAD_RATING')"
          :name="'rating'"
          :format="['xlsx']"
          @input="setValue('rating', $event)"
          v-if="showUpload"
        />

        <upload-file
          :ref="'document'"
          :label="$v('STR_UPLOAD_DOCUMENT')"
          :name="'documents'"
          :format="['doc', 'docx', 'pdf', 'odt', 'jpg', 'jpeg', 'png']"
          @input="setValue('documents', $event)"
        />
      </div>
    </div>

    <div class="form-footer">
      <button type="button" class="button primary" @click="submit">
        {{ $v("STR_CREATE_TOURNAMENT") }}
      </button>
    </div>
  </div>

  <div class="form-description">
    <div class="form-title">{{ $v("STR_CREATE_FORM_TITLE") }}</div>
    <div class="steps">
      <div class="step" :class="{ active: currentStepNumber == 1 }">
        <div class="number">{{ $v("STR_STEP_ONE_NUMBER") }}</div>
        <div class="text">{{ $v("STR_STEP_ONE_TEXT") }}</div>
      </div>
      <div class="step" :class="{ active: currentStepNumber == 2 }">
        <div class="number">{{ $v("STR_STEP_TWO_NUMBER") }}</div>
        <div class="text">{{ $v("STR_STEP_TWO_TEXT") }}</div>
      </div>
    </div>
  </div>
</template>
