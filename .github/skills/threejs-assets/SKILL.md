---
name: threejs-assets
description: "Import only required assets during a Needle-to-web migration. Use for selected GLBs/dependencies, decoder and animation compatibility, deduplicating geometry/material/texture, and InstancedMesh for compatible repeated models. ใช้เฉพาะย้าย asset จาก Needle พร้อมแชร์ทรัพยากรและ instance โมเดลซ้ำเมื่อทำได้ ไม่คัดลอกทั้งโปรเจกต์หรือแก้เกมทั่วไป."
---

# Three.js Assets

## Scope

Make selected source Needle assets render and animate correctly in the agreed web port, with explicit resource ownership.
Start from the named GLB, loader, mesh or visible failure. Keep the original export unchanged when possible.
Do not use this skill for unrelated asset creation, art improvements or general model optimization outside that migration.

## Select Before Copying

- Build an allowlist from the agreed source scenes and actual source gameplay/UI/VFX references. Record source, reason and dependencies for each entry.
- Include required skeletons, clips, material maps, audio/UI/font files and dynamic prefab selections. Check string-based/runtime loads before removing apparent orphans.
- Import only those files and required dependency/license files. Never mirror entire Assets, export/public, package, demo or cache folders.
- Do not bring unused characters/scenes, framework starter art, editor-only content or the Needle runtime into the standalone port.
- A used GLB may still contain unused embedded meshes/clips/images. Extract/export only the selected content when safe, preserve all references, and retest. Report unresolved extra content if a safe extraction has not been achieved.

## Procedure

1. Inspect the exported GLB before re-exporting FBX. Read scenes/nodes, meshes/skins, animations, materials/textures/images and required extensions with a structured parser.
2. Compare source scene/prefab transforms, coordinate conventions, scale and pivot. Do not flip axes or rescale the whole scene by guesswork.
3. Configure only needed decoders: DRACOLoader for Draco, KTX2Loader plus detectSupport(renderer) for Basis/KTX2, MeshoptDecoder for Meshopt.
4. Serve assets over HTTP(S). Use local decoder files when runtime CDN dependency is not wanted. Check secure-context requirements for LAN testing.
5. Load one selected representative scene and character first. Skip animation work for static-only assets. Finish required lazy material/texture dependencies before disposing decoders.
6. Verify clips contain tracks, valid durations and real bone/pixel movement. A successful load with empty tracks is not success.
7. For KHR_animation_pointer, check actual loader/plugin support. A narrow node-target adapter may map translation/rotation/scale/weights to standard node/path targets; reject unsupported pointers rather than dropping them silently.
8. If rewriting GLB metadata, preserve binary keyframes, chunk length, padding and alignment. Test the binary is unchanged. Re-export or use a maintained loader extension when appropriate.
9. Map animations explicitly for the new game. Use SkeletonUtils.clone where independent skinned instances need their own skeletons; scene.clone is not a universal character-cloning solution.

## Shared Material And Texture Checks

Run compatible resource deduplication as part of importing the required asset allowlist. Do not wait for a separate optimization request when equivalence is proven.

Before sharing across independently loaded GLBs, compare:

- Complete material definitions: shader/type, color, emissive, transparency, alpha, blending, sidedness and relevant flags.
- Every texture slot, embedded image content, sampler, color space and UV transform.
- Normals, tangents, vertex colors, morphs, skinning, lightmaps and animated/per-instance material state.

Deduplicate image content, texture objects and materials at their correct levels:

1. Identical images may share one packaged source if target references can be updated safely.
2. Share a texture object only when content, sampler, color space, UV channel and transform match. Distinct materials may still share compatible textures.
3. Use a canonical material only when all material settings and relevant mesh flags also match, before creating pooled/instanced objects.
4. Preserve separate incompatible resources and report the reason. Do not merge materials merely to share a texture.

Preserve geometry and transforms. Never merge by material name alone.
Dispose replaced material/texture resources only when no loaded scene or other owner still uses them.
Check runtime object identity, including pooled copies. Do not confuse shared materials with instancing or draw-call reduction.
Sharing runtime objects does not remove duplicated bytes from GLB files; packaging/download optimization is separate.

## Repeated Geometry And Instanced Rendering

Repeated source models must not automatically become independently allocated geometry/material/texture copies in the target.
For example, 100 compatible copies of one static tree should share one geometry/material and use an InstancedMesh with 100 transforms, not 100 separate drawable Mesh objects by default.

1. Check geometry equivalence by actual attributes, indices, normals/tangents, UVs, groups/draw ranges and relevant deformation state, not mesh names alone.
2. Deduplicate identical geometry across selected GLBs when safe. Sharing geometry saves duplicate resources but does not itself reduce draw calls.
3. Preserve existing EXT_mesh_gpu_instancing when supported. Otherwise prefer InstancedMesh for repeated compatible static geometry/material combinations; group complex prefabs by compatible mesh part rather than assuming a whole prefab is one mesh.
4. Store per-instance transforms and retain a mapping from instanceId to the original logical item/collider/picking identity. Reuse slots and update instance matrices/count/bounds when objects move, spawn, despawn or recycle.
5. Keep groups spatially bounded, such as per reusable world section. Recompute bounds and verify culling; one giant instance group can worsen culling.
6. Do not force ordinary InstancedMesh onto independent skeleton animations, incompatible morph states, negative-scale/transform cases it cannot represent, transparency sorting or independently changing material state. Keep shared resources with separate objects when required and report why instancing was skipped.
7. Static geometry merging is a different fallback: use only for compatible immovable pieces when per-object transforms/picking/culling are preserved or explicitly remapped. Do not merge the whole world indiscriminately.

Do not silently treat pooling as instancing. Pooling controls allocation/reuse; instancing batches drawing.
Do not create a new texture atlas, change UVs, resize/recompress textures or redesign materials without separately agreed scope.

## Focused Validation

- Assert shared instances use the same material/texture objects, and incompatible assets stay separate.
- Assert geometry and transforms are unchanged by resource sharing.
- For repeated models, verify shared geometry identity, instance count/matrices, logical identity/colliders and actual renderer draw calls. Report any group that cannot be instanced.
- Verify spawn/despawn/restart updates instance slots and bounds without missing objects or collisions.
- Report geometry/material/texture object counts, packaged image counts and draw calls separately before/after. Do not claim that fewer objects means fewer triangles or download bytes.
- Verify textures remain correct after reuse and after disposal of unused duplicates.
- Verify missing/unsupported asset data produces explicit errors.
- Verify every copied file and retained embedded asset is required or explicitly reported as unresolved. Do not treat a whole copied folder as a valid allowlist.
- Check the real renderer and one final screenshot when visual fidelity needs confirmation; avoid repeated screenshot loops.

## Boundaries

Do not copy source-game filenames, node indices, clip ordering or prefab counts into a different game without checking.
Do not translate gameplay or Unity particle modules here. Use `web-game-logic` or `quarks-vfx` for those tasks.
Keep importer output separate from hand-authored configuration, and never write back into the original project without permission.