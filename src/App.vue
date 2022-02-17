<template>
    <Lunchbox
        :cameraPosition="[0, 0, 10]"
        :cameraLook="[0, 0, 0]"
        background="black"
        orthographic
    >
        <group :scale="2">
            <mesh
                :scale="0.25 * Math.sin(now * 0.001 + i * 0.5)"
                v-for="(shape, i) in shapes"
                :key="i"
                :position-x="(i % columns) - columns * 0.5 + 0.5"
                :position-y="Math.floor(i / columns) - rows * 0.5 + 0.5"
                :rotation-y="Math.sin(now * 0.001 + i) * Math.PI"
            >
                <component :is="shape" />
                <meshBasicMaterial color="black" />
            </mesh>
        </group>
    </Lunchbox>
</template>

<script setup lang="ts">
import * as THREE from 'three'
import { onBeforeRender, setCustomRender } from 'lunchboxjs'
import { ref } from 'vue'

// config
const pool = [
    'boxGeometry',
    'tetrahedronGeometry',
    'icosahedronGeometry',
    'torusGeometry',
    'torusKnotGeometry',
]
const count = 80
const columns = 10
const rows = Math.ceil(count / columns)

// shapes
const shapes = ref(
    new Array(count)
        .fill(undefined)
        .map(() => pool[Math.floor(Math.random() * pool.length)])
)

// time
const now = ref(Date.now())
onBeforeRender(() => (now.value = Date.now()))

// postprocessing
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer'
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass'
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass'

let composer: EffectComposer | null = null

setCustomRender((opts) => {
    const canvas = opts.renderer?.domElement

    // ignore if no canvas
    if (!canvas || !opts.scene || !opts.camera) return

    // setup effect composer if needed
    if (!composer) {
        composer = new EffectComposer(opts.renderer as THREE.WebGLRenderer)

        const renderPass = new RenderPass(opts.scene, opts.camera)
        composer.addPass(renderPass)

        const outlinePass = new OutlinePass(
            new THREE.Vector2(canvas.width, canvas.height),
            opts.scene,
            opts.camera,
            opts.scene.children
        )
        outlinePass.edgeThickness = 1
        outlinePass.edgeGlow = 0
        composer.addPass(outlinePass)
    }

    composer.render()
})
</script>