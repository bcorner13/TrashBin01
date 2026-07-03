# intent — TrashBin01

> Human-input goal statement (per PROJECT_BOOTSTRAP.md). Draft reconstructed from the
> existing `TrashBin01.FCStd` model on 2026-07-03 — Bradley to correct if the intent differs.

## Goal

A fully parametric decorative trash bin / container: a tapered, thin-walled shell with a
repeating slotted ventilation pattern wrapped around the outer surface.

## Constraints

- Must follow `CAD_STANDARDS.md` (mm units, manifold/watertight, centered at origin).
- Everything parametric — every dimension bound to the embedded `VarSet` (see `CLAUDE.md`).
- Printable on the Creality K2 Plus; wall thickness tuned for a clean single-piece print.
- Target commercial price point $36–$45 (per `CAD_STANDARDS.md`), silk-filament friendly.

## Current state

- Geometry is modelled at **prototype scale** (Ø10 mm × 10 mm tall) — a proportion study,
  not final print size. Scaling up to production dimensions is future work driven entirely
  through the `VarSet` (BaseDiameter, WallHeight, etc.).
- Model is complete and fully constrained; no test print yet.
