<template>
  <fragment class="datapicker">
    <input
      v-model="internalValue"
      v-mask="'##.##.####'"
      placeholder="ДД.MM.ГГГГ"
      class="datapicker__input"
      :class="svzDatapickerInput"
      @click="isOpen = !isOpen"
      @mouseover="hover = true"
      @mouseleave="hover = false"
    >
    <transition name="fade">
      <calendar
        v-if="isOpen"
        class="datapicker__calendar"
        :isOpen="isOpen"
        :inputDay="inputDay"
        :inputMonth="inputMonth"
        :inputYear="inputYear"
        @selected="chosenDate"
      ></calendar>
    </transition>
  </fragment>
</template>

<script>
import Calendar from './Calendar'
import { mask } from 'vue-the-mask'
import { ref, computed, watch } from "vue"

export default {
  name: 'Datapicker',

  components: {
    Calendar
  },

  directives: { mask },

  setup () {
    const isOpen = ref(false)
    const internalValue = ref('')
    const showError = ref(false)
    const closeWindow = ref(false)
    const hover = ref(false)
    const inputDay = ref('')
    const inputMonth = ref('')
    const inputYear = ref('')

    const svzDatapickerInput = computed(() => {
      return [{
        'datapicker__input--error': showError.value,
        'datapicker__input--active': isOpen.value,
      }]
    })

    const calendarStatus = computed(() => {
      return [{
        'calendar--active': !internalValue.value.length && isOpen.value,
        'calendar--hidden': internalValue.value.length,
        'calendar--nonactive': !internalValue.valuelength && !isOpen.value,
      }]
    })

    const closeStatus = computed(() => {
      return [{
        'close--active': internalValue.length,
        'close--hidden': !internalValue.length,
      }]
    })

    watch(() => {
      inputDay.value = (internalValue.value).slice(0, 2)
      inputMonth.value = (internalValue.value).slice(3, 5)
      inputYear.value = (internalValue.value).slice(6, 10)

      if (!internalValue.value.length) { showError.value = false }
      (internalValue.value.length) ? isOpen.value = true : isOpen.value = false

      if (internalValue.length > 1) {
        if (inputDay.value > 31) showError.value = true
        if (inputDay.value === '00') showError.value = true
      }

      if (internalValue.value.length > 4) {
        if (inputMonth > 12 || inputMonth < 1) {
          showError.value = true
        } else if (inputMonth === '02' && inputDay > 29) {
          showError.value = true
        } else {
          showError.value = false
        }
      }
    })

    function closeCalendar () {
      isOpen.value = false
    }

    function chosenDate (value) {
      internalValue.value = value
    }

    function clearInput () {
      internalValue.value = ''
    }

    return {
      isOpen,
      internalValue,
      showError,
      closeWindow,
      hover,
      inputDay,
      inputMonth,
      inputYear,
      svzDatapickerInput,
      calendarStatus,
      closeStatus,
      closeCalendar,
      chosenDate,
      clearInput
    }
  },
}

</script>

<style lang="scss" scoped>

$dark-blue: #4D6C9E;
$error: #FC213B ;
$light-gray: #DCDCDC;
$gray: #808080;
$white: #FFFFFF;

.datapicker {
  height: 40px;
  width: 180px;
  position: relative;
  margin: 0 auto;

  &__input {
    width: 200px;
    height: 25px;
    position: relative;
    border-radius: 10px;
    font-size: 15px;
    text-align: center;
    margin-bottom: 5px;
    outline: none;

    &::placeholder {
      font-size: 16px;
      color: $gray;
      padding: 0 5px;
    }

    &:hover {
      border-color: $gray;
    }
    
    &:focus {
      border-color: $dark-blue;
      cursor: pointer;
    }

    &:disabled {
      border-color: $light-gray;
      color: $light-gray;
      &:hover {
        border-color: $light-gray;
      }
    }

    &--error:focus {
      border: 1px $error solid;
    }

    &--active {
      border-color: $dark-blue;
    }
  }

  &__calendar {
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
    margin: 0 auto;

    &--active {
      fill: $gray;
    }
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

</style>
