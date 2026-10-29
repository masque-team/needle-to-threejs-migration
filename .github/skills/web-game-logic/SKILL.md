---
name: web-game-logic
description: "Port or fix web-game behavior while preserving the original rules. Use for state machines, input timing, delta time, collisions, scoring, spawn patterns, parent movement, pooling, restart, เกมกระตุก, จุดชน, เจนไอเทม และขยับฉาก. Not for GLB shader/material import or visual particle tuning."
---

# Web Game Logic

## Scope

Preserve the original rules and timing through a renderer-independent simulation.
Start at the function that controls the requested behavior, not only UI forwarding code.

## Audit Before Porting

1. Read the original implementation and scene/prefab overrides for the touched behavior.
2. Record states, exact transitions, input edge, timing, scoring/speed thresholds, spawn patterns and collision dimensions.
3. Distinguish deliberate changes from accidental differences. In particular, click on release is not pointerdown.
4. Identify one cheap discriminating test, implement a small change, and run it before expanding the scope.

## Simulation Rules

- Use one coherent game clock/distance for characters, world motion, spawn and collisions. Do not run generation on a separate wall clock that advances while motion is clamped or paused.
- Inspect the original engine version for delta clamping and physics/update order. Do not assume the old implementation was unlimited realtime.
- Choose bounded substeps from the gameplay risk; verify fast movement cannot tunnel through colliders.
- Mark differences between a custom Box3 trigger check and the original physics runtime. Matching box sizes does not prove identical collision timing.
- Keep rendering/UI as consumers of state and events, not owners of scoring or spawn rules.

## Moving Objects And Pooling

1. Preload required assets and prepare reusable instances to match maximum simultaneous usage, including repeated random selections of one variant.
2. For segmented worlds, move every active segment first. Recycle afterward and calculate tail placement from positions at the same simulation time.
3. Verify segment length and connector/pivot positions; do not overlap models merely to hide a position error.
4. For objects moving together, use a shared parent when appropriate. Keep local spawn coordinates and derive world coordinates for collision, generation and cleanup.

```text
groupOffset = -distance
worldPosition = localPosition + groupOffset
```

5. Return collected/offscreen objects to pools; do not destroy and recreate them every cycle.
6. Keep logical identity, model maps and ownership consistent across restart. Do not reset IDs while retaining a conflicting renderer map.
7. Bound look-ahead generation and pool growth. Consider coordinate rebasing for very long runs.

## Lifecycle

Define what start, pause, hidden-tab return, finish, home and restart reset or retain.
Input must not remain enabled in the wrong state. Freeze gameplay consistently during pause.
Publish bounded gameplay events so audio/VFX can subscribe and unsubscribe without changing game rules.
An end-game burst may keep rendering after simulation stops; continuous running effects must stop.

## Focused Tests

- State transitions, repeated input, mid-action input and restart.
- Ordinary/special scoring and exact difficulty thresholds.
- Slow/invalid frame deltas, pause/resume and high-speed collision.
- Spawn spacing/patterns, world/local coordinate agreement and cleanup cutoffs.
- Thousands of world-recycle steps, every segment pair, repeated variants and bounded object counts.

Report which behavior was preserved, which changed intentionally, and which original behavior cannot yet be verified.
Use `quarks-vfx` to change the visual response to events; do not add particle construction to simulation code.