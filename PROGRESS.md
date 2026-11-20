# PROGRESS.md  ESplashfall: Poison Arena

Dokumen pelacak kondisi terkini proyek. Berfungsi sebagai konteks langsung antar sesi kerja.
*Sumber rencana & aturan tetap mengacu pada [AGENTS.md](file:///C:/Kuliah/Semester_5/Praktikum%20Game/UAS/ProjectUAS/AGENTS.md).*

---

## 1. Posisi Sekarang
- **Tanggal Update**: 20 November 2026
- **Tahap**: Minggu 1  E*Duel Prototype* (W1-A1–A4 ✁E W1-B1 ✁E W1-B2 ✁E
- **Ringkasan**:
  1. Dokumen perencanaan 3 minggu telah dibuat dan dikunci di `AGENTS.md` dengan scope couch 1v1 local split-screen (2 gamepad).
  2. 4 Prasyarat awal telah diputuskan: komponen `Health` diputuskan untuk dipakai dan diperbaiki (pada W1-A1/W1-A4); GameMode aktif `/Game/GameMode/BP_ThirdPersonGameMode` dan PlayerController aktif `/Game/Characters/BP_PlayerController`; file Git Modified dipertahankan.
  3. Setup tooling awal: skrip helper dipindahkan ke `tools/unreal_mcp_client.py`, dan `.gitignore` telah diperbarui dengan `.gemini/`, `.agents/`, dan `tools/`.
  4. Pengaturan split-screen horizontal dan Gamepad ID assignment terverifikasi di `DefaultEngine.ini`.
  5. **Perbaikan Lifecycle Local Player (Spike W1-B1) SELESAI**:
      - BP_GameInstance:EventGraph sudah bersih  E6 node lama dihapus via delete_node.
      - Fungsi publik EnsureSecondLocalPlayer tetap ada di BP_GameInstance (guard GetNumLocalPlayerControllers < 2 -> CreateLocalPlayer(-1, true)).
      - EventBeginPlay di BP_ThirdPersonGameMode ditulis ulang **tanpa cast**  Einline GetNumLocalPlayerControllers < 2 -> CreateLocalPlayer(-1, true). Compile + save berhasil.
   6. **Split-Screen Horizontal Terverifikasi (Spike W1-B1)**:
      - Layar split-screen horizontal 2 player telah muncul di PIE setelah restart editor.
   7. **Perbaikan Input Controller & Verifikasi Karakter (Spike W1-B1)**:
      - `BP_PlayerController:EventGraph` diperbaiki via MCP: `EventBeginPlay -> DelayUntilNextTick -> if (IsLocalPlayerController) -> AddMappingContext(IMC_Default, 0) & AddMappingContext(IMC_MouseLook, 0)` tanpa argumen integer pada getter subsystem.
      - Hasil tes PIE oleh user: karakter berhasil bergerak dan membidik via keyboard/mouse tanpa error runtime `Accessed None`. Verifikasi 2 gamepad PS asli menyusul di tahap playtest.
   8. **Task W1-A1 SELESAI (Karakter, Health Component, dan Kamera OTS)**:
      - `/Game/Components/Health`: Bug `GainHealth` diperbaiki dengan menghubungkan output `Clamp(Float)` ke `SetCurrentHealth`. Pemanggilan `GetActorOfClass(HUDClass)` dan `SetHUD` pada `EventBeginPlay` telah dibersihkan. Default value `CurrentHealth = 100.0` dan `MaxHealth = 100.0` telah diset pada CDO. Compile ✁E save ✁E
      - `/Game/Characters/BP_Player`: Komponen `/Game/Components/Health` berhasil dipasangkan (`Variables|Default|GetHealth` tersedia). Variabel `PlayerIndex` (int, default 0), `PotionCount` (int, default 3), dan `MaxPotionCount` (int, default 5) telah ditambahkan dan diset pada CDO.
      - Kamera Over-The-Shoulder (OTS): Dikonfigurasi pada `UserConstructionScript` `BP_Player` (`TargetArmLength = 240.0`, `SocketOffset = (X=0.0, Y=60.0, Z=50.0)`). Compile ✁E save ✁E
   9. **Task W1-A2 SELESAI (Mekanik Lempar / Throw Projectile)**:
      - Input Action `/Game/Input/Actions/IA_Throw` (Boolean) dibuat dan didaftarkan pada `/Game/Input/IMC_Default` untuk `Gamepad_RightTrigger` (R2) dan `LeftMouseButton` (fallback dev).
      - `/Game/Splashfall/Blueprints/BP_PoisonProjectile` dibuat (Parent: Actor) dengan root `SphereComponent` (radius 15, BlockAllDynamic), visual `StaticMeshComponent` (Sphere scale 0.25, NoCollision), `ProjectileMovementComponent` (InitialSpeed=1400, MaxSpeed=1400, GravityScale=1.0, bRotationFollowsVelocity=True), dan CDO `initialLifeSpan = 5.0`. Compile ✁E save ✁E
      - `/Game/Characters/BP_Player`: Variabel `ThrowCooldown` (float, default 0.75) dan `bCanThrow` (bool, default true) ditambahkan. Fungsi `ThrowPotion` mengkalkulasi spawn transform di depan aim rotation karakter (`GetBaseAimRotation`), mengurangi `PotionCount - 1`, dan memanggil `SpawnActorFromClass` (`BP_PoisonProjectile`). Pada `EventGraph`, event `IA_Throw:Started` disambungkan dengan guard `(PotionCount > 0) AND bCanThrow` -> `SetCanThrow(false)` -> `ThrowPotion` -> `Delay(ThrowCooldown)` -> `SetCanThrow(true)`. Compile ✁E save ✁E
   10. **Task W1-A3 SELESAI (Impact & Spawn BP_PoisonAOE Placeholder)**:
      - `/Game/Splashfall/Blueprints/BP_PoisonAOE` dibuat (Parent: Actor): `SphereComponent` (`AOECollision`, radius 275.0, collision profile `OverlapAllDynamic`, `bGenerateOverlapEvents = true`), visual placeholder `StaticMeshComponent` (`AOEMesh`, `/Engine/BasicShapes/Cylinder.Cylinder`, relative scale `(5.5, 5.5, 0.05)`, collision profile `NoCollision`), dan CDO `initialLifeSpan = 5.0` (durasi 5 detik). Compile ✁E save ✁E
      - `/Game/Splashfall/Blueprints/BP_PoisonProjectile`: Mengimplementasikan `Collision|EventHit` di `EventGraph` dengan guard `Other != GetOwner()`. Ketika proyektil menabrak permukaan valid / aktor lain di luar pelempar: memanggil `SpawnActorBPPoisonAOE` di titik `HitLocation` dengan `CollisionHandling = AlwaysSpawn` dan `Owner = GetOwner()`, lalu memanggil `DestroyActor(self)`. `Physics|SetNotifyRigidBodyCollision` diaktifkan pada `CollisionSphere`. Tanpa direct damage dari proyektil. Compile ✁E save ✁E
   11. **Task W1-A4 SELESAI & TERVERIFIKASI PIE (Damage Overlap Counter & Poison Tick di BP_Player)**:
      - Variabel baru di `BP_Player`: `PoisonOverlapCounter` (int, default 0), `PoisonDamagePerTick` (float, default 2.5), dan `PoisonTickInterval` (float, default 0.5) diset pada CDO.
      - Komponen `Health` di-instansiasi pada `BP_Player:UserConstructionScript` via `AddComponent|Custom|AddHealth` -> `SetHealth` sehingga tidak `None` saat runtime.
      - Fungsi `ApplyPoisonTick` dibuat via DSL di `BP_Player`: guard `PoisonOverlapCounter > 0` dan `Utilities|IsValid Health` -> panggil `Health -> TakeDamage(PoisonDamagePerTick)` (2.5 HP/0.5s = 5 DPS) -> `PrintString` menampilkan nama aktor dan sisa HP pemain secara real-time.
      - `EventActorBeginOverlap` di-wire incremental via MCP `create_node` + `connect_pins` (tanpa menimpa Enhanced Input): Cast ke `BP_PoisonAOE` -> Guard `GetOwner != self` (anti-friendly-fire) -> increment `PoisonOverlapCounter + 1` -> jika `PoisonOverlapCounter == 1` panggil `SetTimerbyFunctionName("ApplyPoisonTick", PoisonTickInterval, true)` (anti-stacking).
      - `EventActorEndOverlap` di-wire incremental: Cast ke `BP_PoisonAOE` -> Guard `GetOwner != self` -> decrement `PoisonOverlapCounter = Max(0, PoisonOverlapCounter - 1)` -> jika `PoisonOverlapCounter <= 0` panggil `ClearTimerbyFunctionName("ApplyPoisonTick")`.
      - Perbaikan attachment collision `BP_PoisonAOE` ke `DefaultSceneRoot` dan `SetGenerateOverlapEvents(true)`.
      - Hasil tes PIE user: **SUKSES**. Karakter terkena racun musuh mengalami pengurangan HP bertahap (100 -> 97.5 -> 95.0 dst.) dan berhenti saat keluar dari genangan. Compile ✁E save dirty assets ✁E

    12. **Task W1-B2 SELESAI (2 PlayerStart & Assign PlayerIndex via EventPossessed)**:
       - 2 `PlayerStart` dikonfigurasi di `/Game/ThirdPerson/Lvl_ThirdPerson`:
         - `PlayerStart_P1` di Loc (-1000, 0, 102), Yaw 0 (menghadap +X ke center arena).
         - `PlayerStart_P2` di Loc (1000, 0, 102), Yaw 180 (menghadap -X ke center arena).
       - Guard `GetNumLocalPlayerControllers < 2 -> CreateLocalPlayer(-1, true)` di `BP_ThirdPersonGameMode:EventBeginPlay` tetap utuh dan tidak duplikat.
       - `BP_Player:EventGraph` mendapat chain incremental (tanpa menimpa Enhanced Input): `EventPossessed -> CastToPlayerController -> GetLocalPlayerControllerID -> SetPlayerIndex`. Debug `PrintString` selama 10 detik muncul saat possess.
       - Compile `BP_Player` OK, Compile `BP_ThirdPersonGameMode` OK, Save level + blueprint OK.
    13. **Task W1-B3 SELESAI & TERVERIFIKASI PIE (HUD Per Pemain Real-Time via Add to Player Screen)**:
       - `WBP_PlayerHUD` dibuat di `/Game/Splashfall/UI/WBP_PlayerHUD` (Parent: `UserWidget`):
         - Widget tree: `CanvasPanel_Root` -> `VBox_HUD` -> `HP_Text` (`TextBlock`) + `Potion_Text` (`TextBlock`) dengan `bIsVariable = true`.
         - Binding runtime: Menggunakan getter variabel bawaan UMG `Variables|WBP_PlayerHUD|GetHP_Text` dan `Variables|WBP_PlayerHUD|GetPotion_Text` untuk mengakses instance runtime TextBlock.
         - `EventTick` di-wire via DSL: membaca pawn pemilik via `GetOwningPlayerPawn` -> cast ke `BP_Player` -> update `Potion: <PotionCount> / <MaxPotionCount>` via `Widget|SetText(Text)` -> validasi komponen `Health` -> update `HP: <CurrentHealth> / <MaxHealth>` via `Widget|SetText(Text)`.
         - `BP_PlayerController:EventGraph` diperbarui: setelah `IsLocalPlayerController`, memanggil `CreateWidget(WBP_PlayerHUD, OwningPlayer=self)` lalu `Add to Player Screen` (tampil mandiri di masing-masing layar split-screen).
       - Hasil tes PIE user: **SUKSES**. Nilai HP berkurang saat terkena racun dan nilai Potion berkurang saat melempar secara real-time di layar masing-masing pemain.
       - Compile `WBP_PlayerHUD` OK, Compile `BP_PlayerController` OK, Save dirty assets OK.
---

## 2. Next Action
- **Tindakan**: Masuk ke task **W1-B4 (`DA_Tuning` Data Asset untuk balancing semua nilai §4)**.
  1. Buat Data Asset / Primary Data Asset class `DA_Tuning` di `/Game/Splashfall/Data/` (atau Blueprints).
  2. Isi semua nilai parameter balancing sesuai `AGENTS.md` §4: HP pemain (100), Potion awal/maks (3/5), Jarak lempar maks (2500 uu), Throw cooldown (0.75 s), AOE radius (275 uu), AOE durasi (5 s), DPS (5 HP/s, tick 0.5 s = 2.5 HP/tick), Pickup spawn interval (12 s), Durasi ronde (120 s), Jeda antar ronde (3 s).
  3. Hubungkan `BP_Player` dan `BP_PoisonAOE` untuk membaca nilai tuning dari `DA_Tuning`.
  4. Compile, save, dan verifikasi di PIE.
- **Pelaksana**: Agent via Unreal MCP + verifikasi user.
- **Prasyarat**: W1-B3 terverifikasi di PIE ✅

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
- **Ukuran Arena**: Baseline 60ÁE0 m; evaluasi alternatif 40ÁE0 m pada playtest 1.

---

## 4. Fakta Project

| Fakta | Status | Keterangan |
|---|---|---|
| `BP_GameInstance` selesai dibuat | Terverifikasi di disk via MCP | Berada di `/Game/Splashfall/Blueprints/BP_GameInstance`, parent class `GameInstance`. EventGraph **kosong**. Fungsi publik `EnsureSecondLocalPlayer` tersedia (guard `GetNumLocalPlayerControllers < 2` ↁE`CreateLocalPlayer(-1, true)`). |
| GameMode aktif terkonfirmasi | Terverifikasi via MCP & WorldSettings | `/Game/GameMode/BP_ThirdPersonGameMode` aktif pada `defaultGameMode` WorldSettings level `Lvl_ThirdPerson`. |
| PlayerController aktif terkonfirmasi | Terverifikasi via MCP CDO | `/Game/Characters/BP_PlayerController` digunakan oleh GameMode aktif. |
| Pengaturan Splitscreen & Gamepad di Config | Terverifikasi di `DefaultEngine.ini` | `bUseSplitscreen=true`, `twoPlayerSplitscreenLayout=Horizontal`, `bOffsetPlayerGamepadIds=false` (artinya Skip Assigning Gamepad to P1 = False). |
| `BP_Player` memiliki komponen `Health` terpasang | Terverifikasi via MCP | Komponen `Health` terpasang dan `Variables|Default|GetHealth` tersedia di Blueprint. |
| Komponen `Health` mandiri tanpa HUDClass | Terverifikasi via MCP | Pemanggilan `GetActorOfClass(HUDClass)` dan variabel `HUD` dihapus dari `EventBeginPlay`. |
| Fungsi `GainHealth` pada `Health` normal | Terverifikasi via MCP | Output `Clamp(Float)` terhubung ke input `SetCurrentHealth`, clamping nilai dari 0.0 sampai `MaxHealth`. |
| Nilai default `CurrentHealth` dan `MaxHealth` bernilai 100 | Terverifikasi via MCP CDO | Terbaca `100` pada CDO `/Game/Components/Health.Default__Health_C`. |
| Variabel status gameplay pada `BP_Player` | Terverifikasi via MCP CDO | `PlayerIndex` (0), `PotionCount` (3), `MaxPotionCount` (5). |
| Kamera Over-The-Shoulder (OTS) `BP_Player` | Terverifikasi via MCP | Dikonfigurasi di `UserConstructionScript`: `TargetArmLength = 240.0`, `SocketOffset = (0, 60, 50)`. |
| `IA_Throw` & Mapping `IMC_Default` | Terverifikasi via MCP | `IA_Throw` (Boolean) terdaftar di `IMC_Default` untuk `Gamepad_RightTrigger` (R2) dan `LeftMouseButton`. |
| `BP_PoisonProjectile` siap | Terverifikasi via MCP | `/Game/Splashfall/Blueprints/BP_PoisonProjectile` (SphereComponent radius 15 BlockAllDynamic & NotifyRigidBodyCollision, Mesh scale 0.25 NoCollision, ProjectileMovement speed 1400 gravity 1.0, lifeSpan 5s, EventHit -> spawn AOE di HitLocation + DestroyActor). |
| `BP_PoisonAOE` siap | Terverifikasi via MCP | `/Game/Splashfall/Blueprints/BP_PoisonAOE` (SphereComponent radius 275 OverlapAllDynamic & GenerateOverlapEvents, Mesh cylinder scale 5.5x5.5x0.05 NoCollision, CDO lifeSpan 5s). |
| Mekanik Throw & Cooldown pada `BP_Player` | Terverifikasi via MCP | Event `IA_Throw` -> Guard `(PotionCount > 0) AND bCanThrow` -> `SetCanThrow(false)` -> `ThrowPotion` (spawn projectile di aim rotasi + `PotionCount - 1`) -> `Delay(ThrowCooldown = 0.75)` -> `SetCanThrow(true)`. |
| Damage Overlap Counter & Timer pada `BP_Player` | Terverifikasi via MCP | Variabel `PoisonOverlapCounter` (0), `PoisonDamagePerTick` (2.5), `PoisonTickInterval` (0.5). Event `BeginOverlap` -> guard enemy AOE -> counter+1 -> jika counter==1 jalankan timer looping `ApplyPoisonTick`. Event `EndOverlap` -> counter-1 (clamped min 0) -> jika counter<=0 clear timer. Fungsi `ApplyPoisonTick` memotong 2.5 HP ke Health component. |
| Enhanced Input yang sudah tersedia | Hasil introspeksi agent | `IA_Move`, `IA_Look`, `IA_MouseLook`, `IA_Jump`, `IA_Throw` di `/Game/Input/Actions/` serta `IMC_Default`. |
| Input Actions yang belum tersedia | Hasil introspeksi agent | Input Action untuk Pause Menu (Options). |
| Animasi Mannequin lengkap | Hasil introspeksi agent | Tersedia di `/Game/Characters/Mannequins/Anims/` termasuk Jump, Jog, Walk, Death, dan HitReact. |
| `WBP_PlayerHUD` siap via Add to Player Screen | Terverifikasi via MCP | Berada di `/Game/Splashfall/UI/WBP_PlayerHUD`. TextBlock `HP_Text` & `Potion_Text` di-update setiap tick via polling ke pawn pemain (`Health` component & `PotionCount`). Diinstansiasi dan ditambahkan ke player screen oleh `BP_PlayerController:EventBeginPlay`. |
| Engine Association | Hasil introspeksi agent | Menggunakan build UE 5.8 (Identifier: `{7A12E277-42D4-CC46-1CCF-31986923C4AF}`). |

---

## 5. Isu dan Blocker

1. **Aset Termodifikasi di Git Status (SELESAI)**:
   Perubahan `Health.uasset`, `BP_PlayerController.uasset`, `BP_ThirdPersonGameMode.uasset`, dan `BP_Player.uasset` telah di-commit ke Git secara bersih.
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

10. **Format Asset Reference pada `write_graph_dsl` dan `set_pin_value`**:
    Menulis asset reference inline pada DSL tidak boleh menggunakan format object `{refPath: ...}` ataupun `(load ...)`. Cukup gunakan string path berpetik (contoh: `"/Game/Input/IMC_Default.IMC_Default"`). Alternatifnya, pin default value bertipe object reference dapat di-set langsung menggunakan tool `set_pin_value` dengan string path aset.

---

## 7. Log Sesi

| Tanggal | Siapa | Dikerjakan | Hasil |
|---|---|---|---|
| 2026-11-20 | Antigravity | Setup dokumen rencana `AGENTS.md`, introspeksi aset via Unreal MCP, analisis detail `Health` & `BP_Player`, penyusunan `PROGRESS.md`. | Seluruh aset terinventarisasi, bug `GainHealth` & dependensi `HUDClass` terpetakan, dokumen acuan kerja siap tanpa modifikasi biner. |
| 2026-11-20 | Antigravity | Pengecekan GameMode & PlayerController aktif, setup `.gitignore` (`.gemini/`, `.agents/`, `tools/`), pemindahan helper MCP ke `tools/unreal_mcp_client.py`, pembuatan `BP_GameInstance` di `/Game/Splashfall/Blueprints/` dengan logic guard `EventInit` (`GetNumLocalPlayerControllers < 2` -> `CreateLocalPlayer(-1)`), verifikasi compile & save via Unreal MCP. | Fondasi spike W1-B1 siap, `BP_GameInstance` ter-compile & tersimpan di disk tanpa error. Pengaturan split-screen horizontal terkonfirmasi aktif di config. |
| 2026-11-20 | Antigravity | Refactor lifecycle local player: buat fungsi `EnsureSecondLocalPlayer` di `BP_GameInstance`, panggil pada `Event BeginPlay` di `BP_ThirdPersonGameMode`. Pembersihan node lama `EventInit` di `BP_GameInstance:EventGraph` menggunakan `find_nodes` & `delete_node` via MCP setelah editor restart. | Kedua Blueprint sukses di-compile dan di-save tanpa error. Error "No game mode set" teratasi secara arsitektural. |
| 2026-11-20 | Antigravity | Investigasi split-screen gagal (layar masih 1 player): (1) Output Log PIE tidak ada error/warning LocalPlayer ↁEkonfirmasi `EnsureSecondLocalPlayer` tidak pernah terpanggil. (2) `gameInstanceClass` terkonfirmasi ke-reset ke default `/Script/Engine.GameInstance` setelah editor crash. (3) Fix: tambah `GameInstanceClass` & koreksi `GlobalDefaultGameMode` di `Config/DefaultEngine.ini` supaya persist setelah restart. (4) Tulis ulang `EventBeginPlay` di `BP_ThirdPersonGameMode` tanpa cast—inline `GetNumLocalPlayerControllers < 2 ↁECreateLocalPlayer(-1, true)`—compile ✁Esave ✁E | Root cause teridentifikasi dan diperbaiki. `DefaultEngine.ini` kini menyimpan `GameInstanceClass` dan `GlobalDefaultGameMode` yang benar. Perlu restart editor + tes PIE ulang oleh user. |
| 2026-11-20 | Antigravity | Inspeksi & perbaikan kontrol input karakter (Spike W1-B1): (1) Membaca dan membandingkan `BP_PlayerController:EventGraph` dengan `BP_ThirdPersonPlayerController` (ditemukan `BP_PlayerController` kosong tanpa mapping context). (2) Menambahkan `DelayUntilNextTick` -> `IsLocalPlayerController` -> `AddMappingContext(IMC_Default)` & `AddMappingContext(IMC_MouseLook)` pada `BP_PlayerController:EventGraph` via MCP, compile ✁E save ✁E (3) Menginspeksi pin connectivity `BP_Player:EventGraph`: terverifikasi node Enhanced Input `IA_Move`, `IA_Look`, `IA_MouseLook`, `IA_Jump` sudah terhubung ke fungsi `Move`, `Aim`, `Jump`, `StopJumping`. | Input Controller siap untuk kedua player (gamepad + fallback keyboard/mouse). Menunggu verifikasi gerak di PIE. |
| 2026-11-20 | Antigravity | Bug fix `BP_PlayerController:EventGraph`: perbaiki pemanggilan subsystem Enhanced Input dari `PlayerController|...` dengan argumen `0` menjadi `(LocalPlayerSubsystems|GetEnhancedInputLocalPlayerSubsystem)` tanpa argumen tambahan untuk menghilangkan runtime error `Accessed None`. | Blueprint `BP_PlayerController` berhasil di-compile tanpa error (`returnValue: null`) dan disimpan ke disk. Siap dites ulang di PIE. |
| 2026-11-20 | Antigravity | Eksekusi task W1-A1: (1) `/Game/Components/Health`: perbaiki bug `GainHealth` (sambungkan output clamp ke `SetCurrentHealth`), hapus dependensi `HUDClass` di `EventBeginPlay`, set CDO `CurrentHealth=100.0`, `MaxHealth=100.0`, compile ✁E save ✁E (2) `/Game/Characters/BP_Player`: pasang komponen `Health` via `ActorTools.add_component`, tambah variabel `PlayerIndex` (0), `PotionCount` (3), `MaxPotionCount` (5) via CDO, setup kamera OTS di `UserConstructionScript` (`TargetArmLength=240.0`, `SocketOffset=(0, 60, 50)`), compile ✁E save ✁E | Task W1-A1 selesai penuh. Pondasi karakter, HP, resource potion, dan kamera tempur siap untuk mekanik throw (W1-A2). |
| 2026-11-20 | Antigravity | Eksekusi task W1-A2: (1) Buat `IA_Throw` (Boolean) & daftarkan ke `IMC_Default` (`Gamepad_RightTrigger` & `LeftMouseButton`), save ✁E (2) Buat `BP_PoisonProjectile` (SphereComponent radius 15 BlockAllDynamic, Mesh scale 0.25 NoCollision, ProjectileMovement speed 1400 gravity 1.0, lifeSpan 5.0), compile ✁E save ✁E (3) Buat fungsi `ThrowPotion` di `BP_Player` (spawn projectile di arah aim + kurangi potion), tambah variabel `ThrowCooldown` (0.75) & `bCanThrow` (true), pasang event `IA_Throw` di `BP_Player:EventGraph` dengan guard `PotionCount > 0 && bCanThrow` -> `SetCanThrow(false)` -> `ThrowPotion` -> `Delay(ThrowCooldown)` -> `SetCanThrow(true)`, compile ✁E save ✁E | Task W1-A2 selesai penuh. Mekanik lempar proyektil dengan arc, resource decrement, dan cooldown siap diuji di PIE. |
| 2026-11-20 | Antigravity | Eksekusi task W1-A3: (1) Buat `BP_PoisonAOE` di `/Game/Splashfall/Blueprints/` (SphereComponent radius 275 OverlapAllDynamic & `bGenerateOverlapEvents=true`, StaticMeshComponent Cylinder scale 5.5x5.5x0.05 NoCollision, CDO `initialLifeSpan=5.0`), compile ✁E save ✁E (2) Update `BP_PoisonProjectile`: tambahkan handler `Collision|EventHit` di `EventGraph` (guard `Other != GetOwner()`) -> spawn `BP_PoisonAOE` di `HitLocation` (`Owner = GetOwner()`) -> `DestroyActor(self)`; pasang `Physics|SetNotifyRigidBodyCollision` di `UserConstructionScript`, compile ✁E save ✁E | Task W1-A3 selesai penuh. Proyektil sukses mendeteksi impact permukaan, memunculkan genangan AOE berdurasi 5s, dan hancur tanpa direct damage. |
| 2026-11-20 | Antigravity | Eksekusi task W1-A4: (1) Tambahkan variabel `PoisonOverlapCounter` (int, default 0), `PoisonDamagePerTick` (float, default 2.5), dan `PoisonTickInterval` (float, default 0.5) pada `BP_Player` via CDO. (2) Instansiasi komponen `Health` di `BP_Player:UserConstructionScript` via `AddComponent|Custom|AddHealth` -> `SetHealth`. (3) Buat fungsi `ApplyPoisonTick` via DSL dengan guard `PoisonOverlapCounter > 0` & `IsValid Health` -> `Health->TakeDamage(2.5)` -> `PrintString` info nama aktor & HP. (4) Wiring incremental `EventActorBeginOverlap` & `EventActorEndOverlap` di `BP_Player:EventGraph` (anti-friendly-fire, anti-stacking timer). (5) Perbaiki attachment `BP_PoisonAOE` collision sphere ke root dan `SetGenerateOverlapEvents(true)`. Compile ✁E save ✁E | Task W1-A4 selesai penuh dan terverifikasi di PIE (karakter terkena racun HP berkurang dari 100 secara bertahap 2.5/0.5s, racun sendiri tidak melukai, keluar dari racun menghentikan tick). |

| 2026-11-20 | Antigravity | Eksekusi task W1-B2: (1) Tambah PlayerStart_P2 di level (1000,0,102 Yaw180) dan reposisi PlayerStart_P1 ke (-1000,0,102 Yaw0). Save level OK. (2) Tambah chain incremental EventPossessed -> CastToPlayerController -> GetLocalPlayerControllerID -> SetPlayerIndex pada BP_Player:EventGraph. Guard CreateLocalPlayer < 2 di BP_ThirdPersonGameMode:EventBeginPlay tetap utuh. Compile + save OK. | Task W1-B2 selesai. 2 PlayerStart berjauhan & saling berhadapan; PlayerIndex di-set otomatis (0=P1, 1=P2) saat possess. |
| 2026-11-20 | Antigravity | Eksekusi task W1-B3: Buat `WBP_PlayerHUD` (CanvasPanel, VBox, HP_Text, Potion_Text), binding variabel ke CDO, tulis EventTick DSL (polling HP dari Health component dan PotionCount dari BP_Player), perbarui `BP_PlayerController:EventGraph` untuk CreateWidget + Add to Player Screen. Compile & save OK. | Task W1-B3 selesai. HUD per pemain aktif dan siap diuji di PIE. |
| 2026-11-20 | Antigravity | Perbaikan binding runtime `WBP_PlayerHUD`: gunakan UMG generated getter `Variables|WBP_PlayerHUD|GetHP_Text` dan `GetPotion_Text` di `EventTick` menggantikan template CDO archetype. Hasil tes PIE user: SUKSES (HP dan Potion update real-time di layar split-screen). | Task W1-B3 terverifikasi penuh di PIE. |
