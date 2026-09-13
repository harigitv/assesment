HDB Resale Price Data Engineering ETL

Overview:

This project implements an ETL pipeline for HDB resale flat price data covering January 2012 to December 2016.

The pipeline ingests the source CSV files, combines the datasets into a common structure, validates records using the January 2012 dataset as the reference, handles duplicate records, identifies potentially anomalous resale prices, calculates remaining lease, generates a Resale Identifier, and creates a SHA-256 hashed identifier.

The pipeline is implemented in Python using Pandas and is designed to be reproducible without manually modifying the source datasets.

Project Structure


Assessment/
├── data/
│   ├── input/
│   └── output/
│       └── raw/
├── notebooks/
│   └── etl_pipeline.ipynb
└── README.md


Approach :
Ingest raw CSVs from data.gov.sg.

Apply validation rules to ensure consistency across towns, flat types, models, and storey ranges.

Deduplicate records using composite keys, retaining the highest resale price.

Flag potential anomalies using the IQR method.

Calculate remaining lease based on a standard 99‑year HDB lease.

Generate unique resale identifiers and hashed versions for auditability.

Save cleaned, invalid, duplicate, and final datasets separately.

Assumptions:
HDB flats have a 99‑year lease starting from the lease_commence_date.

Duplicate records are defined by identical composite keys (month, town, block, flat_type, floor_area, flat_model).

For duplicates, the highest resale price is retained.

Anomalies are flagged but not deleted, preserving audit integrity.

Validation Rules

Month validation → Only transactions between January 2012 and December 2016 are included.

Reference checks → Town, flat type, flat model, and storey range must match the January 2012 reference dataset.

Invalid records handling → Any record failing a validation rule is marked invalid and stored separately in hdb_resale_invalid.csv 

 Duplicate Handling

Composite key uses all attributes except resale_price.
Duplicate groups are identified using the composite key.
For duplicate groups, the record with the highest resale price is retained.
2,748 records were identified as part of duplicate groups.
1,388 lower-priced duplicate records were removed.
Duplicate records identified before resolution are stored in hdb_resale_duplicates.csv.

Anomaly Detection

Records are grouped by month and flat type.
The IQR rule is applied:

  Lower bound = Q1 − 1.5 × IQR
  Upper bound = Q3 + 1.5 × IQR
  5,434 potential anomalies were flagged in the price_anomaly column.
  Anomalies are retained in the final dataset for review.

Remaining Lease Calculation

Lease end = lease_commence_date + 99 years.
Remaining lease is calculated based on the transaction date.
The result is stored as X years Y months in remaining_lease.

 Identifiers

Resale Identifier: Generated according to the identifier format specified in the assignment.
SHA-256 hash: A SHA-256 hashed version of the Resale Identifier is generated for irreversible identifier transformation.

 Output Files

raw/ → source datasets used by the pipeline.
hdb_resale_cleaned.csv → cleaned and deduplicated records with anomaly flags.
hdb_resale_invalid.csv → records failing validation.
hdb_resale_duplicates.csv → records identified as part of duplicate groups before resolution.
hdb_resale_final.csv → final dataset containing the calculated lease, anomaly flag, Resale Identifier, and hashed identifier.
