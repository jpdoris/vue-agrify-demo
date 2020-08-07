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
            :v-model="getValue(index, item.defaultValue)"
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
            :v-model="getValue(index, item.defaultValue)"
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
            :v-model="getValue(index, item.defaultValue)"
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
            :v-model="getValue(index, item.defaultValue)"
            :label="item.label"
            :type="item.type"
            :readonly="item.readOnly"
            :prefix="setPrefix(item.type)"
            :append-outer-icon="item.append"
            @input="updateValue(index, item.defaultValue)"
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
          value: ''
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
      // console.log('beforeMount fieldData key', key); // TODO
      this.fieldData[key] = val;
      if (Object.prototype.hasOwnProperty.call(val, 'defaultValue')) {
        tmpConstants[key] = val.defaultValue;
      } else {
        tmpConstants[key] = null;
      }
    });
    this.constants = tmpConstants;
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
    updateValue(item, defaultVal) {
      return this.setValue(item, defaultVal);
    }
  },
  methods: {
    test() {
      // console.log('number as string', Number("test")); // TODO
    // console.log('constants test', this.constants); // TODO
    // console.log('fieldData test', this.fieldData); // TODO
    },
    fetchData(item) {
      // console.log('fetchData'); // TODO
      this.constants = new Function(this.fieldData[item].calculate);
    },
    calculateValues() {
      // need to process the data again to update the calculations that weren't available in the beforeMount() hook
      Object.entries(this.constants).forEach(([key, val]) => {
        console.log('calculateValues val', val); // TODO
        if (val === null) {
          // console.log('calculateValues key', key); // TODO
          // console.log('calculateValues val', val); // TODO
          this.$data.constants[key] = this.getValue(this.fieldData[key], this.fieldData[key].defaultValue);
          this.$data.fieldData[key].value = this.$data.constants[key];
          
          setTimeout(() => {
            console.log('calculateValues result', this.$data.constants[key]); // TODO
          }, 1000);
        }
      });
    },
    processCalculation(calculation) {
      console.log('processCalculation', calculation); // TODO
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
      console.log('processCalculation parts', parts); // TODO
      if (parts.length) {
        operands.left = parts[0];
        operands.right = parts[2];
        operator = parts[1];
      }

      // 5: evaluate operation
      console.log('processCalculation result', this.evalOperation(operator, { left: operands.left, right: operands.right })); // TODO
      return this.evalOperation(operator, { left: operands.left, right: operands.right });
    },
    parseExpression(expr) {
      let parts = [];
      const leftParens = expr.match(/^\((.*)\)/);
      const rightParens = expr.match(/\((.*)\)$/);
      if (leftParens !== null) {
        const remainder = expr
          .replace(leftParens[0], '')
          .replace(/this\.constant_/g, 'constant_')
          .replace(/this\.constants\./g, '')
          .trim()
          .split(' ');
        parts[0] = leftParens[0];
        parts.push(remainder[0], remainder[1]);
      } else if (rightParens !== null) {
        const remainder = expr
          .replace(rightParens[0], '')
          .replace(/this\.constant_/g, 'constant_')
          .replace(/this\.constants\./g, '')
          .trim()
          .split(' ');
        parts = remainder;
        parts.push(rightParens[0]);
      } else {
        return expr
          .replace(/this\.constant_/g, 'constant_')
          .replace(/this\.constants\./g, '')
          .split(' ');
      }

      return parts;
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
      console.log('eval left', left); // TODO
      console.log('eval right', right); // TODO

      const operation = {
        '+': (left, right) => left + right,
        '-': (left, right) => left - right,
        '*': (left, right) => left * right,
        '/': (left, right) => left / right,
      };

      const result = operation[operator](left, right);
      console.log('evalOperation operands', operands); // TODO
      console.log('evalOperation , operator', operator); // TODO
      console.log('evalOperation operation', result); // TODO
      return result;
    },
    getValue(item, defaultVal) {
      console.log('getValue item', item); // TODO
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
