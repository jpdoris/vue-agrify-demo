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
            :v-model="constants[index]"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
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
            :v-model="constants[index]"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
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
            :v-model="constants[index]"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
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
            :v-model="constants[index]"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :prefix="setPrefix(item.type)"
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
      constants: [],
      computedProps: {},
      fieldData: json.fields
    }
  },
  beforeMount() {
    // mount json to a data property
    // create another data set we can use for calculations
    const tmpConstants = {};
    Object.entries(this.fieldData).forEach(([key, val]) => {
      if (Object.prototype.hasOwnProperty.call(val, 'defaultValue')) {
        tmpConstants[key] = val.defaultValue;
      } else {
        tmpConstants[key] = null;
      }
      // this.$set(this.fieldData, [key].value, tmpConstants[key]);
    });
    this.constants = tmpConstants;
    console.log('beforeMount data', this.$data); // TODO
    this.calculateValues();
  },
  mounted() {
    console.log('constants mounted', this.constants); // TODO
    console.log('fieldData mounted', this.fieldData); // TODO
    // this.test();
  },
  filters: {
    format(item, type) {
      let digits = 0;
      if (type === 'float') {
        digits = 2;
      }
      const formatted = new Intl.NumberFormat('en-US', {
        style: 'decimal',
        maximumFractionDigits: digits,
        minimumFractionDigits: digits,
      }).format(item);
      return formatted;
    }
  },
  computed: {
    updated(newVal, oldVal) { // newVal, oldVal
      console.log('computed constants', this.constants); // TODO
      console.log('computed newVal', newVal); // TODO
      console.log('computed oldVal', oldVal); // TODO
      if (this.constants.length) {
        return this.constants.map(i => {
          return i;
        });
      } else {
        return false;
      }
    }
  },
  watch: {
    updated(newVal, oldVal) {
      console.log('watch newval', newVal); // TODO
      console.log('watch oldVal', oldVal); // TODO
    }
    // 'constants': {
    //   handler: 'calculateValues',
    //   deep: true
    // }
    // 'constants': {
    //   handler: function (newValue) { // newValue, oldValue
    //     // this.$set(this.constants, 'properties', this.constants);
    //     // this.constants[index] = newValue;
    //     // console.log('watch old', oldValue); // TODO
    //     console.log('watch new', newValue); // TODO
    //     // return newValue;
    //     this.constants = newValue;
    //   },
    //   deep: true
    // },
    // 'constants.constant_fte_weekly_hours': {
    //   handler: function (newValue) { // newValue, oldValue
    //     this.$set(this.constants, 'constant_fte_annual_hours.value', this.getValue(this.fieldData.constant_fte_annual_hours, 'constant_fte_annual_hours'));
    //     // this.constants[index] = newValue;
    //     // console.log('watch old', oldValue); // TODO
    //     console.log('watch new', newValue); // TODO
    //     // return newValue;
    //   },
    //   deep: true
    // },
    // constant_fte_weekly_hours() {
    //   const newVal = this.getValue(this.constants.constant_fte_annual_hours, 'constant_fte_annual_hours');
    //   console.log('newVal', newVal); // TODO
    //   return newVal;
    // },
    // 'constants.computedProps': {
    //   handler: function (newValue) { // oldValue
    //     this.$set(this.constants, 'computedProps', newValue);
    //   },
    //   deep: true
    // },
    // computed_constant_annual_lowSkilled_fte_per_asstManager_hours() {
    //   const newVal = this.getValue(this.constants.constant_annual_lowSkilled_fte_per_asstManager_hours, 'constant_annual_lowSkilled_fte_per_asstManager_hours');
    //   console.log('newVal', newVal); // TODO
    //   return newVal;
    // },
    // computed_constant_btus_per_kwatt() {
    //   const newVal = this.getValue(this.constants.constant_btus_per_kwatt, 'constant_btus_per_kwatt');
    //   console.log('newVal', newVal); // TODO
    //   return newVal;
    // },
    // computed_constant_days_in_month() {
    //   const newVal = this.getValue(this.constants.constant_days_in_month, 'constant_days_in_month');
    //   console.log('newVal', newVal); // TODO
    //   return newVal;
    // },
    // computed_constant_fte_annual_hours() {
    //   const newVal = this.getValue(this.constants.constant_fte_annual_hours, 'constant_fte_annual_hours');
    //   console.log('newVal', newVal); // TODO
    //   return newVal;
    // },
    // computed_constant_weeks_in_month() {
    //   const newVal = this.getValue(this.constants.constant_weeks_in_month, 'constant_weeks_in_month');
    //   console.log('newVal', newVal); // TODO
    //   return newVal;
    // }
  },
  methods: {
    test() {
      // console.log('number as string', Number("test")); // TODO
    // console.log('constants test', this.constants); // TODO
    // console.log('fieldData test', this.fieldData); // TODO
    },
    alert() {
      console.log('alert() triggered', this.constants); // TODO
    },
    calculateValues() {
      // need to process the data again to update the calculations that weren't available in the beforeMount() hook
      Object.entries(this.constants).forEach(([key, val]) => {
        // console.log('calculateValues val', val); // TODO
        let newVal = null;
        if (val === null) {
          // console.log('calculateValues key', key); // TODO
          // console.log('calculateValues val', val); // TODO
          // this.constants[key] = this.getValue(this.fieldData[key], this.fieldData[key].defaultValue);
          newVal = this.getValue(this.fieldData[key], key);
          this.$set(this.constants, key, newVal);
          this.$set(this.computedProps, key, newVal);
          // console.log('calculateValues constants', this.constants[key]); // TODO
          // console.log('calculateValues fieldData', this.fieldData[key]); // TODO
        }
        // this.$set(this.fieldData, key.value, newVal);
      });
    },
    processCalculation(calculation) {
      // console.log('processCalculation', calculation); // TODO
      // initialize
      let operator = '';
      const operands = {
        left: null,
        right: null
      }
      const parts = [];

      // 1: split calulation on spaces
      const split = this.parseExpression(calculation);

      // 2: find parentheses, preserve order and position
      split.forEach((item, index) => {
        if (item.match(/\(|\)/) !== null) {
          // has parentheses, need to eval
          const expr = item.replace(/\(|\)/g, '');
          
          // 3: eval parentheticals
          const operandsInner = this.parseExpression(expr);
          parts[index] = this.evalOperation(operandsInner[1], { left: operandsInner[0], right: operandsInner[2] });

        } else {
          // no parentheses, pass as is
          parts[index] = item;
        }
      });

      // 4: assign results to operands
      // console.log('processCalculation parts', parts); // TODO
      if (parts.length) {
        operands.left = parts[0];
        operands.right = parts[2];
        operator = parts[1];
      }

      // 5: evaluate operation
      // console.log('processCalculation result', this.evalOperation(operator, { left: operands.left, right: operands.right })); // TODO
      return this.evalOperation(operator, { left: operands.left, right: operands.right });
    },
    parseExpression(expr) {
      let parts = [];
      const leftParens = expr.match(/^\((.*)\)/);
      const rightParens = expr.match(/\((.*)\)$/);
      if (leftParens !== null) {
        let remainder = expr.replace(leftParens[0], '');
        remainder = this.splitExpression(remainder);
        parts[0] = leftParens[0];
        parts.push(remainder[0], remainder[1]);
      } else if (rightParens !== null) {
        const remainder = expr.replace(rightParens[0], '');
        parts = this.splitExpression(remainder);
        parts.push(rightParens[0]);
      } else {
        parts = this.splitExpression(expr);
      }

      return parts;
    },
    splitExpression(expr) {
      return expr
      .replace(/this\.constant_/g, 'constant_')
      .replace(/this\.constants\./g, '')
      .trim()
      .split(' ');
    },
    // getOperator(expr) {
    //   const operator = expr.match(/\+|-|\*|\//);
    //   return operator[0];
    // },
    evalOperation(operator, operands) {
      let left = operands.left;
      let right = operands.right;

      if (isNaN(+left)) {
        left = this.constants[left];
      } else {
        left = Number(left);
      }
      if (isNaN(+right)) {
        right = this.constants[right];
      } else {
        right = Number(right);
      }
      // console.log('eval left', left); // TODO
      // console.log('eval right', right); // TODO

      const operation = {
        '+': (left, right) => left + right,
        '-': (left, right) => left - right,
        '*': (left, right) => left * right,
        '/': (left, right) => left / right,
      };

      const result = operation[operator](left, right);
      // console.log('evalOperation operands', operands); // TODO
      // console.log('evalOperation , operator', operator); // TODO
      // console.log('evalOperation operation', result); // TODO
      return result;
    },
    getValue(item, key) {
      // console.log('getValue key', key); // TODO
      console.log('getValue item', item); // TODO
      // console.log('getValue fieldData', this.fieldData[item]); // TODO
      // get values from updated object(s) to assign to form fields
      let ev = this.fieldData[key].defaultValue;
      if (this.hasCalculation(item)) {
        // generate calculations to evaluate
        ev = this.processCalculation(item.calculate);
        // return ev;
      }
      // return defaultVal;
      this.constants[item] = ev;
      return ev;
    },
    setValue(item, defaultVal) {
      // console.log('getValue item', item); // TODO
      // console.log('getValue fieldData', this.fieldData[item]); // TODO
      // get values from updated object(s) to assign to form fields
      let ev;
      if (this.hasCalculation(item)) {
        // generate calculations to evaluate
        ev = this.processCalculation(item.calculate);
        // return ev;
      }
      // return defaultVal;
      ev = defaultVal;
      this.constants[item] = ev;
      // this.fieldData[item].value = ev;
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
.arrow-up {
  color: red;
}
.arrow-down {
  color: green;
}
</style>
