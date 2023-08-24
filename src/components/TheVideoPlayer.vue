<script setup lang="ts" generic="T extends any, O extends any">
import 'video.js/dist/video-js.min.css'
import 'video.js/dist/video.min.js'
import videojs from 'video.js'

defineOptions({
  name: 'TheVideoPlayer',
})

const { log } = console
const videoPlayer = ref<Element>()
const playerContainer = ref<Element>()

const data = reactive({
  player: null as any,
  controls: null as any,
  options: {
    autoplay: true,
    controls: true,
    sources: [{
      src: '//vjs.zencdn.net/v/oceans.mp4',
      type: 'video/mp4',
    }],
  },
})

/**
 * @see https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement/requestPictureInPicture
 */
// function togglePictureInPicture() {
//   if (!data.player)
//     return
//   if (data.player.isInPictureInPicture())
//     data.player.exitPictureInPicture()
//   else
//     data.player.requestPictureInPicture()
// }

async function togglePictureInPicture() {
  if (window.documentPictureInPicture.window) {
    window.documentPictureInPicture.window.close()
    return
  }

  // Open a Picture-in-Picture window.
  const pipWindow = window.documentPictureInPicture.requestWindow({
    width: 640,
    height: 360,
  }).then((pipWindow: any) => {
    // Make sure the pipWindow object is available
    if (pipWindow && pipWindow.document) {
      // Copy the stylesheets from the main document to the Picture-in-Picture window.
      [...document.styleSheets].forEach((styleSheet) => {
        try {
          const cssRules = [...styleSheet.cssRules].map(rule => rule.cssText).join('')
          const style = document.createElement('style')

          style.textContent = cssRules
          pipWindow.document.head.appendChild(style)
        }
        catch (e) {
          const link = document.createElement('link')

          link.rel = 'stylesheet'
          link.type = styleSheet.type
          link.media = styleSheet.media
          link.href = styleSheet.href
          pipWindow.document.head.appendChild(link)
        }
      })

      const video = document.querySelector('#video')

      // Move video to the Picture-in-Picture window and make it full page.
      pipWindow.document.body.append(video)

      video?.classList.toggle('fullpage', true)

      // Listen for the PiP closing event to move the video back.
      pipWindow.addEventListener('pagehide', (event: any) => {
        const videoContainer = document.querySelector('#videoContainer')
        const pipVideo = event.target.querySelector('#video')
        pipVideo.classList.toggle('fullpage', false)
        if (!videoContainer)
          return
        videoContainer.append(pipVideo)
      })
    }
    else {
      console.error('Picture-in-Picture window or document is not available.')
    }
  })
    .catch((error: any) => {
      console.error('Failed to open Picture-in-Picture window:', error)
    })
}

onMounted(() => {
  if (!videoPlayer.value)
    return
  data.player = videojs(videoPlayer.value, data.options)

  const controls = videojs(videoPlayer.value, { controls: true }).getChild('ControlBar')
  if (!controls)
    return
  const fullscreenToggle = controls.getChild('FullscreenToggle')
  if (!fullscreenToggle)
    return
  controls.removeChild(fullscreenToggle)
  const pictureInPictureToggle = controls.getChild('PictureInPictureToggle')
  if (!pictureInPictureToggle)
    return
  controls.removeChild(pictureInPictureToggle)
  controls.addChild('button', {
    controlText: 'Toggle Picture-in-Picture',
    className: 'vjs-visible-text',
    clickHandler: togglePictureInPicture,
  })

  data.controls = controls
})
</script>

<template>
  <div id="videoContainer" ref="playerContainer">
    <video
      id="video" ref="videoPlayer" class="video-js"
      enableDocumentPictureInPicture="true"
    />
  </div>
</template>

<style scoped>
  #videoContainer {
  height: 100%;
  display: grid;
  place-content: center;
}
.video-js {
  width: 640px;
  height: 360px;
  max-width: 100vw;
}
.fullpage {
  height: 100% !important;
  width: 100% !important;
}
#videoContainer:not(:has(*))::before {
  content: "The video is playing in Picture-in-Picture.";
  color: #39e75f;
  line-height: 360px;
  text-align: center;
}
#videoContainer::after {
  content: "© copyright Rio Huang";
  color: #cdcdcd;
  text-align: center;
  margin-top: 18px;
}
</style>
