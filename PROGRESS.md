# PROGRESS.md — Splashfall: Poison Arena

Dokumen pelacak kondisi terkini proyek. Berfungsi sebagai konteks langsung antar sesi kerja.
*Sumber rencana & aturan tetap mengacu pada [AGENTS.md](file:///C:/Kuliah/Semester_5/Praktikum%20Game/UAS/ProjectUAS/AGENTS.md).*

---

## 1. Posisi Sekarang
- **Tanggal Update**: 20 November 2026
- **Tahap**: Minggu 1, Hari 1 — *Duel Prototype & Project Setup*
- **Ringkasan**:
  1. Dokumen perencanaan 3 minggu telah dibuat dan dikunci di `AGENTS.md` dengan scope couch 1v1 local split-screen (2 gamepad).
  2. 4 Prasyarat awal telah diputuskan: komponen `Health` diputuskan untuk dipakai dan diperbaiki (pada W1-A1/W1-A4); GameMode aktif `/Game/GameMode/BP_ThirdPersonGameMode` dan PlayerController aktif `/Game/Characters/BP_PlayerController`; file Git Modified dipertahankan.
  3. Setup tooling awal: skrip helper dipindahkan ke `tools/unreal_mcp_client.py`, dan `.gitignore` telah diperbarui dengan `.gemini/`, `.agents/`, dan `tools/`.
  4. Pengaturan split-screen horizontal dan Gamepad ID assignment terverifikasi di `DefaultEngine.ini`.
  5. **Perbaikan Lifecycle Local Player (Spike W1-B1) SELESAI**:
      - BP_GameInstance:EventGraph sudah bersih — 6 node lama dihapus via delete_node.
      - Fungsi publik EnsureSecondLocalPlayer tetap ada di BP_GameInstance (guard GetNumLocalPlayerControllers < 2 -> CreateLocalPlayer(-1, true)).
      - EventBeginPlay di BP_ThirdPersonGameMode ditulis ulang **tanpa cast** — inline GetNumLocalPlayerControllers < 2 -> CreateLocalPlayer(-1, true). Compile + save berhasil.
   6. **Fix DefaultEngine.ini — root cause split-screen gagal (belum diverifikasi)**:
      - Setelah editor crash, gameInstanceClass ke-reset ke default /Script/Engine.GameInstance. Fix: ditambahkan GameInstanceClass=/Game/Splashfall/Blueprints/BP_GameInstance.BP_GameInstance_C ke config.
      - GlobalDefaultGameMode dikoreksi dari /Game/ThirdPerson/Blueprints/BP_ThirdPersonGameMode -> /Game/GameMode/BP_ThirdPersonGameMode (konsisten dengan WorldSettings).
      - **Perlu restart editor + tes PIE** untuk verifikasi bahwa split-screen aktif.

---

## 2. Next Action
- **Tindakan**:
  1. **Restart Unreal Editor** agar `DefaultEngine.ini` yang sudah diperbarui dibaca ulang (GameInstanceClass & GlobalDefaultGameMode baru).
  2. Uji **Play In Editor (PIE)** — verifikasi layar terbagi 2 (split-screen horizontal) dan masing-masing player bisa bergerak.
  3. Jika split-screen sudah OK, lanjut ke **W1-A1** (tambah variabel HP/PotionCount/PlayerIndex ke `BP_Player`, kamera over-the-shoulder).
- **Pelaksana**: Manusia di Unreal Editor (restart + PIE test), lalu agent untuk W1-A1.
- **Prasyarat**: Sudah terpenuhi seluruhnya.

---

## 3. Keputusan

### Sudah Diputuskan
| Tanggal | Oleh | Keputusan |
|---|---|---|
| 2026-11-20 | Tim | Scope disederhanakan dari online multiplayer menjadi local couch PvP 1v1 (2 gamepad PS, split-screen) di 1 PC; durasi proyek 3 minggu (deadline 11 Des 2026). |
| 2026-11-20 | Tim | Pendekatan pengembangan berbasis Blueprint dengan Unreal Engine 5. |
| 2026-11-20 | Tim / User | Komponen Health existing `/Game/Components/Health` diputuskan **dipakai dan diperbaiki** pada task W1-A1/W1-A4 (perbaiki `GainHealth`, set default HP 100, hapus dependensi `HUDClass`, pasang ke `BP_Player`). |
| 2026-11-20 | Tim / User | GameMode aktif dikonfirmasi `/Game/GameMode/BP_ThirdPersonGameMode` (aktif di WorldSettings `Lvl_ThirdPerson`) dan PlayerController aktif `/Game/Characters/BP_PlayerController`. |
| 2026-11-20 | Tim / User | File Git berstatus *Modified* (`Health.uasset` & `BP_ThirdPersonGameMode.uasset`) dipertahankan apa adanya (tidak di-reset). |
| 2026-11-20 | Antigravity | `BP_GameInstance` dibuat di `/Game/Splashfall/Blueprints/BP_GameInstance` via Unreal MCP dengan guard `GetNumLocalPlayerControllers < 2` sebelum `CreateLocalPlayer(-1)`. |

> *Catatan*: Seluruh parameter tuning di `AGENTS.md` §4 (HP 100, potion 3/5, jarak lempar 25 m, cooldown 0.75 s, AOE radius 2.75 m / durasi 5 s / DPS 5 HP/s, durasi ronde 120 s) saat ini **masih berstatus usulan baseline** dan belum dikonfirmasi melalui playtest.

### Masih Terbuka
- **Layout Split-screen**: Default dikunci horizontal (atas-bawah); opsi vertikal dievaluasi jika ruang pandang terasa terlalu sempit pada playtest 1.
- **Aturan Timeout Seri**: Default ronde diulang tanpa poin jika HP sama saat timeout; menunggu konfirmasi final tim.
- **Ukuran Arena**: Baseline 60×60 m; evaluasi alternatif 40×40 m pada playtest 1.

---

## 4. Fakta Project

| Fakta | Status | Keterangan |
|---|---|---|
| `BP_GameInstance` selesai dibuat | Terverifikasi di disk via MCP | Berada di `/Game/Splashfall/Blueprints/BP_GameInstance`, parent class `GameInstance`. EventGraph **kosong**. Fungsi publik `EnsureSecondLocalPlayer` tersedia (guard `GetNumLocalPlayerControllers < 2` → `CreateLocalPlayer(-1, true)`). |
| GameMode aktif terkonfirmasi | Terverifikasi via MCP & WorldSettings | `/Game/GameMode/BP_ThirdPersonGameMode` aktif pada `defaultGameMode` WorldSettings level `Lvl_ThirdPerson`. |
| PlayerController aktif terkonfirmasi | Terverifikasi via MCP CDO | `/Game/Characters/BP_PlayerController` digunakan oleh GameMode aktif. |
| Pengaturan Splitscreen & Gamepad di Config | Terverifikasi di `DefaultEngine.ini` | `bUseSplitscreen=true`, `twoPlayerSplitscreenLayout=Horizontal`, `bOffsetPlayerGamepadIds=false` (artinya Skip Assigning Gamepad to P1 = False). |
| `BP_Player` tidak memiliki komponen `Health` terpasang | Hasil introspeksi agent | Properti class CDO hanya berisi Capsule, Mesh, CharacterMovement, CameraBoom, dan FollowCamera. |
| Komponen `Health` tidak direferensikan oleh aset lain | Hasil introspeksi agent | Panggilan `get_referencers` pada `/Game/Components/Health` menghasilkan list kosong `[]`. |
| Fungsi `GainHealth` pada `Health` men-set HP ke 0.0 | Hasil introspeksi agent | Grafik DSL `GainHealth` memanggil `(SetCurrentHealth 0.0)` alih-alih menambahkan nilai `Health To Add` (akan diperbaiki di W1-A1). |
| `EventBeginPlay` pada `Health` memanggil `GetActorOfClass(HUDClass)` | Hasil introspeksi agent | Tidak kompatibel dengan local split-screen (akan dihapus di W1-A1). |
| Nilai default `CurrentHealth` dan `MaxHealth` pada class `Health` bernilai 0 | Hasil introspeksi agent | Terbaca `0` pada CDO `/Game/Components/Health.Default__Health_C`; akan diset 100 di W1-A1. |
| Enhanced Input yang sudah tersedia | Hasil introspeksi agent | `IA_Move`, `IA_Look`, `IA_MouseLook`, `IA_Jump` di `/Game/Input/Actions/` serta `IMC_Default`. |
| Input Actions yang belum tersedia | Hasil introspeksi agent | Input Action untuk Throw Potion (R2 / Gamepad Right Trigger) dan Pause Menu (Options). |
| Animasi Mannequin lengkap | Hasil introspeksi agent | Tersedia di `/Game/Characters/Mannequins/Anims/` termasuk Jump, Jog, Walk, Death, dan HitReact. |
| Engine Association | Hasil introspeksi agent | Menggunakan build UE 5.8 (Identifier: `{7A12E277-42D4-CC46-1CCF-31986923C4AF}`). |

---

## 5. Isu dan Blocker

1. **Aset Termodifikasi di Git Status**:
   File `Content/Components/Health.uasset` dan `Content/GameMode/BP_ThirdPersonGameMode.uasset` tercatat *Modified* pada status git. Sesuai keputusan user, file ini dibiarkan (tidak di-reset dan belum di-commit).
2. **Untracked Directory Tooling (SELESAI)**:
   Folder `.gemini/`, `.agents/`, dan `tools/` sudah didaftarkan ke `.gitignore`.

---

## 6. Catatan Tooling Unreal MCP

### Lokasi Helper & Mekanisme Komunikasi
- Skrip helper: `tools/unreal_mcp_client.py` (di root project).
- Transport: JSON-RPC 2.0 melalui HTTP POST ke endpoint `http://127.0.0.1:8000/mcp`.
- Alur koneksi:
  1. POST `initialize` dengan parameter `{"protocolVersion": "2024-11-05", ...}`.
  2. Tangkap header respons `Mcp-Session-Id` dan sertakan pada header semua HTTP request berikutnya.
  3. POST notifikasi `{"jsonrpc": "2.0", "method": "notifications/initialized"}`.
  4. Eksekusi tool melalui `tools/call` dengan membungkus parameter ke tool meta `call_tool`.

### Fungsi Toolset yang Berhasil Digunakan
- **Toolset Discovery**: `list_toolsets`, `describe_toolset`.
- **AssetTools (`editor_toolset.toolsets.asset.AssetTools`)**:
  - `find_assets`: mencari daftar aset berdasarkan path folder dan nama.
  - `exists`: memeriksa keberadaan aset di disk/project.
  - `get_referencers`: melacak aset lain yang mengonsumsi suatu aset.
  - `save_assets`: menyimpan aset biner ke disk (`asset_paths: [...]`).
- **BlueprintTools (`editor_toolset.toolsets.blueprint.BlueprintTools`)**:
  - `create`: membuat aset Blueprint baru dengan spesifikasi folder, nama, dan parent class.
  - `get_parent`: melihat parent class Blueprint.
  - `list_variables`, `list_functions`, `list_graphs`, `list_event_dispatchers`, `list_events`.
  - `add_function_graph`: menambah grafik fungsi baru ke Blueprint (misal `EnsureSecondLocalPlayer`).
  - `find_nodes`: mencari referensi node di graph berdasarkan judul string.
  - `delete_node`: menghapus node visual script dari graph.
  - `find_node_types`: mencari type ID node di graph berdasarkan filter string dan context pins.
  - `get_node_type_pins`: membaca pin input/output serta tipe data suatu type ID node.
  - `read_graph_dsl`: membaca logika visual script Blueprint ke format S-expression DSL.
  - `write_graph_dsl`: menulis struktur graph visual script secara deklaratif.
  - `compile_blueprint`: memicu compile engine pada Blueprint dan memverifikasi ketiadaan compile error.
- **ObjectTools (`editor_toolset.toolsets.object.ObjectTools`)**:
  - `list_properties`: membaca skema properti UObject/CDO.
  - `get_properties`: membaca nilai variabel langsung dari instance/CDO.
  - `set_properties`: mengatur nilai properti UObject/CDO di memori.

### Kesalahan yang Ditemui & Solusinya (Lessons Learned)
1. **Kegagalan Handshake Otomatis Antigravity**:
   Language Server Antigravity mengirimkan pesan `notifications/roots/list_changed` setelah inisialisasi. Plugin Unreal MCP belum mengimplementasikan method ini dan melempar *400 Bad Request*, yang menyebabkan Antigravity memutuskan sesi MCP secara sepihak.
   *Solusi*: Gunakan skrip Python standalone (`tools/unreal_mcp_client.py`) untuk berkomunikasi langsung ke port 8000 tanpa perantara Language Server internal.
2. **Format Parameter `tool_name` pada `call_tool`**:
   Parameter `tool_name` pada `call_tool` **tidak boleh** memuat namespace toolset.
   *Contoh*: Gunakan `tool_name: "read_graph_dsl"` dan `toolset_name: "editor_toolset.toolsets.blueprint.BlueprintTools"`. Jika ditulis `tool_name: "editor_toolset.toolsets.blueprint.BlueprintTools.read_graph_dsl"`, server mengembalikan error `"Unknown tool"`.
3. **Pencarian Nama pada `find_assets`**:
   Parameter `name` pada `find_assets` tidak mendukung wildcard `*` (pencarian `*Health*` mengembalikan list kosong).
   *Solusi*: Masukkan nama persis (misal `"Health"`) atau gunakan string kosong `""` untuk me-retrieve semua aset di dalam `folder_path` secara rekursif.
4. **Parameter `list_folders`**:
   Menggunakan nama parameter `root_path`, bukan `folder_path`.
5. **Parameter `get_properties`**:
   Membutuhkan parameter `instance` berupa object dictionary `{"refPath": "..."}`, bukan `object`.
6. **Inspeksi Komponen Blueprint vs Actor Instance**:
   `ActorTools.get_components` hanya bekerja untuk actor instance yang ada di level aktif. Jika diberi path class Blueprint (misal `/Game/Characters/BP_Player.BP_Player`), server menghasilkan error `"not valid Actor"`.
   *Solusi*: Untuk memeriksa komponen dan variabel class Blueprint, gunakan `ObjectTools.list_properties` pada Class Default Object (`/Game/.../BP_Name.Default__BP_Name_C`) atau periksa referensi via `AssetTools.get_referencers`.
7. **Pencarian Node Type pada `find_node_types`**:
   Memerlukan input parameter `type_id_filter` (string) dan `context_pins` (array, bisa dilempar `[]` untuk pencarian global tanpa pin context).
8. **Penyimpanan Aset pada `save_assets`**:
   Memerlukan parameter `asset_paths` berupa array string (bukan `assets` berupa array object).
9. **Crash Editor pada Empty Event di `write_graph_dsl`**:
   Menulis form DSL event kosong tanpa statement seperti `(event EventInit)` memicu unhandled exception di parser C++ plugin MCP sehingga Unreal Editor crash (`ConnectionResetError: [WinError 10054]`).
   *Solusi*: Jangan pernah mengirim form event kosong ke `write_graph_dsl`. Untuk membersihkan atau mengosongkan node dari suatu grafik, gunakan kombinasi `find_nodes` untuk mendapatkan node referensi lalu panggil `delete_node`.

---

## 7. Log Sesi

| Tanggal | Siapa | Dikerjakan | Hasil |
|---|---|---|---|
| 2026-11-20 | Antigravity | Setup dokumen rencana `AGENTS.md`, introspeksi aset via Unreal MCP, analisis detail `Health` & `BP_Player`, penyusunan `PROGRESS.md`. | Seluruh aset terinventarisasi, bug `GainHealth` & dependensi `HUDClass` terpetakan, dokumen acuan kerja siap tanpa modifikasi biner. |
| 2026-11-20 | Antigravity | Pengecekan GameMode & PlayerController aktif, setup `.gitignore` (`.gemini/`, `.agents/`, `tools/`), pemindahan helper MCP ke `tools/unreal_mcp_client.py`, pembuatan `BP_GameInstance` di `/Game/Splashfall/Blueprints/` dengan logic guard `EventInit` (`GetNumLocalPlayerControllers < 2` -> `CreateLocalPlayer(-1)`), verifikasi compile & save via Unreal MCP. | Fondasi spike W1-B1 siap, `BP_GameInstance` ter-compile & tersimpan di disk tanpa error. Pengaturan split-screen horizontal terkonfirmasi aktif di config. |
| 2026-11-20 | Antigravity | Refactor lifecycle local player: buat fungsi `EnsureSecondLocalPlayer` di `BP_GameInstance`, panggil pada `Event BeginPlay` di `BP_ThirdPersonGameMode`. Pembersihan node lama `EventInit` di `BP_GameInstance:EventGraph` menggunakan `find_nodes` & `delete_node` via MCP setelah editor restart. | Kedua Blueprint sukses di-compile dan di-save tanpa error. Error "No game mode set" teratasi secara arsitektural. |
| 2026-11-20 | Antigravity | Investigasi split-screen gagal (layar masih 1 player): (1) Output Log PIE tidak ada error/warning LocalPlayer → konfirmasi `EnsureSecondLocalPlayer` tidak pernah terpanggil. (2) `gameInstanceClass` terkonfirmasi ke-reset ke default `/Script/Engine.GameInstance` setelah editor crash. (3) Fix: tambah `GameInstanceClass` & koreksi `GlobalDefaultGameMode` di `Config/DefaultEngine.ini` supaya persist setelah restart. (4) Tulis ulang `EventBeginPlay` di `BP_ThirdPersonGameMode` tanpa cast—inline `GetNumLocalPlayerControllers < 2 → CreateLocalPlayer(-1, true)`—compile ✅ save ✅. | Root cause teridentifikasi dan diperbaiki. `DefaultEngine.ini` kini menyimpan `GameInstanceClass` dan `GlobalDefaultGameMode` yang benar. Perlu restart editor + tes PIE ulang oleh user. |
