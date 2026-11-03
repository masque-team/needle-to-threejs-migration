# Web MiniGame Development Standard V1

**เป้าหมาย:** ให้เกมที่เขียนใหม่และเกมที่ย้ายจาก Unity/Needle ใช้มาตรฐานเว็บเกมปลายทางเดียวกัน แต่ยังไม่กำหนดรูปแบบ GLB/Prefab หรือสร้าง SDK กลาง

> Required = ต้องมีทุกเกม; Optional = เลือกตามลักษณะเกม; Deferred = ไม่แตะใน V1

## 1. Technology Stack

| ส่วน | มาตรฐาน | หมายเหตุ |
| --- | --- | --- |
| ภาษา Gameplay | **TypeScript** | Score, timer, rules, state, inputs |
| UI | **Svelte 5** | standalone ใช้ Svelte + Vite; ไม่บังคับ SvelteKit |
| CSS/Layout | **Tailwind CSS 4** | @tailwindcss/vite + @import 'tailwindcss'; CSS ปกติสำหรับ theme/keyframes |
| Game Rendering | **Three.js** | 2D/2.5D/3D; ไม่บังคับ GLB |
| VFX | **Three.js** / **three.quarks** | ใช้ Quarks เมื่อมี particle ซับซ้อน/เหมาะกับงาน ไม่ต้องลงทุกเกม |
| UI Animation | **CSS** ก่อน, **GSAP** ตามความจำเป็น | สำหรับ sequence/timeline ซับซ้อน; ต้อง cleanup tween |
| Build | **Vite 8** + **Node.js 22.12+** | Node ใช้ development/build; static game ไม่บังคับ backend |
| Config | **JSON + Zod** | โหลดและตรวจ gameplay config ก่อนเริ่ม |
| Testing | **Vitest** สำหรับเกมใหม่ | เกมเก่าที่ใช้ test runner อื่นอยู่ไม่ต้องรีบเปลี่ยน |

เวอร์ชัน dependency ต้องเข้ากันได้จริงและมี package-lock.json ของเกมนั้น ไม่คัดลอก lockfile จากอีกเกม บันทึกข้อยกเว้นหากใช้ baseline ไม่ได้

## 2. Screen Flow (Required)

**Menu -> Gameplay -> Result -> Replay / Home** และต้องมี Loading/Error state ที่จัดการข้อผิดพลาดได้จริง

- MenuScreen.svelte: หน้าก่อนเล่น/เริ่มเกม/กติกา
- GameplayScreen.svelte: Canvas, HUD และ input controls
- ResultScreen.svelte: คะแนน, ชนะ/แพ้, เล่นใหม่, หน้าแรก
- Loading/Error: โหลด assets/config และบอกเหตุผลเมื่อผิดพลาด ไม่ปล่อยหน้าว่าง
- Pause, countdown, character select, level select และ transition เป็น Optional ไม่สร้างไฟล์เปล่าถ้าไม่ได้ใช้
- ชื่อ gameplay phase เฉพาะเกม (เช่น begin, transition) ต่างกันได้ แต่ต้อง map กลับ Screen Flow ชัดเจน
- App.svelte ใช้ประกอบหน้าจอ/เชื่อม state ไม่ใส่ gameplay และ Three.js runtime กองในไฟล์เดียว
- เกม Three.js ใช้ Canvas + Svelte overlay สำหรับ HUD, menu, buttons; หนึ่งแหล่งข้อมูลหลักของ score/time/state
- เมื่อเปลี่ยน Screen พยายาม reuse renderer/scene ถ้าทำได้โดยไม่เพิ่มความซับซ้อน

## 3. Project & Asset Folders

```text
src/
  App.svelte
  main.ts
  app.css
  screens/                 MenuScreen, GameplayScreen, ResultScreen
  components/              HUD, Controls, Dialog (ตามที่ใช้)
  lib/
    game/                  GameSession, rules, state, tests
    three/                 GameScene, renderer, camera, VFX, asset handling
      prefabs/             Optional: TypeScript prefab factory/class ของเกม
    config.ts              Runtime JSON loader + Zod validation
    host.ts                Optional: iframe/Platform bridge
public/
  assets/
    ui/                    Logo, button, HUD, result images
    images/                ภาพทั่วไป
    cards/                 Optional: รูปการ์ด/collection
    models/                GLB และหมวดย่อย characters/environment/items
    textures/              External textures
    audio/
    vfx/
    prefabs/               Optional: ไฟล์ prefab data ที่เกมนั้นใช้จริง
  config/
    game.json              Required
    levels.json            Optional
  generated/               Optional: machine-generated manifests
scripts/                   Optional: scan/import/sync ที่ทำซ้ำได้
package.json
package-lock.json
vite.config.ts
```

Folder เฉพาะเกมเพิ่มได้ตามจริง และไม่ต้องสร้าง folder Optional ที่ไม่ได้ใช้

### Replaceable Assets

- รูปที่คนดูแลต้องเปลี่ยน เช่น การ์ด ให้ใช้ URL/ชื่อไฟล์ที่คงที่และรวมเส้นทางอ้างอิงในที่เดียว เพื่อวาง PNG ชื่อเดิมทับโดยไม่แก้ TypeScript
- ถ้าเพิ่ม/ลบรูปได้แบบ dynamic ให้ใช้ script scan folder สร้าง generated manifest อัตโนมัติ (ตัวอย่าง: Memory Card); ห้ามแก้ generated manifest ด้วยมือ
- Static browser ไม่สามารถ enumerate directory บน server ได้เอง การเพิ่มรูปจึงต้อง sync/manifest และ deploy ผลลัพธ์ตาม workflow
- การแทนไฟล์บน production ต้องตรวจ CDN/browser cache ด้วย ไม่รับประกันว่า overwrite แล้วทุกเครื่องเห็นใหม่ทันที
- อ้างอิง assets ผ่าน import.meta.env.BASE_URL หรือ helper กลาง ไม่ hardcode /assets/ ถ้าจะ deploy ที่ subpath
- แยกต้นฉบับ, asset ที่แก้เอง, และ generated files; เคารพสิทธิ์ของ asset

## 4. Game Config (Required)

ต้องมี public/config/game.json ที่ตั้ง **ค่า gameplay ที่ผู้ดูแลควรปรับได้** เช่น เวลา, คะแนน, speed, difficulty, spawn, lives

```json
{
  "schemaVersion": 1,
  "rules": {
    "startTimeSeconds": 40,
    "matchScore": 100,
    "maxPairs": 8
  }
}
```

- นี่เป็นตัวอย่างสำหรับเกมการ์ด **ไม่ใช่ schema บังคับร่วมทุกเกม**; เกมวิ่งหรือเกมตีไม้ใช้ rules ต่างกันได้
- กำหนดหน่วย, min/max, type และ validate ด้วย Zod ก่อนสร้าง GameSession; config เสียหรือหายต้องแสดง Error ชัดเจน ไม่ silently fallback
- สำหรับการปรับหลัง deploy ให้ fetch config/game.json ตอนเปิดเกม (พิจารณา cache:no-store และ CDN cache policy) เพื่อแก้ค่าบน server แล้ว reload ได้โดยไม่ rebuild JS
- ค่าของ Renderer ที่ไม่เปิดให้ผู้ดูแลปรับ ไม่จำเป็นต้องย้ายเข้า Config ทั้งหมด
- ไม่เก็บ credentials, secret หรือ token ใน public assets, config หรือ VITE_* เพราะอ่านได้จาก browser

## 5. Architecture & Lifecycle (Required)

- แยก GameSession/Game Rules/State จาก Svelte UI และ Three.js Rendering เพื่อทดสอบ Logic โดยไม่เปิด WebGL เท่าที่เหมาะสม
- Gameplay actions เปลี่ยน Session ก่อน แล้ว UI/Renderer sync กับ state/event เดียวกัน; ไม่เก็บคะแนนซ้ำในหลายจุด
- Renderer, GLB/texture, event listener, RAF, interval, sound, tween ต้องมี owner และ cleanup ตอน unmount/dispose; ทดสอบ Restart หลายรอบ
- Load/reuse Asset โดยไม่ fetch หรือ clone ทุก frame; แชร์ resource เฉพาะที่ compatible; Pooling/Instancing เป็น Optional เมื่อมีเหตุผล
- Mobile Web First ตามเป้าหมายเกม: touch, responsive/portrait, 100dvh, safe-area, resize, DPR budget, page visibility/pause
- การเชื่อม Game Zone/iframe เป็น adapter แยกเมื่อใช้จริง; validate origin/messages และไม่ปนกับ game rules
- ทดสอบ production build และ subpath hosting ถ้ามีการฝังใต้ URL ย่อย

## 6. Deferred — ห้ามชนงาน GLB / Prefab

**ยังไม่บังคับ** GLB export format, scene manifest, Prefab JSON/Component Schema, CharacterPrefab/ObstaclePrefab Shared Runtime, nested prefabs, ECS, Physics Engine, Animation Mapping หรือ Mini Studio

เกมใช้ TypeScript Factory/Class, GLB Scene หรือ JSON Prefab ตามความเหมาะสม **โดยจัดโค้ดใน src/lib/three/ และข้อมูลใน public/assets/ ให้ถูกตำแหน่ง** เพื่อเปลี่ยนระบบภายหลังได้ง่าย ไม่บังคับรีแฟกเตอร์เกมที่ทำมาแล้วเพียงเพื่อให้มี Prefab รูปแบบใหม่

## 7. Definition of Done

- [ ] Stack ตรง baseline หรือมีเหตุผลข้อยกเว้น
- [ ] Menu -> Gameplay -> Result -> Restart/Home และ Loading/Error ทำงานจริง
- [ ] Game Logic / UI / Rendering แยกหน้าที่
- [ ] Assets ถูก folder และ replace รูปเดิมได้; ถ้า dynamic ให้มี sync/manifest
- [ ] public/config/game.json โหลดและตรวจด้วย Zod; ปรับค่าที่ระบุได้โดยไม่แก้ Source
- [ ] dispose/restart ไม่มี listener หรือ resource รั่วที่ตรวจพบ
- [ ] npm run check, npm test, npm run build ผ่าน หรือบันทึกเหตุผลที่ไม่ได้รัน
- [ ] Mobile input, viewport, safe-area และ deploy path ผ่านการตรวจบนอุปกรณ์/สภาพแวดล้อมที่เข้าถึงได้
- [ ] ไม่เปลี่ยน GLB/Prefab Architecture ที่อยู่นอก scope

## 8. Usage

- **New game:** ใช้ minigame-standard Skill + เอกสารนี้ก่อนสร้างเกม
- **Needle migration:** ใช้ needle-migration สำหรับการย้ายต้นฉบับ และใช้เอกสารนี้ตรวจผลลัพธ์ปลายทาง
- **Existing games:** ปรับตามมาตรฐานเมื่อมีงานพัฒนาต่อ ไม่ต้องรื้อในครั้งเดียว

ตัวอย่างการตัดสินใจมาจาก Timber Pets, Aniimo Run และ Memory Card; ไม่มีเกมใดเป็น Shared Runtime SDK ที่ทุกเกมต้อง copy ทั้งชุด
