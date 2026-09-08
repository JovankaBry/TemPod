# Library Agent — Memory

Compact, current-state facts only. Not a changelog — update in place, don't append history.

## File
- Target: `PCB/Library/TemPodLib.xlsx`, sheet `PartsLibrary`.
- Columns: Part Number, Description, TempMin, TempMax, Library Ref, Library Path, Footprint Ref, Footprint Path, Datasheet Link.
- No PartID or Value columns — intentionally excluded even if the user pastes data containing
  them (confirmed). Only fill the 9 columns above.

## Conventions
- Library Ref and Footprint Ref can differ (reverted from a brief "always identical" rule —
  real vendor libraries don't follow that, e.g. AP2112K-3.3's PcbLib file is named
  `AP2112K-3P3TRG1` but its footprint inside is named `SOT_RG1_DIO`). Always derive each from
  the actual file/component, never force them to match.
- Library Ref = name of the schematic symbol inside the SchLib (usually = file name).
- Footprint Ref = name of the footprint pattern inside the PcbLib (often a generic package
  descriptor, may differ from the PcbLib file name and from Library Ref).
- Schematic libraries live in `PCB/Library/SCH/`, PCB footprint libraries in `PCB/Library/PCB/`.
  Excel paths are relative to `PCB/Library/` (e.g. `SCH/DHT22.SchLib`).
- User's workflow: they create the SchLib/PcbLib files in Altium first, then ask to add the
  part to Excel — check the actual files before writing the row.

## Open questions
- None currently.
