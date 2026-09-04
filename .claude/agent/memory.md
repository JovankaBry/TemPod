# Library Agent — Memory

Compact, current-state facts only. Not a changelog — update in place, don't append history.

## File
- Target: `PCB/Library/TemPodLib.xlsx`, sheet `PartsLibrary`.
- Columns: Part Number, Description, TempMin, TempMax, Library Ref, Library Path, Footprint Ref, Footprint Path, Datasheet Link.
- No PartID or Value columns — intentionally excluded even if the user pastes data containing
  them (confirmed). Only fill the 9 columns above.

## Conventions
- Library Ref = Footprint Ref, always identical (unified naming, confirmed by user).
- Library Path = `SCH/<Ref>.SchLib`
- Footprint Path = `PCB/<Ref>.PcbLib`
- `SCH/` and `PCB/` folders hold the actual Altium library files as the project grows.

## Open questions
- ESP32-WROOM-32 row (added before the unified-naming rule) still has Library Ref
  `ESP32-WROOM-32` ≠ Footprint Ref `MODULE_ESP32-WROOM-32` — not yet reconciled with the user.
