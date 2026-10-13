# 3D Processing Pipeline

A comprehensive workflow for processing 3D models from video footage. This is a fork code from Lauren Olinger's 3d Modeling repository suited for Windows and OneDrive users. 

## Overview

This project provides a set of Python scripts to automate the workflow of processing underwater video footage into 3D models of coral reefs. The pipeline uses Agisoft Metashape for 3D reconstruction and is designed to work with the Territorial Coral Reef Monitoring Program (TCRMP) methodology.

## Requirements

- Agisoft Metashape Pro (v2.3.3 or later)
- Python 3.12 (for both local environment and Metashape compatibility)
- FFmpeg (for video frame extraction in Step 0)
- Required Python packages (specific versions in `requirements.txt`):
  - PyYAML
  - pandas
  - numpy
  - opencv-python
  - matplotlib
  - pillow
- **Note 1.0:** This workflow has been altered to preform on Windows. This has been tested on OneDrive and hard drive devices, however, performance is not guaranteed. For macOS compatibility, use 3d modeling repository by Lauren Olinger at https://github.com/laurenkolinger/3d . 


## Project Structure

```
./
├── analysis_params.yaml          # Base analysis parameters template
├── docs/                         # Documentation files
│   ├── api_1july.txt            # API documentation
│   └── metashape_python_api_2_1_1.pdf
├── examples/                     # Example project directories (local)
├── images/                       # Supporting images (eg. for RMD rendering)
├── presets/                      # Software preset files
│   ├── lightroom/               # Adobe Lightroom presets
│   │   └── step0_lightroom_hdrphoto_r5c.xmp
│   └── premiere/                # Adobe Premiere presets
│       └── step0_premierepro_uhd_8k_23sept2024.epr
├── src/                          # Source code
│   ├── config.py                # Configuration loading utilities
│   ├── step0.py                 # Frame extraction
│   ├── step1.py                 # Initial 3D processing (most time-consuming)
│   ├── step2.py                 # Chunk management/consolidation
│   ├── step3.py                 # Model processing (automatic scaling)
│   ├── step3_manualScale.py     # Model processing (manual scaling)
│   ├── step4.py                 # Final exports & web publishing
│   ├── legacy/                  # Legacy/archived scripts
│   └── utility/                 # Utility scripts
│       ├── enumerate_gpus.py    # GPU detection for Metashape
│       ├── reset_full.py        # Complete project reset
│       ├── reset_step1.py       # Reset preserving Steps 0&1
│       └── file_naming.py       # Standardized file naming functions
├── README.md                     # This documentation
└── requirements.txt              # Python package dependencies
```

## Initial Setup

### 1. Repository Setup

Clone this repository to your local machine:
```bash
git clone https://github.com/cheyennegoodman/GOODMAN_Coral_Damselfish_3D_Models.git

cd 3d
```

### 2. Create Project Directory Structure

Create all necessary directories for your project:
```bash
# From the workspace root, create the required directories
mkdir -p {PROJECT_DIR}/{video_source,processing,output}
```

This will create the following directory structure:
```
{PROJECT_DIR}/
├── video_source/                    # Input video files
├── processing/                      # Intermediate processing data
│   ├── frames/                      # Extracted frames organized by model (Step 0)
│   └── psxraw/                      # Initial PSX files (Step 1)
└── output/                          # All final outputs
    ├── psx/                         # Consolidated PSX files by site (Step 2)
    ├── orthomosaics/                # Orthomosaic outputs (Step 3)
    │   └── {MODEL_ID}/              # Each model in its own subdirectory
    │       └── {MODEL_ID}.tif       # Clean model ID filename
    ├── models/                      # 3D model outputs (Step 3)
    │   └── {MODEL_ID}/              # Each model in its own subdirectory
    │       ├── {MODEL_ID}.obj       # Clean model ID filename
    │       └── [texture files]      # Associated texture files
    ├── reports/                     # Processing reports (Step 3)
    │   ├── {MODEL_ID}.pdf           # Clean model ID filename (flat structure)
    │   └── {MODEL_ID}.pdf           # All reports in same directory
    ├── logs/                        # Processing logs
    └── final/                       # Final high-resolution outputs (Step 4)
```

**Important:** Once this directory structure is created, do not rename or move the standard subdirectories (`video_source`, `processing`, `output`). The scripts rely on this specific structure. The only manual change expected within `{PROJECT_DIR}` after setup is adding your video files to the `{PROJECT_DIR}/video_source/` directory.

Copy and configure the analysis parameters file:
```bash
# Copy the base configuration file to your project
cp analysis_params.yaml {PROJECT_DIR}/
```

The configuration file (`analysis_params.yaml`) located within your `{PROJECT_DIR}` contains all the settings for the project.

Make sure to:

1. Review and update the description and notes inside the `{PROJECT_DIR}/analysis_params.yaml` file.
2. Adjust any processing parameters within the `{PROJECT_DIR}/analysis_params.yaml` file as needed for your specific project.
3. Note that the primary input/output directory paths (`video_source`, `processing`, `output`, etc.) are derived automatically by the scripts based on the `{PROJECT_DIR}` you provide when running them. 

## Standardized File Naming

The pipeline uses a clean, standardized naming system for all outputs:

- **Model ID Format:** All file names use the exact Model ID (e.g., `04_10_2026_PLOT_18`)
- **No Suffixes:** Files are named simply as `{MODEL_ID}.ext`
- **Organized Structure:** 
  - Orthomosaics and models get their own subdirectories: `output/orthomosaics/{MODEL_ID}/` and `output/models/{MODEL_ID}/`
  - Reports are flat in `output/reports/{MODEL_ID}.pdf`
- **Consistent Across Scripts:** Both `step3.py` and `step3_manualScale.py` produce identical file names and structure

**Example Output:**
```
output/
├── orthomosaics/
│   └── TCRMP20241014_3D_BWR_T2/
│       └── TCRMP20241014_3D_BWR_T2.tif
├── models/
│   └── TCRMP20241014_3D_BWR_T2/
│       ├── TCRMP20241014_3D_BWR_T2.obj
│       └── TCRMP20241014_3D_BWR_T2.jpg  # texture file
└── reports/
    └── TCRMP20241014_3D_BWR_T2.pdf
```

### 3. Installing Dependencies

The pipeline requires dependencies in two Python environments:

1. Your local environment (for frame extraction - step0.py)
2. Metashape's Python environment (for 3D processing - step1.py and beyond)

#### Local Environment Setup

Create a Python virtual environment in your project:
```bash
# Create virtual environment in project directory using Python 3.12
python3.12 -m venv {PROJECT_DIR}/venv312

# Activate the virtual environment
{PROJECT_DIR}\venv312\Scripts\activate.ps1

# Install dependencies
pip install -r requirements.txt
```

#### Running Metashape Scripts

To run Metashape scripts with the correct Python environment, you'll need to set the PYTHONPATH to point to your virtual environment's site-packages and make your project directory into a variable. This ensures Metashape uses the packages from your virtual environment while maintaining compatibility with Metashape's Python 3.12.

The general format for running Metashape scripts is:
```bash
$env:PROJECT_DIR = $project_dir # make your project directory a variable

$env:PYTHONPATH = "$project_dir\venv312\Lib\site-packages" # create a python path to the project directory so Agisoft Metashape uses the correct virtual environment (venv312)

& "C:\Program Files\Agisoft\Metashape Pro\metashape.exe" -r src\stepX.py # Agisoft Metashape access the virtual environment 
```

Where `stepX.py` is the specific step you want to run (step1.py, step2.py, etc.) and `{PROJECT_DIR}` is the path to your project directory containing `analysis_params.yaml`.

## Workflow Overview

The complete processing workflow consists of the following steps:

1. **Frame Extraction** (step0.py): Extract frames from video footage
2. **Initial 3D Processing** (step1.py): Process extracted frames to create initial 3D models
3. **Manual Quality Check & Alignment**: Check and align models (manual step)
4. **Chunk Management** (step2.py): Organize chunks by site
5. **Model Processing and Exports** (step3.py): Add scale bars, remove small components, export assets
6. **Manual Straightening & Scaling**: Straighten and scale models (manual step)
7. **Manual Touchups**: Review and touch up models (manual step)

**Note:** Each script will prompt for the project directory containing the `analysis_params.yaml` file if not provided as a command-line argument. This allows processing different projects without code modifications, as source files will be linked to individual project directories dynamically for each run.

## Detailed Workflow

### Step 0: Frame Extraction

Extracts frames from video footage at a specified rate.

```bash
python src/step0.py {PROJECT_DIR}
```

Scans `video_source/` for videos, extracts frames to `processing/frames/`, and creates tracking CSV files.

### Step 1: Initial 3D Processing ⏱️ *Most Time-Consuming*

Performs initial 3D reconstruction using extracted frames. Creates batched PSX files with multiple models grouped for efficiency.

```bash
$env:PYTHONPATH = "$project_dir\venv312\Lib\site-packages"

& "C:\Program Files\Agisoft\Metashape Pro\metashape.exe" -r src\step1.py
```

Groups models into batches, aligns cameras, builds depth maps, creates 3D models with textures, and saves to `processing/psxraw/`.

### Manual Step: Quality Check & Alignment

After Step 1, manually check the quality of generated models:

1. Open each PSX file in `processing/psxraw/` with Metashape
2. For each model (chunk): review camera alignment, check point cloud quality, verify model IDs are correctly labeled
3. Save the project

### Step 2: Chunk Management

Consolidates chunks by site to prepare for final processing.

```bash
$env:PYTHONPATH = "$project_dir\venv312\Lib\site-packages"

& "C:\Program Files\Agisoft\Metashape Pro\metashape.exe" -r src\step2.py
```

Groups models by site and creates new PSX files organized by site in `output/psx/`.

### Manual Step: Straightening & Scaling Preparation

After Step 2, manually straighten each model and prepare for scaling:

1. Open each project in the `output/psx/` directory
2. For each chunk in the project:

   **Straightening (always required):**
   - Load the textured model
   - Auto-adjust brightness and contrast in one of the images to improve texture
   - Switch to rotate model view
   - Rotate the model so it aligns horizontally at the top of the view
   - Use "Model > Region > Rotate Region to View" to set the alignment
   - Resize the region to "crop" to the model area (use top XY and side views)
   - Use the rectangular crop tool to crop to the model area bounded by the region

   **Scaling Preparation:**
   - **For automatic scaling (Step 3a):** Ensure coded targets are visible and properly positioned
   - **For manual scaling (Step 3b):** Place markers on scale bars, set up at least 2 scale bars at different locations, set known distances in Reference pane, verify error < 0.01
   
3. Save the project and quit Metashape

### Step 3: Model Processing and Exports

This step processes models with scaling, removes small components, and exports orthomosaics, textured models, and reports using standardized file naming.

**Approach 1: Try Automatic Scaling First (Recommended, if coded targets present)**

```bash
# Run automatic scaling first (detects coded targets)
& "C:\Program Files\Agisoft\Metashape Pro\metashape.exe" -r src\step3.py
```

**If automatic scaling fails or no coded targets are present:**

**Approach 2: Reset and Use Manual Scaling**

```bash
# Reset to preserve Step 0&1 work, clear Step 2+ outputs
python src/utility/reset_step1.py {PROJECT_DIR}

# Re-run Step 2 (this is pretty quick)
$env:PYTHONPATH = "$project_dir\venv312\Lib\site-packages" 

& "C:\Program Files\Agisoft\Metashape Pro\metashape.exe" -r src\step2.py

# Manually straighten and add scale bars in Metashape GUI (see Manual Step above)
# Then run manual scale processing:

& "C:\Program Files\Agisoft\Metashape Pro\metashape.exe" -r src\step3.py
```

**Alternative: Start with Manual Scaling**
If you know there are no coded targets or prefer manual scaling from the start, skip `step3.py` and go directly to the manual workflow above.

**Both approaches produce identical outputs:**
- Models: `output/models/{MODEL_ID}/{MODEL_ID}.obj` 
- Reports: `processing/reports/{MODEL_ID}.pdf`

### Manual Step: Model Review and Touchups

After Step 3, manually review and touch up the models:

1. Open each project in Metashape
2. For each chunk: review orthomosaic and textured model quality, fill small holes if needed, adjust texture blending, verify scale bars and small component removal
3. Save the project

## Configuration

Edit `{PROJECT_DIR}/analysis_params.yaml` for project-specific settings. Scripts automatically load configurations from this file, enabling processing of different projects without code modifications.

### Key Configuration Parameters

- **max_chunks_per_psx**: Maximum number of chunks/models per PSX file (default: 5)
- **reconstruction_uncertainty**: Maximum allowable reconstruction uncertainty (default: 50)
- **reprojection_error**: Maximum reprojection error in pixels (default: 1)
- **projection_accuracy**: Maximum projection accuracy value (default: 10)
- **depth_downscale**: Downscale factor for depth maps (default: 4)
- **texture_size**: Texture size in pixels (default: 16384)

See the configuration files for complete parameter descriptions.

## Utility Scripts

These scripts provide helpful utilities for managing the processing environment.

### `src/utility/reset_full.py`

**Complete Project Reset** - Resets project to **BEFORE Step 0** (frame extraction).

**What it does:**
- 🗑️ Empties `processing/` and `output/` directories completely
- 📁 **Keeps** empty folder structure (`processing/`, `output/` directories remain)
- 🗑️ Removes all tracking CSV files
- 🔒 **Preserves:** `video_source/`, `analysis_params.yaml`, `.venv/`

**Usage:**
```bash
python src/utility/reset_full.py /path/to/project
```

**When to use:** Starting completely over from the beginning (frame extraction).

### `src/utility/reset_step1.py`

**Reset After Step 1** - Preserves Steps 0 & 1, clears Steps 2+ outputs.

**What it PRESERVES (the time-consuming work):**
- 🔒 Step 0: Extracted frames (`processing/frames/`)
- 🔒 Step 1: PSX files (`processing/psxraw/`)  
- 🔒 Step 0 & Step 1 tracking status

**What it CLEARS:**
- 🗑️ Step 2+: All `output/` directory contents (consolidated PSX, orthomosaics, models, reports)
- 🗑️ Step 2+ tracking status (resets to "Step 1 complete")

**Usage:**
```bash
python src/utility/reset_step1.py /path/to/project
```

**When to use:** Re-running Step 2 (chunk management) and subsequent steps while preserving hours of Step 0 & Step 1 processing time.

### `src/utility/enumerate_gpus.py`

This Python script lists the available GPUs that Metashape can detect and use. This is useful for verifying GPU configuration and ensuring Metashape is utilizing the expected hardware acceleration.

**Usage:**
```bash
# Run using Metashape's Python environment
C:\Program Files\Agisoft\Metashape Pro\metashape.exe" -r src/utility/enumerate_gpus.py
```

## Troubleshooting

### Common Issues

1. **Package Import Errors in Metashape**
   
   If you encounter import errors for packages like numpy, pandas, or PyYAML when running scripts through Metashape, you'll need to install these packages in Metashape's Python environment. Use the provided installation scripts:
   
   ```bash
   # On Windows
   C:\Program Files\Agisoft\Metashape Pro\metashape.exe" pip install -r requirements.txt
   ```

2. **PSX files not generated**
   
   Ensure that the `psxraw` directory exists and is writable. Check the log file in the `reports` directory for error messages.

3. **"Module 'numpy' has no attribute 'bool'"**
   
   This error typically occurs with incompatible numpy versions. Use our installation scripts to install the compatible version in Metashape's Python environment.

4. **Metashape Python version mismatch**
   
   If your Metashape version uses a different Python version than 3.12, you may need to modify the installation scripts to point to the correct Python interpreter.
   
5. **Metashape unable to communicate with Python environment**
  If you use a hard drive, Agisoft Metashape must be in the same path as your project directory. Use your device's local storage or Cloud or add Metashape in your path.
  
6. **Inaccurate Model Replication **
  If your models can not be replicated, edit extraction_rate from `{PROJECT_DIR}/analysis_params.yaml`. If problem still occurs manually check frames from Step0.py and delete unnecessary/ low quality frames. 

## Field Methods

### Required Materials

- Camera system:
  - Camera with lights
  - Memory card (CF Express)
  - Camera housing
  - External battery pack
  - Strobe light batteries
  - Camera lens
  - Cinema camera gear
  - Handle with clips and rope

- Field equipment:
  - Scale bars (2)
  - Field box containing:
    - Extra towels
    - O-ring grease
    - Cleaning materials
    - Dry towels
  - Slate
  - Vacuum device for housing seal check

### Camera Setup and Maintenance

#### Regular Maintenance
- Camera cinema gear maintenance
- Camera settings verification
- Programmable button configuration
- Housing maintenance (every few weeks or if leaks detected):
  - O-ring greasing

#### Pre-Dive Preparation
1. Day before:
   - Check housing and o-rings
   - Charge camera
   - Charge external battery pack
   - Charge strobe light batteries
   - Initialize media on memory card

2. Morning of:
   - Camera sealing procedure:
     1. Install battery and memory card
     2. Attach lens and verify autofocus is on
     3. Remove lens cap and check for smudges
     4. Prepare housing for camera insertion
     5. Seat camera in housing using cinema camera gear
     6. Connect external battery
     7. Final housing checks:
        - Turn on alarm
        - Check for smudges on housing lens
        - Verify o-ring condition
        - Close housing
        - Use vacuum device until light turns green

   - Equipment verification:
     - Camera and memory card
     - Housing
     - Field box with supplies
     - Slate
     - Scale bars (2)
     - Handle with clips and rope

   - Camera settings verification:
     - CP file: C2 (Canon log 3 / C.Gamut Color matrix neutral)
     - Sensor mode: full frame
     - Frequency: 59.94hz
     - Recording: RAW LT
     - Destination: CFexpress
     - Frame rate: 59.94 fps

### In-Water Procedures

#### Start of Dive
1. **B**uttons: Press all buttons to prime them
2. **P**ower: 
   - Turn on camera and lights (hold in/out buttons 1s, press middle button)
   - Put lights to sleep (hold center 2s)
3. **L**eaks: Monitor green light - if turns red, return to boat

#### Transect and Camera Setup
1. **S**cale bars: 
   - Place at each end of transect (more or less)
   - One perpendicular-ish (45 deg angle), one parallel to transect
   - Ensure circular targets are **visible** in footage and that scale bars **never move** during filming (if they do, restart)

2. **T**ime code: Reset (Mode button)

3. **A**rms: Extend to position lights as far apart as possible

4. **L**ights: Turn on (hold Center Button 2 sec)

5. **W**hite balance: Press Button 13, hold camera over white part of scale bar

6. **E**xposure: 
   - Open WFM (Button 6) and false color (Button 9)
   - Use F-stop dial to slightly overexpose (just below 100% on WFM)

7. **A**ltitude: 
   - Position camera so viewfinder covers length of scale bar
   - Note height (should be ~70cm)
   - Maintain this altitude throughout filming

8. **R**ecord: 
   - Press Record button
   - Show transect number
   - Verify autofocus

#### Filming Protocol (4-Pass Method)
Each pass should be approximately 10 meters long and take about 1 minute, maintaining consistent altitude.

1. **Pass 1**: 
   - Start at one end
   - Camera facing straight down
   - Transect line visible in left quarter of viewfinder

2. **Pass 2**: 
   - Turn around
   - Camera facing straight down
   - Position slightly away from transect line
   - Viewfinder should see 1m distance from transect
   - Maintain ~0.5m overlap with Pass 1
   - Position approximately arm's length from transect

3. **Pass 3 & 4**: 
   - Move ~20cm from pass 1/2 position
   - Tilt camera 45°
   - Capture angled view of transect from either side

#### Post-Filming
- Press center button on each light for 2s to put lights to sleep

## Running a Sample Project

1. Clone this repository
2. Navigate to the sample project directory: `cd examples/sample_project`
3. Follow the setup instructions above
4. Run each step in sequence, performing the manual steps between automated ones
5. Check the output directories for results

## Acknowledgments

This workflow was developed by Lauren Olinger for the Territorial Coral Reef Monitoring Program. 