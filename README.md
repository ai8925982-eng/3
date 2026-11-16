# 🎵 AI Music Generator (Windows)

โปรแกรมสร้างเพลงอัตโนมัติ ทำงานออฟไลน์บน Windows

| ตัวเลือก | ค่าที่ใช้ |
|---|---|
| แพลตฟอร์ม | Windows 10/11 (รันด้วย Python หรือ build เป็น `.exe`) |
| การสร้างดนตรี | `audio_midi` – ได้ทั้งไฟล์เสียง `.wav` และไฟล์ `.mid` |
| เสียงร้อง | `preserve` – เสียงร้องของคุณ **ไม่ถูกแก้ไข** ดนตรีถูกสร้างให้เข้ากับเสียงร้อง |
| แนวเพลง | `all` – 15 แนว (pop, rock, hiphop, edm, house, lofi, jazz, blues, rnb, country, reggae, latin, classical, ambient, metal) หรือสุ่มอัตโนมัติ |

> **หมายเหตุสำคัญ:** นี่คือ *ต้นแบบ (prototype)* ที่ใช้ทฤษฎีดนตรี + กฎของแต่ละแนว + ซินธ์ที่เขียนเอง
> ไม่ได้ใช้โมเดล AI สร้างเสียงสำเร็จรูป ผลลัพธ์เป็นเสียงสังเคราะห์พื้นฐาน เหมาะเป็นสเก็ตช์/แบ็คกิ้งแทร็ก/ไฟล์ MIDI
> เพื่อนำไปโหลดซาวด์ที่ดีกว่าใน DAW ต่อ ส่วนการต่อยอดไปโมเดล AI จริง ดูหัวข้อ [แผนต่อยอด](#-แผนต่อยอด)

---

## 📁 โครงสร้างโปรเจกต์

```
ai-music-generator/
├─ run_app.py              # จุดเริ่มโปรแกรม (ไม่มีอาร์กิวเมนต์ = เปิด GUI, มี = CLI)
├─ setup_windows.bat       # ติดตั้งครั้งแรก (สร้าง .venv + ติดตั้งแพ็กเกจ)
├─ run_gui.bat             # เปิดหน้าต่างโปรแกรม
├─ run_cli.bat             # รันแบบคำสั่ง
├─ build_exe.bat           # build เป็น .exe บนเครื่องตัวเอง
├─ push_to_github.bat      # อัปโหลดทั้งโปรเจกต์ขึ้น GitHub
├─ requirements.txt        # numpy (จำเป็น) + soundfile (ไม่บังคับ)
├─ requirements-dev.txt    # สำหรับทดสอบ / build
├─ pyproject.toml
├─ src/aimusicgen/
│  ├─ theory.py            # สเกล คอร์ด ลำดับคอร์ด
│  ├─ genres.py            # โปรไฟล์ 15 แนวเพลง (BPM, สเกล, กลอง, เบส, คอร์ด, เมโลดี้, เสียง)
│  ├─ composer.py          # สร้างเพลง: ท่อน intro/verse/chorus/outro + กลอง/เบส/คอร์ด/ลีด
│  ├─ midi_io.py           # เขียน/อ่านไฟล์ MIDI (ไม่ต้องใช้ไลบรารีเสริม)
│  ├─ synth.py             # ซินธ์ numpy เรนเดอร์โน้ต → เสียงสเตอริโอ
│  ├─ vocal.py             # วิเคราะห์เสียงร้อง (BPM, คีย์) โดยไม่แตะต้องเสียงร้อง
│  ├─ mixer.py             # มิกซ์ ducking ลิมิตเตอร์ (ประมวลผลเฉพาะดนตรี ไม่ใช่เสียงร้อง)
│  ├─ audio_io.py          # อ่าน/เขียน WAV (16/24-bit)
│  ├─ pipeline.py          # ท่อประมวลผลครบวงจร
│  ├─ cli.py               # คำสั่งในเทอร์มินัล
│  └─ gui.py               # หน้าต่าง Tkinter
├─ tests/test_pipeline.py  # เทสต์อัตโนมัติ (21 เคส)
└─ .github/workflows/
   ├─ ci.yml               # รันเทสต์บน Windows ทุกครั้งที่ push
   └─ build-windows.yml    # build AIMusicGenerator.exe + แนบ Release
```

## ⚙️ ทำงานอย่างไร

```
(เสียงร้อง .wav) ──► vocal.py: หา BPM + คีย์ ──┐
                                               ▼
ตั้งค่า (แนว/BPM/คีย์/ความยาว) ──► composer.py ──► โน้ต (Drums/Bass/Chords/Lead)
                                               │
                              ┌────────────────┴───────────────┐
                              ▼                                ▼
                        midi_io.py → song.mid            synth.py → ดนตรีสเตอริโอ
                                                               │
                  เสียงร้องต้นฉบับ (ไม่แก้ไข) ─────────────────► mixer.py (ducking + มิกซ์)
                                                               ▼
                                                    song.wav + song_accompaniment.wav
```

**เสียงร้องแบบ preserve:** โปรแกรมโหลดไฟล์เสียงร้อง วิเคราะห์ BPM/คีย์เพื่อสร้างดนตรีให้เข้ากัน
แล้วนำเสียงร้องเดิมมาวางทับ *ตามต้นฉบับ* (ปรับได้แค่ระดับความดังด้วย `--vocal-gain` ค่าเริ่มต้น 0 dB = ไม่แตะ)
ดนตรีจะลดเสียงลงเล็กน้อยตอนมีเสียงร้อง (`--duck`) ได้ไฟล์ `song_accompaniment.wav` แยกให้ด้วย

ข้อจำกัดที่ควรรู้:
- BPM ที่ตรวจจับได้เป็นค่าประมาณ ถ้าเสียงร้องไม่มีจังหวะชัด (เช่น ร้องอิสระ) ให้กำหนด `--bpm` เอง
- คีย์ที่ตรวจจับอาจได้คีย์ขนาน/สัมพันธ์ (เช่น C major ↔ A minor) ซึ่งใช้โน้ตชุดเดียวกัน จึงยังเข้ากันได้
- รองรับ `.wav` ได้เลย ส่วน mp3/flac/m4a ต้องติดตั้ง `soundfile` หรือ `ffmpeg` เพิ่ม
- เสียงร้องที่จะ preserve ควรเป็นเสียงร้องเปล่า (acapella) ถ้าใส่เพลงเต็ม โปรแกรมจะไม่แยกดนตรีเดิมออกให้

---

## 🚀 วิธีรันบน Windows (ไม่ผ่าน GitHub)

1. ติดตั้ง **Python 3.10 ขึ้นไป** จาก https://www.python.org/downloads/ (ตอนติดตั้งติ๊ก **Add python.exe to PATH**)
2. ดาวน์โหลดโปรเจกต์ (ปุ่ม **Code → Download ZIP**) แล้วแตกไฟล์
3. ดับเบิลคลิก `setup_windows.bat` (ทำครั้งเดียว)
4. ดับเบิลคลิก `run_gui.bat` → เปิดหน้าต่างโปรแกรม
5. เลือกแนวเพลง (`all` = สุ่ม) → เลือกโหมด (Audio + MIDI) → (ถ้ามี) เลือกไฟล์เสียงร้อง → กด **Generate**
6. ไฟล์จะอยู่ที่โฟลเดอร์ `Music\AIMusicGenerator` (ปุ่ม **Open folder**)

### แบบคำสั่ง (CLI)

```bat
:: สร้างเพลงสุ่มแนว ได้ทั้ง .wav และ .mid
run_cli.bat

:: เลือกแนว ความยาว และ seed (seed เดิม = เพลงเดิม)
run_cli.bat -g lofi --bars 32 --seed 123 -o output -n my_song

:: คงเสียงร้องเดิม + สร้างดนตรีรอบเสียงร้อง (ดึง BPM/คีย์จากเสียงร้องอัตโนมัติ)
run_cli.bat --vocal "C:\Users\me\Music\my_vocal.wav" -g pop -o output -n with_vocal

:: ได้เฉพาะ MIDI / เฉพาะเสียง / แยกสเต็ม
run_cli.bat -g jazz --mode midi
run_cli.bat -g edm --mode audio --stems --bits 24
```

| ตัวเลือก | ความหมาย |
|---|---|
| `-g, --genre` | `all` หรือชื่อแนว (`--list-genres` เพื่อดูทั้งหมด) |
| `--mode` | `audio_midi` (ค่าเริ่มต้น) / `audio` / `midi` |
| `--vocal FILE` | ไฟล์เสียงร้องที่ต้องการคงไว้ |
| `--bars N` | ความยาวเป็นห้อง (ค่าเริ่มต้น 32) |
| `--bpm`, `--key`, `--scale` | กำหนดเอง (ไม่ใส่ = สุ่มตามแนว / ตามเสียงร้อง) |
| `--seed N` | ใช้ซ้ำเพื่อได้เพลงเดิม |
| `--lead on/off` | เมโลดี้ลีดที่โปรแกรมสร้าง (ปิดอัตโนมัติเมื่อมีเสียงร้อง) |
| `--duck DB` | ลดเสียงดนตรีตอนมีเสียงร้อง (ค่าเริ่มต้น 4 dB, 0 = ปิด) |
| `--vocal-gain DB` | ปรับความดังเสียงร้อง (0 = ไม่แตะ) |
| `--stems` | ส่งออกไฟล์แยกเครื่องดนตรี |
| `--bits 16/24` | ความละเอียด WAV |

ไฟล์ที่ได้: `song.mid`, `song.wav`, `song_accompaniment.wav`, `song_info.json` (และ `song_stem_*.wav`)

---

## 🐙 วิธีนำขึ้น GitHub และ build .exe (แบบละเอียด)

> **ถ้า Actions ขึ้นกากบาทแดง ❌ ให้ทำตามหัวข้อนี้ตั้งแต่ต้น** สาเหตุที่พบบ่อยที่สุดคือใน repo มีแค่ไฟล์ `build-windows.yml`
> แต่ไม่มีซอร์สโค้ด (`run_app.py`, โฟลเดอร์ `src`) ทำให้ build ไม่มีอะไรให้ทำ ต้องอัปโหลดโปรเจกต์ **ทั้งโฟลเดอร์**

### ขั้นที่ 1 – เตรียมไฟล์
1. แตกไฟล์ `ai-music-generator.zip` ลงเครื่อง (คลิกขวา → Extract All)
2. เปิดโฟลเดอร์ `ai-music-generator` ต้องเห็น `run_app.py`, `README.md`, โฟลเดอร์ `src`, `tests`, `.github` ในระดับเดียวกัน

### ขั้นที่ 2 – อัปโหลดขึ้น Repository (เลือกทางใดทางหนึ่ง)

**ทางที่ 1: ใช้สคริปต์ (ง่ายสุด ถ้าติดตั้ง Git แล้ว)**
1. ติดตั้ง Git for Windows จาก https://git-scm.com/download/win (กด Next ไปเรื่อย ๆ)
2. เปิด PowerShell หรือ Command Prompt ในโฟลเดอร์โปรเจกต์ แล้วรัน (เปลี่ยน URL เป็น repo ของคุณ):
   ```bat
   push_to_github.bat https://github.com/ชื่อผู้ใช้/ชื่อrepo.git
   ```
3. ครั้งแรก Git จะเปิดหน้าต่างให้ล็อกอิน GitHub ให้ล็อกอินให้เสร็จ
4. สคริปต์ใช้ `--force` จึงเขียนทับไฟล์ที่เคยสร้างผ่านเว็บ (เช่น `build-windows.yml` ตัวเก่า)

**ทางที่ 2: อัปโหลดผ่านหน้าเว็บ (ไม่ต้องลง Git)**
1. เข้า repo → ปุ่ม **Add file → Upload files**
2. เปิดโฟลเดอร์ `ai-music-generator` ใน File Explorer → กด **Ctrl+A** เลือกทุกอย่างข้างใน
   (ต้องรวมโฟลเดอร์ `.github`, `src`, `tests`) แล้ว **ลากทั้งหมดไปวางบนหน้าเว็บ**
3. รอจนอัปโหลดครบ แล้วกด **Commit changes**
4. เช็กหน้า **Code** ของ repo ว่าเห็น `run_app.py`, `src`, `tests`, `.github` ครบ
   ถ้ามีไฟล์ `build-windows.yml` เดิมอยู่แล้ว การอัปโหลดจะเขียนทับให้เอง
5. ถ้าโฟลเดอร์ย่อยไม่ขึ้น (เว็บบางครั้งไม่รับโฟลเดอร์ซ้อนจำนวนมาก) ให้ใช้ทางที่ 1 แทน

### ขั้นที่ 3 – รัน Build .exe
- หลังอัปโหลดเสร็จ workflow **Build Windows EXE** จะรันเองอัตโนมัติ
- หรือสั่งเอง: แท็บ **Actions** → เลือก **Build Windows EXE** ทางซ้าย → **Run workflow** → **Run workflow**
- รอประมาณ 3–6 นาที จนเห็นเครื่องหมายถูกสีเขียว ✅

### ขั้นที่ 4 – ดาวน์โหลดโปรแกรม
1. คลิกชื่อรอบที่รัน (แถวบนสุด)
2. เลื่อนลงล่างสุดหัวข้อ **Artifacts** → คลิก **AIMusicGenerator-windows** เพื่อโหลดไฟล์ zip
3. แตก zip จะได้
   - `AIMusicGenerator.exe` – หน้าต่างโปรแกรม ดับเบิลคลิกใช้ได้เลย
   - `AIMusicGenerator-cli.exe` – แบบคำสั่ง เช่น `AIMusicGenerator-cli.exe -g rock --bars 16`

> Windows SmartScreen อาจเตือน "Windows protected your PC" เพราะไฟล์ไม่ได้ลงนามดิจิทัล → กด **More info → Run anyway**

### ถ้ายังขึ้น ❌ แดง ทำอย่างไร
1. คลิกที่รอบที่ล้มเหลว → คลิกงาน **build** ทางซ้าย
2. หาขั้นตอนที่มีกากบาทแดง กดขยายอ่านบรรทัดสุดท้ายของข้อความ error
3. ตารางสาเหตุที่พบบ่อย

| ข้อความ / ขั้นตอนที่ล้ม | สาเหตุ | วิธีแก้ |
|---|---|---|
| **Check project files are present** → "Project files are missing" | ยังไม่ได้อัปโหลดซอร์สโค้ด | ทำขั้นที่ 2 ให้ครบ |
| `Workflow does not exist` / ไม่มีแท็บ Run workflow | ไฟล์ workflow ไม่อยู่ที่ `.github/workflows/` | ตรวจชื่อโฟลเดอร์ให้ตรง ต้องมีจุดนำหน้า `.github` |
| **Smoke test the CLI exe** ล้ม | โปรแกรมรันไม่ผ่านบน Windows | คัดลอก error มาให้ผมแก้ต่อ |
| Run tests ขึ้นเตือนเหลือง ⚠️ | เทสต์ไม่ผ่านแต่ build ต่อได้ | ใช้ exe ได้ แต่ส่ง log มาให้ตรวจ |
| Release ไม่ขึ้น | ต้องสร้างแท็ก `v*` | ดูขั้นต่อไป |

### (ไม่บังคับ) ออก Release ให้คนอื่นดาวน์โหลด
```bash
git tag v0.1.0
git push origin v0.1.0
```
workflow จะแนบ `AIMusicGenerator-windows.zip` ไว้ในหน้า **Releases** ให้อัตโนมัติ

### Build บนเครื่องตัวเอง (ไม่ผ่าน GitHub)
ดับเบิลคลิก `build_exe.bat` → ได้ไฟล์ในโฟลเดอร์ `dist`

---

## 🧪 ทดสอบ

```bat
.venv\Scripts\activate
pip install -r requirements-dev.txt
python -m pytest -q
```

เทสต์ครอบคลุม: ทุกแนวสร้างเพลงได้, seed เดิมได้เพลงเดิม, MIDI เขียน/อ่านกลับตรงกัน, ไฟล์เสียงออกถูกต้อง,
และ **เสียงร้องถูกคงไว้จริง** (วัดจากการเทียบมิกซ์กับไฟล์เสียงร้องต้นฉบับ)

## 🛠 แก้ปัญหาเบื้องต้น

| อาการ | วิธีแก้ |
|---|---|
| `python` is not recognized | ติดตั้ง Python ใหม่แล้วติ๊ก *Add to PATH* หรือใช้ `py -m venv .venv` |
| GUI ไม่เปิด / ไม่มี tkinter | ติดตั้ง Python จาก python.org (ตัวติดตั้งมี tkinter มาให้ ต้องไม่ถอดติ๊ก *tcl/tk*) |
| อ่านไฟล์ mp3 ไม่ได้ | `pip install soundfile` หรือติดตั้ง ffmpeg หรือแปลงเป็น WAV ก่อน |
| ดนตรีไม่เข้ากับเสียงร้อง | กำหนด `--bpm` / `--key` / `--scale` เองให้ตรงกับเพลง |
| exe โดน antivirus บล็อก | พบบ่อยกับ PyInstaller ให้ใช้วิธีรันด้วย Python แทน หรือ build เองบนเครื่อง |

## 🔭 แผนต่อยอด

- ใช้โมเดล AI จริงสร้างเสียง: **MusicGen / Stable Audio** (ต้องมี GPU + PyTorch) แทนซินธ์ในโมดูล `synth.py`
- แยกเสียงร้องจากเพลงเต็มด้วย **Demucs** ก่อนเข้าโหมด preserve
- เมโลดี้จากเสียงร้อง: ใช้ **basic-pitch / CREPE** ถอดโน้ตแล้วสร้างคอร์ดให้ตรงกับทำนอง
- โหลด SoundFont (`.sf2`) ผ่าน **FluidSynth** เพื่อให้เสียงเครื่องดนตรีสมจริงขึ้น
- ปรับจังหวะเสียงร้องให้ตรงกริด (time-align / beat-sync) และส่งออกเป็น VST3 (โครงสร้างเดียวกับโปรเจกต์ JUCE)

## 📄 License
MIT – ดู [LICENSE](LICENSE)
