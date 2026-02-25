<template>
  <div class="w-full h-full flex flex-col">
    <!-- Main layout: canvas + resizable sidebar -->
    <div ref="layoutContainerRef" class="flex flex-1 overflow-hidden">
      <!-- 3D canvas area -->
      <div class="flex-1 min-w-[300px] bg-base-300 relative">
        <canvas ref="canvasRef" class="w-full h-full block"></canvas>
      </div>

      <!-- Vertical resizer -->
      <div
        ref="resizerRef"
        class="w-1 cursor-col-resize bg-base-200 hover:bg-primary transition-colors"
        @mousedown="onResizerMouseDown"
      ></div>

      <!-- Sidebar -->
      <aside
        ref="sidebarRef"
        class="bg-base-100 border-l border-base-300 flex-shrink-0 overflow-y-auto"
        :style="{ width: sidebarWidth + 'px' }"
      >
        <div class="p-4 space-y-4">
          <!-- Control Panel -->
          <div class="collapse collapse-arrow bg-base-200">
            <input type="checkbox" checked />
            <div class="collapse-title text-lg font-semibold">Control Panel</div>
            <div class="collapse-content space-y-3">
              <div class="form-control">
                <label class="label">
                  <span class="label-text">Mass m (kg)</span>
                </label>
                <input
                  type="number"
                  class="input input-sm input-bordered w-full"
                  v-model.number="controls.mass"
                  min="0.01"
                  step="0.1"
                />
              </div>

              <div class="form-control">
                <label class="label">
                  <span class="label-text">Initial height h₀ (m)</span>
                </label>
                <input
                  type="number"
                  class="input input-sm input-bordered w-full"
                  v-model.number="controls.initialHeight"
                  min="0.1"
                  step="0.1"
                />
              </div>

              <div class="form-control">
                <label class="label">
                  <span class="label-text">Initial velocity v₀ (m/s, upward +)</span>
                </label>
                <input
                  type="number"
                  class="input input-sm input-bordered w-full"
                  v-model.number="controls.initialVelocity"
                  step="0.1"
                />
              </div>

              <div class="form-control">
                <label class="label">
                  <span class="label-text">Linear drag coefficient b (kg/s)</span>
                </label>
                <input
                  type="number"
                  class="input input-sm input-bordered w-full"
                  v-model.number="controls.dragCoefficient"
                  min="0"
                  step="0.1"
                />
              </div>

              <div class="flex gap-2 pt-2">
                <button
                  class="btn btn-sm btn-primary flex-1"
                  @click="onStartRestart"
                  :disabled="isRunning"
                >
                  {{ hasStarted ? 'Restart Simulation' : 'Start Simulation' }}
                </button>
                <button
                  class="btn btn-sm btn-secondary flex-1"
                  @click="onPauseResume"
                  :disabled="!hasStarted"
                >
                  {{ isRunning ? 'Pause' : 'Resume' }}
                </button>
              </div>
            </div>
          </div>

          <!-- Live Data Panel -->
          <div class="collapse collapse-arrow bg-base-200">
            <input type="checkbox" checked />
            <div class="collapse-title text-lg font-semibold">Live Data</div>
            <div class="collapse-content space-y-2">
              <div class="grid grid-cols-2 gap-2 text-sm">
                <div class="font-medium">Time t (s)</div>
                <div>{{ liveData.time.toFixed(3) }}</div>

                <div class="font-medium">Height y (m)</div>
                <div>{{ liveData.position.toFixed(3) }}</div>

                <div class="font-medium">Velocity v (m/s)</div>
                <div>{{ liveData.velocity.toFixed(3) }}</div>

                <div class="font-medium">Kinetic Energy K (J)</div>
                <div>{{ liveData.kineticEnergy.toFixed(3) }}</div>

                <div class="font-medium">Potential Energy U (J)</div>
                <div>{{ liveData.potentialEnergy.toFixed(3) }}</div>

                <div class="font-medium">Total Energy E (J)</div>
                <div>{{ liveData.totalEnergy.toFixed(3) }}</div>
              </div>

              <button
                class="btn btn-sm btn-outline mt-2"
                @click="exportLogsToCSV"
                :disabled="dataLog.length === 0"
              >
                Export Simulation Data (CSV)
              </button>
            </div>
          </div>

          <!-- Chart Panel -->
          <div class="collapse collapse-arrow bg-base-200">
            <input type="checkbox" checked />
            <div class="collapse-title text-lg font-semibold">Chart Panel</div>
            <div class="collapse-content space-y-3">
              <p class="text-sm">
                Charts: Height y(t) and Velocity v(t) vs Time.
              </p>
              <div class="space-x-2">
                <button
                  class="btn btn-sm btn-primary"
                  @click="plotCharts"
                  :disabled="dataLog.length === 0"
                >
                  Plot Chart
                </button>
                <button
                  class="btn btn-sm btn-outline"
                  @click="exportChartDataToCSV"
                  :disabled="!hasChartData"
                >
                  Export Chart Data (CSV)
                </button>
              </div>
              <div class="space-y-4 pt-2">
                <div>
                  <h3 class="font-semibold text-sm mb-1">
                    Height vs Time (y(t))
                  </h3>
                  <canvas ref="heightChartRef" class="w-full h-40"></canvas>
                </div>
                <div>
                  <h3 class="font-semibold text-sm mb-1">
                    Velocity vs Time (v(t))
                  </h3>
                  <canvas ref="velocityChartRef" class="w-full h-40"></canvas>
                </div>
              </div>
            </div>
          </div>
        </div>
      </aside>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import * as CANNON from 'cannon-es'
import {
  Chart,
  ChartData,
  ChartOptions,
  registerables as chartRegisterables,
} from 'chart.js'

Chart.register(...chartRegisterables)

// Layout & DOM refs
const canvasRef = ref<HTMLCanvasElement | null>(null)
const layoutContainerRef = ref<HTMLDivElement | null>(null)
const sidebarRef = ref<HTMLElement | null>(null)
const resizerRef = ref<HTMLDivElement | null>(null)
const heightChartRef = ref<HTMLCanvasElement | null>(null)
const velocityChartRef = ref<HTMLCanvasElement | null>(null)

// Sidebar sizing state
const sidebarWidth = ref(320) // initial width
const isResizing = ref(false)
const minSidebarWidth = 260
const maxSidebarWidthStatic = 600
const minCanvasWidth = 300

let resizeMouseMoveHandler: ((e: MouseEvent) => void) | null = null
let resizeMouseUpHandler: ((e: MouseEvent) => void) | null = null

// Simulation control state
const GRAVITY = 9.81 // m/s^2

const controls = reactive({
  mass: 1.0,
  initialHeight: 10.0,
  initialVelocity: 0.0,
  dragCoefficient: 0.5, // b in F_d = -b v
})

const liveData = reactive({
  time: 0,
  position: 0,
  velocity: 0,
  kineticEnergy: 0,
  potentialEnergy: 0,
  totalEnergy: 0,
})

const isRunning = ref(false)
const hasStarted = ref(false)

// Three.js
let scene: THREE.Scene | null = null
let camera: THREE.PerspectiveCamera | null = null
let renderer: THREE.WebGLRenderer | null = null
let orbitControls: OrbitControls | null = null
let sphereMesh: THREE.Mesh | null = null
let groundMesh: THREE.Mesh | null = null

// Cannon-es
let world: CANNON.World | null = null
let sphereBody: CANNON.Body | null = null
let lastTime = 0
let animationFrameId: number | null = null

// Data logging
interface LogEntry {
  t: number
  y: number
  v: number
  K: number
  U: number
  E: number
}

const dataLog = ref<LogEntry[]>([])

// Chart.js
let heightChart: Chart | null = null
let velocityChart: Chart | null = null
const hasChartData = ref(false)

/**
 * Initialize Three.js scene, camera, renderer, and meshes.
 */
function initThree() {
  if (!canvasRef.value) return

  const width = canvasRef.value.clientWidth || canvasRef.value.offsetWidth || 800
  const height = canvasRef.value.clientHeight || canvasRef.value.offsetHeight || 600

  scene = new THREE.Scene()
  scene.background = new THREE.Color('#0f172a') // base-300 like

  camera = new THREE.PerspectiveCamera(60, width / height, 0.1, 1000)
  camera.position.set(10, 10, 10)
  camera.lookAt(0, 0, 0)

  renderer = new THREE.WebGLRenderer({ canvas: canvasRef.value, antialias: true })
  renderer.setSize(width, height)
  renderer.setPixelRatio(window.devicePixelRatio || 1)

  // Lights
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.6)
  scene.add(ambientLight)

  const dirLight = new THREE.DirectionalLight(0xffffff, 0.8)
  dirLight.position.set(10, 20, 10)
  scene.add(dirLight)

  // Ground
  const groundGeo = new THREE.PlaneGeometry(20, 20)
  const groundMat = new THREE.MeshStandardMaterial({ color: 0x1f2937, side: THREE.DoubleSide })
  groundMesh = new THREE.Mesh(groundGeo, groundMat)
  groundMesh.rotation.x = -Math.PI / 2
  groundMesh.position.y = 0
  scene.add(groundMesh)

  // Sphere representing the falling body
  const sphereGeo = new THREE.SphereGeometry(0.3, 32, 32)
  const sphereMat = new THREE.MeshStandardMaterial({ color: 0x3b82f6 })
  sphereMesh = new THREE.Mesh(sphereGeo, sphereMat)
  sphereMesh.position.set(0, controls.initialHeight, 0)
  scene.add(sphereMesh)

  // Orbit controls
  if (camera && canvasRef.value) {
    orbitControls = new OrbitControls(camera, canvasRef.value)
    orbitControls.enableDamping = true
    orbitControls.dampingFactor = 0.05
    orbitControls.target.set(0, controls.initialHeight / 2, 0)
  }
}

/**
 * Initialize Cannon-es physics world and bodies.
 */
function initPhysics() {
  world = new CANNON.World({
    gravity: new CANNON.Vec3(0, -GRAVITY, 0),
  })
  world.broadphase = new CANNON.NaiveBroadphase()
  world.solver.iterations = 10

  // Ground body
  const groundShape = new CANNON.Plane()
  const groundBody = new CANNON.Body({
    mass: 0,
    shape: groundShape,
  })
  // Rotate so plane is horizontal
  groundBody.quaternion.setFromEuler(-Math.PI / 2, 0, 0)
  world.addBody(groundBody)

  // Sphere body
  const radius = 0.3
  const sphereShape = new CANNON.Sphere(radius)
  sphereBody = new CANNON.Body({
    mass: controls.mass,
    shape: sphereShape,
    position: new CANNON.Vec3(0, controls.initialHeight, 0),
    velocity: new CANNON.Vec3(0, controls.initialVelocity, 0),
    linearDamping: 0, // we will apply linear drag manually
  })
  world.addBody(sphereBody)

  lastTime = performance.now() / 1000
}

/**
 * Reset simulation to initial state.
 */
function resetSimulation() {
  clearLogs()
  hasChartData.value = false

  if (sphereMesh) {
    sphereMesh.position.set(0, controls.initialHeight, 0)
  }

  if (sphereBody) {
    sphereBody.position.set(0, controls.initialHeight, 0)
    sphereBody.velocity.set(0, controls.initialVelocity, 0)
    sphereBody.angularVelocity.setZero()
    sphereBody.quaternion.set(0, 0, 0, 1)
  }

  liveData.time = 0
  liveData.position = controls.initialHeight
  liveData.velocity = controls.initialVelocity
  updateEnergies()

  lastTime = performance.now() / 1000
}

/**
 * Step physics simulation forward with fixed time step.
 */
function stepSimulation(dt: number) {
  if (!world || !sphereBody) return

  const fixedTimeStep = 1 / 60
  const maxSubSteps = 5

  // Apply linear drag force F_d = -b v_y
  const vy = sphereBody.velocity.y
  const dragForceY = -controls.dragCoefficient * vy
  const dragForce = new CANNON.Vec3(0, dragForceY, 0)
  sphereBody.applyForce(dragForce, sphereBody.position)

  world.step(fixedTimeStep, dt, maxSubSteps)

  // Stop condition: when sphere hits the ground
  if (sphereBody.position.y <= 0) {
    sphereBody.position.y = 0
    isRunning.value = false
  }

  // Update live data
  liveData.time += fixedTimeStep
  liveData.position = sphereBody.position.y
  liveData.velocity = sphereBody.velocity.y
  updateEnergies()

  // Log frame
  logFrame()
}

/**
 * Update energy values in liveData.
 */
function updateEnergies() {
  const m = controls.mass
  const y = liveData.position
  const v = liveData.velocity

  const K = 0.5 * m * v * v
  const U = m * GRAVITY * Math.max(y, 0)
  liveData.kineticEnergy = K
  liveData.potentialEnergy = U
  liveData.totalEnergy = K + U
}

/**
 * Log current frame data.
 */
function logFrame() {
  dataLog.value.push({
    t: liveData.time,
    y: liveData.position,
    v: liveData.velocity,
    K: liveData.kineticEnergy,
    U: liveData.potentialEnergy,
    E: liveData.totalEnergy,
  })
}

/**
 * Clear simulation logs.
 */
function clearLogs() {
  dataLog.value = []
}

/**
 * Export full simulation log to CSV.
 */
function exportLogsToCSV() {
  if (dataLog.value.length === 0) return

  const header = ['t (s)', 'y (m)', 'v (m/s)', 'K (J)', 'U (J)', 'E (J)']
  const rows = dataLog.value.map((entry) => [
    entry.t,
    entry.y,
    entry.v,
    entry.K,
    entry.U,
    entry.E,
  ])

  const csvContent =
    [header, ...rows]
      .map((row) => row.map((v) => String(v)).join(','))
      .join('\n')

  downloadCSV(csvContent, 'free_fall_simulation_data.csv')
}

/**
 * Export chart data (what's plotted) to CSV.
 * Here we export t, y(t), v(t) from dataLog.
 */
function exportChartDataToCSV() {
  if (dataLog.value.length === 0) return

  const header = ['t (s)', 'y (m)', 'v (m/s)']
  const rows = dataLog.value.map((entry) => [entry.t, entry.y, entry.v])

  const csvContent =
    [header, ...rows]
      .map((row) => row.map((v) => String(v)).join(','))
      .join('\n')

  downloadCSV(csvContent, 'free_fall_chart_data.csv')
}

/**
 * Trigger browser download of CSV content.
 */
function downloadCSV(content: string, filename: string) {
  const blob = new Blob([content], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.setAttribute('download', filename)
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)
}

/**
 * Initialize Chart.js charts.
 */
function initCharts() {
  if (!heightChartRef.value || !velocityChartRef.value) return

  const commonOptions: ChartOptions<'line'> = {
    responsive: true,
    maintainAspectRatio: false,
    plugins: {
      legend: {
        labels: {
          font: { size: 11 },
        },
      },
    },
    scales: {
      x: {
        title: {
          display: true,
          text: 'Time t (s)',
        },
      },
    },
  }

  const heightData: ChartData<'line'> = {
    labels: [],
    datasets: [
      {
        label: 'Height y (m)',
        data: [],
        borderColor: 'rgb(59, 130, 246)', // blue-500
        backgroundColor: 'rgba(59, 130, 246, 0.3)',
        tension: 0.1,
        pointRadius: 0,
      },
    ],
  }

  const velocityData: ChartData<'line'> = {
    labels: [],
    datasets: [
      {
        label: 'Velocity v (m/s)',
        data: [],
        borderColor: 'rgb(248, 113, 113)', // red-400
        backgroundColor: 'rgba(248, 113, 113, 0.3)',
        tension: 0.1,
        pointRadius: 0,
      },
    ],
  }

  heightChart = new Chart(heightChartRef.value, {
    type: 'line',
    data: heightData,
    options: {
      ...commonOptions,
      scales: {
        ...commonOptions.scales,
        y: {
          title: {
            display: true,
            text: 'Height y (m)',
          },
        },
      },
    },
  })

  velocityChart = new Chart(velocityChartRef.value, {
    type: 'line',
    data: velocityData,
    options: {
      ...commonOptions,
      scales: {
        ...commonOptions.scales,
        y: {
          title: {
            display: true,
            text: 'Velocity v (m/s)',
          },
        },
      },
    },
  })
}

/**
 * Plot charts based on current dataLog.
 */
function plotCharts() {
  if (!heightChart || !velocityChart || dataLog.value.length === 0) return

  const labels = dataLog.value.map((e) => e.t)

  heightChart.data.labels = labels
  heightChart.data.datasets[0].data = dataLog.value.map((e) => e.y)
  heightChart.update()

  velocityChart.data.labels = labels
  velocityChart.data.datasets[0].data = dataLog.value.map((e) => e.v)
  velocityChart.update()

  hasChartData.value = true
}

/**
 * Main animation loop.
 */
function animate() {
  if (!renderer || !scene || !camera) return

  animationFrameId = requestAnimationFrame(animate)

  const now = performance.now() / 1000
  const dt = now - lastTime
  lastTime = now

  if (isRunning.value) {
    stepSimulation(dt)
  }

  // Sync Three.js mesh with physics body
  if (sphereMesh && sphereBody) {
    sphereMesh.position.copy(
      new THREE.Vector3(
        sphereBody.position.x,
        sphereBody.position.y,
        sphereBody.position.z,
      ),
    )
  }

  if (orbitControls) {
    orbitControls.update()
  }

  renderer.render(scene, camera)
}

/**
 * Handle window resize: adjust renderer and camera, and also clamp sidebar width.
 */
function handleWindowResize() {
  if (!canvasRef.value || !renderer || !camera || !layoutContainerRef.value) return

  const containerWidth =
    layoutContainerRef.value.clientWidth || layoutContainerRef.value.offsetWidth || window.innerWidth
  const containerHeight =
    layoutContainerRef.value.clientHeight || layoutContainerRef.value.offsetHeight || window.innerHeight

  // Clamp sidebar width to valid range
  const maxSidebarWidthDynamic =
    Math.min(maxSidebarWidthStatic, containerWidth - minCanvasWidth - 4) // 4 ~ resizer width
  const clampedSidebarWidth = Math.max(
    minSidebarWidth,
    Math.min(sidebarWidth.value, maxSidebarWidthDynamic),
  )
  sidebarWidth.value = clampedSidebarWidth

  const canvasWidth = containerWidth - sidebarWidth.value - (resizerRef.value ? resizerRef.value.offsetWidth : 4)
  const canvasHeight = containerHeight

  renderer.setSize(canvasWidth, canvasHeight)
  camera.aspect = canvasWidth / canvasHeight
  camera.updateProjectionMatrix()
}

/**
 * Mouse down handler to start sidebar resizing.
 */
function onResizerMouseDown(e: MouseEvent) {
  e.preventDefault()
  if (!layoutContainerRef.value) return

  isResizing.value = true

  const container = layoutContainerRef.value
  const containerRect = container.getBoundingClientRect()

  resizeMouseMoveHandler = (event: MouseEvent) => {
    if (!isResizing.value || !layoutContainerRef.value) return
    const totalWidth = containerRect.width
    const resizerWidth = resizerRef.value ? resizerRef.value.offsetWidth : 4

    // Sidebar is on the right: width = distance from cursor to right edge minus half resizer
    const distanceFromLeft = event.clientX - containerRect.left
    const newSidebarWidth = totalWidth - distanceFromLeft - resizerWidth / 2

    const maxSidebarWidthDynamic =
      Math.min(maxSidebarWidthStatic, totalWidth - minCanvasWidth - resizerWidth)
    sidebarWidth.value = Math.max(
      minSidebarWidth,
      Math.min(newSidebarWidth, maxSidebarWidthDynamic),
    )

    handleWindowResize()
  }

  resizeMouseUpHandler = () => {
    isResizing.value = false
    if (resizeMouseMoveHandler) {
      window.removeEventListener('mousemove', resizeMouseMoveHandler)
    }
    if (resizeMouseUpHandler) {
      window.removeEventListener('mouseup', resizeMouseUpHandler)
    }
  }

  window.addEventListener('mousemove', resizeMouseMoveHandler)
  window.addEventListener('mouseup', resizeMouseUpHandler)
}

/**
 * Start or restart the simulation.
 */
function onStartRestart() {
  if (!world || !sphereBody) {
    initPhysics()
  }
  resetSimulation()
  isRunning.value = true
  hasStarted.value = true
}

/**
 * Pause or resume the simulation.
 */
function onPauseResume() {
  if (!hasStarted.value) return
  isRunning.value = !isRunning.value
}

/**
 * Cleanup Three.js, Cannon, and event listeners.
 */
function cleanup() {
  if (animationFrameId !== null) {
    cancelAnimationFrame(animationFrameId)
    animationFrameId = null
  }

  if (renderer) {
    renderer.dispose()
    renderer = null
  }

  if (orbitControls) {
    orbitControls.dispose()
    orbitControls = null
  }

  if (scene && sphereMesh) {
    scene.remove(sphereMesh)
    sphereMesh.geometry.dispose()
    if (Array.isArray(sphereMesh.material)) {
      sphereMesh.material.forEach((m) => m.dispose())
    } else {
      sphereMesh.material.dispose()
    }
    sphereMesh = null
  }

  if (scene && groundMesh) {
    scene.remove(groundMesh)
    groundMesh.geometry.dispose()
    if (Array.isArray(groundMesh.material)) {
      groundMesh.material.forEach((m) => m.dispose())
    } else {
      groundMesh.material.dispose()
    }
    groundMesh = null
  }

  if (world && sphereBody) {
    world.removeBody(sphereBody)
    sphereBody = null
  }
  world = null

  if (heightChart) {
    heightChart.destroy()
    heightChart = null
  }
  if (velocityChart) {
    velocityChart.destroy()
    velocityChart = null
  }

  window.removeEventListener('resize', handleWindowResize)
}

onMounted(() => {
  if (typeof window === 'undefined') return

  initThree()
  initPhysics()
  initCharts()
  handleWindowResize()
  animate()

  window.addEventListener('resize', handleWindowResize)
})

onBeforeUnmount(() => {
  cleanup()
})
</script>

<style scoped>
/* Ensure the component fills available space when placed in a layout or slot */
:host,
:root,
div {
  box-sizing: border-box;
}

.w-full.h-full.flex.flex-col {
  min-height: 400px;
}

/* Allow chart containers to have a reasonable height */
canvas {
  max-width: 100%;
}
</style>
