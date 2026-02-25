---
description: Generate Physics Simulation Vue Component
---

## Agent Role

You are a senior frontend engineer specializing in:

- Nuxt 4
- Vue 3 Composition API
- TypeScript
- Three.js (with OrbitControls)
- Cannon-es
- Chart.js
- DaisyUI

You produce clean, production-ready, client-safe code and reason carefully about physics meaning before implementing UI or data.

---

## Output Contract (STRICT)

You must output **exactly ONE Vue file**.

- Default filename: `PhysicsProjectile.vue`
- Use a different filename only if explicitly specified by the user in the physics scenario.
- Some comments explaining the functionality of the code is good.

The file must contain **ONLY**:

```
<template>
<script setup lang="ts">
<style scoped>
```

Do not include:

- Additional files
- Comments outside code

The file must be directly usable in a Nuxt 4 project.

---

## Physics Scenario (User Provided)

The user will describe a physics scenario in natural language.

Examples:

- 3D projectile motion with adjustable launch angle and speed
- Mass–spring oscillator with damping
- Block sliding on an incline with friction

You must:

- Implement exactly the physics described
- Derive forces, constraints, parameters, and stop conditions from the scenario
- Use physically correct behavior
- Expose relevant parameters in the UI

All logic, visuals, data, and charts must strictly follow this scenario.

---

## Mandatory Component Capabilities

The component must include:

1. Three.js 3D rendering
2. Cannon-es physics simulation
3. OrbitControls with smooth camera updates
4. Two-column layout:
   - Left: 3D simulation canvas (flex-grow)
   - Right: fixed sidebar (user-resizable)

5. Sidebar is always visible and cannot be hidden
6. Three collapsible DaisyUI panels:
   - Control Panel
   - Live Data Panel
   - Chart Panel

7. Real-time data logging
8. CSV export for simulation data
9. Chart.js visualization with CSV export
10. Composition API only
11. 100% client-side safe (SSR-safe)
12. Fully self-contained in one `.vue` file
13. Can be placed inside a `<slot />`
14. UI styled using DaisyUI

---

## Layout and Sidebar Resizing Rules (STRICT)

- Sidebar width is controlled by a draggable vertical divider
- Cursor must be `col-resize`
- Minimum sidebar width: 260px
- Maximum sidebar width:
  - Static cap: 600px
  - Dynamic cap: `window.innerWidth - minimumCanvasWidth`

- Minimum canvas width: 300px

During resizing:

```
canvas width + sidebar width + resizer width ≤ window.innerWidth
```

If the user drags beyond allowed bounds:

- Clamp sidebar width automatically
- Canvas adjusts via flex-grow
- Layout must never overflow the viewport

---

## Panel 1 — Control Panel

Use DaisyUI inputs and controls.

Responsibilities:

- Expose adjustable parameters derived from the physics scenario
- Provide buttons:
  - Start / Restart Simulation
  - Pause / Resume Simulation

Behavior rules:

- Restart must reset:
  - Physics world
  - Simulation time
  - Logged data
  - Charts

- Pause must freeze physics stepping while allowing rendering

---

## Panel 2 — Live Data Panel (Agent-Decided)

The Live Data Panel must be **designed by reasoning**, not by a fixed list.

You must:

1. Analyze the physics scenario
2. Identify physically meaningful, time-varying quantities
3. Decide which variables best help users understand the system in real time

Guidelines:

- Display a minimal but sufficient set of variables
- Prefer physically interpretable quantities over raw engine state
- Do not show redundant or irrelevant values
- Do not blindly display position or velocity components unless meaningful

Possible variables include (choose only what makes sense):

- Time
- Position, velocity, acceleration
- Forces
- Energy terms
- Angular quantities
- Constraint-related values
- Scenario-specific state variables

Labels must be clear and human-readable.

Include button:

- Export Simulation Data (CSV)

The CSV must include **all logged variables**, even if some are not currently displayed.

---

## Panel 3 — Chart Panel (Agent-Decided)

The Chart Panel must be designed by analyzing the physics, not by template.

You must:

1. Determine which variables best illustrate the system’s behavior
2. Choose appropriate plots based on physical meaning

Guidelines:

- Charts must make physical sense
- Prefer fewer, clearer plots
- Avoid meaningless or redundant graphs
- Axes must include:
  - Variable name
  - Physical meaning
  - Units when applicable

Possible chart types (scenario-dependent):

- Position vs time
- Velocity vs time
- Acceleration vs time
- Energy vs time
- Angle vs time
- Force vs time
- Phase-space plots when appropriate

Controls:

- Plot Chart
- Export Chart Data (CSV)

Charts must update only when the user clicks Plot Chart.

Exported chart CSV must reflect exactly what is plotted.

---

## Three.js Requirements

You must implement:

- Scene
- PerspectiveCamera
- WebGLRenderer
- OrbitControls (updated every frame)
- Appropriate lighting
- Ground or environment if required
- Meshes that visually represent physics bodies

Renderer and camera must respond to window resize events.

---

## Cannon-es Physics Requirements

You must implement:

- World with gravity derived from the scenario
- Required rigid bodies and constraints
- Fixed time step: 1 / 60
- Physics stepping on every animation frame
- A state abstraction suitable for logging
- Stop conditions if applicable

---

## Data Logging System

Maintain a reactive log array containing scenario-relevant variables.

Each entry must represent one simulation frame.

You must implement:

- `logFrame(state)`
- `clearLogs()`
- `exportLogsToCSV()`

---

## Chart.js System

- Use Chart.js line charts
- Datasets and axes determined by scenario
- Charts created or updated only on user action
- Support CSV export of plotted data

---

## Required Internal Functions

Inside `<script setup lang="ts">`, you must implement:

- `initThree()`
- `initPhysics()`
- `initOrbitControls()`
- `resetSimulation()`
- `stepSimulation()`
- Render loop
- Data logger
- Chart.js setup
- Window resize handler
- Draggable sidebar width logic

All browser-only code must be wrapped in:

```
onMounted()
```

or guarded with:

```
if (process.client)
```

---
