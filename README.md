# FCP Workflow

Final Cut Pro stack for auteur cinema — plugins, automation, AI integration, archiving.

Not a tutorial. Not a getting-started guide. A working reference documenting every tool, workflow, and integration decision in production use.

**Context**: I'm [Ismaël Joffroy Chandoutis](https://ismaeljoffroychandoutis.com/), a filmmaker and artist based in Paris. FCP is my primary NLE for assemblage and offline editing. DaVinci Resolve for grading. Blender VSE + ComfyUI for AI generation. This repo documents the seams between them.

---

## Stack

| Tool | Role | Status |
|------|------|--------|
| **Final Cut Pro 12.2** | Primary NLE (offline edit, rough cut, narrative assembly) | Production |
| **[SpliceKit](https://github.com/elliotttate/SpliceKit)** | Direct in-process FCP control via injected dylib — 78K ObjC classes, 200+ MCP tools | Production — primary programmatic layer |
| **DaVinci Resolve 20** | Color grading, online conform, sound prep | Production |
| **CommandPost v2.0.5** | Hardware panels, Lua scripting, batch tagging | Production |
| **Blender VSE + Pallaidium** | AI generation zones, generative sequences | Windows (primary) |
| **fcpxml-mcp-server** | Claude Code → FCP via FCPXML roundtrip (batch ops, XML-level work) | Active dev |
| **MLV App** | Magic Lantern RAW processing | Production |

---

## Documentation

| Document | What it covers |
|----------|---------------|
| [SpliceKit MCP](docs/00-splicekit-mcp.md) | Direct in-process FCP control — setup, MCP tools, macOS 26 compat |
| [CommandPost](docs/01-commandpost.md) | Automation, hardware panels, Lua scripting |
| [FCP 12 Features](docs/02-fcp12-features.md) | Semantic search, transcript search, FCPXML v1.14 |
| [FCPXML MCP Server](docs/03-fcpxml-mcp.md) | XML-level timeline editing via Claude Code |
| [Roundtrip Workflows](docs/04-roundtrip.md) | FCP ↔ DaVinci ↔ Blender VSE — pain points and workarounds |
| [Plugins & Tools](docs/05-plugins.md) | Full plugin stack, FCP Cafe recommendations |
| [AV1 & Archiving](docs/06-av1-archiving.md) | AV1/FFV1 archiving, SVT-AV1, hardware encoders, IMF |

---

## Key Findings (Feb 2026)

### What works well
- **SpliceKit MCP**: Claude Code controls FCP directly in-process — blade, transcript, scene detection, markers, color, effects, FCPXML export, captions, audio. No roundtrip. Real-time.
- FCP 12.2 semantic + transcript search saves hours on interview-heavy docs
- CommandPost: hardware panel control, batch keyword tagging
- `fcpxml-mcp-server`: XML-level QC, markers, batch operations (complementary to SpliceKit)
- FCP → DaVinci via FCPXML (old format): functional for grading passes
- FCP → Blender VSE via `fcpxml_import` (tin2tin): stable

### What's fragile
- SpliceKit on macOS 26.3 + FCP 12.2: DualTimeline, Lua VM, and EffectDrag crash — guard patch in [PR #51](https://github.com/elliotttate/SpliceKit/pull/51), works on macOS 26.4+
- FCPXML roundtrip FCP ↔ DaVinci: audio desync, compound clip issues, resolution corruption
- FCP ↔ Blender VSE return trip: unreliable via OTIO → use DaVinci as intermediary

### What's solved (was on roadmap)
- ~~CommandPost + Claude Code direct integration~~ → SpliceKit MCP (in-process, no AppleScript)
- ~~FCP semantic search API (UI-only)~~ → SpliceKit `get_transcript()` + `detect_scene_changes()` fill the gap programmatically
- ~~FCP → ComfyUI direct~~ → SpliceKit `export_xml()` → ComfyUI node → `import_fcpxml()` closes the loop

### Still missing
- [ ] Real SpliceKit fix for macOS 26.3 DualTimeline + Lua VM (waiting upstream)
- [ ] IMF creation from FCP workflow (requires separate tooling)

---

## Plugin Stack

### Essential (daily use)

| Plugin | What it does | Source |
|--------|-------------|--------|
| **BRAW Toolbox** | Native Blackmagic RAW import | FCP Cafe |
| **ScriptStar** | FCP transcript → keyword ranges (interviews) | FCP Cafe |
| **Gyroflow Toolbox** | Gyroscope stabilization | FCP Cafe |
| **CommandPost** | Automation + hardware control | commandpost.app |
| **LUTx** | Color look management | FCP Cafe |

### AI Tools

| Plugin | What it does | Status |
|--------|-------------|--------|
| **Jumper** | AI visual + transcript search, face recognition | Active dev |
| **fcpxml-mcp-server** | Claude Code timeline editing via FCPXML | Active dev |
| **Transcript → Keywords** | CommandPost built-in | Stable |

### Monitoring & QC

| Plugin | What it does |
|--------|-------------|
| **FCPXML MCP QC tools** | Flash frames, gaps, duplicates, health report |
| **CommandPost Search Console** | Rapid command palette |

---

## FCPXML MCP Server

The most interesting development in FCP AI tooling. Claude Code can analyze and edit FCP timelines via FCPXML.

**Repo**: [DareDev256/fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server)
**Tools**: 47 tools across 11 categories
**Tests**: 348 automated tests

### What Claude can do with your FCP timeline

```
Analysis:    project stats, metadata, clip list, EDL generation, keyword detection
Editing:     insert markers, trim clips, reorder, add transitions, speed ramps
QC:          flash frame detection, gap/silence finding, duplicate detection
Generation:  auto rough cut from keywords, montage assembly, A/B roll
Export:      DaVinci Resolve FCPXML, FCP7 XMEML
```

### Workflow

```
FCP → File → Export XML → share .fcpxml with Claude
Claude edits via MCP server
Import result back into FCP
```

### Limitations
- Batch only (no real-time)
- Audio representation limited in FCPXML
- Context limits for timelines with 1000+ clips

---

## FCP 12 — New Features

### Semantic Search (on-device ML)

- **Visual Search**: Natural language → objects, scenes, actions ("shots with water", "car driving")
- **Transcript Search**: Full-text search of auto-transcribed dialogue (~9.5 hrs transcribed in 5 min on M3 Max)
- **No public API** — UI-only, cannot be automated externally
- Both features run entirely on-device (privacy-preserving)

### For documentary work
Transcript search is genuinely production-useful. Import all interviews → FCP auto-transcribes → search by spoken keyword → smart collections. Replaces hours of manual logging.

### FCPXML v1.14
- Better plugin compatibility
- CommandPost v2.0.5 now fully compatible

### Known Bugs (v12.0 — wait for 12.1)
- Adjustment clips disappearing
- Export hangs at 99% (render complete, UI stalled)
- Background render failures with stabilization
- Crashes during force-quit exports

---

## CommandPost

**CommandPost v2.0.5** (released Feb 20, 2026)
Free, open source, MIT
[GitHub](https://github.com/CommandPost/CommandPost) | [commandpost.app](https://commandpost.app)

### What it adds to FCP
- Hardware panel support: Tangent, Monogram, Loupedeck
- Search console (command palette)
- Batch tagging: Titles to Keywords (essential for interview docs)
- Lua scripting engine for custom automation
- Multi-app control: AE, ProTools, any macOS app via AppleScript

### AI Integration
None native. External path: AppleScript → trigger Lua scripts → Claude could invoke via `osascript`. No production implementation exists yet.

### Claude Code + CommandPost Bridge (concept)
```bash
# Hypothetical: Claude triggers CommandPost action via AppleScript
osascript -e 'tell application "CommandPost" to ...'
# No working implementation yet — backlog candidate
```

---

## Roundtrip Workflows

See [docs/04-roundtrip.md](docs/04-roundtrip.md) for full detail.

### Recommended chain

```
FCP (rough cut, narrative assembly)
    │  Export FCPXML
    ▼
DaVinci Resolve (color grading, sound prep)
    │  Optional: OTIO export for Blender
    ▼
Blender VSE (AI generation zones — Pallaidium/ComfyUI)
    │  ProRes export
    ▼
DaVinci (final conform + master)
    │
    ▼  IMF package (if Netflix/major platform delivery)
```

### FCP → DaVinci pain points
- Export as FCPXML **old format** (not .fcpxmld — Resolve can't read it)
- Detach/flatten audio before export
- RED RAW: relink metadata manually after import
- Compound clips: manual decomposition in Resolve
- Storage: creates duplicate copies of all media

### FCP → Blender VSE
- Use [tin2tin/fcpxml_import](https://github.com/tin2tin/fcpxml_import) addon
- Stable, preserves timing and markers
- Media paths must be locally valid

---

## AV1 & Archiving

See [docs/06-av1-archiving.md](docs/06-av1-archiving.md) for full detail.

### TL;DR

| Format | Use | When |
|--------|-----|------|
| **FFV1** | Lossless archiving | Camera originals, proven standard |
| **AV1 lossless** | Lossless archiving | New projects, better compression (-30% vs FFV1) |
| **AV1 lossy** | Delivery | Netflix, streaming, web |
| **ProRes 422 HQ** | Intermediate | Editing, roundtrip |

### Hardware AV1 encoding
- **RTX 5090**: Triple NVENC AV1 (9th gen), 60% faster than RTX 4090
- **Apple Silicon M3+**: Decodes AV1 hardware, but **no hardware AV1 encoding** — CPU-only
- For Mac: offload AV1 encoding to RTX 5090 via Tailscale if needed

### SVT-AV1 3.0
- Now has lossless mode (new in 3.0)
- Best-in-class for cinema archiving
- `pip install av1an` wrapper for professional workflows

---

## IMF (Interoperable Master Format)

For Netflix/Amazon/major platform delivery.

**Tools:**
- [Netflix/photon](https://github.com/Netflix/photon) — validator (Java)
- [IMFTool](https://github.com/IMFTool/IMFTool) — CPL editor, versioning

**Reality check**: IMF is enterprise-grade. For indie cinema distributing to festivals or Vimeo, it's overkill. Matters if you're delivering to Netflix/Amazon.

---

## Resources

- [FCP Cafe](https://fcp.cafe/) — community hub, plugins, tools
- [CommandPost](https://commandpost.app/) — free automation app
- [fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server) — Claude Code + FCP
- [tin2tin/fcpxml_import](https://github.com/tin2tin/fcpxml_import) — Blender VSE import
- [FCPXML spec](https://developer.apple.com/library/archive/documentation/FinalCutProX/Reference/FinalCutProXXMLFormat/Introduction/Introduction.html) — Apple docs
- [Pipeline Neo](https://github.com/TheAhmadOsman/Pipeline) — Swift 6 FCPXML framework
- [SVT-AV1](https://gitlab.com/AOMediaCodec/SVT-AV1) — best AV1 encoder

---

## Influences & Links

- [open-source-cinema](https://github.com/12georgiadis/open-source-cinema) — Magic Lantern RAW, ML workflows, agent-driven editing
- [comfyui-cinema-pipeline](https://github.com/12georgiadis/comfyui-cinema-pipeline) — AI generation pipeline
- [claude-code-workflow](https://github.com/12georgiadis/claude-code-workflow) — Claude Code setup

---

*Filmmaker and artist based in Paris. [ismaeljoffroychandoutis.com](https://ismaeljoffroychandoutis.com)*
