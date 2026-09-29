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
- "Add `<component>` to library" = also move the files: downloaded vendor libraries land under
  `C:\Users\Ruben\Downloads\Chrome\...` (folder name is often a distributor ID, not the part
  name — search by file name). Move `.PcbLib` → `PCB/Library/PCB/`, `.SchLib` →
  `PCB/Library/SCH/`, keep original file names, then add the Excel row. Never open/touch
  Altium itself — user validates manually.
- After moving the two files, delete the source `.zip` AND the whole extracted download folder
  (both live as siblings under `Downloads\Chrome\`, e.g. `10897080.zip` + `10897080\`) — keep
  Downloads clean of vendor-package clutter. Confirmed important by the user, don't skip it.
- "Delete `<component>`" = remove all three: the `.SchLib` file, the `.PcbLib` file, and the
  Excel row (look up exact paths from that row first). State what was deleted since it's
  destructive.
- Generic passives (resistors, capacitors) may be sourced by **copying** an existing SchLib/
  PcbLib from another of the user's Altium projects (e.g. `Marble-Station-ESP32`) rather than
  downloading/creating new — same move+document flow, just `cp` instead of `mv`/new-file.

## Open questions
- None currently.

## Notable exceptions
- WeAct 4.2 (row 6): no real Altium part existed online for the WeAct 4.2" E-Paper module, so
  the user implemented it as a generic 8-position vertical header matching the module's 8-pin
  interface instead — Part Number stays "WeAct 4.2" per the user's choice, but it's not
  actually the module's own footprint. TempMin/TempMax (-40/85) are an unverified generic
  header-connector estimate, not confirmed from the DigiKey page (blocked from fetching) — flag
  if precision matters later.
