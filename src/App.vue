<template>
  <main>
    <h1>Vue クイズ</h1>

    <section v-if="stage === 'category'" class="card">
      <h2>カテゴリを選んでください</h2>
      <p class="hint">カテゴリごとに用意された問題を順番に解いていきます。</p>
      <ul class="category-list">
        <li v-for="category in categories" :key="category.id">
          <button type="button" class="category-button" @click="startQuiz(category.id)">
            <strong>{{ category.name }}</strong>
            <small>{{ category.questions.length }} 問・難易度 {{ category.level }}</small>
          </button>
        </li>
      </ul>

      <section v-if="scoreHistory.length" class="history">
        <h3>スコア履歴</h3>
        <ul>
          <li v-for="item in scoreHistory" :key="item.id">
            <div>
              <strong>{{ item.categoryName }}</strong>
              <span class="history-date">{{ item.playedAt }}</span>
            </div>
            <div class="history-score">
              <span>{{ item.correct }} / {{ item.total }} 正解</span>
              <span>{{ item.accuracy }}%</span>
            </div>
          </li>
        </ul>
      </section>
    </section>

    <section v-else-if="stage === 'quiz'" class="card">
      <header class="quiz-header">
        <p class="progress">{{ currentQuestionIndex + 1 }} / {{ totalQuestions }}</p>
        <p class="timer" :class="{ 'timer-warning': timeLeft <= 5 }">
          残り {{ timeLeft }} 秒
        </p>
      </header>

      <h2 class="question-title">{{ currentQuestion.text }}</h2>
      <p v-if="currentQuestion.description" class="question-desc">{{ currentQuestion.description }}</p>

      <ul>
        <li v-for="option in currentQuestion.options" :key="option.value">
          <button
            type="button"
            class="option-button"
            :class="buttonClass(option.value)"
            :disabled="Boolean(feedback)"
            @click="selectAnswer(option.value)"
          >
            {{ option.label }}
          </button>
        </li>
      </ul>

      <p
        v-if="feedback"
        class="feedback"
        :class="feedbackClass"
      >
        {{ feedback }}
      </p>

      <div class="actions">
        <button v-if="feedback && !isLastQuestion" type="button" @click="goNext">
          次の問題へ
        </button>
        <button v-else-if="feedback && isLastQuestion" type="button" @click="finishQuiz">
          結果を見る
        </button>
      </div>
    </section>

    <section v-else class="card">
      <h2>結果</h2>
      <div class="score">
        <span>正解数</span>
        <strong>{{ correctCount }} / {{ totalQuestions }}</strong>
      </div>
      <p class="accuracy">正答率: {{ accuracy }}%</p>

      <button class="restart-button" type="button" @click="restart">
        カテゴリ選択に戻る
      </button>
    </section>
  </main>
</template>

<script setup>
import { computed, reactive, ref, watch, onBeforeUnmount } from 'vue';

const TIMER_SECONDS = 30;
const SCORE_HISTORY_KEY = 'vue-quiz-score-history';

const categories = [
  {
    id: 'vue-basics',
    name: 'Vue 基礎',
    level: '★☆☆',
    questions: [
      {
        id: 1,
        text: 'Vue 3 で推奨される記法はどれ？',
        options: [
          { value: 'options', label: 'Options API' },
          { value: 'composition', label: 'Composition API' },
          { value: 'class', label: 'Class API' }
        ],
        answer: 'composition'
      },
      {
        id: 2,
        text: 'テンプレート内でリアクティブな値を扱う際に使用するのは？',
        options: [
          { value: 'reactive', label: '`reactive` 関数' },
          { value: 'setup', label: '`setup` 関数' },
          { value: 'ref', label: '`ref` でラップした値' }
        ],
        answer: 'ref'
      },
      {
        id: 3,
        text: 'v-for ディレクティブで推奨される追加属性は？',
        options: [
          { value: 'key', label: '`key` 属性' },
          { value: 'ref', label: '`ref` 属性' },
          { value: 'slot', label: '`slot` 属性' }
        ],
        answer: 'key'
      }
    ]
  },
  {
    id: 'tooling',
    name: 'ツールチェーン',
    level: '★★☆',
    questions: [
      {
        id: 4,
        text: 'Vite の主な特徴は次のうちどれ？',
        options: [
          { value: 'slow', label: 'ビルドが遅い' },
          { value: 'ssr-only', label: 'SSR 専用のフレームワーク' },
          { value: 'fast', label: '高速な開発サーバ' }
        ],
        answer: 'fast'
      },
      {
        id: 5,
        text: 'Vite プロジェクトで Vue プラグインを有効にする設定ファイルは？',
        options: [
          { value: 'vite.config.js', label: '`vite.config.js`' },
          { value: 'package.json', label: '`package.json`' },
          { value: 'main.js', label: '`main.js`' }
        ],
        answer: 'vite.config.js'
      },
      {
        id: 6,
        text: 'Vite で開発サーバを起動するコマンドは？',
        options: [
          { value: 'npm-run-build', label: '`npm run build`' },
          { value: 'npm-run-dev', label: '`npm run dev`' },
          { value: 'npm-start', label: '`npm start`' }
        ],
        answer: 'npm-run-dev'
      }
    ]
  },
  {
    id: 'advanced',
    name: 'Vue 応用',
    level: '★★★',
    questions: [
      {
        id: 7,
        text: 'Composition API で複数コンポーネントをまたいでロジックを共有する方法は？',
        options: [
          { value: 'mixins', label: 'Mixins' },
          { value: 'composables', label: 'Composable 関数' },
          { value: 'filters', label: 'Filters' }
        ],
        answer: 'composables'
      },
      {
        id: 8,
        text: 'Vue で非同期データ取得を行う際に、ローディング状態を持つのに適したフックは？',
        options: [
          { value: 'onMounted', label: '`onMounted`' },
          { value: 'watch', label: '`watch`' },
          { value: 'computed', label: '`computed`' }
        ],
        answer: 'onMounted'
      },
      {
        id: 9,
        text: 'Pinia ストアで状態を定義するプロパティは？',
        options: [
          { value: 'actions', label: '`actions`' },
          { value: 'getters', label: '`getters`' },
          { value: 'state', label: '`state`' }
        ],
        answer: 'state'
      }
    ]
  }
];

const state = reactive({
  stage: 'category',
  currentCategoryId: '',
  questions: [],
  currentQuestionIndex: 0,
  selectedAnswer: '',
  feedback: '',
  feedbackType: '',
  correctCount: 0
});

const timeLeft = ref(TIMER_SECONDS);
const scoreHistory = ref(loadScoreHistory());
let timerHandle = null;

const currentCategory = computed(() => categories.find((category) => category.id === state.currentCategoryId));
const currentQuestion = computed(() => state.questions[state.currentQuestionIndex]);
const totalQuestions = computed(() => state.questions.length);
const currentQuestionIndex = computed(() => state.currentQuestionIndex);
const feedback = computed(() => state.feedback);
const feedbackClass = computed(() => {
  if (state.feedbackType === 'correct') return 'text-green';
  if (state.feedbackType === 'incorrect') return 'text-red';
  if (state.feedbackType === 'timeout') return 'text-orange';
  return '';
});
const correctCount = computed(() => state.correctCount);
const accuracy = computed(() => totalQuestions.value ? Math.round((state.correctCount / totalQuestions.value) * 100) : 0);
const stage = computed(() => state.stage);
const isLastQuestion = computed(() => state.currentQuestionIndex === state.questions.length - 1);

watch(
  () => [state.stage, state.currentQuestionIndex],
  ([newStage]) => {
    if (newStage === 'quiz') {
      restartTimer();
    } else {
      stopTimer();
    }
  }
);

function startQuiz(categoryId) {
  state.currentCategoryId = categoryId;
  state.questions = [...(categories.find((category) => category.id === categoryId)?.questions ?? [])];
  state.stage = 'quiz';
  state.currentQuestionIndex = 0;
  state.selectedAnswer = '';
  state.feedback = '';
  state.feedbackType = '';
  state.correctCount = 0;
  restartTimer();
}

function selectAnswer(value) {
  if (state.feedback) return;

  state.selectedAnswer = value;
  stopTimer();

  const isCorrect = value === currentQuestion.value.answer;
  state.feedback = isCorrect ? '正解！' : '残念！';
  state.feedbackType = isCorrect ? 'correct' : 'incorrect';

  if (isCorrect) {
    state.correctCount += 1;
  }
}

function goNext() {
  if (!state.feedback) return;

  if (isLastQuestion.value) {
    finishQuiz();
    return;
  }

  state.currentQuestionIndex += 1;
  state.selectedAnswer = '';
  state.feedback = '';
  state.feedbackType = '';
  restartTimer();
}

function finishQuiz() {
  stopTimer();
  state.stage = 'finished';
  persistScore();
}

function restart() {
  stopTimer();
  state.stage = 'category';
  state.currentCategoryId = '';
  state.questions = [];
  state.currentQuestionIndex = 0;
  state.selectedAnswer = '';
  state.feedback = '';
  state.feedbackType = '';
  state.correctCount = 0;
  timeLeft.value = TIMER_SECONDS;
}

function buttonClass(value) {
  if (!state.feedback) {
    return value === state.selectedAnswer ? 'is-selected' : '';
  }

  if (value === currentQuestion.value.answer) {
    return 'is-correct';
  }

  if (value === state.selectedAnswer) {
    return 'is-incorrect';
  }

  return '';
}

function restartTimer() {
  stopTimer();
  timeLeft.value = TIMER_SECONDS;
  timerHandle = setInterval(() => {
    if (timeLeft.value <= 1) {
      handleTimeout();
      return;
    }
    timeLeft.value -= 1;
  }, 1000);
}

function stopTimer() {
  if (timerHandle) {
    clearInterval(timerHandle);
    timerHandle = null;
  }
}

function handleTimeout() {
  stopTimer();
  if (state.feedback) return;

  timeLeft.value = 0;
  state.feedback = '時間切れ';
  state.feedbackType = 'timeout';
}

function persistScore() {
  if (!currentCategory.value || typeof window === 'undefined' || !window.localStorage) {
    return;
  }

  const entry = {
    id: `${Date.now()}-${Math.random().toString(16).slice(2)}`,
    categoryName: currentCategory.value.name,
    correct: state.correctCount,
    total: state.questions.length,
    accuracy: accuracy.value,
    playedAt: formatDate(new Date())
  };

  scoreHistory.value = [entry, ...scoreHistory.value].slice(0, 10);
  window.localStorage.setItem(SCORE_HISTORY_KEY, JSON.stringify(scoreHistory.value));
}

function loadScoreHistory() {
  try {
    if (typeof window === 'undefined' || !window.localStorage) {
      return [];
    }

    const saved = window.localStorage.getItem(SCORE_HISTORY_KEY);
    return saved ? JSON.parse(saved) : [];
  } catch (error) {
    console.warn('Failed to load score history', error);
    return [];
  }
}

function formatDate(date) {
  const y = date.getFullYear();
  const m = String(date.getMonth() + 1).padStart(2, '0');
  const d = String(date.getDate()).padStart(2, '0');
  const hh = String(date.getHours()).padStart(2, '0');
  const mm = String(date.getMinutes()).padStart(2, '0');
  return `${y}/${m}/${d} ${hh}:${mm}`;
}

onBeforeUnmount(() => {
  stopTimer();
});
</script>

<style scoped>
main {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.card {
  width: min(520px, 92vw);
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0 12px 32px rgba(17, 24, 39, 0.18);
  padding: 32px 28px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.quiz-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.question-title {
  margin: 0;
  font-size: 1.2rem;
}

.question-desc {
  margin: -12px 0 8px;
  color: #6b7280;
  font-size: 0.95rem;
}

.category-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  gap: 12px;
}

.category-button {
  width: 100%;
  text-align: left;
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  color: #1d4ed8;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.category-button strong {
  font-size: 1.1rem;
}

.category-button small {
  color: #1e3a8a;
}

.hint {
  margin: -8px 0 12px;
  color: #6b7280;
  font-size: 0.9rem;
}

.timer {
  font-weight: 600;
  color: #2563eb;
}

.timer-warning {
  color: #ef4444;
}

.feedback {
  font-weight: 600;
  text-align: center;
}

.text-green {
  color: #16a34a;
}

.text-red {
  color: #ef4444;
}

.text-orange {
  color: #f97316;
}

.actions {
  display: flex;
  justify-content: center;
}

.score {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 1.1rem;
}

.accuracy {
  margin-top: -8px;
  text-align: center;
  font-weight: 600;
}

.history {
  border-top: 1px solid #e5e7eb;
  padding-top: 16px;
}

.history h3 {
  margin: 0 0 12px 0;
}

.history ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.history li {
  background: #f9fafb;
  border-radius: 12px;
  padding: 12px 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.history-score {
  display: flex;
  flex-direction: column;
  text-align: right;
  gap: 2px;
}

.history-date {
  display: block;
  color: #9ca3af;
  font-size: 0.85rem;
}

.restart-button {
  background: linear-gradient(135deg, #f59e0b, #f97316);
}

@media (max-width: 540px) {
  .card {
    padding: 24px 20px;
    gap: 16px;
  }

  .quiz-header {
    flex-direction: column;
    gap: 6px;
    align-items: flex-start;
  }

  .history li {
    flex-direction: column;
    align-items: flex-start;
    gap: 6px;
  }

  .history-score {
    flex-direction: row;
    justify-content: space-between;
    width: 100%;
  }
}
</style>
