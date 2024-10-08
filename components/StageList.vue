<template>
  <div id="stages" ref="container" :class="['stages', {entered: isInHtml}]">

    <stage-mask :elRef="el => mask = el" />

    <div ref="list" class="stages__list">
      <div ref="listInner" class="stages__list-inner">
        <Suspense>
          <template #default>
            <StageClients :active="isInHtml && stageHtmlIndex === 0" @back-webgl="handlePrev" @next-html="stageHtmlIndex += 1" ref="target"/>
          </template>
          <template #fallback>
            <div>Chargement...</div>
          </template>
        </Suspense>
        <Suspense>
          <template #default>
            <StageAbout2 :active="isInHtml && stageHtmlIndex === 1" @back-html="stageHtmlIndex -= 1" @next-html="stageHtmlIndex += 1" />
          </template>
          <template #fallback>
            <div>Chargement...</div>
          </template>
        </Suspense>
        <Suspense>
          <template #default>
            <StageContact :active="isInHtml && stageHtmlIndex === 2" @back-html="stageHtmlIndex -= 1" @next-html="stageHtmlIndex += 1" />
          </template>
          <template #fallback>
            <div>Chargement...</div>
          </template>
        </Suspense>
      </div>
    </div>

  </div>
</template>

<script setup>
import { defineAsyncComponent, ref, onMounted, watch } from 'vue'
import gsap from "gsap"
import emitter from "tiny-emitter/instance"
import { STAGE_HTML_PREV } from "~/assets/js/utils/events"
import { useIsInHtml, useStageHtmlIndex } from "~/composables/stages"
import { useIntersectionObserver } from '@vueuse/core'

// Charger les composants de manière asynchrone
const StageClients = defineAsyncComponent(() => import('./StageClients.vue'))
const StageAbout2 = defineAsyncComponent(() => import('./StageAbout2.vue'))
const StageContact = defineAsyncComponent(() => import('./StageContact.vue'))

// states
const isInHtml = useIsInHtml()
const stageHtmlIndex = useStageHtmlIndex()
const container = ref(null)
const mask = ref(null)
const list = ref(null)
const listInner = ref(null)

const target = ref(null)
const targetIsVisible = ref(false)

// methods
const { stop } = useIntersectionObserver(
  target,
  ([{ isIntersecting }], observerElement) => {
    targetIsVisible.value = isIntersecting
  },
)

const handlePrev = () => {
  emitter.emit(STAGE_HTML_PREV)
}

const handleEnter = (index) => {
  console.log("enter", index)

}

const handleLeave = (index) => {
  console.log("leave", index)

  gsap.fromTo(mask.value, {
    scale: 0,
  }, {
    scale: 1,
    duration: 1,
    ease: "power3.inOut",
    clearProps: true,
  })

}

const handleAnimation = (nextIndex, prevIndex) => {

  console.log("animation", nextIndex, prevIndex)
  
  const tl = gsap.timeline()
  
  // if(prevIndex !== null){
    tl.fromTo(mask.value, {
      autoAlpha: 1,
      scale: 0,
    }, {
      autoAlpha: 1,
      scale: 1,
      duration: 1,
      ease: "power3.inOut",
      // clearProps: true,
    })
  // } else {
    
  // }

  tl.to(mask.value, {
    autoAlpha: 0,
    duration: 1,
    ease: "power2.out"
  })
}

// hook
onMounted(() => {
  gsap.set(mask.value, {scale: 0})
  gsap.set(list.value, {background: 'none'})

  window.gsap = gsap
})

onUnmounted(() => {
  stop() // Arrêter l'observateur lorsque le composant est démonté
})

watch([isInHtml, stageHtmlIndex], ([nextHtml, nextIndex], [prevHtml, prevIndex]) => {

  if(isInHtml.value){
    handleAnimation(nextIndex, prevIndex !== nextIndex ? prevIndex : null)
  }

  if(nextHtml && !prevHtml){
    // enter
 
    gsap.set(list.value, {background: '', delay: 1})

    gsap.fromTo(listInner.value, {
      autoAlpha: 0,
    }, {
      autoAlpha: 1,
      delay: 1,
      duration: 0.5,
      ease: "power2.out"
    })
  }
  else if(!nextHtml && prevHtml){
    gsap.set(list.value, {background: 'none'})
  }
})


</script>

<style lang="scss" scoped>
.stages{
  position: relative;
  z-index: 1;
  width: 100%;
  height: var(--100sh);
  background: radial-gradient(63.3% 103.82% at 50% 50%, #FFFFFF 14.16%, #414141 100%);

  &__list{
    background-color: #000;
  }

  &:not(.entered){
    pointer-events: none;

    ::v-deep(*){
     pointer-events: none!important;
     cursor: auto!important;
    }
  }
}
</style>