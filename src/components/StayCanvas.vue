<script setup lang="ts">
import { ref, onMounted, computed } from "vue"
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
    trigger
})

const props = withDefaults(defineProps<StayCanvasProps>(), {
    width: 500,
    height: 500,
    layers: 2,
})

const canvasLayers = ref<HTMLCanvasElement[]>([])

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

const eventList = computed(() => {
    return props.eventList || []
})

const listenerList = computed(() => {
    return props.listenerList || []
})

onMounted(() => {
    const stay = new StayStage(canvasLayers.value, layers.value, 600, 600)

        ;[...eventList.value, ...Object.values(PredefinedEventList)].forEach((event) => {
            stay.registerEvent(event)
        })
    listenerList.value.forEach((listener) => {
        stay.addEventListener(listener)
    })

    userTrigger = (name: string, payload: Dict) => {
        if (stay) {
            customTrigger(stay, name, payload)
        }
    }

    if (props.mounted) {
        props.mounted(stay.tools)
        stay.draw()
    }
})
</script>
<template>
    <div class="container">
        <canvas ref="canvasLayers" v-for="(_, i) in layers" :key="i" :width="props.width" :height="props.height"
            tabindex="1" class="canvas"></canvas>
    </div>
</template>
<style>
.canvas {
    position: absolute;
    left: 0;
    top: 0;
}

.container {
    display: flex;
    position: relative;
    justify-content: center;
    align-items: center;
}
</style>
