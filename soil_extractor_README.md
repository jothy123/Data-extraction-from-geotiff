# 🌍 Multi-Source Soil & Environmental Parameter Extractor

> Extract 9 environmental variables from multi-CRS raster datasets at any geographic coordinate — with intelligent nearest-neighbour fallback for nodata handling.

![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat&logo=python&logoColor=white)
![rasterio](https://img.shields.io/badge/rasterio-1.3+-green?style=flat)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat)
![Status](https://img.shields.io/badge/Status-Active-1F6B4A?style=flat)

---

## 📌 Overview

This tool extracts a comprehensive set of soil and environmental parameters from large geospatial raster datasets (EPSG:4326 and EPSG:3035) given a single latitude/longitude input. It was developed to support field-level environmental impact assessment and agriculture emissions modelling pipelines.

Key design features:
- **Dual CRS support** — handles 4326 (global) and 3035 (European LDEM/soil) rasters seamlessly
- **Memory-efficient large raster handling** — reads only the required pixel using windowed I/O, avoiding full raster loads
- **Intelligent nodata fallback** — KDTree nearest-neighbour (small rasters) and expanding window search (large rasters)
- **Transparent output** — reports whether each value was extracted exactly or via nearest-neighbour

---

## 🗺️ Parameters Extracted

| Parameter | Source CRS | Raster Source | Unit |
|---|---|---|---|
| Soil pH (CaCl₂) | EPSG:4326 | SoilGrids / EU Soil | — |
| Köppen Climate Zone | EPSG:4326 | Köppen-Geiger 0.0083° | Class code |
| K-Factor (erosion) | EPSG:4326 | RUSLE K-factor | t·ha·h / ha·MJ·mm |
| Terrain Slope | EPSG:3035 | EU-DEM v1.1 | Degrees |
| Clay Content | EPSG:3035 | EU Soil Hydraulic DB | % |
| Curve Number (CN) | EPSG:3035 | Hydrological CN grid | — |
| Bulk Density | EPSG:3035 | EU Soil DB | g/cm³ |
| Soil Nitrogen (N) | EPSG:3035 | EU Nutrient grid | g/kg |
| Soil Phosphorus (P) | EPSG:3035 | EU Nutrient grid | mg/kg |

---

## 🧠 Technical Architecture

```
Input: lat/lon (EPSG:4326)
        │
        ├─► Water mask check → exit if WATER
        │
        ├─► Reproject to EPSG:4326 → extract pH, Köppen, K-factor
        │       └── KDTree fallback if nodata
        │
        └─► Reproject to EPSG:3035 → extract slope, clay, CN, bulk, N, P
                └── Expanding window search fallback if nodata
```

### Nodata Fallback Strategy

| Raster Type | Strategy | Reason |
|---|---|---|
| Small rasters (4326) | **KDTree nearest-neighbour** — builds spatial index over all valid pixels | Fast and spatially accurate for fully loaded arrays |
| Large rasters (3035) | **Expanding window search** (radius 1→10 pixels) | Avoids loading entire large raster; memory-efficient |

---

## 🚀 Getting Started

### Prerequisites

```bash
pip install rasterio pyproj numpy scipy
```

### Required Raster Files

Place the following rasters in the paths specified (or update paths in the script):

**EPSG:4326 rasters** (`/your/path/4326/`):
- `pH_CaCl_4326.tif`
- `water_mask_whole_reprojected_EU.tif`
- `koppen_geiger_0p00833333.tif`
- `K-Factor.tif`

**EPSG:3035 rasters** (`/your/path/3035/`):
- `eudem_slop_3035_europe.tif`
- `Clay.tif`, `CN.tif`, `Bulk_density.tif`
- `N.tif`, `P.tif`

### Run

```bash
python soil_extractor.py
```

```
Enter latitude: 51.4545
Enter longitude: -2.5879

Location is LAND

Final Result
pH: 5.8
Köppen: 13
K-factor: 0.032
Slope: 4.2
Clay: 22.1
CN Ratio: 75
Bulk density: 1.34
Soil Nitrogen (N): 1.8
Soil Phosphorus (P): 42.0

Pixel usage:
4326: Exact
Slope: Exact
Clay: Nearest
...
```

---

## 📁 Project Structure

```
soil-parameter-extractor/
├── soil_extractor.py       # Main extraction script
├── README.md               # This file
├── requirements.txt        # Python dependencies
├── sample_output.txt       # Example run output
└── docs/
    └── architecture.png    # Data flow diagram
```

---

## 🔬 Research Context

This tool was developed as part of a **field-level agriculture emissions monitoring system** for [Nature Preserve ApS](https://naturepreserve.co), designed to feed soil parameters into IPCC Tier-2/3 emissions models. It is intended for use in large-scale environmental monitoring pipelines where manual raster extraction is impractical.

**Related work:**
- Integrated into a cloud-hosted AWS Web GIS platform for near-real-time carbon footprint assessment
- Designed to support IPCC Tier-2 and Tier-3 agriculture emissions accounting workflows

---

## 🤝 Contributing

Pull requests are welcome. For major changes, please open an issue first.

If you use this in research, a citation or acknowledgement is appreciated.

---

## 📜 License

MIT License — free to use, modify, and distribute with attribution.

---

## 👤 Author

**Navajyothy Navendran**
Environmental Engineer | GIS & Remote Sensing
[LinkedIn](https://linkedin.com/in/navajyothy-navendran) · [navasjothy@gmail.com](mailto:navasjothy@gmail.com)
