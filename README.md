# Road Geometry Analyzer QGIS Plugin

## Estimate Road Geometry from GIS Lines for Road Safety Analysis

![QGIS Plugin Icon Placeholder](path/to/your/plugin_icon.png)
*(**Note:** Replace `path/to/your/plugin_icon.png` with the actual path to your plugin's icon file in your repository, or remove this line if you don't have one.)*

## 1. Overview

The **Road Geometry Analyzer** is a QGIS Processing plugin designed to automatically segment road centerlines into distinct **curve** and **tangent (straight)** sections. It addresses the common challenge of lacking detailed road design documents or specialized survey data for geometric analysis. By leveraging existing GIS line data, this plugin provides essential geometric attributes crucial for in-depth road safety studies, infrastructure management, and planning.

## 2. Key Features

* **Curve & Tangent Detection:** Automatically identifies and segments road lines into curved and straight portions.
* **Geometric Attributes:** Calculates and provides `radius`, `angle`, `type` (curve/tangent), and `orientation` (clockwise/counterclockwise/straight) for each segment.
* **Data Simplification:** Includes optional pre-processing tools to simplify input lines, helping to refine results from noisy or overly detailed GIS data.
* **Metric Analysis:** Performs internal calculations in a robust metric projection (EPSG:3857) to ensure accurate radius measurements, regardless of the input layer's CRS.
* **Road Safety Focus:** Designed specifically to support road safety analysis where traditional geometric data sources are unavailable.

## 3. Installation

You can install the **Road Geometry Analyzer** plugin in QGIS using one of the following methods:

### Method A: Install from ZIP (Recommended)

1.  Download the plugin's `.zip` file from the [releases page](LINK_TO_YOUR_RELEASES_PAGE_HERE).
2.  Open QGIS.
3.  Go to `Plugins > Manage and Install Plugins...`.
4.  Switch to the `Install from ZIP` tab.
5.  Browse to the downloaded `.zip` file and click `Install Plugin`.

### Method B: Manual Installation (for Developers/Advanced Users)

1.  Clone this repository or download the source code as a ZIP and extract it.
2.  Locate your QGIS plugins folder. This is typically found in your QGIS profile directory (e.g., `C:\Users\YOUR_USERNAME\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\` on Windows, or `~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/` on Linux/macOS).
3.  Copy the `RoadGeometryAnalyzer` folder (containing `RoadGeometryAnalyzer.py`, `metadata.txt`, etc.) directly into this `plugins` directory.
4.  Restart QGIS.

### Activation

After installation (and potential QGIS restart):
1.  Go to `Plugins > Manage and Install Plugins...`.
2.  In the `Installed` tab, ensure the **"Road Geometry Analyzer"** plugin is checked to activate it.

## 4. Usage

The plugin is available as a Processing algorithm:

1.  Open the **Processing Toolbox** panel (`Processing > Toolbox`).
2.  Expand the **"Road Safety Tools"** group.
3.  Double-click on the **"Estimate Road Geometry"** algorithm.

### Parameters:

* **Input layer:** A single, continuous line vector layer representing a road. you can find an sample line in /example folder.
* **(Advanced) Apply simplification before analysis?:** (Checkbox) Choose to preprocess the line.
* **(Advanced) Simplification method:** (Dropdown) Method if simplification is enabled.
* **(Advanced) Simplification tolerance:** (Number) Tolerance for simplification (in input layer CRS units).
* **Curve radius threshold:** (Number) The crucial threshold in **meters** for classifying a segment as "curve" (radius $\le$ threshold) or "tangent" (radius $>$ threshold). Default: 2000 meters.
* **Output layer:** Destination for the resulting segmented line layer.

## 5. Output

The output is a new line vector layer with segmented road features, each containing the following attributes:

| Field Name | Type | Description |
| :--------- | :--- | :----------------------------------------------------- |
| `segment_id` | String | Unique ID for each continuous geometric segment. |
| `type` | String | "curve" or "tangent". |
| `orientation` | String | "clockwise", "counterclockwise", or "straight". |
| `radius` | Double | Calculated radius of curvature in **meters**. |
| `angle` | Double | Calculated total angle change in **radians**. |

## 6. Tips for Best Results

* **Single Road Line:** Ensure your input layer contains a single, continuous road line, not a network.
* **Metric CRS Recommended:** While the plugin handles CRS transformations internally, using an input layer already in a metric projected CRS (e.g., **EPSG:3857**) is highly recommended for consistent and reliable results, especially concerning the `Curve radius threshold`.
* **Adjust Radius Threshold:** Experiment with the `Curve radius threshold` to best suit your data and analysis needs.
* **Simplification:** Use the simplification options cautiously. While helpful for noisy data, overly aggressive simplification can remove genuine geometric detail.

## 7. License

This plugin is licensed under the GNU General Public License version 2 or any later version (GPLv2+). See the `LICENSE.txt` file for more details.

## 8. Author

**Hamid Madani**
* Email: hamidmadani419@gmail.com
* Copyright: (C) 2025 by Hamid Madani
