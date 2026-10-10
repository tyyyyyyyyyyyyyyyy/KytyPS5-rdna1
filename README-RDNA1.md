# KytyPS5 — RDNA1 / RX 5700 fork (`tyyyyyyyyyyyyyyyy`)

Fork pribadi ini menambahkan dukungan menjalankan **KytyPS5** di GPU
**AMD Radeon RX 5700 (Navi 10 / RDNA1 / gfx10.1)** — yang secara bawaan tidak
didukung — plus beberapa perbaikan game.

Mesin uji: Xeon E3-1230 v3, 15 GB RAM, **Radeon RX 5700**, Linux (Batocera 41 /
Xubuntu 24.04, Mesa RADV). Saat ini diuji di Linux.

---

## Kenapa perlu fork ini

1. **Barycentric.** Build KytyPS5 sejak sekitar `2026-08-22` mewajibkan ekstensi
   Vulkan `VK_KHR_fragment_shader_barycentric`. Driver Linux **RADV** hanya
   mengeksposnya untuk `gfx_level >= GFX10_3` (RDNA2+), jadi RX 5700 keluar dengan
   pesan `Could not find suitable device`. Fork ini membuat syarat itu **opsional**.

2. **Mesh shader.** Game yang butuh mesh shader (Goat Simulator 3, Astro's Playroom)
   diblokir karena RADV juga hanya mengekspos `VK_EXT_mesh_shader` untuk RDNA2+.
   Fork ini **mengemulasi mesh shader** (mesh diturunkan ke compute path di dalam
   Kyty), sehingga jalan di RDNA1.

3. **Sinkronisasi GPU↔CPU.** Menambahkan `WaitForIdle()` per-submit untuk menutup
   race condition yang membuat **Dead Cells** (dan beberapa game lain) hang di Linux.

4. **Ngs2 (audio).** Mengimplementasikan NID Ngs2 yang kurang supaya Raiden IV tidak
   crash.

---

## Branch

| Branch | HEAD | Isi |
|--------|------|-----|
| `main-latest` | `c4f300b9` | Versi paling lengkap di upstream terbaru (`5a880808`): barycentric opsional + Ngs2 (Raiden IV) + `WaitForIdle` (Dead Cells). **Gunakan ini sebagai basis.** |
| `experiment/next-stage` | `eb6a91e2` | **Mesh-emulation build** — Astro's Playroom & Goat Simulator 3 bisa masuk in-game, ~**35-60 FPS** di RX 5700. |
| `spike/mesh-compute` | `eb6a91e2` | Sama dengan `experiment/next-stage` (spike awal mesh→compute). |
| `experiment/mesh-geometry-buffer` | `e34e92e5` | Eksperimen lanjutan: menyambungkan mesh-emulation ke vertex-buffer acquisition & primitive draw. (WIP) |
| `fix/dcb-submit-ordering` | `dcea3e25` | Terpisah: fix Dead Cells (`WaitForIdle` per submit). |
| `main` | `d10eab0e` dkk. | Versi lama (patch barycentric + Ngs2 sebelum rebase). Dibiarkan. |

> Catatan: `main` di fork ini adalah history lama; yang terbaru ada di `main-latest`.
> Rencananya `main-latest` yang dipakai ke depan.

---

## Status game (RX 5700)

| Game | Status |
|------|--------|
| Dreaming Sarah (`PPSA02929`) | jalan |
| Dead Cells (`PPSA15552`) | jalan (butuh `WaitForIdle`) — audio in-game **senyap** (keterbatasan custom-rack Ngs2, upaya upstream) |
| Raiden IV x Mikado (`PPSA08832`) | crash **diperbaiki** (Ngs2 waveform); BGM "original" masih macet |
| Hollow Knight: Silksong (`PPSA12544`) | jalan (~75 FPS dengan `--vblank-frequency 75`) |
| TMNT: Splintered Fate (`PPSA25582`) | jalan, 60 FPS |
| Minecraft Bedrock (`PPSA17221`) | sampai layar loading (perlu verify ulang) |
| **Goat Simulator 3 (`PPSA04159`)** | **jalan via mesh-emulation** (~35-60 FPS) di branch `next-stage` |
| **Astro's Playroom (`PPSA01325`)** | **jalan via mesh-emulation** di branch `next-stage` |

---

## Cara build

### Windows (MSVC)
1. Install: **Visual Studio 2022**, **CMake**, **Vulkan SDK**, **Qt** (opsional, untuk launcher).
2. `git clone -b experiment/next-stage https://github.com/tyyyyyyyyyyyyyyyy/KytyPS5-rdna1.git`
3. Ikuti `README.md` upstream untuk langkah CMake/VS.
   Tanpa launcher: `-DKYTY_BUILD_LAUNCHER=OFF`.
4. Di **Windows**, mesh shader tersedia asli dari driver AMD, jadi **tidak perlu**
   branch mesh-emulation — build normal (`main-latest`) sudah cukup.

### Linux (clang + Ninja)
```bash
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=RelWithDebInfo -DKYTY_BUILD_LAUNCHER=OFF \
  -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ \
  -DCMAKE_C_COMPILER_LAUNCHER=ccache -DCMAKE_CXX_COMPILER_LAUNCHER=ccache
cmake --build build --target kyty_emulator -j8
```

### Lewat GitHub Actions
Workflow `build.yml` otomatis membangun artifact **Linux x86_64** dan **Windows** —
bisa di-download dari tab *Actions* tanpa memasang toolchain.

---

## Cara menjalankan (Linux)
```bash
# cwd harus writable (Kyty membuat _SaveData/)
./kyty_emulator --game "/path/ke/dir-yang-ada-eboot.bin" \
  --screen-width 1920 --screen-height 1080 --vblank-frequency 75
```
- Flag berguna: `--fullscreen`, `--gpu <idx>`, `--present-mode Mailbox|Fifo|Immediate`,
  `--mesh-emulation` (paksa emulasi mesh walau driver mendukung).
- Flag yang butuh nilai harus diberi nilai.

---

## Verifikasi build (patch sudah masuk)
```bash
strings kyty_emulator | grep "continuing without it"     # barycentric opsional (expect 1)
strings kyty_emulator | grep -c VK_KHR_fragment_shader_barycentric   # expect 0
```

---

## Lisensi
Mengikuti upstream KytyPS5 (GPL-2.0). Fork ini bukan afiliasi Sony Interactive
Entertainment / PlayStation.
