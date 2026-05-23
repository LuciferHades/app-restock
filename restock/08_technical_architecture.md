# Technical Architecture

## Recommended Technology Stack
The application is built using a modern, lightweight stack optimized for data processing and single-user performance.

| Layer | Technology | Reason |
| :--- | :--- | :--- |
| **Frontend** | React + Vite + TailwindCSS | Fast development, responsive UI, and efficient state management. |
| **Data Tables** | TanStack Table | Handles large datasets (50k+ rows) with high performance. |
| **Backend** | FastAPI (Python) | High-performance asynchronous API with excellent data processing libraries. |
| **Data Engine** | Pandas / Polars | Industry-standard for complex data manipulation and aggregation. |
| **Excel Handling** | openpyxl / xlrd | Robust support for reading and writing `.xlsx` and `.xls` files. |
| **Storage** | Local File System | Simple and sufficient for single-user job-based storage. |

## System Components and Folder Structure
```text
/backend
  main.py                # API Entry point
  services/
    upload_service.py    # File handling and job creation
    validation_service.py # Data integrity checks
    mapping_service.py    # ItemKey normalization
    reconciliation_engine.py # Core logic
    t_plus_1_engine.py    # Movement impact analysis
    export_service.py    # Excel generation
  data/
    uploads/             # Raw uploaded files
    jobs/                # Processed job artifacts
    exports/             # Generated result files

/frontend
  src/
    pages/               # Main view components
    components/          # Reusable UI elements
    services/            # API communication layer
    stores/              # Global state (e.g., currentJobId)
```

## Processing Design
1. **File Ingestion**: The backend receives the file, generates a `job_id`, and saves the raw file.
2. **Data Loading**: Pandas reads the workbook into memory, applying initial normalization.
3. **Validation**: The system runs a suite of checks, returning any failures to the frontend immediately.
4. **Core Processing**: The engine builds normalized dataframes, performs aggregations, and runs the reconciliation logic.
5. **Artifact Storage**: Results are saved as JSON or Parquet files within the job folder for fast retrieval.
6. **Export**: On request, the system compiles the results into a formatted Excel workbook.

## Performance Requirements
- **Scalability**: Must process 50,000+ In-Out rows in under 10 seconds.
- **Efficiency**: Use vectorized operations in Pandas; avoid nested loops for lookups.
- **Memory**: Optimize data types to ensure the application runs smoothly on standard hardware.
