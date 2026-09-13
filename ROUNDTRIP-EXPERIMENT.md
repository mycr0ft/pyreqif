# ReqIF ⇄ sysmlpy Round-Trip Experiment (2026-09-13)

**Verdict: the "Single Source of Truth" architecture works.** A real-world
ReqIF file (ReqIF Studio flavor, 137 requirements + relations) imports into
sysmlpy as a native requirement tree with full structural fidelity, and
sysmlpy round-trips it losslessly through its own text format.

## What was tested

- Input: `ReqIF_Studio_01_anonimized_example_sample.reqif` from
  strictdoc-project/reqif's anonymized corpus (Apache-2.0, publicly shared).
  137 SpecObjects ("Requirement Type"), 1 specification, 14 spec relations,
  hierarchy 3 levels deep, 138 hierarchy nodes.
- Path: ReqIF → (strictdoc `reqif` parser) → generated SysML v2 text →
  `sysmlpy.loads()` → live Model → re-parse stability check.
- Script: `/tmp/pyreqif-probe/roundtrip.py` (experiment-grade).

## Results

| Check | Result |
|---|---|
| ReqIF parses (strictdoc `reqif`) | 138 hierarchy nodes, 14 relations |
| Generated SysML text parses (`sysmlpy.loads`) | yes, 552 lines |
| Requirements materialize in the model | 138 (1 per hierarchy node) |
| Hierarchy preserved (depth sequence) | exact match, max depth 3 |
| Doc comments preserved | 137/137 (last-wins on duplicates) |
| Re-parse stability (text → model → identical) | yes |

## Mapping established

- SpecObject ("Requirement Type") → `requirement` usage
- `ReqIF.Text` (XHTML) → `doc /* ... */` (tags stripped)
- `ReqIF.ChapterName` / `ReqIF.ForeignID` → doc comment + future shortName
- SpecHierarchy nesting → requirement ownership nesting (1:1, verified
  depth-sequence identical)
- SpecRelations (14) → sidecar JSON (sysmlpy has no generic "derived link";
  mapping to `satisfy` would be semantically wrong for these)

## Caveats / follow-ups

1. **Two doc comments collapse to one.** `doc /* a */ doc /* b */` keeps
   only `b` on the API object (grammar preserves both; `dump()` emits one).
   Experiment merged ForeignID + Text into one doc per requirement, so
   nothing was lost here — but a production adapter should use distinct
   attributes (e.g. `attribute foreignId;`) rather than doc stacking.
2. **Real ForeignIDs are anonymized in this sample** ("...Anonymized...").
   With a real export they'd become `shortName`-style identifiers, which
   sysmlpy supports (`declaredShortName`).
3. **XHTML is stripped to text.** A production version should keep the
   XHTML in an attribute for lossless export back to ReqIF.
4. **SpecRelations → sidecar JSON** for now; if a real file uses
   `satisfy`-semantics relations, map those to actual `satisfy` usages.
5. **pyreqif (the fork we synced) should not be the engine** — it failed to
   parse 2 of 7 real-world samples. strictdoc's `reqif` parsed 7/7.

## Toolchain verdict

- sysmlpy: the SSOT (requirements, hierarchy, traceability, semantics)
- strictdoc `reqif` lib: the ReqIF engine (parse + unparse, active, Apache-2.0)
- pyreqif fork: reference only (dormant 2021, limited flavor support)
- Doorstop: candidate review/publishing front-end (its importer/exporter
  dispatch tables accept new formats easily)