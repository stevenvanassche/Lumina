# Lumina Installation Guide

This guide walks you through installing and running Lumina using Docker Compose.

## Prerequisites

Before you begin, ensure you have:

### All Platforms

- **Docker Engine** (version 20.10 or newer)
- **Docker Compose** (version 2.0 or newer)
- **Disk Space**:
  - ~2.5GB for AI models (downloaded on first run)
  - Storage for your photo collection
  - Additional cache space (recommended: 1GB+)

### Platform-Specific Requirements

#### Linux
- Docker Engine installed via package manager
- User added to `docker` group: `sudo usermod -aG docker $USER`

#### macOS
- **Docker Desktop for Mac** installed
- At least 8GB RAM allocated to Docker (Preferences → Resources)

#### Windows
- **Docker Desktop for Windows** installed
- **WSL2 backend** (required for GPU support, recommended for all installations)
- At least 8GB RAM allocated to Docker (Settings → Resources)

> **Windows Users:** **WSL2** (Windows Subsystem for Linux) is **required** if you want to use GPU acceleration. WSL2 is also recommended for better Docker performance even in CPU mode. Docker Desktop for Windows will prompt you to install WSL2 during setup.

### GPU Installation (Optional)

For up to 10x faster processing, you can use GPU acceleration:

- **NVIDIA GPU** (RTX series recommended, minimum 6GB VRAM)
- **NVIDIA drivers** (version 525.60 or newer)

**Platform Support:**
- **Linux**: NVIDIA Container Toolkit required (see [GPU Setup Guide](#gpu-setup))
- **Windows**: **WSL2 is required** for GPU support. GPU acceleration works via WSL2 + Docker Desktop (automatic setup)
- **macOS**: GPU acceleration not supported (Apple Silicon or AMD GPUs not compatible with CUDA)

> **Windows Users:** GPU acceleration on Windows **requires WSL2**. If you have WSL2 enabled, Docker Desktop installed, and an NVIDIA GPU with recent drivers (525.60+), GPU acceleration should work automatically. No additional NVIDIA Container Toolkit installation needed on Windows - the drivers handle this automatically through WSL2.

See [GPU Setup Guide](#gpu-setup) below for detailed installation instructions.

---

## Installation Steps

### Step 1: Create Project Directory

Create a directory for Lumina and your photo collection:

#### Linux / macOS / WSL2

```bash
mkdir -p lumina-docker
cd lumina-docker
```

#### Windows PowerShell

```powershell
New-Item -ItemType Directory -Path lumina-docker
cd lumina-docker
```

#### Windows Command Prompt (CMD)

```cmd
mkdir lumina-docker
cd lumina-docker
```

### Step 2: Download Configuration Files

Download the docker-compose configuration from GitHub:

#### Linux / macOS / WSL2

```bash
# Download docker-compose.yml
curl -o docker-compose.yml https://raw.githubusercontent.com/stevenvanassche/Lumina/master/docker/docker-compose.yml
```

#### Windows PowerShell

```powershell
# Download docker-compose.yml
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/stevenvanassche/Lumina/master/docker/docker-compose.yml" -OutFile "docker-compose.yml"
```

#### Windows Command Prompt (CMD)

```cmd
REM Download docker-compose.yml
curl -o docker-compose.yml https://raw.githubusercontent.com/stevenvanassche/Lumina/master/docker/docker-compose.yml
```

> **Note:** This configuration uses pre-built images from Docker Hub. No compilation required!

### Step 3: Create Directory Structure

#### Linux / macOS / WSL2

```bash
# Create required directories
mkdir -p photos cache
```

#### Windows PowerShell

```powershell
# Create required directories
New-Item -ItemType Directory -Path photos
New-Item -ItemType Directory -Path cache
```

#### Windows Command Prompt (CMD)

```cmd
REM Create required directories
mkdir photos
mkdir cache
```

Your directory structure should look like:
```
lumina-docker/
├── docker-compose.yml
├── photos/          # Your photo collection goes here
└── cache/           # Models, checkpoints, and logs
```

### Step 4: Configure Environment Variables

Create a `.env` file to configure Lumina. Choose the configuration based on whether you'll use CPU or GPU mode:

#### Option A: CPU Mode Configuration

**Alternative 1: Download pre-configured example:**

```bash
# Download .env.cpu.example and rename to .env
curl -o .env https://raw.githubusercontent.com/stevenvanassche/Lumina/master/docker/github/.env.cpu.example
```

**Alternative 2: Create manually:**

**Linux / macOS / WSL2:**

```bash
cat > .env << 'EOF'
# Required paths
PHOTOS_PATH=./photos
CACHE_PATH=./cache

# CPU Performance tuning
LUMINA_WORKERS=2
LUMINA_BATCH_SIZE=5
LUMINA_PARALLEL_BATCHES=2
LUMINA_TORCH_THREADS=4

# Logging (optional)
LUMINA_LOG_LEVEL=INFO

# Processing mode (optional)
LUMINA_EXECUTION_MODE=normal_scan
LUMINA_ENABLE_CHECKPOINTS=true
EOF
```

**Windows PowerShell:**

```powershell
@"
# Required paths
PHOTOS_PATH=./photos
CACHE_PATH=./cache

# CPU Performance tuning
LUMINA_WORKERS=2
LUMINA_BATCH_SIZE=5
LUMINA_PARALLEL_BATCHES=2
LUMINA_TORCH_THREADS=4

# Logging (optional)
LUMINA_LOG_LEVEL=INFO

# Processing mode (optional)
LUMINA_EXECUTION_MODE=normal_scan
LUMINA_ENABLE_CHECKPOINTS=true
"@ | Out-File -FilePath .env -Encoding ASCII
```

#### Option B: GPU Mode Configuration

**Alternative 1: Download pre-configured example:**

```bash
# Download .env.gpu.example and rename to .env
curl -o .env https://raw.githubusercontent.com/stevenvanassche/Lumina/master/docker/github/.env.gpu.example
```

**Alternative 2: Create manually:**

**Linux / macOS / WSL2:**

```bash
cat > .env << 'EOF'
# Required paths
PHOTOS_PATH=./photos
CACHE_PATH=./cache

# GPU Performance tuning
LUMINA_WORKERS=2
LUMINA_BATCH_SIZE=7
LUMINA_PARALLEL_BATCHES=10

# Logging (optional)
LUMINA_LOG_LEVEL=INFO

# Processing mode (optional)
LUMINA_EXECUTION_MODE=normal_scan
LUMINA_ENABLE_CHECKPOINTS=true
EOF
```

**Windows PowerShell:**

```powershell
@"
# Required paths
PHOTOS_PATH=./photos
CACHE_PATH=./cache

# GPU Performance tuning
LUMINA_WORKERS=2
LUMINA_BATCH_SIZE=7
LUMINA_PARALLEL_BATCHES=10

# Logging (optional)
LUMINA_LOG_LEVEL=INFO

# Processing mode (optional)
LUMINA_EXECUTION_MODE=normal_scan
LUMINA_ENABLE_CHECKPOINTS=true
"@ | Out-File -FilePath .env -Encoding ASCII
```

**Environment Variables Explained:**

| Variable | Default | Description |
|----------|---------|-------------|
| `PHOTOS_PATH` | `./photos` | Directory containing photos to process |
| `CACHE_PATH` | `./cache` | Directory for models, checkpoints, logs |
| `LUMINA_WORKERS` | `2` | Number of parallel workers |
| `LUMINA_BATCH_SIZE` | `5` (CPU) / `7` (GPU) | Photos per batch |
| `LUMINA_PARALLEL_BATCHES` | `2` (CPU) / `10` (GPU) | Concurrent batches |
| `LUMINA_TORCH_THREADS` | `4` (CPU) / `not used` (GPU) | PyTorch CPU threads |
| `LUMINA_LOG_LEVEL` | `INFO` | Logging verbosity (DEBUG, INFO, WARNING, ERROR) |
| `LUMINA_EXECUTION_MODE` | `normal_scan` | Processing mode (see below) |
| `LUMINA_ENABLE_CHECKPOINTS` | `true` | Enable resume capability |

**Execution Modes:**
- `force_update` - Reprocess all photos, overwrite existing XMP files
- `normal_scan` - **Recommended** - Process photos without metadata or with outdated metadata
- `quick_scan` - Only process photos without XMP files (faster for incremental runs)

For complete environment variable documentation, see [Configuration Reference](#configuration-reference).

### Step 5: Add Your Photos

**⚠️ IMPORTANT:** You must add photos to the `photos/` directory before running Lumina!

Copy your photo collection to the photos directory:

#### Linux / macOS

```bash
# Copy photos (recommended for Docker)
cp -r /path/to/your/photos/* photos/

# Or use symbolic links
ln -s /path/to/your/photos/* photos/
```

#### WSL2 (Windows Subsystem for Linux)

When running Lumina in WSL2, you have multiple options for accessing your Windows photos:

**Option 1: Direct access via Windows mounts (easiest, no copying needed)**

You can directly access your Windows photo folders using WSL2's automatic mounts:

```bash
# Update .env to point to Windows photos directory
# Windows C: drive is mounted at /mnt/c
# Example for Photos in C:\Users\YourName\Pictures\Photos
PHOTOS_PATH=/mnt/c/Users/YourName/Pictures/Photos
CACHE_PATH=./cache
```

**Advantages:**
- No need to copy photos
- Saves disk space
- Photos stay in original Windows location

**Potential Issues:**
- May encounter permission issues (see troubleshooting below)
- Slightly slower performance than native WSL2 filesystem

**Option 2: Copy photos to WSL2 filesystem (better performance)**

```bash
# Copy from Windows to WSL2
cp -r /mnt/c/Users/YourName/Pictures/Photos/* photos/

# Or use symbolic links
ln -s /mnt/c/Users/YourName/Pictures/Photos/* photos/
```

**Advantages:**
- Better I/O performance
- Fewer permission issues

**⚠️ Permission Issues with Windows Mounts:**

If you encounter "Permission denied" errors when using `/mnt/c/` paths, this is due to Docker volume permission handling. See the [Troubleshooting section](#4-permission-denied-when-accessing-photos) for solutions, including:
- Setting correct `USER_ID` and `GROUP_ID` in `.env`
- Using `chmod` to adjust permissions on mounted directories

#### Windows PowerShell

```powershell
# Copy photos
Copy-Item -Path "C:\Path\To\Your\Photos\*" -Destination "photos\" -Recurse

# Or copy specific file types (e.g., only JPG and RAW files)
Copy-Item -Path "C:\Path\To\Your\Photos\*.jpg" -Destination "photos\"
Copy-Item -Path "C:\Path\To\Your\Photos\*.NEF" -Destination "photos\"
```

#### Windows Command Prompt (CMD)

```cmd
REM Copy all photos
xcopy "C:\Path\To\Your\Photos\*" "photos\" /E /I

REM Or copy specific file types
copy "C:\Path\To\Your\Photos\*.jpg" "photos\"
copy "C:\Path\To\Your\Photos\*.NEF" "photos\"
```

**Supported File Types:**
- **Images**: `.jpg`, `.jpeg`, `.png`

> **💡 Tip for RAW+JPEG workflows:** If you shoot RAW+JPEG, include both in the `photos/` directory. Lumina will process the JPEG files and create XMP sidecars that work with both the RAW and JPEG files.

### Step 6: Start Lumina

#### CPU Mode (No GPU Required)

```bash
docker compose --profile cpu up
```

First run will download models (~2.5GB, 5-15 minutes depending on internet speed).

Expected performance: **~1 photo/second**

#### GPU Mode (NVIDIA GPU Required)

```bash
docker compose --profile gpu up
```

First run will download models (~2.5GB, 5-15 minutes depending on internet speed).

Expected performance: **~10 photos/second** (RTX 5090)

#### Run in Background (Detached Mode)

```bash
# CPU mode
docker compose --profile cpu up -d

# GPU mode
docker compose --profile gpu up -d

# View logs
docker compose logs -f
```

---

## Post-Installation

### Verify Installation

1. **Check logs** - You should see model download progress, then photo processing:
   ```
   [INFO] Model warmup complete (34.2s)
   [INFO] Processing photos: 100%|████████| 150/150 [2:30<00:00, 1.0 photo/s]
   ```

2. **Check XMP and JSON files** - XMP and JSON sidecar files should appear next to your photos:
   ```bash
   ls -la photos/
   # You should see:
   # photo.jpg
   # photo.xmp  ← AI-generated metadata in XMP (only created when detections found)
   # photo.json ← full EXIF and AI-generated metadata in JSON format
   ```

   **Note:** XMP sidecar files are only created when objects or species are detected in the photo. Photos without any detections will not have an XMP file generated, but will still have a JSON file with EXIF metadata.

   **⚠️ Existing XMP Files:** If your photos already have XMP sidecar files with keywords, Lumina will merge its AI-generated keywords with the existing ones. However, this merging functionality is still being tested and may not work perfectly in all cases. We recommend backing up your existing XMP files before processing photos that already have metadata, or testing on a small subset first.

   **⚠️ JSON Structure:** The JSON file format is subject to change in future versions. If you're building integrations that parse these JSON files, ensure your code is resilient to schema changes (e.g., handle missing fields gracefully, don't rely on exact key names).

3. **Check cache directory** - Models and logs are stored here:
   ```bash
   ls -la cache/
   # models/      - Downloaded AI models
   # checkpoints/ - Processing state (for resume)
   # logs/        - Processing logs
   ```

### Using Results in Photo Management Software

Lumina writes metadata to XMP sidecar files that are compatible with major photo management software:

**✅ Validated Compatibility:**
- **On1 PhotoRAW** - Full hierarchical keyword support (JPEG files with of without RAW files)
- **Immich** - External Libraries with XMP sidecar import (JPEG files)
- **Adobe Lightroom Classic** - Full hierarchical keyword support (only with both RAW and JPEG files)


**Expected Compatibility:**
- **Capture One** - XMP sidecar support
- **Photo Mechanic** - XMP sidecar support
- **digiKam** - XMP sidecar support
- **ACDSee** - XMP sidecar support

**Integration Guides:**
- **[On1 PhotoRAW Integration](../integrations/ON1-PHOTORAW.md)** - Complete workflow guide
- **[Immich Integration](../integrations/IMMICH.md)** - External Libraries setup, field mapping
- **[Lightroom Classic Integration](../integrations/LIGHTROOM.md)** - RAW+JPEG workflow, troubleshooting

> **Important:** Lightroom Classic does NOT read XMP sidecars for JPEG/TIFF files (only if RAW files are also present). See the [Lightroom Integration Guide](../integrations/LIGHTROOM.md) for details on the recommended RAW+JPEG workflow.

### Next Steps

- **Add more photos** - Just copy new photos to the `photos/` directory and restart Lumina
- **Adjust performance** - Edit `.env` to tune batch sizes and worker counts
- **Monitor progress** - Use `docker compose logs -f` to watch real-time progress
- **Explore results** - Open photos in Lightroom or On1 PhotoRAW to see hierarchical keyword trees

---

## GPU Setup

### Windows GPU Setup

**Requirements:**
1. **WSL2 installed and enabled** (REQUIRED)
2. **Docker Desktop for Windows** with WSL2 backend enabled
3. **NVIDIA GPU** with at least 6GB VRAM
4. **NVIDIA GPU drivers** version 525.60 or newer installed on **Windows** (not in WSL2)

**Setup Steps:**

1. **Install WSL2** (if not already installed):
   ```powershell
   # Open PowerShell as Administrator
   wsl --install

   # Restart your computer
   ```

2. **Verify WSL2 is default**:
   ```powershell
   wsl --list --verbose
   # Should show version 2 for your distributions

   # If not, set WSL2 as default:
   wsl --set-default-version 2
   ```

3. **Install NVIDIA GPU Drivers on Windows**:
   - Download and install the latest **Windows NVIDIA drivers** from: https://www.nvidia.com/download/index.aspx
   - Version 525.60 or newer required
   - **Important**: Install drivers on Windows, NOT inside WSL2

4. **Install Docker Desktop**:
   - Download from: https://www.docker.com/products/docker-desktop
   - During installation, ensure "Use WSL2 instead of Hyper-V" is selected
   - After installation, go to Settings → General → "Use the WSL2 based engine" should be checked

5. **Verify GPU Access in WSL2**:
   ```bash
   # Open WSL2 terminal (Ubuntu or your preferred distro)
   nvidia-smi
   ```

   You should see your GPU listed. If not, ensure Windows NVIDIA drivers are installed correctly.

6. **Verify Docker GPU Access**:
   ```bash
   # In WSL2 terminal
   docker run --rm --gpus all nvidia/cuda:12.1.0-base-ubuntu22.04 nvidia-smi
   ```

   You should see your GPU information.

**Troubleshooting:**
- **"nvidia-smi: command not found" in WSL2**: Your Windows NVIDIA drivers are not installed or too old
- **Docker can't access GPU**: Ensure Docker Desktop is using WSL2 backend (Settings → General)
- **"could not select device driver"**: Restart Docker Desktop and/or Windows

Once setup is complete, you can run Lumina in GPU mode from your WSL2 terminal.

### Linux GPU Setup

#### Install NVIDIA Container Toolkit

##### Ubuntu/Debian

```bash
# Add NVIDIA package repositories
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# Install nvidia-container-toolkit
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# Restart Docker
sudo systemctl restart docker

# Verify installation
docker run --rm --gpus all nvidia/cuda:12.1.0-base-ubuntu22.04 nvidia-smi
```

##### Other Distributions

See official NVIDIA Container Toolkit documentation:
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html

#### Verify GPU Access (Linux)

Test that Docker can access your GPU:

```bash
docker run --rm --gpus all nvidia/cuda:12.1.0-base-ubuntu22.04 nvidia-smi
```

You should see your GPU listed with memory usage and driver version.

---

## Troubleshooting

### Common Issues

#### 1. "docker: 'compose' is not a docker command"

**Problem**: Using old `docker-compose` command instead of `docker compose`

**Solution**:
- Update Docker to version 20.10+ which includes Compose V2
- Or use legacy command: `docker-compose` instead of `docker compose`

```bash
# Check Docker version
docker --version

# Update Docker (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

#### 2. "ERROR: could not find an available, non-overlapping IPv4 address pool"

**Problem**: Docker network conflicts

**Solution**: Prune unused networks

```bash
docker network prune
```

#### 3. GPU mode fails with "could not select device driver"

**Problem**: NVIDIA Container Toolkit not installed or Docker not restarted

**Solution**:
1. Install NVIDIA Container Toolkit (see [GPU Setup](#gpu-setup))
2. Restart Docker: `sudo systemctl restart docker`
3. Verify GPU access: `docker run --rm --gpus all nvidia/cuda:12.1.0-base-ubuntu22.04 nvidia-smi`

#### 4. "Permission denied" when accessing photos

**Problem**: Docker user doesn't have permissions to read/write photos directory

**Solutions:**

**For Linux/macOS or WSL2 with local filesystem:**

```bash
# Find your user ID
id -u

# Set ownership (replace 1000 with your user ID)
sudo chown -R 1000:1000 photos/ cache/
```

Or configure user ID in `.env`:

```bash
# Add to .env
USER_ID=1000
GROUP_ID=1000
```

**For WSL2 using Windows mounts (e.g., /mnt/c/):**

Windows mounts in WSL2 can have permission issues with Docker. Here are multiple solutions:

**Solution 1: Configure USER_ID and GROUP_ID in .env (recommended)**

```bash
# In WSL2, find your user ID
id -u    # Usually 1000
id -g    # Usually 1000

# Add to your .env file
USER_ID=1000
GROUP_ID=1000
```

**Solution 2: Adjust mount permissions**

```bash
# Make the Windows photos directory readable/writable in WSL2
sudo chmod -R 755 /mnt/c/Users/YourName/Pictures/Photos

# If you still have issues, you may need to adjust WSL2 mount options
# Create/edit /etc/wsl.conf in WSL2:
sudo nano /etc/wsl.conf

# Add these lines:
[automount]
enabled = true
options = "metadata,umask=22,fmask=11"

# Restart WSL2 (in Windows PowerShell):
wsl --shutdown
# Then reopen WSL2
```

**Solution 3: Copy to WSL2 filesystem instead**

If permission issues persist with Windows mounts, copy photos to the WSL2 filesystem:

```bash
# Copy from Windows to WSL2 local filesystem
cp -r /mnt/c/Users/YourName/Pictures/Photos/* photos/

# Or update .env to use local path
PHOTOS_PATH=./photos
```

This avoids cross-filesystem permission issues and provides better performance.

#### 5. Models download is very slow

**Problem**: Slow internet connection or Docker Hub rate limiting

**Solution**: Be patient on first run. Models are cached and won't be re-downloaded.

```bash
# Monitor download progress
docker compose logs -f

# Models are cached in:
ls -lh cache/models/
```

#### 6. Out of memory errors

**Problem**: Insufficient RAM or shared memory

**Solution**: Reduce batch sizes or increase Docker shared memory

```bash
# Edit .env to reduce batch sizes
LUMINA_BATCH_SIZE=2
LUMINA_PARALLEL_BATCHES=1

# Or increase shared memory in docker-compose.yml
# shm_size: '4gb'  # Increase from 2gb
```

#### 7. Processing stops without completing

**Problem**: Docker container stopped or crashed

**Solution**: Check logs and restart

```bash
# View logs to identify error
docker compose logs

# Restart with checkpoints enabled (resume from last position)
docker compose --profile cpu up
```

Lumina's checkpoint system will automatically resume from where it stopped.

---

## Configuration Reference

### Complete Environment Variables

```bash
# ============================================
# Required Paths
# ============================================
PHOTOS_PATH=./photos              # Photos to process
CACHE_PATH=./cache                # Models, checkpoints, logs

# ============================================
# User/Group IDs (for file permissions)
# ============================================
USER_ID=1000                      # Your user ID (run: id -u)
GROUP_ID=1000                     # Your group ID (run: id -g)

# ============================================
# Performance Tuning - CPU Mode
# ============================================
LUMINA_WORKERS=2                  # Parallel workers (1-5 recommended)
LUMINA_BATCH_SIZE=5               # Photos per batch (3-5 recommended)
LUMINA_PARALLEL_BATCHES=5         # Concurrent batches (1-10 recommended)
LUMINA_RAY_CPUS=auto              # Ray CPU allocation (auto or number)
LUMINA_TORCH_THREADS=4            # PyTorch thread count

# ============================================
# Performance Tuning - GPU Mode
# ============================================
LUMINA_WORKERS=2                  # Parallel workers (1-10 recommended)
LUMINA_BATCH_SIZE=7               # Photos per batch (5-10 recommended)
LUMINA_PARALLEL_BATCHES=10        # Concurrent batches (1-20 recommended)
LUMINA_RAY_CPUS=auto              # Ray CPU allocation
# LUMINA_TORCH_THREADS not needed for GPU

# ============================================
# Processing Configuration
# ============================================
LUMINA_EXECUTION_MODE=normal_scan    # force_update | normal_scan | quick_scan
LUMINA_ENABLE_CHECKPOINTS=true       # Enable resume capability
LUMINA_LOG_LEVEL=INFO                # DEBUG | INFO | WARNING | ERROR

```

### Performance Tuning Guidelines

**CPU Mode (No GPU):**
- **Low-end CPU (4 cores)**: `WORKERS=1, BATCH_SIZE=3, PARALLEL_BATCHES=1, LUMINA_TORCH_THREADS=1`
- **Mid-range CPU (8 cores)**: `WORKERS=2, BATCH_SIZE=5, PARALLEL_BATCHES=5, LUMINA_TORCH_THREADS=4` (default)
- **High-end CPU (16+ cores)**: `WORKERS=5, BATCH_SIZE=5, PARALLEL_BATCHES=5, LUMINA_TORCH_THREADS=4`

**GPU Mode:**
- **8GB VRAM**: `WORKERS=1, BATCH_SIZE=4, PARALLEL_BATCHES=2`
- **12GB VRAM**: `WORKERS=2, BATCH_SIZE=7, PARALLEL_BATCHES=10` (default)
- **16GB VRAM**: `WORKERS=5, BATCH_SIZE=10, PARALLEL_BATCHES=1 0`
- **24GB+ VRAM (RTX 5090)**: `WORKERS=10, BATCH_SIZE=10, PARALLEL_BATCHES=20`

**Memory Requirements:**
- CPU mode: ~8GB RAM minimum, 16GB recommended
- GPU mode: ~8GB VRAM minimum, 16GB+ recommended
- Shared memory (`shm_size`): Should be >30% of available RAM (default: 2GB)

---

## Upgrading

### Upgrade to New Version

```bash
# Stop current container
docker compose down

# Pull new image version
docker pull stevenvanassche/lumina:cpu-1.0.0-rc4  # or latest tag

# Update docker-compose.yml to use new image tag

# Restart
docker compose --profile cpu up -d
```

### Backup Before Upgrading

```bash
# Backup XMP files (your AI-generated metadata)
tar -czf xmp-backup-$(date +%Y%m%d).tar.gz photos/*.xmp

# Backup cache (optional - can be regenerated)
tar -czf cache-backup-$(date +%Y%m%d).tar.gz cache/
```

---

## Uninstalling

### Remove Containers and Images

```bash
# Stop and remove containers
docker compose down

# Remove images
docker rmi stevenvanassche/lumina:cpu-latest
docker rmi stevenvanassche/lumina:gpu-latest

# Optional: Remove all cached data
rm -rf cache/
```

**Important**: This will NOT delete your photos or XMP files in the `photos/` directory.

### Preserve XMP Metadata

Your XMP sidecar files contain valuable AI-generated metadata. Before uninstalling:

```bash
# Backup XMP files
tar -czf xmp-backup-$(date +%Y%m%d).tar.gz photos/*.xmp

# Or keep photos/ directory intact (recommended)
```

---

## Support

- **Issues**: https://github.com/stevenvanassche/Lumina/issues
- **Discussions**: https://github.com/stevenvanassche/Lumina/discussions
- **Documentation**: [README.md](README.md)
- **Docker Hub**: https://hub.docker.com/r/stevenvanassche/lumina

## Version

This installation guide is for Lumina **v1.0.0**.

For version history, see [CHANGELOG.md](../CHANGELOG.md).
