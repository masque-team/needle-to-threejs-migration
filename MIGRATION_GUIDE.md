# กระบวนการย้าย Unity / Needle เป็นเว็บ Three.js

ใช้คู่มือนี้ร่วมกับ [NEW_GAME_BRIEF.md](NEW_GAME_BRIEF.md) และ [CHECKLIST.md](CHECKLIST.md)
ทุกค่า ชื่อไฟล์ จำนวน prefab และกติกาต้องมาจากเกมที่กำลังย้าย ไม่ใช่ค่าเริ่มต้นตายตัวของคู่มือ

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

ตัวอย่างโครงสร้างปลายทาง ไม่ใช่ไฟล์ที่รีโปเอกสารนี้จัดมาให้:

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
แยกไฟล์ import ออกจากงานที่ปรับเอง เช่น reimport รูปควันได้ แต่ไม่เขียนทับ JSON particle ที่ตกแต่งแล้ว
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
ตัวช่วยที่เขียนสำหรับชื่อไฟล์หรือโมเดลแบบใดแบบหนึ่ง ต้องปรับ eligibility สำหรับเกมใหม่
การแชร์ไม่ลด draw calls โดยอัตโนมัติ และไม่ลดข้อมูลซ้ำในไฟล์ GLB บนดิสก์
การลด draw calls ต้องพิจารณารวม mesh/instancing ส่วนการลดดาวน์โหลดต้องจัดแพ็ก asset เพิ่ม

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
โดยทั่วไปควรมี typecheck, unit/regression tests, production build และ dependency audit

วัดเปิด–ปิดเอฟเฟกต์ด้วยฉาก/ความเร็ว/ความละเอียดเดียวกัน หลัง warmup หลายรอบ
เก็บ frame time, p95, draw calls, particle count และจำนวน texture/geometry
วัด restart หลายรอบและช่วงเล่นยาว แยก CPU submission ออกจาก GPU completion
บันทึกเครื่อง เบราว์เซอร์ viewport, DPR และข้อจำกัดของ timer
ผลบนคอมพิวเตอร์ไม่ยืนยัน FPS หรือความร้อนของมือถือ ต้องทดสอบอุปกรณ์จริง

ก่อนเผยแพร่ ให้แยกสิ่งที่ตรงต้นฉบับ สิ่งที่เปลี่ยนโดยตั้งใจ และสิ่งที่ยังตรวจไม่ได้
คู่มือนี้ไม่แทนการสำรวจ Unity shader, lighting/lightmaps, physics และ particle module ที่ซับซ้อน