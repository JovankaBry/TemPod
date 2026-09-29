# Library Agent

## Trigger
Activate when the user gives a new component to add to the parts library — a part number,
a datasheet link/file, or both. Also activate on **"add `<component>` to library"** (or
similar) — this specifically means the pattern below: files need to be moved AND documented.

## Workflow: "add `<component>` to library"
The user downloads vendor Altium library packages via Chrome, which land (still zipped/unzipped)
under `C:\Users\Ruben\Downloads\Chrome\...`. When told to add a component this way:

1. Search `C:\Users\Ruben\Downloads\Chrome\` for `.SchLib` / `.PcbLib` files matching the named
   component (folder names are often a distributor part ID, not the component name — search by
   file name, not folder name).
2. **Move** (not copy) the `.PcbLib` file to `PCB/Library/PCB/` and the `.SchLib` file to
   `PCB/Library/SCH/`, keeping the original file name.
3. **Clean up the download**: delete the source `.zip` the files came from (found as a sibling
   of the extracted folder, e.g. `Downloads\Chrome\10897080.zip` next to `Downloads\Chrome\
   10897080\`) AND delete the whole extracted folder itself — not just the two files taken from
   it. This is important and easy to forget: the goal is a clean Downloads folder with no
   leftover vendor-package clutter after every component is added.
4. Document the part as a new row in the Excel library (see Columns/Workflow below).
5. Do **not** open, import, or otherwise touch Altium itself — the user tests/validates the
   moved library manually in Altium. The agent's job ends at moving files + cleanup + the Excel
   row.
6. If Description/TempMin/TempMax/Datasheet Link aren't already known from earlier in the
   conversation, look them up or ask — don't leave them blank without trying.

**Expected order of operations** for the general case: the user creates/downloads the actual
`.SchLib`/`.PcbLib` files first, then asks to add the part to the Excel library. Always check
the actual library folders for the real file/component names **before** writing the row —
don't guess/default a Ref name if the files may already exist. Only fall back to a default
(part number) if genuinely nothing matching exists yet.

## Workflow: "delete `<component>`"
When the user says to delete a component (by Part Number), remove it completely:

1. Look up its row in `PartsLibrary` (by Part Number) to get its exact Library Path / Footprint
   Path.
2. Delete the `.SchLib` file (`PCB/Library/<Library Path>`) and the `.PcbLib` file
   (`PCB/Library/<Footprint Path>`).
3. Delete that row from the Excel `PartsLibrary` sheet.
4. Confirm what was removed (part number, both file paths, row number) — this is destructive,
   so state clearly what was deleted. If a library file is shared/referenced by more than one
   Excel row (rare, but check), flag that before deleting the file itself.

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
