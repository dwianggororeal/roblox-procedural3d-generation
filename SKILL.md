---
name: roblox_3d_procedural_modeling
description: Panduan baku pemodelan 3D prosedural box-only di Roblox Studio, termasuk aturan koordinat, struktur anatomi, pencegahan Z-fighting, dan pitfalls.
---

# ROBLOX 3D PROCEDURAL MODELING & MASTER RIGGING RULES

Dokumen ini berisi standar baku perancangan model 3D berbasis kode (Luau procedural generation), pencegahan tabrakan geometri (*Z-fighting*), sistem koordinat lokal Roblox, dan kompilasi kesalahan umum (*pitfalls post-mortem*) agar tidak terulang kembali.

---

## 0. ATURAN MUTLAK GEOMETRI: BOX-ONLY

Seluruh model yang dibuat atau diedit menggunakan skill ini **WAJIB 100% tersusun dari `Part` berbentuk box**:

```lua
part.ClassName == "Part"
part.Shape == Enum.PartType.Block
```

Larangan mutlak:

- **DILARANG** memakai `Part.Shape = Ball` atau `Part.Shape = Cylinder`.
- **DILARANG** memakai `WedgePart`, `CornerWedgePart`, `MeshPart`, `UnionOperation`, `TrussPart`, `SpecialMesh`, atau geometri non-box lainnya.
- **DILARANG** memakai generator mesh atau mengganti bentuk box dengan primitive lain demi membuat permukaan terlihat lebih halus.
- Bentuk melengkung, otot, tanduk, ekor, jari, dan siluet organik **WAJIB** dibangun dari rangkaian box kecil yang dirotasi, diskalakan, dan ditumpuk dengan depth yang aman.
- `HumanoidRootPart` yang transparan tetap harus berupa `Part` dengan `Shape = Block`.

### Standar Wajah Sederhana

- Mata cukup dibuat dari `Part` box tipis berwarna. Putih mata, iris, dan pupil boleh dipisah menjadi beberapa box warna bila memang membantu keterbacaan.
- Alis cukup dibuat dari `Part` box tipis berwarna gelap, lalu diposisikan dan dirotasi untuk membentuk ekspresi.
- Rahang (`jaw`) cukup dibuat dari satu atau beberapa `Part` box yang disusun mengikuti siluet kepala; tidak perlu dibuat melengkung atau dipahat rumit.
- Hidung, mulut, gigi, dan fitur wajah lain tetap mengikuti aturan box-only. Prioritaskan siluet dan kontras warna yang terbaca daripada jumlah part berlebihan.
- Jangan memakai Ball, Cylinder, Wedge, atau Mesh untuk memperhalus wajah.

Sebelum model diserahkan, jalankan validasi fail-fast berikut. Hasil selain nol berarti model belum valid:

```lua
local violations = 0
for _, instance in ipairs(model:GetDescendants()) do
    if instance:IsA("BasePart") then
        if not instance:IsA("Part") or instance.Shape ~= Enum.PartType.Block then
            violations += 1
        end
    end
end
assert(violations == 0, (`Box-only violation count: %d`):format(violations))
```

Aturan box-only ini mengalahkan contoh atau kebiasaan lain dalam dokumen ini jika terjadi ambiguitas.

---

## 1. ALUR KERJA BAKU: MODELING DAHULU, RIGGING BELAKANGAN (*MODELING-FIRST, RIGGING-LAST*)

Sangat dilarang memasang sendi gerak (`Motor6D`) sebelum bentuk fisik dan estetika model disetujui 100%. Alur kerja wajib dibagi menjadi 3 fase terpisah:

### 🔹 Fase 1: Pemodelan Estetika & Statis (*Pure Static Sculpting*)
- Seluruh part dibuat dalam kondisi **`Anchored = true`** dan **TANPA sendi `Motor6D`**.
- Fokus 100% pada:
  - Proporsi anatomi dan siluet karakter (A-Pose/T-Pose statis).
  - Wajah, mata, taring (*inward curve*), tanduk bebas tabrakan topi.
  - Lapisan pakaian, jubah, stola, kain paha/rusuk, dan ornamen perhiasan.
  - Pencegahan *Z-fighting* menggunakan *Stepped Depth Layering*.
  - Pengelompokan langsung ke dalam **15 Folder R15 baku** sejak awal (*First-Pass Direct Folder Parenting*).

### 🔹 Fase 2: Review & Penyempurnaan Visual Pengguna
- Pengguna dapat menginspeksi model dari berbagai sudut, mengedit posisi, mengubah warna/material (*Anti-Slop standard*), dan menyempurnakan detail tanpa terganggu oleh batasan matriks `Motor6D`.

### 🔹 Fase 3: Rigging & Kalibrasi Sendi (*Final Rigging Phase*)
- Setelah bentuk visual disetujui 100%:
  - Pasang sendi `Motor6D` menggunakan *Seamless Socket Overlap Formula*.
  - Pasang `AnimationController` + `Animator` (untuk monster/custom rig).
  - Ubah part menjadi `Anchored = false` (kecuali `HumanoidRootPart` jika diperlukan).
  - Kalibrasi `HipHeight` secara matematis.

---

## 2. ATURAN POSE AWAL (DEFAULT RIG STANCE: A-POSE / T-POSE)

Setiap pembuatan model 3D prosedural baru **WAJIB dibuat dalam postur baku (A-Pose atau T-Pose)** sebelum diberikan animasi atau pose dinamis:

* **A-Pose Baku (Default Rig Stance):**
  - **Rentang Bahu (*Shoulder Flare*):** Lengan direntangkan lurus dengan sudut rileks **~20° - 24°** dari torso:
    - *Lengan Kiri:* `-math.rad(24)` pada sumbu Z.
    - *Lengan Kanan:* `+math.rad(24)` pada sumbu Z.
  - **Siku & Pergelangan Tangan (*Elbow & Wrist*):** **WAJIB 100% LURUS & NETRAL sepanjang garis diagonal A-Pose**.
  - **Orientasi Telapak Tangan Alami (*Natural Inward Palm Rotation*):**
    - Telapak tangan **DILARANG menghadap datar ke depan seperti papan dayung**.
    - Telapak tangan **WAJIB dirotasi 90° menghadap ke dalam (*Inward facing the thigh/torso*)**, punggung tangan (*knuckles*) menghadap ke luar samping, jempol mengarah ke depan/dalam, dan jari-jari berbaris dari depan ke belakang dengan kelengkungan cakar alami ke bawah.
  - **Kuda-kuda Kaki (*Leg A-Stance*):** Kaki berdiri lurus dengan sudut melebar halus **~6°** dan telapak kaki menapak rata (*grounded*).
  - *Keunggulan:* Rig bersih, siluet tangan sangat hidup dan natural, sendi lurus sempurna, serta memudahkan pembacaan orientasi segmen pada Animation Clip Editor (ACE).

* **Kelengkungan Taring / Tusks:**
  - Taring rahang bawah (*orc/oni/boar tusks*) **WAJIB melengkung ke ATAS dan ke DALAM (*Inward*)** memeluk rahang dan mengarah ke hidung/mulut (`Z-rotation` inward), **BUKAN mencuat melebar ke luar**.
* **T-Pose (Alternatif):**
  - Lengan direntangkan 90° horizontal mendatar lurus ke samping.
  - Cocok untuk model mekanikal atau karakter berstruktur sederhana.

---

## 2. STRUKTUR HIERARKI FOLDER ANATOMIS R15 (EXACT R15 FOLDER STANDARD)

Setiap model 3D prosedural **WAJIB menggunakan struktur Folder dengan nama persis segmen anatomi R15 baku (tanpa suffix/akhiran)** untuk mengelompokkan seluruh part detail:

```
Model (Contoh: Abyssal_Demon_Bishop)
├── HumanoidRootPart (Invisible Root BasePart)
│
├── Head (Folder)               --> Berisi: Mata, Alis, Hidung, Taring Inward, Tanduk, Mitre Hat, Jenggot, dll.
├── UpperTorso (Folder)         --> Berisi: Otot Dada (Pectorals), Cleft, Traps, Jubah Cape Punggung, Stola Ungu, Kalung Salib.
├── LowerTorso (Folder)         --> Berisi: Abs 8-Pack, Sabuk Cincture, Tabard Salib Latin Depan, Sash.
├── LeftUpperArm (Folder)       --> Berisi: Deltoid Kiri, Lengan Jubah Kiri, Cuff.
├── LeftLowerArm (Folder)       --> Berisi: Otot Lengan Bawah Kiri.
├── LeftHand (Folder)           --> Berisi: 4 Jari Cakar Kiri, Jempol Kiri.
├── RightUpperArm (Folder)      --> Berisi: Deltoid Kanan, Lengan Jubah Kanan, Cuff.
├── RightLowerArm (Folder)      --> Berisi: Otot Lengan Bawah Kanan.
├── RightHand (Folder)          --> Berisi: 4 Jari Cakar Kanan, Jempol Kanan, Rosario & Salib Gantung.
├── LeftUpperLeg (Folder)       --> Berisi: Rok Jubah Kiri, Lis Emas Kiri.
├── LeftLowerLeg (Folder)       --> Berisi: Kaki Bawah Kiri.
├── LeftFoot (Folder)           --> Berisi: 4 Jari Cakar Kaki Kiri.
├── RightUpperLeg (Folder)      --> Berisi: Rok Jubah Kanan, Lis Emas Kanan.
├── RightLowerLeg (Folder)      --> Berisi: Kaki Bawah Kanan.
├── RightFoot (Folder)          --> Berisi: 4 Jari Cakar Kaki Kanan.
│
├── AnimationController + Animator (Khusus Custom Creature/Boss)
└── Humanoid (Terkalibrasi HipHeight)
```

> [!TIP]
> **Aturan Penamaan Folder:**
> Nama folder wajib menggunakan **nama persis PascalCase R15** (misal: `Head`, `UpperTorso`, `LeftUpperArm`, dll.) tanpa imbuhan `_Parts` atau `_Folder` agar Explorer rapi dan konsisten 1:1 dengan standar anatomi Roblox.

---

## 3. SISTEM KOORDINAT LOKAL ROBLOX (ATURAN MUTLAK ANTI-TERBALIK)

Dalam Roblox Studio, sistem koordinat lokal (*Local CFrame*) part/karakter memiliki arah baku sebagai berikut:

| Arah Anatomi Karakter | Sumbu Lokal Roblox | Vektor Baku | Contoh Penempatan Part |
|---|---|---|---|
| **DEPAN (*Front*)** | **`-Z`** | `CFrame.LookVector (0, 0, -1)` | Mata, Alis, Hidung, Taring, Otot Dada, Perut (Abs), Kalung, Sabuk Depan, Tabard, Jari Kaki |
| **BELAKANG (*Back*)** | **`+Z`** | `Vector3.new(0, 0, 1)` | Punggung, Otot Trapezius Belakang, Jubah Belakang, Ekor, Sayap |
| **KANAN KARAKTER (*Right*)** | **`+X`** | `CFrame.RightVector (1, 0, 0)` | Lengan Kanan, Tangan Kanan, Kaki Kanan, Aksesoris Kanan |
| **KIRI KARAKTER (*Left*)** | **`-X`** | `Vector3.new(-1, 0, 0)` | Lengan Kiri, Tangan Kiri, Kaki Kiri, Aksesoris Kiri |
| **ATAS (*Up*)** | **`+Y`** | `CFrame.UpVector (0, 1, 0)` | Kepala, Topi/Mitre, Tanduk, Bahu Atas |
| **BAWAH (*Down*)** | **`-Y`** | `Vector3.new(0, -1, 0)` | Dagu, Kaki, Telapak Kaki, Cakar Bawah |

> [!CAUTION]
> **HUKUM UTAMA SUMBU Z:**
> **JANGAN PERNAH** menggunakan offset positif (`+Z`) untuk fitur depan! Menulis `+Z` berarti menempelkan objek ke **PUNGGUNG**. Seluruh fitur depan **WAJIB bernilai minus (`-Z`)**.

---

## 2. PENCEGAHAN Z-FIGHTING & PROTOKOL SPATIAL LAYERING

### A. Syarat & Penyebab Mutlak Terjadinya Z-Fighting (Z-Fighting Root Criteria)
Z-fighting (kedipan poligon / flickering) pada engine rendering Roblox terjadi ketika:
1. **Coplanar Surface Collision ($\Delta Y = 0$):** Dua atau lebih BasePart yang saling beririsan / bertumpuk secara spasial ($X/Z$ intersection) memiliki elevasi bidang permukaan yang **sama persis** ($Y_{\text{top1}} == Y_{\text{top2}}$ atau $Y_{\text{bot1}} == Y_{\text{bot2}}$).
2. **Sub-Stud Micro-Overlap ($< 0.1\text{ stud}$):** Dua part berhimpit dengan selisih ketinggian yang terlalu tipis sehingga resolusi *depth buffer* GPU tidak mampu menentukan prioritas kedalaman poligon saat kamera bergerak atau menjauh.
3. **Lateral Edge Collision:** Dua part yang berdampingan pada ketinggian yang sama saling menembus batas samping tanpa batas pertemuan yang bersih (*butted boundary*).
4. **Kerentanan Krusial Bagian Permukaan (*Surface Critical Zone*):** Bagian permukaan (seperti lantai tanah, lempeng rumput, papan kayu, bordes batu, dan puing) adalah area yang paling rentan terhadap Z-fighting karena sering dipandang dari sudut atas (*top-down* / *third-person camera*).

---

### B. Rumus & Aturan Baku Eliminasi Z-Fighting (Anti-Z-Fighting Law)
1. **Butted Edge Boundaries untuk Part Selevel:**
   - Part-part yang berada di level ketinggian yang sama ($Y$ identik) **DILARANG KERAS saling menembus (*no spatial overlap*)**. Batas tepi part harus dihitung presisi agar saling bertemu di tepinya:
     $$\text{Position}_B.X = \text{Position}_A.X + \frac{\text{Size}_A.X + \text{Size}_B.X}{2}$$
2. **Discrete Stepped Elevation Offset ($\ge 0.15\text{ - }0.25\text{ studs}$):**
   - Setiap part dekorasi permukaan (jalur batu setapak, balok kayu lapuk, puing, bunga, bordes) yang menumpang di atas lempeng dasar WAJIB memiliki elevasi $Y_{\text{bottom}}$ yang duduk tepat atau sedikit di atas $Y_{\text{top}}$ lempeng bawahnya:
     $$Y_{\text{bottom\_prop}} = Y_{\text{top\_base}} + \Delta \quad (\Delta \ge 0.05\text{ studs})$$
3. **Outward Lateral Offset pada Dinding Vertikal ($\ge 0.25\text{ studs}$):**
   - Part vertikal gantung (seperti *drooping grass fringe*, lis tepi) wajib digeser keluar sebesar $+0.25\text{ studs}$ dari dinding utama agar bidang belakangnya tidak *coplanar* dengan dinding tebing di belakangnya.

---

### C. Hirarki Ruang Vertikal Wajah (*Facial Vertical Clearance Bands*)
Setiap fitur wajah pada `Head` (tinggi standar ~2.8 - 3.0 studs) memiliki zona vertikal (`Y`) dan kedalaman (`Z`) yang terisolasi:

```
[ Y: +1.4 s/d +4.5 ]  --> Topi Mitre / Mahkota / Helm (Z: -1.25)
[ Y: +1.2 s/d +3.5 ]  --> Pangkal & Ujung Tanduk (X: ±1.5, Z: +0.1)
[ Y: +0.7 s/d +0.9 ]  --> Alis / Brow Ridge (Z: -1.35)
[ Y: +0.4 s/d +0.6 ]  --> Mata / Eyes (Z: -1.45, X: ±0.75)  <-- AREA BEBAS HALANGAN
[ Y: +0.1 s/d +0.3 ]  --> Hidung / Snout (Z: -1.48, X: 0)
[ Y: -0.6 s/d +0.0 ]  --> Pangkal & Lengkung Taring (X: ±1.15, Z: -1.55) <-- DI LUAR & DI BAWAH MATA
[ Y: -1.2 s/d -1.8 ]  --> Dagu, Jenggot & Rahang Bawah (Z: -0.9 s/d -1.2)
```

---

## 3. KATALOG KESALAHAN UMUM (*PITFALLS & ROOT CAUSES*)

### 🔴 Kesalahan 1: Sumbu Terbalik (*Mental Axis Inversion*)
- **Penyebab:** Terbiasa menganggap "maju mendekati kamera = positif (+)".
- **Akibat:** Dada, mata, dan aksesoris terbangun di punggung karakter (`+Z`), sedangkan jubah belakang menutupi bagian depan (`-Z`).
- **Pencegahan:** Selalu deklarasikan variabel `FRONT_Z = -1` dan `BACK_Z = 1` sebagai pengali offset.

### 🔴 Kesalahan 2: Tusk Menutupi Mata & Tanduk Menembus Topi (Clipping & Collision)
- **Penyebab Tusk:** Menaruh tusk dengan ukuran terlalu tinggi hingga masuk ke koordinat Y mata (`Y >= 0.35`).
- **Penyebab Tanduk:** Menaruh pangkal tanduk (`X: ±1.5`) di dalam batas radius topi mitre/helm (`X: 1.55`) lalu melengkungkannya ke dalam sehingga menembus kain topi.
- **Pencegahan:**
  - Ujung tusk **WAJIB berhenti di bawah garis mata minimal 0.4 studs** dan diposisikan di sudut luar rahang (`X: ±1.25` vs mata `X: ±0.75`).
  - Pangkal tanduk **WAJIB diletakkan di luar radius topi/helm** (`X: ±1.8+`), melengkung melebar ke luar sebelum naik membingkai topi.

### 🔴 Kesalahan 3: Z-Fighting pada Pakaian & Ornamen (Flat Depth Stacking & Joint Collision)
- **Penyebab Depth Stacking:** Menempelkan part otot dada, jubah, stola kain, dan salib emas di kedalaman Z yang sama persis (`Z = -1.7`).
- **Penyebab Joint Collision (Bahu/Ketiak):** Menempelkan part kain jubah statis di `UpperTorso` (`Shoulder_Drape`) yang bertabrakan dengan part lengan baju yang berputar di `UpperArm` (`Sleeve`). Saat lengan di-A-posekan, kedua part kain dengan warna/material sama akan saling membelah dan memicu Z-fighting bertingkat.
- **Pencegahan (Stepped Layering & Joint Isolation Protocol):**
  - **Isolasi Sendi Bahu (*Shoulder Joint Isolation*):** Jubah punggung di `UpperTorso` **DILARANG melebar melewati soket bahu** (`X <= ±2.8`). Bagian lengan jubah (*Sleeve*) **WAJIB 100% menempel hanya pada `UpperArm`**, sehingga saat lengan berputar, tidak ada tabrakan antar-kain.
  - **Stepped Depth Protocol:**
    - Layer 0 (Kulit / Otot Dada / Abs): `Z = -1.70`
    - Layer 1 (Jubah / Stola Kain): `Z = -2.05` (Selisih +0.35 studs)
    - Layer 2 (Rantai Kalung & Salib Emas): `Z = -2.28` (Selisih +0.23 studs)
    - Layer 3 (Permata / Jewel): `Z = -2.35` (Selisih +0.07 studs)

### 🔴 Kesalahan 4: Loose Parts Tanpa Solid Rig (Weld-Fighting)
- **Penyebab:** Menempelkan ratusan part kecil secara acak tanpa hierarki 15 part solid standar R15.
- **Akibat:** Animation Clip Editor (ACE) lag parah, salah memilih part saat menganimasikan, dan crash matriks transformasi.
- **Pencegahan:** Hanya gunakan **15/16 Part Solid R15** sebagai part motorik utama. Seluruh detail hiasan (otot, tanduk, tusk, jubah, permata) di-weld via `WeldConstraint` langsung ke part anatomi induknya.

### 🔴 Kesalahan 5: Reorganisasi Folder Pasca-Pembuatan vs Direct-to-Folder Parenting
- **Penyebab:** Membuat part-part hiasan terlebih dahulu di root Model, lalu mencoba memindahkannya (*re-parent*) ke Folder menggunakan filter nama/skrip perulangan di akhir.
- **Akibat:** Part-part sub-komponen kecil (seperti ujung salib, manik-manik, permata, kuku cakar) sering kali tidak terdeteksi oleh filter nama sehingga tercecer di luar folder atau hilang terhapus saat pembersihan folder duplikat.
- **Pencegahan (First-Pass Direct Folder Parenting):**
  - Buat ke-15 Folder anatomi R15 **di awal skrip (sebelum part apa pun dibuat)**.
  - Setiap fungsi pembuat part **WAJIB langsung menetapkan `part.Parent = folders[limbName]` pada saat instansiasi**, bukan setelah model selesai dibangun.

### 🔴 Kesalahan 6: Karakter Berubah Bentuk Saat Klik ACE (*Auto-Snap Distortion*)
- **Penyebab:** Memasang `Humanoid` standar pada monster/makhluk dengan proporsi non-manusia.
- **Akibat:** Roblox ACE memaksa menarik tulang rig ke proporsi manusia normal sehingga model hancur.
- **Pencegahan:** Gunakan `AnimationController` + `Animator` untuk custom monster/creature/boss.

### 🔴 Kesalahan 7: Karakter Melayang / Tenggelam (*Uncalibrated HipHeight*)
- **Penyebab:** Memberikan nilai `HipHeight` sembarangan atau default (2.0).
- **Akibat:** Model berdiri melayang di udara atau kakinya mendelep ke dalam tanah.
- **Pencegahan:** Gunakan rumus wajib kalkulasi matematis:
  ```lua
  local cf, size = model:GetBoundingBox()
  local modelBottomY = cf.Position.Y - (size.Y / 2)
  local hrpBottomY = hrp.Position.Y - (hrp.Size.Y / 2)
  humanoid.HipHeight = math.max(0.5, hrpBottomY - modelBottomY)
  ```

### 🔴 Kesalahan 8: Mengedit Langsung Tanpa Duplikasi Backup & Cek Lokasi
- **Penyebab:** Langsung menimpa model aktif atau meregenerasi dari titik origin tanpa mengecek koordinat pengguna.
- **Akibat:** Kehilangan pekerjaan sebelumnya dan model berpindah tempat tanpa disadari pengguna.
- **Pencegahan:** **WAJIB** cek lokasi model aktif terlebih dahulu di Workspace, lakukan `Clone()` sebagai backup bertanda `_Backup`, lalu terapkan edit pada model aktif.

### 🔴 Kesalahan 9: Celah Bolong pada Sendi (*Floating Joint Gap & Angular Gap*)
- **Penyebab:**
  1. *Linear Gap:* Menentukan nilai `C0` dan `C1` lebih besar dari setengah tinggi part (`Size.Y / 2`), menyisakan ruang kosong bolong.
  2. *Angular Gap:* Pada part balok berbidang datar, saat lengan/kaki diputar (misal A-Pose 20°), bidang datar atas dan bawah akan membuka seperti gunting/buku jika tidak ada overlap.
- **Akibat:** Sendi siku, lutut, pinggang, atau leher terlihat bolong dan menembus ke latar langit.
- **Pencegahan (Seamless Socket Overlap Protocol):**
  - Berikan sedikit penetrasi sendi (*socket overlap*) sebesar **`0.05 - 0.1` studs** di dalam formula C0 dan C1:
    $$\text{Motor6D.C0} = \text{CFrame.new}(0, -H_A / 2 + \text{overlap}, 0)$$
    $$\text{Motor6D.C1} = \text{CFrame.new}(0, +H_B / 2 - \text{overlap}, 0)$$
  - Dengan overlap `0.1 studs`, saat part diputar pada sudut mana pun, bagian dalam sendi tetap padat (*solid socket insertion*) dan celah bolong tertutup total 100%.

### 🔴 Kesalahan 10: Jari Miring Keluar / Terpelintir Terbalik (*Inverted Finger Flare Bug*) & Jempol Mendelep (*Sunken Thumb*)
- **Penyebab:**
  1. *Inverted Finger Flare:* Pada A-Pose, lengan kiri berotasi `-22°` pada sumbu Z. Jika jari tangan kiri diberi rotasi Z bertanda negatif (`-15°` s.d `-22°`), nilainya akan menjumlahkan kemiringan lengan sehingga jari-jari justru mencuat melebar ke luar samping (*flare outward* menjauhi tubuh).
  2. *Sunken Thumb:* Menaruh jempol sejajar atau di dalam volume balok telapak tangan (*inside metacarpal bounds*) sehingga jempol tertelan/mendelep ke dalam volume telapak.
- **Akibat:** Tangan terlihat cacat anatomis, jari-jari kaku mencuat ke luar seperti dayung perahu terbalik, dan jempol tenggelam.
- **Pencegahan (Inward Finger Curl & Wrap-Over Thumb Protocol):**
  - **Tangan Kiri:** Rotasi sumbu Z jari-jari **WAJIB BERNILAI POSITIF (`+math.rad(20°)` s.d `+math.rad(24°)`)** untuk mengimbangi sudut lengan `-22°`, sehingga 4 jari menekuk condong ke dalam (*inward*) memeluk paha/torso.
  - **Tangan Kanan:** Rotasi sumbu Z jari-jari **WAJIB BERNILAI NEGATIF (`-math.rad(20°)` s.d `-math.rad(24°)`)** untuk mengimbangi sudut lengan `+22°`, sehingga 4 jari menekuk condong ke dalam (*inward*) memeluk paha kanan.
  - **Wrap-Over Thumb:** Pangkal otot jempol (*Thenar Eminence*) diletakkan menonjol di sisi anterior/depan (`-Z`), dan batang jempol melipat diagonal di atas/luar jari telunjuk sehingga terlihat jelas dan natural dari depan.

### 🔴 Kesalahan 11: Kaki Terseret & Melayang pada Animasi Idle (*Idle Foot Drift & Floating Glitch*)
- **Penyebab:** Menggeser translasi sumbu X/Z pada `LowerTorso` (*lateral weight shift*) atau memiringkan panggul tanpa kompensasi Inverse Kinematics (IK), sehingga posisi absolut telapak kaki (`LeftFoot` dan `RightFoot`) ikut terdorong terseret (*sliding*) atau melayang dari permukaan tanah seolah-olah terbang.
- **Akibat:** Karakter terlihat seperti melayang atau kehilangan pijakan di tanah saat diam.
- **Pencegahan (Strict Ground Plant Protocol for Idle Animations):**
  - Pada seluruh animasi berstatus `Idle`, kedua telapak kaki (`LeftFoot` & `RightFoot`) serta sendi kaki bawah **WAJIB 100% TERKUNCI DIAM MENAPAK DI LANTAI (`CFrame.identity` / Zero Translation & Zero Rotation Drift)**.
  - Seluruh dinamika idle (seperti menoleh santai, menggerakkan tangan, memutar tasbih, bernapas, atau liukan dada) **HANYA BOLEH DIAPLIKASIKAN PADA SENDI DI ATAS PANGGUL (`UpperTorso`, `Head`, `Shoulders`, `Elbows`, `Wrists`)**.

---

## 4. CHECKLIST PRA-GENERASI & PASCA-GENERASI

Sebelum dan sesudah menjalankan kode pembuatan model 3D di Roblox Studio, verifikasi 9 poin berikut:

- [ ] **0. Box-Only:** Apakah setiap `BasePart` merupakan `Part` dengan `Shape = Enum.PartType.Block`, tanpa Ball, Cylinder, WedgePart, MeshPart, UnionOperation, atau geometri non-box?
- [ ] **1. Sumbu Depan-Belakang:** Apakah semua part wajah, dada, dan sabuk depan memakai nilai offset minus (`-Z`)?
- [ ] **2. Sumbu Kanan-Kiri:** Apakah tangan kanan di `+X` dan tangan kiri di `-X`?
- [ ] **3. Layering Wajah:** Apakah taring berada di bawah zona mata dan di sisi luar rahang?
- [ ] **4. Struktur 15 Part Solid:** Apakah sendi `Motor6D` hanya menghubungkan 15 part utama dan part estetika memakai `WeldConstraint`?
- [ ] **5. Safety Backup:** Apakah model asli sudah diduplikasi sebelum pengeditan dilakukan?
- [ ] **6. Kalibrasi HipHeight:** Apakah `HipHeight` sudah dihitung otomatis berdasarkan titik terendah bounding box?
- [ ] **7. Orientasi Jari & Jempol:** Apakah jari tangan kiri memakai rotasi Z positif (`+Z_rot`) dan tangan kanan memakai rotasi Z negatif (`-Z_rot`) agar condong ke dalam (*inward*)? Apakah jempol membungkus rapi di luar jari telunjuk (tidak mendelep)?
- [ ] **8. Pijakan Kaki Idle (*Idle Foot Grounding*):** Apakah telapak kaki pada animasi `Idle` disetel `CFrame.identity` (terkunci 100% diam di lantai tanpa sliding/melayang)?

---

## 5. TEKNIK RENDERING UNLIT 2D FLAT SILHOUETTE & KALIBRASI ANTI-BLOOM IN-GAME

Teknik untuk menghasilkan tampilan **siluet 2D grafis murni (*unlit*, bebas pantulan cahaya/matahari, bebas kilap plastik)** dan **TIDAK MELEDAK SILAU / OVER-BLOOM SAAT PLAY IN-GAME**:

### A. Mengapa di Studio Edit Mode Bagus, Tapi di In-Game Silau Meledak?
1. **Di Studio Edit Mode:** Viewport kamera mematikan / mereduksi efek post-processing HDR (`BloomEffect`) demi optimasi FPS.
2. **Saat Play / In-Game Mode:** Pipeline HDR penuh diaktifkan. Shader `BloomEffect` membaca part `Neon` dengan RGB putih murni `(255, 255, 255)` yang memiliki *luminance* $> 0.8$, lalu menerapkan filter Gaussian blur HDR yang meledakkan cahaya menjadi bola lampu putih menyilaukan (*over-bloom*).

### B. Solusi: Kalibrasi Nilai RGB Neon di Bawah Ambang Batas Bloom (Luminance ~0.65)
- **Cara Kerja:**
  - Tetap gunakan **`Enum.Material.Neon`** dengan **`Reflectance = 0`** dan **`CastShadow = false`** agar part **100% UNLIT (tidak terpengaruh arah sinar matahari, tidak ada bayangan 3D, tidak ada titik kilap plastik)**.
  - Namun, setel warna putihnya ke rentang **`Color3.fromRGB(165, 165, 160)` s.d `(175, 172, 168)`** (bukan 255 murni).
- **Hasil Visual:**
  1. **Kontras Putih Terang:** Karena berlatar belakang badan hitam pekat, mata pemain tetap melihatnya sebagai siluet grafis putih tulang yang sangat terang dan solid.
  2. **0% Silau di In-Game:** Nilai RGB-nya berada tepat di bawah *Threshold Bloom* engine Roblox (~0.8), sehingga shader Bloom **TIDAK AKAN memicu efek pendaran menyilaukan** saat dites main (*playtest*).





