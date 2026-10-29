# Migration Checklist

คัดลอกไปใช้ในรายงานของแต่ละเกม ไม่ต้องติ๊กให้ทุกเกมในรีโปกลาง
ระบุ commit/tag ของคู่มือ และแนบผลตรวจจริง ไม่ใช้เครื่องหมายผ่านแทนหลักฐาน

## ก่อนเริ่ม

- [ ] กรอก [NEW_GAME_BRIEF.md](NEW_GAME_BRIEF.md) และตกลงขอบเขตแล้ว
- [ ] เปิดเล่นต้นฉบับครบ flow หรือระบุชัดว่าส่วนใดเปิดไม่ได้
- [ ] ตรวจ implementation จริงและค่าที่ Inspector ทับไว้
- [ ] ตรวจสิทธิ์ asset และแยก source/target โดยไม่แก้ต้นฉบับ
- [ ] ไม่คัดลอก credentials, env หรือข้อมูลส่วนบุคคล
- [ ] ขอบเขตเป็นการย้ายจาก Needle ไม่รวมการสร้างเกมใหม่หรือเพิ่มฟีเจอร์หลัง port

## Asset และภาพ

- [ ] มี allowlist ระบุ source เหตุผล runtime owner และ dependency ของทุก asset ที่จะย้าย
- [ ] ตรวจ dynamic/string-based loads และ prefab variants ไม่ตัดไฟล์จากการเดา
- [ ] ไม่มีการคัดลอกทั้ง Assets/export/public, demo, cache, editor-only หรือ framework starter art
- [ ] GLB ที่มีของไม่ใช้ฝังอยู่ได้รับการแยกและทดสอบ หรือมีรายงานสิ่งที่ยังติดอยู่ชัดเจน
- [ ] ทดสอบ GLB ที่มีอยู่ก่อนแปลงใหม่
- [ ] decoder ตรง extension และไม่มีไฟล์หาย
- [ ] สเกล แกน pivot, hierarchy และกล้องตรงตามขอบเขต
- [ ] animation มี tracks และขยับจริงทุกตัวที่ต้องใช้
- [ ] texture ทุกช่อง alpha, emissive, sampler และ UV ถูกต้อง
- [ ] แชร์ material/texture เฉพาะที่เข้ากันได้ และนับ reference ตอนรันจริง
- [ ] รวม texture/material/geometry ซ้ำระหว่าง import แล้วโดยไม่ต้องรอคำสั่งแยก พร้อมจำนวนก่อน–หลังและเหตุผลของรายการที่ยังแยก
- [ ] โมเดลซ้ำใช้ InstancedMesh เมื่อรองรับ ไม่ถือว่ามี pool แล้วจบ; ตรวจ count/matrix/instanceId/collision/picking/culling
- [ ] ไม่ทำ atlas ใหม่ เปลี่ยน UV หรือคุณภาพ texture โดยไม่ได้ตกลง และไม่สรุป draw calls จากจำนวน material
- [ ] ไม่ dispose ทรัพยากรที่ผู้ใช้งานอื่นยังอ้างถึง
- [ ] ไม่อ้างว่าการแชร์ material เท่ากับลด draw calls

## Logic และ lifecycle

- [ ] input timing, score, speed, spawn และเงื่อนไขจบตรงต้นฉบับหรือมีรายการความต่าง
- [ ] ฉาก/ไอเทม/ตัวละคร/จุดชนใช้เวลาเกมที่สัมพันธ์กัน
- [ ] ชนตรงกับภาพ ทั้งระหว่างกระโดดและตอนเฟรมค้าง
- [ ] รอยต่อฉากไม่คลาดหลังเลื่อนสะสม และครอบคลุมทุกคู่ที่ใช้จริง
- [ ] pool รองรับการใช้ prefab แบบเดิมซ้ำพร้อมกัน
- [ ] restart/reset parent, item state และ ID/model maps สอดคล้องกัน
- [ ] pause/ซ่อนแท็บ/กลับมาเล่นไม่ทำให้โลกพุ่งหรือสถานะหลุด

## เอฟเฟกต์และ UI

- [ ] ย้ายเฉพาะเอฟเฟกต์และ texture ที่ต้นฉบับเรียกใช้จริง; ข้ามถ้าไม่มี และไม่เพิ่มของตกแต่งใหม่
- [ ] ใช้ prefab JSON มาตรฐานของ Quarks และโหลดภาพครบก่อนเล่น
- [ ] emitter/batch ใช้ซ้ำ จำนวนไม่โตไม่จำกัด
- [ ] เอฟเฟกต์ต่อเนื่องหยุดเมื่อแพ้ และ burst จบเองได้
- [ ] restart clears effects และออกเกมคืน listener/resources ครบ
- [ ] โลโก้ รูป UI เสียง และข้อความยาวครบตามขอบเขต
- [ ] ตรวจมือถือ เดสก์ท็อป แนวนอน จอเตี้ย safe area และ reduced motion
- [ ] host message ตรวจ origin/source และ API ที่ป้องกันตรวจสิทธิ์ฝั่ง server

## ก่อนส่งงาน

- [ ] typecheck, tests, production build และ dependency audit ผ่านหรือมีข้อยกเว้นระบุชัด
- [ ] ไม่มี console/network/WebGL error ที่เกี่ยวกับงาน
- [ ] วัด frame time/p95 และ resource counts หลัง warmup พร้อมระบุอุปกรณ์
- [ ] ทดสอบ restart หลายรอบและการเล่นยาว ไม่ใช้ช่วงสั้นพิสูจน์ว่าไม่มี memory leak
- [ ] ทดสอบโทรศัพท์จริงก่อนอ้างความลื่นหรือความร้อนบนมือถือ
- [ ] บันทึกสิ่งที่ยังไม่เหมือนต้นฉบับและส่วนที่ยังไม่ได้ตรวจ
- [ ] ไม่ push asset หรือข้อมูลเกมที่ไม่มีสิทธิ์ลง public