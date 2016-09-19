<template>
  <div class='cov-vue-date'>
    <div class='datepickbox'>
      <div class='cov-date-body'>
        <div class='cov-date-monthly'>
          <div class='cov-date-previous' @click='nextMonth("pre")'>
            <icon-prev></icon-prev>
          </div>
          <div class='cov-date-caption'>
            <span @click='showMonth'>{{displayInfo.month}}</span>
          </div>
          <div class='cov-date-next' @click='nextMonth("next")'>
              <icon-next></icon-next>
          </div>
        </div>
        <div class='cov-date-box' v-if='showInfo.day'>
          <div class='cov-picker-box'>
            <div class='week'>
              <ul>
                <li v-for='weekie in library.week'>{{weekie}}</li>
              </ul>
            </div>
            <div class='day' v-for='day in dayList' track-by='$index' @click='checkDay(day)'
                :class="{ 'checked':day.checked, 'unavailable':day.unavailable || day.action == 'previous', 'passive-day': !(day.inMonth) }"
              >
              {{day.value}}
            </div>
          </div>
        </div>
        <div class='cov-date-box list-box' v-if='showInfo.month'>
          <div class='cov-picker-box date-list'>
            <div class='date-item' v-for='monthItem in library.month' track-by='$index' @click='setMonth(monthItem)'>{{monthItem}}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
Object.defineProperty(exports, '__esModule', {
  value: true
})
import _ from 'lodash'
var _moment = require('moment')
var _moment2 = _interopRequireDefault(_moment)

function _interopRequireDefault (obj) { return obj && obj.__esModule ? obj : { default: obj } }
import iconNext from './icons/icon_next'
import iconPrev from './icons/icon_prev'

exports.default = {
  props: {
    time: {
      required: false
    },
    option: {
      type: Object,
      default: function _default () {
        return {
          type: 'day',
          SundayFirst: true,
          week: ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa'],
          month: ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
          format: 'YYYY-MM-DD'
        }
      }
    }
  },
  components: {
    iconNext,
    iconPrev
  },
  created () {
    let date = _moment()
    this.getAvailableDates(date)
  },
  data: function data () {
    function hours () {
      var list = []
      var hour = 24
      while (hour > 0) {
        hour--

        list.push({
          checked: false,
          value: hour < 10 ? '0' + hour : hour
        })
      }
      return list
    }
    function mins () {
      var list = []
      var min = 60
      while (min > 0) {
        min--
        list.push({
          checked: false,
          value: min < 10 ? '0' + min : min
        })
      }
      return list
    }
    return {
      limit: {
        type: 'fromto',
        from: '2016-09-15',
        to: '2016-01-30'
      },
      hours: hours(),
      mins: mins(),
      showInfo: {
        hour: false,
        day: false,
        month: false,
        year: false,
        check: false
      },
      displayInfo: {
        month: ''
      },
      library: {
        week: this.option.week,
        month: this.option.month,
        year: []
      },
      checked: {
        oldtime: '',
        currentMoment: null,
        year: '',
        month: '',
        day: '',
        hour: '00',
        min: '00'
      },
      dayList: [],
      selectedDays: [],
      availableArray: []
    }
  },
  methods: {
    getAvailableDates (date) {
      this.$dispatch('getData', date)
    },
    pad: function pad (n) {
      n = Math.floor(n)
      return n < 10 ? '0' + n : n
    },
    nextMonth: function nextMonth (type) {
      var next = null
      type === 'next' ? next = (0, _moment2.default)(this.checked.currentMoment).add(1, 'M') : next = (0, _moment2.default)(this.checked.currentMoment).add(-1, 'M')
      this.getAvailableDates(next)
    },
    showDay: function showDay (time) {
      if (time === undefined || !Date.parse(time)) {
        this.checked.currentMoment = (0, _moment2.default)()
      } else {
        this.checked.currentMoment = (0, _moment2.default)(time, this.option.format)
      }
      this.showOne('day')

      this.checked.year = (0, _moment2.default)(this.checked.currentMoment).format('YYYY')
      this.checked.month = (0, _moment2.default)(this.checked.currentMoment).format('MM')
      this.checked.day = (0, _moment2.default)(this.checked.currentMoment).format('DD')

      this.displayInfo.month = this.library.month[(0, _moment2.default)(this.checked.currentMoment).month()]

      var days = []
      var currentMoment = this.checked.currentMoment
      var firstDay = (0, _moment2.default)(currentMoment).date(1).day()

      // gettting previous and next month
      // let currentMonth = moment(currentMoment)
      var previousMonth = (0, _moment2.default)(currentMoment)
      var nextMonth = (0, _moment2.default)(currentMoment)
      nextMonth.add(1, 'months')
      previousMonth.subtract(1, 'months')

      var monthDays = (0, _moment2.default)(currentMoment).daysInMonth()
      var oldtime = this.checked.oldtime
      for (var i = 1; i <= monthDays; ++i) {
        let isAvailable = false
        let isAvailableItem = _.filter(this.availableArray, (d) => {
          return d.day === i
        })
        if (isAvailableItem.length > 0) {
          isAvailable = isAvailableItem[0].isAvailable
        }

        let item = {
          value: i,
          inMonth: true,
          unavailable: !isAvailable,
          checked: false,
          moment: (0, _moment2.default)(currentMoment).date(i)
        }
        days.push(item)
        if (i === Math.ceil((0, _moment2.default)(currentMoment).format('D')) && (0, _moment2.default)(oldtime, this.option.format).year() === (0, _moment2.default)(currentMoment).year() && (0, _moment2.default)(oldtime, this.option.format).month() === (0, _moment2.default)(currentMoment).month()) {
          days[i - 1].checked = true
        }
        this.checkBySelectDays(i, days)
      }

      if (firstDay === 0) firstDay = 7

      for (var _i = 0; _i < firstDay - (this.option.SundayFirst ? 0 : 1); _i++) {
        var passiveDay = {
          value: previousMonth.daysInMonth() - _i,
          inMonth: false,
          action: 'previous',
          unavailable: false,
          checked: false,
          moment: (0, _moment2.default)(currentMoment).date(1).subtract(_i + 1, 'days')
        }
        days.unshift(passiveDay)
      }

      var passiveDaysAtFinal = 42 - days.length
      for (var _i2 = 1; _i2 <= passiveDaysAtFinal; _i2++) {
        var _passiveDay = {
          value: _i2,
          inMonth: false,
          action: 'next',
          unavailable: false,
          checked: false,
          moment: (0, _moment2.default)(currentMoment).add(1, 'months').date(_i2)
        }
        days.push(_passiveDay)
      }

      if (this.limit.length > 0) {
        var _iteratorNormalCompletion = true
        var _didIteratorError = false
        var _iteratorError = ''

        try {
          for (var _iterator = this.limit[Symbol.iterator](), _step; !(_iteratorNormalCompletion = (_step = _iterator.next()).done); _iteratorNormalCompletion = true) {
            var li = _step.value

            switch (li.type) {
              case 'fromto':
                days = this.limitFromTo(li, days)
                break
              case 'weekday':
                days = this.limitWeekDay(li, days)
                break
            }
          }
        } catch (err) {
          _didIteratorError = true
          _iteratorError = err
        } finally {
          try {
            if (!_iteratorNormalCompletion && _iterator.return) {
              _iterator.return()
            }
          } finally {
            if (_didIteratorError) {
              console.error(_iteratorError)
            }
          }
        }
      }

      this.dayList = days
    },
    checkBySelectDays: function checkBySelectDays (d, days) {
      var _this = this

      this.selectedDays.forEach(function (day) {
        if (_this.checked.year === (0, _moment2.default)(day).format('YYYY') && _this.checked.month === (0, _moment2.default)(day).format('MM') && d === Math.ceil((0, _moment2.default)(day).format('D'))) {
          days[d - 1].checked = true
        }
      })
    },
    limitWeekDay: function limitWeekDay (limit, days) {
      return days.map(function (day) {
        if (limit.available.indexOf(Math.floor(day.moment.format('d'))) === -1) {
          day.unavailable = true
        }
        return day
      })
    },
    limitFromTo: function limitFromTo (limit, days) {
      var _this2 = this

      if (limit.from || limit.to) {
        days.map(function (day) {
          if (_this2.getLimitCondition(limit, day)) {
            day.unavailable = true
          }
        })
      }
      return days
    },
    getLimitCondition: function getLimitCondition (limit, day) {
      var tmpMoment = (0, _moment2.default)(this.checked.year + '-' + this.pad(this.checked.month) + '-' + this.pad(day.value))
      if (limit.from && !limit.to) {
        return !tmpMoment.isAfter(limit.from)
      } else if (!limit.from && limit.to) {
        return !tmpMoment.isBefore(limit.to)
      } else {
        return !tmpMoment.isBetween(limit.from, limit.to)
      }
    },
    checkDay: function checkDay (obj) {
      if (obj.unavailable || obj.value === '') {
        return false
      }

      if (!obj.inMonth) {
        this.nextMonth(obj.action)
        return
      }

      if (this.option.type === 'day' || this.option.type === 'min') {
        this.dayList.map(function (x) {
          x.checked = false
        })
        this.checked.day = this.pad(obj.value)
        obj.checked = true
      } else {
        var day = this.pad(obj.value)
        var ctime = this.checked.year + '-' + this.checked.month + '-' + day
        if (obj.checked === true) {
          obj.checked = false
          this.selectedDays.$remove(ctime)
        } else {
          this.selectedDays.push(ctime)
          obj.checked = true
        }
      }

      switch (this.option.type) {
        case 'day':
          this.picked()
          break
        case 'min':
          this.showOne('hour')
          break
      }
    },
    showYear: function showYear () {
      var year = (0, _moment2.default)(this.checked.currentMoment).year()
      this.library.year = []
      var yearTmp = []
      for (var i = year - 100; i < year + 5; ++i) {
        yearTmp.push(i)
      }
      this.library.year = yearTmp

      this.showOne('year')

      var self = this
      this.$nextTick(function () {
        var listDom = document.getElementById('yearList')
        listDom.scrollTop = listDom.scrollHeight - 100
        self.addYear()
      })
    },
    showOne: function showOne (type) {
      switch (type) {
        case 'year':
          this.showInfo.hour = false
          this.showInfo.day = false
          this.showInfo.year = true
          this.showInfo.month = false
          break
        case 'month':
          this.showInfo.hour = false
          this.showInfo.day = false
          this.showInfo.year = false
          this.showInfo.month = true
          break
        case 'day':
          this.showInfo.hour = false
          this.showInfo.day = true
          this.showInfo.year = false
          this.showInfo.month = false
          break
        case 'hour':
          this.showInfo.hour = true
          this.showInfo.day = false
          this.showInfo.year = false
          this.showInfo.month = false
          break
        default:
          this.showInfo.day = true
          this.showInfo.year = false
          this.showInfo.month = false
          this.showInfo.hour = false
      }
    },
    showMonth: function showMonth () {
      this.showOne('month')
    },
    addYear: function addYear () {
      var _this3 = this

      var listDom = document.getElementById('yearList')
      // let tmp = 0
      listDom.addEventListener('scroll', function (e) {
        if (listDom.scrollTop < listDom.scrollHeight - 100) {
          var len = _this3.library.year.length
          var lastYear = _this3.library.year[len - 1]
          _this3.library.year.push(lastYear + 1)
        }
      }, false)
    },
    setYear: function setYear (year) {
      this.checked.currentMoment = (0, _moment2.default)(year + '-' + this.checked.month + '-' + this.checked.day)
      this.showDay(this.checked.currentMoment)
    },
    setMonth: function setMonth (month) {
      var mo = this.library.month.indexOf(month) + 1
      if (mo < 10) {
        mo = '0' + '' + mo
      }
      this.checked.currentMoment = (0, _moment2.default)(this.checked.year + '-' + mo + '-' + this.checked.day)
      this.getAvailableDates(this.checked.currentMoment)
    },
    showCheck: function showCheck () {
      if (this.time === '') {
        this.showDay()
      } else {
        if (this.option.type === 'day' || this.option.type === 'min') {
          this.checked.oldtime = this.time
          this.showDay(this.time)
        } else {
          this.selectedDays = JSON.parse(this.time)
          if (this.selectedDays.length) {
            this.checked.oldtime = this.selectedDays[0]
            this.showDay(this.selectedDays[0])
          } else {
            this.showDay()
          }
        }
      }
      this.showInfo.check = true
    },
    setTime: function setTime (type, obj, list) {
      var _iteratorNormalCompletion2 = true
      var _didIteratorError2 = false
      var _iteratorError2 = ''

      try {
        for (var _iterator2 = list[Symbol.iterator](), _step2; !(_iteratorNormalCompletion2 = (_step2 = _iterator2.next()).done); _iteratorNormalCompletion2 = true) {
          var item = _step2.value

          item.checked = false
          if (item.value === obj.value) {
            item.checked = true
            this.checked[type] = item.value
          }
        }
      } catch (err) {
        _didIteratorError2 = true
        _iteratorError2 = err
      } finally {
        try {
          if (!_iteratorNormalCompletion2 && _iterator2.return) {
            _iterator2.return()
          }
        } finally {
          if (_didIteratorError2) {
            console.error(_iteratorError2)
          }
        }
      }
    },
    picked: function picked () {
      if (this.option.type === 'day' || this.option.type === 'min') {
        var ctime = this.checked.year + '-' + this.checked.month + '-' + this.checked.day + ' ' + this.checked.hour + ':' + this.checked.min
        this.checked.currentMoment = (0, _moment2.default)(ctime, 'YYYY-MM-DD HH:mm')
        this.time = this.checked.currentMoment
      } else {
        this.time = JSON.stringify(this.selectedDays)
      }
      this.showInfo.check = false
      this.$emit('change', this.time)
    }
  },
  events: {
    getgetAvailableArray (object) {
      this.availableArray = object.response
      this.showDay(object.date)
    }
  }
}
</script>
