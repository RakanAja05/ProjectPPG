# PROGRESS.md  ESplashfall: Poison Arena

Dokumen pelacak kondisi terkini proyek. Berfungsi sebagai konteks langsung antar sesi kerja.
*Sumber rencana & aturan tetap mengacu pada [AGENTS.md](file:///C:/Kuliah/Semester_5/Praktikum%20Game/UAS/ProjectUAS/AGENTS.md).*

---

## 1. Posisi Sekarang
- **Tanggal Update**: 27 November 2026
- **Tahap**: Langkah 3 — *HUD Pemain, Sustain Amunisi, Contest & Data Balancing* (Langkah 1 & Langkah 2 selesai penuh)
- **Ringkasan**:
  1. Dokumen perencanaan berbasis langkah (*step-by-step*, Langkah 1–8) telah dikunci di `AGENTS.md` dengan scope couch 1v1 local split-screen (2 gamepad).
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
    14. **Mekanik Potion Pickup (Amunisi Respawn) SELESAI & SIAP DITES PIE**:
       - Blueprint `/Game/Splashfall/Blueprints/BP_PotionPickup` dibuat (Parent: `Actor`):
         - Komponen: `PickupCollision` (`SphereComponent`, radius `60.0`, collision profile `OverlapAllDynamic`, `bGenerateOverlapEvents = true`) dan `PickupMesh` (`StaticMeshComponent`, `/Engine/BasicShapes/Cylinder.Cylinder`, scale `(0.4, 0.4, 0.4)`, `NoCollision`, relative location Z=20.0, ter-attach ke `PickupCollision`).
         - Variabel: `RespawnTime` (Float, default `8.0`) dan `bIsAvailable` (Boolean, default `true`).
         - Logika `EventActorBeginOverlap`: Cek `bIsAvailable == true` -> Cast `OtherActor` ke `BP_Player` -> Cek `PotionCount < MaxPotionCount` (cap 5) -> Jika True: `SetPotionCount = PotionCount + 1` pada `BP_Player` -> `SetbIsAvailable(false)` -> `SetActorHiddenInGame(true)` & `SetActorEnableCollision(false)` -> `Delay(RespawnTime = 8.0s)` -> `SetActorHiddenInGame(false)` & `SetActorEnableCollision(true)` -> `SetbIsAvailable(true)`.
         - Compile `BP_PotionPickup` OK, Save `BP_PotionPickup` OK.
       - **FIX (2026-11-21)**: `ConstructionScript` diperbarui — node `SetStaticMesh(PickupMesh, Cylinder)` ditambah di awal chain exec sebelum `AttachComponentToComponent`. Mesh cylinder sekarang ter-assign saat spawn.
       - 3 actor baru di-spawn di posisi terbuka (bebas dari SM_Cube13 cover geometry):
         - `BP_PotionPickup_Center` di `(-600, 0, 100)` — antara P1 spawn dan center arena
         - `BP_PotionPickup_Left` di `(500, -800, 100)` — sisi kiri arena
         - `BP_PotionPickup_Right` di `(500, 800, 100)` — sisi kanan arena
        - Level disimpan dan terverifikasi di PIE (pickup silinder muncul, bertambah amunisi, dan respawn).
     15. **Kondisi Eliminasi / Kematian saat HP = 0 (Task M8 Fondasi) SELESAI & TERVERIFIKASI PIE**:
        - Variabel `bIsDead` (Boolean) ditambahkan ke `BP_Player`.
        - Fungsi `HandleDeath` dibuat di `BP_Player`: guard `not bIsDead` -> set `bIsDead=true`, matikan lempar, matikan timer racun, nonaktifkan movement & input, aktifkan Ragdoll Physics pada Mesh, dan nonaktifkan collision kapsul.
        - Hasil tes PIE user: **SUKSES**. Karakter tumbang dengan ragdoll realistis, input terkunci, dan eliminasi terdeteksi akurat.
        - Git Commit Checkpoint: `feat(gameplay): complete potion pickup respawn and player death ragdoll condition` (commit `baeedeb`).
     16. **Result Screen & Auto-Rematch Loop (Task M8 & M11) SELESAI & TERVERIFIKASI PIE**:
        - `WBP_ResultScreen` dibuat di `/Game/Splashfall/UI/WBP_ResultScreen` (Parent: `UserWidget`):
          - Hierarchy: `CanvasPanel_Root` -> `ResultText` (`TextBlock`, font 52 Bold, warna emas outline hitam, centered di Anchor Y=0.4) + `SubText` (`TextBlock`, font 24, "Rematch in 4 seconds...", centered di Anchor Y=0.5).
          - Fungsi `SetWinnerText(WinnerString: String)`: Mengubah teks `ResultText` secara dinamis via runtime getter.
        - `BP_Player` diperbarui:
          - Fungsi `RestartRound`: Memanggil `Game|OpenLevel(byName) "Lvl_ThirdPerson"` untuk restart match bersih.
          - Fungsi `HandleDeath`:
            - Membuat widget `WBP_ResultScreen`.
            - Menetapkan teks pemenang via `SetWinnerText` (`"PLAYER 2 WINS!"` jika P1 gugur, `"PLAYER 1 WINS!"` jika P2 gugur).
            - Menampilkan ke viewport penuh via `Add to Viewport (ZOrder 10)` (menutupi kedua viewport split-screen).
            - Menjalankan timer 4.0 detik `SetTimerbyFunctionName(self, "RestartRound", 4.0, false)`.
        - Integritas `BP_Player:EventGraph` terjaga 100% tanpa modifikasi/overwriting.
        - Hasil tes PIE user: **SUKSES**. Layar split-screen tertutup Result Screen emas di tengah layar saat salah satu pemain gugur, dan setelah 4 detik match me-reload ronde baru secara bersih.
        - Git Commit Checkpoint: `feat(ui): complete WBP_ResultScreen and auto-rematch round reset` (commit `79a4bcb`).
     17. **Charged Throw (Tahan Tombol) & Trajectory Preview SELESAI & TERVERIFIKASI PIE**:
        - `BP_PoisonProjectile` diperbarui:
          - `MaxSpeed` pada `ProjectileMovementComponent` dinaikkan menjadi `2500.0` (dari `1400.0`).
          - Fungsi publik `InitThrowVelocity(NewVelocity: Vector)` dibuat via DSL: meng-set `Velocity` pada `ProjectileMovementComponent` ke `NewVelocity`.
          - Compile OK, Save OK.
        - `BP_Player` diperbarui:
          - Variabel baru di CDO: `bIsChargingThrow` (bool, default false), `CurrentChargeTime` (float, default 0.0), `MaxChargeTime` (float, default 1.5), `MinThrowSpeed` (float, default 600.0), `MaxThrowSpeed` (float, default 2200.0).
          - Fungsi pure `GetCurrentThrowSpeed` dibuat via DSL: menghitung `Lerp(MinThrowSpeed, MaxThrowSpeed, Clamp(CurrentChargeTime / MaxChargeTime, 0.0, 1.0))`.
          - Fungsi `UpdateThrowPreview` dibuat via DSL: jika `bIsChargingThrow == true`, menambah `CurrentChargeTime` sebesar `GetWorldDeltaSeconds()` (clamped `0.0` .. `MaxChargeTime`), menghitung `StartPos` spawn proyektil dan `LaunchVelocity = ForwardVector * GetCurrentThrowSpeed()`, lalu memanggil `PredictProjectilePathByTraceChannel` (`bTracePath=true`, `ProjectileRadius=15.0`, `ECC_WorldDynamic`, `ForOneFrame`, `SimFrequency=15.0`, `MaxSimTime=2.5`).
          - Fungsi `ThrowPotion` diperbarui: menghitung `LaunchVelocity = ForwardVector * GetCurrentThrowSpeed()`, memanggil `SpawnActorBPPoisonProjectile`, lalu memanggil `InitThrowVelocity(LaunchVelocity)` pada proyektil yang baru di-spawn.
          - Wiring `IA_Throw` di `BP_Player:EventGraph` diperbarui secara **incremental** (tanpa menimpa Enhanced Input / Overlap):
            - `IA_Throw:Started` -> Guard `(PotionCount > 0) AND bCanThrow AND (not bIsDead)` -> `Set bIsChargingThrow = true` -> `Set CurrentChargeTime = 0.0` -> `SetTimerbyFunctionName("UpdateThrowPreview", 0.02, true)`.
            - `IA_Throw:Completed` -> Guard `bIsChargingThrow == true` -> `Set bIsChargingThrow = false` -> `ClearTimerbyFunctionName("UpdateThrowPreview")` -> `Set bCanThrow = false` -> `ThrowPotion` -> `Delay(ThrowCooldown = 0.75)` -> `Set bCanThrow = true`.
          - Integritas `BP_Player:EventGraph` terjaga 100% tanpa modifikasi/overwriting graph utama.
          - Hasil tes PIE user: **SUKSES**. Quick tap melempar pendek (~600.0), hold menampilkan garis preview lintasan dinamis real-time (hingga ~2200.0), botol meluncur mengikuti prediksi. Guard amunisi & eliminasi berfungsi normal.
          - Compile OK, Save OK.
     18. **Task W2-A2 SELESAI & TERVERIFIKASI PIE (Visual Warna Owner AOE Hijau P1 / Ungu P2, Warna Pickup Cyan, & Pembersihan Debug Print)**:
        - Material Instances dibuat di `/Game/Splashfall/Materials/` (Parent: `/Game/LevelPrototyping/Materials/M_FlatCol`):
          - `MI_Poison_P1`: Vector Parameter `Base Color` = Hijau racun terang `(R=0.05, G=1.0, B=0.1, A=1.0)` untuk P1 (`PlayerIndex == 0`).
          - `MI_Poison_P2`: Vector Parameter `Base Color` = Ungu neon terang `(R=0.8, G=0.05, B=1.0, A=1.0)` untuk P2 (`PlayerIndex == 1`).
          - `MI_PotionPickup`: Vector Parameter `Base Color` = Biru cyan terang `(R=0.0, G=0.8, B=1.0, A=1.0)` untuk amunisi potion pickup.
        - `/Game/Splashfall/Blueprints/BP_PoisonAOE`:
          - `EventBeginPlay` diperbarui via DSL: membaca `GetOwner` -> Cast ke `BP_Player` -> baca `PlayerIndex`. Jika `PlayerIndex == 0`, memasang `MI_Poison_P1` ke `AOEMesh`; jika `PlayerIndex != 0`, memasang `MI_Poison_P2` ke `AOEMesh`.
        - `/Game/Splashfall/Blueprints/BP_PotionPickup`:
          - `UserConstructionScript` diperbarui via DSL: memanggil `SetMaterial(PickupMesh, 0, MI_PotionPickup)` sehingga mesh pickup silinder berwarna biru cyan terang.
        - `/Game/Characters/BP_Player`:
          - Pembersihan debug: Menghapus pemanggilan `PrintString` pada fungsi grafik `BP_Player:ApplyPoisonTick` sehingga layar split-screen bersih dari spam teks HP saat terkena racun (`BP_Player:EventGraph` tetap utuh 100%).
        - Hasil tes PIE user: **SUKSES**. Warna AOE hijau untuk P1, ungu untuk P2, pickup biru cyan mencolok, dan layar bersih tanpa spam print debug.
        - Seluruh Blueprint ter-compile tanpa error (`BP_PoisonAOE`, `BP_PotionPickup`, `BP_Player`) dan tersimpan bersih ke disk.
     19. **Kustomisasi Result Screen: VICTORY untuk Pemenang & DEFEAT untuk yang Kalah (Task M8 & M11 Polish) SELESAI & TERVERIFIKASI PIE**:
        - `/Game/Splashfall/UI/WBP_ResultScreen`:
          - Fungsi `SetWinnerText(WinnerString: String)` diperbarui via DSL:
            - Jika `WinnerString == "VICTORY"`: meng-set teks menjadi `"VICTORY"` dan warna teks menjadi emas terang `(R=1.0, G=0.84, B=0.0)`.
            - Jika `WinnerString == "DEFEAT"` (atau lainnya): meng-set teks menjadi `"DEFEAT"` dan warna teks menjadi merah tua `(R=0.9, G=0.15, B=0.15)`.
        - `/Game/Characters/BP_Player`:
          - Fungsi `HandleDeath` diperbarui via DSL:
            - Membuat 2 widget `WBP_ResultScreen` mandiri untuk masing-masing PlayerController: `PC0` (Player 1) dan `PC1` (Player 2).
            - Cek `PlayerIndex` dari karakter yang gugur (`_self`):
              - Jika `PlayerIndex == 0` (P1 kalah): widget P0 diberi `"DEFEAT"` dan widget P1 diberi `"VICTORY"`.
              - Jika `PlayerIndex == 1` (P2 kalah): widget P0 diberi `"VICTORY"` dan widget P1 diberi `"DEFEAT"`.
            - Menampilkan masing-masing widget ke layar pemainnya via `AddToPlayerScreen(widget, 10)` (bukan global viewport), sehingga tepat di tengah viewport masing-masing split-screen.
          - Integritas `BP_Player:EventGraph` tetap terjaga 100%.
        - Hasil tes PIE user: **SUKSES**. Status kemenangan akurat di masing-masing layar split-screen (pemenang melihat VICTORY emas, pecundang melihat DEFEAT merah).
        - Compile OK, Save OK.
---

## 2. Next Action
- **Tindakan**: Konfirmasi prioritas pengerjaan task berikutnya dengan tim/user:
  - **Opsi 1 (Langkah 3 - L3-A1)**: Implementasi *AOE Contest* (genangan AOE baru menghapus penuh AOE lawan yang saling overlap).
  - **Opsi 2 (Langkah 3 - L3-A2)**: Spawner acak `BP_PotionPickup` di 5 titik (interval 12 s jika pickup aktif < 2).
  - **Opsi 3 (Langkah 3 - L3-B2)**: Pembuatan Data Asset `DA_Tuning` untuk mengunci seluruh konstanta gameplay §4.
  - **Opsi 4 (Langkah 5 - L5-B1)**: Implementasi `BP_MatchManager` (Best-of-3, ronde timer 120s, jeda 3s, manual round reset).
- **Pelaksana**: Agent & User.
- **Prasyarat**: Checkpoint Git W2-A2 & Polish Layar Hasil ter-commit ✅

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
| 2026-11-27 | User | Rencana pengerjaan di `AGENTS.md` diubah dari jadwal berbasis hari/minggu kalender menjadi alur berbasis langkah berurutan (Langkah 1–8). |

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
| `BP_PoisonProjectile` siap | Terverifikasi via MCP | `/Game/Splashfall/Blueprints/BP_PoisonProjectile` (SphereComponent radius 15 BlockAllDynamic & NotifyRigidBodyCollision, Mesh scale 0.25 NoCollision, ProjectileMovement MaxSpeed 2500.0, lifeSpan 5s, EventHit -> spawn AOE di HitLocation + DestroyActor, fungsi publik `InitThrowVelocity(NewVelocity)` untuk override Velocity). |
| `BP_PoisonAOE` siap | Terverifikasi via MCP | `/Game/Splashfall/Blueprints/BP_PoisonAOE` (SphereComponent radius 275 OverlapAllDynamic & GenerateOverlapEvents, Mesh cylinder scale 5.5x5.5x0.05 NoCollision, CDO lifeSpan 5s). |
| Mekanik Charged Throw & Trajectory Preview pada `BP_Player` | Terverifikasi via MCP | Event `IA_Throw:Started` -> Guard `(PotionCount > 0) AND bCanThrow AND (not bIsDead)` -> `SetIsChargingThrow(true)` -> `SetCurrentChargeTime(0.0)` -> timer `UpdateThrowPreview` (0.02s looping via `PredictProjectilePathByTraceChannel` dengan debug `ForOneFrame`). Event `IA_Throw:Completed` -> Guard `bIsChargingThrow` -> `SetIsChargingThrow(false)` -> `ClearTimer UpdateThrowPreview` -> `SetCanThrow(false)` -> `ThrowPotion` (kecepatan dinamis Lerp 600..2200 via `GetCurrentThrowSpeed` & kirim ke proyektil via `InitThrowVelocity`) -> `Delay(ThrowCooldown = 0.75)` -> `SetCanThrow(true)`. |
| Damage Overlap Counter & Timer pada `BP_Player` | Terverifikasi via MCP | Variabel `PoisonOverlapCounter` (0), `PoisonDamagePerTick` (2.5), `PoisonTickInterval` (0.5). Event `BeginOverlap` -> guard enemy AOE -> counter+1 -> jika counter==1 jalankan timer looping `ApplyPoisonTick`. Event `EndOverlap` -> counter-1 (clamped min 0) -> jika counter<=0 clear timer. Fungsi `ApplyPoisonTick` memotong 2.5 HP ke Health component. |
| Enhanced Input yang sudah tersedia | Hasil introspeksi agent | `IA_Move`, `IA_Look`, `IA_MouseLook`, `IA_Jump`, `IA_Throw` di `/Game/Input/Actions/` serta `IMC_Default`. |
| Input Actions yang belum tersedia | Hasil introspeksi agent | Input Action untuk Pause Menu (Options). |
| Animasi Mannequin lengkap | Hasil introspeksi agent | Tersedia di `/Game/Characters/Mannequins/Anims/` termasuk Jump, Jog, Walk, Death, dan HitReact. |
| `WBP_PlayerHUD` siap via Add to Player Screen | Terverifikasi via MCP | Berada di `/Game/Splashfall/UI/WBP_PlayerHUD`. TextBlock `HP_Text` & `Potion_Text` di-update setiap tick via polling ke pawn pemain (`Health` component & `PotionCount`). Diinstansiasi dan ditambahkan ke player screen oleh `BP_PlayerController:EventBeginPlay`. |
| `BP_PotionPickup` siap & terpasang di level | Terverifikasi via MCP | Berada di `/Game/Splashfall/Blueprints/BP_PotionPickup`. SphereComponent (radius 60.0), Mesh Cylinder (scale 0.4), variabel RespawnTime (8.0s), bIsAvailable (true). Overlap -> validasi player & PotionCount < MaxPotionCount -> +1 potion, hide, disable collision, delay respawn. 3 instance terpasang di Lvl_ThirdPerson (Center, Left, Right). |
| Material Instances warna racun & pickup | Terverifikasi via MCP | Berada di `/Game/Splashfall/Materials/`: `MI_Poison_P1` (hijau terang R=0.05 G=1.0 B=0.1), `MI_Poison_P2` (ungu neon R=0.8 G=0.05 B=1.0), `MI_PotionPickup` (biru cyan R=0.0 G=0.8 B=1.0). Meng-override parameter Vector `Base Color` dari parent `M_FlatCol`. |
| Pewarnaan dinamis AOE berdasarkan PlayerIndex | Terverifikasi via MCP | `BP_PoisonAOE:EventBeginPlay` mengecek `GetOwner` -> `CastToBP_Player` -> `GetPlayerIndex`. Jika 0 pasang `MI_Poison_P1` (hijau), jika != 0 pasang `MI_Poison_P2` (ungu) ke `AOEMesh`. |
| Debug PrintString ApplyPoisonTick dibersihkan | Terverifikasi via MCP | Fungsi `BP_Player:ApplyPoisonTick` bersih tanpa node `PrintString`. Layar split-screen tidak lagi terdistraksi spam teks HP saat duel. |
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
| 2026-11-21 | Antigravity | Eksekusi Result Screen & Auto-Rematch (Task M8 & M11): (1) Buat `WBP_ResultScreen` di `/Game/Splashfall/UI/` dengan `CanvasPanel_Root` -> `ResultText` (font 52 Bold emas) + `SubText` (font 24 putih) di tengah layar. (2) Buat fungsi `SetWinnerText(WinnerString: String)` pada `WBP_ResultScreen`. (3) Buat fungsi `RestartRound` di `BP_Player` (`OpenLevel Lvl_ThirdPerson`). (4) Di `BP_Player:HandleDeath`: spawn `WBP_ResultScreen`, kirim nama pemenang via `SetWinnerText`, `Add to Viewport (ZOrder 10)`, dan pasang timer 4.0s panggil `RestartRound`. EventGraph BP_Player utuh. Compile & save OK. |
| 2026-11-21 | Antigravity | Implementasi Kondisi HP = 0 / Death (Task M8): (1) Tambah variabel `bIsDead` pada `BP_Player`. (2) Buat fungsi `HandleDeath`: guard `not bIsDead` -> `bIsDead=true`, `bCanThrow=false`, `ClearTimer ApplyPoisonTick`, `DisableMovement`, `DisableInput`, aktifkan Ragdoll (`SetCollisionProfileName("Ragdoll")` + `SetSimulatePhysics(true)` pada Mesh & `NoCollision` pada Capsule), tampilkan notifikasi pemenang ronde. (3) Hubungkan `ApplyPoisonTick`: jika HP <= 0.0 -> panggil `HandleDeath`. Compile OK, save OK. |
| 2026-11-21 | Antigravity | Fix BP_PotionPickup: (1) Root cause bug silinder tidak terlihat: mesh Cylinder tidak di-assign di CS. Tambah node `Components|StaticMesh|SetStaticMesh` via create_node + connect_pins ke CS exec chain (Entry->SetMesh->Attach->…). (2) Root cause bug posisi tersembunyi: Center (0,0,100) berada di dalam SM_Cube13 cover. Spawn 3 actor baru di posisi terbuka: Center(-600,0,100), Left(500,-800,100), Right(500,800,100). (3) Set label actors, verify transform, compile OK, save OK. |
| 2026-11-21 | Antigravity | Pembuatan mekanik Potion Pickup: (1) Buat `BP_PotionPickup` di `/Game/Splashfall/Blueprints/` (SphereComponent radius 60.0 OverlapAllDynamic, StaticMesh Cylinder scale 0.4 NoCollision, RespawnTime 8.0s, bIsAvailable true). (2) Wire EventGraph DSL: overlap -> guard bIsAvailable -> cast BP_Player -> guard PotionCount < MaxPotionCount -> PotionCount+1 -> bIsAvailable=false -> hide visual & disable collision -> delay 8s -> unhide, enable collision, bIsAvailable=true. Compile & save OK. (3) Tempatkan 3 instance di `Lvl_ThirdPerson` (Center, Left, Right). Save level OK. | Mekanik Potion Pickup selesai dan siap diverifikasi di PIE. |
| 2026-11-21 | User / Antigravity | Verifikasi PIE WBP_ResultScreen & Auto-Rematch: (1) User mengetes eliminasi saat HP=0 di PIE. (2) Result Screen tampil emas di tengah kedua layar split-screen. (3) Auto-rematch me-reload level setelah 4 detik. | Task M8 & M11 (Result Screen & Auto-Rematch) terverifikasi SUKSES di PIE. |
| 2026-11-21 | Antigravity | Implementasi Charged Throw & Trajectory Preview: (1) Checkpoint commit Result Screen & Auto-Rematch (`79a4bcb`). (2) `BP_PoisonProjectile`: MaxSpeed dinaikkan ke 2500.0, buat fungsi publik `InitThrowVelocity(NewVelocity: Vector)` via DSL. (3) `BP_Player`: variabel `bIsChargingThrow`, `CurrentChargeTime`, `MaxChargeTime` (1.5), `MinThrowSpeed` (600.0), `MaxThrowSpeed` (2200.0) di CDO. Buat fungsi `GetCurrentThrowSpeed` (Lerp), `UpdateThrowPreview` (`PredictProjectilePathByTraceChannel` ForOneFrame), dan perbarui `ThrowPotion` dengan kecepatan dinamis. (4) Wiring incremental `EventGraph`: `IA_Throw:Started` -> guard (Potion > 0 && bCanThrow && not bIsDead) -> charge=true + timer looping 0.02s; `IA_Throw:Completed` -> guard (bIsChargingThrow) -> charge=false + clear timer + ThrowPotion + delay cooldown. Integritas EventGraph terjaga. Compile & save OK. | Mekanik Charged Throw & Trajectory Preview selesai dan siap diverifikasi di PIE. |
| 2026-11-21 | User / Antigravity | Verifikasi PIE Charged Throw & Trajectory Preview: Quick tap menghasilkan lemparan pendek (~600.0), hold memunculkan garis prediksi lintasan dinamis real-time (hingga ~2200.0), botol meluncur sesuai lintasan dan spawn AOE saat impact. Guard amunisi & death aktif. | Charged Throw & Trajectory Preview terverifikasi SUKSES di PIE. |
| 2026-11-21 | Antigravity | Eksekusi task W2-A2: (1) Buat Material Instances `MI_Poison_P1` (hijau racun terang), `MI_Poison_P2` (ungu neon), dan `MI_PotionPickup` (biru cyan) di `/Game/Splashfall/Materials/` via `MaterialInstanceTools.create` & `set_vector_parameter`. (2) Update `BP_PoisonAOE:EventGraph`: EventBeginPlay pasang material P1/P2 dinamis ke AOEMesh berdasarkan `PlayerIndex` pemilik (`GetOwner -> CastToBP_Player -> GetPlayerIndex`). (3) Update `BP_PotionPickup:UserConstructionScript`: pasang `MI_PotionPickup` ke `PickupMesh`. (4) Bersihkan node `PrintString` di `BP_Player:ApplyPoisonTick`. Integritas `BP_Player:EventGraph` terjaga utuh. Compile & save seluruh aset OK. | Task W2-A2 selesai, seluruh Blueprint ter-compile tanpa error dan siap dites di PIE. |
| 2026-11-21 | User / Antigravity | Verifikasi PIE W2-A2 & Polish Layar Hasil: (1) Tes visual AOE hijau P1 dan ungu P2 serta pickup cyan terverifikasi berjalan sesuai. (2) Update `WBP_ResultScreen:SetWinnerText`: styling teks otomatis ("VICTORY" = emas, "DEFEAT" = merah tua via `MakeSlateColor`). (3) Update `BP_Player:HandleDeath`: spawn widget mandiri per player controller (`PC0` & `PC1`), tentukan status pemenang/pecundang berdasarkan `PlayerIndex` karakter yang gugur, dan tampilkan ke layar masing-masing via `AddToPlayerScreen`. Integritas `BP_Player:EventGraph` terjaga. Compile & save OK. | Tampilan hasil disesuaikan: pemain menang mendapat "VICTORY" emas, pemain kalah mendapat "DEFEAT" merah di layar masing-masing. Siap diverifikasi di PIE. |
| 2026-11-27 | Antigravity | Audit status Git & sinkronisasi commit 4 aset Blueprint (`BP_Player`, `BP_PlayerController`, `BP_PoisonProjectile`, `WBP_ResultScreen`) yang ter-resave otomatis oleh Unreal Editor saat penutupan sesi 21 Nov (ukuran byte & string table identik dengan `HEAD`); `Config/DefaultEngine.ini` (`bOffsetPlayerGamepadIds=True`) tetap dibiarkan unstaged sesuai aturan dev. | Working tree seluruh `.uasset` bersih; siap masuk ke pengerjaan task Minggu 2. |
| 2026-11-27 | Antigravity | Restrukturisasi `AGENTS.md` dari jadwal berbasis hari/minggu (`W1–W3`) menjadi rencana berbasis langkah berurutan (Langkah 1–8 / `L1–L8`) beserta sinkronisasi posisi & next action di `PROGRESS.md`. | Seluruh 30 task dan status `[x]` terjaga utuh dalam urutan langkah fungsional tanpa ketergantungan jadwal hari. |
| 2026-11-27 | Antigravity | Membuat `README.md` di root project berisi ringkasan game, fitur, kontrol (PS controller & fallback K+M), cara menjalankan di editor, serta struktur folder utama. | `README.md` siap digunakan sebagai dokumentasi awal repo. |
