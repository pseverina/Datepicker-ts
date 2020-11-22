<template>
  <div>
    <div class="datapicker__top-part">
      <img
        src="../assets/arrow.svg"
        class="arrow"
        @click="previousMonth"
      />
      <span class="datapicker__calendar-month">
        {{ monthToString }}
      </span>
      <img
        src="../assets/arrow.svg"
        class="arrow arrow-right"
        @click="nextMonth"
      />
      <img
        src="../assets/arrow.svg"
        class="arrow"
        @click="previousYear"
      />
      <span class="datapicker__calendar-year">
        {{ year }}
      </span>
      <img
        src="../assets/arrow.svg"
        class="arrow arrow-right"
        @click="nextYear"
      />
    </div>

    <thead class="weekdays">
      <tr
        v-for="name in weekDays"
        :key="name.id"
        class="weekdays__day"
      >
        {{ name }}
      </tr>
    </thead>
    <tbody
      id="datapicker__calendar-body"
      class="date"
    >
      <tr
        v-for="prevDay in prevMonthDays"
        :key="prevDay.id"
        class="date__day date__day--grey"
        @click="selectedValuePrevMonth(prevDay)"
      >
        <td> {{ prevDay }}</td>
      </tr>
      <tr
        v-for="day in days"
        :key="day.id"
        class="date__day date__day--regular"
        :class="{'date__day--selected' : Number(inputDay) === day}"
        tabindex="0"
        @click="selectedValue(day)"
      >
        <td>
          {{ day }}
        </td>
      </tr>
      <tr
        v-for="nextDay in nextMonthDays"
        :key="nextDay.id"
        class="date__day date__day--grey"
        @click="selectedValueNextMonth(nextDay)"
      >
        <td>{{ nextDay }}</td>
      </tr>
    </tbody>
  </div>
</template>

<script>
import { ref, watch, onMounted } from "vue"

export default {
  name: 'Calendar',

  props: {
    isOpen: {
      type: Boolean,
      default: false,
    },
    inputDay: {
      type: String,
      default: '',
    },
    inputMonth: {
      type: String,
      default: '',
    },
    inputYear: {
      type: String,
      default: '',
    },
  },

  setup(props) {
    const internalValue = ref('')
    const days = ref([])
    const prevMonthDays = ref([])
    const nextMonthDays = ref([])
    const day = ref(props.inputDay)
    const monthToString = ref('')
    const month = ref(props.inputMonth)
    const monthNumber = ref('')
    const year = ref(props.inputYear)
    const weekDays = [
      'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс',
    ]
    const monthName = [
      'Январь',
      'Февраль',
      'Март',
      'Апрель',
      'Май',
      'Июнь',
      'Июль',
      'Август',
      'Сентябрь',
      'Октябрь',
      'Ноябрь',
      'Декабрь',
    ]
    const showError = ref(false)
    const closeWindow = ref(false)
    const chosenDay = ref(false)
    const date = ref('')

    watch(() => {
      day.value = props.inputDay
      month.value = props.inputMonth
      year.value = props.inputYear
      genCalendar()
    })

    onMounted(() => {
      genCalendar()
    })

    function genCalendar () {
      const days = ref([])
      date.value = new Date()

      genMonth(date.value.getMonth())
      genYear(date.value.getFullYear())

      const daysInMonth = new Date(year.value, month.value, 0).getDate()
      for (let eachDay = 1; eachDay < daysInMonth + 1; eachDay++) {
        days.value.push(eachDay)
      }

      genPreviousMonth()
      genNextMonth()
    }

    function genMonth (value) {
      month.value = Number(value)

      if (month.value === 0) {
        month.value = date.getMonth() + 1
        monthToString.value = monthName[month.value- 1]
      } else {
        monthToString.value = monthName[month.value - 1]
      }
    }

    function genYear (currentYear) {
      (currentYear.length < 4) ? year.value = date.getFullYear() : year.value = currentYear
    }

    function genPreviousMonth () {
      const prevMonthDays = []

      // первый день недели месяца
      const prevMonth = month.value - 1
      const firstDay = new Date(year.value, prevMonth, 1)

      let firstDayWeekday = ref(firstDay.getDay())
      if (firstDayWeekday.value !== 0) {
        firstDayWeekday.value--
      } else {
        firstDayWeekday.value = 6
      }

      // выщитываем, сколько дней из предыдущего месяца нунжо добавить в календарь
      const daysFromPreviousMonth = new Date(year.value, prevMonth.value, 0).getDate()
      const daysInFirstRow = daysFromPreviousMonth.value - firstDayWeekday.value
      for (let daysInPrevMonth = daysFromPreviousMonth; daysInPrevMonth > daysInFirstRow; daysInPrevMonth--) {
        prevMonthDays.value.push(daysInPrevMonth)
      }
      prevMonthDays.reverse()
    }

    function genNextMonth () {
      nextMonthDays.value = []

      // последний день недели месяца
      const lastDayOfCurrentMonth = new Date(year.value, month.value, 0)
      const lastDayWeekday = lastDayOfCurrentMonth.getDay()
      const daysInLastRow = 7 - lastDayWeekday

      // считаем денёчки всё
      if (daysInLastRow !== 7) {
        for (let dayFromNextMonth = 1; dayFromNextMonth <= daysInLastRow; dayFromNextMonth++) {
          nextMonthDays.value.push(dayFromNextMonth)
        }
      }
    }

    function nextMonth () {
      month.value++
      if (month.value > 12) {
        month.value = 1
        year.value = Number(year) + 1
      }

      monthToString.value = monthName[month.value - 1]
      days.value = new Date(year.value, month.value, 0).getDate()
      
      genPreviousMonth()
      genNextMonth()
    }

    function previousMonth () {
      month.value--

      if (month.value < 1) {
        month.value = 12
        year.value = year.value - 1
      }

      monthToString.value = monthName[month.value - 1]
      days.value = new Date(year.value, month.value, 0).getDate()

      genPreviousMonth()
      genNextMonth()
    }

    function nextYear () {
      year.value = Number(year.value) + 1
      days.value = new Date(year.value, month.value, 0).getDate()

      genPreviousMonth()
      genNextMonth()
    }

    function previousYear () {
      year.value = Number(year.value) - 1
      days.value = new Date(year.value, month.value, 0).getDate()
      
      genPreviousMonth()
      genNextMonth()
    }

    function selectedValue (value) {
      (value < 10 && !(value.toString()).startsWith('0')) ? day.value = '0' + value: day.value = value

      if (month.value < 10 && month.value[0] !== '0') {
        month.value = '0' + month.value
      }

      internalValue.value = `${day.value}.${month.value}.${year.value}`
      this.$emit('selected', internalValue.value)
    }

    function selectedValuePrevMonth (value) {
      (value < 10 && !(value.toString()).startsWith('0')) ? day.value = '0' + value : day.value = value

      if (month.value === 1) {
        month.value = 12
        monthNumber.value = 12
        year.value = year.value - 1
      } else {
        month.value--
        (month.value < 10 && month.value[0] !== '0') ? monthNumber.value = '0' + month.value : monthNumber.value = month.value
      }

      days.value = new Date(year.value, month.value, 0).getDate()
      monthToString.value = monthName[month.value - 1]
      internalValue.value = `${day.value}.${monthNumber.value}.${year.value}`

      genPreviousMonth()
      genNextMonth()
      genCalendar()
      this.$emit('selected', internalValue.value)
    }

    function selectedValueNextMonth (value) {
      (value < 10 && !(value.toString()).startsWith('0')) ? day.value = '0' + value : day.value = value

      if (month.value === 12) {
        month.value = 1
        monthNumber.value = '01'
        year.value = Number(year.value) + 1
      } else {
        month.value++
        (month.value < 10 && month.value[0] !== '0') ? monthNumber.value = '0' + month.value : monthNumber.value = month.value
      }

      days.value = new Date(year.value, month.value, 0).getDate()
      monthToString.value = monthName[month.value - 1]
      internalValue.value = `${day.value}.${monthNumber.value}.${year.value}`
      
      genPreviousMonth()
      genNextMonth()
      genCalendar()
      this.$emit('selected', internalValue.value)
    }

    return {
      internalValue,
      days,
      prevMonthDays,
      nextMonthDays,
      day,
      monthToString,
      month,
      monthNumber,
      year,
      weekDays,
      monthName,
      showError,
      closeWindow,
      chosenDay,
      date,
      nextMonth,
      previousMonth,
      nextYear,
      previousYear,
      selectedValue,
      selectedValuePrevMonth,
      selectedValueNextMonth
    }
  }
}
</script>

<style lang="scss" scoped>

$dark-blue: #4D6C9E;
$gray: #808080;
$white: #FFFFFF;

.datapicker {
  height: 40px;
  width: 180px;
  position: relative;
  margin: 0;

  &__top-part {
    display: flex;
    justify-content: space-between;
  }

  &__calendar {
    margin-top: 5px;
    position: relative;
    height: 310px;
    width: 300px;
    background: $white;
    z-index: 100;
    font-size: 15px;
    border-radius: 20px;
    border: 2px solid $dark-blue;
    padding-top: 6px;
    padding-bottom: 6px;
    padding: 20px;

    &-year {
      height: 20px;
      padding: 0 20px;
      font-weight: bold;
    }

    &-month {
      height: 20px;
      padding: 0 20px;
      width: 110px;
      font-weight: bold;
      text-align: center;
    }
  }
}

.arrow {
  cursor: pointer;
  height: 15px;

  &-right {
    transform: rotate(180deg);
  }
}

.weekdays {
  color: $gray;
  display: inline-grid;
  grid-template-columns: repeat(7, 1fr);
  width: 100%;
  height: 12px;
  margin: 20px 0;

  &__day {
    display: flex;
    justify-content: center;
    align-items: center;
  }
}

.date {
  display: inline-grid;
  grid-template-columns: repeat(7, 1fr);
  width: 100%;
  height: 75%;

  &__day {
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 28px;
    width: 28px;
    margin: auto;

    &--regular:hover {
      background-color:  rgb(241, 237, 245);
    }

    &--grey {
      color:gainsboro;
    }

    &--grey:hover {
      color: $gray;
    }

    &--grey:active {
      color: black;
    }

    &--selected {
      color: $white;
      background-color: $dark-blue;
      border-radius: 50%;
      outline: none;
    }

    &:hover {
      border-radius: 50%;
    }

    &:focus {
      color: $white;
      background-color: $dark-blue;
      border-radius: 50%;
      outline: none;
    }
  }
}

</style>
