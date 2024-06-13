<script setup lang="ts">
import { ref, onMounted, computed, watch } from "vue"
import { ContextLayerSetFunction, StayCanvasProps } from "./types"
import { Dict } from "./userTypes"
import StayStage from "./stay/stayStage"
import * as PredefinedEventList from "./predefinedEvents"

let userTrigger: (name: string, payload: Dict) => void | undefined

const customTrigger = (stay: StayStage, name: string, payload: Dict) => {
  const customEvent = new Event(name)
  stay.triggerAction(customEvent, { [name]: customEvent }, payload)
}

const trigger = (name: string, payload?: Dict) => {
  if (userTrigger) {
    userTrigger(name, payload || {})
  }
}

defineExpose({
  trigger,
})

const props = withDefaults(defineProps<StayCanvasProps>(), {
  width: 500,
  height: 500,
  layers: 2,
  eventList: () => [],
  listenerList: () => [],
})

const canvasLayers = ref<HTMLCanvasElement[]>([])
const stay = ref<StayStage>()

const layers = computed(() => {
  let contextLayerSetFunctionList: ContextLayerSetFunction[] = []

  if (typeof props.layers === "number") {
    Array(props.layers)
      .fill(0)
      .forEach(() => {
        contextLayerSetFunctionList.push((canvas: HTMLCanvasElement) => {
          return canvas.getContext("2d")
        })
      })
  } else {
    contextLayerSetFunctionList = props.layers
  }

  if (contextLayerSetFunctionList.length < 1) {
    throw new Error("layers must be greater than 0")
  }
  return contextLayerSetFunctionList
})

watch(
  () => props.eventList,
  (currentEventList) => {
    if (!stay.value || !currentEventList) {
      return
    }
    stay.value.clearEvents()
    ;[...Object.values(PredefinedEventList), ...currentEventList].forEach((event) => {
      stay.value!.registerEvent(event)
    })
  },
  {
    immediate: true, // Run this watcher immediately on component mount
  }
)

watch(
  () => props.listenerList,
  (currentListenerList) => {
    if (!stay.value || !currentListenerList) {
      return
    }

    stay.value.clearEventListeners()
    currentListenerList.forEach((listener) => {
      stay.value!.addEventListener(listener)
    })
  },
  {
    immediate: true, // Run this watcher immediately on component mount
  }
)

onMounted(() => {
  stay.value = new StayStage(canvasLayers.value, layers.value, props.width, props.height)
  ;[...props.eventList!.values(), ...Object.values(PredefinedEventList)].forEach((event) => {
    stay.value!.registerEvent(event)
  })
  ;[...props.listenerList!.values()].forEach((listener) => {
    stay.value!.addEventListener(listener)
  })

  userTrigger = (name: string, payload: Dict) => {
    if (stay) {
      customTrigger(stay.value!, name, payload)
    }
  }

  if (props.mounted) {
    props.mounted(stay.value!.tools)
    stay.value!.draw()
  }
})

// watch
</script>
<template>
  <div
    :style="{
      display: 'flex',
      position: 'relative',
      justifyContent: 'center',
      alignItems: 'center',
      width: `${props.width}px`,
      height: `${props.height}px`,
    }"
  >
    <canvas
      ref="canvasLayers"
      v-for="(_, i) in layers"
      :key="i"
      :width="props.width"
      :height="props.height"
      style="position: absolute; left: 0; top: 0"
      tabindex="1"
      class="canvas"
    ></canvas>
  </div>
</template>
