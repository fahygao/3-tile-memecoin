<script setup lang="ts">
import { onMounted, ref } from 'vue'
import Card from './components/card.vue'
import { useGame } from './core/useGame'
import { basicCannon, schoolPride } from './core/utils'


const initialTime = 100; // Initial time in seconds
const timeLeft = ref(initialTime); // Current time left (in seconds)
const containerRef = ref<HTMLElement | undefined>()
const clickAudioRef = ref<HTMLAudioElement | undefined>()
const dropAudioRef = ref<HTMLAudioElement | undefined>()
const winAudioRef = ref<HTMLAudioElement | undefined>()
const loseAudioRef = ref<HTMLAudioElement | undefined>()
const welAudioRef = ref<HTMLAudioElement | undefined>()
const curLevel = ref(1)
const showTip = ref(false)
const LevelConfig = [
  { cardNum: 4, layerNum: 2, trap: false },
  { cardNum: 9, layerNum: 3, trap: false },
  { cardNum: 15, layerNum: 6, trap: false },
]

// Simulate countdown decrementing timeLeft every second
let countdownInterval: NodeJS.Timeout | null = null; // Explicitly type countdownInterval

// Function to start the countdown timer
const startCountdown = () => {
  countdownInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value -= 1;
    } else {
      clearInterval(countdownInterval!); // Use non-null assertion operator
      handleLose(); // Call lose logic when time runs out
    }
  }, 1000);
};



const isWin = ref(false)


const {
  nodes,
  selectedNodes,
  handleSelect,
  handleBack,
  backFlag,
  handleRemove,
  removeFlag,
  removeList,
  handleSelectRemove,
  initData,
} = useGame({
  container: containerRef,
  cardNum: 4,
  layerNum: 2,
  trap: false,
  events: {
    clickCallback: handleClickCard,
    dropCallback: handleDropCard,
    winCallback: handleWin,
    loseCallback: handleLose,
  },
})



function handleClickCard() {
  if (clickAudioRef.value?.paused) {
    clickAudioRef.value.play()
  }
  else if (clickAudioRef.value) {
    clickAudioRef.value.load()
    clickAudioRef.value.play()
  }
}

function handleDropCard() {
  dropAudioRef.value?.play()
}

function handleWin() {
  winAudioRef.value?.play()
  // fireworks()
  if (curLevel.value < LevelConfig.length) {
    basicCannon()
    showTip.value = true
    setTimeout(() => {
      showTip.value = false
    }, 1500)
    setTimeout(() => {
      initData(LevelConfig[curLevel.value])
      curLevel.value++
    }, 2000)
  }
  else {
    isWin.value = true
    schoolPride()
  }
}

function handleLose() {
  loseAudioRef.value?.play()
  setTimeout(() => {
    alert('Mannn, you loss. Try again!')
    nodes.value = []
    removeList.value = []
    selectedNodes.value = []
    welAudioRef.value?.play()
    curLevel.value = 0
    showTip.value = true
    setTimeout(() => {
      showTip.value = false
    }, 1500)
    setTimeout(() => {
      initData(LevelConfig[curLevel.value])
      curLevel.value++
    }, 2000)
  }, 500)
}


onMounted(() => {
  initData(); // Call initData() when component is mounted
});
</script>

<template>
  <div flex flex-col w-full h-full>
    <div class="header">
  3 TILES - $SICK Edition 
  <!-- <div v-if="timeLeft > 0" class="timer">{{ timeLeft }}s</div> -->
</div>
    <div ref="containerRef" flex-1 flex style="padding-top: 100px;">
      <div class="container" w-full relative flex-1>
        <template v-for="item in nodes" :key="item.id">
          <transition name="slide-fade">
            <Card
              v-if="[0, 1].includes(item.state)"
              :node="item"
              @click-card="handleSelect"
            />
          </transition>
        </template>
      </div>
      <transition name="bounce">
        <div v-if="isWin" color="#fff" flex items-center justify-center w-full text-28px fw-bold>
          You have got the first round!
        </div>
      </transition>
      <transition name="bounce">
        <div v-if="showTip" color="#fff" flex items-center justify-center w-full text-28px fw-bold>
          Level. {{ curLevel + 1 }}
        </div>
      </transition>
    </div>

    <div text-center h-150px w-400px flex items-center justify-center>
      <Card
        v-for="item in removeList" :key="item.id" :node="item"
        is-dock
        @click-card="handleSelectRemove"
      />
    </div>
    <div w-full flex items-center justify-center>
      <div border="~ 2px dashed #fff" w-315px h-44px flex>
        <template v-for="item in selectedNodes" :key="item.id">
          <transition name="bounce">
            <Card
              v-if="item.state === 2"
              :node="item"
              is-dock
            />
          </transition>
        </template>
      </div>
    </div>

    <div h-50px mt-20px flex items-center w-full justify-center>
      <button :disabled="removeFlag" mr-20px @click="handleRemove">
        Remove the first three
      </button>
      <button :disabled="backFlag" @click="handleBack">
        Undo
      </button>
    </div>
    <audio
      ref="clickAudioRef"
      style="display: none;"
      controls
      src="./audio/click.mp3"
    />
    <audio
      ref="dropAudioRef"
      style="display: none;"
      controls
      src="./audio/drop.mp3"
    />
    <audio
      ref="winAudioRef"
      style="display: none;"
      controls
      src="./audio/win.mp3"
    />
    <audio
      ref="loseAudioRef"
      style="display: none;"
      controls
      src="./audio/lose.mp3"
    />
    <audio
      ref="welAudioRef"
      style="display: none;"
      controls
      src="./audio/welcome.mp3"
    />
  </div>
</template>

<style>
body{
  background-color: rgb(0, 0, 0);
  position: relative; /* Required for pseudo-element positioning */
  z-index: 1;
}

body::before {
  content: "";
  position: absolute;
  top: -100px;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url('/images/hero.png'); /* Path to your image */
  background-size: 60% auto; /* Adjust size: 50% width, auto height */
  background-position: center;
  background-repeat: no-repeat;
  z-index: -1; /* Ensure the image stays behind the content */
}


.header {
  font-size: 36px; /* Adjusted font size */
  text-align: center;
  color: #ff8500;
  font-weight: 900;
  height: 60px;
  align-items: center;
  justify-content: center;
  margin-top: 30px;
}

/* Adjustments for iPhone 15 Pro Max */
@media only screen 
  and (device-width: 428px) 
  and (device-height: 926px) 
  and (-webkit-device-pixel-ratio: 3)
  and (orientation: portrait) {
  .header {
    font-size: 32px; /* Further adjusted font size */
    line-height: 1.2;
    padding-bottom: 50px;
    margin-top: 20px;
  }
  .container {
    padding-top: 50px;
  }
  .slide-fade-enter-active {
    transition: all 0.2s ease-out;
  }

  .slide-fade-leave-active {
    transition: all 0.2s cubic-bezier(1, 0.5, 0.8, 1);
  }

  .slide-fade-enter-from,
  .slide-fade-leave-to {
    transform: translateY(20vh); /* Adjusted for smaller screen */
    opacity: 0;
  }

  .v-enter-active,
  .v-leave-active {
    transition: opacity 0.5s ease;
  }

  .v-enter-from,
  .v-leave-to {
    opacity: 0;
  }

  .bounce-enter-active {
    animation: bounce-in 0.5s;
  }

  .bounce-leave-active {
    animation: bounce-in 0.5s reverse;
  }

  @keyframes bounce-in {
    0% {
      transform: scale(0);
    }
    50% {
      transform: scale(1.15); /* Adjusted scale */
    }
    100% {
      transform: scale(1);
    }
  }
}

.container {
  /* Add padding-top for mobile devices */
  padding-top: 50px;
}


/* General styles */
.bounce-enter-active {
  animation: bounce-in 0.5s;
}
.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.25);
  }
  100% {
    transform: scale(1);
  }
}

.slide-fade-enter-active {
  transition: all 0.2s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.2s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateY(25vh);
  opacity: 0;
}

.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}


/* Timer styles */
.timer {
  font-size: 24px;
  color: #ff8500;
  margin-top: 10px;
}

/* Progress bar styles */
.progress {
  width: 100%;
  height: 10px;
  background-color: #fff;
  margin-top: 10px;
}

.progress-inner {
  height: 100%;
  background-color: #ff8500;
}

html {
  touch-action: manipulation;
}
</style>
