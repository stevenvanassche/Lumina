# Lumina - AI Photo Metadata Pipeline

> **Status**: Version 1.1.0 - Production Release

AI-powered photo analysis pipeline that extracts EXIF metadata, detects objects, classifies species, and writes results to XMP sidecar files compatible with Adobe Lightroom Classic, On1 PhotoRAW, Immich and other photo management software.

**Docker Hub**: https://hub.docker.com/r/stevenvanassche/lumina <br>
**GitHub**: https://github.com/stevenvanassche/Lumina

## Features

- 🔍 **EXIF Extraction** - Camera metadata, GPS coordinates, exposure settings
- 🎯 **Object Detection** - RT-DETR detection (COCO + Objects365 dataset)
- 🦜 **Species Classification** - iNaturalist 21 model (10,000 species)
- 📝 **XMP Sidecar Files** - Non-destructive metadata storage
- ⚡ **GPU Acceleration** - 10x faster processing with NVIDIA GPU
- 🔄 **Resumable Processing** - Checkpoint-based pipeline for large collections
- 🐳 **Easy Deployment** - Single docker-compose command to start
- 👁️ **Watch Mode** *(New in v1.1)* - Continuous monitoring for new photos
- 📷 **RAW+JPEG Pairing** *(New in v1.1)* - Smart pairing of RAW and JPEG files

## Quick Start

📖 **[Installation Guide](docs/INSTALL.md)** - Complete setup instructions with platform-specific details

**TL;DR for experienced users:**

```bash
# Create project directory
mkdir -p lumina-docker/{photos,cache} && cd lumina-docker

# Download docker-compose.yml
curl -o docker-compose.yml https://raw.githubusercontent.com/stevenvanassche/Lumina/master/docker/docker-compose.yml

# Create basic .env file
cat > .env << EOF
PHOTOS_PATH=./photos
CACHE_PATH=./cache
EOF

# Add your photos to the photos/ directory, then run:
docker compose --profile cpu up    # CPU mode (~1 photo/sec)
docker compose --profile gpu up    # GPU mode (~10 photos/sec, requires NVIDIA GPU + WSL2 on Windows)

# NEW in v1.1: Watch mode - continuously monitor for new photos
LUMINA_WATCH_MODE=on docker compose --profile cpu up
```

**First Run:** Models download automatically (~2.5GB, 5-15 minutes)

**⚠️ Windows GPU Users:** WSL2 is **required** for GPU acceleration. See the [Installation Guide](docs/INSTALL.md#windows-gpu-setup) for setup instructions.

## Example Output

### XMP Sidecar File
```xml
<dc:subject>
  <rdf:Seq>
    <rdf:li>COCO Detection</rdf:li>
    <rdf:li>bird</rdf:li>
    <rdf:li>Inat21</rdf:li>
    <rdf:li>European Starling</rdf:li>
  </rdf:Seq>
</dc:subject>

<lr:hierarchicalSubject>
  <rdf:Seq>
    <rdf:li>COCO Detection|bird</rdf:li>
    <rdf:li>Inat21|bird|European Starling</rdf:li>
  </rdf:Seq>
</lr:hierarchicalSubject>
```

### Compatible Software

✅ **Validated:**
- **On1 PhotoRAW** - Full hierarchical keyword support (JPEG files with or without RAW files)
- **Immich** - External Libraries with XMP sidecar import (JPEG files)
- **Adobe Lightroom Classic** - Full hierarchical keyword support (RAW files required)

✅ **Expected:**
- Capture One
- Photo Mechanic
- digiKam
- ACDSee

📖 **Integration Guides:** See [Installation Guide](docs/INSTALL.md#using-results-in-photo-management-software) for detailed setup instructions for each platform.

## Requirements

### All Platforms
- **Docker & Docker Compose** (version 20.10 or newer)
- **Disk Space:** ~2.5GB for AI models + storage for photos

### CPU Mode
- **RAM:** 16GB+ recommended
- **Performance:** ~1 photo/second

### GPU Mode (Optional - 10x faster)
- **GPU:** NVIDIA GPU with 6GB+ VRAM (RTX series recommended)
- **Drivers:** NVIDIA drivers version 525.60 or newer
- **Platform-Specific:**
  - **Linux:** NVIDIA Container Toolkit required
  - **Windows:** WSL2 required (automatic GPU passthrough via Docker Desktop)
  - **macOS:** Not supported (Apple Silicon/AMD GPUs incompatible with CUDA)
- **Performance:** ~10 photos/second (RTX 5090)

📖 **Detailed Requirements:** See [Installation Guide](docs/INSTALL.md#prerequisites) for platform-specific setup instructions

## Configuration

Basic configuration via `.env` file:

| Variable | Description |
|----------|-------------|
| `PHOTOS_PATH` | Directory containing your photos (required) |
| `CACHE_PATH` | Directory for models, checkpoints, and logs (required) |
| `LUMINA_WORKERS` | Number of parallel workers (default: 2 CPU, 1 GPU) |
| `LUMINA_BATCH_SIZE` | Photos per batch (default: 5 CPU, 10 GPU) |
| `LUMINA_LOG_LEVEL` | Logging level: DEBUG, INFO, WARNING, ERROR (default: INFO) |
| `LUMINA_EXECUTION_MODE` | Processing mode: quick_scan/normal_scan/force_update (default: normal_scan) |
| `LUMINA_WATCH_MODE` | Watch mode: on/off (default: off) *(New in v1.1)* |
| `LUMINA_MODEL_IDLE_TIMEOUT` | Model unload timeout: auto/disabled/<seconds> (default: auto) *(New in v1.1)* |
| `LUMINA_WATCH_IDLE_EXIT` | Auto-exit after N seconds idle, 0=disabled (default: 0) *(New in v1.1)* |
| `LUMINA_RAW_SUPPORT` | RAW file handling: on/off/only (default: on) *(New in v1.1)* |

**Example `.env` for CPU mode:**
```bash
PHOTOS_PATH=./photos
CACHE_PATH=./cache
LUMINA_WORKERS=2
LUMINA_BATCH_SIZE=5
```

**Example `.env` for GPU mode:**
```bash
PHOTOS_PATH=./photos
CACHE_PATH=./cache
LUMINA_WORKERS=1
LUMINA_BATCH_SIZE=10
LUMINA_PARALLEL_BATCHES=3
```

**Example `.env` for Watch mode (continuous monitoring):**
```bash
PHOTOS_PATH=./photos
CACHE_PATH=./cache
LUMINA_WATCH_MODE=on
LUMINA_MODEL_IDLE_TIMEOUT=auto   # auto = unload after 300s idle in watch mode
```

**Example `.env` to disable RAW support (JPEG-only mode):**
```bash
PHOTOS_PATH=./photos
CACHE_PATH=./cache
LUMINA_RAW_SUPPORT=off   # Disable RAW+JPEG pairing (default is on)
```

📖 **Complete Configuration Reference:** See [Installation Guide](docs/INSTALL.md#configuration-reference) for all environment variables, performance tuning, and advanced options.

## Performance Benchmarks

Tested on 1627 photos with EXIF → RT-DETR → iNat21 → XMP Writer pipeline.

### CPU Mode
- **Throughput**: 1.14 photos/sec
- **Time**: ~23.7 minutes for 1627 photos

### GPU Mode (RTX 5090)
- **Throughput**: 10.74 photos/sec
- **Time**: ~2.5 minutes for 1627 photos
- **Speedup**: **10x faster than CPU**

## Supported Image Formats

**Standard formats:**
- JPEG (`.jpg`, `.jpeg`)
- PNG (`.png`)

**RAW formats (enabled by default in v1.1):**
- Nikon (`.nef`)
- Canon (`.cr2`, `.cr3`)
- Sony (`.arw`)
- Adobe (`.dng`)
- Fujifilm (`.raf`)
- Olympus (`.orf`)
- Panasonic (`.rw2`)

**RAW+JPEG Pairing:** When enabled, Lumina intelligently pairs RAW and JPEG files with the same base filename (e.g., `IMG_001.jpg` + `IMG_001.nef`). The JPEG is used for faster AI analysis, and one XMP sidecar is created for the pair.

**⚠️ RAW Format Limitations:** Lumina uses [LibRaw](https://www.libraw.org/supported-cameras) for RAW processing. Some newer formats like Nikon Z9 High Efficiency (HE/HE*) are not yet supported. For unsupported formats, shoot RAW+JPEG and enable pairing mode.

**Note:** XMP sidecar files are only created when objects or species are detected in the photo. Photos without any detections will not have an XMP file generated.

**⚠️ Existing XMP Files:** Lumina will merge AI-generated keywords with existing XMP metadata. This feature is still being tested - backup your XMP files before processing photos with existing metadata.

**⚠️ JSON Format:** JSON metadata files are provided for advanced integrations, but the structure may change in future versions. Build integrations with schema flexibility in mind.

## License

**This software is proprietary.** The Docker images are provided for evaluation and non-commercial use.

### Usage Rights

| Use Case | Allowed | Requirements |
|----------|---------|--------------|
| Personal use | ✅ Yes | Free |
| Educational/Research | ✅ Yes | Free |
| Commercial use | ❌ No | Requires commercial license |

### Important License Notes

**iNaturalist 21 Model Restriction:**
- The iNat21 species classification model is licensed under **CC BY-NC 4.0** (Non-Commercial)
- This restricts the **entire pipeline** to non-commercial use only
- Commercial users must disable iNat21 and use alternative classification models

**RT-DETR Object Detection:**
- Licensed under **Apache 2.0** (commercial-friendly)
- Can be used commercially when iNat21 is disabled

### For Commercial Use

Commercial licensing is available for organizations wanting to use Lumina in production.

**Contact:** info@lumina.photo

See [LICENSE](LICENSE) for complete terms.

## Support

- **Documentation**: [Installation Guide](docs/INSTALL.md) | [On1 PhotoRAW Integration](integrations/ON1-PHOTORAW.md) | [Immich Integration](integrations/IMMICH.md) | [Lightroom Integration](integrations/LIGHTROOM.md)
- **Issues**: https://github.com/stevenvanassche/Lumina/issues
- **Discussions**: https://github.com/stevenvanassche/Lumina/discussions
- **Docker Hub**: https://hub.docker.com/r/stevenvanassche/lumina

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

## Credits

This project uses the following AI models:

- **RT-DETR** - Real-Time Detection Transformer (Apache 2.0)
  - Model: PekingU/rtdetr_r50vd_coco_o365
  - Source: https://github.com/lyuwenyu/RT-DETR

- **iNat21** - EVA-02 model fine-tuned on iNaturalist 2021 (CC BY-NC 4.0)
  - Model: timm/eva02_large_patch14_clip_336.merged2b_ft_inat21
  - Source: https://huggingface.co/timm/eva02_large_patch14_clip_336.merged2b_ft_inat21
  - Trainer: Ross Wightman (timm)
  - Dataset: [iNaturalist 2021 competition](https://github.com/visipedia/inat_comp/tree/master/2021)

Built with:
- PyTorch (BSD-3)
- Ray (Apache 2.0) - Distributed computing
- Transformers (Apache 2.0) - HuggingFace
- timm (Apache 2.0) - PyTorch Image Models

---

**Built with ❤️ for wildlife photographers and photo enthusiasts**
