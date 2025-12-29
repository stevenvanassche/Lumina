# Lumina + Immich Integration Guide

This guide explains how to use Lumina-generated XMP sidecars with Immich for enhanced photo management and AI-powered metadata.

## Overview

**Workflow:**
1. **Lumina** generates AI metadata (object detection, species classification) and writes to XMP sidecars
2. **Immich** reads XMP sidecars and imports metadata into its database
3. You get the best of both worlds: Lumina's specialized AI models + Immich's photo management features

📖 **For complete Lumina setup instructions**, see the [Installation Guide](../docs/INSTALL.md).

---

## Method: mounting External Library

This is the **best approach** for importing existing photo collections with XMP sidecars without copying files.

### ⚠️ IMPORTANT: Read-Write Access Required for Lumina

**If you are using Lumina to generate XMP sidecars**, the photo directory **MUST have write permissions**:
- Lumina writes `.xmp` sidecar files (AI metadata)
- Lumina writes `.json` files (raw analysis data)
- **Do NOT use `:ro` (read-only)** mount for Lumina-processed directories!

### Step 1: Configure Immich Docker Compose

Add an **extra volume mount** to your Immich server for your photo collection:

**For Lumina-processed photos (read-write):**
```yaml
services:
  immich-server:
    volumes:
      # Immich's own upload storage (required)
      - ${UPLOAD_LOCATION}:/data

      # Lumina-processed photo collection (read-write for XMP/JSON generation)
      - ${IMPORT_LOCATION}:/mnt/media/photos

      # You can mount multiple libraries
      # - /nas/Photography:/mnt/media/nas
      # - /home/user/Photos/Wildlife:/mnt/media/wildlife

      - /etc/localtime:/etc/localtime:ro
```

**For pre-existing photos without Lumina (read-only):**
```yaml
      # Static photo archive (no XMP generation, read-only is safe)
      - /backup/Photos:/mnt/media/backup:ro
```

**Note**:
- **Without `:ro`** = read-write mount (required for Lumina to write XMP/JSON files)
- **With `:ro`** = read-only mount (safe for archives you won't modify)

### Step 2: Configure .env File

**Add only the `IMPORT_LOCATION` line** to your existing Immich `.env` file (don't modify the other parameters):

```bash
# Add this line to your existing .env file:
IMPORT_LOCATION=/home/user/Photos/Wildlife
```

**Your complete `.env` file should look like this:**

```bash
# Immich's own storage (uploaded files) - ALREADY EXISTS, don't change
UPLOAD_LOCATION=/home/user/immich-library

# External photo library (Lumina-processed photos) - ADD THIS LINE
IMPORT_LOCATION=/home/user/Photos/Wildlife

# Database location - ALREADY EXISTS, don't change
DB_DATA_LOCATION=/home/user/immich-database

# Database credentials - ALREADY EXISTS, don't change
DB_PASSWORD=your-secure-password
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
```

**⚠️ Important**: Only add `IMPORT_LOCATION`. The other parameters (`UPLOAD_LOCATION`, `DB_*`) should already exist in your Immich installation.

### Step 3: Add External Library in Immich UI

1. Start Immich: `docker compose up -d`
2. Open Immich web interface: `http://localhost:2283`
3. Navigate to **Administration** → **External Libraries**
4. Click **"Create External Library"**
5. Configure the library:
   - **Import paths**: `/mnt/media/photos` (this is `/home/user/Photos/Wildlife` inside the container)
   - **Watch for changes**: Enable (optional, for automatic rescans)
   - **Scan schedule**: Configure automatic scan intervals

### Step 4: Trigger Initial Scan

Click **"Scan Library"** to import all photos and their XMP metadata.

---

## ⚠️ Why Standard Immich Photo Upload Doesn't Work

**Important**: The typical Immich workflow of uploading photos via the mobile app or web interface **does NOT work** with Lumina-generated XMP sidecars.

### The Problem with Photo Upload

When you upload photos using Immich's standard upload methods:

1. **Mobile App Upload**:
   - Photos are uploaded from your phone/camera
   - Only the image file (JPG/PNG) is transferred
   - XMP sidecars **do not exist yet** on your phone
   - Result: ❌ No metadata from Lumina

2. **Web Interface Upload**:
   - You select photos from your computer
   - Immich copies only the image files
   - XMP sidecars **are ignored** during upload
   - Result: ❌ Lumina metadata is lost

3. **Upload Location Storage**:
   - Uploaded photos go to `UPLOAD_LOCATION` directory
   - Immich manages this directory internally
   - No way to add XMP sidecars after upload
   - Result: ❌ Cannot add Lumina metadata later

### The Solution: External Library

**For Lumina workflows, you MUST use External Library** (Method 1 above):

| Feature | Standard Upload | External Library (Lumina) |
|---------|-----------------|---------------------------|
| Photo upload method | Mobile app / Web upload | Process in-place on disk |
| XMP sidecar support | ❌ Not uploaded / Ignored | ✅ Read automatically |
| Lumina metadata | ❌ Lost permanently | ✅ Fully preserved |
| File duplication | ⚠️ Photos copied (2x storage) | ✅ Original files used |
| Workflow | Upload → View | Process with Lumina → Import → View |

### Correct Lumina Workflow

1. **Keep photos on your computer/NAS** (don't upload to Immich yet)
2. **Run Lumina** on the photo directory → generates XMP sidecars
3. **Mount directory as External Library** in Immich
4. **Immich reads photos + XMP** together automatically

**Why this works**:
- Photos and XMP sidecars stay together in the same directory
- Immich reads both the image file and XMP file
- No file copying or duplication needed
- Metadata is always synchronized

---

## Complete Workflow: Lumina → Immich

### Step 1: Generate XMP Sidecars with Lumina

Process your photo collection with Lumina to generate AI metadata:

```bash
# Run Lumina on your photo collection
docker run --rm \
  -v /home/user/Photos/Wildlife:/data/photos \
  -v /home/user/lumina-cache:/data/cache \
  stevenvanassche/lumina:cpu-latest

# Or with RAW+JPEG pairing (new in v1.1)
docker run --rm \
  -v /home/user/Photos/Wildlife:/data/photos \
  -v /home/user/lumina-cache:/data/cache \
  -e LUMINA_RAW_SUPPORT=on \
  stevenvanassche/lumina:cpu-latest
```

> **New in v1.1:** Enable `LUMINA_RAW_SUPPORT=on` to automatically pair RAW and JPEG files with the same base filename. The JPEG is used for AI analysis, and both files get XMP sidecars with identical keywords.

**Result:**
```
/home/user/Photos/Wildlife/
├── bird001.jpg
├── bird001.xmp          ← AI-generated metadata (COCO Detection, iNat21)
├── bird002.jpg
├── bird002.xmp
```

### Step 2: Mount Directory in Immich

**Add to your Immich `.env` file:**
```bash
IMPORT_LOCATION=/home/user/Photos/Wildlife
```

**Immich docker-compose.yml:**
```yaml
services:
  immich-server:
    volumes:
      - ${UPLOAD_LOCATION}:/data
      # Mount Lumina-processed photos (read-write for XMP/JSON generation)
      - ${IMPORT_LOCATION}:/mnt/media/photos
      - /etc/localtime:/etc/localtime:ro
```

**⚠️ Important**:
- No `:ro` on the photo mount - Lumina needs to write XMP/JSON files!
- The `${IMPORT_LOCATION}` variable references the path defined in your `.env` file above

### Step 3: Import into Immich

1. Navigate to **Administration → External Libraries**
2. Add library: `/mnt/media/photos`
3. Immich scans and reads the XMP sidecars

---

## XMP Metadata Mapping

### What Immich Reads from XMP

According to [Immich documentation](https://docs.immich.app/features/xmp-sidecars), the following fields are imported:

| Lumina XMP Field | Immich Support | Keyword Priority | Notes |
|------------------|----------------|------------------|-------|
| `digiKam:TagsList` | ❌ Not written by Lumina | #1 | Highest priority - DigiKam native format |
| `lr:hierarchicalSubject` | ✅ Imported | #2 | Flattened to individual tags (hierarchy lost) |
| `Iptc4xmpCore:Keywords` | ✅ Imported | #3 | **Lumina's primary tag field** - species/object names |
| `dc:subject` | ❌ Not read | - | Written by Lumina for other software compatibility |
| `dc:description` | ✅ Imported | - | Photo caption/description |
| `tiff:ImageDescription` | ✅ Imported | - | Alternative description field |
| `xmp:Rating` | ✅ Imported | - | Star rating (1-5) |
| `exif:GPSLatitude/Longitude` | ✅ Imported | - | GPS coordinates for map view |
| `exif:DateTimeOriginal` | ✅ Imported | - | Photo timestamp |
| Custom Lumina fields | ❌ Not supported | - | Confidence scores, bounding boxes ignored |

**Keyword Priority Explanation**:
- Immich reads keyword fields in priority order: #1 → #2 → #3
- Priority #1 (`digiKam:TagsList`) is not used by Lumina
- Priority #2 (`lr:hierarchicalSubject`) is written by Lumina but flattened by Immich
- Priority #3 (`Iptc4xmpCore:Keywords`) is **Lumina's primary field** and works perfectly with Immich

### Lumina's Smart Keyword Filtering

Lumina automatically filters technical prefixes from flat keywords to keep your tags clean:

**Input (from AI models):**
```
"Inat21|bird|European Starling"
"COCO Detection|bird"
```

**Output in XMP:**

```xml
<!-- IPTC Keywords: Clean, user-friendly tags (Immich reads this) -->
<Iptc4xmpCore:Keywords>
  <rdf:Bag>
    <rdf:li>bird</rdf:li>
    <rdf:li>European Starling</rdf:li>
  </rdf:Bag>
</Iptc4xmpCore:Keywords>

<!-- Description: Most specific classification -->
<dc:description>
  <rdf:Alt>
    <rdf:li xml:lang="x-default">European Starling</rdf:li>
  </rdf:Alt>
</dc:description>

<!-- Hierarchical: Full AI classification path (for advanced software) -->
<lr:hierarchicalSubject>
  <rdf:Seq>
    <rdf:li>Inat21|bird|European Starling</rdf:li>
    <rdf:li>COCO Detection|bird</rdf:li>
  </rdf:Seq>
</lr:hierarchicalSubject>

<!-- DC Subject: For compatibility with other software -->
<dc:subject>
  <rdf:Seq>
    <rdf:li>bird</rdf:li>
    <rdf:li>European Starling</rdf:li>
  </rdf:Seq>
</dc:subject>
```

**What Immich sees:**
- ✅ Tags: `bird`, `European Starling` (from `Iptc4xmpCore:Keywords`)
- ✅ Description: "European Starling" (from `dc:description`)
- ✅ No technical prefixes like "Inat21" or "COCO Detection" cluttering your tags!

---

## Practical Example: Complete Setup

### Directory Structure

```
/home/user/
├── Photos/
│   ├── Wildlife/              ← Original photos
│   │   ├── bird001.jpg
│   │   ├── bird001.xmp       ← Lumina-generated
│   │   └── ...
│   └── Travel/
├── immich-library/            ← Immich's managed storage
├── immich-database/           ← PostgreSQL data
└── lumina-cache/              ← Lumina model cache
```

### Immich docker-compose.yml

```yaml
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    volumes:
      # Immich's own library
      - ${UPLOAD_LOCATION}:/data

      # External libraries (read-write for Lumina XMP/JSON generation)
      - /home/user/Photos/Wildlife:/mnt/media/wildlife
      - /home/user/Photos/Travel:/mnt/media/travel

      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    ports:
      - '2283:2283'
    depends_on:
      - redis
      - database
    restart: always

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: always

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8
    restart: always

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    restart: always

volumes:
  model-cache:
```

### .env File

```bash
# Immich storage paths
UPLOAD_LOCATION=/home/user/immich-library
DB_DATA_LOCATION=/home/user/immich-database

# Database credentials
DB_PASSWORD=your-secure-password
DB_USERNAME=postgres
DB_DATABASE_NAME=immich

# Immich version
IMMICH_VERSION=release

# Note: No IMPORT_LOCATION needed here - paths are hardcoded in docker-compose.yml above
```

### Complete Workflow

1. **Process with Lumina:**
   ```bash
   docker run --rm \
     -v /home/user/Photos/Wildlife:/data/photos \
     -v /home/user/lumina-cache:/data/cache \
     stevenvanassche/lumina:cpu-latest
   ```

2. **Start Immich:**
   ```bash
   cd /path/to/immich
   docker compose up -d
   ```

3. **Add External Library** in Immich UI:
   - Path: `/mnt/media/wildlife`
   - Enable "Watch for changes"

4. **Initial Scan:**
   - Click "Scan Library" to import photos and XMP metadata

5. **Ongoing Updates:**
   - Reprocess photos with Lumina when needed
   - Immich automatically detects XMP changes (if "Watch for changes" is enabled)
   - Or manually trigger rescan in Immich UI

6. **Watch Mode (New in v1.1):**
   - Enable `LUMINA_WATCH_MODE=on` for continuous monitoring
   - Lumina automatically processes new photos as they're added
   - Combine with Immich's "Watch for changes" for seamless integration

---

## XMP Sidecar Limitations in Immich

### ⚠️ Known Limitations

1. **Hierarchical Keywords Are Flattened**
   ```xml
   <!-- Lumina writes: -->
   <lr:hierarchicalSubject>
     <rdf:li>Inat21|bird|European Starling</rdf:li>
   </lr:hierarchicalSubject>

   <!-- Immich imports as separate tags (hierarchy lost): -->
   Tags: "Inat21", "bird", "European Starling"
   ```

   **However**, Lumina also writes clean keywords to `Iptc4xmpCore:Keywords`:
   ```xml
   <Iptc4xmpCore:Keywords>
     <rdf:li>bird</rdf:li>
     <rdf:li>European Starling</rdf:li>
   </Iptc4xmpCore:Keywords>
   ```
   Result: ✅ Clean tags in Immich without technical prefixes!

2. **`dc:subject` Is NOT Read by Immich**
   - Immich prioritizes `Iptc4xmpCore:Keywords` and `lr:hierarchicalSubject`
   - Lumina writes `dc:subject` for compatibility with other software (Lightroom, On1 PhotoRAW)
   - This is **by design** and works correctly

3. **Custom XMP Namespaces Are Ignored**
   - Lumina's custom fields (confidence scores, bounding boxes, detection metadata) are not imported
   - Only standard Dublin Core, EXIF, IPTC, and Lightroom fields are recognized
   - Raw analysis data is still available in Lumina's JSON files

4. **XMP Sidecar Sync Can Be Bidirectional**
   - By default, XMP changes made in Immich are **not** written back to sidecar files
   - **You can enable bidirectional sync** in Immich settings:
     - Go to **Administration → Settings → Library**
     - Enable **"Write to XMP sidecars"**
     - Changes in Immich (tags, descriptions, ratings) will update XMP files
   - ⚠️ **Important for Lumina**: If enabled, manual changes in Immich will overwrite Lumina-generated XMP metadata

5. **Recommended Sync Strategy for Lumina**
   - **Option A - One-Way (Recommended)**:
     - Keep "Write to XMP sidecars" **disabled** in Immich
     - Use Lumina to generate all AI metadata → XMP sidecars
     - Use Immich only for browsing/viewing
     - Re-run Lumina when you need to update metadata
   - **Option B - Bidirectional**:
     - Enable "Write to XMP sidecars" in Immich
     - Manual tags added in Immich are written to XMP files
     - ⚠️ Risk: May conflict with Lumina-generated metadata if you re-process photos
     - Best for: Adding manual tags alongside Lumina's AI tags

### ✅ What Works Perfectly with Lumina

- ✅ **IPTC Keywords** (`Iptc4xmpCore:Keywords`) - Immich's primary tag source
- ✅ **Clean species/object names** - No "Inat21" or "COCO Detection" clutter
- ✅ **Photo descriptions** - Auto-generated from most specific classification
- ✅ **EXIF metadata** - Camera make/model, GPS coordinates, timestamps
- ✅ **Star ratings** (`xmp:Rating`) - If added by Lumina or other software
- ✅ **Hierarchical subjects** (`lr:hierarchicalSubject`) - Imported (but flattened)

### 🎯 Lumina + Immich Integration Result

**Before Lumina integration:**
- Manual tagging required
- Inconsistent keywords
- Time-consuming organization

**After Lumina + Immich:**
- ✅ Automatic AI-powered species identification
- ✅ Clean, searchable tags in Immich
- ✅ Hierarchical classification preserved in XMP (for other software)
- ✅ Photo descriptions auto-generated
- ✅ Compatible with Lightroom, On1 PhotoRAW, and other XMP-aware software

---

## Advanced: Combining Lumina + Immich ML

You can use **both** Lumina's specialized models **and** Immich's built-in ML features:

### Lumina Provides:
- **iNat21 Species Classification** (10,000+ species)
- **RT-DETR Object Detection** (COCO + Objects365)
- **EXIF Extraction**
- **XMP Sidecar Generation**

### Immich Provides:
- **Face Recognition**
- **CLIP-based Search** (natural language photo search)
- **Object Detection** (via Immich ML)
- **Duplicate Detection**
- **Web/Mobile Interface**

### Recommended Setup:

1. **Use Lumina for:**
   - Wildlife/nature photography (species identification)
   - Initial AI tagging of large collections
   - Generating XMP sidecars for portability

2. **Use Immich for:**
   - Face recognition and people tagging
   - General photo browsing and organization
   - Sharing albums with family/friends
   - Mobile access to photo library

3. **Workflow:**
   - Run Lumina on new photo batches → generates XMP
   - Or enable **Watch Mode** (v1.1+) for continuous processing
   - Immich imports photos + XMP metadata
   - Use Immich UI to browse, search, and organize
   - Immich's ML adds additional metadata (faces, CLIP embeddings)

---

## Troubleshooting

### XMP Metadata Not Appearing in Immich

**Problem**: Photos are imported but XMP metadata (tags, descriptions) is missing.

**Solutions:**

1. **Check XMP file location:**
   - XMP sidecar must be in the same directory as the photo
   - File naming: `photo.jpg` → `photo.xmp` (exact match)

2. **Verify XMP format:**
   - Must be valid XML
   - Use standard namespaces (Dublin Core, EXIF, IPTC)

3. **Rescan library:**
   - Go to **Administration → External Libraries**
   - Click **"Scan Library"** to force re-import

4. **Check Immich logs:**
   ```bash
   docker logs immich_server
   ```
   Look for XMP parsing errors

### Hierarchical Keywords Lost

**Problem**: Lumina's hierarchical keywords (e.g., `Inat21|bird|European Starling`) become flat tags.

**Explanation**: This is **expected behavior**. Immich does not support hierarchical tag structures.

**Workaround**: Use flat tags in XMP if hierarchy is not important, or accept that Immich will flatten them.

### External Library Not Updating

**Problem**: New XMP files or changes to existing XMP are not reflected in Immich.

**Solutions:**

1. **Enable "Watch for changes"** in External Library settings
2. **Manually trigger scan** after processing with Lumina
3. **Check file permissions** - Immich container must have read access to mounted directories

### Permission Errors

**Problem**: `Permission denied` errors when accessing external library, or Lumina cannot write XMP/JSON files.

**Solutions:**

1. **Check mount permissions:**
   ```bash
   # Ensure Immich container user can read files
   ls -la /home/user/Photos/Wildlife
   ```

2. **Fix ownership and permissions for Lumina:**
   ```bash
   # Make files readable AND writable (for XMP/JSON generation)
   chmod -R a+rw /home/user/Photos/Wildlife

   # OR set ownership to your user/group
   chown -R $(id -u):$(id -g) /home/user/Photos/Wildlife
   ```

3. **Verify mount is NOT read-only:**
   - **Do NOT use `:ro`** if you are using Lumina to generate XMP sidecars
   - Lumina requires write access to create `.xmp` and `.json` files
   - Correct: `- /home/user/Photos/Wildlife:/mnt/media/wildlife`
   - Wrong: `- /home/user/Photos/Wildlife:/mnt/media/wildlife:ro` ❌

---

## Performance Considerations

### Large Libraries

For large photo collections (10,000+ photos):

1. **Initial import takes time** - Be patient with first scan
2. **Incremental scans are faster** - Enable "Watch for changes"
3. **Consider scanning during off-hours** - Set scan schedule to nighttime

### Storage Requirements

- **Immich database** grows with library size (thumbnails, metadata, ML embeddings)
- **Estimate**: ~10-50MB per 1000 photos (varies with ML features enabled)
- **Lumina cache**: ~2.5GB for models (one-time download)

### Network Considerations

If mounting photos from NAS:

- **Use fast network** (Gigabit Ethernet recommended)
- **NFS mounts may be slow** for initial scans
- **Consider local cache** for frequently accessed photos

---

## Summary

### ✅ Recommended Workflow

1. **Process photos with Lumina** → generates AI metadata in XMP sidecars
   - Species/object detection with iNat21 and RT-DETR models
   - Clean keywords written to `Iptc4xmpCore:Keywords`
   - Photo descriptions auto-generated from classifications
2. **Mount photo directory in Immich** as External Library (read-only)
3. **Immich imports XMP metadata** → tags, descriptions, GPS, camera info
4. **Browse and organize** using Immich's web/mobile interface
   - Search by species name: "European Robin"
   - Filter by location using GPS data
   - View photos on map
5. **Reprocess with Lumina** when needed → Immich auto-detects updates

### ✅ Benefits of Integration

- **Clean Tags**: No technical prefixes ("Inat21", "COCO Detection") cluttering your library
- **Auto Descriptions**: Each photo gets a meaningful caption from AI classification
- **Portability**: XMP sidecars work with Lightroom, On1 PhotoRAW, and other software
- **Specialized AI**: Lumina's iNat21 species classification (10,000+ species)
- **Photo Management**: Immich's modern web/mobile interface with face recognition
- **Non-destructive**: Original photos remain untouched
- **Best of Both Worlds**: Lumina's ML models + Immich's browsing/sharing features

### ⚠️ Limitations to Accept

- **Hierarchical keywords are flattened** - But Lumina compensates by writing clean flat keywords
- **Custom XMP fields ignored** - Only affects advanced fields (confidence scores, bounding boxes)
- **One-way sync** - XMP → Immich works, Immich → XMP doesn't (standard Immich behavior)
- **Manual rescan needed** - Unless "Watch for changes" is enabled in External Library settings

### 🎯 Perfect For

- **Wildlife photographers** - Automatic species identification and tagging
- **Nature enthusiasts** - Organize thousands of photos by species/habitat
- **Photo libraries** - Professional cataloging with AI-powered metadata
- **Multi-software workflows** - XMP sidecars work everywhere (Immich, Lightroom, On1, etc.)

---

## Related Documentation

- **Lumina Installation**: [INSTALL.md](INSTALL.md)
- **Lumina Configuration**: [README.md](README.md)
- **Immich Documentation**: https://immich.app/docs
- **Immich External Libraries**: https://immich.app/docs/features/libraries

## Support

- **Lumina Issues**: https://github.com/stevenvanassche/Lumina/issues
- **Immich Issues**: https://github.com/immich-app/immich/issues
- **Lumina Discussions**: https://github.com/stevenvanassche/Lumina/discussions

---

**Version**: Lumina 1.1.0
**Last Updated**: 2025-12-21
