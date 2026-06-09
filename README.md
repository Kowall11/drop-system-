# drop-system-
# 🔥 DS3 Tracker — Ashen One Chronicle

> *"Bearer of the curse... seek souls."*

A real-time Dark Souls III session tracker built in C# / WPF. Monitor your journey through Lothric — track deaths, boss fights, HP, stamina, souls, and your path through the world.

---

![DS3 Tracker Screenshot](tracker_screenshot.png)

---

## ✨ Features

| Feature | Description |
|---|---|
| **HP / Stamina Charts** | Live line charts showing HP and stamina history over the last 5 minutes |
| **Death Heatmap** | Area-specific death map overlaid on the DS3 world map — see exactly where you keep dying |
| **Boss Fights** | Tracks every boss encounter — attempts, best time, last killed |
| **Journey Timeline** | Full list of all DS3 bosses with split times as you defeat them |
| **IGT Timer** | In-game time (pauses during loads) — same timer used by speedrunners |
| **Souls Counter** | Real-time soul count |
| **Location Tracker** | Shows your current area using online area IDs |
| **Auto-reconnect** | Automatically reconnects if the game restarts |

---

## 🎮 How It Works

The tracker reads Dark Souls III's memory in real time using the Windows `ReadProcessMemory` API. It resolves **pointer chains** — sequences of memory addresses that lead to the actual values like HP, position, and IGT.

```
DarkSoulsIII.exe
  + Static Offset          ← anchored in the .exe (survives ASLR)
    → Heap Object A        ← dereferenced 8-byte pointer
      → Heap Object B      ← dereferenced 8-byte pointer
        → Player Object
          + Field Offset   ← e.g. +0x138 = Current HP ✓
```

Boss detection uses **event flags** — the same system the game uses internally to track which bosses have been killed. When a boss flag flips from `false` to `true`, a fight-ended event fires and the Journey Timeline updates.

---

## 🏗️ Architecture

```
DS3Tracker/
├── Core/
│   ├── readMemory.cs        ← Windows API, process handle
│   ├── resolveChain.cs      ← walks pointer chains
│   └── DS3Addresses.cs      ← all known pointer paths for v1.15.2
│
├── Tracking/
│   ├── DS3Poller.cs         ← background thread, 200ms tick loop
│   ├── DS3Snapshot.cs       ← typed data container for one tick
│   ├── EventDetector.cs     ← diffs snapshots, fires events
│   └── DataStore.cs         ← session history storage
│
├── Assets/                  ← icons, world map, background
└── MainWindow.xaml/.cs      ← WPF dashboard
```

### Data Flow

```
DarkSoulsIII.exe
    ↓  ReadProcessMemory (every 200ms)
DS3Poller  ──→  SoulMemory.DarkSouls3 (IGT, position, event flags)
    ↓
EventDetector  ──→  PlayerDied
               ──→  BossFightStarted / BossFightEnded
               ──→  SplitReached
    ↓
DataStore  ──→  MainWindow (WPF charts + panels)
```

---

## 🛠️ Tech Stack

- **C# / .NET 8** — Windows WPF application
- **SoulMemory** (NuGet) — IGT, position, boss event flags
- **Windows API** — `ReadProcessMemory`, `OpenProcess`
- **WPF Canvas** — custom chart and heatmap rendering

---

## 📋 Requirements

- Windows 10/11 (x64)
- .NET 8 Runtime
- Dark Souls III v1.15.2
- Run both the game and tracker **as Administrator**

---

## 📍 Tracked Memory Addresses (v1.15.2)

| Value | Pointer Chain |
|---|---|
| Current HP | `0x477FDB8 → +0x80 → +0x1F90 → +0x18 → +0xD8` |
| Max HP | `0x477FDB8 → +0x80 → +0x1F90 → +0x18 → +0xDC` |
| Stamina | `0x477FDB8 → +0x80 → +0x1F90 → +0x18 → +0x148` |
| Position X/Y/Z | via SoulMemory `GetPosition()` |
| Deaths | `0x47572B8 → +0x98` |
| Souls | `0x478ECC0 → +0x0 → +0x40 → +0x3BC` |
| IGT | via SoulMemory `GetInGameTimeMilliseconds()` |
| Area ID | `0x478ECC0 → +0x0 → +0x40 → +0x1F8` |
| Boss Flags | via SoulMemory `ReadEventFlag(Boss.*)` |

> ⚠️ These offsets are valid for **v1.15.2** only. A game patch may change them.

---

## 🎨 Concept & Design

The UI is inspired by the dark, atmospheric aesthetic of Dark Souls III — parchment text, gold accents, and a world map heatmap that shows the player's deaths across Lothric.

![Concept Art](concept_art.png)

---

## ⚠️ Disclaimer

This tool is for **single-player / offline use only**. Using memory readers online may violate FromSoftware's Terms of Service and could result in a ban. Always launch DS3 offline when using this tracker.

---

## 📚 Credits & Resources

- **SoulMemory** by [FrankvdStam](https://github.com/FrankvdStam/SoulSplitter) — DS3 memory reading library
- **The Grand Archives** — DS3 Cheat Engine table v3.3.2 (v1.15.2)
- **DS3 modding community** — pointer addresses and event flag documentation
- [DarkSoulsMemoryReader](https://github.com/srogee/DarkSoulsMemoryReader) — online area ID reference

---

*Praise the sun.* ☀️
