# IDS 706 — Week 9 AWS Basic Practice

**Course:** IDS 706 — Data Engineering Systems  
**Assignment:** Week 9 AWS Basic Practice  
**Platform:** AWS Serverless Data Lake Day workshop (US East / `us-east-1`)

## Overview

This assignment explores a serverless AWS data lake workflow, from generating and streaming records into Amazon S3 to discovering schemas, transforming data, and querying an external dataset with SQL. The work covers Amazon Data Firehose, Amazon S3, AWS Glue, AWS Glue Studio, Glue interactive notebooks, and Amazon Athena.

## Architecture

```text
Kinesis Data Generator
        |
        v
Amazon Data Firehose --> Amazon S3 (raw JSON/GZIP)
                                |
                                v
                        AWS Glue Crawler
                                |
                                v
                        Glue Data Catalog
                                |
                     +----------+----------+
                     |                     |
                     v                     v
              Glue Studio ETL       Glue Notebook ETL
                     |                     |
                     v                     v
                S3 Parquet            S3 Parquet

GDELT data in S3 --> Athena external tables / Glue Data Catalog --> SQL analysis
```

## Lab Exercises and Evidence

### Real-Time Data Streaming into Amazon S3 (35 points)

I configured an Amazon Data Firehose delivery stream named `sdl-firehose-stream` to ingest synthetic e-commerce records and deliver them to Amazon S3. I used AWS CloudFormation to set up access to the Kinesis Data Generator (KDG), tested a JSON record template, and streamed records to Firehose. The stream wrote compressed `.gz` objects to a date-partitioned `raw/` prefix in the S3 bucket. The monitoring view and stored objects document successful ingestion and delivery.

**Screenshots**

![CloudFormation Stack](screenshots/CloudFormation_Stack.png)

![KDG Template Test](screenshots/KDG_Template_Test.png)

![KDG Streaming](screenshots/KDG_Streaming.png)

![Firehose Monitoring](screenshots/Firehose_Monitoring.png)

![S3 Streaming Data](screenshots/S3_Streaming_Data.png)

### Create a Glue Crawler to Auto-Discover Schema (10 points)

I created and ran the AWS Glue crawler `sdl-demo-crawler` against the raw S3 data. The crawler completed successfully, created the `raw` table in the Glue Data Catalog database `sdl-demo-data`, and inferred fields and partition columns from the ingested records.

**Screenshots**

![Glue Crawler Success](screenshots/Glue_Crawler_Success.png)

![Glue Discovered Schema](screenshots/Glue_Discovered_Schema.png)

### Transform Data with AWS Glue Studio (15 points)

I built a visual AWS Glue Studio ETL job named `transform-json-to-parquet`. The pipeline reads the cataloged raw records, applies a **Change Schema** transformation (including renaming date fields and removing `color`), and writes **Snappy-compressed Parquet** files to `compressed-parquet/` in S3. The job completed successfully and the output files were verified in S3.

**Screenshots**

![Glue Transformation Pipeline](screenshots/Glue_Transformation_Pipeline.png)

![Glue Job Succeeded](screenshots/Glue_Job_Succeeded.png)

![S3 Parquet Output](screenshots/S3_Parquet_Output.png)

### Develop and Test ETL in a Glue Jupyter Notebook (15 points)

I used an AWS Glue interactive notebook for additional data preparation. The notebook loaded the cataloged dataset, inspected the schema and record count, dropped the `color` and `hour` fields, renamed `imageUrl` to `thumbnailImageUrl` and `campaign` to `campaignType`, and exported Parquet data to `output-etl-nb-jobs/`. I also wrote single-partition output and data partitioned by `department`, then confirmed the resulting S3 folders and files.

**Screenshots**

![Glue Notebook Data Loading](screenshots/Glue_Notebook_Data_Loading.png)

![Notebook Record Count](screenshots/Notebook_Record_Count.png)

![Glue Notebook Transformation](screenshots/Glue_Notebook_Transformation.png)

![Glue Notebook Parquet Export](screenshots/Glue_Notebook_Parquet_Export.png)

![Glue Notebook S3 Output](screenshots/Glue_Notebook_S3_Output.png)

### Create a Database and Query Data with Amazon Athena (25 points)

I used Amazon Athena to execute `CREATE DATABASE gdelt;`, then created external tables for the GDELT dataset: `events`, `eventcodes`, `types`, `groups`, and `countries`. Supporting lookup data was stored under `gdelt/` in S3. I verified that Athena registered the tables and successfully ran SQL analysis queries, including event frequencies by year, top event categories, and queries involving Barack Obama and Angela Merkel.

**Screenshots**

![Athena Glue Database](screenshots/Athena_Glue_Database.png)

![GDELT S3 Folders](screenshots/GDELT_S3_Folders.png)

![Athena Events Table](screenshots/Athena_Events_Table.png)

![Athena Eventcodes Table](screenshots/Athena_Eventcodes_Table.png)

![Athena Types Table](screenshots/Athena_Types_Table.png)

![Athena Groups Table](screenshots/Athena_Groups_Table.png)

![Athena Countries Table](screenshots/Athena_Countries_Table.png)

![Athena Events Per Year](screenshots/Athena_Events_Per_Year.png)

![Athena Top Event Categories](screenshots/Athena_Top_Event_Categories.png)

![Athena Obama Events](screenshots/Athena_Obama_Events.png)

![Athena Obama Merkel Events](screenshots/Athena_Obama_Merkel_Events.png)

## Results and Takeaways

The screenshots document successful Firehose delivery to S3, automated schema discovery, a completed Glue Studio transformation, notebook-based Parquet exports, and successful Athena database/table creation and SQL queries. Together, the labs demonstrate how S3 provides durable storage, Glue organizes and transforms data, and Athena supports serverless SQL analysis.

## Repository Contents

- `README.md` — assignment overview, implementation notes, and evidence.
- `screenshots/` — screenshots captured from the AWS workshop.

## Notes

- AWS resources were created in a temporary workshop account; this repository documents the work rather than providing a live deployment.
- Screenshots show the state of the workshop during execution.
- No credentials, access keys, or secrets are included.
