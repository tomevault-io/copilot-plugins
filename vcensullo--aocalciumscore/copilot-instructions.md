## aocalciumscore

> **Project Name:** AoCalciumScore (formerly SlicerCaScore)

# AoCalciumScore - Claude Development Documentation

## 📋 Project Overview

**Project Name:** AoCalciumScore (formerly SlicerCaScore)
**Version:** 3.0
**Author:** Vittorio Censullo
**Institution:** AITeRTC - Associazione Italiana Tecnici di Radiologia Esperti in TC
**Year:** 2025
**License:** MIT
**Repository:** https://github.com/vcensullo/AoCalciumScore

### Purpose
Professional 3D Slicer extension for automated quantification of aortic valve calcification using the Agatston scoring method. Provides guided workflow for calcium scoring with sex-specific severity classification and professional PDF reporting.

---

## 🏗️ Architecture Overview

### File Structure
```
AoCaScore/
├── AoCaScore/
│   ├── AoCaScore.py              # Main plugin file (~4200 lines)
│   ├── CMakeLists.txt             # Module configuration
│   ├── Resources/
│   │   ├── Icons/
│   │   │   ├── AoCaScore.png     # Main icon
│   │   │   ├── SlicerCaScore.png # Legacy icon
│   │   │   ├── click_grow.png    # Point & Click method icon
│   │   │   ├── roi_box.png       # ROI Box method icon
│   │   │   └── brush.png         # Paint mode icon
│   │   └── UI/
│   │       └── SlicerCaScore.ui  # Qt UI definition
│   └── Testing/
│       ├── CMakeLists.txt
│       └── Python/
│           └── CMakeLists.txt
├── CMakeLists.txt                 # Extension configuration
├── LICENSE                        # MIT License
├── README.md                      # User documentation
├── AoCaScore.png                  # Extension icon
└── .gitignore                     # Git ignore rules
```

---

## 🔬 Technical Implementation

### Core Algorithm: Agatston Scoring Method

#### 1. **Minimum Lesion Area Standard**
**Location:** `AoCaScore.py` lines ~2345-2370

**Implementation:**
```python
MIN_AREA_MM2_STANDARD = 1.0  # mm² - fixed clinical standard

# Calculate minimum pixels needed to reach 1 mm² threshold
minPixelsFor1mm2 = int(np.ceil(MIN_AREA_MM2_STANDARD / sliceArea))
MIN_PIXELS_PER_SLICE = max(2, minPixelsFor1mm2)  # At least 2 pixels, but ≥1 mm²
MIN_AREA_MM2 = MIN_PIXELS_PER_SLICE * sliceArea  # Actual threshold applied
```

**Clinical Standard:**
- Minimum lesion area: **≥1 mm² per slice** (Agatston et al. 1990)
- Traditional criterion: ≥3 contiguous pixels (for ~0.5mm pixel spacing)
- Modern high-resolution CT: Adaptive pixel threshold based on actual pixel size
- Example: 0.377mm spacing → 2 pixels × 0.142 mm² = 0.284 mm² (too small!)
- Correct: ceil(1.0 / 0.142) = 8 pixels needed for 1 mm²

**Reference:** "Only contiguous voxels totaling ≥1 mm² are counted as lesions"

#### 2. **Density-Based Weighting**
**Location:** `AoCaScore.py` lines ~2400-2450

**Density Factors:**
- Factor 1: 130-199 HU
- Factor 2: 200-299 HU
- Factor 3: 300-399 HU
- Factor 4: ≥400 HU

**Formula:**
```
Agatston Score = Σ (Lesion Area × Density Factor)
```

#### 3. **Slice-by-Slice Processing**
**Location:** `AoCaScore.py` lines ~2300-2600

**Method:**
- 2D labeling per slice (NOT 3D volumetric)
- Automatic overlap detection for thick slices (e.g., 3mm thickness, 1.5mm increment)
- Per-slice density factor calculation
- Lesion area calculation in mm² per slice

**Validation:**
- Matches commercial software (Syngo.via)
- ±1.3% accuracy in clinical validation

---

## 📊 Sex-Specific Severity Classification

### Current Thresholds (JAHA 2024)
**Location:** `AoCaScore.py` lines ~2503-2520

**Women:**
- **Severe AS:** ≥1300 AU
- **Moderate AS:** 400-1300 AU
- **Mild:** 100-400 AU
- **Normal/Minimal:** <100 AU

**Men:**
- **Severe AS:** ≥2000 AU
- **Moderate AS:** 1000-2000 AU
- **Mild:** 100-1000 AU
- **Normal/Minimal:** <100 AU

### Clinical Reference
**Primary Source:**
Tastet L, Ali M, Pibarot P, et al. "Grading of Aortic Valve Calcification Severity and Risk Stratification in Aortic Stenosis." *J Am Heart Assoc.* 2024;13(15):e035605.

**Key Findings:**
- First sex-specific thresholds to differentiate mild vs moderate AS
- Multicenter validation
- Women: 1300 AU (lower threshold reflects different disease progression)
- Men: 2000 AU (higher threshold due to larger valve area)

**Previous Standard (Deprecated):**
- Pawade et al. 2018: Women ≥1377 AU, Men ≥2062 AU
- Updated October 2024 to JAHA 2024 values

---

## 🎨 User Interface Architecture

### Tab-Based Workflow
**Implementation:** Qt TabWidget with sequential enablement

**Tab 0: Settings**
- Dependency installation (reportlab)
- Threshold configuration (default: 130 HU)
- Company logo and description for PDF reports
- Settings persistence using QSettings

**Tab 1: Setup View**
- CT volume selection
- Layout optimization
- Automatic tab 2 enablement after volume selection

**Tab 2: Calcium Segmentation**
Three segmentation methods:

1. **Point and Click** (Recommended)
   - One-click intelligent segmentation
   - Automatic region growing from seed point
   - Threshold-guided expansion

2. **ROI Box**
   - Manual box placement around aortic valve
   - Region-based threshold segmentation
   - Precise control for complex cases

3. **Paint Mode**
   - Manual painting with threshold-guided brush
   - Fine-grained control
   - Useful for difficult anatomy

**Automatic noise filtering:** Lesions < 1 mm² removed

**Tab 3: Results & Analysis**
- Agatston Score calculation
- Quantitative results display:
  - Total Agatston Score (AU)
  - Total calcium volume (mm³)
  - Equivalent mass (mg)
  - Number of lesions
  - Mean and max density (HU)
  - Sex-specific severity classification
- 3D visualization with color-coded density
- Density distribution histogram

**Tab 4: Generate Report**
- Output directory selection
- Report options:
  - Include screenshots
  - Include 3D visualization
  - Include density charts
- PDF generation with:
  - Institution logo and description
  - Patient information
  - Color-coded severity classification
  - Comprehensive quantitative results
  - Density distribution table
  - 3D visualization and axial MIP
  - Clinical disclaimer
  - Scientific references

---

## 📝 PDF Report Generation

### Implementation
**Location:** `AoCaScore.py` lines ~3500-3900

**Technology:** ReportLab library

**Report Sections:**
1. **Header**
   - Institution logo (if configured)
   - Company description
   - Report title and patient info

2. **Severity Classification**
   - Color-coded severity indicator
   - Sex-specific thresholds displayed

3. **Quantitative Results Table**
   - Agatston Score
   - Volume and mass
   - Lesion count
   - Density statistics

4. **Density Distribution**
   - Table with 4 density levels
   - Visual color coding

5. **Images**
   - Axial MIP screenshot
   - 3D visualization (if enabled)
   - Density distribution chart (if enabled)

6. **Clinical Disclaimer**
   - Professional medical-legal text
   - Diagnostic limitations
   - Intended use statement

7. **Scientific References**
   - Complete bibliography
   - Key publications with DOI

---

## 🔧 Settings Persistence

### Implementation
**Technology:** Qt QSettings

**Stored Settings:**
- `threshold`: HU threshold for calcium detection (default: 130)
- `logoPath`: Path to institution logo
- `companyDescription`: Institution description text

**Location in Code:** `AoCaScore.py` lines ~400-500

**Storage Location:**
- Windows: Registry `HKEY_CURRENT_USER\Software\AITeRTC\AoCaScore`
- Linux: `~/.config/AITeRTC/AoCaScore.conf`
- macOS: `~/Library/Preferences/com.AITeRTC.AoCaScore.plist`

---

## 🧪 Testing & Validation

### Clinical Validation
- **Commercial Software Comparison:** Syngo.via
- **Accuracy:** ±1.3% agreement
- **Test Cases:** Multiple patient datasets
- **Validation Date:** October 2024

### Code Testing
**Test Structure:**
```
AoCaScore/Testing/
├── CMakeLists.txt
└── Python/
    └── CMakeLists.txt
```

**Current Status:** Basic structure in place, comprehensive tests pending

---

## 📚 Key Dependencies

### Required Python Libraries
1. **reportlab** - PDF generation
   - Installation: Via plugin Settings tab or `pip install reportlab`
   - Version: Any recent version compatible with 3D Slicer Python

### 3D Slicer Modules Used
- `slicer.util` - Utility functions
- `slicer.ScriptedLoadableModule` - Module base classes
- `vtk` - Visualization Toolkit
- `qt` - Qt GUI framework
- `ctk` - Common Toolkit
- `numpy` - Numerical operations

---

## 🔄 Recent Development History

### October 2024 - Major Updates

#### 1. **Agatston 1 mm² Standard Implementation**
**Date:** October 23, 2024
**Commit:** 7dc46b3

**Problem Identified:**
- Previous implementation used fixed 2-pixel minimum
- For high-resolution CT (e.g., 0.377mm spacing): 2 pixels = 0.284 mm² (below standard!)
- Clinical standard requires ≥1 mm² area per slice

**Solution:**
```python
MIN_AREA_MM2_STANDARD = 1.0  # mm² - fixed clinical standard
minPixelsFor1mm2 = int(np.ceil(1.0 / sliceArea))
MIN_PIXELS_PER_SLICE = max(2, minPixelsFor1mm2)
```

**Impact:**
- Accurate Agatston scoring across all CT scanner resolutions
- Compliance with original Agatston 1990 standard
- Matches 3D Slicer official implementation by lassoan

#### 2. **JAHA 2024 Severity Thresholds Update**
**Date:** October 23, 2024
**Commit:** 3032c80

**Changes:**
- Women: 1377 AU → **1300 AU** (Severe AS)
- Men: 2062 AU → **2000 AU** (Severe AS)
- Moderate thresholds updated accordingly

**Rationale:**
- Latest clinical evidence from Tastet et al. JAHA 2024
- First sex-specific thresholds for mild vs moderate differentiation
- Slightly lower thresholds → earlier detection of severe AS
- Improved risk stratification

**Code Locations Updated:**
- Lines 19-20: Header documentation
- Line 80: HTML tooltip
- Lines 350-351: UI severity info text
- Line 1647: 3D visualization threshold
- Lines 2503-2516: `classifySeverity()` function
- Lines 2964, 2967: Chart threshold arrays
- Line 3672: PDF report reference

#### 3. **Repository Cleanup**
**Date:** October 23, 2024

**Actions:**
- Removed `.claude/` directory from Git tracking
- Removed backup files (*.bak) from repository
- Added to `.gitignore`: `.claude/` and `*.bak`
- Clean Git history with single initial commit

#### 4. **Git History Reset**
**Date:** October 23, 2024
**Commit:** dfb1a13

**Action:** Complete history reset to single clean commit

**Reason:**
- Remove development artifacts from public history
- Professional appearance for public repository
- Single "Initial release: AoCalciumScore v3.0" commit

**Backup:** Created `backup-old-history` branch with full history

---

## 🐛 Known Issues & Future Improvements

### Current Known Issues
None at this time.

### Planned Enhancements

1. **Contrast-Enhanced CT Support**
   - Dynamic threshold adjustment
   - Different severity thresholds for contrast studies
   - Code structure already in place, needs validation

2. **Automated Testing Suite**
   - Unit tests for Agatston calculation
   - Integration tests with sample data
   - Regression tests for accuracy validation

3. **Enhanced 3D Visualization**
   - Interactive rotation in report
   - Multiple viewing angles
   - Slice-by-slice animation

4. **Export Formats**
   - DICOM Structured Report
   - CSV export for research
   - JSON export for integration

5. **Multi-Language Support**
   - Italian interface
   - Internationalization framework

---

## 🔍 Debugging Tips

### Common Issues

#### Issue: Agatston Score Too Low
**Possible Causes:**
1. Threshold too high (>130 HU)
2. Segmentation incomplete
3. Small lesions filtered out

**Debug Steps:**
1. Check threshold setting in Tab 0
2. Verify ROI includes all calcium
3. Check console output for lesion filtering messages
4. Review slice spacing - ensure ≥1 mm² criterion is met

#### Issue: PDF Generation Fails
**Possible Causes:**
1. ReportLab not installed
2. Invalid output directory
3. Logo file not found

**Debug Steps:**
1. Install dependencies via Tab 0
2. Check write permissions on output directory
3. Verify logo path in settings

#### Issue: Segmentation Not Working
**Possible Causes:**
1. Wrong volume selected (not CT)
2. Contrast-enhanced CT with 130 HU threshold
3. Very dense calcium (>1000 HU) with narrow window

**Debug Steps:**
1. Verify volume is non-contrast CT
2. Check image intensity range
3. Adjust window/level in Slicer
4. Try different segmentation method

---

## 📖 Code Navigation Guide

### Key Functions

#### `calculateAgatston(self, segmentationNode, volumeNode, sex='M')`
**Location:** Lines ~2200-2600
**Purpose:** Main Agatston calculation algorithm
**Parameters:**
- `segmentationNode`: Segmentation containing calcium
- `volumeNode`: Original CT volume
- `sex`: Patient sex for classification ('M' or 'F')

**Returns:** Dictionary with all results

#### `classifySeverity(self, agatstonScore, sex)`
**Location:** Lines ~2491-2520
**Purpose:** Sex-specific severity classification
**Returns:** Tuple of (classification_text, severity_level)

#### `generatePDFReport(self, results, outputDir, options)`
**Location:** Lines ~3500-3900
**Purpose:** Generate comprehensive PDF report
**Dependencies:** ReportLab library

#### `onPointAndClickSegmentation(self)`
**Location:** Lines ~1850-2000
**Purpose:** Point & Click segmentation method
**Features:** Automatic region growing from seed point

---

## 🎯 Modification Guidelines

### Adding New Features

#### 1. New Segmentation Method
**Steps:**
1. Add new button in Tab 2 UI section (~lines 1200-1400)
2. Implement method function (~lines 1800-2200)
3. Update documentation in tooltip (~lines 350-400)
4. Add icon to Resources/Icons/
5. Test with various CT datasets

#### 2. New Severity Classification
**Steps:**
1. Update `classifySeverity()` function (~lines 2491-2520)
2. Update documentation header (~lines 19-20)
3. Update UI tooltips (~lines 80, 350-351)
4. Update 3D visualization colors (~line 1647)
5. Update chart thresholds (~lines 2964, 2967)
6. Update PDF references (~line 3672)
7. Document in this file under "Sex-Specific Severity Classification"

#### 3. Adding New Report Element
**Steps:**
1. Locate PDF generation section (~lines 3500-3900)
2. Add new ReportLab elements (Paragraph, Table, Image, etc.)
3. Test PDF generation with various cases
4. Update report options in Tab 4 if needed

### Code Style Guidelines

1. **PEP 8 Compliance:** Follow Python style guide
2. **Documentation:** Add docstrings to all new functions
3. **Comments:** Explain complex algorithms, especially Agatston calculations
4. **Validation:** Add print statements for debugging (can be seen in Slicer Python console)
5. **Error Handling:** Use try-except blocks for file operations and calculations

---

## 📞 Support & Contact

**Developer:** Vittorio Censullo
**Institution:** AITeRTC - Associazione Italiana Tecnici di Radiologia Esperti in TC
**GitHub:** [@vcensullo](https://github.com/vcensullo)
**Repository:** https://github.com/vcensullo/AoCalciumScore
**Issues:** https://github.com/vcensullo/AoCalciumScore/issues

---

## 📄 License

MIT License - See LICENSE file for full text.

Copyright (c) 2025 Vittorio Censullo & AITeRTC

---

## 🤖 Development Tools

This project was developed with assistance from:
- **Claude Code** by Anthropic
- **3D Slicer** development environment
- **Git** version control

---

**Last Updated:** October 24, 2025
**Documentation Version:** 1.0
**Plugin Version:** 3.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vcensullo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
