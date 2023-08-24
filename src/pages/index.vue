<script setup lang="ts" generic="T extends any, O extends any">
defineOptions({
  name: 'IndexPage',
})

const { log } = console
</script>

<template>
  <div>
    <div i-carbon-campsite inline-block text-4xl />
    <p>
      <a rel="noreferrer" href="https://github.com/antfu/vitesse-lite" target="_blank">
        Vitesse Lite
      </a>
    </p>
    <p>
      <em text-sm op75>This is a Demo for Chrome 116 Document in Picture</em>
    </p>

    <div flex items-center justify-center pt-6>
      <Tomodoro pt-8 />
      <!-- <TheVideoPlayer /> -->
    </div>
  </div>
</template>
