# ProcessMono

Automated astrophotography image processing script for PixInsight. Processes narrowband (SHO) and broadband (RGB) master images through a complete pipeline with intermediate XISF saves at each stage for quality inspection and debugging.

## Features

- **Multi-branch processing**: RGB, SHO (Hubble palette), and Luminance workflows
- **Profile-driven parameters**: Predefined processing profiles (Standard DSO, Extended Nebulosity, Faint Dust, Bright Cores)
- **Complete stage saves**: XISF files saved after each processing step for review and quality control
- **Automatic crop mask removal**: Base files represent input data without overlay masks
- **FITS header validation**: Detects channel type from filename, FILTER-X naming, or FITS header
- **Processing summary**: Markdown report with profile name, processing time, and applied parameters
- **Multiple input formats**: Single-letter names (R.xisf), FILTER-X naming, or FITS header detection

## Requirements

**PixInsight Version:**
- PixInsight 1.9.6 or later (build 1719+)

**Required PixInsight Process Plugins:**
- BlurXTerminator ML5 or later
- NoiseXTerminator
- StarXTerminator 2.6+
- Spectrophotometric Color Calibration (SPCC)
- MultiscaleAdaptiveStretch

**External Tools:**
- GraXpert (command-line, for background extraction)
- Python 3.x (optional; GraXpert may use it internally)

## Installation

### Method 1: Local Deployment (Recommended for Development)
1. Copy `src/scripts/ProcessMono.js` to your PixInsight Scripts folder
2. In PixInsight, go to **Scripts → Feature Scripts**
3. Navigate to the directory where ProcessMono.js is located
4. The script will appear in **Scripts** menu

### Method 2: Script Menu Navigation
1. In PixInsight, go to **Scripts → Refresh Scripts**
2. ProcessMono will appear in **Scripts → Treehouse Astro → ProcessMono**

## Quick Start

1. **Launch the script** from PixInsight Scripts menu
2. **Add input files** using the Add button (R.xisf, G.xisf, B.xisf for RGB or S.xisf, H.xisf, O.xisf for SHO)
3. **Select a processing profile** from the dropdown
4. **Click OK** to begin processing
5. Review intermediate XISF files and final results in the output directory
6. Check `Processing_Summary.md` for a complete record

## Input File Preparation

### WBPP Output Format

If you used WBPP with drizzle 2x and auto-crop, your output files will look like:
```
masterLight_BIN-1_6248x4176_EXPOSURE-30.00s_FILTER-R_mono_drizzle_2x_autocrop.xisf
masterLight_BIN-1_6248x4176_EXPOSURE-30.00s_FILTER-G_mono_drizzle_2x_autocrop.xisf
masterLight_BIN-1_6248x4176_EXPOSURE-30.00s_FILTER-B_mono_drizzle_2x_autocrop.xisf
```

These files contain:
- A `crop_mask` window (automatically removed by ProcessMono)
- Filter information in the filename or FITS header
- Full 32-bit or 16-bit data

### Input Formats

ProcessMono accepts master files in three formats:

| Format | Example | Detection |
|--------|---------|-----------|
| Single-letter | `R.xisf`, `S.xisf`, `L.xisf` | Filename parsing |
| FILTER-X naming | `master_FILTER-R.xisf` | Regex extraction |
| FITS header | Any name + FILTER keyword | FITS header read |

If filename and FITS header disagree, the FITS header value is trusted and a warning is logged.

### Workflow Examples

**Narrowband Nebula (SHO + RGB for stars):**
Select all 6 channels: S.xisf, H.xisf, O.xisf, R.xisf, G.xisf, B.xisf
Result: Separate SHO and RGB processed images

**Galaxy (Luminance + RGB):**
Select 4 channels: L.xisf, R.xisf, G.xisf, B.xisf
Result: L and RGB processed separately

**Narrowband Nebula Only:**
Select 3 channels: S.xisf, H.xisf, O.xisf
Result: SHO processed only

## Processing Profiles

| Profile | Use Case | Settings |
|---------|----------|----------|
| Standard DSO | General deep-sky objects | Moderate sharpening, balanced noise reduction |
| Extended Nebulosity | Large nebulae, galaxies | Enhanced detail extraction, aggressive processing |
| Faint Dust / Dark Nebula | Dust lanes, molecular clouds | Conservative sharpening, detail preservation |
| Bright Cores / High Contrast | M42, M31, bright cores | High contrast enhancement, strong star suppression |

Profile parameters control Gradient Correction, BlurXTerminator, NoiseXTerminator, and MultiscaleAdaptiveStretch settings for consistent, predictable results across different data types.

## Processing Branches

### RGB Branch (Broadband)
Processes red, green, blue channels independently, then combines and applies color-specific enhancements.

```
R, G, B
  ↓ Gradient Correction (per-channel)
  ↓ BlurXTerminator Correct Only (per-channel)
RGB (combined)
  ↓ Spectrophotometric Color Calibration (combined)
  ↓ BlurXTerminator Full
  ↓ NoiseXTerminator
  ↓ StarXTerminator → Splits into Starless + Stars
RGB_starless_mas.xisf (Multiscale Adaptive Stretch)
RGB_stars_stretch.xisf
```

**Output files:**
- Base: `R.xisf`, `G.xisf`, `B.xisf`
- Stages: `R_gc.xisf`, `G_bxt_correct.xisf`, `RGB_spcc.xisf`, `RGB_bxt_full.xisf`, `RGB_nxt.xisf`
- Final: `RGB_starless_mas.xisf`, `RGB_stars_stretch.xisf`, `.tif` exports

### SHO Branch (Narrowband)
Processes Sulfur, Hydrogen, Oxygen channels with optional palette mode (Hubble, Foraxx, etc.).

```
S, H, O
  ↓ Gradient Correction (per-channel)
  ↓ BlurXTerminator Correct Only (per-channel)
  ↓ Linear Fit (cross-channel normalization)
SHO (combined, palette applied)
  ↓ BlurXTerminator Full
  ↓ NoiseXTerminator
  ↓ StarXTerminator → Splits into Starless + Stars
SHO_starless_mas.xisf (Multiscale Adaptive Stretch)
SHO_stars_stretch.xisf
```

**Output files:**
- Base: `S.xisf`, `H.xisf`, `O.xisf`
- Stages: `S_gc.xisf`, `H_linear_fit.xisf`, `O_bxt_correct.xisf`, `SHO_bxt_full.xisf`, `SHO_nxt.xisf`
- Final: `SHO_starless_mas.xisf`, `SHO_stars_stretch.xisf`, `.tif` exports

### Luminance Branch (Single Channel)
Processes a single luminance channel for use with RGB or for grayscale astrophotography.

```
L
  ↓ Gradient Correction
  ↓ BlurXTerminator Correct Only
  ↓ BlurXTerminator Full
  ↓ NoiseXTerminator
  ↓ StarXTerminator → Splits into Starless
L_starless_mas.xisf (Multiscale Adaptive Stretch)
```

**Output files:**
- Base: `L.xisf`
- Stages: `L_gc.xisf`, `L_bxt_correct.xisf`, `L_bxt_full.xisf`, `L_nxt.xisf`
- Final: `L_starless_mas.xisf`, `.tif` export

## Output Directory

Processing creates the following structure in your output directory:

```
output/
├── R.xisf, G.xisf, B.xisf        (Base files, input minus crop_mask)
├── R_gc.xisf, G_bxt_correct.xisf (Per-stage XISF files)
├── RGB_spcc.xisf, RGB_bxt_full.xisf, RGB_nxt.xisf
├── RGB_starless_mas.xisf         (Final: MAS applied to starless)
├── RGB_stars_stretch.xisf        (Final: Star stretch applied)
├── RGB_starless_mas.tif          (TIFF export of starless)
├── RGB_stars_stretch.tif         (TIFF export of stars)
└── Processing_Summary.md         (Processing report)
```

Each branch (RGB/SHO/L) generates its own set of files. All XISF files are cumulative — each stage file includes all previous processing steps.

## Processing Summary

After completion, `Processing_Summary.md` contains:

- Processing start and end times
- Total processing duration (MM:SS format)
- Selected profile name
- Branches processed (SHO/RGB/L)
- Complete list of applied processes with profile-based parameters
- Output file locations

Example:
```
Processing started at 11/25/2026 7:45 PM
Profile: Extended Nebulosity
Processing completed in 15:32

Branches detected: SHO=true, RGB=false, L=false

Processes Applied:
  • Gradient Correction (profile: Extended Nebulosity)
  • BlurXTerminator Full (profile: Extended Nebulosity)
  • NoiseXTerminator (profile: Extended Nebulosity)
  • MultiscaleAdaptiveStretch (profile: Extended Nebulosity)

Output directory: C:\path\to\output
```

## Workflow Options

Dialog checkboxes control which processing steps to apply:

- **Run Gradient Correction**: Apply automatic gradient removal per-channel (before BXT Correct Only)
- **Run BlurXTerminator Correct Only**: Correct-only mode without full deconvolution (per-channel, before channel combination)
- **Run BlurXTerminator Full**: Full deconvolution with blur removal (after channel combination and SPCC)
- **Run NoiseXTerminator**: Noise reduction with detail preservation
- **Run StarXTerminator**: Extract and separate stars from nebulosity
- **Run MultiscaleAdaptiveStretch**: Apply MAS to starless images for detail extraction
- **Run Star Stretch**: Apply histogram stretch to extracted stars
- **Export TIFFs**: Save final results as TIFF files in addition to XISF

## Processing Completion Features

### Final Status Popup

After processing completes, ProcessMono displays a summary popup showing:
- Total processing duration (HH:MM:SS format)
- Selected profile name
- Branches processed (SHO/RGB/L details)
- List of all exported output files
- Warnings or errors encountered during processing

### Auto-Save of Open Windows

Before displaying the final popup, ProcessMono automatically saves all open image windows to prevent unsaved-data warnings when you close PixInsight. Windows that have file paths and were modified are saved silently.

### Image Identifier Assignment

Each new image created during processing (after gradient correction, BXT, channel combination, etc.) has its Image Identifier automatically set to match its filename (minus the `.xisf` suffix). This makes it easier to track processing stages and identify windows in the console output.

## Narrowband Palette Processing

By default, SHO combination uses the Hubble palette to map S→R, H→G, O→B. This can be customized.

### Hubble Palette (Modified SHO Formula)

The standard industry approach to avoid green dominance:

- **R (Red)**: `(0.75 * Sii) + (0.25 * Ha)`
- **G (Green)**: `(0.50 * Ha) + (0.50 * Oiii)`
- **B (Blue)**: `Oiii`

**How it works:**
- Gold & Copper in Red: Mixing 25% Hydrogen-alpha into Sulfur-II shifts dull reddish-brown into vibrant copper and gold
- Suppresses Green Cast: Cutting Hydrogen-alpha in half and mixing with Oxygen-III reduces the dominant green
- Boosts Blues: Pure Oxygen-III in blue ensures outer structures pop with clean, electric blue

**Workflow:** Apply this formula while Sii, Ha, and Oiii masters are linear (unstretched) but color-calibrated and star-aligned. After stretching, run the SCNR tool (Green, 1.00 amount) to remove any remaining green gradient.

### Foraxx Palette

An alternative palette formula for different aesthetics:

- **R**: `(O^~O)*S + ~(O^~O)*H`
- **G**: `((O*H)^~(O*H))*H + ~((O*H)^~(O*H))*O`
- **B**: `O`

## Code Architecture

### Processing Engine

The script separates UI logic from processing:

- **ProcessMono class**: Main engine containing all processing workflows
- **ProcessMonoDialog class**: User interface for file selection and options
- **Individual process functions**: RunGradientCorrection(), RunBlurXterminatorFull(), etc.

Most processes use default PixInsight parameters, but these are easily updated. To modify a process:

1. Open the process in PixInsight's Process menu
2. Click **Instance Source Code** to see the PJSR code
3. Update the corresponding function in ProcessMono.js (e.g., RunBlurXterminatorFull)

Example process function:
```javascript
function RunStarXterminator(window, stars) {
    console.noteln("Running StarXTerminator on " + window.mainView.id + ".");

    var P = new StarXTerminator;
    P.ai_file = "StarXTerminator.11.pb";
    P.stars = stars;
    P.unscreen = false;
    P.overlap = 0.50;

    return P.executeOn(window.mainView);
}
```

## Troubleshooting

### No base files (R.xisf, S.xisf, etc.) created

**Problem:** Input files not being recognized.

**Solutions:**
- Verify file format: Use single-letter names (R.xisf) or FILTER-X naming
- Check FITS header: If filename detection fails, ProcessMono reads FITS header
- Review console output for error messages about filter detection

### Stage files missing at certain step

**Problem:** Processing failed partway through.

**Solutions:**
- Check console output for error messages (e.g., "BlurXTerminator failed on L")
- Verify input data quality and bit depth
- Try a less aggressive profile (Standard DSO instead of Extended Nebulosity)
- Check disk space for intermediate XISF saves

### Processing_Summary.md not created

**Problem:** Summary file missing from output directory.

**Solutions:**
- Verify output directory is writable
- Check that processing completed without fatal errors
- Ensure PixInsight has write permissions to the directory

### Filter mismatch warning in console

**Problem:** Filename detected one filter type but FITS header shows another.

**Solutions:**
- Review which is correct: filename or FITS header
- FITS header is trusted by default (most reliable source)
- Rename file to match FITS header if filename is incorrect

## Performance Notes

- Processing time depends on image size, bit depth, and selected processes
- Typical 4K 32-bit image: 10-20 minutes for full RGB pipeline
- Each intermediate XISF save requires ~200-400 MB of disk space
- Total disk usage for one complete run: 3-5 GB (plan accordingly)

## Development

See [DEVELOPMENT.md](./DEVELOPMENT.md) for:
- Code style and conventions
- Testing procedures
- Backup and safety practices
- AI assistant interaction preferences

## License

MIT. See `LICENSE`.

---

**Current Version**: 2.0  
**Last Updated**: 2026-11-25  
**Maintained By**: Ken Faubel
