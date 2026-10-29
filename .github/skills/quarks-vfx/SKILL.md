---
name: quarks-vfx
description: "Create or tune three.quarks particle effects for Three.js games. Use for Quarks prefab JSON, smoke, running dust, collection glow, sparkles, death bursts, effect pooling, พาร์ติเคิล, พาติเคส, ควัน, แสงเก็บของ และเอฟเฟกต์. Keep visual settings in standard JSON; do not load the full migration workflow for a small VFX edit."
---

# Quarks VFX

## Scope

Improve a specific visual cue without changing gameplay rules.
Keep particle decoration in one standard Quarks prefab JSON per agreed effect bundle.
Runtime code should load, position, trigger, update and clean up effects, not contain many procedural emitter builders.

## Procedure

1. Read the named effect JSON and the small controller that triggers it. Identify the actual event, group/emitter and visible issue before exploring engine internals.
2. When migrating from Needle, follow real controller references and child emitters. Needle metadata is not standalone Quarks JSON even though Needle uses Quarks internally.
3. Reuse authorized textures/colors/timing as reference. Inspect image alpha, blending, UV, scale and camera framing before assuming a shader bug.
4. Edit or export standard Quarks JSON using tools compatible with the installed version, then load it back through QuarksLoader to validate it.
5. Store material, texture/image references, emission, bursts, shape, lifetime, color, size curves and transforms in JSON. Do not turn a small tuning request into a new particle framework.
6. Keep continuous effects, one-shot action bursts, collection and death cues in separately triggerable groups. Do not restart a continuous trail every time an unrelated action happens.
7. Use worldSpace when old particles should remain behind the moving emitter. Update the emitter position, not every existing particle by hand.

## Runtime Contract

- Add emitters below a real Three.js Scene; a detached or Group-only tree may cause Quarks to dispose systems.
- Add systems to one suitable BatchedRenderer through QuarksUtil and prepare assets/shaders before play.
- Reuse emitters and batches; do not allocate systems for every tap or pickup.
- Group names and userData for offsets/follow are application conventions, not universal Quarks APIs. Match the controller explicitly.
- Freeze update during pause/hidden tabs. Clear on restart, stop continuous effects on death, and let the death burst expire.
- Unsubscribe events and dispose owned batches, geometry, material and textures without disposing resources borrowed from other systems.

## Budgets And Appearance

Budget active particles from emission rate times lifetime, burst counts, overlap and retrigger behavior.
A budget field in JSON is documentation/test input unless the runtime actually enforces it.
Use short readable effects before increasing particle count or transparent screen coverage.
Bloom-like glow need not require a full-screen postprocessing pass; first use suitable textures/blending.
Additive blending may suit bright shapes on black; normal blending usually needs appropriate alpha for smoke. Verify the actual art.
Never promise every Unity noise, trail, collision or sub-emitter module is imported automatically.

## Focused Validation

1. Load the real JSON, not a second hand-built version of the effect in tests.
2. Check event playback, follow offsets/directions, lifetime/expiry, reuse, particle counts, pause, restart and disposal.
3. For rising smoke, assert positive upward velocity and continued rendering after game over, then zero particles after expiry.
4. Run typecheck/build when controller code changes. For JSON tuning, run the focused effect check before broader validation.
5. Use one targeted final visual confirmation of the short-lived effect; prefer controlled time in the test, not debug UI in the game. Restore real time afterward.
6. If performance is questioned, compare effects off/on with identical scene, camera, resolution and warmup. Record frame time/p95, draw calls, counts and device. CPU submission is not GPU completion; desktop results do not establish mobile thermals.

Finish by naming the JSON changed, what looks different and the checks actually run. Surface real blockers early rather than spending many exploratory calls on a simple appearance change.