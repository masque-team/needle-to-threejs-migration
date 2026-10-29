# กระบวนการย้ายจาก Needle เป็นเว็บ Three.js

ใช้คู่มือนี้ร่วมกับ [NEW_GAME_BRIEF.md](NEW_GAME_BRIEF.md) และ [CHECKLIST.md](CHECKLIST.md)
ทุกค่า ชื่อไฟล์ จำนวน prefab และกติกาต้องมาจากเกมที่กำลังย้าย ไม่ใช่ค่าเริ่มต้นตายตัวของคู่มือ
ขอบเขตคือโปรเจกต์ Unity ที่มี Needle source/export อยู่แล้ว ไม่ใช่การสร้างเกมใหม่หรือเพิ่มงานทั่วไปหลัง port เสร็จ

## Stack ปลายทางและการแบ่งไฟล์

baseline ของโปรเจกต์ที่ย้ายคือ **Node.js 22.12+ + Svelte 5 + TypeScript + Three.js + Tailwind CSS 4 + Vite 8** ตามรูปแบบเว็บเกมของทีม
Vite config ใช้ Svelte plugin และ `@tailwindcss/vite`; CSS entry โหลด Tailwind 4 ด้วย `@import 'tailwindcss'`
ใช้ version range ที่เข้ากันและ lockfile ของโปรเจกต์นั้น อย่าคัดลอก lockfile หรือเวอร์ชัน dependency จากเกมตัวอย่างโดยไม่ตรวจ Node/runtime compatibility

โครงสร้างแนะนำ:

```text
src/
  App.svelte              ประกอบหน้าจอและเชื่อม UI กับ session
  app.css                 Tailwind entry, theme tokens และ base styles
  components/             menu, HUD, pause/end และ UI ที่แบ่งตามหน้าที่
  lib/game/               state, rules, input, session และ logic tests
  lib/three/              scene, camera, asset loading, animation และ renderer
  lib/assets.ts           URL/schema/loading boundary ของ asset
public/
  assets/models/          GLB ที่เลือกและ dependency ที่จำเป็น
  assets/ui/              ภาพ UI ที่เลือก
  assets/audio/           เสียงที่ถูกเรียกใช้จริง
  assets/vfx/             Quarks JSON และภาพที่ระบบใช้
  config/                 JSON ปรับค่าได้ ตรวจด้วย Zod ก่อนใช้งาน
scripts/                  audit/import/sync/generate แบบอ่าน source และเขียนเฉพาะ target
package.json              dependencies และ scripts ของเกมนี้
package-lock.json         dependency graph ที่ reproducible ของเกมนี้
vite.config.ts            Svelte/Tailwind plugins, deploy base และ dev-only sync hooks
```

`App.svelte` ทำหน้าที่ประกอบ UI ไม่ใส่ game rules, GLB conversion หรือ renderer lifecycle ทั้งหมดไว้ใน component เดียว
เก็บ logic เกมที่ทดสอบได้โดยไม่สร้าง renderer ใน `src/lib/game`; ให้ `src/lib/three` รับผิดชอบ scene/camera/mixer/Quarks และคืน listener/GPU resources ตอน dispose
ใช้ Tailwind สำหรับ layout/utility และ theme tokens; component CSS ใช้เฉพาะกรณีที่ทำให้ animation, pseudo-element หรือ layout ซับซ้อนอ่านชัดขึ้น

รูปแบบจัดไฟล์สำหรับเว็บเกมมีสองชนิดและไม่ควรปะปนกัน:

| ชนิด | การจัดการ |
| --- | --- |
| ไฟล์ต้นฉบับที่เปลี่ยนตามรายการ เช่น card PNG | วางใน directory ที่กำหนดอย่างชัดเจน, สแกนชื่อ/metadata ด้วย script, สร้าง manifest ใหม่ และ watch add/change/unlink ใน dev เมื่อจำเป็น |
| asset คงที่จาก Needle เช่น GLB, UI, audio, VFX | เลือกผ่าน allowlist, import เฉพาะ dependency, คัดลอกเข้าปลายทาง และบันทึก source/owner; ห้าม scan ทั้งต้นฉบับหรือ mirror directory |
| config ที่ทีมตั้งใจแก้ | เก็บ JSON แยกใน `public/config`, ตรวจด้วย Zod/schema ก่อนใช้ และไม่ปนกับ generated manifest |

ตัวอย่าง dynamic catalog สร้างรายการจากชื่อ PNG และเขียน `manifest.json` ผ่าน sync script; ผู้ดูแลเพิ่ม/ลบ PNG ใน input directory และไม่แก้ manifest เอง
สำหรับเกม Needle ให้ใช้วิธีนี้เฉพาะ dynamic asset set ที่ source flow ต้องรองรับจริง ไม่สร้างระบบ catalog หรือ file watcher ให้เกมที่ไม่มีความต้องการนั้น

ถ้าต้นฉบับของเกมที่กำลังย้ายมี framework/runtime constraint ที่ใช้ baseline นี้ไม่ได้ ให้กรอกข้อยกเว้นใน NEW_GAME_BRIEF ก่อนเริ่ม พร้อมเหตุผล, ผลกระทบต่อ file layout และผู้อนุมัติ ห้ามเปลี่ยนเป็น vanilla DOM/CSS หรือเลือก Vite รุ่นเก่าโดยไม่บันทึก
`package.json` เป็นที่แก้ scripts และ dependency; regenerate/commit lockfile จากโปรเจกต์นั้นเองทุกครั้ง ส่วน `vite.config.ts` เก็บ plugin และ `base` path ที่ต้องตรงกับปลายทาง deploy; dev watcher เป็น optional และต้องไม่ติด production โดยไม่ตั้งใจ

## 1. สำรวจต้นฉบับก่อนลงมือ

1. เปิดเกมเดิม เล่นให้ครบหน้าแรก กติกา เล่น จบ และเริ่มใหม่
2. จด state, input, scoring, ความเร็ว, การเกิดวัตถุ, การชน และเงื่อนไขแพ้/ชนะ
3. ตรวจค่าจริงใน scene/prefab เพราะ Inspector อาจทับค่าเริ่มต้นในสคริปต์
4. แยกไฟล์ generated ออกจาก implementation จริง โดยเฉพาะ C# ที่สร้างจาก TypeScript
5. บันทึกกล้อง สเกล แกน pivot, animation, เสียง, UI และการสื่อสารกับเว็บแม่
6. สรุปขอบเขตและข้อแตกต่างให้ชัดก่อนปรับพฤติกรรมที่มีผลกับการเล่น

รักษาต้นฉบับไว้แบบอ่านอย่างเดียว สร้างปลายทางแยก และไม่เผยแพร่ asset โดยไม่มีสิทธิ์

**ผ่านขั้นนี้เมื่อ:** อธิบายกติกาและแหล่งค่าจริงได้ รวมถึงรู้ว่าส่วนใดยังตรวจไม่ได้

## 2. หา GLB ที่มีอยู่ก่อนแปลงใหม่

ตรวจโฟลเดอร์เว็บ export ที่ Needle ตั้งไว้ใน Unity ตำแหน่งแตกต่างกันตามเกมและเวอร์ชัน
ตัวอย่างตำแหน่งที่อาจพบคือ `Needle/app/assets` ไม่ใช่ path ที่ทุกเกมต้องมี

ใช้ parser ตรวจข้อมูลต่อไปนี้:

| ข้อมูล | สิ่งที่ต้องตรวจ |
| --- | --- |
| scenes / nodes | hierarchy, transform, ชื่อวัตถุและกล้อง |
| meshes / skins | ฉากนิ่ง ตัวละคร skeleton และ morph targets |
| animations | clips, channels, target, keyframes และ duration |
| materials / textures / images | ทุก texture slot, alpha, emissive, UV และ sampler |
| extensionsUsed / Required | compression และส่วนขยายที่ loader ต้องรองรับ |
| NEEDLE_components | ค่าอ้างอิง collider, prefab, emitter และคอมโพเนนต์เดิม |

ถ้ามี GLB ใช้งานได้ ให้ทดสอบก่อน ไม่แปลง FBX ทั้งหมดใหม่ทันที
ถ้าไม่มี ให้ export ผ่าน Unity exporter ที่รองรับ glTF หรือ Blender glTF 2.0
ตรวจ baked animation, skeleton, texture, สเกลและแกนหลัง export
อย่ากลับแกน Z หรือปรับสเกลทั้งฉากจากการคาดเดา เพราะ exporter อาจจัดการให้แล้ว

GLB ย้ายข้อมูลภาพได้ แต่ไม่ได้ทำให้ Logic, Animator Controller, physics หรือ Unity particle ทำงานเอง

## 3. จัด asset และสคริปต์นำเข้า

ตัวอย่างตำแหน่ง asset ในโครงสร้าง Svelte/Vite ปลายทาง ไม่ใช่ไฟล์ที่รีโปเอกสารนี้จัดมาให้:

```text
public/assets/
  models/
  ui/
  audio/
  decoders/draco/
  decoders/basis/
  vfx/effects.json
  vfx/
  scene-manifest.json
```

ทำ importer ที่อ่านต้นฉบับแล้วคัดลอกออกมา ไม่เขียนย้อนกลับ
ก่อนคัดลอก ให้ทำ allowlist จาก entry scene, screen flow และ reference ที่เกมใช้จริง
แต่ละรายการต้องมี source, เหตุผล, runtime owner และ dependency เช่น skeleton, clip, texture และเสียงที่เรียกใช้
ไล่ dependencies รวมการสุ่ม prefab และการโหลดด้วยชื่อ/string ในสคริปต์ ไม่ตัดของที่ยังพิสูจน์ว่าไม่ใช้ไม่ได้
import เฉพาะ allowlist กับ dependency ไม่ mirror ทั้ง Assets/export/public หรือแพ็กเกจ
ตัด scene/ตัวละครที่ไม่ใช้ demo, cache, editor-only content, framework starter art และ runtime Needle เก่าออก
สร้าง target scaffolding ของ Svelte/Vite/Tailwind ตามโครงสร้างด้านบนได้ แต่ไม่คัดลอก starter application หรือภาพตัวอย่างจากเกมอื่นมาเป็น content ของเกมที่ย้าย
decoder และไฟล์ license ที่จำเป็นต้องคงไว้ แต่ไม่คัดลอก node_modules ทั้งชุด
ถ้า GLB ที่ต้องใช้มีของไม่จำเป็นฝังอยู่ ให้ export/extract เฉพาะส่วนอย่างปลอดภัยแล้วตรวจ references/animation ใหม่
หากยังแยกอย่างปลอดภัยไม่ได้ ให้รายงานของที่ติดอยู่และข้อจำกัดก่อน ไม่เรียกการคัดลอกทั้ง GLB ว่าเลือก asset สะอาดแล้ว
แยกไฟล์ import ออกจากงานที่ปรับเอง เช่น reimport รูปควันได้ แต่ไม่เขียนทับ JSON particle ที่ตกแต่งแล้ว
กำหนด owner ให้แต่ละไฟล์: `public/assets` เป็น output ของ allowlist/import, `public/config` เป็นข้อมูลที่ทีมแก้และมี schema validation, manifest ที่ script สแกนสร้างเป็น generated output และ `src/` เป็น implementation ที่ทีมดูแล
ถ้าต้อง sync ไฟล์แบบ dynamic ให้ทำ script idempotent และตรวจเฉพาะ directory ที่ระบุ; watch add/change/unlink ระหว่าง dev ได้เมื่อ flow ต้องการ พร้อม cleanup watcher/timer ตอน server ปิด
ห้ามแก้ manifest ที่สร้างจาก script ด้วยมือ เพราะรอบ sync/build จะเขียนทับ ให้แก้ input/config ต้นทางแล้ว generate ใหม่
ให้ script sync เปรียบเทียบเนื้อหาก่อนเขียน, เขียนไฟล์ชั่วคราวแล้ว rename เมื่อทำได้, จำกัด output ให้อยู่ใน target directory และรายงานไฟล์ชื่อผิดที่ถูกข้าม
manifest ควรเก็บ reference และค่าฉากที่จำเป็น ไม่คัดลอกข้อมูลลับหรือข้อมูลส่วนบุคคลมาด้วย
บันทึกที่มาและเงื่อนไขสิทธิ์ของ asset และให้ build รันได้โดยไม่ต้องมี Unity หรือต้นฉบับติดตั้งอยู่

## 4. โหลด compression ให้ตรงชนิดไฟล์

| Extension | แนวทาง |
| --- | --- |
| KHR_draco_mesh_compression | GLTFLoader + DRACOLoader |
| KHR_texture_basisu | KTX2Loader + Basis transcoder + detectSupport(renderer) |
| EXT_meshopt_compression | MeshoptDecoder ถ้าไฟล์นั้นใช้ |
| ส่วนขยายเฉพาะ Needle | ตรวจเป็นรายส่วน อย่าสมมติว่า Three.js จะรันให้เอง |

โหลดและ decode ของที่ต้องใช้ก่อนอนุญาตให้เริ่มเกม
ถ้ามี material/texture ที่โหลดแบบ lazy ต้องดึง dependency ให้ครบก่อน dispose decoder
ใช้ไฟล์ decoder แบบ local เมื่อไม่ต้องการพึ่ง CDN และเสิร์ฟเกมผ่าน HTTP(S) ไม่ใช่ `file://`
ตรวจด้วยว่าฟังก์ชันเสริมต้องการ secure context หรือไม่ เมื่อทดสอบมือถือผ่าน HTTP ในวง LAN

## 5. ยืนยัน animation ว่าขยับจริง

อย่านับแค่ชื่อ clips: ไฟล์อาจโหลดสำเร็จแต่ได้ tracks ว่าง
`KHR_animation_pointer` อาจต้องใช้ extension เพิ่ม ขึ้นกับรุ่นของ loader

ถ้าจะทำ compatibility adapter สำหรับ pointer ไปยัง node:

1. ตรวจ schema และ target ที่รองรับอย่างชัดเจน
2. แปลง `/nodes/N/translation`, `/rotation`, `/scale` หรือ `/weights` เป็น target มาตรฐานที่มี `node` และ `path`
3. ถ้าเขียน GLB JSON ใหม่ ให้รักษา binary keyframe, alignment, padding และความยาว chunk
4. ไม่แปลง pointer ไปยัง material หรือ property อื่นแบบเงียบ ๆ ถ้ายังไม่รองรับ
5. ตรวจ tracks, duration และการขยับของกระดูกหรือพิกเซลจริง

อย่าสมมติลำดับหรือจำนวน clips ต่อหนึ่งตัวละคร ให้ทำ mapping จากเกมนั้น
เมื่อต้อง clone skinned character ให้พิจารณา `SkeletonUtils.clone` แทนวิธี clone ฉากนิ่งทั่วไป

## 6. แยก Logic, prefab และ renderer

Logic ดูแล state, score, spawn, input และผลการชน
prefab ดูแล hierarchy, animation และ transform ของวัตถุที่นำมาใช้ซ้ำ
renderer ดูแล scene, camera, update loop และการคืนทรัพยากร ส่วน UI แสดงสถานะและส่งคำสั่ง

ใช้ transform/กล้องเดิมเป็นจุดเริ่มต้น แล้วตรวจหลาย aspect ratios
แยกการจัดกรอบหน้าเมนูออกจากกล้องขณะเล่น และอย่าให้การปรับ UI เปลี่ยนกติกาโดยไม่ตั้งใจ
node index อาจเปลี่ยนหลัง export ใหม่ ต้องตรวจ reference และ mapping ใหม่เสมอ

## 7. รักษาการแชร์ material และ texture

การรวมทรัพยากรซ้ำที่พิสูจน์ว่าเข้ากันได้เป็นส่วนหนึ่งของ import ให้ทำได้เลยก่อนสร้าง pool ไม่ต้องรอสั่ง optimization แยก
ทำเฉพาะ asset ใน allowlist ไม่ย้ายทั้งคลังมาเพื่อรวมของที่ไม่ได้ใช้

material เดียวใน Unity ไม่ได้รับประกันว่าจะเป็น object เดียวเมื่อโหลด GLB แยกหลายไฟล์
สำเนาจาก prefab เดียวกันอาจแชร์ทรัพยากรอยู่แล้ว แต่ต้องตรวจข้าม GLB ด้วย

1. เทียบ material definition ไม่ใช่ชื่ออย่างเดียว: shader, alpha, blending, sidedness และ emissive
2. เทียบ image content ของทุก texture slot พร้อม sampler, UV transform และ color space
3. ตรวจ normals, tangents, vertex color, morph, skin และ material animation
4. เลือก canonical material แล้วแทน reference เฉพาะกลุ่มที่เข้ากันได้ หลังโหลดและก่อนสร้าง pool
5. เก็บ geometry และ transform เดิม ไม่รวม mesh โดยอัตโนมัติ
6. dispose ชุดซ้ำเฉพาะเมื่อไม่มี scene หรือผู้ใช้งานอื่นอ้างถึงแล้ว
7. นับ object identity ตอนรันจริง รวมสำเนาใน pool และทดสอบว่าภาพไม่เปลี่ยน

อย่าบังคับแชร์ lightmap หรือวัสดุที่ต้องเปลี่ยนสถานะแยกชิ้น
แยกการรวม image file, texture object และ material ให้ถูกระดับ: รูปเหมือนกันแต่ sampler/color space/UV ต่างกันยังอาจต้องใช้ texture object แยก
material คนละตัวก็ยังใช้ texture ที่เข้ากันได้ร่วมกันได้ ไม่ต้องบังคับรวม material
อย่าทำ atlas ใหม่ เปลี่ยน UV หรือลดความละเอียด/recompress โดยถือว่าเป็นการแชร์ texture ธรรมดา
ตัวช่วยที่เขียนสำหรับชื่อไฟล์หรือโมเดลแบบใดแบบหนึ่ง ต้องปรับ eligibility สำหรับเกมใหม่
การแชร์ไม่ลด draw calls โดยอัตโนมัติ และไม่ลดข้อมูลซ้ำในไฟล์ GLB บนดิสก์
การลด draw calls ต้องพิจารณารวม mesh/instancing ส่วนการลดดาวน์โหลดต้องจัดแพ็ก asset เพิ่ม

### โมเดลซ้ำให้ใช้ geometry ร่วมและ instancing เมื่อทำได้

ตัวอย่างต้นไม้แบบเดียวกัน 100 ต้น: ใช้ geometry/material ชุดเดียวและ InstancedMesh ที่มี 100 transforms แทน Mesh แยก 100 ตัว เมื่อไม่มีข้อจำกัดที่ทำให้ใช้ร่วมกันไม่ได้
ตรวจ geometry จาก attributes, indices, normals/tangents, UV, groups และ deformation state ไม่ดูชื่อ mesh อย่างเดียว
ถ้า prefab มีหลายส่วน ให้แยกกลุ่ม instance ตาม geometry/material ที่เข้ากันได้ ไม่สมมติว่าทั้ง prefab เป็น mesh เดียว

รักษา instanceId → logical object/collider mapping และ world transform ของแต่ละชิ้น
ตอนเกิด เก็บคืน pool หรือ restart ต้องอัปเดต slot/matrix/count/bounds ให้ตรงกัน
แบ่งกลุ่มตามพื้นที่หรือท่อนฉากเพื่อไม่ให้ culling กลายเป็นกลุ่มใหญ่เกินจำเป็น
ไม่บังคับใช้ InstancedMesh ปกติกับ skeleton ที่ animate แยก, morph/transform ที่ไม่รองรับ, transparency sorting หรือวัสดุที่เปลี่ยนแยกชิ้น
ถ้าใช้ไม่ได้ ให้แชร์ทรัพยากรที่ปลอดภัย เก็บ object แยก และรายงานเหตุผล

pooling ไม่เท่ากับ instancing: อย่างแรกลดการสร้าง/ทำลาย อย่างหลังลดการสั่งวาดสำหรับกลุ่มที่รองรับ
รายงาน geometry/material/texture counts, image files และ draw calls แยกกัน พร้อมทดสอบภาพ ตำแหน่ง การชน picking และ culling
อย่ารับประกันว่าเหลือหนึ่ง draw call เสมอ เพราะหลาย material/ส่วนของ mesh และ shadow/render passes อาจยังต้องวาดหลายครั้ง

## 8. รักษาจังหวะเล่นและเวลาเกม

ใช้เวลา/ระยะเกมชุดเดียวกันกับตัวละคร ฉาก ไอเทม การเกิดวัตถุ และจุดชน
อย่าให้ generator เดินตามเวลาจริงอีกชุดขณะที่การเคลื่อนที่ใช้เวลาที่ clamp แล้ว

| เรื่อง | จุดที่ต้องเทียบ |
| --- | --- |
| Input | pointerdown กับ click ตอนปล่อยนิ้วให้ความไวต่างกัน |
| เวลาเฟรม | ตรวจ delta clamp, pause และวิธีรับมือเฟรมค้างของ engine รุ่นต้นฉบับ |
| Collision | กล่องคำนวณเองไม่เท่ากับ physics เดิม แม้ใช้ขนาดเดียวกัน |
| Spawn | เติมหนึ่งแถวต่อ update กับเติมจนระยะล่วงหน้าครบให้ผลต่างตอนกระตุก |
| Restart | state, score, speed, parent offset และ pool ต้องกลับสถานะที่กำหนด |

เลือก substep และขีดจำกัดเวลาจากกติกาจริง ไม่ใช้ค่าตายตัวกับทุกเกม
ระบุความต่างที่ตั้งใจเปลี่ยน แยกจากบั๊ก และทดสอบการชนระหว่างกระโดดหรือเมื่อเครื่องช้า

## 9. เคลื่อนที่และ pooling

เตรียม instance ตามจำนวนที่อาจใช้พร้อมกัน รวมกรณีสุ่ม prefab แบบเดียวกันซ้ำ
วัตถุที่พ้นฉากให้ซ่อนคืน pool แทนโหลดไฟล์หรือสร้าง/ทำลายทุกครั้ง

สำหรับฉากเป็นท่อน ให้เลื่อนทุกท่อนก่อน แล้วค่อย recycle และคำนวณตำแหน่งต่อท้าย
ห้ามคำนวณจากตำแหน่งปนกันระหว่างท่อนที่เลื่อนแล้วกับท่อนที่ยังไม่เลื่อน เพราะจะเกิดช่องว่างค้างได้
ทดสอบทุกคู่ของท่อนฉากและเลื่อนสะสมหลายพันเฟรม

สำหรับวัตถุที่เคลื่อนด้วยระยะเดียวกัน อาจรวมใต้ parent เดียว:

```text
groupZ = -distance
itemWorldZ = itemLocalZ + groupZ
```

ใช้ world position ตรวจชน ระยะสร้างล่วงหน้า และจุดเก็บหลังฉาก
เปลี่ยน local position เฉพาะตอนเกิดหรือ reuse; reset parent กับสถานะ item ให้ตรงกัน
อย่า reset ID แต่ปล่อย map ของโมเดลเก่าค้าง และประเมินการ rebase พิกัดถ้าเกมเล่นได้นานมาก

## 10. เอฟเฟกต์เป็น Quarks JSON ก้อนเดียว

ทำขั้นนี้เฉพาะเมื่อเกมต้นทางมีเอฟเฟกต์ที่ใช้จริงใน flow ที่ตกลง ไม่เพิ่ม particle ให้เกมที่ไม่มี
เลือก emitter จาก reference ของ controller และรวมเฉพาะ texture/material/child emitter ที่จำเป็น
ไม่คัดลอกทั้ง VFX pack และไม่แต่งควัน/แสงใหม่หรือสร้างภาพทดแทนโดยไม่ได้รับอนุญาต

Needle ใช้ three.quarks ภายใน แต่ `NEEDLE_components` ไม่ใช่ standalone Quarks prefab JSON
ใช้ข้อมูลเดิมเป็น reference แล้วจัดหรือ export เป็น JSON มาตรฐานด้วยเครื่องมือรุ่นที่เข้ากันได้
เก็บ shape, lifetime, color, size, burst, emission, materials และ behaviors ใน JSON ไม่กระจายตัวสร้างระบบหลายไฟล์

prefab ควรมี metadata, geometry, material, texture/image references และ Group ที่มี ParticleEmitter
ข้อมูล emitter อยู่ใน `ps`; ทดสอบ round-trip ด้วย `QuarksLoader` ไม่ใช้เพียง JSON ที่มีหน้าตาคล้ายกัน

ลำดับ runtime:

1. โหลด JSON และภาพให้เสร็จก่อนเล่น
2. เพิ่มใต้ Scene จริง และใช้ QuarksUtil เพิ่มระบบให้ BatchedRenderer
3. เตรียม emitter/buffer และ compile shader ที่จำเป็น
4. รับ event เกมเพื่อเล่นระบบเดิมซ้ำ ไม่สร้าง emitter ใหม่ทุกเหตุการณ์
5. update ด้วยเวลาเกม หยุดตาม pause/visibility
6. ตอนแพ้ให้หยุดระบบต่อเนื่อง แต่ปล่อย burst จบก่อน clear
7. ตอนออกเกมให้ถอด listener และคืน batch/material/texture ตาม ownership

ชื่อกลุ่มและ `userData` สำหรับ offset/follow เป็นข้อตกลงของ controller แต่ละเกม ไม่ใช่ Quarks API มาตรฐาน
ค่า budget ใน metadata ก็ไม่ใช่ hard cap อัตโนมัติ ต้องตรวจ burst, rate × lifetime และการ retrigger
ใช้ worldSpace เมื่ออยากให้อนุภาคเก่าคงอยู่หลัง emitter ขยับ
เริ่มด้วยเอฟเฟกต์สั้น จำกัดจำนวน และทดสอบภาพจริงบนจอเป้าหมาย

ตรวจ alpha, blending, UV และ texture ก่อนแก้ shader
ภาพพื้นดำอาจเหมาะกับ additive ส่วนควันใช้ภาพโปร่งใสกับ normal blending ตามงานภาพ
การย้าย prefab ไม่ได้รับประกันว่าจะรองรับ Unity noise, trails, collision หรือ sub-emitter ทุก module

## 11. UI เสียง และ host integration

นำภาพ UI/เสียงเดิมมาให้ครบ รวม animation โลโก้และจังหวะเปลี่ยนหน้า
แยก transform ของ animation ออกจากการจัดกึ่งกลาง และตรวจข้อความยาวทุกตัวเลือก
รองรับ safe area, จอเตี้ย, แนวนอน และ reduced motion
เริ่มเสียงจาก user gesture และหยุด/คืนทรัพยากรตาม lifecycle

คงชื่อ event/payload ของเว็บแม่ แต่ตรวจ origin และ source ของข้อความรับ
ตรวจ object ownership และสิทธิ์ฝั่ง server ถ้ามี API ที่ต้องป้องกัน
ห้ามใส่ secret ใน client bundle หรือ environment variable ที่ส่งไป client
อย่ารวมไฟล์ env, credentials หรือข้อมูลส่วนบุคคลใน repo/public asset

## 12. ตรวจผ่านและวัดผล

ใช้ [CHECKLIST.md](CHECKLIST.md) พร้อมแนบหลักฐานจากเกมที่ย้าย
คำสั่งตรวจขึ้นกับโปรเจกต์ปลายทาง ไม่ใช่คำสั่งที่มีอยู่ในรีโปเอกสารนี้
สำหรับ baseline Svelte/Vite ให้มี `svelte-check` + TypeScript check, Vitest/unit regression tests, Vite production build และ dependency audit
แยก test ของ rules/session/file naming ออกจาก browser/render tests; ระบุว่าการเช็กใดใช้ source assets หรือ WebGL จริง

วัดเปิด–ปิดเอฟเฟกต์ด้วยฉาก/ความเร็ว/ความละเอียดเดียวกัน หลัง warmup หลายรอบ
เก็บ frame time, p95, draw calls, particle count และจำนวน texture/geometry
วัด restart หลายรอบและช่วงเล่นยาว แยก CPU submission ออกจาก GPU completion
บันทึกเครื่อง เบราว์เซอร์ viewport, DPR และข้อจำกัดของ timer
ผลบนคอมพิวเตอร์ไม่ยืนยัน FPS หรือความร้อนของมือถือ ต้องทดสอบอุปกรณ์จริง

ก่อนเผยแพร่ ให้แยกสิ่งที่ตรงต้นฉบับ สิ่งที่เปลี่ยนโดยตั้งใจ และสิ่งที่ยังตรวจไม่ได้
คู่มือนี้ไม่แทนการสำรวจ Unity shader, lighting/lightmaps, physics และ particle module ที่ซับซ้อน