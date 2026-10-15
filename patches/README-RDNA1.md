# Patch RDNA1 + loop-guard (ASTRO's Playroom)

Patch di folder ini (`astro-loopguard.diff`) diterapkan otomatis oleh CI
(`.github/workflows/build.yml`) sebelum build, untuk Windows / Linux / macOS.

Semua edit di repo ini oleh **AI DeepSeek V4.1 Flash**.

## Isi patch

| File | Perubahan |
|---|---|
| `spirvEmitterProgram.cpp` | **Loop guard**: counter per loop guest; loop tanpa batas dipaksa keluar sebelum ring gfx timeout (guest hang → device lost). Env `KYTY_LOOP_GUARD_LIMIT` (default `1000000`, `0` = mati). |
| `spirvEmitterInternal.h` | Struktur data loop guard. |
| `spirvEmitterMemoryHelpers.cpp` | Barrier pasca-atomic LDS memakai scope **subgroup** (hindari deadlock wave divergen). Env `KYTY_LDS_WAITCNT_BARRIER` = `subgroup` (default) / `workgroup` / `0`. |
| `vulkanWindow.cpp` | `VK_KHR_fragment_shader_barycentric` dibuat **opsional** (RADV/RX 5700 tidak mengeksposnya; tanpa ini device selection gagal). |
| `pipelineCache.cpp` | `KYTY_FORCE_PIPELINE_CACHE=1` (izinkan cache saat build dirty/non-release) + `KYTY_CACHE_KEEP_AFTER_SAVE=1` (cache tetap hidup setelah save, bisa save berulang). |
| `window.cpp` | `SIGUSR1` memicu save pipeline cache tanpa harus keluar bersih (`kill -USR1 <pid>`). Di-guard `#ifdef SIGUSR1` agar tetap bisa dikompilasi di Windows. |

## Catatan

- Patch **HOSTBT** (`runtimeLinker.cpp`, `backtrace()`) **tidak** disertakan di
  patch ini karena Linux-only (dipakai hanya untuk analisis crash lokal).
- Env var ada di tabel di atas; set sebelum menjalankan emulator/launcher.
- macOS: build x86_64 + MoltenVK. Build berhasil, tapi **runtime** macOS belum
  diuji (emulator ini target utama Windows/Linux).
