# Splashfall: Poison Arena

Game arena PvP 1v1 lokal (*couch multiplayer* / *split-screen* horizontal) berbasis **Unreal Engine 5**, di mana dua pemain saling berduel melempar botol racun dan memperebutkan kontrol area genangan (*poison AOE*).

> Dokumen rencana pengerjaan dan aturan development tersedia di [`AGENTS.md`](AGENTS.md). Catatan progres dan log sesi tersedia di [`PROGRESS.md`](PROGRESS.md).

---

## Fitur Utama

- **Local Split-Screen 1v1**: Dua pemain di satu PC dengan layar terbagi horizontal (P1 atas, P2 bawah), mendukung 2 gamepad PlayStation (DualShock 4 / DualSense) atau fallback Keyboard + Mouse untuk Player 1.
- **Charged Throw & Trajectory Preview**: Tahan tombol lempar untuk mengatur kekuatan lemparan (*Min–Max Speed*) sambil melihat prediksi lintasan proyektil secara *real-time*.
- **Poison AOE & Owner Color**: Botol racun pecah saat mengenai permukaan dan menghasilkan genangan AOE berdurasi 5 detik dengan warna pemilik (**Hijau** untuk P1, **Ungu** untuk P2). Memberikan *damage* berkala hanya kepada lawan (tanpa *friendly fire* dan tanpa *stacking*).
- **Potion Resource & Pickup**: Pemain mulai dengan 3 potion (maksimal 5). Item pickup berwarna **Biru Cyan** di arena memulihkan +1 potion dan *respawn* secara berkala.
- **HUD & Result Screen Per-Pemain**: Masing-masing viewport menampilkan status `HP` & `Potion` secara *real-time*, kondisi eliminasi *ragdoll* saat HP mencapai 0, serta layar hasil **VICTORY** / **DEFEAT** per pemain dengan *auto-rematch*.

---

## Kontrol

| Aksi | Gamepad PlayStation (Utama) | Keyboard + Mouse (Fallback Dev P1) |
|---|---|---|
| Gerak | Left Stick | `W` `A` `S` `D` |
| Bidik / Kamera | Right Stick | Mouse |
| Charge & Lempar Potion | `R2` (Tahan lalu lepas) | Klik Kiri (Tahan lalu lepas) |
| Lompat | `Cross` (X) | `Space` |
| Pause *(rencana)* | `Options` | `Esc` |

---

## Persyaratan & Cara Menjalankan (Development)

- **Engine**: Unreal Engine 5 (lihat `EngineAssociation` di `ProjectUAS.uproject`).
- **Platform**: Windows PC.

### Menjalankan di Editor
1. Buka `ProjectUAS.uproject` menggunakan Unreal Editor.
2. Pastikan level aktif adalah `/Game/ThirdPerson/Lvl_ThirdPerson`.
3. Tekan **Play (PIE)** untuk langsung menguji split-screen 2 pemain.

### Catatan Controller (1 Keyboard + 1 Stik PS)
Jika menguji dengan 1 Keyboard + Mouse (Player 1) dan 1 Stik PlayStation (Player 2), aktifkan opsi **Skip Assigning Gamepad to Player 1** di `Project Settings > Project > Maps & Modes > Local Multiplayer` (`bOffsetPlayerGamepadIds=True` di `Config/DefaultEngine.ini`). Jika stik PS tidak terbaca langsung oleh Windows, gunakan Steam Input atau DS4Windows.

---

## Struktur Folder Utama

```text
ProjectUAS/
├── Config/                  # Konfigurasi engine, input, dan split-screen
├── Content/
│   ├── Characters/          # BP_Player, BP_PlayerController, Mannequin & animasi
│   ├── Components/          # Komponen Health
│   ├── GameMode/            # BP_ThirdPersonGameMode
│   ├── Input/               # IMC_Default & Input Actions (IA_Move, IA_Throw, dll.)
│   ├── Splashfall/
│   │   ├── Blueprints/      # BP_GameInstance, BP_PoisonProjectile, BP_PoisonAOE, BP_PotionPickup
│   │   ├── Materials/       # MI_Poison_P1, MI_Poison_P2, MI_PotionPickup
│   │   └── UI/              # WBP_PlayerHUD, WBP_ResultScreen
│   └── ThirdPerson/         # Map aktif Lvl_ThirdPerson
├── AGENTS.md                # Scope, aturan agent, dan rencana berbasis langkah (Langkah 1–8)
├── PROGRESS.md              # Status terkini, fakta project, dan log sesi
└── ProjectUAS.uproject      # File project Unreal Engine
```
