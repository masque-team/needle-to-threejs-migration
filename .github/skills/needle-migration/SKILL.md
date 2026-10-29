---
name: needle-migration
description: "Migrate a Unity or Needle game to a Three.js web game. Use for full game ports, migration planning, auditing original gameplay, exporting a web version, ย้ายเกม Needle เป็นเว็บ, ย้าย Unity และวางแผนแปลงเกม. Routes asset, logic and particle work to focused skills; do not load this for a small isolated VFX or material edit."
---

# Needle Migration

## Scope

Coordinate a complete migration while preserving the original gameplay and assets.
Read repository instructions first. Treat the original project as read-only unless explicitly authorized.
This skill is a workflow, not a game template or an automatic converter.

## Start With Evidence

1. Establish source project/scene, target directory, target devices and allowed changes.
2. Locate the real implementation. Generated C# components may only declare fields for TypeScript behavior.
3. Play the original start/rules/play/end/restart flow when possible. Mark unavailable checks honestly.
4. Inspect Inspector/scene overrides, not only script defaults. Record camera, transforms, input timing, score, spawn, collision, audio and host messages.
5. Summarize the smallest migration slice and a check that can fail before editing. Ask only about unresolved scope or behavior changes.

## Load Only The Relevant Skill

- `threejs-assets`: GLB, compression, skeleton/animation, prefab transforms and shared material/texture.
- `web-game-logic`: state, input, game time, collision, spawn, parent movement, pooling and restart.
- `quarks-vfx`: particle appearance, standard Quarks prefab JSON and VFX lifecycle.

If the focused skill is installed, read it when entering that task. Do not load all skills or the full guide for every small change.
These are skill names, not separate agents; do not spawn agents unless separately authorized.

## Implementation Order

1. Audit the original and identify a real source asset or controlling behavior.
2. Load one scene/character before copying all assets; prove geometry, textures and animation work.
3. Separate simulation, rendering/prefabs and UI. Preserve original values unless a change is agreed.
4. Restore shared resources before pooling. Keep copied assets separate from authored JSON/config.
5. Port gameplay in small testable steps; validate immediately after a substantive edit.
6. Reuse original UI/audio and reproduce transitions. Check long labels, portrait/landscape, short screens and reduced motion.
7. Add effects through standard Quarks JSON, not a collection of procedural emitter builders.
8. Run project-specific typecheck/tests/build/audit and exercise the real production build.

## Acceptance And Reporting

- No missing assets or relevant console/network/WebGL errors.
- Animation tracks and actual movement verified, not clip names alone.
- Pooling, collisions and restart tested across repeated cycles and slow frames.
- Report matching behavior, intentional changes, remaining mismatches and unverified devices separately.
- Compare performance using the same scene, camera, resolution and warmup. Desktop results do not prove mobile FPS or thermals.
- Validate parent origin/source for host messages; never put secrets in public assets or client environment values.
- Do not commit/push, publish assets, or create public repositories without user authorization.

## Optional Background

For detailed explanations, consult the public repository only as needed:
https://github.com/masque-team/needle-to-threejs-migration

The installed workflow above is usable without that repository being present in the game workspace.