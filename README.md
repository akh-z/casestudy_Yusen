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

## How to run the notebook
* Create a new Catalog in Databricks (e.g., yusen_catalog).
* Create a new volume under the created catalog via "Create Volume".
* Upload the carrier and warehouse files from the given_files folder into the new volume.
* Import the p_shipmentTraking.ipynb into the Workspace.
* Update the SCHEMA to [Catalog Name].default (e.g., yusen_catalog.default).
* Update the BASE path to the location of the uploaded files (e.g., Volumes/yusen_catalog/default/datasource).
* Run all with Serverless compute attached.

## Architecture
* This pipeline strictly follows a medallion style architecture.
* It includes mandatory data quality gates to prevent corrupt data from moving forward, processing in the following stages: 
  `Bronze -> [GATE] -> Silver -> [GATE] -> Gold -> [GATE]`
