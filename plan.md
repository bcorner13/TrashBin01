# plan — TrashBin01

> Per PROJECT_BOOTSTRAP.md, this is the AI-generated plan. The model in `TrashBin01.FCStd`
> **already exists** and is fully constrained, so this document is written as an *as-built*
> record of the parametric structure rather than a forward build plan. Future geometry
> changes still route through this plan + `CLAUDE.md` rules (macros, no raw coordinate edits).

## PARAMETERS (embedded `VarSet` in `TrashBin01.FCStd`, group "Base")

| Name | Type | Default | Drives |
|---|---|---|---|
| `BaseDiameter` | Length | 10.0 mm | Base circle radius (`Sketch.Constraints[0]`) |
| `WallHeight` | Length | 10.0 mm | Pad length (bin height); slot-band height |
| `WallTaper` | Angle | 2.0° | Pad taper angle (outward draft) |
| `WallThickness` | Length | 0.2 mm | Shell (`Thickness`) value; surface-pattern depth |
| `BottomOffset` | Length | 1.5 mm | Slot band bottom margin (`Sketch001`) |
| `TopOffset` | Length | 0.4 mm | Slot band top margin |
| `SlotWidth` | Length | 0.4 mm | Slot half-width + array pitch (`SlotWidth*3`) |
| `SlotAngle` | Angle | 52.43° | Slot inclination in the pattern |
| `Psd` | Length | 0.1 mm | Pattern-surface offset gap (surface-mapped sketch) |

## FEATURE TREE (as built)

1. `Sketch` — base circle on `XY_Plane`, radius bound to `BaseDiameter`.
2. `Pad` — extrude `Sketch`; `Length ← WallHeight`, `TaperAngle ← WallTaper`.
3. `Thickness` — shell the pad into a thin-walled bin; `Value ← WallThickness`. **Body tip.**
4. `Sketch001` — slot profile on `XY_Plane`; height/width/angle/offsets bound to VarSet.
5. `Array` (Part::FeaturePython) — repeat the slot profile; `IntervalX ← SlotWidth*3`.
6. `Mapped_Sketch` + `Sketch_On_Surface` (Part::FeaturePython, Sketch-on-Surface addon) —
   wrap the pattern onto the shell face; `Offset ← -(WallThickness+Psd)`,
   `Thickness ← WallThickness*2`.

## CONSTRAINT STRATEGY

- `Sketch` and `Sketch001` are attached to `XY_Plane` (a datum, not a feature face) and are
  **fully constrained** with every dimensional constraint bound by expression to the VarSet.
- `Mapped_Sketch` is intentionally unconstrained and attached to `Thickness.Face1` — this is
  inherent to the Sketch-on-Surface addon (it maps a profile onto a target face) and is an
  accepted exemption, not a DAG-risk to be "fixed" (see `CLAUDE.md`).

## VALIDATION

- `python3 scripts/audit_parametric.py` (note the known constraint-type-map caveat in `CLAUDE.md`).
- Visual recompute in FreeCAD 1.1 with no errors; `Body.Tip = Thickness` resolves.
- Manifold/watertight check before any STL/3MF export to `stl/` and `3mf/`.
- Scale-up validation (future): raise `BaseDiameter`/`WallHeight` to production size and confirm
  the slot array and surface mapping recompute cleanly.
