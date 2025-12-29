# Changelog

All notable changes to Lumina Photo Analysis System will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2025-12-27

### Added
- **RAW+JPEG Pairing** - Intelligent pairing of RAW and JPEG files with matching filenames
  - Enabled by default (`LUMINA_RAW_SUPPORT=on`)
  - RAW formats supported: NEF, CR2, CR3, ARW, DNG, RAF, ORF, RW2
  - One XMP sidecar created per pair (shared by both RAW and JPEG)
  - JPEG used for AI analysis (faster)
  - Works seamlessly with JPEG-only libraries (no overhead)
  - `LUMINA_RAW_SUPPORT=only` mode for RAW-only workflows
- **Watch Mode** - Continuous monitoring for new photos
  - Enable with `LUMINA_WATCH_MODE=on`
  - Automatically processes new photos as they're added
  - Ideal for tethered shooting or live imports
  - File stability detection (waits for files to finish copying)
  - Pending buffer for staggered RAW+JPEG arrivals (e.g., SD card imports)
- **Model Idle Timeout** - Smart model unloading to save memory
  - Configure with `LUMINA_MODEL_IDLE_TIMEOUT=auto|disabled|<seconds>`
  - Auto mode: unloads models after 300s idle in watch mode
  - Graceful recovery from OOM with exponential backoff retry
- **Watch Mode Auto-Exit** - Exit after inactivity period
  - Configure with `LUMINA_WATCH_IDLE_EXIT=<seconds>`
  - Useful for batch-style processing workflows

### Fixed
- **XMP Keyword Preservation** - User keywords in existing XMP files are now preserved and merged with AI-generated keywords

## [1.0.0] - 2025-12-07

### Added
- Public release on Docker Hub with both cpu and gpu version
- Complete documentation
- Guides for use with photo management tools (github)

## [1.0.0-rc1] - 2025-11-27

### Added
- **Docker Hub Release** - First release candidate on Docker Hub
- **2-Directory Structure** - Simplified volume mounts (photos + cache)
- **GPU Optimization** - Full RTX 5090 support with PyTorch nightly
- **Warmup Tracking** - Accurate ETA calculation excluding model loading time
- **Progress Display** - Real-time progress bars with photo/sec metrics
- **Environment Profiles** - NEW and LEGACY config format support
- **Processor Registry** - Dynamic processor loading based on config
- **Checkpoint System** - Resumable processing for large photo collections
- **XMP Metadata Integration** - On1 PhotoRAW validated compatibility
- **Hierarchical Keywords** - Lightroom-compatible keyword trees
- **Top-K Predictions** - Multiple classification alternatives with confidence scores
- **Flush Logging** - Immediate Docker console output (no buffering)

### Changed
- **Configuration Format** - Switched from LEGACY to NEW format with environment profiles
- **Cache Organization** - All persistent data in single cache directory
- **Model Storage** - Consolidated in cache/models/ subdirectory
- **Logging Architecture** - Environment-aware with FlushingStreamHandler
- **Performance** - 10x GPU speedup (RTX 5090: ~10 photos/sec vs 1 photo/sec CPU)

### Fixed
- **TRANSFORMERS_CACHE** - Removed FutureWarning by using HF_HOME
- **Docker Output Buffering** - Real-time log streaming
- **Progress Tracking** - File logging during progress bar display
- **Environment Config** - Proper profile resolution for logging
- **Model Loading** - Removed hardcoded model_path checks

### Security
- **Non-root User** - Docker images run as lumina (UID 1000)
- **No Network Access** - Container doesn't need network after model download
- **Copyright Headers** - All source files include proprietary notice

## [0.9.0] - 2025-11-20

### Added
- Initial Docker implementation
- RT-DETR object detection
- iNat21 species classification
- XMP sidecar file writing
- Ray distributed processing
- GPU support (CUDA)

### Known Issues
- Docker image size optimization needed
- Configuration complexity
- Multiple volume mounts required

---

## Version Numbering

- **Major.Minor.Patch** - Semantic versioning
- **rc#** - Release candidate (beta testing)
- **beta#** - Beta versions (internal testing)

## Links

- **Docker Hub**: https://hub.docker.com/r/stevenvanassche/lumina
- **GitHub**: https://github.com/stevenvanassche/Lumina
- **Issues**: https://github.com/stevenvanassche/Lumina/issues

---

**Note**: This is proprietary software. See LICENSE file for usage terms.
