<template>
  <v-container>
    <v-row class="mt-4">
      <v-col cols="3" class="px-5">
        <div class="subheading">Time</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Time'"
            :name="index"
            :value="getValue(fieldData[index], true)"
            :v-model="getValue(constants[index])"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :class="{ noline: item.readOnly }"
            :append-outer-icon="fieldData[index].append"
            @keyup="updateValues($event)"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Heat/Cooling</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Heat/Cooling'"
            :name="index"
            :value="getValue(fieldData[index], true)"
            :v-model="getValue(constants[index])"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :class="{ noline: item.readOnly }"
            :append-outer-icon="fieldData[index].append"
            @keyup="updateValues($event)"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Weights and Measures</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Weights and Measures'"
            :name="index"
            :value="getValue(fieldData[index], true)"
            :v-model="getValue(constants[index])"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :class="{ noline: item.readOnly }"
            :append-outer-icon="fieldData[index].append"
            @keyup="updateValues($event)"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Staff Expense</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Staff Expense'"
            :name="index"
            :value="getValue(fieldData[index], true)"
            :v-model="getValue(constants[index])"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :prefix="setPrefix(item.type)"
            :class="{ noline: item.readOnly }"
            :append-outer-icon="fieldData[index].append"
            @keyup="updateValues($event)"
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
  data() {
    return {
      constants: { key: null },
      fieldData: {
        index: {
          label: '',
          type: 'number',
          defaultValue: null,
          readOnly: true,
          category: '',
          calculate: '',
          append: '',
          appendColor: ''
        }
      }
    }
  },
  beforeMount() {
    // mount json to a data property
    this.fieldData = json.fields;
    // create another data set we can use for calculations
    let tmpConstants = {};
    Object.entries(this.fieldData).forEach(([key, val]) => {
      tmpConstants[key] = val.defaultValue;
    });
    this.constants = tmpConstants;
    this.calculateValues();
  },
  methods: {
    calculateValues() {
      // need to process the data again to update the calculations that weren't available in the beforeMount() hook
      Object.entries(this.constants).forEach(([key, val]) => {
        if (val === undefined) {
          this.constants[key] = this.getValue(this.fieldData[key]);
          this.fieldData[key].value = this.getValue(this.constants[key], true);
        }
      });
    },
    updateValues(event) {
      // update objects when input is changed
      const fieldname = event.target.name;
      const fieldval = event.target.value;
      this.constants[fieldname] = fieldval;
      Object.entries(this.constants).forEach(([key, val]) => {
        if (this.hasCalculation(this.fieldData[key])) {
          this.constants[key] = this.getValue(this.fieldData[key]);
          this.fieldData[key].value = this.getValue(this.constants[key], true);
          let icon = 'arrow_upward';
          let iconColor = 'red';
          if (val < this.constants[key]) {
            icon = 'arrow_downward';
            iconColor = 'green';
          }
          this.fieldData[key].append = icon;
          this.fieldData[key].appendColor = iconColor;
        }
      });
    },
    getValue(item, format = false) {
      // get values from updated object(s) to assign to form fields
      if (this.hasCalculation(item)) {
        // generate calculations to evaluate
        const expr = item.calculate
          .toString()
          .replace(/this.constant_/g, 'this.constants.constant_'); // fix bad json data
        /**
         *  eval() is evil. 
         *  But in the interest of time, rather than work out how to pass the Vue instance ('this')
         *  to a more secure and performant sandbox function construct, we'll assume this data is 
         *  trusted and small.
         */
        let ev = eval(expr);
        ev = format ? this.format(ev, item.type) : ev;
        return ev;
      }
      return format ? this.format(item.defaultValue, item.type) : item.defaultValue;
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
    }
  },
};
</script>

<style lang="scss">
.subheading {
  font-weight: bold;
}
// .v-input--is-readonly {
//   .v-input__control {
//     .v-input__slot {
//       &::after {
//         width: 0;
//         content: unset;
//       }
//     }
//   }
// }
.arrow-up {
  color: red;
}
.arrow-down {
  color: green;
}
</style>
