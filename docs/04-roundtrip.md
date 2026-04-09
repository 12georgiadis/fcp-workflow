# 04 - Roundtrip Workflows

FCP ↔ DaVinci Resolve ↔ Blender VSE. What works, what breaks, and how to survive it.

Updated April 2026.

---

## The reality

No perfect roundtrip exists between FCP and Resolve. FCPXML is the bridge, but it's lossy. You need to know exactly what survives and what doesn't before you commit to a workflow.

**Core recommendation**: decide which tool owns which phase. Don't bounce back and forth.

```
FCP = assembly, rough cut, narrative structure, interview-driven editing
DaVinci Resolve = color grading, online conform, audio prep
Blender VSE = AI generation zones, generative sequences, VFX compositing
```

---

## FCP → DaVinci Resolve

### Method
1. In FCP: File > Export XML (FCPXML)
2. In Resolve: File > Import > Timeline > select .fcpxml

### What survives

| Element | Status |
|---------|--------|
| Cuts and edits | ✅ Yes |
| Clip order | ✅ Yes |
| Basic transitions (dissolves) | ✅ Usually |
| Audio levels | ⚠️ Partially |
| Markers | ❌ No |
| FCP native effects | ❌ No (FCP-only) |
| FXFactory effects | ❌ No |
| Color grades from FCP | ❌ No |
| Compound clips | ⚠️ Becomes linked clips |
| Multicam clips | ❌ No (collapse to single angle before export) |
| Connected storylines | ⚠️ Sometimes |
| Audio AIF files | ❌ No (convert to WAV first) |

### Critical gotcha: FCPXML version mismatch

FCP exports FCPXML v1.9–v1.11. Older Resolve versions only import up to v1.8.

**Fix**: Use **Capacitor** (LateNite) to downconvert FCPXML from v1.11 → v1.8 before importing into Resolve.

Check Resolve's current FCPXML support in: Preferences > System > General.

### Critical gotcha: timecode conflicts

FCP ignores source timecode by default. Resolve reads it. If your clips have source timecode and FCP has reinterpreted start times, media may go offline in Resolve.

**Fix**: before editing in FCP, check that timecodes are consistent. Or relink manually in Resolve.

### Preparation checklist before FCP → Resolve

- [ ] Collapse all multicam clips to single angle
- [ ] Convert any AIF audio to WAV
- [ ] Export FCPXML and check version (use Capacitor if needed)
- [ ] Note any FCP-native effects — they won't survive, add them again in Resolve
- [ ] Lock your edit before sending to Resolve. Do not continue editing in FCP after this point.

---

## DaVinci Resolve → FCP

### When you'd do this
Grading is done in Resolve. You want to bring the graded timeline back into FCP for finishing, titles, or delivery.

### Reality
Don't expect grades to survive. FCPXML has no concept of Resolve color nodes.

**Viable approach**: export graded media from Resolve as self-contained files, then relink in FCP.

1. Resolve: File > Export > Individual Clips (graded, self-contained)
2. FCP: create new project, import graded files
3. Relink to the graded media

This is not a true roundtrip — it's an export of baked media.

### For conform workflow (FCP rough cut → Resolve grade → FCP final)

1. FCP: export FCPXML (rough cut, ungraded)
2. Resolve: import, grade
3. Resolve: export EDL or AAF
4. FCP: use EDL to rebuild timeline with graded media

This is how professional post houses handle it. It requires an intermediary EDL step.

---

## FCP → Blender VSE

For AI generation zones and generative sequences.

### Method
1. FCP: export FCPXML of the relevant sequence
2. Python script: parse FCPXML, extract clip timecodes and durations
3. Blender Python API: rebuild timeline in VSE from parsed data
4. Add AI generation nodes (ComfyUI via `comfyui-cinema-pipeline`)
5. Render AI sequences
6. Import rendered sequences back into FCP

### fcpxml-mcp-server
Claude Code can read and write FCPXML directly via the `fcpxml-mcp-server` MCP.

This enables:
- AI-generated rough cuts: Claude analyzes transcripts and builds FCPXML timelines
- Marker automation: Claude adds markers at significant moments
- QC reports: Claude checks FCPXML for structural issues
- Blender bridge: Claude coordinates FCPXML → Blender VSE pipeline

→ See `docs/03-fcpxml-mcp.md` for full documentation.

---

## Blender VSE → FCP

After AI generation in Blender:

1. Render sequences as ProRes 4444 or EXR sequences
2. Import rendered files into FCP
3. Drop into timeline at correct timecode positions (from FCPXML reference)

Timecode alignment requires careful tracking — keep a spreadsheet or use Claude Code to cross-reference.

---

## Audio: FCP → Pro Tools / Logic

For final sound mix.

### Via X2Pro (FCP → Pro Tools)
- Exports audio tracks with EDL to AAF format
- Pro Tools imports AAF natively
- Preserves: audio levels, cuts, fades, basic audio effects
- Loses: FCP-native audio processing beyond the basics

### Via Logic Pro (native Apple)
- **DAWBridge** plugin: live sync between FCP timeline and Logic session
- Changes in FCP reflect in Logic in real-time
- More integrated than Pro Tools route for Apple-native workflows

---

## Recommended workflow for Goldberg

Given the production context (BRAW, 1000+ clips, AI generation zones, DIT backup):

```
PHASE 1 — Assembly (FCP)
  - Import via BRAW Toolbox
  - Auto-apply per-clip LUTs with LUT Robot
  - Log and tag in Lumberjack / FCP keywords
  - Fast Collections for character/theme navigation
  - Rough cut: interview structure first, then B-roll
  - Generate transcript-driven assembly with Claude Code via fcpxml-mcp-server

PHASE 2 — AI Generation (Blender VSE / ComfyUI)
  - Export FCPXML of sequences needing AI treatment
  - Claude Code parses FCPXML, generates Blender scene
  - ComfyUI pipeline: Wan 2.2 / SV3D / FLUX LoRA
  - Render to ProRes 4444
  - Import back into FCP

PHASE 3 — Grade (DaVinci Resolve)
  - Export locked FCPXML from FCP
  - Capacitor: downconvert to v1.8 if needed
  - Import into Resolve
  - Grade
  - Export graded media (self-contained clips)

PHASE 4 — Online / Deliver (FCP or Resolve)
  - FCP: reassemble with graded media, add titles, graphics
  - Or Resolve: full online conform in Resolve 20
  - Export DCP or broadcast master
```

---

*Last updated: April 2026.*
