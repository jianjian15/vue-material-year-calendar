<template>
  <div class="vue-calendar__container">

    <div class="container__months" v-for="cur in yearList">
      <month-calendar
        class="container__month"
        v-for="n in monthList(cur)"
        :key="`year-${cur}-month-${n}`"
        :year="cur"
        :month="n"
        :activeDates="month[cur + '-' + n]"
        :activeClass="activeClass"
        @toggleDate="toggleDate"
        :lang="lang"
        :prefixClass="prefixClass"
      >
      </month-calendar>
    </div>
  </div>
</template>

<script>
import dayjs from 'dayjs'
import MonthCalendar from './MonthCalendar'
export default {
  name: 'year-calendar',
  props: {
    showYearSelector: {
      type: Boolean,
      default: () => true
    },
    activeDates: {
      type: Array,
      default: () => [],
      validator: (dateArray) => {
        let isGood = true
        let curDate = null

        dateArray.forEach(date => {
          if (typeof date === 'string') {
            curDate = date
          } else if (typeof date === 'object' && date.hasOwnProperty('date')) {
            curDate = date.date
          }

          // 以下程式碼參考「How to validate date with format mm/dd/yyyy in JavaScript?」in Stackoverflow
          // 由於 「^\d{4}\-\d{1,2}\-\d{1,2}$」會被ESLint 判為錯誤，所以暫時關閉 EsLint 對下一行的驗證 by丁丁
          // eslint-disable-next-line no-useless-escape
          if (!/^\d{4}\-\d{1,2}\-\d{1,2}$/.test(curDate)) {
            isGood = false
          }
          // Parse the date parts to integers
          var parts = curDate.split('-')
          var day = parseInt(parts[2], 10)
          var month = parseInt(parts[1], 10)
          var year = parseInt(parts[0], 10)

          if (year < 1000 || year > 3000 || month === 0 || month > 12) isGood = false
          let monthLength = [ 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 ]
          if (year % 400 === 0 || (year % 100 !== 0 && year % 4 === 0)) monthLength[1] = 29
          if (!(day > 0 && day <= monthLength[month - 1])) isGood = false
        });
        return isGood
      }
    },
    // value 為從外層傳進來的 v-model="year"
    value: {
      type: [String, Number],
      default: dayjs().year()
    },
    startDate: {
      type: [String]
    },
    endDate: {
      type: [String]
    },
    lang: {
      type: String,
      default: 'en'
    },
    activeClass: {
      type: String,
      default: () => ''
    },
    prefixClass: {
      type: String,
      default: () => 'calendar--active'
    }
  },
  data () {
    return {
      isUsingString: true,
      startYear: null,
      endYear: null
    }
  },
  components: {
    MonthCalendar
  },
  computed: {
    month () {
      const month = {}
      this.activeDates.forEach(date => {
        let oDate

        if (typeof date === 'string') {
          oDate = {
            date: date,
            className: this.activeClass
          }
        } else {
          // 若 activeDate 裡的物件少了 className 屬性，就自動填入空字串。否則會變成undefined
          oDate = {
            date: date.date,
            className: date.className || ''
          }
        }

        let y = dayjs(oDate.date).year().toString();
        let m = (dayjs(oDate.date).month() + 1).toString()
        let key = y + "-" + m;
        if(!month[key]) month[key] = [];
        month[key].push(oDate)
      });
      return month
    },
    yearList(){
      const list = [];
      if(this.startDate && this.endDate) {
        let startDayJs = new dayjs(this.startDate);
        let endDayJs = new dayjs(this.endDate);
        let startYear = startDayJs.year();
        let endYear = endDayJs.year();
        for(let i = startYear; i <= endYear; i++){
          list.push(i);
        }
      }
      return list;
    },
    activeYear: {
      get () {
        return parseInt(this.value) // this.value 為從外層傳進來的 v-model="year"
      },
      set (val) {
        this.$emit('input', val)
      }
    }
  },
  methods: {
    changeYear (idx) {
      this.activeYear = idx + this.activeYear - 3
    },
    toggleDate (dateObj) {
      const activeDate = dayjs()
        .set('year', dateObj.year)
        .set('month', dateObj.month - 1)
        .set('date', dateObj.date)
        .format('YYYY-MM-DD')
      this.$emit('toggleDate', {
        date: activeDate,
        selected: dateObj.selected,
        className: dateObj.className
      })

      let dateIndex
      let newDates

      if (this.isUsingString) {
        dateIndex = this.activeDates.indexOf(activeDate)
        newDates = this.modifiedActiveDates(dateIndex, activeDate)
      } else {
        let oDate = {
          date: activeDate,
          className: dateObj.className // 原為 this.defaultClassName ，修正bug(丁丁)
        }

        dateIndex = this.activeDates.indexOf(this.activeDates.find((i) => i.date === activeDate))
        newDates = this.modifiedActiveDates(dateIndex, oDate)
      }
      console.log(newDates);
      this.$emit('update:activeDates', newDates)
    },
    monthList(curYear){
      const monthList = [];

      if(this.startDate && this.endDate) {
        let startDayJs = new dayjs(this.startDate);
        let endDayJs = new dayjs(this.endDate);
        //是同一年就是
        let startYear = startDayJs.year();
        let endYear = endDayJs.year();
        let startMonth = startDayJs.month() + 1;
        let endMonth = endDayJs.month() + 1;

        //开始年和当前要计算的年不是同一年就从0开始
        startMonth = curYear === startYear ? startMonth : 1;
        //结束年和当前要计算的年不是同一年就从11结束
        endMonth = curYear === endYear ? endMonth : 12;

        for(let i = startMonth; i <= endMonth; i++){
          monthList.push(i);
        }
      }

      return monthList;
    },
    modifiedActiveDates (dateIndex, activeDate) {
      let newDates = [...this.activeDates]
      if (dateIndex === -1) {
        newDates.push(activeDate)
      } else {
        newDates.splice(dateIndex, 1)
      }
      return newDates
    }
  },
  mounted () {
    if(this.startDate) {
      let startDayJs = new dayjs(this.startDate);
      let endDayJs = new dayjs(this.endDate);
      //是同一年就是
      this.startYear = startDayJs.year();
      this.endYear = endDayJs.year();
    }
    else {
      this.endYear = this.startYear = datajs().year();
    }
  },
  created () {
    this.isUsingString = this.activeDates.length && typeof this.activeDates[0] === 'string'
  }
}
</script>

<style lang="stylus" scoped>
.vue-calendar__container
  border-radius: 2px
  min-width: 0
  position: relative
  text-decoration: none
  box-shadow: 0 2px 1px -1px rgba(0,0,0,.2), 0 1px 1px 0 rgba(0,0,0,.14), 0 1px 3px 0 rgba(0,0,0,.12)
  background-color #F6F6F3
  .container__year
    user-select none
    height 65px
    background-color #fff
    font-size 24px
    flex 100%
    text-align center
    display flex
    .year__chooser
      height 100%
      flex: 1
      cursor pointer
      display flex
      align-items center
      justify-content center
      color rgba(black, 0.9)
      &:hover
        background-color rgba(#666, 0.1)
      &:nth-child(4n-3)
        color rgba(black, 0.3)
      &:nth-child(2n)
        color rgba(black, 0.6)
      &:nth-child(3)
        box-shadow inset 0px -3px #4792BD
  .container__months
    flex-wrap wrap
    display flex
    padding: 15px
  .container__month
    padding: 8px
    flex 16.66%
    @media (max-width: 1300px)
      flex: 25%
    @media (max-width: 992px)
      flex: 33.3%
    @media (max-width: 768px)
      flex: 50%
    @media (max-width: 450px)
      flex: 100%
</style>
