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

## Color space across the roundtrip

### BRAW color science
BRAW enters FCP via BRAW Toolbox decoded to Rec.709 by default, or in BMD Film Gen 5 log for grading. Set this per-clip in BRAW Toolbox inspector.

**Rule**: if you're sending to Resolve for grading, import BRAW as BMD Film Gen 5 (log). Don't bake Rec.709 before grading — you lose latitude.

### What happens to color in FCP → Resolve

| Element | What happens |
|---------|-------------|
| FCP Color Board / Color Curves | ❌ Ignored by Resolve |
| FCP Enhance Light and Color (ML) | ❌ Ignored |
| Camera LUT applied in FCP | ⚠️ Baked into clip, non-editable |
| **BRAW source parameters (ISO, exposure, color temp)** | ✅ **Survive** — read directly from the .braw file by Resolve, not transmitted via FCPXML |
| Color Finale grades | ❌ FCP-only, incompatible with Resolve |

**Key distinction**: BRAW parameters survive because Resolve reads them from the source file directly — not because FCP transmitted them. The FCPXML is not involved. This means: if you leave your BRAW clips at their native parameters in FCP (no additional correction), Resolve will see the same starting point.

**Best practice**: apply no color correction in FCP during assembly. Leave BRAW raw. All grading happens in Resolve.

### Color Finale + Resolve
These two do not coexist. Color Finale is FCP-native — its node structure has no equivalent in Resolve. Grade in one or the other, not both.

### AI-generated clips in roundtrip
ComfyUI default output: 8-bit PNG or JPEG. **Do not use these in a grading timeline.**

Required pipeline:
```
ComfyUI → EXR 32-bit float (frame sequence)
        → ffmpeg → ProRes 4444 10-bit
                 → import into FCP/Resolve
                 → right-click → Input Color Space: Rec.709 SDR
```

See `docs/06-av1-archiving.md` for full bit-depth and color pipeline detail.

---

## Audio: what survives in roundtrip

Audio is actually the strongest part of the FCP → Resolve roundtrip.

### FCP audio export → Resolve or DAW

FCP exports audio via **Roles** — dialogue, music, effects, etc. as separate stems. Two paths:

**Path 1 — AAF via X2Pro**
- X2Pro exports an AAF with all audio tracks
- Resolve imports AAF: levels, panning, cuts, fades survive
- Pro Tools imports AAF natively
- This is the most information-preserving transfer for audio

**Path 2 — Stems via Roles export (FCP native)**
- FCP: File > Share > Export Roles as separate files
- Export each role as ProRes or WAV
- Import stems into Resolve's Fairlight
- No AAF needed, but you lose the mix structure (each track is flat)

**Path 3 — Logic Pro via DAWBridge**
- DAWBridge plugin: live sync between FCP timeline and Logic
- Changes in FCP reflect in Logic in real-time
- Best for Apple-native sound design workflow

### What survives audio-wise

| Element | Via AAF/X2Pro | Via Stems |
|---------|--------------|-----------|
| Audio cuts | ✅ | ✅ |
| Level/pan | ✅ | ❌ (flat) |
| Fades | ✅ | ❌ |
| FCP audio effects | ⚠️ basic only | ❌ |
| Role structure | ✅ | ✅ (as separate files) |

**Practical recommendation**: use X2Pro for audio roundtrip if you're going to Pro Tools or Resolve Fairlight. It preserves the most information.

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
