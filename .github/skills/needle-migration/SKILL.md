---
name: needle-migration
description: "Migrate an existing Needle-exported Unity game to a standalone Three.js web game. Use for Needle-to-web conversion, auditing the source Needle game and coordinating only required asset, logic and VFX migration. ใช้เฉพาะตอนย้ายเกมจาก Needle เป็นเว็บ ไม่ใช้สร้างเกมใหม่หรือแก้เกม/ตกแต่งเอฟเฟกต์ทั่วไป."
---

# Needle Migration

## Scope

Coordinate a Needle-to-web migration while preserving the agreed original behavior and required assets.
Read repository instructions first. Treat the original project as read-only unless explicitly authorized.
This is not a new-game template, a general Three.js development skill or a post-migration enhancement workflow.
Stop using this skill when the agreed port is verified; do not add redesign, new effects, features or generic optimization work.

## Start With Evidence

1. Establish source project/scene, target directory, target devices and allowed changes.
2. Locate the real implementation. Generated C# components may only declare fields for TypeScript behavior.
3. Play the original start/rules/play/end/restart flow when possible. Mark unavailable checks honestly.
4. Inspect Inspector/scene overrides, not only script defaults. Record camera, transforms, input timing, score, spawn, collision, audio and host messages.
5. Summarize the smallest migration slice and a check that can fail before editing. Ask only about unresolved scope or behavior changes.

## Required Asset Allowlist

1. Start from the agreed entry scenes, original screen flow and actual runtime references.
2. Record each selected model, UI image, audio clip, font, VFX texture/JSON and decoder with its source, reason and dependencies.
3. Follow dependencies recursively, including skeletons, textures, animation clips and dynamically selected prefab variants. Inspect script/string-based loads; uncertain usage is not proof an asset is unused.
4. Copy only this selected dependency closure. Do not copy entire Assets, export, public or package folders just because they exist.
5. Exclude unused scenes/characters, editor files, caches, demos, framework starter images and the old Needle runtime. Retain required decoder/license files, not whole dependency trees.
6. If a necessary GLB embeds unused content, safely export/extract the selected content and revalidate references, transforms and animation. If extraction is not yet safe, report the extra embedded content and resolve that limitation rather than claiming the asset set is clean.

## Load Only The Relevant Skill

- `threejs-assets`: only assets selected for this Needle port and their required resource dependencies.
- `web-game-logic`: only behavior that exists in this Needle game and is in the agreed migration scope.
- `quarks-vfx`: only original Needle effects required by this port, if such effects exist.

If the focused skill is installed, read it when entering that task. Do not load all skills or the full guide for every small change.
These are skill names, not separate agents; do not spawn agents unless separately authorized.

## Implementation Order

1. Audit the original and identify a real source asset or controlling behavior.
2. Test one selected scene/character before importing the rest of the allowlist; prove geometry, textures and any required animation work.
3. Separate simulation, rendering/prefabs and UI. Preserve original values unless a change is agreed.
4. Deduplicate compatible required images/textures/materials/geometry during import without waiting for a separate request. Prefer instanced rendering for compatible repeated static models, preserve incompatible cases with reasons, and keep copied assets separate from authored JSON/config.
5. Port gameplay in small testable steps; validate immediately after a substantive edit.
6. Reuse only required original UI/audio and reproduce existing transitions on the agreed target screens.
7. If original effects are in scope, port them through standard Quarks JSON. Do not add decorative effects or create procedural emitter frameworks.
8. Run project-specific typecheck/tests/build/audit and exercise the real production build.

## Acceptance And Reporting

- No missing assets or relevant console/network/WebGL errors.
- Animation tracks and actual movement verified, not clip names alone.
- Pooling, collisions and restart tested across repeated cycles and slow frames.
- Every copied asset has an allowlist reason; excluded and unresolved embedded content are reported separately.
- Repeated models share compatible resources and use instancing where safe; verify transforms, picking/collisions, culling and draw calls, not pooling alone.
- Report matching behavior, intentional changes, remaining mismatches and unverified devices separately.
- Compare performance using the same scene, camera, resolution and warmup. Desktop results do not prove mobile FPS or thermals.
- Validate parent origin/source for host messages; never put secrets in public assets or client environment values.
- Do not commit/push, publish assets, or create public repositories without user authorization.

## Optional Background

For detailed explanations, consult the public repository only as needed:
https://github.com/masque-team/needle-to-threejs-migration

The installed workflow above is usable without that repository being present in the game workspace.