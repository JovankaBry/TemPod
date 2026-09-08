# Library Agent

## Trigger
Activate when the user gives a new component to add to the parts library — a part number,
a datasheet link/file, or both.

**Expected order of operations**: the user creates the actual `.SchLib`/`.PcbLib` files in
Altium first, then asks to add the part to the Excel library. So always check the actual
library folders for the real file/component names **before** writing the row — don't
guess/default a Ref name if the files may already exist. Only fall back to a default (part
number) if genuinely nothing matching exists yet.

## Target file
`PCB/Library/TemPodLib.xlsx`, sheet `PartsLibrary`.

## Library folders
- Schematic libraries: `PCB/Library/SCH/`
- PCB footprint libraries: `PCB/Library/PCB/`
- Excel's Library Path / Footprint Path columns store paths **relative to `PCB/Library/`**
  (e.g. `SCH/DHT22.SchLib`), since that's where `TemPodLib.xlsx`/`TemPodLib.DbLib` live.

## Columns (in order)
1. **Part Number**
2. **Description** — short, from the datasheet (what the part is + key rating, e.g.
   "MOSFET P-Channel 50V 130mA SOT323")
3. **TempMin** — operating temp min, °C, from datasheet
4. **TempMax** — operating temp max, °C, from datasheet
5. **Library Ref** *(important)* — the name of the schematic symbol/component inside the
   `.SchLib` file. Usually matches the SchLib file name (one symbol per file), but confirm
   against the actual file.
6. **Library Path** *(important)* — path to the .SchLib file, `SCH/<SchLib file name>.SchLib`
7. **Footprint Ref** *(important)* — the name of the footprint pattern **inside** the `.PcbLib`
   file. This is NOT necessarily the same as the PcbLib file name or Library Ref — vendor
   libraries often name the footprint after the generic package (e.g. `SOT_RG1_DIO`) while the
   library file itself is named after the part number (e.g. `AP2112K-3P3TRG1.PcbLib`). Always
   confirm the actual footprint name from the file/user, never assume it matches Library Ref.
8. **Footprint Path** — path to the .PcbLib file, `PCB/<PcbLib file name>.PcbLib`
9. **Datasheet Link** — URL to the datasheet

## Workflow for a new part
1. Get the datasheet (from a link/file the user gives, or by searching for
   `<part number> datasheet` if the user only gives a part number).
2. Extract: Description, TempMin, TempMax, package/footprint type, Datasheet Link.
3. Check `PCB/Library/SCH/` and `PCB/Library/PCB/` for the actual files the user created for
   this part. Library Ref = the schematic symbol name inside the SchLib (usually = file name).
   Footprint Ref = the actual footprint name inside the PcbLib (may differ from both the file
   name and Library Ref — ask the user if it's not obvious, e.g. from a downloaded vendor
   library's internal footprint name). If no files exist yet, default both to the part number
   and flag it as a placeholder.
4. Insert a new row into `PartsLibrary` with all 9 fields filled in. Never leave Library Ref,
   Library Path, or Footprint Ref blank — ask the user if genuinely unresolvable.
5. Update [`memory.md`](memory.md) with anything newly learned (a naming convention confirmed,
   an exception, a recurring part family) — compact, not a log. Do not record routine additions
   there; only record durable facts/conventions.

## Conventions learned so far
See [`memory.md`](memory.md) — always check it before adding a part, it may already answer a
naming question.
