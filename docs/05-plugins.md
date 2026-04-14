# 05 - Plugins & Tools

Complete catalogue of every FCP plugin and tool. Opinionated. Updated April 2026.

---

## LateNite Ecosystem (fcp.cafe)

The most coherent plugin ecosystem for professional FCP use. Built by Chris Hocking (LateNite Films, Melbourne).

### Bundles

| Bundle | Price USD | Apps included | Recommendation |
|--------|-----------|---------------|----------------|
| **Pro Editor Bundle** | $100 | BRAW Toolbox, Gyroflow, Marker Toolbox, Recall Toolbox, Fast Collections | Best starting point |
| **Pro Editor Bundle v2** | $150 | + LUT Robot, Capacitor | Add if you do per-clip LUT workflows |
| **Pro Editor Bundle v3** | $180 | + Metaburner, News Import | Only if you need burn-ins or newsroom integration |

**"Complete My Bundle"**: App Store credits you for apps already owned. Check before buying a bundle.

---

### Individual Apps

#### BRAW Toolbox — $79.99
Native Blackmagic RAW import into FCP. No transcoding.
- RAW parameter adjustment before/after import (ISO, exposure, color temp, tint, gamma)
- Keyframeable controls
- Auto decode quality for HD/UltraHD projects
- Works with DaVinci Resolve for grading roundtrips

**Verdict**: Essential if you shoot BRAW. The alternative is transcoding everything to ProRes, which wastes time and space.

---

#### LUT Robot — in Bundle v2+
Automatically matches Camera LUTs to clips by filename.

**The manual problem**: DIT delivers LUTs named per roll or per clip. Without LUT Robot, you open each clip in Inspector > Effects > Custom LUT and select manually. On a 1000-clip documentary, that's hours.

**How LUT Robot works**:
1. Point it at your LUT folder
2. It scans filenames and matches them to clip names in your FCP event
3. Camera LUT applied automatically to every matched clip
4. Specify subfolder to narrow the scan

**When it's useful**: DIT provides per-clip or per-roll LUTs (naming convention: `A022_04061818_C001.cube` matches `A022_04061818_C001.braw`). If you use a single project LUT, you don't need it.

**Verdict for Goldberg**: Useful if Hadrien delivers LUTs per roll or per clip.

---

#### Gyroflow Toolbox — in Bundle
Native gyroscope stabilization inside FCP. Processes gyro data from camera internal recording or external logger.
- No export/import loop
- Works directly on timeline
- Supports most camera formats including BRAW

---

#### Fast Collections — in Bundle
Generates Smart Collections from keyword lists instantly.

FCP's built-in Smart Collection panel becomes slow with thousands of keywords. Fast Collections bypasses the bottleneck.

**For documentary**: Define your full keyword taxonomy before import (characters, themes, locations, quality tags). Fast Collections creates all collections in seconds instead of waiting minutes.

---

#### Marker Toolbox — in Bundle
Imports feedback markers from review platforms into FCP as native markers.
- Vimeo
- Wipster
- Dropbox Replay
- Email (via .csv export)

---

#### Capacitor — in Bundle v2+
Converts FCPXML between versions (v1.8 ↔ v1.11).

**Important**: This is NOT a Premiere/Resolve converter. It only converts FCPXML version numbers. Useful for compatibility issues between FCP versions or third-party tools that lag on FCPXML spec updates.

---

#### CommandPost — FREE (open source)
**Download**: [commandpost.fcp.cafe](https://commandpost.fcp.cafe)

258,000+ downloads. Used at Netflix, Pixar, BBC.

**Use CommandPost for:**
- Hardware panel support (Stream Deck, Tangent, TourBox, Monogram) — SpliceKit has no hardware layer
- Batch tagging: Titles to Keywords (still the fastest path for interview logging)
- Lua scripting for FCP UI automation (external, AppleScript-based)
- DaVinci Resolve bridge

**Use SpliceKit instead for:**
- Claude Code automation (in-process, 10ms, 200+ tools)
- Real-time timeline editing, transcript editing, scene detection
- Anything programmatic via MCP

CommandPost and SpliceKit are complementary. CommandPost handles hardware interfaces; SpliceKit handles agent-driven editing. They don't overlap.

---

#### Marker Data — separate app
Export FCP markers to:
- Excel
- Notion database
- Airtable
- Screenshots included

FCPXML v1.14 support. Works for tracking edit progress in Notion (cut status by sequence, feedback by character, etc.).

---

#### Recall Toolbox — in Bundle
Shared pasteboard inside FCP. Stores any copyable content with thumbnails. Syncs across devices via iCloud.

Useful if you work across MacBook and Mac Mini on the same project.

---

### Free + Open Source (LateNite and community)

| Tool | What it does | Link |
|------|-------------|------|
| **CommandPost** | FCP automation + control surfaces | commandpost.fcp.cafe |
| **Doza Assist** | Local AI transcription + FCP timeline export | GitHub |
| **Keyframeless** | Caption generation | GitHub |
| **FCP Backup Manager** | Auto-backup .fcpbundle to NAS | GitHub |
| **QuickLook Video** | Finder preview for wide codec support | GitHub |

---

## Color

| Tool | Role | Price |
|------|------|-------|
| **Color Finale 2** | In-timeline grading in FCP | $99 |
| **Dehancer Pro** | Film emulation (grain, halation, Orton) | $79/yr |
| **FilmConvert Nitrate** | Camera-specific film emulation | $99 |
| **CineMatch** | Multi-camera color matching | $89 |
| **Colourlab AI v4** | AI-assisted grading | TBD (Apr 2026) |

---

## Transcription

| Tool | Approach | Price |
|------|----------|-------|
| **MacWhisper** | Parakeet v3, local, best accuracy | $30 one-time |
| **Doza Assist** | Local AI + FCP export | Free |
| **ScriptStar** | Multicam sync + Whisper/Parakeet | Paid |
| **Simon Says** | Cloud service | Paid |

**Rule**: Always MacWhisper + Parakeet for Goldberg. Never OpenAI API.

---

## Media Management / Ingest

| Tool | Role | Price |
|------|------|-------|
| **Kyno** | Ingest, preview, rename, proxy gen | Paid |
| **Lumberjack System** | Real-time logging on set + text editing | Paid |
| **Silverstack Lab** | Professional DIT: checksum, backup, reports | Paid |

---

## Collaboration

| Tool | Role | Price |
|------|------|-------|
| **Frame.io** | Review and approval (native FCP integration) | Adobe CC |
| **PostLab** (Hedge) | FCP library version control + collaboration | Paid |
| **Strada** | Secure file transfer with upload links | Paid |

---

## SpliceKit — Primary programmatic layer

**[github.com/elliotttate/SpliceKit](https://github.com/elliotttate/SpliceKit)** — dylib injected into FCP's process, exposes 78,000+ ObjC classes via JSON-RPC on `127.0.0.1:9876`. Claude Code connects as an MCP client with 200+ tools. No AppleScript, no UI simulation, no roundtrip — direct in-process control.

**What it replaces/complements:**
- CommandPost for automation: SpliceKit is faster and more capable for Claude Code integration (10ms latency vs ~500ms AppleScript)
- fcpxml-mcp-server for Claude Code + FCP workflows: SpliceKit is the primary tool; fcpxml-mcp-server complements for XML-level or multi-NLE work
- Manual FCP ↔ ComfyUI roundtrip: `export_xml()` / `import_fcpxml()` automates the loop

**Core capabilities:**
- `blade_at_times([...])`, `get_timeline_clips()`, `get_transcript()`, `detect_scene_changes()`
- `apply_effect()`, `apply_transition()`, `set_inspector_property()`
- `export_xml()`, `import_fcpxml()` — no dialog, programmatic
- `capture_timeline()`, `capture_viewer()` — GPU screenshots for visual verification
- `open_transcript()`, `delete_transcript_silences()` — text-based editing
- Lua scripting VM (macOS 26.3 disabled, macOS 26.4+ fully functional)

**macOS 26.3 note:** DualTimeline + Lua VM + EffectDrag crash on macOS 26.3 + FCP 12.2 build 447037. Guard patch in [PR #51](https://github.com/elliotttate/SpliceKit/pull/51). All other tools work. Fully stable on macOS 26.4+.

**Status**: Production. In active daily use.

---

## Not recommended / Superseded

| Tool | Why not |
|------|---------|
| **7toX** | FCP 7 → FCP X only. FCP 7 is dead. |
| **X2Pro** | Only useful for FCP → Pro Tools AAF. Narrow use case. |
| **Capacitor** (for Resolve roundtrip) | Does NOT convert FCP↔Resolve. FCPXML version only. |

---

*Last updated: April 2026. Source: fcp.cafe, production use.*
