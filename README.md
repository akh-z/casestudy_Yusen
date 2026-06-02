# Shipment Tracking Pipeline

## Objective
Design and implement a pipeline that ingests both sources, standardizes them, and produces a single `shipment_status` table that can be used by operations and analytics teams.

This document provides the setup and execution requirements for the Shipment Tracking notebook.

## Environment & Execution
* **Environment:** Free Databricks Version.
* **Execution:** Run via a notebook attached to serverless compute.

## File Storage & Configuration
* **Files:** These should be stored in the default volume and the path should be configured.
* Within the notebook, the schema defaults to `yusen_catalog.default` where tables are written.
* The base file path is set to `/Volumes/yusen_catalog/default/datasource`.
* *Note:* Because this dictates where the files are written in Databricks, please ensure you change this path accordingly to match your specific volume configuration.

## Architecture
* This pipeline strictly follows a medallion style architecture.
* It includes mandatory data quality gates to prevent corrupt data from moving forward, processing in the following stages: 
  `Bronze -> [GATE] -> Silver -> [GATE] -> Gold -> [GATE]`
