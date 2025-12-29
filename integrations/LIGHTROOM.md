# Lumina + Adobe Lightroom Classic Integration Guide

This guide explains how to use Lumina-generated XMP sidecars with Adobe Lightroom Classic for enhanced photo management and AI-powered metadata.

> **Note**: This guide is for **Lightroom Classic**. Lightroom CC (cloud version) has limited XMP sidecar support.

## Overview

**Workflow:**
1. **Lumina** generates AI metadata (object detection, species classification) and writes to XMP sidecars
2. **Lightroom Classic** reads XMP sidecars and imports metadata into its catalog
3. You get the best of both worlds: Lumina's specialized AI models + Lightroom's professional editing and organization tools

### ⚠️ Important: RAW vs JPEG XMP Support

**Lightroom Classic XMP Sidecar Behavior:**
- ✅ **RAW files** (.NEF, .CR2, .ARW, etc.): **Reads XMP sidecars automatically**
- ❌ **JPEG/TIFF files**: **Does NOT read XMP sidecars** - embeds metadata in file instead

**Recommended Workflow for Lumina:**
- Process RAW+JPEG pairs from your camera
- Keep both RAW and JPEG files together with XMP sidecar
- Import RAW files into Lightroom → XMP metadata automatically loaded
- JPEG files can be used for other software (Immich, On1 PhotoRAW)

---

## XMP Metadata Mapping

### What Lightroom Classic Reads from XMP

Lightroom Classic has **comprehensive XMP sidecar support**:

| Lumina XMP Field | Lightroom Support | Notes |
|------------------|-------------------|-------|
| `lr:hierarchicalSubject` | ✅ **Full support** | **Native Lightroom field** - Shows as hierarchical keyword trees |
| `dc:subject` | ✅ Imported | Flat keywords shown in Keyword panel |
| `Iptc4xmpCore:Keywords` | ✅ Imported | IPTC keywords merged with dc:subject |
| `dc:description` | ✅ Imported | Photo caption (Caption field) |
| `dc:title` | ✅ Imported | Photo title |
| `xmp:Rating` | ✅ Imported | Star rating (1-5) |
| `xmp:Label` | ✅ Imported | Color labels |
| `exif:GPSLatitude/Longitude` | ✅ Imported | GPS coordinates for Map module |
| `photoshop:SupplementalCategories` | ✅ Imported | Categories field |
| `tiff:ImageDescription` | ✅ Imported | Alternative caption field |

### Lumina's XMP Output for Lightroom

Lumina writes XMP metadata optimized for Lightroom Classic:

**Input (from AI models):**
```
"Inat21|bird|European Starling"
"COCO Detection|bird"
```

**Output in XMP:**

```xml
<!-- Hierarchical Keywords: Native Lightroom format -->
<lr:hierarchicalSubject>
  <rdf:Bag>
    <rdf:li>Inat21|bird|European Starling</rdf:li>
    <rdf:li>COCO Detection|bird</rdf:li>
  </rdf:Bag>
</lr:hierarchicalSubject>

<!-- Flat Keywords: All parts of hierarchy -->
<dc:subject>
  <rdf:Bag>
    <rdf:li>Inat21</rdf:li>
    <rdf:li>bird</rdf:li>
    <rdf:li>European Starling</rdf:li>
    <rdf:li>COCO Detection</rdf:li>
  </rdf:Bag>
</dc:subject>

<!-- Lightroom Weighted Keywords -->
<lr:weightedFlatSubject>
  <rdf:Bag>
    <rdf:li>Inat21</rdf:li>
    <rdf:li>bird</rdf:li>
    <rdf:li>European Starling</rdf:li>
    <rdf:li>COCO Detection</rdf:li>
  </rdf:Bag>
</lr:weightedFlatSubject>

<!-- Description: Most specific classification -->
<dc:description>
  <rdf:Alt>
    <rdf:li xml:lang="x-default">European Starling</rdf:li>
  </rdf:Alt>
</dc:description>
```

**What Lightroom sees:**

### Keyword Panel (Hierarchical View):
```
▶ Inat21
  ▶ bird
    • European Starling        (3 photos)
    • American Robin          (7 photos)
▶ COCO Detection
  • bird                      (10 photos)
```

### Flat Keywords List:
- `Inat21`
- `bird`
- `European Starling`
- `COCO Detection`

### Caption Field:
- "European Starling"

> **Note**: Lightroom shows ALL parts of hierarchical keywords as flat keywords, including the technical prefixes ("Inat21", "COCO Detection"). This is the standard XMP behavior that tools like Lightroom expect. Use the hierarchical view to see organized keyword trees.

---

## Complete Workflow: Lumina → Lightroom Classic

### Step 1: Generate XMP Sidecars with Lumina

**⚠️ Important:** If your photos already have XMP sidecar files with keywords, Lumina will merge its AI-generated keywords with the existing ones. This merging functionality is still being tested and may not work perfectly in all cases. We recommend:
- **Backing up your existing XMP files** before processing photos that already have metadata
- **Testing on a small subset** of photos first to verify the results
- Checking the merged keywords in Lightroom Classic to ensure they meet your expectations

Process your photo collection with Lumina:

```bash
# Run Lumina on your photo collection with RAW+JPEG pairing (recommended)
docker run --rm \
  -v /path/to/your/Photos:/data/photos \
  -v /path/to/lumina-cache:/data/cache \
  -e LUMINA_RAW_SUPPORT=on \
  stevenvanassche/lumina:cpu-latest
```

> **New in v1.1:** Enable `LUMINA_RAW_SUPPORT=on` to automatically pair RAW and JPEG files with the same base filename. The JPEG is used for faster AI analysis, and both files get identical XMP sidecars with the detected keywords.

**Result (RAW+JPEG workflow with LUMINA_RAW_SUPPORT=on):**
```
/path/to/your/Photos/
├── bird001.NEF          ← RAW file (for Lightroom)
├── bird001.JPG          ← JPEG file (used for AI analysis)
├── bird001.NEF.xmp      ← XMP for RAW (read by Lightroom)
├── bird001.JPG.xmp      ← XMP for JPEG (same keywords)
├── bird002.NEF
├── bird002.JPG
├── bird002.NEF.xmp
└── bird002.JPG.xmp
```

**Result (JPEG-only workflow - requires ExifTool later):**
```
/path/to/your/Photos/
├── bird001.jpg
├── bird001.xmp          ← ⚠️ Not read by Lightroom for JPEG!
├── bird002.jpg
└── bird002.xmp          ← Use ExifTool to embed if needed
```

### Step 2: Import Photos into Lightroom Classic

**Option A: Import New RAW Photos with XMP Sidecars**

1. Open **Lightroom Classic**
2. Click **Import** (bottom-left)
3. Select source folder: `/path/to/your/Photos`
4. **Important Settings**:
   - ✅ **File Handling** → "Add" (don't copy, use original location)
   - ✅ **Don't Import Suspected Duplicates** - Unchecked (if adding to existing catalog)
   - ✅ Only import RAW files (.NEF, .CR2, .ARW) - deselect JPEG if you have RAW+JPEG pairs
5. Click **Import**

Lightroom automatically reads XMP sidecars for RAW files during import.

> **💡 Tip**: If you have RAW+JPEG pairs, import only the RAW files into Lightroom. The JPEG files can be used by other software (Immich, On1 PhotoRAW) which will also read the same XMP sidecar.

**Option B: Update Existing Photos in Catalog**

If photos are already in your catalog but XMP files are new:

1. Navigate to folder in **Library** module
2. Select photos (Ctrl+A for all)
3. **Metadata** → **Read Metadata from Files** (Ctrl+Alt+Shift+R)
4. Lightroom reads XMP sidecars and updates catalog

### Step 3: Verify Metadata Import

1. **Select a photo** that has an XMP sidecar
2. Open **Metadata Panel** (right side):
   - Check **Caption** field - Should show species name
   - Check **EXIF and IPTC** - Shows GPS, camera info
3. Open **Keywording Panel**:
   - View **Keyword Tags** - Shows flat keywords
   - View **Keyword List** - Shows hierarchical tree structure

### Step 4: Synchronize Folder (If Keywords Don't Appear)

If keywords don't show up:

1. **Library** → Right-click on folder
2. **Synchronize Folder...**
3. Check ✅ **Scan for metadata updates**
4. Click **Synchronize**

---

## Troubleshooting: Keywords Not Appearing

### Problem 1: JPEG Files - XMP Sidecars Not Read

**Symptoms:**
- JPEG photos imported but no keywords from XMP
- Caption/description fields empty
- No GPS data despite XMP containing it

**Root Cause:**
Lightroom Classic **does NOT read XMP sidecars for JPEG/TIFF files**. This is by design - Lightroom embeds metadata directly in JPEG/TIFF files instead of using sidecars.

**Solutions:**

#### Solution 1: Use RAW Files Instead (Recommended)
Import the RAW files (.NEF, .CR2, .ARW) instead of JPEG. Lightroom **does** read XMP sidecars for RAW files.

```bash
# Directory structure for RAW+JPEG workflow
/path/to/Photos/
├── bird001.NEF    ← Import this into Lightroom ✅
├── bird001.JPG    ← Skip this for Lightroom
└── bird001.xmp    ← Automatically read with RAW file
```

#### Solution 2: Embed XMP into JPEG with ExifTool (Advanced)
If you only have JPEG files, use ExifTool to embed XMP metadata into the JPEG:

```bash
# Install ExifTool first
apt-get install libimage-exiftool-perl

# Embed XMP sidecar into JPEG
exiftool -tagsFromFile bird001.xmp -all:all bird001.jpg

# Then import JPEG into Lightroom with embedded metadata
```

> **Note**: This is a workaround for JPEG-only workflows. The recommended approach is to use RAW files with Lightroom.

### Problem 2: RAW Files - XMP Sidecars Not Being Read

**Symptoms:**
- RAW photos imported but no keywords
- Caption/description fields empty despite XMP sidecar existing

**Solutions:**

#### Solution 1: Manually Read Metadata

1. Select photos in Library module
2. **Metadata** → **Read Metadata from Files**
3. Or keyboard shortcut: **Ctrl+Alt+Shift+R** (Windows) / **Cmd+Opt+Shift+R** (Mac)

#### Solution 2: Synchronize Folder

1. Right-click folder in Library panel
2. **Synchronize Folder...**
3. Enable **Scan for metadata updates**
4. Click **Synchronize**

#### Solution 3: Check XMP File Naming

Lightroom requires **exact name matching**:

```bash
# Check your XMP files
ls -la /path/to/Photos/

# Correct naming for RAW files:
bird001.NEF
bird001.xmp  ← Matches base name

# Correct naming for RAW+JPEG:
bird001.NEF
bird001.JPG
bird001.xmp  ← Shared by both (only RAW uses it in Lightroom)

# WRONG naming:
bird001.NEF.xmp  ← Too many extensions
bird001_xmp      ← Wrong separator
```

### Problem 3: Hierarchical Keywords Appear Flat

**Symptoms:**
- Keywords imported but not in hierarchy
- See "Inat21|bird|European Starling" as single keyword instead of tree

**Solution:**

Lightroom needs the `lr:hierarchicalSubject` field:

1. **Verify XMP contains hierarchical field:**
   ```bash
   grep -A5 "hierarchicalSubject" bird001.xmp
   ```

   Should see:
   ```xml
   <lr:hierarchicalSubject>
     <rdf:Bag>
       <rdf:li>Inat21|bird|European Starling</rdf:li>
     </rdf:Bag>
   </lr:hierarchicalSubject>
   ```

2. **Force hierarchy interpretation:**
   - **Library** → **Metadata** → **Read Metadata from Files**
   - If still flat, check Lightroom Preferences:
     - **Edit** → **Preferences** → **Metadata** tab
     - Enable ✅ **Treat EXIF captions as Lightroom caption**

3. **Rebuild keyword hierarchy:**
   - **Metadata** → **Keyword List**
   - Right-click root keyword → **Import Keywords**
   - Select XMP file → Lightroom parses hierarchy

### Problem 3: Duplicate Keywords

**Symptoms:**
- Same keyword appears multiple times
- Both flat and hierarchical versions visible

**Explanation:**
This is **normal**. Lumina writes keywords to multiple XMP fields for maximum compatibility:
- `dc:subject` - Flat keywords
- `lr:hierarchicalSubject` - Hierarchical keywords
- `Iptc4xmpCore:Keywords` - IPTC standard

**Lightroom handles this intelligently:**
- Shows hierarchical version in Keyword List
- Shows flat version in Keyword Tags
- No visible duplicates in UI

**If you do see duplicates:**
1. **Keyword List** panel → Find duplicate
2. Right-click → **Delete**
3. Lightroom keeps the hierarchical version

### Problem 4: XMP Changes Lost After Lightroom Edits

**Symptoms:**
- Edit keywords in Lightroom
- Run Lumina again
- Manual keywords disappear

**Cause:**
Lumina's `force_update` mode overwrites XMP files.

**Solution:**

Use **quick_scan** execution mode:

```bash
docker run --rm \
  -v /path/to/Photos:/data/photos \
  -v /path/to/cache:/data/cache \
  -e LUMINA_EXECUTION_MODE=quick_scan \
  stevenvanassche/lumina:cpu-latest

```

**Lightroom XMP Write-Back:**

Enable automatic XMP write-back in Lightroom:
1. **Edit** → **Catalog Settings** → **Metadata** tab
2. Enable ✅ **Automatically write changes into XMP**

This ensures Lightroom updates XMP files when you edit keywords.

---

## Hierarchical Keywords in Lightroom Classic

### How Hierarchical Keywords Work

Lightroom natively supports **hierarchical keywords** using pipe separators (`|`) in the `lr:hierarchicalSubject` XMP field.

**Format:** `Parent|Child|Grandchild`

**Lumina's AI models create:**
- `Inat21|bird|European Starling`
- `Inat21|mammal|Red Fox`
- `COCO Detection|vehicle|car`

### Viewing Hierarchical Keywords

In **Keyword List Panel**:
```
▼ Inat21                        (4,832 photos)
  ▼ bird                        (3,245 photos)
    • European Starling         (342 photos)
    • American Robin            (456 photos)
    • Blue Jay                  (156 photos)
  ▼ mammal                      (892 photos)
    • White-tailed Deer         (234 photos)
    • Red Fox                   (45 photos)
▼ COCO Detection                (5,123 photos)
  ▼ vehicle                     (456 photos)
    • car                       (234 photos)
    • bicycle                   (112 photos)
  • person                      (2,345 photos)
```

### Working with Hierarchical Keywords

**Filter by parent category:**
1. Click **Inat21** → Shows all AI-identified species
2. Click **bird** → Shows only birds
3. Click **European Starling** → Shows only that species

**Export with hierarchy:**
1. Select photos
2. **Export** → **Metadata** section
3. Choose:
   - **All Metadata** - Includes hierarchical keywords
   - **All Except Camera Info** - Includes keywords, excludes EXIF

**Add to hierarchy:**
1. **Keyword List** → Select parent (e.g., "Inat21")
2. Right-click → **Create Keyword Tag**
3. Add new species manually
4. Lumina's keywords coexist with your manual additions

---

## Lightroom Classic Features Enhanced by Lumina

### 1. Smart Collections

Create **Smart Collections** based on AI keywords:

**Example: "All Birds"**
1. **Library** → **Smart Collection** → **New Smart Collection**
2. Name: "Birds - All Species"
3. Rules: **Keywords** → **Contains** → "bird"
4. Result: Auto-updates with all bird photos

**Example: "Rare Species"**
1. Rules: **Keywords** → **Contains** → "European Roller"
2. Result: Quickly find rare sightings

**Example: "Birds by Location"**
1. Rules:
   - **Keywords** → **Contains** → "bird"
   - **GPS** → **Contains GPS Coordinates**
2. Result: All geotagged bird photos for Map module

### 2. Map Module Integration

**Combine Lumina keywords + GPS:**
1. Open **Map** module
2. Filter by keyword: "European Starling"
3. Map shows locations where you photographed starlings
4. Identify migration patterns, hotspots, breeding grounds

### 3. Collections by Species

**Organize photos by species:**
1. **Keyword List** → Click species (e.g., "American Robin")
2. Select all filtered photos (Ctrl+A)
3. **Collection** → **Create Collection**
4. Name: "American Robin - Portfolio"
5. Use for:
   - Species-specific editing presets
   - Exporting to iNaturalist
   - Creating photo books

### 4. Batch Processing Workflows

**Example: Apply preset to all bird photos:**
1. Filter: **Keywords** → "bird"
2. Select all (Ctrl+A)
3. Apply **Develop** preset: "Wildlife - Birds"
4. All bird photos enhanced consistently

**Example: Export by species for contests:**
1. Filter: **Keywords** → "Blue Jay"
2. Select best photos
3. **Export** with metadata embedded
4. Submit to bird photography contests

### 5. Publish Services

**Publish to platforms with keywords:**
1. **Publish Services** → Configure (Flickr, SmugMug, etc.)
2. Export includes Lumina keywords
3. Photos automatically tagged on destination platform
4. Searchable by species name

---

## XMP Sidecar Compatibility

### Lightroom's XMP Behavior

**Reading XMP Sidecars:**
- Lightroom reads XMP on import
- Manual refresh: **Metadata** → **Read Metadata from Files**
- Auto-detection: Limited (use Synchronize Folder)

**Writing XMP Sidecars:**
- **Catalog Settings** → **Metadata** → Enable "Automatically write changes into XMP"
- Or manual: **Metadata** → **Save Metadata to Files**

### Two-Way Editing with Lumina

**Scenario: You add manual keywords in Lightroom, then reprocess with Lumina**

**Safe Workflow:**

1. **Initial Lumina processing:**
   ```bash
   # Generates XMP with AI keywords
   docker run --rm ... stevenvanassche/lumina:cpu-latest
   ```

2. **Import to Lightroom:**
   - Lightroom reads XMP
   - You add manual keywords: "Location|Belgium|Antwerp"
   - **Metadata** → **Save Metadata to Files** (writes to XMP)

3. **Reprocess with Lumina (skip mode):**
   ```bash
   # Skips photos that already have XMP
   docker run --rm \
     -e LUMINA_EXECUTION_MODE=skip_existing \
     ... stevenvanassche/lumina:cpu-latest
   ```

4. **Result:**
   - Original photos: Keep manual keywords + AI keywords
   - New photos: Get AI keywords

**Merge Workflow (Advanced):**

If you need to update AI keywords on existing photos:

1. Lightroom → **Save Metadata to Files** (preserves your edits)
2. Lumina → Use `execution_mode: "quick_scan"` in config 
3. Lightroom → **Read Metadata from Files** (loads updated AI keywords)

### XMP File Naming

Lightroom Classic supports:

**Sidecar XMP (Recommended):**
```
photo.jpg
photo.xmp    ← Lumina writes here
```

**Embedded XMP (DNG only):**
- DNG files can have embedded XMP
- Lightroom reads/writes embedded XMP automatically

---

## Advanced Workflows

### Workflow 1: iNaturalist Integration

**Goal:** Export photos to iNaturalist with AI-generated species IDs

1. **Lumina processes photos** → Identifies species
2. **Lightroom** → Filter by species keyword
3. **Review identifications:**
   - Correct? → Add **5-star rating**
   - Wrong? → Edit keyword manually
4. **Smart Collection:**
   - Rules: **Rating** ≥ 5, **Keywords** contains "Inat21"
5. **Export:**
   - Include keywords in metadata
   - Upload to iNaturalist
   - AI species name pre-fills observation

### Workflow 2: Species Portfolio

**Goal:** Create portfolio of best photos per species

1. **Keyword List** → Select species (e.g., "Northern Cardinal")
2. Apply **color label** (green = portfolio quality)
3. **Smart Collection:**
   - **Keywords**: "Northern Cardinal"
   - **Label Color**: Green
4. **Collection Set**: "Wildlife Portfolio"
   - Subcollections per species
5. **Export** for website, prints, contests

### Workflow 3: Migration Tracking

**Goal:** Track bird migration using GPS + keywords

1. **Lumina** → Process photos with GPS data
2. **Lightroom Map Module** → View species locations
3. **Filter** by date range + species:
   - Spring migration: March-May
   - Fall migration: September-November
4. **Create map book:**
   - **Book Module** → Include maps with keywords
   - Document migration patterns

### Workflow 4: Location-Based Tagging

**Combine Lumina AI keywords + Lightroom GPS workflows:**

1. **Lumina** identifies species
2. **Lightroom** reads GPS coordinates
3. **Add location keywords manually:**
   - "Location|North America|USA|Pennsylvania"
4. **Smart Collections:**
   - "Cardinals in Pennsylvania"
   - "European birds" (if you travel)
5. **Result:** Species + Location metadata

---

## Performance Considerations

### Large Catalogs

Lightroom Classic handles XMP sidecars efficiently:

- **50,000+ photos** - No performance issues
- **Keyword indexing** - Fast search even with 10,000+ keywords
- **XMP read** - Cached in catalog database

### Catalog vs. XMP as Source of Truth

**Lightroom uses catalog as primary:**
- Edits stored in catalog database
- XMP sidecars updated only when requested

**Best Practice:**
- Enable **Automatically write changes into XMP**
- Ensures XMP stays in sync with catalog
- Allows portability to other software (On1, Capture One, etc.)

### Network Storage (NAS)

If photos are on NAS:

- **XMP read performance** - Lightroom caches, reads XMP once on import
- **Catalog location** - Keep catalog on local SSD for speed
- **Preview cache** - Store locally for faster browsing

---

## Comparison: Lightroom Classic vs. Other Software

| Feature | Lightroom Classic | On1 PhotoRAW | Immich |
|---------|-------------------|--------------|--------|
| Hierarchical Keywords | ✅ Full support (native) | ✅ Full support | ⚠️ Flattened |
| XMP Read | ✅ Manual/Auto | ✅ Automatic | ✅ Automatic |
| XMP Write | ✅ Configurable | ✅ Automatic | ✅ Configurable |
| GPS/Map | ✅ Map module | ✅ Map view | ✅ Map view |
| RAW Editing | ✅ Professional | ✅ Professional | ❌ View only |
| Smart Collections | ✅ Yes | ✅ Smart Albums | ⚠️ Limited |
| Platform | Desktop | Desktop | Server/Web |

**When to use Lightroom Classic:**
- Professional photo editing required
- Large catalog (50,000+ photos)
- Desktop-based workflow
- Need full hierarchical keyword trees

**When to use Immich:**
- Web/mobile access required
- Family photo sharing
- Face recognition needed
- Simple browsing/organization

**When to use On1 PhotoRAW:**
- Alternative to Lightroom subscription
- One-time purchase preference
- Similar features to Lightroom

---

## Summary

### ✅ Recommended Workflow

1. **Shoot photos RAW + JPEG** → Copy RAW + JPEG photos to computer
2. **Run Lumina** → Generate AI metadata (species, objects, GPS)
   - Or enable **Watch Mode** (v1.1+) with `LUMINA_WATCH_MODE=on` for continuous processing
3. **Import to Lightroom** → XMP sidecars read automatically
4. **Verify keywords** → Check Keyword List for hierarchical trees
5. **Add manual keywords** → Locations, events, ratings
6. **Save metadata to files** → Keep XMP in sync
7. **Edit photos** → With full metadata context
8. **Export/Publish** → Keywords embedded for sharing

### ✅ Key Advantages

- **Native hierarchical keywords** - Full tree structure preserved
- **Professional editing** - RAW processing with metadata context
- **Two-way XMP** - Safe to edit keywords in Lightroom
- **Smart Collections** - Auto-update based on AI keywords
- **Map integration** - Species + location visualization
- **Industry standard** - Compatible with all XMP-aware software

### 🎯 Perfect For

- **Professional wildlife photographers** - Need editing + organization
- **Lightroom users** - Enhance existing workflow with AI
- **Large photo libraries** - Lightroom handles 100,000+ photos
- **Species documentation** - iNaturalist, eBird, scientific projects
- **Desktop workflows** - Full control over files and metadata

---

## Related Documentation

- **Lumina Installation**: [INSTALL.md](../docs/INSTALL.md)
- **Lumina Configuration**: [README.md](../README.md)
- **Immich Integration**: [IMMICH.md](IMMICH.md)
- **On1 PhotoRAW Integration**: [ON1-PHOTORAW.md](ON1-PHOTORAW.md)

## Support

- **Lumina Issues**: https://github.com/stevenvanassche/Lumina/issues
- **Lumina Discussions**: https://github.com/stevenvanassche/Lumina/discussions
- **Adobe Support**: https://helpx.adobe.com/lightroom-classic/

---

**Version**: Lumina 1.1.0
**Last Updated**: 2025-12-21
