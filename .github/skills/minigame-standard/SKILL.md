---
name: minigame-standard
description: "Build or standardize a standalone browser minigame using Svelte 5, TypeScript, Tailwind CSS 4, Three.js, Vite 8; screen flow, FX/CSS, asset folders, editable config, and testing. Use for new games and post-migration web games. ไม่เปลี่ยน GLB/Prefab architecture."
---

# MiniGame Development Standard V1

Use for new web games and the final structure of migrated games. For fidelity-preserving Unity/Needle migration, use needle-migration first. This skill is **not** a game engine, shared prefab SDK or migration converter.

## Requirements

1. **Stack:** TypeScript game logic, Svelte 5 UI, Tailwind CSS 4 via @tailwindcss/vite, Three.js, Vite 8, Node.js 22.12+. Prefer Svelte + Vite standalone; only choose SvelteKit when necessary.
2. **Screen flow:** Menu, Gameplay, Result with Restart/Home; Loading/Error states. Put Svelte screens in src/screens/, HUD/dialogs/controls in src/components/. Extra pause, countdown and selection screens only when used.
3. **Ownership:** src/lib/game/ owns state, rules and GameSession; src/lib/three/ owns scenes, rendering, models and effects; Svelte owns overlay/UI. App.svelte composes them; one source of truth for score and phase.
4. **CSS/FX:** Tailwind 4 + CSS keyframes for layout/simple transitions; GSAP optional for complex UI animation. Three.js effects as needed; three.quarks optional for complex particles. Cleanup transitions/tweens/emitters.
5. **Assets:** public/assets/ui/, images/, models/, textures/, audio/, vfx/; cards/ and prefabs/ only if game needs them. Keep file references centralized and use import.meta.env.BASE_URL to support subpath hosting. Allow replacing same-name PNG without modifying rules. Use reproducible scanner/generated manifest when adding/removing dynamic image files, never hand-edit generated output.
6. **Config:** mandatory public/config/game.json, loaded at runtime and validated with Zod before play. Expose intended tuning knobs (time, score, speed, difficulty, lives) with valid ranges. Invalid/missing config displays a real error. Do not place secrets in public files.
7. **Lifecycle/QA:** handle input, mobile touch/portrait/safe-area/resize, pause/visibility, restart, asset caching and resource disposal; check/test/build. New games should use Vitest for GameSession and rule tests. Verify production build and deployment path.
8. **Scope boundary:** DO NOT prescribe or rewrite GLB exports, prefab JSON, component schemas, CharacterPrefab/ObstaclePrefab runtime, ECS, animation rigging or SDK. Keep current implementations working; organize only their files in agreed locations. Do not copy a sample game's prefabs blindly.

## Preferred folder shape

```text
src/
  App.svelte
  app.css
  screens/                 MenuScreen, GameplayScreen, ResultScreen
  components/              HUD, Controls, Dialogs as needed
  lib/
    game/                  GameSession / rules / tests
    three/                 scene, loader, VFX; prefabs/ optional for TS code
    config.ts              JSON + Zod
public/
  assets/
    ui/ images/ models/ textures/ audio/ vfx/
    cards/                 optional
    prefabs/               optional, existing game-specific data only
  config/game.json
  generated/               optional machine-generated manifests
scripts/                   optional asset sync
```

## Workflow

1. Review current repo, target devices, game flow and existing asset representations.
2. Implement required stack/screens/structure without unnecessary migration or prefab redesign.
3. Move user-editable gameplay parameters into config with validation.
4. Define game state, renderer and UI lifecycles; document cleanup and asset ownership.
5. Run typecheck/tests/build; exercise Menu -> Play -> Result -> Restart, loading failure, asset replacement and mobile layout.
6. Report exact tests performed, limitations and any justified exceptions.

See MINIGAME_STANDARD.md at the repository root for full requirements. This skill remains usable on its own after copying into a game project.
