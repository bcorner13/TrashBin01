# TrashBin01

Fully parametric decorative trash bin / container — a tapered thin-walled shell with a
repeating slotted ventilation pattern wrapped around the outer surface. Built in FreeCAD 1.1.

## Files

| File | Role |
|---|---|
| `TrashBin01.FCStd` | The model — Body (shell) + surface-mapped slot pattern + embedded `VarSet` |
| `intent.md` | Goal statement |
| `plan.md` | As-built parametric structure |
| `CLAUDE.md` | Project rules + assembly/topology notes for future sessions |
| `scripts/audit_parametric.py` | Parametric-integrity audit |
| `CAD_STANDARDS.md`, `MCP_TOOLS_REFERENCE.md`, `TOOLS.md` | Workspace standards (read-only copies) |

## Parametric usage

1. Open `TrashBin01.FCStd` in FreeCAD 1.1+ (with the FreeCAD MCP + Sketch-on-Surface addons).
2. Edit the **embedded `VarSet`** — every dimension binds to it (`VarSet.VarName`).
3. FreeCAD recomputes: shell, slot array, and surface mapping all update.

Key parameters (see `plan.md` for the full table):

- `BaseDiameter` — base circle diameter (default 10 mm)
- `WallHeight` — bin height (default 10 mm)
- `WallTaper` — outward draft angle (default 2°)
- `WallThickness` — shell thickness (default 0.2 mm)
- `SlotWidth` / `SlotAngle` — ventilation-slot size and inclination
- `BottomOffset` / `TopOffset` — slot-band margins

> **Scale:** the model is currently at prototype scale (Ø10 mm). Scale to production size by
> raising `BaseDiameter` / `WallHeight` in the VarSet — no geometry rework needed.

## Notes

- The embedded `VarSet` is a **deliberate deviation** from the workspace `Params.FCStd`
  convention (kept to avoid a risky migration of a working model). See `CLAUDE.md`.
- No successful test print yet — print profile is TBD.

## Print targets

- Bed: Creality K2 Plus / ELEGOO Saturn 4 · centered at origin · mm units.
- Target price point $36–$45; silk-filament friendly geometry.
