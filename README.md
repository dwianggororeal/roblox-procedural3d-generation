# Roblox Procedural 3D Modeling & Anti-Slop Standards

[![Roblox Engine](https://img.shields.io/badge/Roblox-Engine-00A2FF?logo=roblox&logoColor=white)](https://roblox.com)
[![Luau](https://img.shields.io/badge/Language-Luau-00A2FF?logo=lua&logoColor=white)](https://luau-lang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![AI Agent Skill](https://img.shields.io/badge/Agent%20Skill-Compatible-brightgreen)](#cara-integrasi-ke-ai-agent)

Standar baku dan panduan teknis pembuatan model 3D prosedural berbasis kode (Luau) di Roblox Studio. Dirancang khusus untuk menghasilkan karakter, makhluk (*creatures/bosses*), senjata, dan prop lingkungan dengan estetika premium bebas dari visual murahan (*anti-slop*), anti-*z-fighting*, anatomi proporsional, dan rig motorik R15 yang stabil tanpa *weld-fighting*.

---

## 📑 Daftar Isi

- [Filosofi & Pilar Utama](#-filosofi--pilar-utama)
- [Struktur Dokumen Repository](#-struktur-dokumen-repository)
- [Ringkasan Standar Baku](#-ringkasan-standar-baku)
  - [1. Geometri Mutlak: Box-Only](#1-geometri-mutlak-box-only)
  - [2. Alur 3-Fase: Modeling Dulu, Rigging Belakangan](#2-alur-3-fase-modeling-dulu-rigging-belakangan)
  - [3. Pose Awal Baku: A-Pose & Natural Inward Curl](#3-pose-awal-baku-a-pose--natural-inward-curl)
  - [4. Koordinat Lokal Roblox (-Z Front Rule)](#4-koordinat-lokal-roblox--z-front-rule)
  - [5. Eliminasi Z-Fighting (Stepped Depth)](#5-eliminasi-z-fighting-stepped-depth)
  - [6. Anti-Slop & Pencegahan Over-Bloom In-Game](#6-anti-slop--pencegahan-over-bloom-in-game)
- [Katalog 11 Pitfalls Umum](#-katalog-11-pitfalls-umum)
- [Snippet Kode & Validasi Cepat](#-snippet-kode--validasi-cepat)
- [Cara Integrasi ke AI Agent](#-cara-integrasi-ke-ai-agent)
- [Lisensi](#-lisensi)

---

## 🏛 Filosofi & Pilar Utama

1. **Zero-Slop Aesthetic:** Menghindari tampilan simulator/obby generik tahun 2017 (spam neon terang, billboard text raksasa tembus dinding, dan model ratusan part balok lepas yang lag).
2. **Deterministic Box-Only Geometry:** 100% bentuk dibangun dari part `Block` yang diskalakan, dirotasi, dan ditumpuk berundak. Tanpa `Ball`, `Cylinder`, `Wedge`, atau generator mesh instan.
3. **Rig-Friendly R15 Architecture:** Detail estetika dipisahkan secara hierarkis ke dalam 15 Folder R15 baku, dipasang dengan `WeldConstraint`, dan hanya 15 part solid motorik utama yang disambung `Motor6D`.
4. **Fail-Fast Validation:** Setiap iterasi model wajib melewati fungsi inspeksi kode sebelum dinyatakan selesai.

---

## 📂 Struktur Dokumen Repository

| File | Deskripsi |
|---|---|
| [**`SKILL.md`**](SKILL.md) | **Dokumen Master Skill**: Berisi seluruh aturan geometri box-only, alur 3-fase, rotasi A-pose matematis, 15 folder R15, sistem koordinat lokal, formula eliminasi Z-fighting, katalog 11 kesalahan umum (*pitfalls post-mortem*), checklist pra/pasca generasi, dan formula Unlit Neon anti-bloom. |
| [**`antislop_standart_3d.md`**](antislop_standart_3d.md) | **Standar Visual 3D**: Berisi larangan keras (*what to avoid*) dan praktik terbaik lingkungan 3D (larangan neon spam pada part non-lampu, batasan `MaxDistance` pada `BillboardGui`, pencegahan weld-fighting, palet material fisik realistis, dan pencahayaan atmosferik). |
| [**`LICENSE`**](LICENSE) | Lisensi resmi MIT. |

---

## 📌 Ringkasan Standar Baku

### 1. Geometri Mutlak: Box-Only
Semua part wajib memenuhi:
```lua
part.ClassName == "Part"
part.Shape == Enum.PartType.Block
```
- Dilarang memakai `Part.Shape = Ball` atau `Cylinder`.
- Dilarang memakai `WedgePart`, `CornerWedgePart`, `MeshPart`, `UnionOperation`, atau `SpecialMesh`.
- Bentuk melengkung, taring, siluet otot, dan tanduk wajib disusun dari balok-balok kecil bertingkat.

### 2. Alur 3-Fase: Modeling Dulu, Rigging Belakangan
1. **Fase 1 (Pemodelan Statis):** Seluruh part dibuat `Anchored = true`, tanpa sendi `Motor6D`. Fokus 100% pada proporsi, wajah, pakaian, dan pencegahan Z-fighting. Part langsung dimasukkan ke dalam 15 folder anatomi R15 (*First-Pass Direct Parenting*).
2. **Fase 2 (Review Visual):** Validasi visual dari berbagai sudut tanpa batasan motorik.
3. **Fase 3 (Rigging & Kalibrasi):** Pasang `Motor6D` dengan *Seamless Socket Overlap*, pasang `AnimationController` (khusus creature/monster), setel `Anchored = false`, dan hitung `HipHeight` secara matematis.

### 3. Pose Awal Baku: A-Pose & Natural Inward Curl
- **Shoulder Flare:** Sudut lengan rileks ~20° - 24° dari torso (`-math.rad(24)` kiri, `+math.rad(24)` kanan). Siku dan pergelangan lurus netral.
- **Natural Inward Palm:** Telapak tangan dirotasi 90° menghadap ke dalam (memeluk paha/torso), bukan datar menghadap depan seperti papan dayung.
- **Inward Finger Curl:** Jari tangan kiri memakai `+math.rad(20°)` dan tangan kanan `-math.rad(20°)` pada sumbu Z untuk mengimbangi kemiringan lengan. Jempol membungkus rapi di sisi luar (*Wrap-Over Thumb*).
- **Inward Tusks:** Taring melengkung ke atas dan ke dalam memeluk rahang, bukan mencuat keluar.

### 4. Koordinat Lokal Roblox (-Z Front Rule)
| Arah Anatomi Karakter | Sumbu Lokal Roblox | Keterangan |
|---|---|---|
| **DEPAN (*Front*)** | **`-Z`** (`LookVector`) | Wajah, dada, perut, taring, sabuk depan, jari kaki |
| **BELAKANG (*Back*)** | **`+Z`** | Punggung, jubah belakang, sayap, ekor |
| **KANAN (*Right*)** | **`+X`** (`RightVector`) | Lengan kanan, kaki kanan |
| **KIRI (*Left*)** | **`-X`** | Lengan kiri, kaki kiri |
| **ATAS (*Up*)** | **`+Y`** (`UpVector`) | Kepala, tanduk, mahkota, bahu |
| **BAWAH (*Down*)** | **`-Y`** | Rahang, kaki, telapak kaki |

> ⚠️ **Hukum Mutlak:** Jangan pernah memakai `+Z` untuk fitur depan. Nilai `+Z` menaruh objek di punggung karakter!

### 5. Eliminasi Z-Fighting (Stepped Depth)
- **Part selevel:** Wajib *Butted Edge* (tidak saling menembus batas samping).
- **Part dekorasi/kain:** Terapkan *Stepped Layering Protocol*:
  - Kulit / Otot: `Z = -1.70`
  - Jubah / Stola: `Z = -2.05` (offset +0.35)
  - Kalung / Aksesoris: `Z = -2.28` (offset +0.23)
  - Permata / Ornamen: `Z = -2.35` (offset +0.07)
- **Sendi Bahu:** Jubah pundak di `UpperTorso` tidak boleh melebar menembus lengan baju di `UpperArm`.

### 6. Anti-Slop & Pencegahan Over-Bloom In-Game
- Dilarang memberikan efek `Neon` pada lantai, dinding, atau tiang yang bukan sumber cahaya nyata.
- **Formula Unlit 2D Silhouette Anti-Bloom:**
  - Material: `Enum.Material.Neon`
  - `Reflectance = 0`, `CastShadow = false`
  - Warna: **`Color3.fromRGB(165, 165, 160)` s.d `(175, 172, 168)`** (di bawah ambang batas Gaussian bloom HDR Roblox ~0.8). Hasilnya tetap berupa siluet putih pekat, tetapi 0% silau saat *playtest*.
- `BillboardGui` wajib memiliki `MaxDistance` terbatas (25–40 studs), jangan dibiarkan `0` (tak terhingga).

---

## ⚠️ Katalog 11 Pitfalls Umum

1. **Sumbu Terbalik:** Mengira depan adalah `+Z`.
2. **Tusk Menutupi Mata & Tanduk Menembus Topi:** Tidak mematuhi zona vertikal wajah.
3. **Z-Fighting pada Pakaian & Sendi:** Menempelkan part kain pada depth Z identik atau menembus soket lengan.
4. **Loose Parts Tanpa Solid Rig:** 100+ part lepas dengan ratusan WeldConstraint memicu lag di ACE.
5. **Reorganisasi Folder Pasca-Pembuatan:** Part kecil tercecer saat parenting belakangan.
6. **Auto-Snap Distortion:** Memakai `Humanoid` pada monster proporsi custom alih-alih `AnimationController`.
7. **Karakter Melayang/Tenggelam:** Tidak mengkalibrasi `HipHeight`.
8. **Edit Langsung Tanpa Backup:** Mengedit tanpa membuat clone duplikat cadangan.
9. **Celah Bolong Sendi (*Socket Gap*):** Formula Motor6D tidak menyertakan penetrasi overlap 0.05–0.1 studs.
10. **Jari Miring Keluar (*Inverted Finger Flare*):** Tanda rotasi Z jari salah mengimbangi sudut A-Pose lengan.
11. **Kaki Melayang pada Animasi Idle:** Menggeser panggul tanpa mengunci `LeftFoot` dan `RightFoot` di `CFrame.identity`.

---

## 💻 Snippet Kode & Validasi Cepat

### 1. Fail-Fast Box-Only Assertion
```lua
local function assertBoxOnly(model: Model)
    local violations = 0
    for _, inst in ipairs(model:GetDescendants()) do
        if inst:IsA("BasePart") then
            if not inst:IsA("Part") or inst.Shape ~= Enum.PartType.Block then
                violations += 1
                warn("[VIOLATION] Non-box part:", inst:GetFullName(), inst.ClassName, inst.Shape)
            end
        end
    end
    assert(violations == 0, string.format("Box-only violation count: %d", violations))
end
```

### 2. Kalibrasi HipHeight Otomatis
```lua
local function calibrateHipHeight(model: Model, humanoid: Humanoid, hrp: BasePart)
    local cf, size = model:GetBoundingBox()
    local modelBottomY = cf.Position.Y - (size.Y / 2)
    local hrpBottomY = hrp.Position.Y - (hrp.Size.Y / 2)
    humanoid.HipHeight = math.max(0.5, hrpBottomY - modelBottomY)
end
```

### 3. Pembuatan Folder R15 Baku di Awal (*First-Pass*)
```lua
local R15_NAMES = {
    "Head", "UpperTorso", "LowerTorso",
    "LeftUpperArm", "LeftLowerArm", "LeftHand",
    "RightUpperArm", "RightLowerArm", "RightHand",
    "LeftUpperLeg", "LeftLowerLeg", "LeftFoot",
    "RightUpperLeg", "RightLowerLeg", "RightFoot",
}

local function setupR15Folders(model: Model)
    local folders = {}
    for _, name in ipairs(R15_NAMES) do
        local f = Instance.new("Folder")
        f.Name = name
        f.Parent = model
        folders[name] = f
    end
    return folders
end
```

---

## 🤖 Cara Integrasi ke AI Agent

Skill ini dirancang agar dapat dibaca langsung oleh AI coding assistant (seperti Antigravity, Cursor, Windsurf, Claude Code, GitHub Copilot).

### Opsi A: Workspace Agent Skill
Letakkan dokumen ini di dalam folder proyek Anda:
```
my-roblox-project/
├── .agents/
│   └── skills/
│       └── roblox_3d_procedural_modeling/
│           ├── SKILL.md
│           └── antislop_standart_3d.md
```

### Opsi B: Global Agent Skill (Antigravity IDE)
Salin folder ini ke:
```
C:\Users\<User>\.gemini\config\skills\roblox-3d-procedural-modeling\
├── SKILL.md
└── references/
    └── antislop_standart_3d.md
```

### Opsi C: Prompt Direct Reference
Cukup cantumkan tautan repository ini atau lampirkan `SKILL.md` saat meminta AI menggenerasi kode 3D di Roblox Studio:
> *"Buatkan model monster prosedural 3D box-only dengan mematuhi seluruh aturan di SKILL.md dan antislop_standart_3d.md."*

---

## 📄 Lisensi

Proyek ini dilisensikan di bawah [MIT License](LICENSE) &copy; 2026 dwianggororeal.
Bebas digunakan, dimodifikasi, dan didistribusikan untuk proyek game komersial maupun non-komersial.
