# LunaPark — Presentation Script

---

## Part 1 — Opening (P1)

**P1:** So, our project is *LunaPark*, an interactive 3D amusement park built with Three.js.

Let's open the site. *(clicks the link)*

The park has 4 rides, a panoramic train, 3 hot air balloons, and a shooting gallery. You can explore everything with click-to-fly, and hop into first-person view on the rides.

---

## Part 2 — Code Architecture

**P1:** This is the project structure. *(slide with tree)*

We organized the code into folders by domain:
- `src/environment/` — the scenery: ground, sky, river, vegetation, benches, food stalls, fireworks.
- `src/rides/` — the rides: each one is a separate component with its own animation.
- `src/people/` — the NPCs walking around the park and ride passengers.
- `src/controls/` — navigation and interaction: click-to-fly, raycasting, camera switching.
- `src/ui/` — the HUD and control panels.
- `src/lighting/` — lighting and day/night cycle.
- `src/utils/` — various utilities.

The bulk of it is in `App.js`: it creates the scene, the renderer, and wires everything together. Each component updates itself in the animation loop and communicates through an event system.

**P2:** *(next slides)* Let's go into the individual files.

Starting with `App.js`: here we initialize the Three.js scene, the renderer with post-processing — we used Three.js's bloom effect for the night lights — and mount everything.

For the environment: `Sky.js` handles the transition between day, night, sunrise and sunset. The textures are HDR images from Polyhaven, and the transition is an interpolation between the four textures based on the sun's position. `Water.js` animates the river with a custom vertex shader for wave motion, plus foam and caustics effects. `Fireworks.js` generates fireworks with a particle system — each particle has its own position, velocity, and color, updated frame by frame.

For interaction: `CameraManager.js` manages the camera — orbit, click-to-fly, first-person on rides, and 6 fixed positions triggered by keyboard. `InteractionManager.js` does raycasting: it casts a ray from the mouse position and checks if it hits interactive objects like lampposts or rides.

For the rides: they all extend `RideBase.js`, which handles start, stop, and the base animation loop. Then each one specializes. The *Coaster* moves along a spline curve — a curve that passes through control points — and the speed varies with height, like a real roller coaster. The *FerrisWheel* has gondolas on a separate group that counter-rotates to keep them upright. The *Carousel* animates the horses with a wave motion, each one phase-shifted from the others. The *Balloons* drift with the wind, which is adjustable.

For the NPCs: `Visitors.js` handles pathfinding. Each visitor has a destination, computes the shortest path on a grid representing the park, and walks along it. The walking animation is procedural: a gait engine with Fourier-series kinematics and two-bone IK.

For lighting: `DayNightCycle.js` simulates the solar cycle — the sun and moon rotate on an orbit, the lighting changes, and at sunset the lampposts turn on automatically. You can also override them manually: Auto, always on, or always off.

---

## Part 3 — Demo

**P1:** *(at the demo)* Let's try it out.

As you can see, the park has 4 rides, a train running around the perimeter, and 3 balloons in the sky.

To move around, you click where you want to go — click-to-fly. *(clicks on the ground)* The camera flies toward the point — the transition is handled with Tween.js, which interpolates position and target in about 1.2 seconds with an easing that smooths the arrival.

Lampposts are clickable *(clicks a lamppost)*: Auto, On, Off. They turn on automatically at sunset, but you can override them manually.

Clicking the control panel on a ride *(clicks a control panel)* toggles the ride on and off.

**P2:** Keyboard shortcuts.

Keys `1` through `6` each offer a fixed camera angle.

`Space` pauses the day/night cycle. *(presses Space)* You can see the lighting change — the sun rotates on an orbit and the sky transitions between phases.

`F` triggers fireworks. *(presses F)*

Now let's get on the rides. The bar at the bottom has the ride buttons *(points at the hotbar)*: click one and you go into first-person view on that ride.

*[P2 gets on the Ferris Wheel]*

Let's start with the **Ferris Wheel**. *(clicks button)*

The challenge here was keeping the gondolas upright while the wheel rotates. If they were attached directly, they'd flip over with the wheel and passengers would end up upside down. The solution was to put each gondola in a separate group that counter-rotates, canceling out the rotation.

*[P2 exits, gets on the Carousel]*

The **Carousel**. *(clicks button)*

The hard part was animating the horses naturally. Each horse moves up and down with a phase offset from the others, creating a wave effect.

*[P2 exits, gets on the Roller Coaster]*

The **Roller Coaster**. *(clicks button)*

The track is automatically extracted from the GLB rail-tube mesh — each ring yields a control point on a CatmullRom spline. The train's speed depends on height — it accelerates downhill, slows uphill. On curves it banks sideways.

*[P2 exits, gets on the Tagada]*

The **Tagada**. *(clicks button)*

It combines multiple simultaneous movements: the platform rotates and bounces rhythmically, giving that signature bouncing feel of this ride.

*[P2 exits]*

**P1:** Now the **Shooting Gallery**. *(moves toward it)*

It doesn't have a button in the bar because it's a separate experience. When you get close enough, pressing `T` enters aiming mode.

*(presses T, goes into first-person with crosshair)*

You have 30 seconds to hit as many targets as possible. Close targets are slower and give fewer points. Far ones are faster but give more points — so it pays to aim far.

*(shoots a few times)*

Each click uses raycasting: a ray fires from the crosshair and checks if it hits a target. It's immediate and doesn't need physics simulation.

To exit, `ESC`.

---

## Part 4 — Closing (P1 + P2)

**P1:** To sum up:
- 3D scene with dynamic environment and lighting
- 4 rides, each with different animation mechanics
- Panoramic train and balloons with wind drift
- NPCs with autonomous pathfinding and procedural walking
- Click-to-fly, clickable lampposts and rides
- FPS timed shooting gallery
- Bloom effect for night lighting

**P2:** All built with Three.js and WebGL 2.0, imported 3D models, PBR textures, and HDR skyboxes. All animations are procedural — no pre-loaded animation clips — including the NPC passengers riding along on every attraction.

**P1:** That's it.
