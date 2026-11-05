# Solar Data Platform

**Version:** 0.0.0  
**Status:** Under construction

## Overview

Solar Data Platform is an experimental, serverless-oriented data engineering and scientific analysis project focused on solar magnetic activity and its relationship to space-weather events.

The initial goal is to build a reusable AWS data platform, provisioned primarily with Terraform, that can collect time-series observations from public heliophysics data sources, preserve the original source data, process selected measurements with Python, and make normalized and derived datasets available for analysis.

The scientific interests guiding the project include:

- Solar photospheric magnetic fields and polarimetric observations.
- Active-region magnetic properties and their evolution over time.
- Solar flares.
- Coronal mass ejections (CMEs).
- Interplanetary magnetic fields and solar-wind measurements.
- Relationships between solar magnetic activity and subsequent solar and heliospheric events.

This repository is a starting scaffold. **The application, infrastructure, data schemas, and scientific analysis are not yet implemented.** Dataset selection, access methods, scientific assumptions, and AWS architecture will be refined as development proceeds.

## Initial data-source candidates

The following sources are candidates for investigation and integration. Availability, access methods, cadence, licensing, and dataset suitability will be verified during implementation.

| Source | Potential use |
|---|---|
| SDO/HMI SHARP | Time-series photospheric vector magnetic-field observations for solar active regions |
| Hinode/SOT Spectro-Polarimeter | Higher-detail spectropolarimetric observations, including Stokes measurements |
| NASA DONKI | Solar flare, CME, and other space-weather event metadata |
| GOES XRS | Solar X-ray flux time series and flare evolution |
| NASA OMNI | Near-Earth solar-wind and interplanetary magnetic-field measurements |
| CDAWeb HAPI | Standardized access to time-series measurements from heliophysics missions |

The initial magnetic-field work is expected to use derived products such as SDO/HMI SHARP. Direct processing of Stokes I, Q, U, and V observations may be added as a later capability.

Magnetic field lines are not generally direct time-series measurements. They may be reconstructed from photospheric magnetic-field observations using methods such as potential-field or nonlinear force-free field extrapolation. Such scientific processing is a possible later extension, not part of the initial scaffold.

## Planned architecture

The intended architecture is serverless where practical, with AWS services provisioned and configured through reusable Terraform modules.

Candidate components include:

- **Amazon S3:** Central data lake for original source files, normalized observations, and analysis-ready datasets.
- **AWS Lambda:** Lightweight API ingestion, metadata handling, and small processing tasks.
- **AWS Step Functions:** Coordination of multi-step ingestion and processing workflows.
- **Amazon EventBridge:** Scheduled ingestion and workflow triggers.
- **Amazon SQS:** Optional queues for decoupled ingestion, retries, and failure handling.
- **Amazon DynamoDB:** Operational state, ingestion checkpoints, and event metadata; not the primary scientific time-series store.
- **AWS Glue Data Catalog:** Table and schema metadata for data stored in S3.
- **Amazon Athena:** SQL queries over supported datasets in S3.
- **Amazon ECS with AWS Fargate:** Optional containerized execution for larger Python scientific-processing workloads that are not a good fit for Lambda.
- **Amazon CloudWatch:** Logs, metrics, alarms, and operational monitoring.

The exact services and deployment topology will be selected incrementally. The project should support a low-cost development configuration and avoid provisioning optional services unless they are needed.

## Data organization

The planned S3 data lake follows a layered structure:

1. **Raw (Bronze):** Original FITS files, API responses, and other source data, retained with provenance.
2. **Processed (Silver):** Validated and normalized observations, with consistent timestamps, units, coordinate conventions, and tabular formats such as Parquet where appropriate.
3. **Analytics (Gold):** Derived magnetic features, event associations, and datasets prepared for scientific analysis.

Python will be used for scientific processing and analysis. Candidate libraries include Astropy, SunPy, NumPy, SciPy, and Pandas. Athena and SQL will support querying and joining suitable tabular datasets.

## Planned scientific analysis

An initial analysis workflow may:

1. Retrieve time-series magnetic observations for a solar active region.
2. Calculate or extract magnetic-field properties at each observation time.
3. Retrieve relevant flare and CME event records.
4. Associate events with active regions when the source data supports that association.
5. Align observations and events on a common UTC time axis.
6. Compare magnetic-field evolution with flare characteristics and CME properties.
7. Extend the analysis to solar-wind and interplanetary magnetic-field measurements, accounting for propagation delays between solar eruptions and near-Earth observations.

Potential magnetic features include unsigned magnetic flux, field strength, inclination, shear, and other parameters available in or derived from vector magnetograms. Scientific associations will be treated as correlations unless supported by further physical analysis; correlation alone does not establish causation.

## Repository structure

The following is the planned initial layout. Empty directories are retained with `.gitkeep` files until implementation files are added.

```text
solar-data-platform/
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── providers.tf
│   ├── versions.tf
│   │
│   ├── modules/
│   │   ├── s3-data-lake/
│   │   ├── ingestion-lambda/
│   │   ├── step-functions/
│   │   ├── sqs-queues/
│   │   ├── dynamodb-state/
│   │   ├── glue-catalog/
│   │   ├── athena/
│   │   ├── ecs-processing/
│   │   ├── eventbridge/
│   │   └── monitoring/
│   │
│   └── environments/
│       ├── dev/
│       └── prod/
│
├── src/
│   ├── ingestion/
│   │   ├── hmi/
│   │   ├── donki/
│   │   ├── hapi/
│   │   └── goes/
│   │
│   ├── processing/
│   │   ├── magnetograms/
│   │   ├── solar_wind/
│   │   └── events/
│   │
│   └── analysis/
│       ├── magnetic_features/
│       ├── flare_correlations/
│       └── cme_correlations/
│
├── tests/
├── notebooks/
├── docs/
└── README.md
```

The Terraform root configuration files shown above are planned files; the scaffold script currently creates the directory structure only.

## Development principles

- **Infrastructure as code:** AWS resources should be defined and managed with Terraform.
- **Modularity:** Terraform modules and Python source components should have clear responsibilities.
- **Reproducibility:** Preserve source data, provenance, processing versions, and analysis parameters.
- **Idempotency:** Repeated ingestion or workflow execution should not unintentionally duplicate data or corrupt state.
- **Data quality:** Validate schemas, timestamps, units, missing values, and expected data ranges.
- **Observability:** Record workflow status, failures, processing duration, and relevant operational metrics.
- **Cost awareness:** Prefer managed and serverless services, use on-demand processing where practical, and make optional components configurable.
- **Scientific care:** Preserve coordinate systems, units, uncertainties, cadence, and source metadata. Document assumptions and distinguish measured quantities from derived quantities.

## Initial implementation milestones

1. **Repository and Terraform foundation:** Establish Terraform versions, provider configuration, environment conventions, and an initial S3 data lake.
2. **First ingestion source:** Implement a small Python ingestion workflow for a straightforward API source, such as DONKI.
3. **Time-series ingestion:** Add a HAPI-based solar-wind data source and define common timestamp and measurement conventions.
4. **Magnetic observations:** Investigate and implement retrieval of SDO/HMI SHARP observations, including the appropriate archive access workflow.
5. **Scientific processing:** Extract and normalize selected magnetic-field measurements and write analysis-ready Parquet datasets.
6. **Query and correlation:** Register datasets in Glue, query them with Athena, and create initial flare-correlation analyses.
7. **Reliability and expansion:** Add ingestion checkpoints, retries, monitoring, additional sources, and optional containerized processing.
8. **Advanced solar analysis:** Investigate coronal magnetic-field extrapolation, field-line tracing, and CME/solar-wind associations.

## Current status

This project is in the **initial planning and scaffolding stage**.

- [x] Initial repository structure defined.
- [ ] Terraform foundation implemented.
- [ ] AWS development environment provisioned.
- [ ] First data-source adapter implemented.
- [ ] Raw and processed data conventions established.
- [ ] Scientific analysis implemented.

No AWS resources are provisioned by this repository scaffold. Review costs, IAM permissions, data-source terms, and Terraform state configuration before deploying infrastructure.
