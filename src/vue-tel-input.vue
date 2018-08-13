<template>
  <div class="vue-tel-input"
       :class="{ disabled: disabled }">
    <div class="dropdown"
         @click="toggleDropdown"
         v-click-outside="clickedOutside"
         :class="{open: open}">
      <span class="selection">
        <div class="iti-flag"
             :class="activeCountry.iso2.toLowerCase()"></div>
        <span class="dropdown-arrow">
          {{ open ? '▲' : '▼' }}
        </span>
      </span>
      <ul v-show="open">
        <li class="dropdown-item"
            v-for="pb in allCountries"
            :key="pb['iso2']"
            @click="choose(pb)">
          <div class="iti-flag"
               :class="pb.iso2.toLowerCase()"></div>
          <strong>{{ pb.name }} </strong>
          <span>+{{ pb.dialCode }}</span>
        </li>
      </ul>
    </div>
    <input v-model="phone"
           :placeholder="placeholder"
           :state="isValid"
           :formatter="format"
           :disabled="disabled"
           @blur="onBlur"
           @input="onInput">
  </div>
</template>

<style src="./assets/sprite.css"></style>
<style scoped>
:local {
  --border-radius: 2px;
}
.iti-flag {
  margin-right: 5px;
  margin-left: 5px;
}
.dropdown-item .iti-flag {
  display: inline-block;
  margin-right: 5px;
}
.selection {
  cursor: pointer;
  font-size: 0.8em;
  display: flex;
  align-items: center;
}
.vue-tel-input {
  border-radius: 3px;
  display: flex;
  border: 1px solid #bbb;
  text-align: left;
}
.vue-tel-input:focus-within {
  box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075),
    0 0 8px rgba(102, 175, 233, 0.6);
  border-color: #66afe9;
}
input {
  border: none;
  border-radius: 0 var(--border-radius) var(--border-radius) 0;
  width: 100%;
  outline: none;
  padding-left: 7px;
}
ul {
  padding: 0;
  margin: 0;
  text-align: left;
  list-style: none;
  max-height: 200px;
  overflow-y: scroll;
  position: absolute;
  top: 33px;
  left: -1px;
  background-color: #fff;
  border: 1px solid #ccc;
  width: 390px;
}
.dropdown {
  display: flex;
  flex-direction: column;
  align-content: center;
  justify-content: center;
  position: relative;
  padding: 7px;
}
.dropdown.open {
  background-color: #f3f3f3;
}
.dropdown:hover {
  background-color: #f3f3f3;
}
.dropdown-arrow {
  transform: scaleY(0.5);
  display: inline-block;
  color: #666;
}
.dropdown-item {
  cursor: pointer;
  padding: 4px 15px;
}
.dropdown-item:hover {
  background-color: #f3f3f3;
}
.dropdown-menu.show {
  max-height: 300px;
  overflow: scroll;
}
.vue-tel-input.disabled .selection,
.vue-tel-input.disabled .dropdown,
.vue-tel-input.disabled input {
  cursor: no-drop;
}
</style>

<script>
import { parseNumber, formatNumber } from 'libphonenumber-js';
import allCountries from './assets/all-countries';
import getCountry from './assets/default-country';

export default {
  name: 'vue-tel-input',
  props: {
    value: {
      type: String,
    },
    placeholder: {
      type: String,
      default: 'Enter a phone number',
    },
    disabledFetchingCountry: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
  },
  mounted() {
    console.log('MOUNTED 3!!!!');
    console.log('this.activeCountry: ', this.activeCountry);

    if (!this.disabledFetchingCountry) {
      getCountry().then((res) => {
        this.activeCountry = allCountries.find(country => country.iso2 === res) ||
          allCountries[0];
      });
    }
  },
  created() {
    if (this.value) {
      this.phone = this.value;
    }
  },
  data() {
    return {
      phone: '',
      allCountries,
      activeCountry: allCountries[0], // TODO Make this U.S. ???
      open: false,
    };
  },
  computed: {
    mode() {
      if (!this.phone) {
        return '';
      }
      if (this.phone[0] === '+') {
        return 'code';
      }
      if (this.phone[0] === '0') {
        return 'prefix';
      }
      return 'normal';
    },

    numbersOnly() {
      // The model value, without any spaces, hyphens, or parens.
      // A leading plus sign is allowed.
      const numbersOnly = this.phone.replace(/\(|\)|\s|-/g, '');

      console.log('numbersOnly: ', numbersOnly); // WHY ISN'T THIS LOGGING OUT?

      return numbersOnly;
    },

    parsedNumber() {
      // eslint-disable-next-line prefer-destructuring
      let phone = this.phone;
      // Remove any leading zero.
      if (this.mode === 'prefix') {
        phone = this.phone.slice(1);
      }

      let parsedNumber;
      // If user is manually typing the country code, infer the country and
      // try to find it in the countries list.
      if (this.mode === 'code') {
        parsedNumber = parseNumber(phone);
        if (parsedNumber.country && this.allCountries) {
          const matchedCountry = this.allCountries.find(e =>
            e.iso2.toUpperCase() === parsedNumber.country);
          if (matchedCountry) {
            // eslint-disable-next-line vue/no-side-effects-in-computed-properties
            this.activeCountry = matchedCountry;
          }
        }
      // Else parse based on the current activeCountry.
      } else {
        parsedNumber = parseNumber(phone, this.activeCountry.iso2);
      }

      console.group('parsedNumber');
      console.log('phone: ', phone);
      console.log('this.activeCountry: ', this.activeCountry);
      console.log('parsedNumber: ', parsedNumber);
      console.groupEnd();

      return parsedNumber;
    },

    isValid() {
      // Note that this is a very loose validity check.
      console.log('isValid: ', !!this.parsedNumber.phone);
      return !!this.parsedNumber.phone;
    },

    formattedNumber() {
      const fNum = this.parsedNumber.phone ?
        formatNumber(this.parsedNumber, 'National') : '';
      console.log('formattedNumber: ', fNum);
      return fNum;
    },

    response() {
      return {
        number: this.parsedNumber,
        isValid: this.isValid,
        country: this.activeCountry,
      };
    },
  },

  watch: {
    isValid(value) {
      console.log('watched isValid is: ', value);

      // When the parsed input is valid, overwrite the model with the
      // formatted string.
      if (value) {
        console.log('formatting the input');
        this.phone = this.formattedNumber;
      }
    },
  },

  methods: {
    // TODO Write a method that listens for letter typing when the dropdown is
    // open, and auto scrolls to the first country with that first letter,
    // and auto-selects it.

    choose(country) {
      this.activeCountry = country;
      this.$emit('onInput', this.response);
    },
    onInput() {
      // Emit the input event in case v-model is used in the parent
      this.$emit('input', this.response.number);

      // Emit the response, includes phone, validity and country
      this.$emit('onInput', this.response);
    },
    onBlur() {
      this.$emit('onBlur');
    },
    toggleDropdown() {
      if (this.disabled) {
        return;
      }
      this.open = !this.open;
    },
    clickedOutside() {
      this.open = false;
    },
  },

  directives: {
    // Click-outside from BosNaufal: https://github.com/BosNaufal/vue-click-outside
    'click-outside': {
      bind(el, binding, vNode) {
        // Provided expression must evaluate to a function.
        if (typeof binding.value !== 'function') {
          let warn = `[Vue-click-outside:] provided expression ${binding.expression}
             is not a function, but has to be`;
          const compName = vNode.context.name;
          if (compName) {
            warn += `Found in component ${compName}`;
          }
          console.warn(warn);
        }
        // Define Handler and cache it on the element
        const { bubble } = binding.modifiers;
        function handler(e) {
          if (bubble || (!el.contains(e.target) && el !== e.target)) {
            binding.value(e);
          }
        }
        // eslint-disable-next-line
        el.__vueClickOutside__ = handler;
        // Add Event Listeners.
        document.addEventListener('click', handler);
      },
      unbind(el) {
        // Remove Event Listeners.
        // eslint-disable-next-line no-underscore-dangle
        document.removeEventListener('click', el.__vueClickOutside__);
        // eslint-disable-next-line no-underscore-dangle, no-param-reassign
        el.__vueClickOutside__ = null;
      },
    },
  },
};
</script>
