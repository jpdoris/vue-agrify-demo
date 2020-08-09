<template>
  <v-container>
    <v-row class="mt-4">
      <v-col cols="3" class="px-5">
        <div class="subheading">Time</div>
        <div v-for="(value, name) in fieldData" :key="name">
          <v-text-field
            v-if="fieldData[name].category === 'Time'"
            :name="name"
            :label="value.label"
            :type="value.type"
            :readonly="value.readOnly"
            :prefix="setPrefix(value.type)"
            :append-outer-icon="value.append"
            v-model.lazy="formattedConstants[name]"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Heat/Cooling</div>
        <div v-for="(value, name) in fieldData" :key="name">
          <v-text-field
            v-if="fieldData[name].category === 'Heat/Cooling'"
            :name="name"
            :label="value.label"
            :type="value.type"
            :readonly="value.readOnly"
            :prefix="setPrefix(value.type)"
            :append-outer-icon="value.append"
            v-model.lazy="formattedConstants[name]"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Weights and Measures</div>
        <div v-for="(value, name) in fieldData" :key="name">
          <v-text-field
            v-if="fieldData[name].category === 'Weights and Measures'"
            :name="name"
            :label="value.label"
            :type="value.type"
            :readonly="value.readOnly"
            :prefix="setPrefix(value.type)"
            :append-outer-icon="value.append"
            v-model.lazy="formattedConstants[name]"
          />
        </div>
      </v-col>
      <v-col cols="3" class="px-5">
        <div class="subheading">Staff Expense</div>
        <div v-for="(value, name) in fieldData" :key="name">
          <v-text-field
            :ref="name"
            v-if="value.category === 'Staff Expense'"
            :name="name"
            :label="value.label"
            :type="value.type"
            :readonly="value.readOnly"
            :prefix="setPrefix(value.type)"
            :append-outer-icon="value.append"
            v-model.lazy="formattedConstants[name]"
          />
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import json from '@/data/constants.json';
import _ from 'lodash';

export default {
  name: 'InteractiveForm',
  data() {
    return {
      constants: {},
      formattedConstants: {},
      fieldData: {}
    }
  },
  beforeMount() {
    // mount json data to fields obj
    this.fieldData = json.fields;
    // create another data model we can use for calculations
    Object.entries(this.fieldData).map(([key, val]) => {
      if (Object.prototype.hasOwnProperty.call(val, 'defaultValue')) {
        this.$set(this.constants, key, val.defaultValue);
        this.$set(this.formattedConstants, key, this.numberOut(val.defaultValue, val.type));
      } else {
        this.$set(this.constants, key, null);
        this.$set(this.formattedConstants, key, null);
      }
    });
  },
  mounted() {
    this.calculateValues();
  },
  computed: {
    // watcher-helper: recreate constants as a new object to maintain reactivity
    computedConstants: function() {
      return Object.assign({}, this.formattedConstants);
    }
  },
  watch: {
    // watch constants for changes, then recalculate
    computedConstants: {
      deep: true,
      handler(newVal, oldVal) {        
        const vm = this;
        const changed = _.transform(newVal, function(result, value, key) {
          if (!_.isEqual(value, oldVal[key])) {
            result[key] = (_.isObject(value) && _.isObject(oldVal[key])) ? changed(value, oldVal[key]) : value
            vm.constants[key] = vm.numberIn(value);
          }
        });

        const propList = Object.getOwnPropertyNames(changed);
        if (propList.length === 1) {
          const deps = this.findDependencies(propList[0]);
          for (const item of deps.values()) {
            this.constants[item] = this.getValue(this.fieldData[item]);
            this.formattedConstants[item] = this.numberOut(this.getValue(this.fieldData[item]), this.fieldData[item].type);
          }
        }
      }
    }
  },
  methods: {
    findDependencies(property) {
      const vm = this;
      const deps = Object.keys(this.fieldData).filter(function(row) {
        if (Object.prototype.hasOwnProperty.call(vm.fieldData[row], 'calculate')) {
          if (vm.fieldData[row].calculate.indexOf(property) !== -1) {
            return row;
          }
        }
      });
      return deps;
    },
    calculateValues() {
      // need to process the data again to update the calculations that weren't available in the beforeMount() hook
      Object.entries(this.constants).forEach(([key, val]) => {
        if (val === null) {
          this.$set(this.constants, key, this.getValue(this.fieldData[key]));
          this.$set(this.formattedConstants, key, this.numberOut(this.getValue(this.fieldData[key]), this.fieldData[key].type));
        }
      });
    },
    processCalculation(calculation) {
      // initialize
      let operator = '';
      const operands = {
        left: null,
        right: null
      }
      const parts = [];

      // 1: split calculation on spaces
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
      if (parts.length) {
        operands.left = parts[0];
        operands.right = parts[2];
        operator = parts[1];
      }

      // 5: evaluate operation
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
    evalOperation(operator, operands) {
      let left = operands.left;
      let right = operands.right;

      if (isNaN(+left)) {
        left = this.numberIn(this.constants[left]);
      } else {
        left = this.numberIn(left);
      }
      if (isNaN(+right)) {
        right = this.numberIn(this.constants[right]);
      } else {
        right = this.numberIn(right);
      }

      const operation = {
        '+': (left, right) => left + right,
        '-': (left, right) => left - right,
        '*': (left, right) => left * right,
        '/': (left, right) => left / right,
      };

      const result = operation[operator](left, right);
      return result;
    },
    getValue(item) {
      // generate calculations to evaluate
      let result = item.defaultValue;
      if (Object.prototype.hasOwnProperty.call(item, 'calculate')) {
        result = this.processCalculation(item.calculate);
      }
      return result;
    },
    setPrefix(type) {
      return type === 'percent' ? '%' : '';
    },
    numberIn(val) {
      const num = Number(val.toString().replace(/,/g, ''));
      return num;
    },
    numberOut(val, type) {
      let digits = 0;
      if (type === 'float') {
        digits = 2;
      }
      const result = new Intl.NumberFormat('en-US', {
        style: 'decimal',
        maximumFractionDigits: digits,
        minimumFractionDigits: digits,
      }).format(val);

      return result;
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
