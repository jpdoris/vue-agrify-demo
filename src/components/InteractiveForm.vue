<template>
  <v-container fluid>
    <v-row class="mt-4">
      <v-col cols="3" class="px-5">
        <div class="subheading">Time</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Time'"
            :name="index"
            :value="getValue(fieldData[index])"
            :v-model="getValue(fieldData[index])"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :class="{ noline: item.readOnly }"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Heat/Cooling</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Heat/Cooling'"
            :name="index"
            :value="getValue(fieldData[index])"
            :v-model="getValue(fieldData[index])"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :class="{ noline: item.readOnly }"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Weights and Measures</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Weights and Measures'"
            :name="index"
            :value="getValue(fieldData[index])"
            :v-model="getValue(fieldData[index])"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :class="{ noline: item.readOnly }"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Staff Expense</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Staff Expense'"
            :name="index"
            :value="getValue(fieldData[index])"
            :v-model="getValue(fieldData[index])"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :prefix="setPrefix(item.type)"
            :class="{ noline: item.readOnly }"
            @keyup="updateValues"
          />
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import json from '@/data/constants.json';

export default {
  name: 'InteractiveForm',
  data: () => ({
    return: {
      constants: { key: null },
      fieldData: {
        index: {
          label: '',
          type: 'number',
          defaultValue: null,
          readOnly: true,
          category: '',
          calculate: '',
        },
      },
    },
  }),
  beforeMount() {
    this.fieldData = json.fields;
    let tmpConstants = {};
    Object.entries(this.fieldData).forEach(([key, val]) => {
      tmpConstants[key] = val.defaultValue;
    });
    this.constants = tmpConstants;
    this.calculateValues();
  },
  methods: {
    calculateValues() {
      Object.entries(this.constants).forEach(([key, val]) => {
        if (val === undefined) {
          this.constants[key] = this.getValue(this.fieldData[key]);
          this.fieldData[key].value = Number(this.constants[key]);
        }
      });
    },
    updateValues(field) {
      console.log('calculateValues() called on ', field); // TODO
      Object.entries(this.constants).forEach(([key, val]) => {
        if (val === undefined) {
          this.constants[key] = this.getValue(this.fieldData[key]);
          this.fieldData[key].value = Number(this.constants[key]);
        }
      });
    },
    getValue(item) {
      if (this.hasCalculation(item)) {
        const expr = item.calculate
          .toString()
          .replace(/this.constant_/g, 'this.constants.constant_');
        let ev = eval(expr);
        ev = isNaN(ev) ? null : this.format(ev, item.type);
        return ev;
      }
      return this.format(item.defaultValue, item.type);
    },
    hasCalculation(item) {
      if (Object.prototype.hasOwnProperty.call(item, 'calculate')) {
        return true;
      }
      return false;
    },
    setPrefix(type) {
      return type === 'percent' ? '%' : '';
    },
    format(val, type) {
      let digits = 0;
      if (type === 'float') {
        digits = 2;
      }
      return new Intl.NumberFormat('en-US', {
        style: 'decimal',
        maximumFractionDigits: digits,
        minimumFractionDigits: digits,
      }).format(val);
    },
  },
};
</script>

<style lang="scss">
.subheading {
  font-weight: bold;
}
.v-input--is-readonly {
  .v-input__control {
    .v-input__slot {
      &::after {
        width: 0;
        content: none;
      }
    }
  }
}
</style>
