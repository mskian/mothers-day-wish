<template>
  <section class="section has-text-centered min-vh-100">
    <div class="container">

      <div class="is-flex is-justify-content-center is-align-items-center py-5">
          <label class="switch" aria-label="Toggle dark mode">
          <input type="checkbox" v-model="isDarkMode" />
          <span class="slider"></span>
          </label>
      </div>

      <div v-if="!name" class="box p-6 animated fadeInDown has-background-white-ter">
        <h1 class="title is-4 has-text-warning-dark mb-4 has-text-weight-medium">💐 Send a Mother's Day Wish 💐</h1>
        <p class="subtitle is-6 has-text-black has-text-weight-medium mb-5">
          Celebrate the love and care of your mom with a heartfelt message.
        </p>

        <div class="field">
          <label class="label has-text-black">Enter your Name</label>
          <div class="control has-icons-left">
            <input
              class="input is-danger"
              type="text"
              v-model="inputName"
              :class="{ 'is-danger': error }"
              placeholder="Your name"
              maxlength="40"
              aria-label="Mom's name input"
            />
            <span class="icon is-left">
              ❤️
            </span>
          </div>
          <p v-if="error" class="help is-danger mt-2">{{ error }}</p>
        </div>

        <button class="button is-info has-background-link-dark has-text-white is-rounded mt-4" @click="submitName">
          🌸 Generate Wish
        </button>
      </div>

      <div v-else class="box p-6 animated fadeInUp has-background-white-ter">
        <div class="mb-5">
          <h2 class="title is-5 has-text-danger-dark has-text-centered has-text-weight-semibold">
            Happy Mother's Day ❤️ {{ name }}
          </h2> 
          <AudioPlayer
              src="/mothers-day.mp3"
              title="Mother's Day 🎵"
           />
          <ImageCanvas :image="imageUrl" :text="name" />
         <transition name="fade" mode="out-in">
          <p :key="quoteIndex" class="subtitle has-text-weight-medium is-5 has-text-black mt-6 mb-4">
            {{ isTamil ? tamilQuotes[quoteIndex] : englishQuotes[quoteIndex] }}
          </p>
         </transition>

        <div class="is-flex is-justify-content-center is-align-items-center is-flex-direction-column mt-4">
            <label class="switch mb-2">
              <input type="checkbox" v-model="isTamil" />
              <span class="slider"></span>
            </label>
            <span class="has-text-weight-semibold has-text-black">Quotes in {{ isTamil ? 'தமிழ்' : 'English' }}</span>
          </div>
        </div>
        <hr class="has-background-black mt-5">
        <div class="buttons is-centered mt-5">
          <button class="button is-warning has-background-warning-dark is-rounded has-text-white" @click="newQuote">
            ✨ Refresh
          </button>
          <a class="button is-danger has-background-danger-dark is-rounded has-text-white" href="/">
            💌 Create
          </a>
        </div>
        <hr class="has-background-black">
       <div v-if="name" class="buttons is-centered mt-6 mb-4">
         <button class="button is-danger has-background-danger-light is-rounded has-text-black" @click="copyGreetingUrl">
            🔗 Copy Greeting URL
         </button>

        <transition name="toast">
          <div
            v-if="toastVisible"
           class="snackbar-notification"
           role="alert"
           aria-live="assertive"
          >
           ✅ Greeting URL copied
          </div>
        </transition>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import DOMPurify from 'dompurify'
import slugify from 'slugify'
import AudioPlayer from './components/AudioPlayer.vue'
import ImageCanvas from './components/ImageCanvas.vue'

const imageUrl = 'mothers-day.png';

const inputName = ref('')
const name = ref('')
const error = ref('')
const isTamil = ref(false)
const quoteIndex = ref(0)
const toastVisible = ref(false)
const isDarkMode = ref(false)

onMounted(() => {
  const saved = localStorage.getItem('darkMode')
  if (saved === 'true') {
    isDarkMode.value = true
    document.body.classList.add('dark-mode')
  }
})

watch(isDarkMode, (value) => {
  document.body.classList.toggle('dark-mode', value)
  localStorage.setItem('darkMode', value)
})

const englishQuotes = [
  "A mother's love is strength wrapped in softness 💗",
  "Every strong soul was first held by a loving mom 💪",
  "Your arms were my first home and your heart is still my world 🏡",
  "You gave me life and filled it with unconditional love 🌷",
  "Your voice still soothes my heart like a gentle song 🎵",
  "No crown shines like the quiet sacrifice of a mother 👑",
  "You are my guide my courage and the light in every step 🌟",
  "Life begins with love and that love begins with you ❤️"
]

const tamilQuotes = [
  "தாயின் அன்பு பாசத்தால் நிரம்பிய கடல் 💖",
  "அம்மாவின் ஆசீர்வாதம் வாழ்க்கையின் இசை 🎶",
  "தாய் என்பது உயிர் மூச்சில் இயங்கும் ஒரு அற்புதம் 🌸",
  "அம்மாவின் நிழல் தான் என் வாழ்வின் நிழற்குடை ☂️",
  "அம்மாவின் புன்னகை மனதின் அமைதி 😊",
  "அம்மா என்ற சொல் என் வாழ்வின் ஒளி ✨",
  "அம்மாவின் இதயம் பாசத்தின் கோபுரம் 🏰",
  "தாயின் தொட்டிலில் துவங்கும் என் கனவுகள் 🎼"
]

function sanitizeAndValidate(raw) {
  const clean = DOMPurify.sanitize(raw.trim())

  if (
    !clean ||
    clean.length < 2 ||
    clean.length > 40 ||
    /[!@#$%^&*()_{}\[\]:";?/><.,]/.test(clean)
  ) {
    return null
  }

  return clean
}

function submitName() {
  const safe = sanitizeAndValidate(inputName.value)
  if (!safe) {
    error.value = 'Please enter a valid name (only letters and spaces, 2–40 chars).'
    return
  }

  error.value = ''
  const slug = slugify(safe, {  replacement: '-',  remove: /[*+~.()'"!:@]/g, lower: false, strict: false })
  window.location.href = `/?name=${slug}`
}

function copyGreetingUrl() {
  const url = `${window.location.origin}/?name=${slugify(name.value, {
    replacement: '-',
    remove: /[*+~.()'"!:@]/g,
    lower: false,
    strict: false
  })}`

  navigator.clipboard.writeText(url).then(() => {
    toastVisible.value = true
    setTimeout(() => (toastVisible.value = false), 3000)
  })
}

onMounted(() => {
  const urlParams = new URLSearchParams(window.location.search)
  const rawName = urlParams.get('name')

  if (rawName === '') {
    window.location.href = '/'
    return
  }

  if (rawName) {
    const cleanParam = DOMPurify.sanitize(rawName).replace(/-/g, ' ')
    const safeName = sanitizeAndValidate(cleanParam)

    if (safeName) {
      const slug = slugify(safeName, {
        replacement: '-',
        remove: /[*+~.()'"!:@]/g,
        lower: false,
        strict: false,
      })

      if (slug !== rawName) {
        window.location.href = `/?name=${slug}`
        return
      }

      name.value = safeName
      setMetaTags(safeName)
    } else {
      window.location.href = '/'
    }
  }

  quoteIndex.value = Math.floor(Math.random() * englishQuotes.length)
})

function newQuote() {
  let newIndex
  do {
    newIndex = Math.floor(Math.random() * englishQuotes.length)
  } while (newIndex === quoteIndex.value)

  quoteIndex.value = newIndex
}
function setMetaTags(momName) {
  const cleanTitle = `Happy Mother's Day Wishes From ${momName} 💐`
  const cleanDesc = `Mother's Day Greeting Wishes From ${momName} with heartfelt bilingual wishes.`
  const cleanURL = `${window.location.origin}/?name=${slugify(momName, {  replacement: '-',  remove: /[*+~.()'"!:@]/g, lower: false, strict: false })}`

  document.title = cleanTitle

  const metaDesc = document.querySelector('meta[name="description"]') || document.createElement('meta')
  metaDesc.name = 'description'
  metaDesc.content = cleanDesc
  document.head.appendChild(metaDesc)

  const canonical = document.querySelector('link[rel="canonical"]') || document.createElement('link')
  canonical.rel = 'canonical'
  canonical.href = cleanURL
  document.head.appendChild(canonical)
}

</script>

<style>
.min-vh-100 {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.animated {
  animation-duration: 1s;
  animation-fill-mode: both;
}

.fadeInDown {
  animation-name: fadeInDown;
}

.fadeInUp {
  animation-name: fadeInUp;
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

.switch {
  position: relative;
  display: inline-block;
  width: 52px;
  height: 28px;
  margin-right: 10px;
}
.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}
.slider {
  position: absolute;
  cursor: pointer;
  background-color: #ccc;
  border-radius: 34px;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  transition: 0.4s;
}
.slider:before {
  position: absolute;
  content: "";
  height: 22px;
  width: 22px;
  left: 3px;
  bottom: 3px;
  background-color: white;
  transition: 0.4s;
  border-radius: 50%;
}
input:checked + .slider {
  background-color: #4cd964;
}
input:checked + .slider:before {
  transform: translateX(24px);
}

@media (max-width: 768px) {
  .title.is-3 {
    font-size: 1.5rem;
  }

  .switch {
    width: 46px;
    height: 24px;
  }

  .slider:before {
    height: 18px;
    width: 18px;
    left: 3px;
    bottom: 3px;
  }

  input:checked + .slider:before {
    transform: translateX(22px);
  }
}
.box {
  border: 2px solid #ff3860;
  font-family: "Anek Tamil", sans-serif;
  border-radius: 1rem;
  box-shadow: 0 0 12px rgba(255, 56, 96, 0.4);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.box:hover {
  transform: translateY(-4px);
  box-shadow: 0 0 20px rgba(255, 56, 96, 0.6);
}
.snackbar-notification {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: #1e2421;
  color: white;
  padding: 1rem 1.5rem;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  z-index: 1000;
  font-weight: 600;
  max-width: 90%;
  text-align: center;
}
.toast-enter-active,
.toast-leave-active {
  transition: opacity 0.4s ease, transform 0.4s ease;
}
.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translateY(20px);
}
</style>