# User Manual: Estimate Road Geometry Plugin for QGIS

## 1. Introduction: The Importance of Road Geometry in Road Safety

Road geometry, which describes the alignment and shape of a road, is a critical factor in road safety analysis. Roads are composed of various geometric elements, primarily **curves** and **tangents (straight sections)**. The characteristics of these elements – such as curve radius, length, and orientation – directly influence driver behavior, vehicle dynamics, and ultimately, crash risk.

Accurate information about a road's geometric properties is essential for:
* Identifying high-risk road sections.
* Designing effective safety countermeasures.
* Evaluating the impact of road improvements.
* Developing predictive safety models.

Traditionally, geometric data is extracted from design documents or collected using specialized road surface profiler vehicles. However, in many regions or for older roads, such detailed information may be unavailable or outdated.

### The Role of the Estimate Road Geometry Plugin

This QGIS plugin, **"Estimate Road Geometry,"** addresses this challenge by enabling users to extract fundamental geometric properties (curve and tangent identification) directly from standard GIS line data representing roads. Even if the source GIS lines contain minor inaccuracies, the plugin is designed to process them and provide valuable insights for road safety analysis where other data sources are lacking.

The plugin processes a given road centerline (line string) and segments it into distinct **curve** and **tangent** sections, providing key attributes for each segment.

## 2. Installation and Accessing the Plugin

The "Estimate Road Geometry" plugin is a QGIS Processing algorithm. You can install it using a few methods:

**Method 1: Install from ZIP (Recommended for testing/distribution)**
1.  Obtain the plugin's `.zip` file.
2.  In QGIS, go to `Plugins > Manage and Install Plugins...`.
3.  In the Plugin Manager, go to the `Install from ZIP` tab.
4.  Browse to your plugin's `.zip` file and click `Install Plugin`.

**Method 2: Manual Installation (for development/advanced users)**
1.  Locate your QGIS plugins folder. This is typically found in your QGIS profile directory (e.g., `C:\Users\YOUR_USERNAME\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\` on Windows, or `~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/` on Linux/macOS).
2.  Copy the unzipped plugin folder (e.g., `RoadGeometryAnalyzer`) directly into this `plugins` directory.

**After Installation:**
Regardless of the installation method, you might need to restart QGIS. Then:
1.  Go to `Plugins > Manage and Install Plugins...`.
2.  In the `Installed` tab, ensure the **"Road Geometry Analyzer"** plugin is checked to activate it.

**Accessing the Algorithm:**
Once activated, you can find and run the algorithm in the Processing Toolbox:
1.  Open the **Processing Toolbox** panel in QGIS (if not already open, go to `Processing > Toolbox`).
2.  Navigate to the **"Road Safety Tools"** group.
3.  Expand the group and double-click on the **"Estimate Road Geometry"** algorithm.

*(**Note to Developer:** For optimal QGIS integration, consider setting the `name()` method in your code to `'estimate_road_geometry'` and the `groupId()` method to `'road_safety_tools'`. This ensures consistency in internal naming and display within the Processing Toolbox.)*

## 3. Plugin Parameters

The "Estimate Road Geometry" algorithm requires a line layer input and offers several parameters to control the analysis.

| Parameter                      | Description                                                                                                                                                                                                                                                                                                                                                                                                 | Type                     | Default Value |
| :----------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------- | :------------ |
| **Input layer** | The source vector layer containing the road centerline you wish to analyze. This layer must be a line geometry type. **Important:** This plugin is designed to process a *single, continuous road line*. Using a network of roads (multiple disconnected line features) as input may lead to incorrect or disconnected segments, as the algorithm attempts to process all features as a continuous sequence. you can find an sample line in /example folder.                                     | Vector Layer (Line)      | (Required)    |
| **Apply simplification before analysis?** | (Advanced) Check this box if you want to simplify the input line geometry before the curve/tangent analysis. Simplification can help correct minor errors or reduce noise in the input data, which might otherwise lead to many very small, spurious segments.                  | Checkbox (Boolean)       | No            |
| **Simplification method** | (Advanced, Optional) If simplification is enabled, select the algorithm to use for simplifying the line. Options include: <br> - **Distance (Douglas-Peucker):** A common method that removes vertices within a specified tolerance. <br> - **Snap to grid:** Snaps vertices to a grid. <br> - **Area (Visvalingam):** Simplifies based on triangle area. | Dropdown (Enum)          | Distance      |
| **Simplification tolerance** | (Advanced, Optional) If simplification is enabled, specify the maximum allowed offset for simplification. This value should be in the units of your input layer's Coordinate Reference System (CRS). A larger tolerance results in more aggressive simplification.                         | Number (Double)          | 1.0           |
| **Curve radius threshold** | This is the most crucial parameter for defining curves and tangents. Road segments with a calculated radius *smaller than or equal to* this threshold will be classified as a **"curve,"** while segments with a radius *greater than* this threshold will be classified as a **"tangent"** (straight). <br> The value should be in **meters**. A common threshold for distinguishing between curves and relatively straight sections might be in the hundreds or thousands of meters, depending on the road type and design standards. | Number (Double)          | 2000.0        |
| **Output layer** | Specify where the resulting segmented road layer will be saved. You can choose to save to a temporary layer (useful for quick analysis) or save to a file (e.g., GeoPackage, Shapefile) for permanent storage.                                                                     | Vector Layer (Line Sink) | (Temporary)   |

## 4. Understanding the Output Layer

The plugin generates a new line vector layer where the original road lines are segmented into distinct curve and tangent sections. Each segment in the output layer will have the following attributes:

| Field Name   | Type   | Description                                                                                                                                       | Examples                                 |
| :----------- | :----- | :------------------------------------------------------------------------------------------------------------------------------------------------ | :--------------------------------------- |
| `segment_id` | String | A unique identifier for each continuous geometric segment (curve or tangent). Segments with the same orientation will share an ID until the orientation changes significantly. | `1`, `2`, `3`                            |
| `type`       | String | Indicates whether the segment is classified as a "curve" or a "tangent." This is determined by the `Curve radius threshold` parameter.               | `curve`, `tangent`                       |
| `orientation`| String | Describes the direction of curvature for curve segments, or "straight" for tangent segments.                                                      | `clockwise`, `counterclockwise`, `straight` |
| `radius`     | Double | The calculated radius of curvature for the segment, expressed in **meters**. For straight segments (tangents), this value will be very large (effectively infinite). | `150.32`, `890.50`, `1000000.0`          |
| `angle`      | Double | The calculated total angle change across the segment, expressed in **radians**. This represents how sharply the road turns over that segment.    | `0.52` (approx 30 degrees), `0.0` (for tangents) |

## 5. How the Algorithm Works (A Short Insight)

The core of the "Estimate Road Geometry" plugin is an iterative process that analyzes the geometry of your input line layer:

1.  **Preprocessing (Optional Simplification):** If enabled, the input line is first simplified using the chosen method and tolerance. This helps to smooth out minor irregularities in the input data.
2.  **Vertex Extraction and Transformation:** The plugin extracts individual vertices from the (potentially simplified) input line. For accurate geometric calculations, these points are internally transformed to a projected Coordinate Reference System (CRS) suitable for distance and area measurements (specifically, EPSG:3857, a global Mercator projection, which uses meters).
3.  **Three-Point Analysis:** The algorithm iterates along the line, considering groups of three consecutive vertices at a time. For each triplet of points, it calculates:
    * The **radius** of the circle that passes through these three points.
    * The **angle** formed by these three points.
    * The **orientation** of the curve (clockwise, counterclockwise) or if it's straight, which is directly derived based on the calculated angle.
4.  **Segmentation:** Based on the calculated radius, angle, and orientation, the plugin determines if a small segment between two consecutive points is part of a curve or a tangent. **A new `segment_id` will be generated if any change is detected in the continuous orientation** (e.g., changing from clockwise to counterclockwise, or from curve to tangent).
5.  **Aggregation:** All these small, individual segments are then grouped together by their `segment_id` and common `type` and `orientation`. For each aggregated segment, the mean radius and angle are calculated to provide a representative value for that entire curve or tangent section.
6.  **Re-projection:** Finally, the resulting segmented lines are re-projected back to the original CRS of your input layer before being saved to the output.

This process allows the plugin to effectively break down complex road geometries into meaningful, measurable segments for in-depth road safety analysis.

## 6. Tips for Best Results

* **Input Data Quality:** While the plugin can handle some errors, cleaner input line data will generally yield more accurate and reliable results.
* **Single Road Line:** Always ensure your input layer represents a single, continuous road segment, not a complex network.
* **Coordinate Reference System (CRS):**
    * The `Curve radius threshold` parameter *always* expects values in **meters**.
    * The `Simplification tolerance` parameter, however, depends on the units of your **input layer's CRS**.
    * For the most reliable and consistent results, **it is highly recommended to use an input layer that is already in a metric projected CRS, ideally EPSG:3857 (WGS 84 / Pseudo-Mercator)**. This CRS has been thoroughly tested with the plugin's internal calculations and ensures direct compatibility with the metric `radius_threshold`.
* **Radius Threshold:** Experiment with the `Curve radius threshold` parameter. The optimal value can vary significantly depending on the specific road network, local design standards, and the level of detail required for your analysis. Lower values will identify sharper curves, while higher values will group gentler curves with straight sections.
* **Simplification:** Use the simplification options cautiously. While helpful for noisy data, overly aggressive simplification can remove genuine geometric detail.
