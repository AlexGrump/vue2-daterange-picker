<template>
<div class="time-picker">
  <div class="form-row flex-nowrap align-items-center">
    <div class="col-auto flex-grow-1 flex-shrink-1">
      <select 
        class="form-control form-control-sm time-picker-input"
        v-model="hours"
        :disabled="!value"
      >
        <option
          v-for="item in hoursOptions"
          :key="item"
          :value="item"
        >{{ item }}</option>
      </select>
    </div>
    <div class="col-auto">:</div>
    <div class="col-auto flex-grow-1 flex-shrink-1">
      <select 
        class="form-control form-control-sm time-picker-input"
        v-model="minutes"
        :disabled="!value"
      >
        <option
          v-for="item in minutesOptions"
          :key="item"
          :value="item"
        >{{ item | paddy }}</option>
      </select>
    </div>
    <template v-if="showSeconds">
      <div class="col-auto">:</div>
      <div class="col-auto flex-grow-1 flex-shrink-1">
        <select 
          class="form-control form-control-sm time-picker-input"
          v-model="seconds"
          :disabled="!value"
        >
          <option
            v-for="item in secondsOptions"
            :key="item"
            :value="item"
          >{{ item | paddy }}</option>
        </select>
      </div>
    </template>
  </div>
</div>
</template>

<script>
import moment from 'moment'

function paddy(num, padlen = 2, padchar = '0') {
  let pad = new Array(1 + padlen).join(padchar);

  return (pad + num).slice(-pad.length);
}

export default {
  props: {
    value: {
      type: [ String, Object, Date ],
      default() {
        return moment()
      }
    },

    showSeconds: {
      type: Boolean,
      default: false
    },

    hoursFormat24: {
      type: Boolean,
      default: true,
    },

    step: {
      type: Number,
      default: 5
    } 
  },

  filters: {
    paddy(num, padlen, padchar) {
      return paddy(num, padlen, padchar)
    }
  },

  data() {
    const secondsOptions = []
    const minutesOptions = []
    const hoursOptions = []
    const maxHours = this.hoursFormat24 ? 24 : 12

    for (let i = 0; i < 60; i = i + this.step) {
      secondsOptions.push(i)
    }

    for (let i = 0; i < 60; i = i + this.step) {
      minutesOptions.push(i)
    }

    for (let i = 0; i < maxHours; i++) {
      hoursOptions.push(i)
    }

    return {
      secondsOptions,
      minutesOptions,
      hoursOptions,
    }
  },

  computed: {
    seconds: {
      get() {
        return moment(this.value).seconds()
      },
      set(val) {
        this.$emit('input', moment(this.value).seconds(val))
      }
    },

    minutes: {
      get() {
        return moment(this.value).minutes()
      },
      set(val) {
        this.$emit('input', moment(this.value).minutes(val))
      }
    },

    hours: {
      get() {
        return moment(this.value).hours()
      },
      set(val) {
        this.$emit('input', moment(this.value).hours(val))
      }
    }
  },

  methods: {
    paddy(num, padlen, padchar) {
      return paddy(num, padlen, padchar)
    }
  }
}
</script>

<style lang="scss" scoped>
.time-picker {
  width: 100%;

  &-input {
    min-width: 0;
    padding-left: 0;
    padding-right: 0;
    text-align: center;
  }
}
</style>