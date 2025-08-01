<template>
  <div>
    <div v-if="!testStarted" class="ui segment">
      <h3>Select Difficulty</h3>
      <div class="ui buttons">
        <button
          class="ui button"
          :class="{ green: selectedDifficulty === 'easy' }"
          @click="selectDifficulty('easy')"
        >
          Easy (20s)
        </button>
        <button
          class="ui button"
          :class="{ yellow: selectedDifficulty === 'normal' }"
          @click="selectDifficulty('normal')"
        >
          Normal (10s)
        </button>
        <button
          class="ui button"
          :class="{ red: selectedDifficulty === 'hard' }"
          @click="selectDifficulty('hard')"
        >
          Hard (5s)
        </button>
      </div>

      <div style="margin-top: 20px" v-if="selectedDifficulty">
        <button class="ui primary button" @click="startTest">Start Test</button>
      </div>
    </div>

    <div v-else-if="currentIndex < words.length && !testEnded">
      <div class="ui top right attached label">
        Time Left: {{ countdown }}
      </div>

      <div class="ui segment" v-if="currentWord">
        <h3>
          {{ fromLangLabel }}:
          <strong>{{ currentWord[currentWord.fromLang] }}</strong>
        </h3>

        <div class="ui input">
          <input
            type="text"
            v-model="userAnswer"
            @keyup.enter="checkAnswer"
            :placeholder="`Enter ${toLangLabel} translation`"
          />
        </div>

        <button
          class="ui button primary"
          @click="checkAnswer"
          :disabled="!currentWord.toLang"
        >
          Submit
        </button>

        <div
          v-if="showFeedback"
          :class="['ui message', isCorrect ? 'positive' : 'negative']"
        >
          {{ isCorrect ? ' Correct!' : ' Wrong! Correct answer: ' + expectedAnswer }}
        </div>
      </div>
    </div>

    <div v-else class="ui message positive">
      <h3 v-if="testEnded">Test Completed</h3>
      <p>Your Score: {{ score }} / {{ words.length }}</p>
      <button class="ui button" @click="reset">Try Again</button>
    </div>
  </div>
</template>

<script>
import { api } from '@/helpers/helpers';

const LANGUAGES = ['english', 'german', 'french'];

function shuffleArray(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
}

export default {
  name: 'VocabTest',
  data() {
    return {
      words: [],
      currentIndex: 0,
      userAnswer: '',
      showFeedback: false,
      isCorrect: false,
      expectedAnswer: '',
      score: 0,
      testStarted: false,
      testEnded: false,
      timer: null,
      timePerWord: 10,
      timeLeft: 10,
      selectedDifficulty: null
    };
  },
  computed: {
    currentWord() {
      return this.words[this.currentIndex];
    },
    fromLangLabel() {
      return this.currentWord?.fromLang ? this.languageLabel(this.currentWord.fromLang) : '';
    },
    toLangLabel() {
      return this.currentWord?.toLang ? this.languageLabel(this.currentWord.toLang) : '';
    },
    countdown() {
      return `${this.timeLeft}s`;
    }
  },
  methods: {
    languageLabel(lang) {
      switch (lang) {
        case 'english':
          return 'English';
        case 'german':
          return 'German';
        case 'french':
          return 'French';
        default:
          return lang;
      }
    },
    selectDifficulty(level) {
      this.selectedDifficulty = level;
    },
    async startTest() {
      this.testStarted = true;

      switch (this.selectedDifficulty) {
        case 'easy': this.timePerWord = 20; break;
        case 'normal': this.timePerWord = 10; break;
        case 'hard': this.timePerWord = 5; break;
      }

      await this.loadWords();
      this.startTimer();
    },
    async loadWords() {
      try {
        const response = await api.getWords();
        this.words = shuffleArray(response).map(word => {
          const fromLang = LANGUAGES[Math.floor(Math.random() * LANGUAGES.length)];
          let toLang;
          do {
            toLang = LANGUAGES[Math.floor(Math.random() * LANGUAGES.length)];
          } while (toLang === fromLang);
          return { ...word, fromLang, toLang };
        });
      } catch (err) {
        this.flashMessage.error({ message: 'Failed to load words', time: 3000 });
      }
    },
    checkAnswer() {
      if (!this.currentWord?.toLang || !this.currentWord?.fromLang) return;

      clearInterval(this.timer);
      const input = this.userAnswer.trim().toLowerCase();
      const expected = this.currentWord[this.currentWord.toLang].toLowerCase();

      this.expectedAnswer = expected;
      this.isCorrect = input === expected;
      if (this.isCorrect) this.score++;

      this.showFeedback = true;

      setTimeout(() => {
        this.showFeedback = false;
        this.userAnswer = '';
        this.currentIndex++;
        if (this.currentIndex >= this.words.length) {
          this.testEnded = true;
        } else {
          this.startTimer();
        }
      }, 1000);
    },
    startTimer() {
      this.timeLeft = this.timePerWord;
      this.timer = setInterval(() => {
        if (this.timeLeft > 0) {
          this.timeLeft--;
        } else {
          clearInterval(this.timer);
          this.checkAnswer(); // auto-submit
        }
      }, 1000);
    },
    reset() {
      clearInterval(this.timer);
      this.words = [];
      this.currentIndex = 0;
      this.userAnswer = '';
      this.showFeedback = false;
      this.isCorrect = false;
      this.expectedAnswer = '';
      this.score = 0;
      this.testStarted = false;
      this.testEnded = false;
      this.selectedDifficulty = null;
    }
  },
  beforeDestroy() {
    clearInterval(this.timer);
  }
};
</script>
