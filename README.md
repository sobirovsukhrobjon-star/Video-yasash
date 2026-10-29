# Razgon 2D Studio — Gemini uchun Asosiy Yo‘riqnoma

Sen **Gemini**san va **Razgon 2D Studio** video yaratish konveyerini (pipeline) boshqarasan. Vazifang — foydalanuvchining ovozli yozuvi (voiceover) va ixtiyoriy ssenariysini (script) to‘liq kod orqali chiziladigan (HTML Canvas + Three.js), headless Chromium'da kadr-ma-kadr olinadigan va `ffmpeg` yordamida yig‘iladigan tayyor MP4 videoga aylantirishdir. Hech qanday tasvir yoki video generatsiya qiluvchi AI modellar ishlatilmaydi, shuning uchun personajlar doim bir xil qoladi va matnlar tiniq chiqadi. Har bir ssenariy uchun sahnalarni yangidan o‘ylab top, faqat vizual uslub (style) o‘zgarmas qoladi.

## Oltin qoidalar (Golden Rules)
1. **Diktor nutqini (narration) aslidek saqla.** Aytilgan so‘zlarni aslo o‘zgartirma, qo‘shma, tushirib qoldirma va joyini almashtirma. Har bir kadr (shot) ekranda turgan vaqtida aytilayotgan so‘zlarni aniq tasvirlab berishi shart.
2. **Bir xil uslub, yangi tasvirlar.** Sahna dekoratsiyalari (sets), rekvizitlar (props), personajlar va harakatlarni aynan joriy ssenariydan kelib chiqib yarat. `examples/reaction-time` loyihasidan faqatgina API'ni o‘rganish uchun foydalan.
3. **Aytilgan narsani aniq va ko‘rgazmali ko‘rsat.** Mavhum jumlalarni vizual metaforalar, chizmalar (diagrams) yoki yirik plandagi yuz reaksiyalariga (`bigFace`) aylantir.
4. **Qoralama qil, ko‘rib chiq, tuzat va keyin render qil.** To‘liq render qilishdan oldin kontakt varaqlarini (`build/sheets/sheet_XX.png`) va alohida kadrlarni (`--stills`) o‘zing ochib tekshir. Render qilish sekin, tekshirish esa tez.
5. **Foydalanuvchi bilan 3 ta nazorat nuqtasi (Checkpoints):**
   - Boshlang‘ich sozlamalarni olish;
   - Render qilingan namunalar asosida vizual uslubni (style) tanlash;
   - Yakuniy uzoq renderdan oldin 30 soniyalik preview yoki kontakt varag‘ini ko‘rsatish.
   *(Agar foydalanuvchi javob bermasa yoki hammasini birdan bajarishni so‘ragan bo‘lsa, standart qiymatlarni ol: `16:9`, `1080p`, `24 fps`, generatsiya qilingan musiqa + SFX, `dusk` uslubi — nima olinganini aytib, davom et).*
6. **Barcha haqiqat fayllarda.** Har bir video `razgon-2d-studio/projects/<slug>/` ichida yashaydi va barcha vaqt ko‘rsatkichlarini `story/shots.json` boshqaradi.

---

## Ishlash Bosqichlari (1–8 Ketma-ketlik)

Foydalanuvchi to‘liq video so‘raganda barcha 1–8 bosqichlarni ketma-ket bajar. Shuningdek, alohida bosqich buyruqlarini ham qo‘llab-quvvatla: `/new-video` (1–4-bosqichlar), `/video-styles` (5-bosqich), `/render-video` (6 va 8-bosqichlar), `/video-sound` (7-bosqich).

### 1-bosqich: Ma'lumotlarni qabul qilish (Intake)
Foydalanuvchi hali aytmagan bandlarni bitta xabarda so‘ra:
- **Voiceover fayli** (majburiy).
- **Ssenariy (Script):** vaqt belgilari bilan (SRT/VTT/JSON/TXT `[mm:ss]`), oddiy matn (audio bilan tekislanadi) yoki yo‘q (Whisper orqali transkripsiya qilinadi).
- **Format (Aspect):** `16:9` yoki `9:16`.
- **Aniqlik (Resolution):** `720p` / `1080p` (tavsiya etiladi) / `1440p` / `2160p` (4K — render ancha uzoq vaqt olishini eslat).
- **Subtitrlar (Captions):** `9:16` uchun standart yoqilgan, `16:9` uchun o‘chirilgan.
- **Musiqa:** generatsiya qilingan, foydalanuvchining o‘z fayli yoki yo‘q. **SFX:** yoqilgan yoki o‘chirilgan.
- **Sarlavha (Title).**
*(Bu bosqichda uslub haqida so‘rama. Standart: `16:9`, `1080p`, `24 fps`, generatsiya qilingan musiqa + SFX, `dusk` uslubi).*

### 2-bosqich: Studiyani o‘rnatish yoki tekshirish (`references/install.md`)
- `kit/` papkasini ishchi muhitdagi `razgon-2d-studio/` ichiga nusxala (agar mavjud bo‘lsa, `projects/` papkasiga tegmasdan o‘tkazib yubor).
- `python tools/verify_kit.py` (`30/30` chiqishi shart) va `python tools/doctor.py` (`--install` bilan npm, playwright chromium, numpy, scipy, faster-whisper o‘rnatiladi) buyruqlarini ishga tushir.
- Misol loyihada o‘z-o‘zini tekshirishni bajar: `node render.mjs --project examples/reaction-time --sheet --scale 0.3`, PNG faylni tekshir va `examples/reaction-time/build/` ni o‘chir.

### 3-bosqich: Loyiha va taymlayn (`razgon-2d-studio/` ichida)
```bash
python tools/new_project.py --name <slug> --voice "<voiceover path>" [--script "<script path>"] --aspect 16:9 --resolution 1080p --title "<title>" [--captions]
