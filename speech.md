# LunaPark — Presentation Script

---

## Part 1 — Opening (P1)

**P1:** Our project is *LunaPark*, an interactive 3D amusement park built with Three.js.

Let's open the site. *(clicks the link)*

The park has 4 rides, a panoramic train, 3 hot air balloons, and a shooting gallery. You can explore everything with click-to-fly, and hop into first-person view on the rides.

---

## Part 2 — Code Architecture

**P1:** This is the project structure. *(slide with tree)*

We organized the code into domain folders: environment, rides, people, controls, lighting, UI.

`App.js` creates the scene and the renderer with post-processing — UnrealBloomPass for the night glow — and wires everything together in the animation loop. Components communicate through an event bus.

**P2:** Let me highlight three hard problems.

First, the **water shader**. The river uses a vertex shader summing four Gerstner waves — crests sharpen naturally and the lighting reacts to every ripple. Normals are computed analytically from partial derivatives, with noise-based caustics flowing over the riverbed.

Second, the **roller coaster track**. It's a CatmullRom spline extracted automatically from the rail-tube mesh. We use a rotation-minimizing frame — unlike Frenet, it doesn't flip at inflection points — and the train speed follows a conservation-of-energy model: accelerates downhill, slows uphill.

Third, the **NPC gait**. Each visitor pathfinds with A* on a navigation grid and smooths paths with string-pulling. The walk is procedural: each foot slides backward at walking speed — zero foot-skate — with two-bone IK solving the knee.

The rides all extend `RideBase.js`. The FerrisWheel counter-rotates each gondola with quaternion multiplication. The Carousel drives horses with phase-shifted sine waves. The Balloons drift with adjustable wind. Fireworks use GPU particles: 5000 per burst in the vertex shader.

For lighting: the sun and moon orbit on a celestial sphere; the sky crossfades four HDR textures via GLSL mixing. At sunset, lampposts switch to Auto — overridable to On or Off.

---

## Part 3 — Demo

**P1:** *(at the demo)* Let's try it out.

As you can see, the park has 4 rides, a train running around the perimeter, and 3 balloons in the sky.

To move around, click where you want to go — click-to-fly. *(clicks on the ground)* The camera flies there in about 1.2 seconds using Tween.js with a custom easing curve.

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

It combines six simultaneous movements: the platform spins, the arm yaws, pitches, and rolls, the ride bounces on two axes, and the arm telescopes outward — giving it that signature chaotic feel.

*[P2 exits]*

**P1:** Now the **Shooting Gallery**. *(moves toward it)*

It doesn't have a button in the bar because it's a separate experience. When you get close enough, pressing `T` enters aiming mode.

*(presses T, goes into first-person with crosshair)*

You have 30 seconds to hit as many targets as possible. Close targets are slower and give fewer points. Far ones are faster but give more points — so it pays to aim far.

*(shoots a few times)*

Each click fires a ray from the crosshair using raycasting — instant hit detection, no physics engine needed.

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

**P2:** Built with WebGL 2.0 and ACES Filmic tone mapping, imported 3D models with PBR textures, and HDR skyboxes. All animations are procedural — no pre-loaded animation clips — including the NPC passengers riding along on every attraction.

**P1:** That's it.
