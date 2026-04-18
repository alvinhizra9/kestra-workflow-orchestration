# Kestra Workflow Orchestration

A comprehensive workflow orchestration project using Kestra to manage ETL (Extract, Transform, Load) processes for NYC taxi data on Google Cloud Platform (GCP).

## Project Overview

This project demonstrates how to use Kestra for orchestrating data pipelines that extract taxi data from public sources, transform it, and load it into Google BigQuery tables. The workflow includes:

- Setting up GCP environment variables
- Creating GCS buckets and BigQuery datasets
- Scheduled extraction and processing of NYC taxi data (Yellow and Green taxis)
- Data deduplication and merging into partitioned tables

## Workflow Structure

The project consists of three main workflow files:

### 1. `01_set_kv.yaml`
- **Purpose**: Sets up key-value pairs for GCP configuration
- **Tasks**:
  - Sets GCP project ID
  - Sets GCP location (asia-southeast2)
  - Sets GCS bucket name
  - Sets BigQuery dataset name

### 2. `02_gcp_setup.yaml`
- **Purpose**: Creates necessary GCP resources
- **Tasks**:
  - Creates GCS bucket (if it doesn't exist)
  - Creates BigQuery dataset (if it doesn't exist)
- **Plugin Defaults**: Configures GCP plugin settings for all subsequent tasks

### 3. `03_gcp_taxi_scheduled.yaml`
- **Purpose**: Main ETL workflow for taxi data processing
- **Features**:
  - Supports both Yellow and Green taxi data
  - Scheduled to run monthly on the 1st of each month
  - Extracts data from NYC TLC Data releases
  - Uploads to GCS
  - Creates external and temporary BigQuery tables
  - Merges data into partitioned tables with deduplication
  - Automatically cleans up temporary files

## Prerequisites

1. **Kestra Instance**: Access to a Kestra instance (local or cloud)
2. **GCP Account**: Google Cloud Platform account with appropriate permissions
3. **GCP Service Account**: Service account with permissions for:
   - Cloud Storage (GCS)
   - BigQuery
   - (Optional) Cloud Logging for monitoring

## Setup Instructions

### 1. Configure GCP Credentials

Set your GCP service account credentials in Kstra's configuration:
```yaml
# In Kstra's configuration or environment variables
GCP_CREDS: "your-service-account-json-content"
```

### 2. Import Workflows

Import the YAML files into your Kestra instance in the following order:
1. `01_set_kv.yaml`
2. `02_gcp_setup.yaml`
3. `03_gcp_taxi_scheduled.yaml`

### 3. Execute Initial Setup

1. Run `01_set_kv.yaml` to set up configuration values
2. Run `02_gcp_setup.yaml` to create GCP resources
3. Verify that the bucket and dataset were created successfully

## Usage

### Manual Execution

1. Navigate to the `03_gcp_taxi_scheduled.yaml` workflow in Kstra UI
2. Click "Run" and select the taxi type (Yellow or Green)
3. The workflow will:
   - Download the specified taxi data for the current month
   - Upload to GCS
   - Process and load data into BigQuery

### Scheduled Execution

The workflow is configured with two scheduled triggers:
- **Green Taxi**: Runs on the 1st of each month at 9:00 AM
- **Yellow Taxi**: Runs on the 1st of each month at 10:00 AM

### Backfill Execution

To process historical data:
1. Add a label `backfill:true` when creating executions from the UI
2. This helps track backfill executions vs. scheduled ones

## Data Sources

The workflow downloads taxi data from [NYC TLC Data releases](https://github.com/DataTalksClub/nyc-tlc-data/releases):
- Yellow taxi data: `yellow_tripdata_YYYY-MM.csv.gz`
- Green taxi data: `green_tripdata_YYYY-MM.csv.gz`

## BigQuery Table Structure

### Yellow Taxi Table
```sql
CREATE TABLE `project_id.dataset.yellow_tripdata` (
    unique_row_id BYTES,
    filename STRING,
    VendorID STRING,
    tpep_pickup_datetime TIMESTAMP,
    tpep_dropoff_datetime TIMESTAMP,
    passenger_count INTEGER,
    trip_distance NUMERIC,
    RatecodeID STRING,
    store_and_fwd_flag STRING,
    PULocationID STRING,
    DOLocationID STRING,
    payment_type INTEGER,
    fare_amount NUMERIC,
    extra NUMERIC,
    mta_tax NUMERIC,
    tip_amount NUMERIC,
    tolls_amount NUMERIC,
    improvement_surcharge NUMERIC,
    total_amount NUMERIC,
    congestion_surcharge NUMERIC
)
PARTITION BY DATE(tpep_pickup_datetime);
```

### Green Taxi Table
```sql
CREATE TABLE `project_id.dataset.green_tripdata` (
    unique_row_id BYTES,
    filename STRING,
    VendorID STRING,
    lpep_pickup_datetime TIMESTAMP,
    lpep_dropoff_datetime TIMESTAMP,
    store_and_fwd_flag STRING,
    RatecodeID STRING,
    PULocationID STRING,
    DOLocationID STRING,
    passenger_count INTEGER,
    trip_distance NUMERIC,
    fare_amount NUMERIC,
    extra NUMERIC,
    mta_tax NUMERIC,
    tip_amount NUMERIC,
    tolls_amount NUMERIC,
    ehail_fee NUMERIC,
    improvement_surcharge NUMERIC,
    total_amount NUMERIC,
    payment_type INTEGER,
    trip_type STRING,
    congestion_surcharge NUMERIC
)
PARTITION BY DATE(lpep_pickup_datetime);
```

## Monitoring and Logging

- All executions are logged in Kstra's UI
- Each execution includes labels for:
  - File name being processed
  - Taxi type (yellow/green)
- For backfill executions, the `backfill:true` label helps distinguish them from scheduled runs

## Cleanup

The workflow automatically purges temporary files after execution to avoid storage clutter.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Support

For issues or questions:
1. Check the Kstra documentation
2. Review the workflow execution logs
3. Ensure GCP credentials have the necessary permissions