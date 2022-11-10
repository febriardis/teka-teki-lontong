<template>
  <div style="display: flex">
    <SingleOtpInput
      v-for="(item, i) in numInputs"
      :key="i"
      :focus="activeInput === i"
      :value="otp[i]"
      :separator="separator"
      :input-type="inputType"
      :input-classes="inputClasses"
      :input-mode="inputMode"
      :is-last-child="i === numInputs - 1"
      :should-auto-focus="shouldAutoFocus"
      @on-change="handleOnChange"
      @on-keydown="handleOnKeyDown"
      @on-paste="handleOnPaste"
      @on-focus="handleOnFocus(i)"
      @on-blur="handleOnBlur"
    />
  </div>
</template>

<script>
import SingleOtpInput from './SingleOtpInput.vue'
// keyCode constants
const BACKSPACE = 8
const LEFT_ARROW = 37
const RIGHT_ARROW = 39
const DELETE = 46
export default {
  name: 'OtpInput',
  components: {
    SingleOtpInput,
  },
  props: {
    numInputs: {
      type: [String, Number],
      default: 4,
    },
    separator: {
      type: String,
      default: '**',
    },
    inputClasses: {
      type: String,
      default: 'otp-input',
    },
    inputType: {
      type: String,
      default: 'text',
      validator(value) {
        return ['text', 'number', 'tel', 'password'].includes(value)
      },
    },
    inputMode: {
      type: String,
      default: 'text',
      validator(value) {
        return ['text', 'numeric', 'tel', 'none'].includes(value)
      },
    },
    shouldAutoFocus: {
      type: Boolean,
      default: false,
    },
    value: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      activeInput: 0,
      //   otp: [],
      oldOtp: [],
    }
  },
  computed: {
    otp: {
      get() {
        const values = []
        for (let i = 0; i < this.numInputs; i++) {
          values[i] = this.value[i]
        }
        return values
      },
      set(newValue) {
        this.$emit('input', newValue)
      },
    },
  },
  watch: {
    otp(val) {
      const invalidArr = val.filter((el) => el === undefined || el === '')
      if (invalidArr.length === 0) this.$emit('on-complete', this.otp.join(''))
    },
  },
  methods: {
    handleOnFocus(index) {
      this.activeInput = index
    },
    handleOnBlur() {
      this.activeInput = -1
    },
    // Helper to return OTP from input
    checkFilledAllInputs() {
      if (this.otp.join('').length === this.numInputs) {
        return this.$emit('on-complete', this.otp.join(''))
      }
      return 'Wait until the user enters the required number of characters'
    },
    // Focus on input by index
    focusInput(input) {
      this.activeInput = Math.max(Math.min(this.numInputs - 1, input), 0)
    },
    // Focus on next input
    focusNextInput() {
      this.focusInput(this.activeInput + 1)
    },
    // Focus on previous input
    focusPrevInput() {
      this.focusInput(this.activeInput - 1)
    },
    // Change OTP value at focused input
    changeCodeAtFocus(value) {
      this.oldOtp = Object.assign([], this.otp)
      this.$set(this.otp, this.activeInput, value)
      if (this.oldOtp.join('') !== this.otp.join('')) {
        this.$emit('on-change', this.otp.join(''))
        this.$emit('input', this.otp)
        this.checkFilledAllInputs()
      }
    },
    // Handle pasted OTP
    handleOnPaste(event) {
      event.preventDefault()
      const pastedData = event.clipboardData
        .getData('text/plain')
        .slice(0, this.numInputs - this.activeInput)
        .split('')
      if (this.inputType === 'number' && !pastedData.join('').match(/^\d+$/)) {
        return 'Invalid pasted data'
      }
      // Paste data from focused input onwards
      const currentCharsInOtp = this.otp.slice(0, this.activeInput)
      const combinedWithPastedData = currentCharsInOtp.concat(pastedData)
      this.$set(this, 'otp', combinedWithPastedData.slice(0, this.numInputs))
      this.focusInput(combinedWithPastedData.slice(0, this.numInputs).length)
      return this.checkFilledAllInputs()
    },
    handleOnChange(value) {
      this.changeCodeAtFocus(value)
      this.focusNextInput()
    },
    clearInput() {
      if (this.otp.length > 0) {
        this.$emit('on-change', '')
      }
      this.otp = []
      this.activeInput = 0
    },
    // Handle cases of backspace, delete, left arrow, right arrow
    handleOnKeyDown(event) {
      switch (event.keyCode) {
        case BACKSPACE:
          event.preventDefault()
          this.changeCodeAtFocus('')
          this.focusPrevInput()
          break
        case DELETE:
          event.preventDefault()
          this.changeCodeAtFocus('')
          break
        case LEFT_ARROW:
          event.preventDefault()
          this.focusPrevInput()
          break
        case RIGHT_ARROW:
          event.preventDefault()
          this.focusNextInput()
          break
        default:
          break
      }
    },
  },
}
</script>

<style lang="scss">
.otp-input {
  text-transform: capitalize;
  width: 40px;
  height: 40px;
  padding: 5px;
  margin: 0 10px;
  font-size: 24px;
  border-radius: 4px;
  border: 1px solid rgba(0, 0, 0, 0.3);
  text-align: center;
  &.error {
    border: 1px solid red !important;
  }
}
.otp-input::-webkit-inner-spin-button,
.otp-input::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
</style>
