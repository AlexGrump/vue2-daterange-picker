<template>
    <div style="position: relative; display: inline-block;">
        <div class="form-control" @click="togglePicker">
            <slot
                name="input"
                :startDate="pickedDate.start"
                :endDate="pickedDate.end"
                :ranges="ranges"
            >
                <i class="glyphicon glyphicon-calendar fa fa-calendar"></i>&nbsp;
                <span>{{startText}} - {{endText}}</span>
                <b class="caret"></b>
            </slot>
        </div>
        <transition name="slide-fade" mode="out-in">
            <div
                class="daterangepicker dropdown-menu ltr show-ranges"
                :class="pickerStyles()"
                v-if="open"
                v-on-clickaway="clickAway"
            >
                <div class="calendars">
                    <calendar-ranges
                      @clickRange="clickRange"
                      :ranges="ranges"
                    ></calendar-ranges>

                    <div class="drp-calendar left">
                        <div class="daterangepicker_input hidden-xs" v-if="false">
                            <input 
                              class="input-mini form-control" 
                              type="text" 
                              name="daterangepicker_start"
                              :value="startText"/>
                            <i class="fa fa-calendar glyphicon glyphicon-calendar"></i>
                        </div>
                        <time-picker 
                          v-if="time"
                          :showSeconds="seconds"
                          v-model="start" 
                          class="px-2 mb-2"
                        ></time-picker>
                        <div class="calendar-table">
                            <calendar 
                              :monthDate="monthDate"
                              :locale="locale"
                              :start="pickedDate.start" :end="pickedDate.end"
                              :minDate="min" :maxDate="max"
                              @nextMonth="nextMonth" @prevMonth="prevMonth"
                              @dateClick="dateClick" @hoverDate="hoverDate"
                            ></calendar>
                        </div>
                    </div>

                    <div class="drp-calendar right hidden-xs">
                        <div class="daterangepicker_input" v-if="false">
                            <input 
                              class="input-mini form-control" 
                              type="text" name="daterangepicker_end"
                              :value="endText"/>
                            <i class="fa fa-calendar glyphicon glyphicon-calendar"></i>
                        </div>
                        <time-picker 
                          v-if="time"
                          :showSeconds="seconds"
                          v-model="end" 
                          class="px-2 mb-2"
                        ></time-picker>
                        <div class="calendar-table">
                            <calendar 
                              :monthDate="nextMonthDate"
                              :locale="locale"
                              :start="pickedDate.start" :end="pickedDate.end"
                              :minDate="min" :maxDate="max"
                              @nextMonth="nextMonth" @prevMonth="prevMonth"
                              @dateClick="dateClick" @hoverDate="hoverDate"
                            ></calendar>
                        </div>
                    </div>
                </div>

                <div class="drp-buttons">
                    <button
                      class="applyBtn btn btn-sm btn-success"
                      :disabled="in_selection"
                      type="button"
                      @click="clickedApply"
                    >{{locale.applyLabel}}</button>
                    <button
                      class="cancelBtn btn btn-sm btn-default"
                      type="button"
                      @click="clickedCancel"
                    >{{locale.cancelLabel}}</button>
                </div>

            </div>
        </transition>
    </div>
</template>

<script>
import moment                   from 'moment'
import TimePicker               from './TimePicker'
import Calendar                 from './Calendar.vue'
import CalendarRanges           from './CalendarRanges'
import { nextMonth, prevMonth } from './util'
import { mixin as clickaway }   from 'vue-clickaway'

export default {
  components: {
    Calendar, 
    CalendarRanges,
    TimePicker
  },

  mixins: [
    clickaway
  ],

  props: {
    minDate: [ String, Object ],

    maxDate: [ String, Object ],

    localeData: {
      type: Object,
      default () {
        return {}
      },
    },

    startDate: {
      default () {
        return new Date()
      }
    },

    endDate: {
      default () {
        return new Date()
      }
    },

    ranges: {
      type: Object,
      default () {
        return {
          'Today': [moment(), moment()],
          'Yesterday': [moment().subtract(1, 'days'), moment().subtract(1, 'days')],
          'This month': [moment().startOf('month'), moment().endOf('month')],
          'This year': [moment().startOf('year'), moment().endOf('year')],
          'Last week': [moment().subtract(1, 'week').startOf('week'), moment().subtract(1, 'week').endOf('week')],
          'Last month': [moment().subtract(1, 'month').startOf('month'), moment().subtract(1, 'month').endOf('month')],
        }
      }
    },

    opens: {
      type: String,
      default: 'center'
    },

    time: {
      type: Boolean,
      default: true
    },

    seconds: {
      type: Boolean,
      default: false
    }
  },

  data () {
    const default_locale = {
      direction: 'ltr',
      format: 'M/DD/YYYY hh:mm A',
      separator: ' - ',
      applyLabel: 'Apply',
      cancelLabel: 'Cancel',
      weekLabel: 'W',
      customRangeLabel: 'Custom Range',
      daysOfWeek: moment.weekdaysMin(),
      monthNames: moment.monthsShort(),
      firstDay: moment.localeData().firstDayOfWeek()
    }

    const data = {
      locale: {
        ...default_locale, 
        ...this.localeData
      },

      monthDate: new Date(this.startDate),
      start: new Date(this.startDate),
      end: new Date(this.endDate),
      in_selection: false,
      open: false
    }

    // update day names order to firstDay
    if (data.locale.firstDay !== 0) {
      let iterator = data.locale.firstDay
      while (iterator > 0) {
        data.locale.daysOfWeek.push(data.locale.daysOfWeek.shift())
        iterator--
      }
    }

    return data
  },

  computed: {
    pickedDate() {
      const dates = [
        new Date(this.start),
        new Date(this.end)
      ];

      dates.sort((a, b) => {
        return b - a;
      })

      return {
        start: dates[1],
        end: dates[0]
      }
    },

    nextMonthDate() {
      return nextMonth(this.monthDate)
    },

    startText() {
      return moment(new Date(this.pickedDate.start)).format(this.locale.format)
    },

    endText() {
      return moment(new Date(this.pickedDate.end)).format(this.locale.format)
    },

    min() {
      return this.minDate ? new Date(this.minDate) : null
    },

    max() {
      return this.maxDate ? new Date(this.maxDate) : null
    }
  },

  watch: {
    startDate(value) {
      this.start = new Date(value)
    },

    endDate(value) {
      this.end = new Date(value)
    }
  },
  
  methods: {
    nextMonth() {
      this.monthDate = nextMonth(this.monthDate)
    },

    prevMonth() {
      this.monthDate = prevMonth(this.monthDate)
    },

    dateClick(value) {
      if (this.in_selection) {
        this.in_selection = false
        this.end = new Date(value)
      } else {
        this.in_selection = true
        this.start = new Date(value)
        this.end = new Date(value)
      }
    },

    hoverDate(value) {
      if(this.in_selection) {
        this.end = new Date(value)
      }
    },

    togglePicker() {
      this.open = !this.open
    },

    pickerStyles() {
      return {
        'show-calendar': this.open,
        opensright: this.opens === 'right',
        opensleft: this.opens === 'left',
        openscenter: this.opens === 'center'
      }
    },

    clickedApply() {
      this.open = false
      this.$emit('update', { 
        startDate: this.pickedDate.start, 
        endDate: this.pickedDate.end
      })
    },

    clickedCancel() {
      this.open = false
      this.start = this.startDate
      this.end = this.endDate
    },

    clickAway() {
      if (this.open) {
        this.open = false
      }
    },

    clickRange(value) {
      this.start = new Date(value[0])
      this.end = new Date(value[1])
      this.monthDate = new Date(value[0])
      this.clickedApply()
    }
  }
}
</script>

<style lang="scss">
    @import '../assets/daterangepicker.css';
</style>

<style lang="scss" scoped>
    .reportrange-text {
        background: #fff;
        cursor: pointer;
        padding: 5px 10px;
        border: 1px solid #ccc;
        width: 100%;
    }

    .daterangepicker{
        flex-direction: column;
        display: flex;
        width: auto;

        &.show-calendar {
            display: block;
        }
    }

    .calendars {
        display: flex;
    }

    div.daterangepicker.opensleft {
        top: 35px;
        right: 10px;
        left: auto;
    }

    div.daterangepicker.openscenter {
        top: 35px;
        right: auto;
        left: 50%;
        transform: translate(-50%, 0);
    }

    div.daterangepicker.opensright {
        top: 35px;
        left: 10px;
        right: auto;
    }

    /* Enter and leave animations can use different */
    /* durations and timing functions.              */
    .slide-fade-enter-active {
        transition: all .2s ease;
    }

    .slide-fade-leave-active {
        transition: all .1s cubic-bezier(1.0, 0.5, 0.8, 1.0);
    }

    .slide-fade-enter, .slide-fade-leave-to
        /* .slide-fade-leave-active for <2.1.8 */
    {
        transform: translateX(10px);
        opacity: 0;
    }
</style>
