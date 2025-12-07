# Lumina Roadmap

This document outlines the planned features and improvements for Lumina. The roadmap is subject to change based on community feedback and priorities.

**Current Version**: 1.0.0
**Last Updated**: 2025-12-07

---

## 🎯 Feedback & Feature Requests

We want to hear from you! Help us prioritize features by:

- **GitHub Discussions**: Share ideas and vote on features at [Lumina Discussions](https://github.com/stevenvanassche/Lumina/discussions)
- **GitHub Issues**: Report bugs or request features at [Lumina Issues](https://github.com/stevenvanassche/Lumina/issues)
- **Feature Voting**: Comment with 👍 on features you want most

--- 

## 🎯 v1.1 - Short-term Quality of Life Improvements

**Focus**: Quick wins and user experience enhancements based on early feedback

**Target**: Q1 2026

### Workflow Enhancements

- [ ] **Watch Mode / Auto-processing**
  - Monitor photos directory for new files
  - Automatically process new photos as they arrive
  - Ideal for tethered shooting or live imports
  - Configurable file system watch with debouncing
  - Enable with `LUMINA_WATCH_MODE=true`

- [ ] **Selective Processor Control**
  - Enable/disable individual processors (EXIF, RT-DETR, iNat21, XMP)
  - Skip species classification for non-wildlife photos
  - Skip object detection for faster species-only workflows
  - Configuration: `LUMINA_PROCESSORS=exif,rtdetr,inat21,xmp`

- [ ] **Dry Run Mode**
  - Preview what will be processed without writing files
  - Validate configuration and photo access
  - Estimate processing time based on photo count
  - Enable with `LUMINA_DRY_RUN=true`

### Output Improvements

- [ ] **Custom Keyword Prefixes**
  - Configurable prefix for hierarchical keywords
  - Example: `Wildlife|bird|European Starling` instead of `Inat21|bird|European Starling`
  - Better integration with existing photo library taxonomies
  - Configuration: `INAT21_KEYWORD_PREFIX=Wildlife`

### Developer Experience

- [ ] **Improved Logging**
  - Structured logging with timestamps
  - Per-processor log levels
  - Progress indicators with percentage complete
  - Error summaries at end of run

- [ ] **Health Checks**
  - Docker health check endpoint
  - GPU availability and VRAM monitoring
  - Model loading status
  - Processing queue status

- [ ] **Performance Metrics**
  - Export processing statistics (photos/sec, time per stage)
  - CSV export for benchmarking
  - Memory and GPU utilization tracking
  - Identify bottlenecks in pipeline

---

## 🚀 v1.x - Performance & Usability

**Focus**: Performance optimization, better configuration control, and quality-of-life improvements

### Pipeline Enhancements

- [ ] **Fine-grained Pipeline Configuration**  
  - Custom pipeline profiles for different use cases (wildlife, events, landscapes)
  - User guide for fine-grained pipeline configuration
  
- [ ] **Performance Optimizations**
  - Custom pipeline profiles for different performance settings (run as background task, execute with maximum performance, etc.)
  - Adaptive batch sizing and worker configuration based on available memory
  - Memory-efficient model loading and unloading (load on demand)

### Metadata Improvements

- [ ] **iNat21 Enhancements**
  - Scientific species names (e.g., *Sturnus vulgaris* alongside "European Starling")
  - Taxonomic hierarchy (Kingdom → Phylum → Class → Order → Family → Genus → Species)
  - Common names in multiple languages (configurable)

- [ ] **XMP Embedding in JPEG**
  - Embed XMP metadata directly into JPEG files (not just sidecars)
  - Seamless Lightroom Classic import without RAW files
  - ExifTool integration for reliable embedding
  - Configurable: sidecar-only, embedded-only, or both

### User Experience

- [ ] **Configuration Validation**
  - Pre-flight checks for config syntax errors
  - Resource estimation (RAM, VRAM, disk space) before processing
  - Warnings for suboptimal configurations

- [ ] **Progress & Monitoring**
  - Real-time progress dashboard (web UI or terminal)
  - Per-processor statistics and bottleneck detection
  - ETA calculations with historical performance data

---

## 🔬 - Advanced AI Models

**Focus**: Expanding AI capabilities with state-of-the-art models

### New AI Processors

- [ ] **BioCLIP for Wildlife**
  - Specialized CLIP variant trained on biological imagery
  - Better species recognition than general-purpose models
  - Zero-shot classification for rare/undocumented species
  - Integration with iNat21 for consensus predictions

- [ ] **General CLIP**
  - OpenAI CLIP or OpenCLIP for general image understanding
  - Natural language image search ("photos with sunset over mountains")
  - Scene classification (landscape, portrait, macro, etc.)
  - Semantic similarity for image grouping

- [ ] **Automated Image Captioning**
  - Generate natural language descriptions of photos
  - BLIP-2, LLaVA, or similar vision-language models
  - Multi-sentence captions with context
  - Customizable caption style (technical, artistic, concise)

- [ ] **Face Detection & Recognition**
  - Face detection with bounding boxes
  - Face recognition for people tagging
  - Privacy-preserving local processing (no cloud services)
  - Integration with Immich face recognition

### Model Improvements

- [ ] **Multi-Model Ensemble**
  - Combine predictions from multiple models for higher accuracy
  - Confidence-weighted voting
  - Configurable consensus thresholds

- [ ] **Custom Model Support**
  - Plugin architecture for user-provided models
  - Fine-tuned models for specific domains (birds of Europe, marine life, etc.)
  - Model marketplace or sharing community

---

## 📸 Photo Culling & Selection

**Focus**: AI-powered photo review and selection workflows to dramatically reduce post-shoot processing time

### Intelligent Photo Analysis

- [ ] **Burst Detection & Analysis**
  - Detect photo bursts (rapid sequences of similar shots)
  - Group burst sequences automatically
  - Identify best frame in burst (sharpness, composition, subject expression)
  - Smart burst compression (keep best 1-3, mark rest for deletion)

- [ ] **Photo Quality Scoring**
  - Technical quality metrics:
    - Sharpness/focus quality (edge detection, contrast analysis)
    - Exposure quality (histogram analysis, clipping detection)
    - Noise levels (ISO performance assessment)
  - Aesthetic quality metrics:
    - Composition analysis (rule of thirds, leading lines, symmetry)
    - Color harmony and dynamic range
    - Subject prominence and background clutter
  - Overall quality score (0-100) for quick filtering

- [ ] **Similar Photo Detection & Grouping**
  - Perceptual hashing for near-duplicate detection
  - Subject similarity (same animal/person in different poses)
  - Scene similarity (same location, different angles)
  - Smart grouping with best representative selection
  - Semantic diversity scoring (prioritize variety over repetition)

### Smart Navigation & Review

- [ ] **Intelligent Photo Summarization**
  - Auto-select best N photos from a shoot (e.g., top 50 from 500)
  - Diversity optimization (avoid repetitive shots)
  - Key moment detection (peak action, emotional expressions)
  - Story arc generation (beginning, middle, end of event)

- [ ] **Priority-Based Review Workflow**
  - High-priority queue (potential keepers based on quality scores)
  - Medium-priority queue (review if time permits)
  - Low-priority queue (likely deletes, quick confirmation)
  - AI-suggested deletions (blurry, duplicates, closed eyes)

- [ ] **Advanced Search & Filtering**
  - Natural language search ("sharp photos of birds in flight")
  - Multi-criteria filtering (quality > 80 AND sharpness > 90)
  - Semantic search via CLIP ("sunset with warm tones")
  - Face expression filtering (smiling, serious, eyes open/closed)

### Workflow Integration

- [ ] **Culling UI/Dashboard**
  - Fast keyboard-driven review interface
  - Side-by-side burst comparison
  - Quick star rating based on AI suggestions
  - One-click "keep best, delete rest" per burst
  - Progress tracking (reviewed vs remaining)

- [ ] **RAW Import Assistant**
  - Analyze RAW files during import
  - Pre-tag photos by quality tier (5-star, 4-star, etc.)
  - Auto-reject obvious failures (out of focus, severe overexposure)
  - Smart preview generation (best shots first)

- [ ] **Export Recommendations**
  - Suggest which photos to edit/export
  - Flag photos needing manual review
  - Create "best of" collections automatically

### Use Cases

**Wildlife Photography:**
- Import 2000 RAW files from safari
- AI detects bursts → groups 150 sequences
- Quality scoring → flags top 200 photos
- Species detection → groups by animal type
- Semantic diversity → ensures variety in final selection
- **Result:** 2 hours of culling reduced to 20 minutes

**Event Photography:**
- Import 1500 wedding photos
- Face detection → groups by person/couple
- Burst analysis → selects best smiles, open eyes
- Moment detection → flags key moments (kiss, cake cutting)
- Quality scoring → removes blurry/poorly exposed shots
- **Result:** Deliver 300 best photos instead of overwhelming client

**Landscape Photography:**
- Import 500 bracketed sequences
- Bracket detection → groups HDR sets
- Best exposure selection per bracket
- Composition analysis → flags best angles
- Diversity scoring → avoids repetitive compositions
- **Result:** Quick identification of portfolio-worthy shots

---

## 🌐 - Enterprise & Integration

**Focus**: Production-grade features, integrations, and commercial offerings

### Integrations

#### Immich Deep Integration

- [ ] **API-Based Metadata Updates directly into Immich** 
  - Direct Immich API integration (no XMP sidecars needed)
  - Real-time metadata sync
  - Bidirectional synchronization (Immich ↔ Lumina)
  - Bulk updates for large libraries

- [ ] **Immich ML Pipeline Integration**
  - Lumina as an addition to Immich ML (or a drop-in replacement)
  - Unified processing workflow
  - Shared model cache and resources
  - Native Immich UI for Lumina settings

#### Other Photo Management Tools

- [ ] **Lightroom Plugin** 
  - Native Lightroom Classic plugin for one-click processing
  - Direct catalog integration
  - Publish services for automated workflows

- [ ] **DigiKam Integration**
  - DigiKam-native tag format (`digiKam:TagsList`)
  - Batch processing via DigiKam UI

- [ ] **Adobe Bridge / Camera Raw**
  - XMP compatibility testing and optimization

### Persistence & Scalability

- [ ] **MongoDB Backend**
  - Add MongoDB persistence for improved querying of full metadata set
  - Replace JSON files with MongoDB for better performance
  - Full-text search across metadata
  - Historical metadata versions (audit trail)
  - Multi-user support with access control

- [ ] **PostgreSQL Option**
  - Alternative to MongoDB for SQL-based workflows
  - Direct Immich database integration

- [ ] **Cloud Storage Support**
  - S3-compatible object storage (MinIO, AWS S3, Backblaze B2)
  - Process photos directly from cloud without local copies
  - Distributed processing across multiple nodes

### File Format Support

- [ ] **RAW Image Support**
  - Nikon NEF, Canon CR2/CR3, Sony ARW, Adobe DNG
  - RAW thumbnail extraction for fast preview
  - Preserve RAW metadata during processing
  - Test with dcraw, LibRaw, or rawpy

- [ ] **Video Analysis**
  - Frame extraction (keyframes or regular intervals)
  - Apply AI models to video frames
  - Scene detection and shot boundary detection
  - Video metadata (resolution, codec, duration)
  - Generate video thumbnails with AI-detected highlights

- [ ] **Multi-Page Documents**
  - TIFF, PDF with multiple pages
  - Per-page metadata

### Multi-GPU & Distributed Processing

- [ ] **Multi-GPU Support (NVIDIA)**
  - Utilize multiple NVIDIA GPUs on a single machine
  - Automatic load balancing across GPUs
  - Model parallelism for large models
  - Mixed GPU configurations (e.g., RTX 3090 + RTX 4090)

- [ ] **Apple Silicon Support**
  - Metal Performance Shaders (MPS) backend for M1/M2/M3/M4 chips
  - Unified memory optimization (CPU+GPU shared memory)
  - Neural Engine (ANE) acceleration where applicable
  - Native macOS Docker support or standalone app

- [ ] **AMD GPU Support**
  - ROCm backend for AMD Radeon GPUs
  - Support for Radeon RX 6000/7000 series
  - MI-series accelerators for enterprise users
  - Linux-first, with Windows support if feasible

- [ ] **Intel GPU Support**
  - Intel Arc discrete GPUs (A-series)
  - Integrated Intel Iris Xe graphics
  - oneAPI / Intel Extension for PyTorch
  - Useful for budget-conscious users

- [ ] **NPU / AI Accelerator Support**
  - Intel NPU (Neural Processing Unit) on recent laptops
  - Qualcomm Hexagon NPUs
  - Dedicated AI chips (Google Coral, Hailo, etc.)
  - Power-efficient inference on edge devices

- [ ] **Heterogeneous Computing**
  - Combine different accelerators (e.g., NVIDIA GPU + Apple Silicon)
  - Automatic workload distribution based on hardware capabilities
  - Fallback strategies when specific hardware unavailable

- [ ] **Distributed Processing** 
  - Ray cluster with multiple nodes
  - Head node + worker nodes architecture
  - Network-attached storage (NAS) support
  - Horizontal scaling for large photo libraries (100k+ photos)

- [ ] **Cloud GPU Support**
  - AWS, GCP, Azure GPU instances
  - Spot instance support for cost optimization
  - Auto-scaling based on queue depth

### Commercial Solution

- [ ] **Lumina Pro for local / on-premise pipelines**
  - On-premise self-hosted solution
  - Multi-GPU and distributed multi-server support
  - Premium AI models (higher accuracy, faster processing)
  - Priority support
  - DAM and team features (shared libraries, collaboration)
  - Usage-based pricing or subscription tiers

- [ ] **Lumina Pro in the cloud**
  - Hosted SaaS offering (no self-hosting required)
  - Premium AI models (higher accuracy, faster processing)
  - Priority support and dedicated infrastructure
  - Team features (shared libraries, collaboration)
  - Usage-based pricing or subscription tiers

- [ ] **Enterprise Edition**
  - On-premise deployment with support contracts
  - Custom model training for specific domains
  - SLA guarantees and dedicated support
  - Multi-tenancy and advanced access control



---

## 💡 Future Considerations (No ETA)

**Ideas for exploration - may or may not be implemented**

### Advanced AI Features

- [ ] **Generative AI**
  - Image upscaling / super-resolution
  - Noise reduction and deblurring
  - Style transfer and artistic filters
  - Background removal / subject isolation

- [ ] **Anomaly Detection**
  - Detect blurry, overexposed, or low-quality photos
  - Duplicate detection (near-duplicate via perceptual hashing)
  - Photo quality scoring

- [ ] **Temporal Analysis**
  - Track subjects across photos (e.g., wildlife individual identification)
  - Photo series detection (burst mode, bracketing)
  - Time-lapse assembly and analysis

### Metadata & Interoperability

- [ ] **IPTC Extension Support**
  - Full IPTC Extension Schema compliance
  - Artwork or Object in the Image
  - Image Region (for face/object bounding boxes)

- [ ] **Linked Data / Semantic Web**
  - RDF/OWL ontologies for biodiversity data
  - Integration with Darwin Core standard
  - Export to biodiversity databases (GBIF, iNaturalist)

- [ ] **Blockchain Provenance**
  - Immutable metadata trail
  - Copyright and attribution tracking
  - Content authenticity verification (C2PA standard)

### User Interface

- [ ] **Web Dashboard**
  - Real-time processing status
  - Interactive configuration editor
  - Model performance metrics and visualizations
  - Photo browser with metadata overlay

- [ ] **Mobile App**
  - Trigger processing remotely
  - View results and metadata on mobile
  - Push notifications for completed jobs

- [ ] **Desktop Application** 
  - Native GUI for Windows/macOS/Linux
  - Drag-and-drop photo processing
  - Integrated photo viewer

### Developer Features

- [ ] **REST API**
  - HTTP API for remote processing
  - Webhook notifications for completed jobs
  - OpenAPI/Swagger documentation

- [ ] **Python SDK**
  - Official Python library for Lumina
  - Custom workflows and integrations
  - Jupyter notebook examples

- [ ] **Plugin System** 
  - Custom processors via plugins
  - Community-contributed extensions
  - Plugin marketplace

---

## 📊 Community Wishlist

**Top-voted features from the community will be added here**

*Help us prioritize! Vote on features in [GitHub Discussions](https://github.com/stevenvanassche/Lumina/discussions)*

---

## 🗓️ Version History

### v1.0.0 (December 2025) - Initial Release

**Core Features:**
- ✅ RT-DETR object detection (COCO + Objects365, 365 categories)
- ✅ iNat21 species classification (10,000+ species)
- ✅ EXIF metadata extraction
- ✅ XMP sidecar generation (Lightroom, Immich, On1 PhotoRAW compatible)
- ✅ Docker deployment (CPU and GPU variants)
- ✅ Checkpoint-based resumable processing
- ✅ Ray-based distributed processing
- ✅ Cross-platform support (Linux, macOS, Windows via WSL2)

**Performance:**
- ✅ CPU: ~1.0 photo/sec (4 threads, optimized)
- ✅ GPU: ~10.7 photos/sec (RTX 5090, 7 workers)

**Integrations:**
- ✅ Lightroom Classic (RAW XMP sidecars)
- ✅ Immich (External Libraries with XMP import)
- ✅ On1 PhotoRAW (hierarchical keywords)

---

## 🤝 Contributing

We welcome contributions to Lumina! Here's how you can help:

1. **Feature Requests**: Open a discussion in [GitHub Discussions](https://github.com/stevenvanassche/Lumina/discussions)
2. **Bug Reports**: File an issue in [GitHub Issues](https://github.com/stevenvanassche/Lumina/issues)
3. **Vote on Features**: Use 👍 reactions on issues/discussions to indicate interest

---

## 📝 Notes

- **Community Input**: Features with strong community support may be prioritized higher
- **Breaking Changes**: Major versions (v2.0, v3.0) may include breaking changes
- **Experimental Features**: Some features may be released as experimental/beta before stable

---

**Questions?** Join the discussion at [Lumina Discussions](https://github.com/stevenvanassche/Lumina/discussions)
