# 07 - Ecosystem Repos

Open source reference stack for professional cinema production. All repos active as of April 2026.

---

## ASWF — Academy Software Foundation

Hollywood's open source foundation. OpenEXR, OpenColorIO, OpenTimelineIO are maintained here — not at Adobe. Founded by ILM/Lucasfilm/Disney/Microsoft, used at every major studio.

| Repo | Stars | What it is | Use |
|------|-------|------------|-----|
| [AcademySoftwareFoundation/OpenColorIO](https://github.com/AcademySoftwareFoundation/OpenColorIO) | ~2K | Color management framework (OCIO 2.5). ACES 2, LUTs, Rec.709/P3/Rec.2020. | Color pipeline across all tools |
| [AcademySoftwareFoundation/OpenColorIO-Config-ACES](https://github.com/AcademySoftwareFoundation/OpenColorIO-Config-ACES) | — | Official ACES 2.0 config files for OCIO. Ready to drop into Resolve/Nuke/Flame. | ACES config for festival/platform delivery |
| [AcademySoftwareFoundation/openexr](https://github.com/AcademySoftwareFoundation/openexr) | ~3K | OpenEXR spec + reference implementation. The HDR image format for VFX/cinema. | AI generation output, compositing, archival |
| [AcademySoftwareFoundation/OpenTimelineIO](https://github.com/AcademySoftwareFoundation/OpenTimelineIO) | ~2K | Editorial timeline interchange format. Adapters for FCPX, FCP7, CMX EDL, AAF. | NLE automation, timeline round-trip |
| [OpenTimelineIO/otio-fcpx-xml-adapter](https://github.com/OpenTimelineIO/otio-fcpx-xml-adapter) | — | FCPX OTIO adapter | Python-based FCPXML manipulation |

---

## Netflix Open Source

| Repo | Stars | What it is | Use |
|------|-------|------------|-----|
| [Netflix/photon](https://github.com/Netflix/photon) | ~1K | IMF package validator (Java). Validates CPL, AssetMap, PackingList against SMPTE ST 2067. | Validate IMF before platform delivery |
| [Netflix/vmaf](https://github.com/Netflix/vmaf) | ~2.5K | Video Multi-Method Assessment Fusion. Emmy-winning perceptual video quality metric. | QC encode quality before delivery |
| [Netflix/image_compression_comparison](https://github.com/Netflix/image_compression_comparison) | ~1K | Framework for comparing codec quality (VMAF, SSIM). | Evaluate ProRes vs AV1 vs H.265 quality |

---

## Integrity / Hashing — DIT chain of custody

### ASC MHL (industry standard)

| Repo | Stars | What it is | Use |
|------|-------|------------|-----|
| [ascmitc/mhl](https://github.com/ascmitc/mhl) | ~200 | ASC Media Hash List — CLI reference implementation. XML manifest with checksums, file metadata, copy tracking. | DIT backup verification, set to post chain |
| [ascmitc/mhl-specification](https://github.com/ascmitc/mhl-specification) | — | Official MHL specification document | Understanding the format |
| [pomfort/mhl-tool](https://github.com/pomfort/mhl-tool) | ~150 | Pomfort's MHL implementation (seal/verify). Cross-platform. | Alternative to ascmhl CLI |

MHL is the legal-grade standard for cinema media integrity. Silverstack (commercial) uses it. DIT on set, lab on ingest, post on archive — all should generate and verify MHL.

### Hash tools

| Repo | Stars | What it is | Use |
|------|-------|------------|-----|
| [Cyan4973/xxHash](https://github.com/Cyan4973/xxHash) | ~4K | xxHash (XXH32/64/128) — RAM-speed non-cryptographic hash. | Fast checksums for work files and backups |
| [rhash/RHash](https://github.com/rhash/RHash) | ~600 | Multi-format hasher: MD5, SHA-1/256/512, BLAKE3, CRC32. | Full verification when xxHash isn't enough |
| [jessek/hashdeep](https://github.com/jessek/hashdeep) | ~600 | Batch hash audit tool. Recursive audit mode. | Batch verification of large media collections |

**xxHash vs SHA-256 for media**: xxHash is 10-20x faster and sufficient for detecting corruption/transfer errors. SHA-256 is needed if you want cryptographic proof of authenticity (C2PA, NFT, legal chain of custody).

---

## DCP / IMF

| Repo | Stars | What it is | Use |
|------|-------|------------|-----|
| [cth103/dcpomatic](https://github.com/cth103/dcpomatic) | ~600 | DCP-o-matic — full DCP mastering suite. GUI + CLI. Feature-length capable. | Create DCPs for festival theatrical distribution |
| [cth103/libdcp](https://github.com/cth103/libdcp) | — | Library for reading/writing DCPs. Core of DCP-o-matic. | Headless DCP creation |
| [Ymagis/ClairMeta](https://github.com/Ymagis/ClairMeta) | ~250 | Python DCP prober and validator. CPL, signatures, Atmos, SMPTE specs. | QC DCP before sending to cinema |
| [cinecert/asdcplib](https://github.com/cinecert/asdcplib) | ~400 | AS-DCP/AS-02 (IMF essence) read/write. MXF PCM/JPEG2000. | Low-level MXF manipulation for IMF packages |
| [tmeiczin/opendcp](https://github.com/tmeiczin/opendcp) | ~1.3K | OpenDCP — encoder suite: TIFF→J2K, seq→MXF, XML generation. | Full open-source DCP encode pipeline |

---

## Codecs & Archiving

| Repo | Stars | What it is | Use |
|------|-------|------------|-----|
| [FFmpeg/FFmpeg](https://github.com/FFmpeg/FFmpeg) | ~40K | FFmpeg — the foundation of everything. | ProRes, FFV1, EXR, J2K, any format conversion |
| [FFmpeg/FFV1](https://github.com/FFmpeg/FFV1) | ~500 | FFV1 lossless codec spec (IETF RFC 9043). Library of Congress preferred format. | Long-term lossless archival |
| [gitlab.com/AOMediaCodec/SVT-AV1](https://gitlab.com/AOMediaCodec/SVT-AV1) | ~3K+ | SVT-AV1 3.x — best AV1 encoder. Lossless mode in 3.0. | Streaming delivery, lossless archive alternative |
| [kkroening/ffmpeg-python](https://github.com/kkroening/ffmpeg-python) | ~11K | Python bindings for FFmpeg with DAG-based signal graph. | Python batch transcoding, color conversion |

---

## Color science

| Repo | Stars | What it is | Use |
|------|-------|------------|-----|
| [colour-science/colour](https://github.com/colour-science/colour) | ~2.5K | Python color science library. OCIO processor integration, XYZ conversion, spectral analysis. | Programmatic colorimetry |

---

## ComfyUI / AI generation

| Repo | Stars | What it is | Use |
|------|-------|------------|-----|
| [Comfy-Org/ComfyUI](https://github.com/Comfy-Org/ComfyUI) | ~69K | ComfyUI — the core. Node-based diffusion. | Everything |
| [Kosinkadink/ComfyUI-AnimateDiff-Evolved](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved) | ~3K | AnimateDiff with sliding context windows, infinite length, temporal consistency. | Long-shot temporal consistency |
| [MrForExample/ComfyUI-3D-Pack](https://github.com/MrForExample/ComfyUI-3D-Pack) | — | 3D mesh/UV/NeRF nodes. Bridges Blender 3D → ComfyUI. | Blender geometry → ControlNet conditioning |
| [kijai/comfyui-svd-temporal-controlnet](https://github.com/kijai/comfyui-svd-temporal-controlnet) | — | SVD + Temporal ControlNet for optical consistency. | Shot-level temporal stability |

---

## FCP / NLE automation

| Repo | Stars | What it is | Use |
|------|-------|------------|-----|
| [elliotttate/SpliceKit](https://github.com/elliotttate/SpliceKit) | — | **Primary**: ObjC dylib injected into FCP process. 78K+ classes via JSON-RPC. 200+ MCP tools. In-process, no roundtrip. | Claude Code direct FCP control (blade, transcript, export, color, effects) |
| [DareDev256/fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server) | — | **Complement**: 47 MCP tools for Claude Code → FCP via FCPXML roundtrip. 348 automated tests. | Multi-NLE workflows (FCP → DaVinci, FCP → Blender), XML-level batch ops |
| [TheAcharya/pipeline-neo](https://github.com/TheAcharya/pipeline-neo) | — | Swift 6 FCPXML framework. Type-safe parsing, creation, manipulation. v1.5→v1.14 conversion. | Swift-native FCPXML automation |
| [tin2tin/fcpxml_import](https://github.com/tin2tin/fcpxml_import) | — | Blender add-on to import FCPXML into Blender VSE. | FCP timeline → Blender for AI treatment |

**When to use SpliceKit vs fcpxml-mcp-server:**
- Claude Code + FCP only (blade, transcript, color, markers) → **SpliceKit**
- Multi-NLE roundtrip (FCP → DaVinci, FCP → Blender) → **fcpxml-mcp-server + OTIO**
- FCP → ComfyUI → FCP loop → **SpliceKit** (`export_xml` + `import_fcpxml`)

---

## Note on Adobe

Adobe does not open-source its cinema tools (Premiere SDK, Media Encoder, Audition are proprietary). The professional open standards you might associate with "Adobe" — like OpenEXR and OpenColorIO — are actually hosted at the Academy Software Foundation (ASWF) and were created by ILM (Disney/Lucasfilm), not Adobe.

---

## Relevant film/media GitHub orgs

- [github.com/AcademySoftwareFoundation](https://github.com/AcademySoftwareFoundation) — OCIO, OpenEXR, OTIO, OpenVDB
- [github.com/Netflix](https://github.com/Netflix) — VMAF, Photon
- [github.com/ascmitc](https://github.com/ascmitc) — ASC MHL
- [github.com/Comfy-Org](https://github.com/Comfy-Org) — ComfyUI official
- [github.com/FFmpeg](https://github.com/FFmpeg) — FFmpeg, FFV1
- [github.com/pomfort](https://github.com/pomfort) — Silverstack open tools

---

*Last updated: April 2026.*
