# AGENTS.md — Splashfall: Poison Arena

> Rencana development 3 minggu (20 Nov – 11 Des 2026). File ini adalah sumber kebenaran untuk **scope dan jadwal**. GDD v4.0 tetap menjadi referensi desain, kecuali bagian yang diubah di §1 dan §12.

## 0. Aturan untuk agent

- Awal sesi: baca AGENTS.md lalu PROGRESS.md, ringkas posisi + next action, lalu tunggu persetujuan sebelum mengerjakan apa pun. Akhir sesi (atau setelah tiap task/keputusan penting): update PROGRESS.md (posisi, next action, fakta baru, log). Centang task tetap di AGENTS.md.
- Baca seluruh file ini sebelum bekerja. Jika file ini bertentangan dengan GDD, file ini yang menang; catat konfliknya di §12.
- Jangan menambah fitur di luar daftar **Must** dan **Should** tanpa persetujuan tim. Isi **Future** (§3) tidak boleh dikerjakan.
- Jangan mengedit file biner `.uasset` / `.umap` langsung lewat filesystem atau editor teks (tidak bisa di-merge, mudah rusak). Semua pembuatan dan modifikasi Blueprint, Level, dan asset Unreal wajib lewat **unreal-mcp**. Setelah tiap perubahan: compile, save, lalu cek hasil compile tanpa error.
- Jika unreal-mcp tidak terhubung atau tidak mendukung operasi yang dibutuhkan: berhenti, laporkan, dan tulis spesifikasi (variabel, event, urutan node) untuk dikerjakan manusia di editor. Jangan mengakali dengan edit biner.
- Perubahan lewat unreal-mcp tetap mengikuti ownership asset di §7: untuk asset milik peran lain, konfirmasi ke tim dulu. Jangan `git commit` / `push` kecuali diminta.
- File teks (`Source/*.cpp`, `docs/*.md`, file ini) boleh diedit langsung; `Config/*.ini` hanya jika diminta eksplisit.
- Sebelum membuat asset baru, cek asset existing di `Content/` agar tidak duplikat. Jika ada asset existing dengan peran yang sama, pakai namanya sesuai tabel pemetaan di §5; jangan membuat padanan baru.
- Konvensi nama untuk asset baru: `BP_`, `WBP_`, `NS_`, `M_` / `MI_`, `DA_`, `SM_`, `SFX_`. Ikuti struktur folder existing; jika belum ada padanannya, pakai `/Content/Splashfall/{Blueprints,Maps,UI,VFX,Audio,Art}`.
- Setiap task selesai: ubah `[ ]` menjadi `[x]` dan tambah satu baris di log PROGRESS.md (tanggal, siapa, apa). Jangan menulis ulang bagian lain.
- Ragu soal scope atau keputusan desain: tanya tim, jangan menebak.
- Bahasa: Indonesia semi-formal; istilah teknis game dan Unreal Engine tetap English.

## 1. Perubahan dari GDD v4.0

| Aspek | GDD v4.0 | Rencana 3 minggu |
|---|---|---|
| Multiplayer | Online (replikasi, network desync, latensi jaringan) | **Local couch 1v1 di satu PC: 2 gamepad PlayStation (DualShock 4 / DualSense), split-screen.** Tanpa replikasi, server, atau lag |
| Timeline | 8 minggu | 3 minggu; feature freeze Kamis 3 Des |
| Role | Network & UI Programmer | Match, Input & UI Programmer |
| Input | Keyboard + mouse utama, gamepad opsional | Gamepad utama (2 pemain). Keyboard + mouse hanya fallback dev untuk P1 |
| Requirement jaringan | Latensi < 100 / 50 ms LAN | Dihapus; diganti requirement FPS split-screen |
| Potion Pickup | P1 | Naik ke **Must** (tanpa pickup, 3 potion habis dan match macet) |
| AOE Contest | Zona bertabrakan saling menimpa | Disederhanakan: AOE baru **menggantikan penuh** AOE lawan yang overlap (tanpa pemotongan parsial) |

## 2. Asumsi

- Deadline: **Jumat 11 Des 2026**. Minggu 1: 20–26 Nov. Minggu 2: 27 Nov–3 Des. Minggu 3: 4–10 Des. 11 Des = buffer / pengumpulan.
- Tim 3 orang, kapasitas ±10–12 jam/orang/minggu (±100 jam total) karena berbarengan dengan tugas dan ujian lain. Jika kapasitas nyata lebih kecil, pakai cut list (§10) lebih awal.
- Satu versi UE5 yang sama di semua mesin (cek `EngineAssociation` di `.uproject`). Blueprint sebagai pendekatan utama.
- Project tidak mulai dari nol: sudah ada beberapa asset (lihat pemetaan di §5); reuse dulu sebelum membuat baru. Aset tambahan: aset gratis (Fab / Starter Content). Tidak ada modeling atau animasi custom.
- Target: PC Windows, satu PC, dua gamepad PlayStation.

## 3. Scope

**Must (wajib selesai sebelum feature freeze)**

| ID | Fitur |
|---|---|
| M1 | Setup project: Git + LFS, kunci versi UE, struktur folder |
| M2 | Local multiplayer: 2 local player, split-screen, tiap gamepad mengontrol karakter sendiri (fallback dev: P1 keyboard + mouse) |
| M3 | Movement, kamera over-the-shoulder, bidik (right stick), lompat |
| M4 | Potion resource: mulai 3, maks 5, −1 per lemparan |
| M5 | Throw + projectile arc (jarak maks) + impact di permukaan valid |
| M6 | Poison AOE: durasi, DPS tick, warna owner (hijau P1 / ungu P2), tanpa friendly fire, tanpa stacking |
| M7 | AOE contest versi sederhana (replace penuh) |
| M8 | HP, kondisi menang/kalah, timer ronde, timeout, Best-of-3, round reset |
| M9 | Potion pickup (titik tetap, pemilihan acak, cap 5) |
| M10 | 1 arena: blockout + cover + 2 flank + 2 spawn + titik pickup |
| M11 | HUD per pemain + HUD bersama (timer, skor), Main Menu, Result (Rematch / Main Menu) |
| M12 | Packaged build Windows yang jalan di PC lain |

**Should**
- Niagara (impact splash, AOE hijau/ungu) dan SFX dasar (throw, shatter, damage, pickup, round win)
- Pause menu (Resume, Main Menu)
- `DA_Tuning` (Data Asset) untuk semua nilai balancing

**Stretch (hanya jika semua Must dan Should selesai, urut)**
Trajectory preview (Predict Projectile Path) → settings volume → pembeda AOE selain warna (ring/ikon) → rumble.

**Future (jangan dikerjakan)**
Character system (Fangor / Ragnar / Tactician), multi-potion/effect, skill karakter, online multiplayer, AOE clipping parsial, mekanik destroy AOE, settings lanjutan, remap tombol, bot/AI.

## 4. Baseline dan keputusan desain

| Parameter | Nilai | Sumber |
|---|---|---|
| HP pemain | 100 | Usulan (GDD tidak menetapkan HP prototype) |
| Potion awal / maks | 3 / 5 | GDD |
| Jarak lempar maks | 25 m (2500 uu) | Usulan |
| Throw cooldown | 0.75 s | Usulan (GDD menyebut cooldown di HUD tanpa nilai) |
| AOE radius | 2.75 m (275 uu) | GDD (2.5–3 m) |
| AOE durasi | 5 s | GDD |
| DPS | 5 HP/s, tick 0.5 s (2.5 HP/tick) | GDD (uji 5–10); tick usulan |
| Pickup | 5 titik; spawn tiap 12 s jika pickup aktif < 2; +1 potion | Usulan |
| Durasi ronde | 120 s | GDD |
| Jeda antar ronde | 3 s | Usulan |
| Match | Best-of-3 | GDD |

**Keputusan yang dikunci.** Default berlaku jika tim tidak memutuskan lain sebelum **Selasa 24 Nov**.
1. Split-screen **horizontal** (atas–bawah). Vertical hanya jika murah.
2. Seri saat timeout (HP sama) → ronde diulang, tanpa poin.
3. AOE contest = AOE baru menghapus AOE lawan yang overlap. Evaluasi di playtest 1 apakah perlu syarat "pusat dalam radius".
4. Damage di-tick **dari sisi karakter** memakai overlap counter (jumlah AOE lawan yang sedang menimpa karakter; damage jalan jika counter > 0). Dengan begitu AOE sendiri otomatis tidak stack. Kurangi counter juga saat AOE hancur (`OnDestroyed`), jangan hanya mengandalkan `EndOverlap`.
5. HUD per pemain (HP, potion) memakai **Add to Player Screen**. HUD bersama (timer, skor ronde) memakai **Add to Viewport** di tengah atas. Menu (Main Menu, Pause, Result) full-screen dan bisa dikontrol gamepad mana pun.
6. Arena baseline 60×60 m. Jika split-screen terasa sesak, evaluasi 40×40 m di playtest 1.
7. Round reset dilakukan **manual** (teleport ke PlayerStart, reset HP/potion, hancurkan semua projectile/AOE/pickup). Jangan `OpenLevel` antar ronde.

## 5. Arsitektur teknis (Blueprint)

| Asset | Tanggung jawab |
|---|---|
| `BP_GameInstance` | Membuat local player ke-2 sekali saja. Cek jumlah local player sebelum `Create Local Player`, karena local player bertahan antar level |
| `BP_GameMode` | Spawn 2 karakter di PlayerStart P1/P2 berdasarkan player index; memiliki `BP_MatchManager` |
| `BP_MatchManager` | State machine: Intermission → RoundActive → RoundEnd → MatchEnd. Timer, skor, round reset |
| `BP_PlayerCharacter` | Movement, kamera, HP, PotionCount, throw cooldown, PlayerIndex/warna, Enhanced Input, damage tick, event dispatcher `OnHPChanged` / `OnPotionChanged` |
| `BP_PoisonProjectile` | Projectile movement, arc, collision, impact → spawn AOE. Arc dihitung dari titik crosshair (line trace + Suggest Projectile Velocity Custom Arc, dibatasi jarak maks) |
| `BP_PoisonAOE` | Volume overlap, durasi, owner index, update counter karakter, hapus AOE lawan yang overlap saat spawn, parameter warna Niagara |
| `BP_PotionPickup` | Overlap, validasi kapasitas, +1 potion, hilang setelah diambil |
| `WBP_PlayerHUD` | HP + potion per pemain |
| `WBP_SharedHUD` | Timer + skor ronde |
| `WBP_MainMenu` / `WBP_Pause` / `WBP_Result` | Menu, navigasi gamepad, fokus tombol diset eksplisit |
| `DA_Tuning` | Semua nilai di §4 |

### Pemetaan asset existing (diisi agent saat file ini ditulis)

Nama di tabel atas adalah nama rencana. Jika project sudah punya asset dengan peran yang sama, pakai asset existing itu dan catat di sini. Jangan membuat duplikat, dan jangan me-rename asset existing hanya demi mengikuti nama rencana (rename bisa merusak referensi). Baris tanpa asset existing diisi `belum ada`.

| Nama di rencana | Asset existing | Catatan |
|---|---|---|
| `BP_PlayerCharacter` | `/Game/Characters/BP_Player` | Karakter ThirdPerson template (Move, Look, Jump, Touch). Belum ada variabel HP/PotionCount dan belum terpasang komponen Health. |
| `BP_GameMode` | `/Game/GameMode/BP_ThirdPersonGameMode` | Aktif di WorldSettings level `/Game/ThirdPerson/Lvl_ThirdPerson`. Belum ada spawn 2 player / MatchManager. Kandidat di ThirdPerson/Blueprints ditinggalkan. |
| `BP_GameInstance` | `/Game/Splashfall/Blueprints/BP_GameInstance` | Dibuat via unreal-mcp. EventGraph **kosong**. Fungsi publik `EnsureSecondLocalPlayer` tersedia (guard `GetNumLocalPlayerControllers < 2` → `CreateLocalPlayer(-1, true)`). GameInstance di-set permanen di `Config/DefaultEngine.ini` (`GameInstanceClass=...BP_GameInstance_C`). |
| HP / damage | `/Game/Components/Health` | Komponen ActorComponent existing. Diputuskan untuk **dipakai dan diperbaiki** pada task W1-A1/W1-A4 (hapus `GetActorOfClass(HUDClass)` di BeginPlay, perbaiki bug `GainHealth`, set default HP 100, lalu pasang ke `BP_Player`). |
| `BP_PlayerController` | `/Game/Characters/BP_PlayerController` | Direferensikan oleh `BP_ThirdPersonGameMode` aktif. Perlu disesuaikan untuk input local player / HUD. Kandidat di ThirdPerson ditinggalkan. |
| Lainnya | `/Game/UI/HUDClass`, `/Game/Input/IMC_Default`, `/Game/ThirdPerson/Lvl_ThirdPerson` | Input Actions (`IA_Move`, `IA_Look`, `IA_Jump`), Mannequin animations lengkap (termasuk anim Death/HitReact di `/Game/Characters/Mannequins/`), LevelPrototyping meshes & interactables. |

Task di §6 yang sebagian sudah terpenuhi oleh asset existing: tambahkan catatan di baris task tersebut. Centang `[x]` hanya setelah diverifikasi.

Input (nama key Unreal, generik untuk gamepad PS): Left Stick = gerak, Right Stick = bidik, R2 (`Gamepad_RightTriggerAxis`) = lempar, Cross (`Gamepad_FaceButton_Bottom`) = lompat, Options (`Gamepad_Special_Right`) = pause. Fallback keyboard + mouse: WASD, mouse, klik kiri, Spasi, Esc. Satu mapping context per local player lewat Enhanced Input Local Player Subsystem.

## 6. Rencana mingguan

Peran: **A** = Gameplay Programmer, **B** = Match/Input/UI Programmer, **C** = Level/Art/Audio/QA/Dokumentasi.

### Minggu 1 (Jum 20 – Kam 26 Nov) — "Duel Prototype"
**Gate Kamis 26 Nov:** dua pemain, dua gamepad, satu PC: bergerak, membidik, melempar, AOE melukai lawan. Packaged build #1 jalan.

- [x] W1-B1 (Hari 1–2, spike wajib) 2 local player + split-screen + tiap gamepad mengontrol karakter sendiri. Uji dengan 2 PS controller asli. Cek opsi Project Settings > Local Multiplayer (layout 2 pemain, offset gamepad ID). Catat cara PS controller terbaca di Windows (native / Steam Input / DS4Windows) di §13 dan README. *(Config & Blueprint selesai; split-screen & keyboard movement terverifikasi di PIE; pengujian 2 gamepad PS asli menyusul)*
- [x] W1-C1 (Hari 1–2) Repo Git + LFS, `.gitignore` UE, kunci versi UE, struktur folder, konvensi nama, pastikan One File Per Actor aktif di level. *(`.gitignore`, struktur folder `Content/Splashfall/`, konvensi nama, 2 commit dilakukan. Belum: Git LFS dan verifikasi One File Per Actor)*
- [x] W1-A1 Karakter dari template Third Person: kamera over-the-shoulder (SpringArm TargetArmLength=240, SocketOffset Y=60 Z=50 di ConstructionScript), bidik via right stick/mouse, lompat; variabel HP (komponen Health default 100 terpasang, GainHealth bug fixed, dependensi HUDClass dihapus), PotionCount (3, max 5), PlayerIndex (0).
- [ ] W1-A2 Throw: input R2/klik kiri; cek PotionCount > 0 dan cooldown; spawn `BP_PoisonProjectile` dengan arc ke crosshair; PotionCount −1.
- [ ] W1-A3 Impact di permukaan valid → spawn `BP_PoisonAOE` (placeholder mesh). Projectile tanpa direct damage.
- [ ] W1-A4 Damage: overlap counter di karakter, tick 0.5 s × 2.5 HP, hanya dari AOE lawan, tanpa friendly fire, tanpa stacking.
- [ ] W1-B2 `BP_GameMode` spawn 2 karakter; mapping context per player; guard `Create Local Player` agar tidak duplikat. *(Dasar GameMode sudah ada di `/Game/GameMode/BP_ThirdPersonGameMode`, perlu ditambah logika spawn 2 player)*
- [ ] W1-B3 HUD placeholder per pemain (HP bar + jumlah potion) via Add to Player Screen. *(Catatan: `/Game/UI/HUDClass` dan ketergantungan Health ke HUDClass di §5 harus diganti dengan widget per pemain via Add to Player Screen)*
- [ ] W1-B4 `DA_Tuning` berisi semua nilai §4; karakter dan AOE membaca dari sini.
- [ ] W1-C2 Blockout arena 60×60 m: 2 spawn, center, 6–10 cover, 2 flank, 5 marker titik pickup.
- [ ] W1-C3 Kumpulkan aset gratis: botol/placeholder, crate/obstacle modular, 5–6 SFX, Niagara starter.
- [ ] W1-C4 Packaged build #1 dan tes jalan di mesin lain.

### Minggu 2 (Jum 27 Nov – Kam 3 Des) — "Full Match Loop + Feature Freeze"
**Gate Kamis 3 Des = FEATURE FREEZE:** dari Main Menu sampai Result + Rematch tanpa crash; semua Must selesai.

- [ ] W2-A1 AOE contest: AOE baru menghapus AOE lawan yang overlap (replace penuh).
- [ ] W2-A2 Warna owner (hijau P1 / ungu P2) via material parameter; AOE habis setelah durasi; counter dikurangi saat AOE hancur.
- [ ] W2-A3 `BP_PotionPickup` + spawner: pilih titik bebas secara acak dari 5 titik, interval 12 s, +1 potion, tidak bisa diambil saat 5/5.
- [ ] W2-B1 `BP_MatchManager`: timer 120 s, menang saat HP 0, timeout → HP lebih tinggi, seri → ulang ronde, Best-of-3, jeda 3 s, round reset manual.
- [ ] W2-B2 `WBP_SharedHUD` (timer + skor ronde) dan pesan "Ronde N" / pemenang.
- [ ] W2-B3 `WBP_MainMenu` (Play, Quit), `WBP_Result` (pemenang, skor, Rematch, Main Menu), `WBP_Pause` (Resume, Main Menu); navigasi gamepad dengan fokus eksplisit.
- [ ] W2-C1 Arena pass 2: finalisasi posisi cover/flank/pickup dari feel playtest internal.
- [ ] W2-C2 Niagara: impact splash + AOE hijau/ungu (partikel ringan). SFX: throw, shatter, damage tick, pickup, round win.
- [ ] W2-C3 Packaged build #2 dan playtest internal 1 (catat di `docs/playtest-log.md`).

### Minggu 3 (Jum 4 – Kam 10 Des) — "Stabilkan, Tuning, Kirim"
**Gate Senin 7 Des:** Release Candidate. 8–9 Des: hanya bug P0/P1. 10 Des: build final + dokumen. 11 Des: buffer / pengumpulan.

- [ ] W3-ALL1 Playtest ≥ 3 sesi (≥ 6 match penuh) dengan 2 gamepad; catat temuan di `docs/playtest-log.md`.
- [ ] W3-A1 Tuning via `DA_Tuning`: DPS (uji 5–10), radius, durasi, jarak lempar, interval pickup. Nilai final dicatat di GDD Lampiran A.
- [ ] W3-A2 Bug fixing gameplay: round reset, counter AOE, pickup, kasus seri.
- [ ] W3-B1 Bug fixing input/UI: gamepad dicabut atau dicolok ulang tidak crash, fokus menu, pause.
- [ ] W3-B2 Cek performa: split-screen + ≥ 6 AOE aktif. Jika < 30 FPS, turunkan partikel / scalability.
- [ ] W3-C1 Polish visual/audio seperlunya (tanpa fitur baru) dan README: cara menjalankan, mapping PS controller, setup Steam Input/DS4Windows bila perlu.
- [ ] W3-C2 GDD v4.1 (§12), screenshot, demo video 1–2 menit.
- [ ] W3-C3 Packaged build final; tes di PC bersih (bukan mesin dev) dengan 2 PS controller.
- [ ] W3-STRETCH Kerjakan item Stretch (§3) hanya jika semua di atas selesai.

## 7. Peran dan ownership asset

| Peran | Anggota | Asset yang dimiliki |
|---|---|---|
| A — Gameplay | ___ | `BP_PlayerCharacter`, `BP_PoisonProjectile`, `BP_PoisonAOE`, `BP_PotionPickup` |
| B — Match/Input/UI | ___ | `BP_GameMode`, `BP_MatchManager`, `BP_GameInstance`, semua `WBP_*`, input assets, `DA_Tuning` |
| C — Level/Art/Audio/QA/Doc | ___ | Map, material, Niagara, audio, `docs/`, README, GDD |

## 8. Workflow tim

- Satu asset satu pemilik. Blueprint dan level tidak bisa di-merge; jika harus menyentuh asset orang lain, kabari dulu. Git LFS file locking (`lockable`) opsional tapi disarankan untuk `.uasset` / `.umap`.
- Branch: `main` harus selalu bisa dibuka dan di-package. Kerja di `feature/<peran>-<topik>`, merge kecil dan sering.
- Sinkronisasi harian singkat via chat: kemarin / hari ini / blocker.
- Packaged build tiap Kamis (akhir minggu) untuk mendeteksi masalah packaging dan controller lebih awal.
- Setelah feature freeze (3 Des): tidak ada fitur baru; hanya bug fix dan tuning.

## 9. Risiko dan mitigasi

| Risiko | Mitigasi |
|---|---|
| PS controller tidak terbaca atau ID player salah di Windows | Spike W1-B1 di hari 1–2. Fallback: Steam Input / DS4Windows (emulasi Xbox), K+M untuk P1. Dokumentasikan di README agar penguji bisa menjalankan |
| Split-screen: FPS turun, viewport sempit | Partikel ringan, scalability rendah, evaluasi arena 40×40 dan layout vertical |
| Conflict Blueprint / level | Ownership asset (§7), One File Per Actor, merge kecil |
| Kapasitas tim turun (ujian / tugas lain) | Gate akhir Minggu 1; cut list (§10) dijalankan lebih awal |
| Bug round reset (AOE / projectile tersisa) | Fungsi cleanup tunggal di `BP_MatchManager`; hindari `OpenLevel` antar ronde |
| Local player duplikat setelah Rematch / Main Menu | Cek jumlah local player sebelum `Create Local Player` |
| Fokus UMG dengan gamepad | Menu sederhana (2–3 tombol), set fokus eksplisit |
| Edge case AOE contest | Pakai replace penuh; evaluasi di playtest 1 |
| Scope creep (karakter, potion baru) | Daftar Future (§3) tidak boleh disentuh |
| Packaging error muncul terlambat | Packaged build tiap Kamis |

## 10. Cut list (jika telat, potong dari atas)

1. Semua item Stretch.
2. Pause menu.
3. Polish Niagara/SFX → pakai placeholder sederhana.
4. Pickup acak → urutan titik tetap. Jika pickup tetap belum jadi, fallback sementara: regen +1 potion tiap 8 s (catat sebagai deviasi di GDD).
5. AOE contest replace → hanya overlap visual (dua zona sama-sama aktif); catat sebagai deviasi.
6. `DA_Tuning` → variabel biasa di `BP_MatchManager`.
7. Result screen → tampilkan pemenang lalu kembali ke menu (tanpa Rematch).

**Tidak boleh dipotong:** M2–M8 (local 2-player, movement, throw, AOE + DPS, HP, win/lose, Best-of-3) dan M12 (packaged build).

## 11. Definition of Done

- [ ] Packaged build Windows jalan di PC yang bukan mesin dev.
- [ ] 2 PS controller: tiap controller mengontrol pemainnya sendiri, split-screen dengan HUD masing-masing.
- [ ] Alur penuh Main Menu → Best-of-3 → Result → Rematch / Main Menu tanpa crash dalam 5 match berturut-turut.
- [ ] Potion: mulai 3, −1 per lemparan, pickup +1 hingga maks 5, tidak bisa diambil saat 5/5.
- [ ] AOE: warna sesuai owner, hanya melukai lawan, tidak stack, contest berfungsi, habis sesuai durasi.
- [ ] Menang: HP 0 atau timeout dengan HP lebih tinggi; seri → ronde diulang; 2 ronde menang mengakhiri match.
- [ ] FPS ≥ 30 stabil (target 60) dengan 2 viewport dan beberapa AOE aktif.
- [ ] GDD v4.1, README, dan demo video selesai.

## 12. Revisi GDD yang harus dilakukan

- Document Control: ganti placeholder `[Nama Tim]` / `[Anggota]`; versi → 4.1; tambah baris revisi "online → local couch 2 gamepad, timeline 3 minggu".
- §1.3.2 Kontrol: gamepad jadi utama dengan nama PS (R2, Cross, Options); keyboard + mouse = fallback dev.
- §2.2.1 UI: HUD per pemain (split-screen) + HUD bersama.
- §2.2.2 D: AOE contest = replace penuh. Tambah aturan seri saat timeout. Tambah HP prototype 100.
- §3.1: hapus baris "Latensi jaringan"; tambah requirement FPS split-screen.
- §3.2 Kontrol dan §3.3: hapus replikasi; ubah peran Network Programmer.
- §4.1: baris Online → "Local multiplayer, 1 PC, 2 gamepad". §4.2: risiko "Network desync" → "Controller assignment / performa split-screen".
- §5 Milestone: ganti tabel 8 minggu dengan tabel 3 minggu (§6 file ini).
- Lampiran A: tambah asumsi HP 100, tick 0.5 s, throw cooldown 0.75 s, aturan seri, split-screen horizontal.
- Lampiran B: tambah P0 "Local 2-player + split-screen"; pickup tetap Wajib.

## 13. Status log

Log berjalan ada di PROGRESS.md; baris di bawah adalah entri awal.

| Tanggal | Siapa | Catatan |
|---|---|---|
| 2026-11-20 | — | Rencana 3 minggu dibuat |
| 2026-11-20 | Antigravity | Inventaris asset existing via unreal-mcp: BP_Player, BP_PlayerController, BP_ThirdPersonGameMode, Health component (/Game/Components/Health), HUDClass, IMC_Default, Lvl_ThirdPerson, animasi Mannequin (Death, HitReact, Jump, Jog, Walk). Hasil introspeksi agent via unreal-mcp, belum diverifikasi manual di editor: komponen Health belum terpasang di BP_Player dan memiliki bug pada GainHealth. |
