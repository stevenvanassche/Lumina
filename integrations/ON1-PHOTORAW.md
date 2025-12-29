# Lumina + On1 PhotoRAW Integration Guide

This guide explains how to use Lumina-generated XMP sidecars with On1 PhotoRAW for enhanced photo management and AI-powered metadata.

## Overview

**Workflow:**
1. **Lumina** generates AI metadata (object detection, species classification) and writes to XMP sidecars
2. **On1 PhotoRAW** reads XMP sidecars and imports metadata into its catalog
3. You get the best of both worlds: Lumina's specialized AI models + On1 PhotoRAW's powerful editing and organization tools

📖 **For complete Lumina setup instructions**, see the [Installation Guide](../docs/INSTALL.md).

---

## XMP Metadata Mapping

### What On1 PhotoRAW Reads from XMP

On1 PhotoRAW has **excellent XMP sidecar support** and reads most standard XMP fields:

| Lumina XMP Field | On1 PhotoRAW Support | Notes |
|------------------|----------------------|-------|
| `lr:hierarchicalSubject` | ✅ **Full support** | **Hierarchical keywords preserved!** Shows as expandable trees |
| `dc:subject` | ✅ Imported | Flat keywords shown in Keywords panel |
| `Iptc4xmpCore:Keywords` | ✅ Imported | IPTC keywords merged with dc:subject |
| `dc:description` | ✅ Imported | Photo caption/description |
| `tiff:ImageDescription` | ✅ Imported | Alternative description field |
| `xmp:Rating` | ✅ Imported | Star rating (1-5) |
| `xmp:Label` | ✅ Imported | Color labels (red, yellow, green, etc.) |
| `exif:GPSLatitude/Longitude` | ✅ Imported | GPS coordinates for map view |
| `photoshop:SupplementalCategories` | ✅ Imported | Additional categories |

### Lumina's XMP Output for On1 PhotoRAW

Lumina writes comprehensive XMP metadata optimized for On1 PhotoRAW:

**Input (from AI models):**
```
"Inat21|bird|European Starling"
"COCO Detection|bird"
```

**Output in XMP:**

```xml
<!-- Hierarchical Keywords: On1 PhotoRAW shows these as expandable trees -->
<lr:hierarchicalSubject>
  <rdf:Seq>
    <rdf:li>Inat21|bird|European Starling</rdf:li>
    <rdf:li>COCO Detection|bird</rdf:li>
  </rdf:Seq>
</lr:hierarchicalSubject>

<!-- Flat Keywords: Clean species/object names -->
<dc:subject>
  <rdf:Seq>
    <rdf:li>bird</rdf:li>
    <rdf:li>European Starling</rdf:li>
  </rdf:Seq>
</dc:subject>

<!-- IPTC Keywords: Same as dc:subject -->
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

<!-- Supplemental Categories -->
<photoshop:SupplementalCategories>
  <rdf:Seq>
    <rdf:li>bird</rdf:li>
    <rdf:li>European Starling</rdf:li>
  </rdf:Seq>
</photoshop:SupplementalCategories>
```

**What On1 PhotoRAW sees:**

### Keywords Panel View:
```
📁 Inat21
  📁 bird
    📷 European Starling        ← Hierarchical, expandable tree!
📁 COCO Detection
  📷 bird
```

### Flat Keywords:
- `bird`
- `European Starling`

### Description:
- "European Starling"

---

## Complete Workflow: Lumina → On1 PhotoRAW

### Step 1: Generate XMP Sidecars with Lumina

**⚠️ Important:** If your photos already have XMP sidecar files with keywords, Lumina will merge its AI-generated keywords with the existing ones. This merging functionality is still being tested and may not work perfectly in all cases. We recommend:
- **Backing up your existing XMP files** before processing photos that already have metadata
- **Testing on a small subset** of photos first to verify the results
- Checking the merged keywords in On1 PhotoRAW to ensure they meet your expectations

Process your photo collection with Lumina:

```bash
# Run Lumina on your photo collection
docker run --rm \
  -v /path/to/your/Photos:/data/photos \
  -v /path/to/lumina-cache:/data/cache \
  stevenvanassche/lumina:cpu-latest

# Or with RAW+JPEG pairing (new in v1.1)
docker run --rm \
  -v /path/to/your/Photos:/data/photos \
  -v /path/to/lumina-cache:/data/cache \
  -e LUMINA_RAW_SUPPORT=on \
  stevenvanassche/lumina:cpu-latest
```

> **New in v1.1:** Enable `LUMINA_RAW_SUPPORT=on` to automatically pair RAW and JPEG files with the same base filename. The JPEG is used for AI analysis, and both files get XMP sidecars with identical keywords.

**Result:**
```
/path/to/your/Photos/
├── bird001.jpg
├── bird001.xmp          ← AI-generated metadata
├── bird002.jpg
├── bird002.xmp
└── bird003.jpg
    bird003.xmp
```

### Step 2: Open Photos in On1 PhotoRAW

1. Launch **On1 PhotoRAW**
2. Navigate to your photo folder:
   - **Browse** → Navigate to `/path/to/your/Photos`
   - Or: **File** → **Add Folder** → Select your photo directory
3. On1 automatically reads XMP sidecars next to your photos

### Step 3: Verify Metadata Import

1. **Select a photo** that has an XMP sidecar
2. Open **Info Panel** (right side):
   - View **Keywords** - Should show hierarchical trees
   - View **Metadata** - Shows EXIF, GPS, descriptions
3. Open **Keywords Panel**:
   - Expand keyword trees to see hierarchical structure
   - Example: `Inat21 > bird > European Starling`

### Step 4: Use Keywords for Organization

**Search by keywords:**
1. In **Browse** module, use the **Search** bar
2. Type species name: "European Starling"
3. On1 finds all photos with that keyword

**Filter by keyword:**
1. Open **Keywords Panel**
2. Check/uncheck keywords to filter visible photos
3. Create **Smart Albums** based on keywords

**Batch operations:**
1. Select photos by keyword
2. Apply ratings, labels, or edits to all selected photos
3. Export with specific keywords

---

## Hierarchical Keywords in On1 PhotoRAW

### How Hierarchical Keywords Work

On1 PhotoRAW fully supports **Lightroom-style hierarchical keywords** using the pipe separator (`|`):

**Format:** `Category|Subcategory|Specific`

**Lumina's AI models create hierarchies like:**
- `Inat21|bird|European Starling`
- `Inat21|mammal|Red Fox`
- `COCO Detection|person`
- `COCO Detection|bicycle`

### Viewing Hierarchical Keywords

In **Keywords Panel** (On1 PhotoRAW):
```
📁 Inat21
  📁 bird
    📷 European Starling       (3 photos)
    📷 American Robin         (7 photos)
    📷 Blue Jay               (2 photos)
  📁 mammal
    📷 Red Fox                (1 photo)
📁 COCO Detection
  📁 vehicle
    📷 car                    (12 photos)
    📷 bicycle                (5 photos)
  📷 person                   (45 photos)
```

### Benefits of Hierarchical Keywords

1. **Organized Taxonomy** - Browse your library by category
2. **Multi-level Filtering** - Filter at any level (all birds, or just robins)
3. **Easy Navigation** - Expand/collapse categories
4. **Batch Tagging** - Add parent categories automatically
5. **Export Control** - Choose to export flat or hierarchical keywords

---

## Practical Example: Wildlife Photography Workflow

### Scenario: Wildlife Photographer with 5,000+ Photos

**Before Lumina:**
- Manual keyword tagging takes hours
- Inconsistent naming (robin vs. American Robin)
- No hierarchical organization
- Hard to find specific species

**After Lumina + On1 PhotoRAW:**

### Step 1: Process Entire Library with Lumina

```bash
# Process all photos (~2 hours for 5000 photos on CPU)
docker run --rm \
  -v /home/user/Wildlife-Photos:/data/photos \
  -v /home/user/lumina-cache:/data/cache \
  stevenvanassche/lumina:cpu-latest
```

### Step 2: Open in On1 PhotoRAW

**Keywords Panel automatically shows:**
```
📁 Inat21 (4,832 photos)
  📁 bird (3,245 photos)
    📷 American Robin (342 photos)
    📷 Blue Jay (156 photos)
    📷 Northern Cardinal (89 photos)
    ... (500+ species)
  📁 mammal (892 photos)
    📷 White-tailed Deer (234 photos)
    📷 Red Fox (45 photos)
    📷 Eastern Gray Squirrel (178 photos)
  📁 reptile (234 photos)
  📁 amphibian (78 photos)
  📁 insect (383 photos)
```

### Step 3: Find & Organize

**Find all robin photos:**
1. Keywords Panel → Expand `Inat21 > bird`
2. Click **American Robin**
3. All 342 robin photos displayed

**Create Smart Album:**
1. **File** → **New Smart Album**
2. Name: "Birds of North America"
3. Criteria: Keyword contains "Inat21|bird"
4. Auto-updates as you add more photos

**Export Selection:**
1. Select all robins
2. **Export** with keywords embedded
3. Share on iNaturalist, social media, or photo contests

---

## On1 PhotoRAW Features Enhanced by Lumina

### 1. Automatic Cataloging

**Without Lumina:**
- Import photos → Manual keyword entry → Hours of work

**With Lumina:**
- Import photos → XMP already present → Instant organization

### 2. Smart Albums

Create Smart Albums based on AI keywords:

**Examples:**
- **"All Birds"** - Keyword contains `bird`
- **"Rare Species"** - Specific species keywords
- **"Mammals by Location"** - Keyword + GPS coordinates
- **"Unidentified Wildlife"** - Photos without species keywords

### 3. Keyword Hierarchy Management

**Add to existing hierarchies:**
1. Lumina creates: `Inat21|bird|European Starling`
2. You manually add: `Location|Europe|Belgium`
3. Result: Photos have both AI + manual keyword trees

**Keyword Synonyms:**
- Create synonyms for common names
- Example: "Robin" → "American Robin"
- On1 finds photos using either term

### 4. Batch Operations

**Select by keyword, then:**
- Apply star ratings to all photos of specific species
- Add color labels (e.g., green for "needs review")
- Apply presets (e.g., "bird enhancement" preset to all bird photos)
- Export by species for different projects

### 5. GPS + Keywords

**Combine location + species:**
- View photos on **Map** module
- Filter by keyword: "European Starling"
- See where you photographed starlings on the map
- Identify hotspots for specific species

---

## XMP Sidecar Compatibility

### Two-Way Editing Support

On1 PhotoRAW has **excellent two-way XMP support**:

**Lumina writes XMP → On1 reads ✅**
- All Lumina-generated keywords appear in On1
- Hierarchical structures preserved

**On1 writes XMP → Lumina preserves ✅**
- Manual keywords you add in On1 are preserved
- Lumina updates only its own fields when reprocessing

**Result:** Safe to use both Lumina and On1 for keyword management!

### XMP File Location

On1 PhotoRAW supports both XMP formats:

**Sidecar XMP (Recommended):**
```
photo.jpg
photo.xmp    ← Lumina writes here
```

**Embedded XMP (Less common):**
- On1 can also read XMP embedded in DNG files
- Lumina focuses on sidecar files for maximum compatibility

### Synchronization Behavior

**When you add keywords in On1:**
1. On1 writes changes to XMP sidecar
2. XMP file is updated with new keywords
3. Lumina sees existing XMP when reprocessing
4. Lumina **merges** its keywords with yours (doesn't overwrite)

**Best Practice:**
- Use On1 for manual keyword additions (location, events, people)
- Use Lumina for AI species/object keywords
- Both coexist in the same XMP file

---

## Advanced: Custom Keyword Workflows

### Workflow 1: Species Identification Review

**Goal:** Review AI classifications before publishing

1. **Lumina processes photos** → Generates species keywords
2. **Open in On1 PhotoRAW** → Review keywords
3. **Add color label** (yellow = "needs review")
4. **Review each photo:**
   - Correct identification? → Green label
   - Wrong species? → Edit keyword manually
   - Uncertain? → Red label for expert review
5. **Export only green-labeled photos** for publication

### Workflow 2: Multi-Level Categorization

**Combine AI + Manual keywords:**

```
📁 Inat21 (AI-generated)
  📁 bird
    📷 European Starling
📁 Location (Manual)
  📁 Belgium
    📁 Antwerp
📁 Event (Manual)
  📁 Spring Migration 2025
📁 Quality (Manual)
  📷 Portfolio
  📷 Social Media
  📷 Archive
```

**Result:** Find photos like "European Starling, photographed in Antwerp during Spring Migration, portfolio quality"

### Workflow 3: Progressive Disclosure

**Start with high-level categories, drill down:**

1. **All Wildlife** (5000 photos)
2. Click **Inat21** → Shows all AI-identified species (4800 photos)
3. Click **bird** → All bird species (3200 photos)
4. Click **European Starling** → Just starlings (156 photos)

Each level narrows the selection while preserving context.

---

## Troubleshooting

### Keywords Not Showing in On1 PhotoRAW

**Problem:** XMP sidecars exist but keywords don't appear in On1.

**Solutions:**

1. **Verify XMP file location:**
   ```bash
   # XMP must be next to photo with exact name match
   photo.jpg  ← Photo
   photo.xmp  ← XMP sidecar (exact name match required)
   ```

2. **Refresh catalog:**
   - Right-click photo → **Refresh Metadata**
   - Or: **File** → **Rescan Folder**

3. **Check XMP format:**
   - Open XMP file in text editor
   - Verify it contains valid XML
   - Look for `<lr:hierarchicalSubject>` or `<dc:subject>` tags

4. **Rebuild cache:**
   - On1 caches metadata
   - **Edit** → **Preferences** → **Cache** → **Clear Cache**
   - Restart On1 PhotoRAW

### Hierarchical Keywords Appear Flat

**Problem:** Keywords show as "Inat21|bird|European Starling" instead of tree structure.

**Solution:**

On1 PhotoRAW should automatically parse pipe-separated hierarchies. If not:

1. **Check namespace:**
   - Must be `lr:hierarchicalSubject` (Lightroom namespace)
   - Not just `dc:subject`

2. **Verify format:**
   ```xml
   <lr:hierarchicalSubject>
     <rdf:Seq>
       <rdf:li>Category|Subcategory|Item</rdf:li>
     </rdf:Seq>
   </lr:hierarchicalSubject>
   ```

3. **Rebuild keyword list:**
   - **Keywords Panel** → Right-click → **Rebuild Keyword List**

### Duplicate Keywords

**Problem:** Same keyword appears multiple times.

**Cause:** Keywords written to multiple XMP fields (`dc:subject`, `Iptc4xmpCore:Keywords`, `lr:hierarchicalSubject`).

**This is normal and expected:**
- `dc:subject`: Flat keywords for compatibility
- `Iptc4xmpCore:Keywords`: IPTC standard keywords
- `lr:hierarchicalSubject`: Hierarchical structure

On1 PhotoRAW intelligently merges these - you won't see duplicates in the UI.

### XMP Changes Lost After Reprocessing

**Problem:** Manual keywords added in On1 disappear after running Lumina again.

**Solution:**

Check Lumina's execution mode:

```bash
# Use skip_existing mode to preserve existing XMP
-e LUMINA_EXECUTION_MODE=quick_scan
```

Or use `add` mode which merges instead of overwriting:

```bash
# In Lumina config
write_mode: "add"  # Instead of "update"
```

---

## Performance Considerations

### Large Libraries

On1 PhotoRAW handles large libraries efficiently:

- **10,000+ photos** - No performance issues with XMP sidecars
- **Keyword indexing** - First scan may take 5-10 minutes, then fast
- **Search** - Keyword search is near-instant even with complex hierarchies

### Network Storage (NAS)

If photos are on NAS:

- **XMP read performance** - On1 caches metadata locally, reads XMP only once
- **Recommendation** - Enable "Watch for changes" to detect new XMP files
- **Tip** - Process photos with Lumina before copying to NAS for optimal speed

### Catalog vs. Browse Mode

**Browse Mode (Recommended for Lumina integration):**
- Reads XMP directly from sidecar files
- No catalog database required
- Changes to XMP are immediately visible

**Catalog Mode:**
- Faster for very large libraries
- Metadata cached in database
- Requires rescan after Lumina reprocessing

**Recommendation:** Use Browse mode for most workflows.

---

## Summary

### ✅ Why On1 PhotoRAW + Lumina Is Perfect

1. **Full Hierarchical Keyword Support** - Unlike Immich, On1 preserves keyword trees
2. **Two-Way XMP Editing** - Add manual keywords without conflicts
3. **Excellent Performance** - Handles thousands of XMP sidecars efficiently
4. **Professional Editing Tools** - Edit photos with AI metadata context
5. **Multi-Platform** - Works on Windows, Mac (Immich is server-based)

### ✅ Recommended Workflow

1. **Shoot photos** → Copy to computer
2. **Run Lumina** → Generate AI metadata (species, objects)
   - Or enable **Watch Mode** (v1.1+) with `LUMINA_WATCH_MODE=on` for continuous processing
3. **Open in On1 PhotoRAW** → Instant organization by species
4. **Add manual keywords** → Locations, events, people
5. **Edit photos** → With full metadata context
6. **Export** → Keywords embedded for sharing/archiving

### ✅ Key Advantages Over Manual Tagging

| Task | Manual Tagging | Lumina + On1 |
|------|----------------|--------------|
| Keyword 1000 wildlife photos | ~10 hours | ~2 hours (Lumina) + instant (On1) |
| Species identification accuracy | Depends on knowledge | 10,000+ species database |
| Hierarchical organization | Manual creation | Automatic AI hierarchies |
| Consistency | Varies (typos, synonyms) | Perfect consistency |
| Update existing library | Re-tag everything | Reprocess with Lumina |

### 🎯 Perfect For

- **Wildlife photographers** - Automatic species ID with hierarchical categories
- **Professional photographers** - Need editing tools + metadata management
- **Local storage users** - Don't want server-based solution like Immich
- **Multi-software workflows** - On1 + Lightroom + Lumina all work together
- **Offline workflows** - No internet required after Lumina processing

---

## Related Documentation

- **Lumina Installation**: [INSTALL.md](../docs/INSTALL.md)
- **Lumina Configuration**: [README.md](../README.md)
- **Immich Integration**: [IMMICH.md](IMMICH.md)
- **On1 PhotoRAW Documentation**: https://www.on1.com/support/

## Support

- **Lumina Issues**: https://github.com/stevenvanassche/Lumina/issues
- **Lumina Discussions**: https://github.com/stevenvanassche/Lumina/discussions
- **On1 Support**: https://www.on1.com/support/

---

**Version**: Lumina 1.1.0
**Last Updated**: 2025-12-21
