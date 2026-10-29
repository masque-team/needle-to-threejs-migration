# Needle to Three.js Migration

คู่มือและ Skills เฉพาะการย้ายเกมที่มีต้นทาง Needle เป็นเว็บ Three.js
รักษากติกาเดิม ย้ายเฉพาะ asset ที่ต้องใช้จริง และเก็บเอฟเฟกต์เดิมที่จำเป็นเป็น Quarks JSON

**รีโปนี้มีคู่มือและ Agent Skills ไม่ใช่ template เกมหรือตัวแปลงอัตโนมัติ**
ไม่มีโมเดล ภาพ เสียง โค้ดเฉพาะเกม หรือข้อมูลรับรองของโปรเจกต์ต้นทาง

## ขอบเขตที่ตั้งใจให้ใช้

- ใช้เฉพาะช่วงแปลงเกมจาก Needle ไม่ใช่ Skill สร้างเกมใหม่ แก้เว็บทั่วไป หรือเพิ่มฟีเจอร์หลังย้ายเสร็จ
- เริ่มจาก scene/flow ที่ตกลง แล้วทำ asset allowlist พร้อมเหตุผลและ dependency ของแต่ละรายการ
- ย้ายเฉพาะโมเดล animation, UI, เสียง ฟอนต์ และเอฟเฟกต์ที่ถูกใช้จริง รวม dependency ที่จำเป็น
- ไม่คัดลอกทั้ง Assets/export/public, demo, cache, ไฟล์เริ่มต้น framework หรือ runtime Needle เก่า
- ตรวจการเรียกไฟล์แบบ dynamic ก่อนตัดออก; ถ้า GLB ฝังของที่ไม่ใช้ ให้แยกอย่างปลอดภัยหรือรายงานข้อจำกัด ไม่อ้างว่าทั้งไฟล์จำเป็น
- ไม่เพิ่มภาพหรือเอฟเฟกต์ตกแต่งใหม่โดยไม่ได้ตกลง
- ระหว่าง import ให้รวม image/texture/material/geometry ซ้ำที่เข้ากันได้เลย ไม่ต้องรอสั่งแยก แต่ต้องไม่เปลี่ยนภาพหรือพฤติกรรม
- โมเดลนิ่งแบบเดียวกันหลายชิ้นให้ใช้ InstancedMesh เมื่อรองรับ เช่น ต้นไม้เดียวกัน 100 ต้น ใช้ geometry/material ชุดเดียวและ 100 transforms
- การแชร์ทรัพยากร pooling และ instancing เป็นคนละเรื่อง ต้องตรวจ draw calls จริง; ไม่ทำ atlas ใหม่หรือเปลี่ยน UV เอง

## Skills แยกตามงาน

| Skill | ขอบเขต |
| --- | --- |
| [.github/skills/needle-migration/SKILL.md](.github/skills/needle-migration/SKILL.md) | สำรวจและย้ายเกมทั้งชุด เลือกอ่านเฉพาะ Skill ย่อยที่ต้องใช้ |
| [.github/skills/threejs-assets/SKILL.md](.github/skills/threejs-assets/SKILL.md) | ย้ายเฉพาะ asset และ dependency ที่จำเป็นจากเกม Needle |
| [.github/skills/web-game-logic/SKILL.md](.github/skills/web-game-logic/SKILL.md) | ย้าย Logic เดิมของเกม Needle ไม่เพิ่มกติกาหรือฟีเจอร์ใหม่ |
| [.github/skills/quarks-vfx/SKILL.md](.github/skills/quarks-vfx/SKILL.md) | ย้ายเฉพาะเอฟเฟกต์เดิมที่ใช้จริงเป็น Quarks JSON; ข้ามเมื่อไม่มี |

การใช้งานแบบ Skill ต้องติดตั้งในโปรเจกต์ปลายทางก่อน ไม่ใช่แนบลิงก์แล้วจะติดตั้งเอง
ดู [INSTALL_SKILLS.md](INSTALL_SKILLS.md) สำหรับวิธีติดตั้งและตัวอย่างคำสั่ง

## เริ่มใช้งาน

1. อ่าน [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)
2. กรอก [NEW_GAME_BRIEF.md](NEW_GAME_BRIEF.md) สำหรับเกมที่จะย้าย
3. ใช้ [CHECKLIST.md](CHECKLIST.md) ตรวจหลักฐานและผลทดสอบ
4. ระบุ commit หรือ tag ของคู่มือที่ใช้ในรายงานของเกมนั้น เพราะเนื้อหาบน `main` เปลี่ยนได้

## ข้อความสำหรับเริ่มย้ายเกมใหม่

```text
ใช้กระบวนการจาก https://github.com/masque-team/needle-to-threejs-migration
อ้างอิง commit/tag: <รุ่นของคู่มือ>

อ่าน README และ MIGRATION_GUIDE ก่อนเริ่มทำงาน
ข้อมูลเกมและขอบเขต: <แนบ NEW_GAME_BRIEF ที่กรอกแล้ว>

ตรวจเกมต้นฉบับและค่าจริงใน scene/prefab ก่อน สรุปสิ่งที่จะย้ายกับข้อแตกต่างที่พบ
อย่าแก้ต้นฉบับ อย่านำค่าหรือ asset ของเกมตัวอย่างมาใช้โดยไม่ตรวจ
ทำทีละส่วนและทดสอบทันที ย้ายเฉพาะเอฟเฟกต์เดิมที่จำเป็นเป็น JSON มาตรฐานของ three.quarks
ทำ asset allowlist พร้อมเหตุผลก่อน import ย้ายเฉพาะรายการที่ใช้จริงและ dependency
รวม texture/material/geometry ที่ใช้ร่วมกันได้เลย และทำ instancing ให้โมเดลซ้ำที่เหมาะสม
รักษาตำแหน่ง จุดชน และ identity ของแต่ละชิ้น รายงานจำนวนทรัพยากรกับ draw calls ก่อน–หลัง และเหตุผลของกลุ่มที่รวมไม่ได้
ไม่คัดลอกทั้งโฟลเดอร์ ไม่เพิ่มฟีเจอร์หรือของตกแต่งใหม่
รักษาการแชร์ material/texture ที่เข้ากันได้ และตรวจ reference ตอนรันจริง
แจ้งข้อจำกัดที่ยังไม่พิสูจน์ เช่น physics, แสง และประสิทธิภาพบนมือถือ
ห้ามเผยแพร่ asset หรือข้อมูลเฉพาะเกมขึ้น public โดยไม่ได้รับอนุญาต
```

การแนบลิงก์ไม่ได้รับประกันว่าเครื่องมือ AI ทุกตัวจะอ่านให้เอง
จึงควรระบุชื่อเอกสารให้เปิดอ่าน และให้ตรวจไฟล์ต้นทางที่เข้าถึงได้จริงใน workspace

## หลักที่ใช้ร่วมกัน

- เริ่มจากเกม Needle เดิมที่เล่นได้และค่าจริง ไม่เดาจากภาพหรือชื่อสคริปต์
- ตรวจ allowlist ของ asset และ dependency ไม่ย้ายของที่ไม่เกี่ยวกับ scene/flow ปลายทาง
- ทดสอบ GLB ที่ export อยู่แล้วก่อนแปลงใหม่
- ตรวจ animation tracks และการขยับจริง ไม่ดูเพียงจำนวน clips
- แชร์ material/texture หลังตรวจความเข้ากันได้ ไม่รวมจากชื่ออย่างเดียว
- ใช้เวลาเกมชุดเดียวกันกับการเคลื่อนที่ การเกิดวัตถุ และการชน
- ใช้ pooling ตามจำนวนที่ต้องใช้จริง และรักษา ownership ตอน dispose
- เก็บค่าตกแต่ง particle ใน Quarks prefab JSON ก้อนเดียว โค้ดมีหน้าที่โหลดและสั่งเล่น
- รายงานผลวัดพร้อมอุปกรณ์และข้อจำกัด ไม่สรุปความร้อนมือถือจากผลบนคอมพิวเตอร์

## ขอบเขตและสิทธิ์

เครื่องมือและเวอร์ชันต่างกันอาจต้องใช้ loader extension หรือ export ใหม่
คู่มือนี้ไม่รับประกันว่า shader, lighting, physics หรือ Unity particle module ทุกชนิดจะย้ายตรงตัวได้
สิทธิ์ใน asset ต้นทางยังเป็นไปตามเจ้าของและ license ของแต่ละชิ้น ไม่ได้ถูกโอนโดยคู่มือนี้

## อ้างอิง

- [Three.js](https://threejs.org/docs/)
- [glTF specification](https://github.com/KhronosGroup/glTF)
- [three.quarks](https://github.com/Alchemist0823/three.quarks)
- [Needle Engine](https://engine.needle.tools/docs/)