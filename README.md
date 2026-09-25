# SMC-PAD for VirtualDJ

Configuration files for using the **M-VAVE SMC-PAD** (16-pad wireless MIDI pad controller) with **VirtualDJ**.

[中文说明（Chinese）](README.zh-CN.md)

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [How It Works](#2-how-it-works)
3. [Choosing & Using Config Files](#3-choosing--using-config-files)
4. [FAQ](#4-faq)
   - [4-1 Hardcoded buttons](#4-1-hardcoded-buttons-some-keys-send-no-midi)
   - [4-2 Device appears as multiple devices](#4-2-the-device-appears-as-multiple-devices)
5. [Directory Structure](#5-directory-structure)
6. [License](#6-license)

---

## 1. Introduction

This repository provides **Device definitions** and **Mapper files** for controlling VirtualDJ with the M-VAVE SMC-PAD.

The SMC-PAD is a 16-pad wireless MIDI controller featuring:

- 16 RGB back-lit pads (velocity + aftertouch sensitive)
- 8 endless 360° encoders
- 5 assignable buttons: `LEFT` `RIGHT` `PLAY` `STOP` `RECORD`
- Hardware buttons (`BT`, `PAD BANK`, `KNOB BANK`, `SHIFT`, `Note Repeat`) — **not mappable** (see FAQ)

> Note: the RGB pad lighting is **not** controllable via MIDI (idle color is set in the MidiSuite software; pads flash white when pressed). This is a hardware limitation, not a mapping issue.

---

## 2. How It Works

VirtualDJ uses two kinds of XML files:

| File | Location | Purpose |
| --- | --- | --- |
| **Device definition** | `Documents\VirtualDJ\Devices\` | Defines the hardware elements (pads, encoders, buttons) and how the controller is detected (VID/PID). |
| **Mapper** | `Documents\VirtualDJ\Mappers\` | Maps the defined elements to VDJ actions. |

This project is based on the SMC-PAD **default preset** (slots 3–8 of its 8 preset slots).

**MIDI layout used (default preset):**

| Control | Type | Channel | Values |
| --- | --- | --- | --- |
| Pads 1–16 | Note On/Off | CH10 | 36–51 (mirror-reordered for DJ convention) |
| Pad velocity | Note On velocity | CH10 | 0–127 → 0–100% slider |
| Encoders 1–8 | Relative CC | CH1 | 30–37 (reordered) |
| Buttons | CC push | CH1 | 25–29 (`LEFT`–`RECORD`), 127 = press, 0 = release |

**Key implementation points:**

- Pads are **vertically mirrored** so that pad 1 is top-left (DJ convention) instead of bottom-left (SMC-PAD factory order).
- Encoders are **reordered** to match conventional DJ knob layout.
- Pad **velocity** is exposed as an extra `<slider>` element per pad, so hitting a pad harder can drive continuous parameters (filter, effect amount, etc.).

---

## 3. Choosing & Using Config Files

### 3.1 Choose

1. **Device definition** — from `devices/`, pick one based on your preferred deck layout (e.g. `decks4` = 2 sides × 2 layers, `decks2` = 2 decks). See `devices/README.md`.
2. **Mapper** — from `mappers/<same-layout>/`, pick one based on your preferred button/function layout. Each mapper folder has its own README describing the button layout.

### 3.2 Install

1. Copy the chosen **Device** XML into:
   ```
   Documents\VirtualDJ\Devices\
   ```
2. Copy the chosen **Mapper** XML into:
   ```
   Documents\VirtualDJ\Mappers\
   ```
3. Restart VirtualDJ.
4. Open **Settings → Controllers**, find the SMC-PAD and select the mapper.

---

## 4. FAQ

### 4-1 Hardcoded buttons (some keys send no MIDI)

The following keys are **hardware-encoded** — they send **no MIDI message** and therefore **cannot be mapped**:

- `BT` — Bluetooth on/off
- `PAD BANK` — toggles pads between notes 36–51 and 52–67
- `KNOB BANK` — toggles encoders between CC 30–37 and 38–45
- `SHIFT` — modifier for presets / velocity curve / transpose / octave
- `Note Repeat` — internal note-repeat function

Only the **16 pads**, **8 encoders**, and **5 buttons** (`LEFT` `RIGHT` `PLAY` `STOP` `RECORD`) are mappable. This project intentionally does not define the second bank (notes 52–67 / CC 38–45).

### 4-2 The device appears as multiple devices

**Symptom:** VirtualDJ shows three devices, e.g. `SMC PAD`, `SMC PAD (MIDI IN 1)`, `SMC PAD (MIDI IN 2)`.

**Cause:** the SMC-PAD exposes **multiple MIDI ports**, and all ports share the same USB **VID/PID**. VirtualDJ treats each MIDI port as a separate controller, so the same definition matches all of them.

**Solution:** in **Settings → Controllers**, set the **Mapping** of the extra ports to **`None`** (or use the power toggle next to each entry), keeping only the port that carries the controller data. This is normal behavior for multi-port MIDI devices.

---

## 5. Directory Structure

```
ForGithub/
├── README.md                    ← this file (English)
├── README.zh-CN.md              ← Chinese version
├── LICENSE                      ← MIT license
├── devices/                     ← Device definitions (multiple)
│   ├── README.md                ← how to choose a device definition
│   ├── SMC-PAD_decks4.xml
│   ├── SMC-PAD_decks2.xml
│   └── ...
├── mappers/                     ← Mapper files (grouped by device layout)
│   ├── README.md                ← how to choose a mapper
│   ├── decks4/
│   │   ├── README.md            ← button/function layout for this set
│   │   ├── SMC-PAD_decks4_basic.xml
│   │   └── ...
│   └── decks2/
│       ├── README.md
│       └── ...
└── docs/                        ← supplementary documents
    └── layout.md                ← pad/knob/button layout reference
```

---

## 6. License

Released under the **MIT License**. See [LICENSE](LICENSE).
