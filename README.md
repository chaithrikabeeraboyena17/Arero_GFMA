# GeoMeasure: A Geospatial File Measurement System

**Project assignment report**

## Project overview

GeoMeasure is a web application for processing geographic data and reviewing measurements for its features. A user uploads a KML file or a ZIP archive containing a Shapefile. The application reads the geometries and attributes, calculates area or length where supported, stores the processed results, and presents them in a map and feature table.

The project brings together geospatial data processing, a REST API, database storage, and a browser interface in one working application.

## Problem statement

Geographic datasets often contain many boundaries and routes. Measuring each polygon or line manually takes time and can lead to inconsistent results. GeoMeasure automates these measurements and gives users a practical way to inspect the results alongside the source features.

## Project objectives

- Accept commonly used KML and Shapefile data.
- Read feature geometry and associated attributes.
- Calculate area for polygon features and length for line features.
- Respect coordinate reference systems when transforming and measuring geometry.
- Store processed datasets and feature results for later retrieval.
- Present results through an interactive map, a table, and downloadable files.

## How the application works

```text
User uploads a file
        ↓
FastAPI validates and parses the data
        ↓
Geometries are transformed and measured
        ↓
Processed results are saved in the database
        ↓
The browser displays the map, table, and downloads
```

The upload endpoint accepts `.kml` files and `.zip` archives containing one Shapefile. KML coordinates are interpreted in WGS 84. For Shapefiles, the `.prj` file supplies the coordinate reference system when available. The application transforms features to WGS 84 for map display and selects a local UTM coordinate system for planar area and length calculations.

## Main features

- Upload KML or zipped Shapefile data through the browser.
- Display supported point, line, and polygon features on a Leaflet map.
- Calculate line length in metres and polygon area in square metres.
- Inspect feature attributes, coordinate reference information, measurements, and processing errors.
- Select features from the map or table to connect the two views.
- Search, filter, and sort features in the dashboard.
- Export processed results as CSV or GeoJSON.
- Persist dataset metadata and feature results in a MySQL-compatible database.
- Apply upload and archive checks, including a 50 MiB upload limit.

Points can be displayed but do not receive an area or length measurement. Individual features with unsupported, invalid, or unmeasurable geometry can still be returned with an explanatory processing error.

## Technology used

| Area | Tools |
|---|---|
| Backend and API | Python, FastAPI, Uvicorn |
| Geometry processing | Shapely, pyproj, pyshp |
| Database | MySQL-compatible database through PyMySQL |
| Frontend | HTML, CSS, and JavaScript |
| Map | Leaflet and OpenStreetMap/Esri map tiles |

The interface is served directly by FastAPI, so a separate frontend build step is not required. Python dependencies are listed in `Arero_QGIS/server/requirements.txt`.

## Project structure

```text
.
├── README.md
└── Arero_QGIS/
    └── server/
        ├── requirements.txt
        └── app/
            ├── main.py
            ├── routes/files.py
            ├── schemas/file.py
            ├── services/
            │   ├── file_service.py
            │   ├── measurement_service.py
            │   └── storage_service.py
            ├── static/index.html
            └── utils/database.py
```

## Run locally

Requirements: Python 3.10 or later, a configured MySQL-compatible database, and browser internet access for map assets and tiles.

From the repository root, run these commands in PowerShell:

```powershell
cd Arero_QGIS/server
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --reload
```

Then open the application at <http://127.0.0.1:8000/>. Interactive API documentation is available at <http://127.0.0.1:8000/docs>.

Database connection settings must be provided through the environment variables expected by `app/utils/database.py`. The database must be reachable for health checks, uploads, retrieval, and exports.

## API summary

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/api/health` | Check the database connection and initialize application tables. |
| `POST` | `/api/files/` | Upload, process, and save a KML or zipped Shapefile. |
| `GET` | `/api/files/{file_id}/` | Retrieve dataset metadata. |
| `GET` | `/api/files/{file_id}/measurements/` | Retrieve saved feature results. |
| `GET` | `/api/files/{file_id}/export.csv` | Download feature data as CSV. |
| `GET` | `/api/files/{file_id}/export.geojson` | Download mapped features as GeoJSON. |

## Measurement approach and limitations

Area and length are calculated after projecting each feature into a UTM coordinate system selected from a representative point. This provides useful local measurements for many regional datasets, but results may be less suitable for features spanning large distances, crossing UTM zones, or lying outside the supported UTM latitude range. The application does not perform geodesic measurements or repair invalid geometries.

A Shapefile should include its `.shp`, `.shx`, and `.dbf` components. A matching `.prj` file is recommended so the source coordinate system is known. The original upload is processed in temporary storage and is not retained as a source file. Processing occurs during the upload request; background jobs and progress reporting are not implemented.

## Conclusion

GeoMeasure demonstrates an end-to-end geospatial workflow: file upload, format parsing, coordinate transformation, feature measurement, database persistence, API access, and interactive visualization. It reduces repetitive manual work while making measurement results available for inspection and export.

## Submission notes

This repository does not include a license file or automated test suite. The project uses external browser services for map assets and tiles. These points can be considered when extending or deploying the application.
