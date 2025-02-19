/**
* @Description:    日期、时间选择器
* @Author:         TSY
* @CreateDate:     2019/5/17 16:22
* @Email:          t@tsy6.com
*/
<template>
  <div class="hash-calendar"
       :class="{'calendar_inline': model === 'inline'}"
       v-show="isShowDatetimePicker"
       :style="{'height': `${model === 'inline' ? calendarContentHeight + calArrowHeight() : undefined}px`}"
       @click="close">
    <div class="calendar_content"
         :style="{'height': `${calendarContentHeight}px`, 'bottom': `${calArrowHeight()}px`}"
         @click.stop>
      <div class="calendar_title"
           v-if="isShowAction"
           ref="calendarTitle">
        <slot name="action">
          <div class="calendar_title_date">
            <span v-if="pickerType !== 'time'"
                  class="calendar_title_date_year"
                  :class="{'calendar_title_date_active': isShowCalendar}"
                  @click="showCalendar">{{ formatDate(`${checkedDate.year}/${checkedDate.month + 1}/${checkedDate.day}`, language.DEFAULT_DATE_FORMAT) }}</span>
            <span v-if="pickerType !== 'date'"
                  class="calendar_title_date_time"
                  :class="{'calendar_title_date_active': !isShowCalendar}"
                  @click="showTime">{{ formatDate(`${checkedDate.year}/${checkedDate.month + 1}/${checkedDate.day} ${fillNumber(checkedDate.hours)}:${fillNumber(checkedDate.minutes)}`, language.DEFAULT_TIME_FORMAT)}}</span>
          </div>
          <div v-if="showTodayButton"
               class="calendar_confirm"
               :class="{'today_disable': disabledDate(new Date())}"
               @click="today">
            <slot name="today">
              {{ language.TODAY }}
            </slot>
          </div>
          <div class="calendar_confirm"
               v-if="model === 'dialog'"
               @click="confirm">
            <slot name="confirm">
              {{ language.CONFIRM }}
            </slot>
          </div>
        </slot>
      </div>
      <calendar ref="calendar"
                v-if="pickerType !== 'time'"
                :show="isShowCalendar"
                :isShowWeekView.sync="isShowWeek"
                v-bind="{...$props, ...$attrs}"
                :calendarTitleHeight="calendarTitleHeight"
                @height="heightChange"
                :default-date="currDateTime"
                @touchstart="touchStart"
                @touchmove="touchMove"
                @touchend="touchEnd"
                @slidechange="slideChange"
                @change="dateChange"
                @click="dateClick">
        <template v-if="hasSlot('week')"
                  slot="week"
                  slot-scope="scope">
          <slot name="week"
                :week="scope.week">
          </slot>
        </template>
        <template v-if="hasSlot('day')"
                  slot="day"
                  slot-scope="scope">
          <slot name="day"
                :date="scope.date"
                :extendAttr="scope.extendAttr">
          </slot>
        </template>
      </calendar>

      <time-picker v-if="pickerType !== 'date'"
                   :show="!isShowCalendar"
                   :default-time="currDateTime"
                   :calendarDate="checkedDate"
                   v-bind="{...$props, ...$attrs}"
                   @change="timeChange"></time-picker>

      <year-month-picker v-if="changeYearFast"
                         :calendarTitleHeight="calendarTitleHeight"
                         :calendarContentHeight="calendarContentHeight"
                         :calendarDate="checkedDate"
                         @touchstart="touchStart"
                         @touchmove="touchMove"
                         @touchend="touchEnd"
                         @slidechange="slideChange"
                         v-bind="{...$props, ...$attrs}"
                         @click="dateClick"
                         :type="yearMonthType"></year-month-picker>

    </div>
    <div class="ctrl-img"
         ref="arrow"
         v-if="isShowArrowImg"
         @click.stop="toggleWeek"
         :style="{'margin-top': `${calendarContentHeight}px`}">
      <slot name="arrow"
            :show="isShowWeek">
        <img class="ctrl-img-handler" :src="isShowWeek ? arrowDownImg : arrowUpImg">
      </slot>
    </div>
  </div>
</template>

<script>
import Calendar from './Calendar.vue'
import TimePicker from './TimePicker.vue'
import YearMonthPicker from './YearMonthPicker.vue'
import { formatDate } from '../utils/util'
import { ARROW_DOWN_IMG, ARROW_UP_IMG } from '../constant/img'
import languageUtil from '../language'

const defaultDate = {
  year: new Date().getFullYear(),
  month: new Date().getMonth(),
  day: new Date().getDate(),
  hours: new Date().getHours(),
  minutes: new Date().getMinutes()
}

export default {
  props: {
    // 主题颜色
    themeColor: {
      type: Object,
      default: () => {}
    },
    // 是否支持点击日期区域快速切换年份
    changeYearFast: {
      type: Boolean,
      default: false
    },
    // 是否显示 周月视图切换指示箭头，model 等于 inline 时生效
    isShowArrow: {
      type: Boolean,
      default: false
    },
    // 是否展示周视图
    isShowWeekView: {
      type: Boolean,
      default: false
    },
    // 是否显示日历组件
    visible: {
      type: Boolean,
      default: false
    },
    // 是否显示日历组件操作栏
    isShowAction: {
      type: Boolean,
      default: true
    },
    pickerType: {// 选择器类型 datetime：日期+时间   date：日期   time：时间
      type: String,
      default: 'datetime'
    },
    showTodayButton: {// 是否显示返回今日按钮
      type: Boolean,
      default: true
    },
    defaultDatetime: {// 默认时间
      type: Date,
      default() {
        return new Date()
      }
    },
    format: null, // 确认选择之后，返回的日期格式
    model: {
      type: String,
      default: 'inline'
    },
    // 日期下面的标记
    markDate: {
      type: Array,
      default: () => []
    },
    // 禁用的日期
    disabledDate: {
      type: Function,
      default: () => {
        return false
      }
    },
    // 使用的语言包
    lang: {
      type: String,
      default: 'CN'
    }
  },
  components: {
    YearMonthPicker,
    TimePicker,
    Calendar
  },
  name: 'VueHashCalendar',
  data() {
    return {
      arrowDownImg: ARROW_DOWN_IMG,
      arrowUpImg: ARROW_UP_IMG,
      language: {}, // 使用的语言包
      checkedDate: defaultDate, // 被选中的日期
      isShowWeek: false,
      isShowCalendar: false, // 是否显示日历选择控件
      calendarBodyHeight: 0, // 日历内容的高度
      calendarTitleHeight: 0, // 日历组件标题高度
      firstTimes: true, // 第一次触发
      currDateTime: new Date(), // 当前日期
      yearMonthType: 'date' // 年月选择面板默认展示类型
    }
  },
  mounted() {
    if (this.model === 'inline') {
      this.isShowDatetimePicker = true
    }

    this.language = languageUtil[this.lang.toUpperCase()]
  },
  watch: {
    themeColor: {
      handler(val) {
        val && this.changeThemeColor()
      },
      immediate: true
    },
    defaultDatetime: {
      handler(val) {
        if (!(val instanceof Date)) {
          throw new Error('The calendar component\'s defaultDate must be date type!')
        }

        this.currDateTime = val
      },
      immediate: true
    },
    pickerType: {
      handler(val) {
        if (val === 'time') {
          this.showTime()
        }
      },
      immediate: true
    },
    isShowAction(flag) {
      if (!flag) {
        this.calendarTitleHeight = 0
      } else {
        setTimeout(() => {
          this.calendarTitleHeight = this.$refs.calendarTitle ? this.$refs.calendarTitle.offsetHeight : 0
        })
      }
    },
    checkedDate: {
      handler() {
        let date = new Date(`${this.checkedDate.year}/${this.checkedDate.month + 1}/${this.checkedDate.day} ${this.checkedDate.hours}:${this.checkedDate.minutes}`)
        if (this.format) {
          date = formatDate(date, this.format, this.lang)
        }
        this.$emit('change', date)
      },
      deep: true
    },
    visible: {
      handler(val) {
        this.isShowCalendar = val

        setTimeout(() => {
          this.calendarTitleHeight = this.$refs.calendarTitle ? this.$refs.calendarTitle.offsetHeight : 0
        })
      },
      immediate: true
    },
    isShowWeekView: {
      handler(val) {
        this.isShowWeek = val
      },
      immediate: true
    }
  },
  computed: {
    isShowArrowImg() {
      return this.isShowArrow && this.model === 'inline'
    },
    // 是否显示周视图 (为兼容旧版本，舍弃这种方式)
    // isShowWeek: {
    //   get() {
    //     return this.isShowWeekView
    //   },
    //   set(val) {
    //     this.$emit('update:isShowWeekView', val)
    //   }
    // },
    // 是否显示日期控件
    isShowDatetimePicker: {
      get() {
        return this.visible
      },
      set(val) {
        this.$emit('update:visible', val)
      }
    },
    // 日历组件的高度
    calendarContentHeight() {
      return this.calendarBodyHeight + this.calendarTitleHeight
    }
  },
  methods: {
    // 判断是否有插槽
    hasSlot(slotName) {
      return !!this.$scopedSlots[slotName]
    },
    // 周视图开关
    toggleWeek() {
      this.isShowWeek = !this.isShowWeek

      if (this.isShowWeek) this.slideChange('up')
      else this.slideChange('down')
    },
    today() {
      if (this.disabledDate(new Date())) return

      this.$refs.calendar.today()
    },
    lastMonth() {
      this.$refs.calendar.getLastMonth()
    },
    nextMonth() {
      this.$refs.calendar.getNextMonth()
    },
    lastWeek() {
      this.$refs.calendar.getLastMonth()
      this.$refs.calendar.changeWeekView({isNext: false})
    },
    nextWeek() {
      this.$refs.calendar.getNextMonth()
      this.$refs.calendar.changeWeekView({isNext: true})
    },
    dateChange(date) {
      date.hours = this.checkedDate.hours
      date.minutes = this.checkedDate.minutes
      this.checkedDate = date
    },
    dateClick(date) {
      date.hours = this.checkedDate.hours
      date.minutes = this.checkedDate.minutes
      this.checkedDate = date

      let fDate = new Date(`${this.checkedDate.year}/${this.checkedDate.month + 1}/${this.checkedDate.day} ${this.checkedDate.hours}:${this.checkedDate.minutes}`)
      if (this.format) {
        fDate = formatDate(fDate, this.format, this.lang)
      }

      // 控制点击之后进入下一选择面板
      if (date.type) {
        switch (date.type) {
          case 'yearRange':
            this.yearMonthType = 'year'
            break
          case 'year':
            this.yearMonthType = 'month'
            break
          case 'month':
            this.currDateTime = new Date(fDate)
            this.yearMonthType = 'date'
            break
        }

        this.$emit('calendarTypeChange', this.yearMonthType)
      }

      this.$emit('click', fDate)
    },
    timeChange(date) {
      date.year = this.checkedDate.year
      date.month = this.checkedDate.month
      date.day = this.checkedDate.day
      this.checkedDate = date
    },
    // 确认选择时间
    confirm() {
      let date = new Date(`${this.checkedDate.year}/${this.checkedDate.month + 1}/${this.checkedDate.day} ${this.checkedDate.hours}:${this.checkedDate.minutes}`)
      if (this.format) {
        date = formatDate(date, this.format, this.lang)
      }
      this.$emit('confirm', date)
      if (this.model === 'dialog') {
        this.close()
      }
    },
    show() {
      this.isShowDatetimePicker = true
    },
    close() {
      this.isShowDatetimePicker = false
    },
    // 小于10，在前面补0
    fillNumber(val) {
      return val > 9 ? val : '0' + val
    },
    formatDate(time, format) {
      return formatDate(time, format, this.lang)
    },
    // 显示日历控件
    showCalendar() {
      if (this.isShowCalendar) {
        this.showYearMonthPicker()
      }
      this.isShowCalendar = true
    },
    // 显示时间选择控件
    showTime() {
      this.isShowCalendar = false

      // 重置年月选择面板
      this.yearMonthType = 'date'
    },
    // 显示年月选择面板
    showYearMonthPicker() {
      if (!this.changeYearFast) return

      if (this.yearMonthType === 'date') {
        this.yearMonthType = 'month'
      } else if (this.yearMonthType === 'month') {
        this.yearMonthType = 'year'
      } else if (this.yearMonthType === 'year') {
        this.yearMonthType = 'yearRange'
      } else {
        this.yearMonthType = 'date'
      }

      this.$emit('calendarTypeChange', this.yearMonthType)
    },
    // 高度变化
    heightChange(height) {
      if (!this.firstTimes && this.model === 'dialog') return

      this.calendarBodyHeight = height
      this.firstTimes = false
    },
    // 根据传入的 arrow slot 计算高度
    calArrowHeight() {
      const height = (this.isShowArrowImg && this.$refs.arrow && this.$refs.arrow.offsetHeight) || 0
      return height
    },
    // 切换主题颜色
    changeThemeColor() {
      const themeColorKeys = Object.keys(this.themeColor || {})

      if (themeColorKeys.length) {
        let cssText = ''

        themeColorKeys.forEach(k => {
          cssText += `--hash-calendar-${k}: ${this.themeColor[k]};`
        })

        this.$nextTick(() => {
          document.querySelector('.hash-calendar').style.cssText = cssText
        })
      }
    },
    // 监听手指开始滑动事件
    touchStart(event) {
      this.$emit('touchstart', event)
    },
    // 监听手指开始滑动事件
    touchMove(event) {
      this.$emit('touchmove', event)
    },
    // 监听手指开始滑动事件
    touchEnd(event) {
      this.$emit('touchend', event)
    },
    // 滑动方向改变
    slideChange(direction) {
      this.$emit('slidechange', direction)
    }
  }
}
</script>

<style lang="stylus" scoped>
@import '../style/common.styl';

.hash-calendar {
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 999;
}
.calendar_inline {
  position: relative;
  width: 100%;
  height: auto;
  background: none;
  height: px2vw(710px);
  z-index: 1;
}
.calendar_content {
  position: absolute;
  width: 100%;
  left: 0;
  bottom: 0;
  display: flex;
  padding-bottom: px2vw(26px);
  flex-wrap: wrap;
  background: white;
  height: px2vw(710px);
  overflow: hidden;
}
.calendar_title {
  position: absolute;
  width: 100%;
  left: 0;
  top: 0;
  bgColor(background);
  borderBottom();
  display: flex;
  align-items: center;
  justify-content: space-between;
  z-index: 1;
}
.calendar_title_date {
  viceFontColor(color);
  background: white;
  padding: px2vw(30px) px2vw(15px);
}
.calendar_title_date_active {
  mainFontColor(color);
  font-weight: bold;
}
.calendar_title_date_time {
  margin-left: px2vw(20px);
}
.calendar_confirm {
  mainColor(color);
  margin-right: px2vw(34px);
}
.today_disable {
  disabledFontColor(color);
}
.ctrl-img {
  width: 100%;
  text-align: center;
  .ctrl-img-handler {
    width: 28px;
  }
}
</style>
