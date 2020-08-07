<template>
  <v-container>
    <v-row class="mt-4">
      <v-col cols="3" class="px-5">
        <div class="subheading">Time</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Time'"
            :name="index"
            :value="constants[index] | format(item.type)"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :class="{ noline: item.readOnly }"
            :append-outer-icon="item.append"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Heat/Cooling</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Heat/Cooling'"
            :name="index"
            :value="constants[index] | format(item.type)"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :class="{ noline: item.readOnly }"
            :append-outer-icon="item.append"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Weights and Measures</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Weights and Measures'"
            :name="index"
            :value="constants[index] | format(item.type)"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :class="{ noline: item.readOnly }"
            :append-outer-icon="item.append"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Staff Expense</div>
        <div v-for="(item, index) in fieldData" :key="index">
          <v-text-field
            v-if="item.category === 'Staff Expense'"
            :name="index"
            :value="constants[index] | format(item.type)"
            :v-model="getValue(index, item.defaultValue)"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :prefix="setPrefix(item.type)"
            :class="{ noline: item.readOnly }"
            :append-outer-icon="item.append"
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
      constants: {},
      fieldData: {
        index: {
          label: '',
          type: 'number',
          defaultValue: null,
          readOnly: true,
          category: '',
          calculate: '',
          append: '',
          appendColor: '',
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
      if (Object.prototype.hasOwnProperty.call(val, 'defaultValue')) {
        tmpConstants[key] = val.defaultValue;
      } else {
      //   tmpConstants[key] = this.parseExpression(val.calculate, tmpConstants);
        tmpConstants[key] = null;
      }
    });
    this.constants = tmpConstants;
    this.calculateValues();
  },
  mounted() {
    // console.log('constants mounted', this.constants); // TODO
    // console.log('fieldData', this.fieldData); // TODO
    // this.test();
  },
  filters: {
    format(item, type) {
      // console.log('format filter arg', item); // TODO
      let digits = 0;
      if (type === 'float') {
        digits = 2;
      }
      const formatted = new Intl.NumberFormat('en-US', {
        style: 'decimal',
        maximumFractionDigits: digits,
        minimumFractionDigits: digits,
      }).format(item);
      // console.log('format filter result', formatted); // TODO
      return formatted;
    }
  },
  methods: {
    test() {
      // const str = "(this.constants.constant_days_in_year / this.constants.constant_days_in_week) * this.constants.constant_fte_weekly_hours";
      const str = "this.constants.constant_fte_weekly_hours * (this.constants.constant_days_in_year / this.constants.constant_days_in_week)";
      const leftParens = str.match(/^\((.*)\)/);
      const rightParens = str.match(/\((.*)\)$/);
      console.log('str', str); // TODO
      console.log('leftParens', leftParens); // TODO
      console.log('rightParens', rightParens); // TODO
    },
    fetchData(item) {
      console.log('fetchData'); // TODO
      this.constants = new Function(this.fieldData[item].calculate);
    },
    calculateValues() {
      // need to process the data again to update the calculations that weren't available in the beforeMount() hook
      Object.entries(this.constants).forEach(([key, val]) => {
        if (val === null) {
          this.$data.constants[key] = this.getValue(this.fieldData[key]);
          this.$data.fieldData[key].value = this.getValue(this.constants[key]);
        }
      });
    },
    evalExpression(calculation, data) {
      console.log('evalExpression:', calculation); // TODO
      const leftParens = calculation.match(/^\((.*)\)/);
      const rightParens = calculation.match(/\((.*)\)$/);
      const isCalculated = calculation.match(/[0-9]/);

      if (isNaN(calculation)) {
        let expr = calculation;
        let parsed = [];
        let operands = {};
        let operator;
        let operation;

        if (leftParens !== null) {
          console.log('leftParens', leftParens); // TODO
          const operLeft = this.evalExpression(leftParens[1], data);
          const operRight = calculation.replace(leftParens[1], '').replace("()", "");
          console.log('right operand match', calculation.replace(leftParens[0], '').replace("()", "")); // TODO
          console.log('operLeft', operLeft); // TODO
          console.log('operRight', operRight); // TODO
          operands.left = this.parseExpression(operLeft + operRight);
          operation = {
            '+': (a, b) => a + data[b],
            '-': (a, b) => a - data[b],
            '*': (a, b) => a * data[b],
            '/': (a, b) => a / data[b],
          };
        } else if (rightParens !== null) {
          console.log('right inner calc', rightParens[1]); // TODO
          console.log('left operand match', calculation.replace(rightParens[0], '').replace("()", "")); // TODO
          const operLeft = calculation.replace(rightParens[1], '').replace("()", "");
          const operRight = this.evalExpression(rightParens[1], data);
          console.log('operLeft', operLeft); // TODO
          console.log('operRight', operRight); // TODO
          operands.right = this.evalExpression(operLeft + operRight);
          operation = {
            '+': (a, b) => data[a] + b,
            '-': (a, b) => data[a] - b,
            '*': (a, b) => data[a] * b,
            '/': (a, b) => data[a] / b,
          };
        } else if (isCalculated !== null) {
          console.log('numeric operand', isCalculated); // TODO
          const operLeft = calculation;
          const operRight = this.evalExpression(rightParens[1], data);
          console.log('operLeft', operLeft); // TODO
          console.log('operRight', operRight); // TODO
          operands.right = this.evalExpression(operLeft + operRight);
          operation = {
            '+': (a, b) => data[a] + b,
            '-': (a, b) => data[a] - b,
            '*': (a, b) => data[a] * b,
            '/': (a, b) => data[a] / b,
          };
        } else {
          parsed = this.parseExpression(expr);
          operands = { first: parsed[0], second: parsed[2] };
          operator = parsed[1];
          operation = {
            '+': (a, b) => data[a] + data[b],
            '-': (a, b) => data[a] - data[b],
            '*': (a, b) => data[a] * data[b],
            '/': (a, b) => data[a] / data[b],
          };
        }

        console.log('operands', operands); // TODO
        console.log('operator', operator); // TODO

        console.log('return', operation[operator](operands.first, operands.second)); // TODO
        return operation[operator](operands.first, operands.second);
      
      } else {
        return calculation;
      }
    },
    parseExpression(expr) {
      return expr
        .replace(/this\.constant_/g, 'constant_')
        .replace(/this\.constants\./g, '')
        .split(' ');
    },
    updateValues(event) {
      // update objects when input is changed
      // const fieldname = event.target.name;
      // const fieldval = event.target.value;
      // this.constants[fieldname] = fieldval;
      // Object.entries(this.constants).forEach(([key, val]) => {
      //   if (this.hasCalculation(this.fieldData[key])) {
      //     this.constants[key] = this.getValue(this.fieldData[key]);
      //     this.fieldData[key].value = this.getValue(this.constants[key]);
      //     let icon = 'arrow_upward';
      //     let iconColor = 'red';
      //     if (val < this.constants[key]) {
      //       icon = 'arrow_downward';
      //       iconColor = 'green';
      //     }
      //     this.fieldData[key].append = icon;
      //     this.fieldData[key].appendColor = iconColor;
      //   }
      // });
      console.log('updateValues'); // TODO
      console.log('event', event); // TODO
      console.log('return', event.target.calculate); // TODO
      return event.target.calculate;
    },
    getValue(item, defaultVal) {
      // get values from updated object(s) to assign to form fields
      if (this.hasCalculation(item)) {
        // generate calculations to evaluate
        const ev = this.evalExpression(item.calculate, this.constants);
        console.log('ev', ev);
        return ev;
      }
      // console.log('ev (default)', defaultVal);
      return defaultVal;
    },
    hasCalculation(item) {
      if (Object.prototype.hasOwnProperty.call(item, 'calculate')) {
        return true;
      }
      return false;
    },
    hasSetValue(item) {
      if (Object.prototype.hasOwnProperty.call(item, 'value')) {
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
