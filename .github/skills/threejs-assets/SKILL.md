---
name: threejs-assets
description: "Import Unity or Needle GLB assets into Three.js. Use for glTF/GLB export, Draco/KTX2/Meshopt loading, missing animation tracks, skeleton cloning, prefab transforms, shared materials/textures, โมเดล GLB, แอนิเมชัน และแชร์ material. Not for scoring rules or particle decoration."
---

# Three.js Assets

## Scope

Make an existing asset render and animate correctly, with explicit resource ownership.
Start from the named GLB, loader, mesh or visible failure. Keep the original export unchanged when possible.

## Procedure

1. Inspect the exported GLB before re-exporting FBX. Read scenes/nodes, meshes/skins, animations, materials/textures/images and required extensions with a structured parser.
2. Compare source scene/prefab transforms, coordinate conventions, scale and pivot. Do not flip axes or rescale the whole scene by guesswork.
3. Configure only needed decoders: DRACOLoader for Draco, KTX2Loader plus detectSupport(renderer) for Basis/KTX2, MeshoptDecoder for Meshopt.
4. Serve assets over HTTP(S). Use local decoder files when runtime CDN dependency is not wanted. Check secure-context requirements for LAN testing.
5. Load one representative scene and character first. Finish lazy material/texture dependencies before disposing decoders.
6. Verify clips contain tracks, valid durations and real bone/pixel movement. A successful load with empty tracks is not success.
7. For KHR_animation_pointer, check actual loader/plugin support. A narrow node-target adapter may map translation/rotation/scale/weights to standard node/path targets; reject unsupported pointers rather than dropping them silently.
8. If rewriting GLB metadata, preserve binary keyframes, chunk length, padding and alignment. Test the binary is unchanged. Re-export or use a maintained loader extension when appropriate.
9. Map animations explicitly for the new game. Use SkeletonUtils.clone where independent skinned instances need their own skeletons; scene.clone is not a universal character-cloning solution.

## Shared Material And Texture Checks

Before sharing across independently loaded GLBs, compare:

- Complete material definitions: shader/type, color, emissive, transparency, alpha, blending, sidedness and relevant flags.
- Every texture slot, embedded image content, sampler, color space and UV transform.
- Normals, tangents, vertex colors, morphs, skinning, lightmaps and animated/per-instance material state.

Use a canonical material only for compatible assets, before creating pooled clones.
Preserve geometry and transforms. Never merge by material name alone.
Dispose replaced material/texture resources only when no loaded scene or other owner still uses them.
Check runtime object identity, including pooled copies. Do not confuse shared materials with instancing or draw-call reduction.
Sharing runtime objects does not remove duplicated bytes from GLB files; packaging/download optimization is separate.

## Focused Validation

- Assert shared instances use the same material/texture objects, and incompatible assets stay separate.
- Assert geometry and transforms are unchanged by resource sharing.
- Verify textures remain correct after reuse and after disposal of unused duplicates.
- Verify missing/unsupported asset data produces explicit errors.
- Check the real renderer and one final screenshot when visual fidelity needs confirmation; avoid repeated screenshot loops.

## Boundaries

Do not copy source-game filenames, node indices, clip ordering or prefab counts into a different game without checking.
Do not translate gameplay or Unity particle modules here. Use `web-game-logic` or `quarks-vfx` for those tasks.
Keep importer output separate from hand-authored configuration, and never write back into the original project without permission.