---
name: quarks-vfx
description: "Port existing Needle particle effects required by a Needle-to-web migration into standard three.quarks JSON. Use only for translating referenced original emitters and required textures during that port. ใช้ย้ายเอฟเฟกต์เดิมจาก Needle เท่านั้น ไม่ใช้ตกแต่งควัน/แสงทั่วไป เพิ่มเอฟเฟกต์ใหม่ หรือปรับความสวยหลังการย้าย."
---

# Quarks VFX

## Scope

Reproduce only original Needle visual cues required by the agreed migration, without changing gameplay rules.
Keep particle decoration in one standard Quarks prefab JSON per agreed effect bundle.
Runtime code should load, position, trigger, update and clean up effects, not contain many procedural emitter builders.
Skip this skill when the source has no required particle effects. Do not add smoke, bloom, sparkles or replacement art for decoration unless separately authorized.

## Procedure

1. Read the source Needle controller and its referenced emitters, then the target JSON/controller. Identify the original event and group before exploring engine internals.
2. When migrating from Needle, follow real controller references and child emitters. Needle metadata is not standalone Quarks JSON even though Needle uses Quarks internally.
3. Allowlist only required emitter textures/material dependencies, including used child emitters. Do not copy an entire VFX pack, demos or unrelated effects. Preserve source colors/timing and inspect alpha, blending, UV, scale and framing before assuming a shader bug.
4. Edit or export standard Quarks JSON using tools compatible with the installed version, then load it back through QuarksLoader to validate it.
5. Store material, texture/image references, emission, bursts, shape, lifetime, color, size curves and transforms in JSON. Document unsupported modules and ask before replacing their appearance; do not add a particle framework or new art by default.
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
Match original density, timing and appearance; do not increase counts, coverage or add postprocessing as a new enhancement.
If the original effect cannot be reproduced within the agreed constraints, report that limitation before changing its appearance.
Additive blending may suit bright shapes on black; normal blending usually needs appropriate alpha for smoke. Verify the actual art.
Never promise every Unity noise, trail, collision or sub-emitter module is imported automatically.

## Focused Validation

1. Load the real JSON, not a second hand-built version of the effect in tests.
2. Check event playback, follow offsets/directions, lifetime/expiry, reuse, particle counts, pause, restart and disposal.
3. Test the original effect's required direction, event and expiry. For example, test rising smoke only if the source actually requires it.
4. Run typecheck/build when controller code changes. For JSON tuning, run the focused effect check before broader validation.
5. Use one targeted final visual confirmation of the short-lived effect; prefer controlled time in the test, not debug UI in the game. Restore real time afterward.
6. If performance is questioned, compare effects off/on with identical scene, camera, resolution and warmup. Record frame time/p95, draw calls, counts and device. CPU submission is not GPU completion; desktop results do not establish mobile thermals.

Finish by listing the original emitters/textures migrated, any agreed visual differences and the checks run. Stop after the port is verified; general particle decoration is outside this skill.