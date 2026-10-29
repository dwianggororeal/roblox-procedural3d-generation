# ANTI-SLOP STANDARDS: 3D WORLD, MODELS & ENVIRONMENT GENERATION

Panduan standar untuk pembuatan dan modifikasi objek 3D di Workspace Roblox Studio guna menghindari tampilan murahan (*generic Roblox obby/simulator slop*), masalah visual over-bloom, dan glitching pada objek fisik.

---

## 1. DAFTAR LARANGAN KERAS (WHAT TO AVOID / 3D SLOP)

### ❌ 1. Part Glow & Neon Spam pada Part Non-Lampu
* **Larangan:** 
  * Dilarang memberikan material `Enum.Material.Neon` atau efek glowing pada part struktural sembarangan (seperti lantai pijakan/pad, dinding, lis pintu, tiang, balok dekorasi, atau pinggiran senjata) **yang sebenarnya bukan sumber cahaya nyata (lampu/bulb/obor/kristal energi)**.
* **Kenapa dilarang:** 
  * Ciri khas game simulator/obby murahan tahun 2017.
  * Memicu *over-bloom* menyilaukan yang merusak sistem bayangan alami (*shadow depth*) dan membuat visual game terlihat amatir (*flat & noisy*).
* **Aturan Penggunaan Glow / Neon yang Legal:**
  1. **Hanya untuk elemen sumber cahaya nyata:** Filamen bola lampu (`Bulb`), kaca lentera, bara api obor, mata monster, atau kristal relik magis kuno.
  2. **Jika diminta secara eksplisit oleh konsep game** (misal: gaya retro synthwave atau laser barrier).

### ❌ 2. Floating Text Tembus Objek / Wallhack Spam (`BillboardGui`)
* **Larangan:** 
  * Jangan menyalakan `AlwaysOnTop = true` secara default pada `BillboardGui` atau nama/teks melayang di Workspace. Teks yang tembus dinding, bukit, dan bangunan merusak kedalaman 3D (*depth immersion*) dan mengotori pandangan pemain dengan sampah visual.
  * **Aturan:** Hanya gunakan `AlwaysOnTop` jika **secara eksplisit diminta oleh konsep game** (misal: tactical marker atau outline target tembus pandang).

### ❌ 3. Skala Konstan Jarak Jauh (Teks Membesar Saat Menjauh)
* **Larangan:** Menggunakan ukuran pixel offset murni tanpa batas jarak pandang (`MaxDistance = 0`), sehingga teks dari jarak 200–500 studs tetap berukuran raksasa di layar dan menutupi pandangan game.
* **Solusi Wajib:**
  * Selalu pasang **`MaxDistance` terbatas** (misal: `25 - 40 studs`), jangan dibiarkan `0` (tak terhingga).
  * Gunakan **`SurfaceGui` fisik** yang menempel langsung pada part/dinding atau `ProximityPrompt` bawaan (`RequiresLineOfSight = true`) daripada menabur floating Billboard liar di udara.

### ❌ 4. Ratusan Loose Parts & Weld-Fighting pada Model Custom
* **Larangan:** Membuat model karakter/makhluk dari 100+ part balok kecil yang masing-masing dihubungkan ratusan `WeldConstraint`.
* **Kenapa dilarang:** ACE (Animation Clip Editor) lag, tabrakan matriks transformasi (*weld-fighting*), dan part salah terpilih saat animasi.
* **Solusi Wajib:** Gabungkan menjadi 15/16 Part Solid (`Head`, `UpperTorso`, `LowerTorso`, dll.) via `UnionAsync` atau segmentasi anatomis yang bersih.

---

## 2. STANDAR PRAKTIK TERBAIK (BEST PRACTICES 3D)

### A. Realistis & Atmospheric Lighting
* Gunakan material fisik standar: `SmoothPlastic`, `Slate`, `Metal`, `Glass`, `WoodPlanks`, `Concrete` dengan warna-warna matte / netral.
* Biarkan sistem pencahayaan **`Lighting` (PointLight terukur, Atmosphere, ShadowMap / Future Lighting)** yang membentuk kedalaman dan suasana visual, bukan memaksakan part itu sendiri menjadi neon.

### B. Proximity & Diegetic 3D UI
* Gunakan `ProximityPrompt` bawaan Roblox:
  * `RequiresLineOfSight = true` (tidak bisa ditekan menembus tembok).
  * `MaxActivationDistance = 8 - 12 studs`.
* Jika butuh tulisan penunjuk arah/papan nama, gunakan `SurfaceGui` yang menempel di plat papan fisik 3D (*Diegetic UI*).

### C. Rigging & Skala Non-Humanoid
* Untuk monster, bos, atau custom creature dengan proporsi non-manusia:
  * Wajib gunakan **`AnimationController` + `Animator`**, bukan `Humanoid` (agar tidak kena *auto-snap deformation* oleh Roblox ACE).
  * Kalibrasi `HipHeight` berdasarkan jarak titik terendah `HumanoidRootPart` ke telapak kaki.

---

## 3. CHECKLIST PRA-DEPLOY 3D
- [ ] Apakah ada 3D Part biasa yang dijadikan `Neon` padahal bukan lampu/sumber cahaya nyata? $\rightarrow$ **Kembalikan ke material fisik (`SmoothPlastic`/`Metal`/`Slate`).**
- [ ] Apakah floating text (`BillboardGui`) disetel `AlwaysOnTop = true` tanpa diminta? $\rightarrow$ **Ubah `AlwaysOnTop = false`.**
- [ ] Apakah floating text memiliki `MaxDistance` terbatas (25–40 studs)? $\rightarrow$ **Pasang batasan jarak.**
- [ ] Apakah model custom terdiri dari part solid terstruktur (bukan 100+ loose parts dengan weld bertabrakan)?
