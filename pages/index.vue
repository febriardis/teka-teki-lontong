<template>
  <div class="index">
    <div
      v-if="isFirstOpened"
      style="height: calc(100vh - 100px)"
      class="welcome"
    >
      <div class="box">
        <p class="mb-4">
          Selamat datang di game Teka Teki Sulit. Jika anda merasa pusing, mual,
          muntah, darah tinggi kami tidak bertanggung jawab.
        </p>
        <button
          type="button"
          class="btn btn-next"
          @click="isFirstOpened = false"
        >
          MULAI
        </button>
      </div>
    </div>
    <div v-else-if="currentQuestion" class="index">
      <div
        class="bg-lontong min-h-[200px] sm:min-h-[250px] md:min-h-[300px] relative"
      >
        <div class="question-box">
          <p class="uppercase">{{ currentQuestion.question }}</p>
        </div>
        <button type="button" class="btn btn-help" @click="handleHelp">
          BANTUAN
        </button>
      </div>
      <div class="bg-black p-5">
        <div class="flex flex-row justify-center items-center">
          <v-otp-input
            ref="answerInput"
            v-model="currentValues"
            :num-inputs="currentQuestion.answer.length"
            :should-auto-focus="true"
            @on-change="handleOnChange"
            @on-complete="handleOnComplete"
          />
        </div>
      </div>
      <div class="bg-[#1F5F9C] h-[400px] relative">
        <div v-if="isWrong" class="reason-box">
          <p class="uppercase text-center">kurang tepat, coba lagi!!</p>
        </div>
        <div v-else-if="isCompleted" class="reason-box">
          <p class="uppercase">{{ currentQuestion.reason }}</p>
        </div>
        <div class="footer">
          <button type="button" class="btn btn-reset" @click="reset()">
            <img src="/img/reset.svg" alt="" />
          </button>
          <button
            v-if="arrQuestions.length - 1 !== index"
            type="button"
            class="btn btn-next"
            @click="nextQuestion()"
          >
            {{ isCompleted ? 'LANJUT' : 'LEWATI' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { QUESTIONS } from '../static/questions'
import VOtpInput from '@/components/v-otp-input/index.vue'
export default {
  components: {
    'v-otp-input': VOtpInput,
  },
  asyncData({ query }) {
    const key = parseInt(query?.index) || 0
    return {
      index: key,
      isFirstOpened: !key,
      arrQuestions: QUESTIONS,
      currentQuestion: QUESTIONS[key],
    }
  },
  data: () => ({
    isFirstOpened: true,
    isNext: false,
    isWrong: false,
    isCompleted: false,
    currentValues: [],
    currentQuestion: null,
  }),
  computed: {
    query: (vm) => vm.$route.query.index,
  },
  watch: {
    query(value) {
      if (!value) this.index = 1
      if (!this.isNext) this.index = this.index - 1
      this.isWrong = false
      this.isCompleted = false
      this.currentQuestion = QUESTIONS[value || 0]
      this.setFirstValues()
      this.isNext = false
    },
  },
  mounted() {
    this.setFirstValues()
  },
  methods: {
    reset() {
      this.isWrong = false
      this.isCompleted = false
      this.setFirstValues()
    },
    nextQuestion() {
      this.isNext = true
      this.index = this.index + 1
      this.$router.push({
        query: {
          index: this.index,
        },
      })
    },
    setFirstValues() {
      this.currentValues = []
      if (this.currentQuestion) {
        const key = this.currentQuestion.keyIndex
        const answerSplit = this.currentQuestion.answer.split('')
        const values = []
        for (let i = 0; i < answerSplit.length; i++) {
          if (i === key) values[i] = answerSplit[key]
          else values[i] = undefined
        }
        this.currentValues = values
      }
    },
    stringShuffle(value) {
      const a = value.split('')
      const n = a.length
      for (let i = n - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1))
        const tmp = a[i]
        a[i] = a[j]
        a[j] = tmp
      }
      return a.join('')
    },
    handleHelp() {
      const keyTemp = []
      for (let i = 0; i < this.currentValues.length; i++) {
        if (!this.currentValues[i]) keyTemp.push(i)
      }
      const key = parseInt(this.stringShuffle(keyTemp.join('')).charAt(0))
      const answerSplit = this.currentQuestion.answer.split('')
      const values = [...this.currentValues]
      for (let i = 0; i < answerSplit.length; i++) {
        if (i === key) values[i] = answerSplit[key]
      }
      this.currentValues = values
    },
    handleOnComplete(value) {
      if (value.toLowerCase() === this.currentQuestion.answer) {
        this.isWrong = false
        this.isCompleted = true
        this.handleSound('benar')
      } else {
        this.isWrong = true
        this.isCompleted = false
        this.handleSound('salah')
      }
    },

    handleSound(type) {
      const audio = new Audio(`/sound/${type}.mp3`)
      // audio.play()
      // // audio.muted = false;
      const promise = audio.play()
      promise
        .then(() => {
          // Autoplay started!
          console.log('sound loaded')
        })
        .catch((error) => {
          console.log('sound error', error)
          audio.muted = false
          // Autoplay was prevented.
          // Show a "Play" button so that user can start playback.
        })
    },
    handleOnChange() {
      this.isWrong = false
      this.isCompleted = false
    },
  },
}
</script>

<style lang="scss">
.welcome {
  @apply flex justify-center items-center bg-[#fafafa];
  height: calc(100vh - 50px);
  .box {
    @apply border w-4/5 lg:w-3/5 text-center p-10 rounded-xl;
    font-family: cursive;
  }
}
.footer {
  @apply flex justify-between py-3 px-4 bg-[#213760] fixed w-full md:w-[640px] bottom-0;
}
.btn {
  @apply shadow-md px-6  text-white font-bold;
  &-help {
    @apply py-2 absolute right-0 bottom-0;
    background: rgb(36 79 255 / 80%);
  }
  &-next {
    @apply py-1 bg-[#f59e0b];
  }
  &-reset {
    @apply py-1 px-3 bg-[#707070];
  }
}
.question-box {
  @apply absolute w-3/4 mt-10 left-0 right-0 mx-auto rounded-md p-5 text-lg;
  background: rgba(255, 255, 255, 0.8);
}
.reason-box {
  @apply w-3/4 mx-auto mt-10 absolute left-0 right-0 rounded-md p-5 text-lg;
  background: rgba(255, 255, 255, 0.8);
}
.bg-lontong {
  background-image: url('/img/bg-lontong.jpg');
  background-repeat: no-repeat;
  background-size: 100% 100%;
}
</style>
