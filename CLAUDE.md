# TempPod

Battery-powered ESP32 device that measures temperature + humidity, shows readings on an E-Ink
display, and reports data over WiFi every 30 minutes. Firmware updates handled via OTA,
built/published through a GitHub Actions pipeline.

## Project layout

- `PCB/` — Altium PCB library files (`.PcbLib`) and layout. `PCB/Library/TemPodLib.xlsx` is the
  parts database (see Agents below).
- `SCH/` — Altium schematic library files (`.SchLib`).
- `.claude/agent/` — agent instruction files (see Agents below). Not firmware/hardware — do not
  treat as project source.

Firmware, server, and casing directories don't exist yet (see Status).

## Hardware reference

**Sensor — DHT22 (AM2302)**, temp + humidity, through-hole:
- Pin 1 VCC → 3.3V
- Pin 2 DATA → ESP32 GPIO (any) + 10k pull-up to VCC
- Pin 3 NC → not connected
- Pin 4 GND → GND

**Display — WeAct 4.2" E-Paper, 400x300, SPI, Black-White-Red**:
- VCC → 3.3V, GND → GND
- SDA(MOSI) → GPIO23, SCL(Clock) → GPIO18, CS → GPIO5, D/C → GPIO17, RES → GPIO16, BUSY → GPIO4
- Labeled SDA/SCL but this is SPI, not I2C — use WeAct/Waveshare example library.

**MCU — ESP32-WROOM-32** (bare module, not dev board):
- Needs external 3.3V LDO (low quiescent current, e.g. MCP1700 — not generic AMS1117, to
  preserve deep sleep battery life)
- Needs EN pull-up (10k to 3.3V + ~100nF cap to GND) and IO0 pull-up (10k to 3.3V)

**Power**: LiPo 3.7V 500–1000mAh + TP4056 charging module (with battery protection) + 3.3V LDO.
No 5V step-up needed — DHT22 and E-Ink both run on 3.3V.

**Wake button** (manual "check for update now"):
- One side → GPIO33 (RTC-capable), other side → GND, 10k pull-up GPIO33 → 3.3V regulated rail
- `esp_sleep_enable_ext0_wakeup(GPIO_NUM_33, 0)` — wake on LOW
- On wake, check `esp_sleep_get_wakeup_cause()` to distinguish button press
  (`ESP_SLEEP_WAKEUP_EXT0`) from timer wake, and trigger OTA check accordingly.

**Programming/flashing**: bare UART pins on PCB, no onboard USB-serial chip. Use external
CP2102/CH340G breakout (manual boot: hold IO0 LOW, toggle EN, release IO0, upload) or ESP-Prog.

## Firmware flow (not yet implemented)

1. Wake (30 min timer, or button press via ext0 interrupt)
2. Read DHT22 (temp + humidity)
3. Update E-Ink display with latest reading
4. Connect WiFi
5. HTTP POST reading to server
6. Periodically / daily / on button wake: check GitHub Release for newer firmware, OTA-flash if found
7. Back to deep sleep

## OTA / CI/CD (not yet implemented)

- Firmware built via PlatformIO in GitHub Actions on push to `main`
- Built `.bin` published as a GitHub Release
- ESP32 polls the GitHub Release URL during wake cycle, downloads + OTA-flashes if newer
- Needs two firmware partitions (current + incoming) — standard 4MB WROOM flash handles this

## Status / open next steps

1. Firmware code (deep sleep, sensor read, display update, WiFi/HTTP POST, OTA check, button
   interrupt) — not started
2. GitHub Actions workflow (PlatformIO build → Release) — not started
3. Server-side data receiver for the 30-min readings — not decided
4. Schematic (Altium) — in progress, done manually by the user in Altium; Claude does not drive
   Altium directly
5. PCB layout/routing — not started
6. Casing (3D printed) — not started

## Agents

- **Library agent** ([.claude/agent/library-agent.md](.claude/agent/library-agent.md)) — trigger:
  the user gives a new component (part number and/or datasheet) to add to the parts library.
  Maintains `PCB/Library/TemPodLib.xlsx`. Always read its compact memory first —
  [.claude/agent/memory.md](.claude/agent/memory.md) — it may already answer a naming/convention
  question. Update that memory only with durable facts/conventions, never a log of routine additions.

## Working conventions

- Confirm before creating/editing files unless the user has clearly already approved the specific
  change in this conversation.
- Never invent Library Ref / Library Path / Footprint Ref / Footprint Path values — derive them
  from the conventions in the library agent file, or ask.
- Claude cannot operate Altium (GUI-only tool). For schematic/PCB layout work, provide reference
  data (net lists, pin mappings, part values) — the user does the actual Altium work.
