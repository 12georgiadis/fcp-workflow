# 06 - AV1, Archiving & IMF Delivery

Formats, codecs, color spaces, and delivery specs. April 2026.

---

## Archival formats

### Codec decision tree

```
Camera originals         → keep in native format (BRAW, MLV, .mov ProRes)
Intermediate / roundtrip → ProRes 422 HQ or ProRes 4444
AI-generated sequences   → EXR 32-bit float (master) → ProRes 4444 (delivery)
Long-term archive        → FFV1 (proven) or AV1 lossless via SVT-AV1 3.0
Streaming delivery       → AV1 lossy (Netflix, web) or H.265
Festival / broadcast     → ProRes 4444 XQ (archive master), ProRes 422 HQ (broadcast)
DCP                      → JPEG 2000 (J2K) via DCP-o-matic
IMF                      → JPEG 2000 or HEVC (Netflix uses both)
```

### FFV1 vs AV1 lossless

| | FFV1 | AV1 lossless (SVT-AV1 3.0) |
|--|------|---------------------------|
| Compression | Baseline | ~30% better than FFV1 |
| Speed | Fast | Slower to encode |
| Compatibility | Wide (ffmpeg) | Growing |
| Use | Proven archival standard | New projects |
| Container | MKV or MOV | MKV |

```bash
# FFV1 archival (proven)
ffmpeg -i input.mov -c:v ffv1 -level 3 -g 1 -slices 16 -slicecrc 1 \
  -c:a pcm_s24le output.mkv

# AV1 lossless (SVT-AV1 3.0 — new default)
ffmpeg -i input.mov -c:v libsvtav1 -crf 0 -preset 6 \
  -c:a pcm_s24le output.mkv
```

---

## Bit depth and color precision

### 8-bit vs 10-bit vs 12-bit vs 16-bit

| Bit depth | Values per channel | Use case |
|-----------|-------------------|----------|
| 8-bit | 256 | Web delivery, proxies |
| 10-bit | 1024 | Broadcast standard, FCP/Resolve timeline |
| 12-bit | 4096 | BRAW native, grading |
| 16-bit float | 65536+ | AI generation output, EXR, compositing |
| 32-bit float | ~4 billion | EXR master, Blender render |

### BRAW bit depth
Blackmagic RAW records at 12-bit. When imported into FCP via BRAW Toolbox, it's decoded to 10-bit for the FCP timeline (FCP uses a 10-bit pipeline internally with YCbCr 4:2:2 or 4:4:4 depending on project settings).

### AI generation — critical: bit depth matters

**Default ComfyUI output: 8-bit PNG or 8-bit JPEG.**

8-bit baked into a 10-bit timeline = banding and quality loss in color grade.

**Force 16-bit float output from ComfyUI:**
```python
# In ComfyUI SaveImage node: use "Save EXR" node instead
# EXR 32-bit float = zero quality loss, full grading range
# Then convert to ProRes 4444 for NLE import:
ffmpeg -i generated_frame_%04d.exr -c:v prores_ks -profile:v 4 \
  -pix_fmt yuv444p10le output.mov
```

**Format pipeline for AI-generated sequences:**
```
ComfyUI render → EXR 32-bit float (frame sequence)
                → ffmpeg → ProRes 4444 10-bit (for DaVinci/FCP)
                         → ffmpeg → ProRes 422 HQ (for offline edit)
```

**Never** import 8-bit H.264 or 8-bit PNG sequences from ComfyUI directly into a grading timeline. You lose headroom for color correction.

### FCP project settings for mixed BRAW + AI workflow
- Project resolution: 4K (3840×2160) or native
- Color space: **Rec. 709** (for standard) or **Rec. 2020** (for HDR)
- Format: Apple ProRes 422 (for offline) or 4444 (for finishing)
- Frame rate: match camera (usually 24, 25, or 29.97)

---

## Color space pipeline

### BRAW color science
BRAW files carry their own color science metadata. Two options in BRAW Toolbox / Resolve:
- **Blackmagic Design Film Gen 5** → wide gamut log, designed for grading
- **Blackmagic Design Gen 5** → standard look, more natural out of box

For grading workflow: **BMD Film Gen 5** → grade in Resolve → convert to Rec.709 or P3 for delivery.

### Color spaces by destination

| Destination | Color space | Transfer function | Gamut |
|-------------|------------|-------------------|-------|
| Web / Vimeo / YouTube | Rec.709 | SDR (Gamma 2.4) | sRGB |
| Broadcast France | Rec.709 | SDR | sRGB |
| DCP | P3-D65 | Gamma 2.6 | DCI-P3 |
| Netflix SDR | Rec.709 | SDR | sRGB |
| Netflix HDR | Rec.2020 | PQ (ST.2084) | Rec.2020 |
| Apple TV+ | Rec.2020 | Dolby Vision P8 | P3 D65 |

### Recommended grading pipeline (Goldberg)
```
BRAW source (12-bit BMD Film Gen 5)
  → DaVinci Resolve: Color Managed project (ACES or Resolve Color Science)
  → Input: Blackmagic Design Film Gen 5
  → Timeline: Rec.709 scene-linear or DaVinci Wide Gamut
  → Output: Rec.709 (Gamma 2.4) for primary delivery
  → Export: ProRes 4444 (for archival), H.265 (for streaming)
```

### AI-generated clips in color pipeline
AI sequences are RGB. They have no embedded color science.

**Treat them as Rec.709 SDR unless you specifically generated in a wider gamut.**

In DaVinci Resolve: right-click AI clip → Input Color Space → Rec.709 SDR. Then grade normally.

If generating in FLUX or Wan2.2 and targeting HDR: use a tone mapping LUT to bring into Rec.2020 PQ. This is an advanced workflow with few established tools.

---

## HDR delivery

### HLG vs PQ

| | HLG (Hybrid Log-Gamma) | PQ (ST.2084) |
|--|----------------------|--------------|
| Standard | ITU-R BT.2100 | SMPTE ST.2084 |
| Who uses it | Broadcast (BBC, NHK) | Netflix, Amazon, Apple TV+ |
| Backward compat | Yes (looks OK on SDR screens) | No (looks blown out on SDR) |
| Metadata | None required | MaxCLL, MaxFALL |

For festival cinema or art distribution: **SDR Rec.709 is fine.** HDR matters for streaming platform delivery.

---

## IMF (Interoperable Master Format)

### What it is
IMF is the professional delivery format for Netflix, Amazon, Apple TV+, and other major platforms. It's a self-contained package with:
- Video essence (J2K or HEVC)
- Audio essence (PCM stems)
- Subtitles (IMSC1)
- Composition Playlist (CPL) — the timeline
- Output Profile List (OPL) — delivery specs
- Asset Map — inventory

### When you need it
- Delivering to Netflix, Amazon, Apple TV+, Disney+
- Not needed for: festivals (DCP), broadcast (ProRes), web (H.265)

### Tools

| Tool | Use | License |
|------|-----|---------|
| **IMFTool** | Create and edit IMF packages, CPL editing, versioning | Open source |
| **Netflix/photon** | Validate IMF packages against Netflix specs | Open source (Java) |
| **Clipster** | Professional IMF mastering | Commercial |
| **EasyDCP+** | IMF + DCP creation | Commercial |

```bash
# Validate IMF package with Netflix photon
java -jar photon.jar -m /path/to/imf_package/

# Check against Netflix delivery spec
java -jar photon.jar -m /path/to/imf_package/ -d /path/to/netflix_app.xml
```

### IMF workflow from DaVinci Resolve
1. Resolve: finish grade, export ProRes 4444 master + PCM stems
2. IMFTool: create new IMF package, import master, create CPL
3. Add audio tracks (5.1, Atmos, M&E, textless)
4. Add subtitles (IMSC1 from .srt via subtitle conversion tool)
5. Validate with photon
6. Deliver package (typically Aspera or Signiant for large files)

### IMF versioning
One of IMF's key advantages: you can create multiple CPLs (versions) from the same essence.

Example for Goldberg:
- CPL_01: French festival version (FR subtitles)
- CPL_02: International version (EN subtitles)
- CPL_03: Version without music (M&E)
- All share the same video and audio essence files = no duplication

---

## DCP (Digital Cinema Package)

For theatrical festival distribution.

### Tools
- **DCP-o-matic** — free, open source, production-quality
- **Wraptor** — commercial alternative

### Specs
```
Video:  JPEG 2000, 24/25fps, 2048×1080 (2K Flat) or 4096×1716 (4K Scope)
Audio:  PCM 24-bit 48kHz, up to 16 channels
Encryption: optional (required for commercial theatrical)
```

```bash
# DCP-o-matic workflow
# 1. Export ProRes 4444 from FCP/Resolve at correct resolution
# 2. Open DCP-o-matic
# 3. Create new DCP, import ProRes
# 4. Set cinema audio (5.1 or stereo)
# 5. Add subtitles as XML
# 6. Build DCP (takes hours for feature)
```

---

## Summary: format pipeline per output

```
CAMERA ORIGINALS
  BRAW 12-bit → keep as-is on NavTGV3/4 + R2

OFFLINE EDIT
  BRAW via BRAW Toolbox → FCP timeline (10-bit ProRes 422)

AI GENERATION
  ComfyUI render → EXR 32-bit → ProRes 4444 10-bit → FCP/Resolve

GRADING
  ProRes 422 HQ → DaVinci Resolve (10-bit or 12-bit float)

ARCHIVE MASTER
  ProRes 4444 XQ (all footage graded)
  OR FFV1/AV1 lossless MKV (long-term archive)

DELIVERY
  Festival theatrical:   DCP (J2K) via DCP-o-matic
  Broadcast (France):    ProRes 422 HQ 1080p, -23 LUFS
  Web / Vimeo:           H.265 4K, -14 LUFS, Rec.709
  Major streaming:       IMF (J2K or HEVC) + Atmos stems
  Art installation:      ProRes 4444 (loop-safe, no compression artifacts)
```

---

*Last updated: April 2026.*
