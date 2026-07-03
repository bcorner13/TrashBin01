# Project rules — TrashBin01

**Everything in this project must be parametric.** The model reached the current session
already fully constrained and fully bound to its `VarSet` — that state is the asset. The
primary risk here is *degrading* it: hand-editing a coordinate or unbinding an expression to
"quickly" resize the bin instead of turning a `VarSet` knob. Don't. Read the rules below
before any geometry change.

> **How to use this template:** every rule cites *this project's* actual topology. This is a
> single-part model (one Body + surface-mapped pattern) with an **embedded VarSet**, not the
> multi-file `Params.FCStd` layout — that difference is deliberate and documented below.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context.

1. **Everything parametric.** All nine dimensions bind to the embedded `VarSet` today
   (`Sketch`, `Pad`, `Thickness`, `Sketch001`, `Array`, `Sketch_On_Surface` are all bound —
   verified 2026-07-03). The scale-up temptation is the danger: resize the bin by raising
   `BaseDiameter`/`WallHeight` in the VarSet, never by editing the base circle's radius or the
   pad length directly. If a value you need has no `VarSet` variable, **add one first**, then bind.

2. **No fixing geometry by editing raw sketch coordinates.** No prior incident in this project;
   rule applies preventively. `Sketch` and `Sketch001` are fully constrained — keep them that
   way. A misplaced slot is fixed by adjusting `SlotWidth`/`SlotAngle`/`BottomOffset`/`TopOffset`
   or a bound constraint, never by dragging a vertex or typing `StartX`/`CenterX`.

3. **Attach sketches to datum planes, not feature faces.** No prior DAG incident; rule applies
   preventively. `Sketch` and `Sketch001` are correctly attached to `XY_Plane` (a datum).
   **Known accepted exemption:** `Mapped_Sketch` is attached to `Thickness.Face1` and carries
   zero constraints — this is *required* by the Sketch-on-Surface addon, which maps a profile
   onto a target face. It is not a DAG-risk to be retargeted and not a parametric violation.
   Do not "fix" it by moving it to a datum — that breaks the surface mapping.

4. **Clearance concepts stay decoupled.** This is a **single printed part** with no mating
   interfaces (no lid, no snap, no press-fit), so there are no print-fit clearance Params yet.
   The one gap variable, `Psd` (0.1 mm), is a *surface-pattern offset* used in
   `Sketch_On_Surface.Offset = -(WallThickness + Psd)` — it is **not** a print-fit clearance and
   must not be overloaded as one. If a mating part is added later (e.g. a lid), give it its own
   clearance Param (`LidRailClearance`, `RecessClearance`, etc.) — never reuse `Psd` or
   `WallThickness`.

---

## Assembly architecture

Single body, no multi-part assembly. Bottom-up topology (this is what cannot be recovered by
skimming the FCStd):

- **Base circle** (`Sketch`, on `XY_Plane`) → **`Pad`** upward `WallHeight` with a `WallTaper`
  outward draft → **`Thickness`** shells it into a thin-walled open bin (the **Body tip**).
- **Slot band**: `Sketch001` defines one ventilation slot; `Array` repeats it along X at pitch
  `SlotWidth*3`; the band spans `WallHeight - TopOffset - BottomOffset` vertically.
- **Surface wrap**: `Sketch_On_Surface` (Sketch-on-Surface addon) maps `Mapped_Sketch` onto the
  shell's outer face (`Thickness.Face1`), cutting/embossing the slot pattern into the wall at
  offset `-(WallThickness + Psd)` and depth `WallThickness*2`.

Slot orientation and count follow from `SlotAngle` and the array interval — change those in the
VarSet to retune the pattern density.

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `TrashBin01.FCStd` | The whole model: Body (shell) + slot array + surface-mapped pattern + embedded `VarSet` | — (self-contained) | ✅ fully constrained, all dims bound |

No `Params.FCStd`, no cross-document XLinks, no broken files.

**Deliberate deviation from the workspace standard:** PROJECT_BOOTSTRAP.md expects a separate
`Params.FCStd` (VarSet-only) with `<<Params>>#VarSet.Var` cross-doc bindings. This project keeps
the **VarSet embedded** in `TrashBin01.FCStd` with local `VarSet.Var` bindings. Reason: the model
arrived complete and working; migrating the VarSet is a parametric operation with real breakage
risk and no functional payoff for a single-file model. Decided 2026-07-03. If the project ever
grows into multiple linked FCStds, revisit and migrate to `Params.FCStd` then.

---

## Params variables (summary)

Embedded `VarSet` (group "Base"), 9 driving variables:

- **Geometry:** `BaseDiameter` (10), `WallHeight` (10), `WallTaper` (2°), `WallThickness` (0.2)
- **Slot pattern:** `SlotWidth` (0.4), `SlotAngle` (52.43°), `BottomOffset` (1.5), `TopOffset` (0.4)
- **Surface offset:** `Psd` (0.1) — pattern-to-surface gap, not a print-fit clearance

All values are prototype-scale; production scale-up is driven from `BaseDiameter`/`WallHeight`.

---

## How to verify your change didn't break parametric

After any FreeCAD edit, before considering the task done:

```bash
python3 scripts/audit_parametric.py
```

It flags unconstrained sketches, unbound dimensional constraints, feature-face attachments, and
unbound feature dims.

**Known exemptions / caveats for this project:**

- `Mapped_Sketch` will show as `UNCONSTRAINED` (0 constraints) and attached to `Thickness.Face1`.
  This is **expected and correct** for the Sketch-on-Surface addon — ignore both flags for it.
- **Audit script has a known constraint-type-map bug** (documented in
  `~/.claude/projects/.../memory/reference_audit_parametric_type_bug.md`): the shipped
  `DIMENSIONAL_TYPES` table is off, so it can report geometric constraints (Tangent, Perpendicular,
  Block) as "unbound dimensions" and miss/mislabel real Radius/Diameter. When a flag looks
  suspicious, cross-check against the live API (`obj.Constraints[i].Type`) before touching geometry.
  The corrected enum is `{6:Distance, 7:DistanceX, 8:DistanceY, 9:Angle, 11:Radius, 18:Diameter}`.

---

## Memory files (deeper context)

`~/.claude/projects/-Users-bradleycorner/memory/MEMORY.md` indexes persistent memories. Most
relevant to this project:

- `reference_audit_parametric_type_bug.md` — the audit-script enum caveat cited above
- `feedback_freecad_use_mcp.md` — inspect/edit FCStd only via the MCP bridge

No project-scoped incident memories yet — this is a fresh project.

---

## Workflow notes

**Invariant (apply to every FreeCAD project — do not edit):**

- **Inspect/edit FreeCAD models via the MCP bridge — never with shell tools.** Do **not**
  `unzip`/`grep`/`cat`/`sed`/`strings`/etc. a `.FCStd`. Use `get_connection_status` first, then
  `open_document`, `list_objects`, `inspect_object`, `execute_python`, and macros. Enforced by the
  PreToolUse hook in `.claude/settings.json`.
- **MCP server auto-starts with FreeCAD.** If an `mcp__freecad__*` call fails, interpret it as
  "FreeCAD isn't running" — ask whether to launch it. Do not silently fall back to `unzip`.
- **Write changes as `macros/*.FCMacro` files**, not direct XML edits.
- **Cross-document expressions**: canonical form `<<Params>>#VarSet.VarName` — but note this
  project uses a **local embedded VarSet**, so bindings are `VarSet.VarName` (no `<<Params>>#`).
- **Run `python3 scripts/audit_parametric.py` before committing.**

**Project-specific:**

- Requires the **Sketch-on-Surface** FreeCAD addon (provides the `Part::FeaturePython`
  `Sketch_On_Surface` / `Array` features). Without it, those features won't recompute.
- Single-file model — no XLinks, so renaming is low-risk, but still rename via FreeCAD Save-As
  (as was done to produce `TrashBin01.FCStd` on 2026-07-03), not `mv`.

---

## Print profile

**No successful test print yet — profile TBD.** Geometry is at prototype scale; fill this in
from the first real print (material, layer height, walls, infill, orientation, date) before
treating any slicer settings as canonical.
