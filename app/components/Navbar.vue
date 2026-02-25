<template>
  <div class="navbar bg-base-100/80 backdrop-blur shadow-sm sticky top-0 z-30">
    <div class="navbar-start">
      <!-- Mobile Menu Dropdown -->
      <div class="dropdown">
        <label tabindex="0" class="btn btn-ghost lg:hidden" aria-label="Menu">
          <Icon name="lucide:menu" class="h-5 w-5" />
        </label>
        <ul
          tabindex="0"
          class="menu menu-sm dropdown-content mt-3 z-[1] p-2 shadow bg-base-100 rounded-box w-52 max-h-[80vh] overflow-y-auto"
        >
          <li><nuxt-link to="/">Home</nuxt-link></li>
          <li><nuxt-link to="/about">About</nuxt-link></li>
          <li>
            <details>
              <summary>Chapter</summary>
              <ul class="p-2">
                <li v-for="chapter in chapters" :key="chapter.path">
                  <nuxt-link :to="chapter.path" class="flex items-center">
                    <Icon :name="chapter.icon" class="mr-2 h-4 w-4" />
                    {{ chapter.name }}
                  </nuxt-link>
                </li>
              </ul>
            </details>
          </li>
          <li><nuxt-link to="/about">Contact me</nuxt-link></li>
        </ul>
      </div>

      <!-- Website Brand/Logo -->
      <nuxt-link class="btn btn-ghost text-xl normal-case font-bold" to="/"
        >PhySim</nuxt-link
      >
    </div>

    <!-- Desktop Menu -->
    <div class="navbar-end hidden lg:flex">
      <ul class="menu menu-horizontal px-5 font-bold">
        <li><nuxt-link to="/">Home</nuxt-link></li>
        <li><nuxt-link to="/about">About</nuxt-link></li>
        <li>
          <details>
            <summary>Chapter</summary>
            <ul class="p-2 bg-base-100 rounded-box shadow-lg min-w-max z-50">
              <li v-for="chapter in chapters" :key="chapter.path">
                <nuxt-link
                  :to="chapter.path"
                  class="flex items-center hover:bg-primary hover:text-primary-content transition-colors"
                >
                  <Icon :name="chapter.icon" class="mr-2 h-4 w-4" />
                  {{ chapter.name }}
                </nuxt-link>
              </li>
            </ul>
          </details>
        </li>
        <li>
          <nuxt-link to="/about">Contact me</nuxt-link>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
const chapters = [
  { name: "Incline", path: "/Chapter/Incline", icon: "lucide:triangle" },
  {
    name: "Projectile",
    path: "/Chapter/Projectile",
    icon: "lucide:move-up-right",
  },
  { name: "Pendulum", path: "/Chapter/Pendulum", icon: "lucide:timer" },
  {
    name: "Three Body Simulation",
    path: "/Chapter/ThreeBodySimulation",
    icon: "lucide:orbit",
  },
  {
    name: "Doppler effect",
    path: "/Chapter/PhysicsDoppler",
    icon: "lucide:waves",
  },
  {
    name: "Stopping Distance",
    path: "/Chapter/StoppingDistanceSim",
    icon: "lucide:car",
  },
  {
    name: "Bowl Simulation",
    path: "/Chapter/BowlSimulation",
    icon: "lucide:circle-dashed",
  },
  {
    name: "Ideal Gas Simulation",
    path: "/Chapter/IdealGasSimulation",
    icon: "lucide:wind",
  },
  { name: "Collision", path: "/Chapter/collision", icon: "lucide:zap" },
  { name: "Buoyancy", path: "/Chapter/Buoyancy", icon: "lucide:droplets" },
  {
    name: "Simple Harmonic Motion",
    path: "/Chapter/SimpleHarmonicMotion",
    icon: "lucide:waves",
  },
  {
    name: "Rutherford Scattering",
    path: "/Chapter/RutherfordScattering",
    icon: "lucide:circle-pile",
  },
  {
    name: "Free Fall with Air Resistance",
    path: "/Chapter/FreeFallAirResistance",
    icon: "lucide:arrow-down"
  },
];
</script>

<style scoped>
.navbar {
  transition:
    background-color 0.3s ease,
    backdrop-filter 0.3s ease;
}
</style>
