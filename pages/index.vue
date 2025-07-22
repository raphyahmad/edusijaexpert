<template>
  <div class="quiz-container">
    <!-- Animated Gradient Background -->
    <div class="animated-bg"></div>

    <!-- Quiz Complete Screen -->
    <transition name="fade">
      <div v-if="quizComplete" class="quiz-card complete-card">
        <div class="trophy animate-bounce">🏆</div>
        <h2 class="complete-title">Quiz Complete!</h2>
        <div class="final-score">{{ score }}/{{ questions.length }}</div>
        <p class="score-message">{{ getScoreMessage() }}</p>
        <button @click="restartQuiz" class="restart-btn">
          🔄 Try Again
        </button>
      </div>
    </transition>

    <!-- Quiz Screen -->
    <transition name="slide">
      <div  class="quiz-card">
        <!-- Header -->
        <div class="header">
          <div class="header-left">
            <div class="brain-icon animate-pulse">🧠</div>
            <h1 class="app-title">EduSijaExpert</h1>
          </div>
          <div class="score-display">
            <div class="score-label">Score</div>
            <div class="score-value">{{ score }}/{{ questions.length }}</div>
          </div>
        </div>

        <!-- Progress Bar -->
        <div class="progress-section">
          <div class="progress-info">
            <span class="question-number">Question {{ currentQuestion + 1 }} of {{ questions.length }}</span>
            <span class="category">{{ questions[currentQuestion].category }}</span>
          </div>
          <div class="progress-bar">
            <div 
              class="progress-fill"
              :style="{ width: `${((currentQuestion + 1) / questions.length) * 100}%` }"
            ></div>
          </div>
        </div>

        <!-- Timer -->
        <div class="timer-section">
          <div 
            :class="['timer', { 'timer-warning': timeLeft <= 5, 'timer-shake': timeLeft <= 3 }]"
          >
            <svg viewBox="0 0 36 36" class="circular-timer">
              <path
                class="circle-bg"
                d="M18 2.0845
                  a 15.9155 15.9155 0 0 1 0 31.831
                  a 15.9155 15.9155 0 0 1 0 -31.831"
              />
              <path
                class="circle"
                :stroke="timeLeft > 5 ? '#38bdf8' : '#ef4444'"
                :d="describeArc(18, 18, 15.9155, 0, ((timeLeft / 15) * 360))"
              />
              <text x="18" y="22" class="timer-text">{{ timeLeft }}</text>
            </svg>
          </div>
        </div>

        <!-- Question -->
        <transition name="slide-up" mode="out-in">
          <div class="question-section" :key="currentQuestion">
            <h2 class="question-text">
              {{ questions[currentQuestion].question }}
            </h2>
            
            <div class="options-container">
              <button
                v-for="(option, index) in questions[currentQuestion].options"
                :key="index"
                @click="handleAnswer(index)"
                :disabled="showResult"
                :class="getButtonClass(index)"
                class="option-button"
                tabindex="0"
              >
                <div class="option-content">
                  <span class="option-text">{{ option }}</span>
                  <!-- Only use v-if, not v-else-if! -->
                  <span
                    v-if="showResult && index === questions[currentQuestion].correct"
                    class="result-icon correct"
                  >✅</span>
                  <span
                    v-if="showResult && index === selectedAnswer && index !== questions[currentQuestion].correct"
                    class="result-icon incorrect"
                  >❌</span>
                </div>
              </button>
            </div>
          </div>
        </transition>

        <!-- Result Message -->
        <div v-if="showResult" class="result-section">
          <div v-if="selectedAnswer === questions[currentQuestion].correct" class="result-message correct">
            🎉 Correct! Well done!
          </div>
          <div v-if="selectedAnswer !== questions[currentQuestion].correct" class="result-message incorrect">
            ❌ {{ selectedAnswer === null ? "Time's up!" : "Incorrect." }} The answer was: {{ questions[currentQuestion].options[questions[currentQuestion].correct] }}
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const currentQuestion = ref(0)
const selectedAnswer = ref(null)
const showResult = ref(false)
const score = ref(0)
const quizComplete = ref(false)
const timeLeft = ref(15)
const timerActive = ref(true)
let timer = null

const questions = [
  {
    question: "What is the capital of France?",
    options: ["London", "Berlin", "Paris", "Madrid"],
    correct: 2,
    category: "Geography"
  },
  {
    question: "Which planet is known as the Red Planet?",
    options: ["Venus", "Mars", "Jupiter", "Saturn"],
    correct: 1,
    category: "Science"
  },
  {
    question: "Who painted the Mona Lisa?",
    options: ["Vincent van Gogh", "Pablo Picasso", "Leonardo da Vinci", "Michelangelo"],
    correct: 2,
    category: "Art"
  },
  {
    question: "What is the largest mammal in the world?",
    options: ["African Elephant", "Blue Whale", "Giraffe", "Polar Bear"],
    correct: 1,
    category: "Nature"
  },
  {
    question: "In which year did World War II end?",
    options: ["1944", "1945", "1946", "1947"],
    correct: 1,
    category: "History"
  }
]

const startTimer = () => {
  timer = setInterval(() => {
    if (timeLeft.value > 0 && timerActive.value && !showResult.value && !quizComplete.value) {
      timeLeft.value--
    } else if (timeLeft.value === 0 && !showResult.value) {
      handleAnswer(null)
    }
  }, 1000)
}

const stopTimer = () => {
  if (timer) {
    clearInterval(timer)
    timer = null
  }
}

const handleAnswer = (answerIndex) => {
  if (showResult.value) return
  
  selectedAnswer.value = answerIndex
  showResult.value = true
  timerActive.value = false
  stopTimer()
  
  if (answerIndex === questions[currentQuestion.value].correct) {
    score.value++
  }
  
  setTimeout(() => {
    if (currentQuestion.value + 1 < questions.length) {
      currentQuestion.value++
      selectedAnswer.value = null
      showResult.value = false
      timeLeft.value = 15
      timerActive.value = true
      startTimer()
    } else {
      quizComplete.value = true
    }
  }, 1800)
}

const restartQuiz = () => {
  currentQuestion.value = 0
  selectedAnswer.value = null
  showResult.value = false
  score.value = 0
  quizComplete.value = false
  timeLeft.value = 15
  timerActive.value = true
  startTimer()
}

const getScoreMessage = () => {
  const percentage = (score.value / questions.length) * 100
  if (percentage === 100) return "Perfect! 🌟"
  if (percentage >= 80) return "Excellent! 🎉"
  if (percentage >= 60) return "Good job! 👏"
  if (percentage >= 40) return "Not bad! 👍"
  return "Keep practicing! 💪"
}

const getButtonClass = (index) => {
  let classes = []
  if (showResult.value) {
    if (index === questions[currentQuestion.value].correct) {
      classes.push('correct-answer')
    } else if (index === selectedAnswer.value) {
      classes.push('incorrect-answer')
    }
  }
  if (selectedAnswer.value === index) {
    classes.push('selected')
  }
  return classes.join(' ')
}

// SVG Arc for timer circle
function polarToCartesian(centerX, centerY, radius, angleInDegrees) {
  const angleInRadians = (angleInDegrees - 90) * Math.PI / 180.0
  return {
    x: centerX + (radius * Math.cos(angleInRadians)),
    y: centerY + (radius * Math.sin(angleInRadians))
  }
}

function describeArc(x, y, radius, startAngle, endAngle) {
  const start = polarToCartesian(x, y, radius, endAngle)
  const end = polarToCartesian(x, y, radius, startAngle)
  const arcSweep = endAngle - startAngle <= 180 ? "0" : "1"
  return [
    "M", start.x, start.y,
    "A", radius, radius, 0, arcSweep, 0, end.x, end.y
  ].join(" ")
}

onMounted(() => {
  startTimer()
})

onUnmounted(() => {
  stopTimer()
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&family=Montserrat:wght@700&display=swap');

* {
  box-sizing: border-box;
}

.quiz-container {
  min-height: 100vh;
  width: 100vw;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Inter', 'Montserrat', Arial, sans-serif;
  z-index: 1;
}

.animated-bg {
  position: fixed;
  inset: 0;
  z-index: 0;
  background: linear-gradient(120deg, #74ebd5 0%, #ACB6E5 100%);
  background-size: 200% 200%;
  animation: gradientMove 6s ease-in-out infinite;
}

@keyframes gradientMove {
  0%, 100% { background-position: 0% 50%;}
  50% { background-position: 100% 50%;}
}

.quiz-card {
  position: relative;
  background: rgba(255, 255, 255, 0.18);
  backdrop-filter: blur(15px) brightness(1.1);
  border-radius: 2.2rem;
  padding: 2.5rem 2rem 2rem 2rem;
  max-width: 540px;
  width: 100%;
  border: 1.5px solid rgba(255, 255, 255, 0.22);
  box-shadow: 0 28px 60px 0 rgba(59,130,246,0.15), 0 1.5px 6px 0 rgba(0,0,0,0.10);
  z-index: 2;
  margin: 2rem 0;
  overflow: hidden;
}

.complete-card {
  max-width: 420px;
  text-align: center;
  animation: pulse 2s infinite;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2.2rem;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 0.85rem;
}

.brain-icon {
  font-size: 2.25rem;
}

.app-title {
  font-size: 1.6rem;
  font-family: 'Montserrat', Arial, sans-serif;
  font-weight: bold;
  color: #312e81;
  margin: 0;
  letter-spacing: 2px;
}

.score-display {
  text-align: right;
}

.score-label {
  color: #6c63ff;
  font-size: 0.98rem;
  font-family: 'Montserrat', Arial, sans-serif;
}

.score-value {
  font-size: 1.32rem;
  font-weight: bold;
  color: #312e81;
}

.progress-section {
  margin-bottom: 2rem;
}

.progress-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.58rem;
}

.question-number,
.category {
  color: #22223b;
  font-size: 0.99rem;
  opacity: 0.85;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: rgba(128, 90, 213, 0.13);
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 1px 2px 0 rgba(59,130,246,0.08);
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #6c63ff, #5eead4, #6366f1, #06b6d4);
  transition: width 0.3s cubic-bezier(.79,.14,.15,.86);
  border-radius: 8px 0 0 8px;
}

.timer-section {
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
}

.timer {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.35rem;
  font-weight: bold;
  background: rgba(255, 255, 255, 0.23);
  color: #1e293b;
  transition: all 0.25s;
  position: relative;
  box-shadow: 0 2px 10px rgba(93,188,252,0.12);
}

.timer-warning {
  background: rgba(252, 165, 165, 0.38);
  color: #dc2626;
  animation: pulse 0.7s infinite;
}

.timer-shake {
  animation: shake 0.15s infinite alternate;
}

@keyframes shake {
  0% { transform: translateX(0);}
  100% { transform: translateX(3px);}
}

.circular-timer {
  width: 80px;
  height: 80px;
  transform: rotate(0deg);
  position: absolute;
  top: 0; left: 0;
}

.circle-bg {
  fill: none;
  stroke: #cbd5e1;
  stroke-width: 3.2;
}

.circle {
  fill: none;
  stroke-width: 4;
  stroke-linecap: round;
  transition: stroke 0.4s;
}

.timer-text {
  fill: #1e293b;
  font-size: 1.48rem;
  font-family: 'Montserrat', Arial, sans-serif;
  font-weight: 700;
  text-anchor: middle;
}

.question-section {
  margin-bottom: 2rem;
}

.question-text {
  font-size: 1.37rem;
  font-weight: 700;
  color: #22223b;
  margin-bottom: 1.3rem;
  line-height: 1.35;
  letter-spacing: 0.2px;
  font-family: 'Montserrat', Arial, sans-serif;
}

.options-container {
  display: flex;
  flex-direction: column;
  gap: 0.85rem;
}

.option-button {
  width: 100%;
  padding: 1.13rem;
  border-radius: 15px;
  border: 2px solid rgba(128, 90, 213, 0.18);
  background: rgba(245, 245, 255, 0.45);
  color: #22223b;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.19s cubic-bezier(.79,.14,.15,.86);
  font-size: 1.04rem;
  box-shadow: 0 1.5px 6px 0 rgba(0,0,0,0.06);
  outline: none;
  position: relative;
}

.option-button:focus {
  border-color: #8b5cf6;
  box-shadow: 0 0 0 2px #c7d2fe;
}

.option-button:hover:not(:disabled), .option-button:focus:not(:disabled) {
  background: rgba(125, 211, 252, 0.15);
  color: #312e81;
  transform: scale(1.025);
  border-color: #38bdf8;
}

.option-button:disabled {
  cursor: not-allowed;
  opacity: 0.82;
}

.option-button.selected {
  box-shadow: 0 0 0 3px #fbbf24, 0 1.5px 6px 0 rgba(0,0,0,0.12);
  border-color: #fbbf24;
}

.option-button.correct-answer {
  background: linear-gradient(90deg, #a7ff83 0%, #17ead9 100%);
  color: #1e293b;
  border-color: #4ade80;
}

.option-button.incorrect-answer {
  background: linear-gradient(90deg, #ffafcc 0%, #ffb88c 100%);
  color: #991b1b;
  border-color: #f87171;
}

.option-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.option-text {
  font-weight: 600;
}

.result-icon {
  font-size: 1.22rem;
}

.result-section {
  text-align: center;
  margin-top: 1.1rem;
}

.result-message {
  font-weight: 700;
  font-size: 1.13rem;
  font-family: 'Montserrat', Arial, sans-serif;
}

.result-message.correct {
  color: #16a34a;
  animation: bounce 1s infinite;
}

.result-message.incorrect {
  color: #f87171;
}

.trophy {
  font-size: 5rem;
  margin-bottom: 1.5rem;
  text-shadow: 0 2px 20px #fbbf24;
}

.complete-title {
  font-size: 2.5rem;
  font-weight: bold;
  color: #22223b;
  margin-bottom: 1rem;
}

.final-score {
  font-size: 3.5rem;
  font-weight: bold;
  background: linear-gradient(45deg, #fbbf24, #f97316);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 1rem;
}

.score-message {
  font-size: 1.25rem;
  color: #22223b;
  margin-bottom: 2rem;
}

.restart-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  background: linear-gradient(45deg, #8b5cf6, #ec4899);
  color: white;
  padding: 1rem 2rem;
  border-radius: 16px;
  border: none;
  font-weight: 600;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.12);
}

.restart-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.18);
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(-18%);
    animation-timing-function: cubic-bezier(0.8, 0, 1, 1);
  }
  50% {
    transform: translateY(0);
    animation-timing-function: cubic-bezier(0, 0, 0.2, 1);
  }
}

@media (max-width: 768px) {
  .quiz-card {
    padding: 1.2rem;
  }
  
  .question-text {
    font-size: 1.18rem;
  }
  
  .complete-title {
    font-size: 2rem;
  }
  
  .final-score {
    font-size: 2.5rem;
  }
}
</style>
