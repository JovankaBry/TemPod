# Library Agent

## Trigger
Activate when the user gives a new component to add to the parts library — a part number,
a datasheet link/file, or both.

## Target file
`PCB/Library/TemPodLib.xlsx`, sheet `PartsLibrary`.

## Columns (in order)
1. **Part Number**
2. **Description** — short, from the datasheet (what the part is + key rating, e.g.
   "MOSFET P-Channel 50V 130mA SOT323")
3. **TempMin** — operating temp min, °C, from datasheet
4. **TempMax** — operating temp max, °C, from datasheet
5. **Library Ref** *(important)* — the Altium schematic library reference name. Always the same
   as Footprint Ref (unified naming convention).
6. **Library Path** *(important)* — path to the .SchLib file, always `SCH/<Library Ref>.SchLib`
7. **Footprint Ref** *(important)* — the Altium PCB footprint reference name. Always the same
   as Library Ref.
8. **Footprint Path** — path to the .PcbLib file, always `PCB/<Footprint Ref>.PcbLib`
9. **Datasheet Link** — URL to the datasheet

## Workflow for a new part
1. Get the datasheet (from a link/file the user gives, or by searching for
   `<part number> datasheet` if the user only gives a part number).
2. Extract: Description, TempMin, TempMax, package/footprint type, Datasheet Link.
3. Determine **Library Ref / Footprint Ref** (always identical — unified naming convention):
   check `SCH/`/`PCB/` for an existing library matching this part or its family. If none exists
   yet, default Ref = the part number (or a sensible short form matching this project's existing
   naming pattern — check `PCB/Library/TemPodLib.xlsx` for precedent first).
   Library Path = `SCH/<Ref>.SchLib`, Footprint Path = `PCB/<Ref>.PcbLib`.
4. Insert a new row into `PartsLibrary` with all 9 fields filled in. Never leave Library Ref,
   Library Path, or Footprint Ref blank — ask the user if genuinely unresolvable.
5. Update [`agent/memory.md`](memory.md) with anything newly learned (a naming convention
   confirmed, an exception, a recurring part family) — compact, not a log. Do not record
   routine additions there; only record durable facts/conventions.

## Conventions learned so far
See [`agent/memory.md`](memory.md) — always check it before adding a part, it may already
answer a naming question.
