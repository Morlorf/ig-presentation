---
theme: seriph
background: https://images.unsplash.com/photo-1513889961551-628c1e5e2ee9?q=80&w=2070&auto=format&fit=crop
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Luna Park Presentation
  Interactive Graphics Course
drawings:
  persist: false
title: Luna Park - Interactive 3D
---

# Luna Park
### Interactive 3D Amusement Park

<div class="mt-8 text-xl font-light">
Interactive Graphics Course<br>
<b>Enrico Battistoni</b> & <b>Davide De Blasio</b>
</div>

<div class="abs-br m-6 flex gap-2">
  <carbon-play class="text-xl" />
</div>

---
layout: center
---
# The Vision
<br>
<v-clicks>

- **Goal:** Unify the *entire* Interactive Graphics syllabus in one project.
- **The Golden Rule:** 🚫 No baked animation clips.
- **Result:** Everything is procedural, math-driven, and real-time.

</v-clicks>

---
layout: two-cols
---
<template #default>

# Tech Stack
<br>

<v-click>

### <carbon-code class="inline-block align-middle mr-1" /> Core Rendering
- **Three.js (r170)** & WebGL 2.0
- Built from scratch (No Unity/Unreal)

</v-click>

<br>

<v-click>

### <carbon-color-palette class="inline-block align-middle mr-1" /> Materials & Shaders
- **PBR** (Physically Based Rendering)
- Custom **GLSL** `#version 300 es`

</v-click>

</template>
<template #right>

<div class="h-full flex items-center justify-center">
  <div class="text-2xl space-y-8 pl-12 border-l-2 border-gray-300">
    <v-click>
      <div><carbon-camera class="inline-block align-middle mr-1" /> <b>Post-Processing</b><br>
      <span class="text-lg opacity-75">ACES Filmic & UnrealBloom</span></div>
    </v-click>
    <v-click>
      <div><carbon-chart-line-smooth class="inline-block align-middle mr-1" /> <b>Animations</b><br>
      <span class="text-lg opacity-75">Mathematical Transforms & tween.js</span></div>
    </v-click>
  </div>
</div>

</template>

---
layout: default
---
# Scene Graph & Hierarchical Modeling

Building complex machines through simple parent-child relationships.

<div class="grid grid-cols-2 gap-8 mt-10">

<div class="bg-gray-100 dark:bg-gray-800 p-6 rounded-lg shadow-lg border border-gray-200 dark:border-gray-700">
<h3 class="text-xl font-bold flex items-center gap-2 mb-4 text-orange-500"><carbon-rotate /> Turbo Tagada</h3>
<ul>
  <li><b>Arm Pivot:</b> Pitch & Roll (Lissajous curve)</li>
  <li><b>Platform:</b> Continuous Yaw Spin</li>
  <li class="mt-2 text-sm opacity-80">Compound oscillation without complex single-mesh math.</li>
</ul>
</div>

<div class="bg-gray-100 dark:bg-gray-800 p-6 rounded-lg shadow-lg border border-gray-200 dark:border-gray-700">
<h3 class="text-xl font-bold flex items-center gap-2 mb-4 text-blue-500"><carbon-circle-dash /> Sky Wheel</h3>
<ul>
  <li><b>Main Wheel:</b> Rotates by $+angle$</li>
  <li><b>Gondolas:</b> Counter-rotate by $-angle$</li>
  <li class="mt-2 text-sm opacity-80">Riders remain perfectly upright automatically.</li>
</ul>
</div>

</div>

---
layout: center
---

# Tangled Twister
## Advanced Mechanics on the Roller Coaster

<br>

<div class="flex flex-col gap-6 text-xl">

<div v-click class="flex items-center gap-4">
  <div class="w-12 h-12 rounded-full bg-blue-500 flex items-center justify-center text-white shrink-0"><carbon-edge-node /></div>
  <span><b>Procedural Track:</b> Extracted rail centre-lines directly from mesh vertices.</span>
</div>

<div v-click class="flex items-center gap-4">
  <div class="w-12 h-12 rounded-full bg-green-500 flex items-center justify-center text-white shrink-0"><carbon-data-1 /></div>
  <span><b>Parallel Transport Frame:</b> Replaced the Frenet frame to prevent violent flipping at zero curvature.</span>
</div>

<div v-click class="flex items-center gap-4">
  <div class="w-12 h-12 rounded-full bg-purple-500 flex items-center justify-center text-white shrink-0"><carbon-flash /></div>
  <span><b>Energy-Governed Speed:</b> $v^2 = v_0^2 + 2g(y_{top} - y)$</span>
</div>

</div>

---
layout: default
---
# Custom GLSL Shaders
Pushing the GPU beyond standard Three.js materials.

<div class="grid grid-cols-2 gap-8 mt-12">

<div class="p-4">
  <h2 class="text-blue-500 flex items-center gap-2"><carbon-humidity /> Water Shader</h2>
  <ul class="space-y-4 mt-4 text-lg">
    <li><b>Vertex:</b> Sum of 4 Gerstner waves.</li>
    <li><b>Normals:</b> Analytical calculation via partial derivatives.</li>
    <li><b>Fragment:</b> Procedural caustics & depth-based color blending.</li>
  </ul>
</div>

<div class="p-4">
  <h2 class="text-orange-500 flex items-center gap-2"><carbon-sun /> Sky Shader</h2>
  <ul class="space-y-4 mt-4 text-lg">
    <li><b>Geometry:</b> Inverted sphere.</li>
    <li><b>Matrix Trick:</b> Ignores camera translation.</li>
    <li><b>Dynamic:</b> Crossfades 4 HDR maps based on the time of day.</li>
  </ul>
</div>

</div>

---
layout: two-cols
---

# Lighting & Shadows
<br>

<ul class="text-lg space-y-4 pr-4">
  <li><b>Diurnal Cycle:</b> Parameter $t \in [0,1]$ controls sun/moon orbit.</li>
  <li><b>Global Illumination:</b>
    <ul>
      <li><code class="text-sm">HemisphereLight</code> (sky scattering)</li>
      <li><code class="text-sm">DirectionalLight</code> (Sun/Moon with PCF Soft Shadows)</li>
    </ul>
  </li>
  <li><b>Nighttime Ambiance:</b>
    <ul>
      <li>20+ PointLights & SpotLights.</li>
      <li>Phase-staggered sinusoidal pulsing.</li>
    </ul>
  </li>
</ul>

::right::

<div class="h-full flex items-center justify-center p-8">
  <div class="bg-gray-100 dark:bg-gray-800 p-8 rounded-xl shadow-xl border-l-4 border-yellow-500">
    <h3 class="text-xl mb-4 font-bold flex items-center gap-2"><carbon-magic-wand /> Post-Processing</h3>
    <p class="text-lg opacity-80 leading-relaxed">
      Using <b>ACES Filmic Tone Mapping</b> to compress HDR to LDR, combined with <b>UnrealBloomPass</b> to make the neon lights and fireworks truly glow.
    </p>
  </div>
</div>

---
layout: center
class: text-center
---

# GPU Particles: Fireworks
<br>

<div class="text-2xl font-light mb-8">
  Offloading thousands of particles from the CPU to the GPU.
</div>

<div class="bg-gray-100 dark:bg-gray-800 p-6 rounded-lg shadow-lg inline-block text-left text-xl border border-gray-200 dark:border-gray-700">
  <b class="text-red-400">Vertex Shader Ballistic Integration:</b><br><br>
  <code class="text-blue-500 text-lg">$pos = center + velocity \times age - 0.5 \times gravity \times age^2$</code>
</div>

<br><br>

<div class="flex justify-center gap-8 mt-6 text-lg opacity-80">
  <span class="flex items-center gap-2"><carbon-circle-dash /> Spherical Bursts</span>
  <span class="flex items-center gap-2"><carbon-shape-except /> Equatorial Coronas</span>
  <span class="flex items-center gap-2"><carbon-align-vertical-bottom /> Drooping Willow Trails</span>
</div>

---
layout: default
---
# Procedural NPCs & Pathfinding
Making the park feel alive and independent.

<div class="grid grid-cols-2 gap-8 mt-10 text-lg">

<div class="bg-gray-100 dark:bg-gray-800 p-6 rounded-lg">
  <h3 class="font-bold flex items-center gap-2 text-green-500 text-xl"><carbon-map /> Navigation</h3>
  <ul class="mt-4 space-y-2">
    <li><b>Grid:</b> 2x2 unit occupancy mapping.</li>
    <li><b>Algorithm:</b> A* Pathfinding.</li>
    <li><b>Smoothing:</b> "String-pulling" to avoid robotic, grid-aligned zig-zags.</li>
  </ul>
</div>

<div class="bg-gray-100 dark:bg-gray-800 p-6 rounded-lg">
  <h3 class="font-bold flex items-center gap-2 text-purple-500 text-xl"><carbon-pedestrian /> Gait Engine</h3>
  <ul class="mt-4 space-y-2">
    <li><b>Zero Foot-Skate:</b> Planted foot slides backward exactly at walking speed.</li>
    <li><b>Leg IK:</b> Analytic two-bone Inverse Kinematics.</li>
    <li><b>Rhythm:</b> Dynamic pelvis bob & spine counter-rotation.</li>
  </ul>
</div>

</div>

---
layout: center
class: text-center
---

# Conclusion
<br>

<div class="text-xl space-y-6 max-w-2xl mx-auto leading-relaxed">
  <p><b>Luna Park</b> successfully unifies hierarchical modeling, procedural mathematics, and custom GLSL shaders.</p>
  
  <p class="opacity-80">Despite the heavy procedural logic, it maintains a smooth <b>60 FPS</b> on consumer hardware, creating a highly interactive and visually stunning 3D sandbox.</p>
</div>

<div class="mt-12">
  <h2 class="text-4xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-blue-400 to-purple-500">
    Thank You!
  </h2>
  <p class="mt-4 opacity-75">Questions? Let's dive into the Live Demo.</p>
</div>
