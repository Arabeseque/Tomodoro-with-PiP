<script setup lang="ts" generic="T extends any, O extends any">
defineOptions({
  name: 'Tomodoro',
})

const { log } = console

const currentTime = ref('')

const TomodoroData = reactive({
  session: 1, // 完成的番茄钟数量
  totalSessions: 4, // 总番茄钟数量
  workDuration: 25, // 工作时间（分钟）
  breakDuration: 5, // 休息时间（分钟）
  timer: null as any, // 计时器对象
  time: 0, // 当前计时时间（秒）
  isPaused: true, // 计时器是否暂停
  isWorking: true, // 是否工作状态
})

const progressBarWidth = computed(() => {
  const { time, workDuration, breakDuration } = TomodoroData
  if (time < workDuration * 60)
    return `${(time / (workDuration * 60)) * 100}%`

  else if (time >= workDuration * 60 && time < (workDuration + breakDuration) * 60)
    return `${((time - workDuration * 60) / (breakDuration * 60)) * 100}%`
})

function start() {
  TomodoroData.isPaused = false

  TomodoroData.timer = setInterval(() => {
    if (TomodoroData.isPaused)
      return

    TomodoroData.time += 1

    if (TomodoroData.time >= (TomodoroData.workDuration + TomodoroData.breakDuration) * 60) {
      TomodoroData.time = 0
      TomodoroData.isWorking = true
      TomodoroData.session += 1
      clearInterval(TomodoroData.timer)
    }
    else if (TomodoroData.time >= TomodoroData.workDuration * 60) {
      TomodoroData.isWorking = false
    }
  }, 1000)
}

function getCurrentTime() {
  const currentDate = new Date()
  const currentHour = currentDate.getHours()
  const currentMinute = currentDate.getMinutes()
  const formattedHour = currentHour < 10 ? `0${currentHour}` : currentHour
  const formattedMinute = currentMinute < 10 ? `0${currentMinute}` : currentMinute
  return `${formattedHour}:${formattedMinute}`
}

async function togglePiP() {
  async function enterPiP() {
    if (window.documentPictureInPicture.window) {
      window.documentPictureInPicture.window.close()
      return
    }
    const timer = document.querySelector('#timer') as any
    const timerContainer = timer.parentNode as any
    timerContainer.classList.add('pip')

    const pipOptions = {
      lockAspectRatio: true,
      copyStyleSheets: true,
      width: 250,
      height: 200,
    }

    const pipWindow = await window.documentPictureInPicture.requestWindow(pipOptions)

    pipWindow.addEventListener('pagehide', (event: any) => {
      const timerContainer = document.querySelector('#timerContainer')
      const pip = event.target.querySelector('#timer')
      // pipVideo.classList.toggle('fullpage', false)
      if (!timerContainer)
        return
      timerContainer.append(pip)
    })
    pipWindow.document.body.append(timer)
  }

  await enterPiP()
}

onMounted(() => {
  currentTime.value = getCurrentTime()
  setInterval(() => {
    currentTime.value = getCurrentTime()
  }, 1000)
})
</script>

<template>
  <div id="timerContainer" flex flex-col items-center justify-center>
    <div id="timer" flex items-center justify-center pt-8>
      <div>
        <div>
          Tomodoro
        </div>

        <div py-2 />

        <div w-60>
          <div flex items-end justify-between>
            <div>
              SESSION {{ TomodoroData.session }}/4
            </div>
            <div text-3xl font-400>
              {{ currentTime }}
            </div>
          </div>

          <div py-1 />

          <!-- progress -->
          <div flex flex-col gap-1>
            <div h-3 w-full rounded-sm bg-gray-200>
              <div
                class="h-full rounded-sm bg-green-500"
                :style="{ width: progressBarWidth }"
              />
            </div>
            <div flex justify-between pt-1>
              <div text-xs>
                0
              </div>
              <div text-xs>
                {{ TomodoroData.isWorking ? TomodoroData.workDuration : TomodoroData.breakDuration }}
              </div>
            </div>
          </div>
        </div>

        <div>
          <TransitionGroup name="buttonList">
            <button
              key="start"
              class="m-3 text-sm btn"
              @click="start"
            >
              Go
            </button>

            <button
              v-if="!TomodoroData.isPaused"

              key="pause"
              class="m-3 bg-gray-300 text-sm btn"
              @click="TomodoroData.isPaused = true"
            >
              Pause
            </button>

            <button
              key="pip"
              class="m-3 bg-gray-300 text-sm btn"
              @click="togglePiP"
            >
              PiP
            </button>
          </TransitionGroup>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.buttonList-move, /* 对移动中的元素应用的过渡 */
.buttonList-enter-active,
.buttonList-leave-active {
  transition: all 0.5s ease;
}
.buttonList-enter-from,
.buttonList-leave-to {
  opacity: 0;
}

/* 确保将离开的元素从布局流中删除
  以便能够正确地计算移动的动画。 */
  .buttonList-leave-active {
  position: absolute;
}

#timerContainer:not(:has(*))::before {
  content: "Tomodoro is playing in Picture-in-Picture.";
  color: #cbd5e1;
  line-height: 360px;
  text-align: center;
}
</style>
