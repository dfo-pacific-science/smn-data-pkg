# Salmon Data Package Specification

**Version**: sdp-0.1.0  
**Author**: Brett Johnson, Data Stewardship Unit (DFO Pacific Region Science Branch)

## Overview

The **Salmon Data Package (SDP)** is a lightweight, frictionless-style specification for exchanging salmon datasets between scientists, assessment biologists, and data stewards.

It is designed to:

- Be **simple to adopt** with Excel and CSV files.
- Be **tool-friendly** for R/Python packages and custom GPT assistants.
- Be **ontology-aware**, linking columns and codes to the **DFO Salmon Ontology** (a formal specification of concepts, their properties, and relationships) and related vocabularies.
- Be compatible with Frictionless Data Package (a specification for describing data packages with metadata) and Darwin Core (a biodiversity data standard) without forcing a single rigid schema.

Each SDP instance is a small directory of CSV (Comma-Separated Values - a simple text file format where each line represents a row and columns are separated by commas) data files plus a set of metadata CSVs that describe the dataset, tables, columns, and controlled codes.

## Design Goals

- **Interoperable but flexible**: support multiple schemas (FSAR, SPSR-like, project-specific) while providing shared semantics via IRIs.
- **Frictionless-compatible**: align conceptually with frictionless/datapackage ideas so existing tooling can be reused or extended.
- **Ontology-linked**: allow columns and codes to reference DFO Salmon Ontology and SKOS (Simple Knowledge Organization System - a W3C standard for representing controlled vocabularies) vocab IRIs.
- **Incremental adoption**: existing tables can be wrapped in a package with minimal changes (no need to refactor everything at once).
- **Machine- and human-readable**: scientists can work in spreadsheets; machines can consume the same metadata.

## Package Layout

A Salmon Data Package consists of:

- One **root directory** (e.g., `SDP_myproject/`) containing:
  - `dataset.csv` – dataset-level metadata (required).
  - `tables.csv` – one row per logical table (required).
  - `column_dictionary.csv` – one row per column in each table (required).
  - `codes.csv` – required: controlled value lists (predefined lists of valid values for a field) and SKOS links for categorical columns.
  - `datapackage.json` – optional: Frictionless Data Package descriptor (see [Frictionless Integration](#frictionless-data-package-compatibility)).
- One or more **data files** (typically CSV) referenced from `tables.csv`, usually in a `data/` subdirectory.

Minimal example layout:

- `dataset.csv`
- `tables.csv`
- `column_dictionary.csv`
- `codes.csv`
- `data/your_table_1.csv`
- `data/your_table_2.csv`
- etc.

**Note**: A valid SDP package requires all four CSV metadata files (`dataset.csv`, `tables.csv`, `column_dictionary.csv`, `codes.csv`). The `datapackage.json` file is optional and can be generated automatically by tools like the `metasalmon` R package to enable Frictionless Data Package compatibility.

The SDP spec does **not** prescribe a single canonical salmon schema. Instead, it standardizes the **metadata** about whatever schema a project uses and ties it to ontology and vocabularies.

## Identifiers and Conventions

### Identifier Format Rules

All identifiers (`dataset_id`, `table_id`, `column_name`) must follow these rules:

- **Allowed characters**: Alphanumeric (a-z, A-Z, 0-9), underscore (`_`), and hyphen (`-`)
- **Case sensitivity**: Identifiers are case-sensitive. `CU_ID` and `cu_id` are different identifiers.
- **Uniqueness**:
  - `dataset_id`: Must be unique within your organization or project scope
  - `table_id`: Must be unique within a dataset (scoped by `dataset_id`)
  - `column_name`: Must be unique within a table (scoped by `table_id`)
- **Length**: Recommended maximum of 64 characters for readability, though no hard limit is enforced
- **Starting character**: Should start with a letter or underscore (not a number or hyphen)

### Identifier Descriptions

- `dataset_id`
  - Short string identifier unique within your organization or project (e.g., `fsar_spsr_chinook_2025`).
  - Used to link rows across `dataset.csv`, `tables.csv`, `column_dictionary.csv`, and `codes.csv`.
  - Examples: `nuseds_fraser_coho_sample`, `spsr_chinook_2024`, `cu_index_benchmark_2023`

- `table_id`
  - Short ID unique within a dataset (e.g., `cu_year_index`, `survey_events`).
  - Examples: `cu_year_index`, `survey_events`, `population_status`, `escapement_estimates`

- `column_name`
  - Exact column name as it appears in the data file (e.g., `TOTAL_SPAWNERS`, `run_size_total`).
  - Must match exactly including case, whitespace, and special characters as they appear in the actual data file.
  - Examples: `POP_ID`, `ANALYSIS_YR`, `NATURAL_SPAWNERS_TOTAL`, `cu_id`, `run_year`

- **IRIs (Internationalized Resource Identifiers)**
  - An IRI is a web address (URL) that uniquely identifies a resource on the internet. IRIs enable machine-readable linking: software can look up the IRI to retrieve the definition, relationships, and metadata about that resource. For example, `https://www.ncbi.nlm.nih.gov/taxonomy/9606` points to the specific definition of *Homo sapiens* in NCBI's taxonomy database. IRIs are similar to URLs but support international characters.
  - All IRI fields use persistent HTTP IRIs where available (e.g., w3id for DFO Salmon Ontology terms, vocabulary concepts, and units).
  - IRI fields are **recommended** (not required) to enable semantic linking and interoperability.
  - IRI fields include: `entity_iri` (table-level), `term_iri` (column-level), `unit_iri` (measurement units), `vocabulary_iri` (controlled vocabularies).
  - **For machine-readable integration**: `term_iri` is critical - it points to specific term definitions that enable automated validation and linking.
  - **IRI Format Examples**:
    - DFO Salmon Ontology class: `https://w3id.org/gcdfo/salmon#EscapementMeasurement`
    - DFO Salmon Ontology property: `https://w3id.org/gcdfo/salmon#hasEscapement`
    - UCUM (Unified Code for Units of Measure - a standard system for representing units of measurement) unit: `http://unitsofmeasure.org/ucum#v1.0` (for specific units, append unit code)
    - Controlled vocabulary (SKOS concept scheme): `https://w3id.org/gcdfo/salmon#BiologicalBenchmarkAndIndicatorScheme`
    - Controlled vocabulary (NCBI (National Center for Biotechnology Information) taxonomy - general): `https://www.ncbi.nlm.nih.gov/taxonomy`
    - Specific NCBI taxon (for `term_iri`): `https://www.ncbi.nlm.nih.gov/taxonomy/9606` (points to *Homo sapiens*)
    - SKOS concept: `https://w3id.org/gcdfo/salmon#TargetOrLimitRateOrAbundance`

## `dataset.csv` Schema

One row per logical dataset (often one per SDP directory). Can describe multiple related tables.

### Required columns:

| Column          | Type    | Required | Description                                                                 |
|-----------------|---------|----------|-----------------------------------------------------------------------------|
| dataset_id      | string  | yes      | Stable identifier used to join to other metadata files.                     |
| title           | string  | yes      | Human-readable dataset title.                                               |
| description     | string  | yes      | Short description of the dataset contents and purpose.                      |
| creator         | string  | yes      | Name(s) of primary creator(s) or project.                                   |
| contact_name    | string  | yes      | Primary contact person.                                                     |
| contact_email   | string  | yes      | Contact email address.                                                      |
| license         | string  | yes      | License name or URL (e.g., `CC-BY-4.0`).                                    |

### Optional columns:

| Column          | Type    | Description                                                                 |
|-----------------|---------|-----------------------------------------------------------------------------|
| temporal_start  | date    | Start date or year covered by the dataset. Format: ISO 8601 (an international standard for representing dates and times) date (`YYYY-MM-DD`) or year as integer (`YYYY`). Examples: `2024-01-15`, `1996`. |
| temporal_end    | date    | End date or year covered by the dataset. Format: ISO 8601 date (`YYYY-MM-DD`) or year as integer (`YYYY`). Examples: `2024-12-31`, `2024`. |
| spatial_extent  | string  | Textual description of spatial coverage (e.g., CUs, regions, coordinates). |
| dataset_type    | string  | High-level type (e.g., `cu_year_index`, `survey_timeseries`, `benchmark`). |
| source_citation | string  | Free-text citation for publications, reports, or internal docs.            |
| provenance_note | string  | Narrative about data lineage.                                               |
| created         | datetime | Timestamp when dataset was created. Format: ISO 8601 datetime (`YYYY-MM-DDTHH:MM:SSZ` or `YYYY-MM-DDTHH:MM:SS+00:00`). Examples: `2024-01-15T10:30:00Z`, `2024-01-15T10:30:00-08:00`. |
| modified        | datetime | Timestamp when dataset was last modified. Format: ISO 8601 datetime (`YYYY-MM-DDTHH:MM:SSZ` or `YYYY-MM-DDTHH:MM:SS+00:00`). Examples: `2024-01-20T14:45:00Z`, `2024-01-20T14:45:00-08:00`. |
| spec_version    | string  | SDP specification version (e.g., `sdp-1.0.0`).                           |

## `tables.csv` Schema

One row per table in the package.

### Required columns:

| Column       | Type   | Required | Description                                                                 |
|--------------|--------|----------|-----------------------------------------------------------------------------|
| dataset_id   | string | yes      | References `dataset_id` in `dataset.csv`.                                  |
| table_id     | string | yes      | Short ID for the table (e.g., `cu_year_index`).                             |
| file_name    | string | yes      | Relative path to the data file, relative to the SDP package root directory (e.g., `data/cu_year_index.csv`). Must not use `../` to reference files outside the package directory. |
| table_label  | string | yes      | Human-readable label for the table.                                         |
| description  | string | yes      | Description of what each row represents and how the table is used.          |

### Optional columns:

| Column       | Type   | Category | Description                                                                 |
|--------------|--------|----------|-----------------------------------------------------------------------------|
| entity_type  | string | optional | Human-readable entity type label (e.g., `CU-year index`, `survey event`). Use this if you don't know the `entity_iri`. |
| entity_iri   | string | recommended | IRI of the ontology class that represents what each row in the table is (the row-level entity type). For example, if each row represents a CU-year combination, this would be the IRI for a CU-year index class. |
| primary_key  | string | optional | Comma-separated list of column names forming a primary key (a column or set of columns that uniquely identifies each row in a table, e.g., `cu_id,year`). No spaces around commas. Column names must exist in the data file and match exactly (case-sensitive) with `column_name` entries in `column_dictionary.csv`. |

### Notes:

- **`entity_iri` vs `term_iri`**: 
  - `entity_iri` (in `tables.csv`) identifies the **type of thing** each row represents (e.g., a CU-year index, a survey event, a population).
  - `term_iri` (in `column_dictionary.csv`) identifies the **concept** that a column measures or represents (e.g., escapement measurement, species, run type).
  - Think of it this way: `entity_iri` = "what is each row?", `term_iri` = "what does this column mean?"
- `entity_type` is a human-readable label that can be used instead of or alongside `entity_iri`. If you provide `entity_iri`, `entity_type` is optional.
- `primary_key` is advisory; it guides downstream validation and graph loading.

## `column_dictionary.csv` Schema

One row per column in each table. This is the core of how SDP links data columns to ontology and vocabularies.

### Required columns:

| Column              | Type    | Required | Description                                                                                   |
|---------------------|---------|----------|-----------------------------------------------------------------------------------------------|
| dataset_id          | string  | yes      | References `dataset_id` in `dataset.csv`.                                                     |
| table_id            | string  | yes      | References `table_id` in `tables.csv`.                                                        |
| column_name         | string  | yes      | Exact column name in the data file. Must match exactly including case, whitespace, and special characters as they appear in the actual data file header. Case-sensitive matching is required.                                                           |
| column_label        | string  | yes      | Short human-readable label.                                                                   |
| column_description  | string  | yes      | Clear definition of the column's meaning.                                                     |
| column_role         | string  | yes      | One of: `identifier`, `attribute`, `temporal`, `categorical`, `measurement`.                  |
| value_type          | string  | yes      | Basic type: `integer`, `double`, `string`, `boolean`, `date`, `datetime`.                     |

### Optional columns:

| Column              | Type    | Category | Description                                                                                   |
|---------------------|---------|----------|-----------------------------------------------------------------------------------------------|
| required            | boolean | optional | `TRUE` (uppercase) if the column is required for each row, otherwise `FALSE` (uppercase) or leave blank. Case-sensitive: must be exactly `TRUE` or `FALSE` if provided.                    |
| unit_label          | string  | optional | Human-readable unit label (e.g., `number of fish`, `proportion`).                             |
| unit_iri            | string  | recommended | IRI for the unit (e.g., UCUM or other unit ontology). Use when the column represents a measurement with units. |
| term_iri            | string  | recommended | IRI for the ontology term that represents what this column measures or represents. This links the column to concepts in the DFO Salmon Ontology (e.g., measurement types, dimensions, concepts). |
| term_type           | string  | optional | Type of ontology term (e.g., `owl_class` (OWL - Web Ontology Language, a W3C standard for defining ontologies), `owl_object_property`, `skos_concept`). Helps validators understand how to interpret the `term_iri`. |

### Guidance:

- Use `column_role = "identifier"` for identifiers such as `cu_id`, `survey_event_id`, `POP_ID`.
- Use `column_role = "measurement"` for numeric quantities (e.g., escapement, run size, exploitation rate).
- Use `column_role = "temporal"` for time-related columns (e.g., year, date, datetime).
- Use `column_role = "categorical"` for columns with controlled vocabularies or enumerated values (e.g., species, run type, status codes).
- Use `column_role = "attribute"` for descriptive attributes that are not identifiers, measurements, temporal, or categorical (e.g., population name, waterbody name).
- `term_iri` should reference a term in the DFO Salmon Ontology (e.g., measurement types, dimensions, concepts) when available.
- `term_type` indicates the type of ontology term (e.g., `owl_class` for classes, `owl_object_property` for properties, `skos_concept` for SKOS concepts).

## `codes.csv` Schema

Required file used for columns that have controlled enumerated values (status codes, methods, gear, species, etc.). This is where SKOS vocabularies plug in.

Each row corresponds to one code value for one column, or references a controlled vocabulary via `vocabulary_iri`.

**Relationship to Data Files**: 
- `codes.csv` defines the valid values for categorical columns (columns with `column_role = "categorical"`).
- All values present in categorical columns of data files should have corresponding entries in `codes.csv` for documentation and validation purposes.
- If a categorical column uses a large external controlled vocabulary (e.g., NCBI taxonomy for species), you may list the `vocabulary_iri` instead of enumerating all possible codes. In this case, include at least one row with the `vocabulary_iri` specified, and optionally list the codes actually used in your data.
- `codes.csv` may include codes that are not yet used in data files (e.g., for documenting all possible values in a controlled vocabulary).

**Large Controlled Vocabularies**:
When a categorical column references a large external controlled vocabulary (e.g., NCBI taxonomy, GBIF (Global Biodiversity Information Facility - an international network providing open access to biodiversity data) species lists, SKOS concept schemes), it is acceptable to:
1. Include a row with `vocabulary_iri` pointing to the vocabulary system (e.g., `https://www.ncbi.nlm.nih.gov/taxonomy` for NCBI, or a SKOS concept scheme IRI) - this provides context about which vocabulary is used
2. List the specific `code_value` entries that appear in your data
3. **Provide `term_iri` for each `code_value`** - this is what enables machine-readable integration. Each `term_iri` should point to the specific term definition (e.g., `https://www.ncbi.nlm.nih.gov/taxonomy/9606` for a specific NCBI taxon)

**Important for Machine Integration**: While `vocabulary_iri` provides useful context, `term_iri` is what enables automated validation, linking, and integration. For machine-readable data integration, provide `term_iri` for each code value that appears in your data.

Example: For a species column using NCBI taxonomy, you might have:
- One row with `vocabulary_iri = "https://www.ncbi.nlm.nih.gov/taxonomy"` and `code_value` left blank (provides context)
- Additional rows for each species code actually used in your data:
  - `code_value = "Oncorhynchus kisutch"` (Coho salmon)
  - `term_iri = "https://www.ncbi.nlm.nih.gov/taxonomy/8019"` (points to the specific NCBI taxon ID for Coho salmon)
  - `code_label = "Coho salmon"`

### Required columns:

| Column             | Type   | Required | Description                                                                  |
|--------------------|--------|----------|------------------------------------------------------------------------------|
| dataset_id         | string | yes      | References `dataset_id` in `dataset.csv`.                                   |
| table_id           | string | yes      | References `table_id` in `tables.csv`.                                      |
| column_name        | string | yes      | Name of the column in the data file that uses this code.                    |
| code_value         | string | no*      | Stored value in the data (e.g., `R`, `Y`, `G`, `critical`, `adipose_intact`). *Required if `vocabulary_iri` is not provided. May be omitted if `vocabulary_iri` is specified for large controlled vocabularies (see guidance below). |

### Optional columns:

| Column             | Type   | Category | Description                                                                  |
|--------------------|--------|----------|------------------------------------------------------------------------------|
| code_label         | string | optional | Human-readable label corresponding to the code.                             |
| code_description   | string | optional | Longer description of what the code means.                                  |
| vocabulary_iri     | string | recommended | IRI of the controlled vocabulary system that defines valid values for this column. Provides context about which vocabulary is used (e.g., `https://www.ncbi.nlm.nih.gov/taxonomy` for NCBI taxonomy, or a SKOS concept scheme IRI). **Note**: This points to the vocabulary system, not individual terms. For machine-readable integration, use `term_iri` to point to specific terms. **Required** when referencing large external vocabularies where enumerating all codes is impractical. |
| term_iri           | string | recommended | **Critical for machine-readable integration**: IRI of the specific term that `code_value` represents. This should point to the specific term definition (e.g., `https://www.ncbi.nlm.nih.gov/taxonomy/9606` for a specific NCBI taxon, `https://w3id.org/gcdfo/salmon#TargetOrLimitRateOrAbundance` for a SKOS concept). This enables automated validation, linking, and integration with other datasets. For each `code_value` in your data, provide its corresponding `term_iri` to enable machine-assisted data integration. |
| term_type          | string | optional | Type of ontology term (e.g., `skos_concept`, `owl_class`).                 |

### Guidance:

- **For machine-readable integration**: Provide `term_iri` for each `code_value` that appears in your data. This enables automated validation, linking, and integration with other datasets. `term_iri` should point to the specific term definition (e.g., `https://www.ncbi.nlm.nih.gov/taxonomy/9606` for a specific species, or `https://w3id.org/gcdfo/salmon#TargetOrLimitRateOrAbundance` for a SKOS concept).

- **For small controlled vocabularies**: List each `code_value` that appears in your data with its corresponding `term_iri`. `code_value` must match exactly (case-sensitive) the values present in the data table.

- **For large external vocabularies** (e.g., NCBI taxonomy, GBIF species, SKOS concept schemes): 
  - Provide `vocabulary_iri` pointing to the vocabulary system (for context/documentation)
  - **Provide `term_iri` for each `code_value`** that appears in your data (required for machine integration)
  - Each `term_iri` should point to the specific term definition (e.g., `https://www.ncbi.nlm.nih.gov/taxonomy/9606` for a specific NCBI taxon)

- `code_label` and `code_description` can be used to auto-generate documentation.

- **Relationship between `vocabulary_iri` and `term_iri`**: 
  - `vocabulary_iri` provides context about which vocabulary system is used (e.g., "this column uses NCBI taxonomy")
  - `term_iri` is what enables actual machine-readable integration - it points to the specific term definition for each code value
  - For automated data integration, `term_iri` is essential; `vocabulary_iri` alone is not sufficient

- `term_type` indicates the type of ontology term (e.g., `owl_class` for classes, `owl_object_property` for properties, `skos_concept` for SKOS concepts) and helps validators understand how to interpret the `term_iri`.

## Relationship to Ontology, R Package, and GPT

- The **ontology** defines the formal semantics for:
  - measurement types, entities, dimensions, and concepts (via `term_iri` and `term_type`),
  - entities (`entity_iri` in `tables.csv`),
  - controlled vocabularies (`vocabulary_iri` in `codes.csv`).
- The **R package** (`metasalmon`) will:
  - read SDP metadata and data files,
  - validate packages against this specification,
  - join in ontology/vocabulary information,
  - help reshape data into analysis-friendly tidy formats,
  - automatically generate `datapackage.json` from CSV metadata for Frictionless compatibility.
- The **custom GPT** will:
  - help users draft `column_dictionary.csv` and `codes.csv`,
  - suggest ontology-aligned IRIs and term types,
  - assist in decomposing compound terms into multiple columns with appropriate roles and IRIs,
  - propose SDP-compliant package structures for new projects.

## Versioning and Backwards Compatibility

- Include a `spec_version` field in `dataset.csv` in future iterations if needed (e.g., `sdp-0.1.0`).
- New columns should be added in a backwards-compatible way:
  - existing tooling must continue to work when extra columns are present.
- Breaking changes to required columns or semantics should bump the major version.

## Implementation Notes

### File Format

- **Encoding**: CSV files must use UTF-8 (a character encoding standard that can represent any character in the Unicode standard) encoding without BOM (Byte Order Mark). Files with BOM should be converted before use.
- **Delimiter**: Comma (`,`) is the standard delimiter.
- **Line endings**: Both LF (`\n`) and CRLF (`\r\n`) are accepted, though LF is recommended for consistency.
- **Quoting**: Fields containing commas, quotes, or newlines must be quoted with double quotes (`"`). Embedded quotes are escaped by doubling them (`""`).
- **Header row**: First row of each CSV file contains column names.

### Date and Time Formats

- **Dates**: Use ISO 8601 format `YYYY-MM-DD` (e.g., `2024-01-15`). Years may be encoded as integers (e.g., `2024`) for temporal_start/temporal_end fields.
- **Datetime**: Use ISO 8601 format `YYYY-MM-DDTHH:MM:SSZ` or `YYYY-MM-DDTHH:MM:SS+00:00` (e.g., `2024-01-15T10:30:00Z`, `2024-01-15T10:30:00-08:00`).
- **Time zones**: UTC (Z) is recommended. If timezone offset is used, include full offset (e.g., `-08:00`, not `-8`).

### Missing Values

- **Optional fields**: Missing optional fields may be represented as:
  - Empty string (blank cell in CSV)
  - Omitted column (if entire column is optional and unused)
- **Required fields**: Must always have a value (non-empty string).
- **Boolean fields**: Use `TRUE` or `FALSE` (uppercase, case-sensitive). Leave blank if not applicable.
- **Tooling behavior**: R and Python tooling should treat missing optional columns as `NA`/`null` without failing validation.

### Value Types

- **integer**: Whole numbers only (e.g., `123`, `-45`, `0`). No decimal points or scientific notation.
- **double**: Floating-point numbers (e.g., `123.45`, `-0.001`, `1.23e-4`).
- **string**: Any text. May contain Unicode characters (UTF-8 encoded).
- **boolean**: In data files, use `TRUE`/`FALSE` or `1`/`0` or `yes`/`no` as appropriate. In metadata (e.g., `required` field), use `TRUE`/`FALSE` only.
- **date**: ISO 8601 date format `YYYY-MM-DD` or year as integer `YYYY`.
- **datetime**: ISO 8601 datetime format `YYYY-MM-DDTHH:MM:SSZ` or with timezone offset.

### Extensibility

- The spec is intentionally minimal; domain-specific extensions (e.g., FSAR/SPSR templates) can be layered on top as recommended profiles.
- Additional columns may be added to any metadata file; tooling should ignore unknown columns gracefully.

## Frictionless Data Package Compatibility

SDP is designed to be fully compatible with the [Frictionless Data Package](https://specs.frictionlessdata.io/data-package/) specification. SDP extends Frictionless with salmon-specific metadata while maintaining compatibility with Frictionless tooling.

### Core Frictionless Alignment

SDP aligns with Frictionless Data Package in the following ways:

1. **Package Structure**: An SDP package can be wrapped in a `datapackage.json` descriptor to become a fully valid Frictionless Data Package.

2. **Resource Model**: Each SDP metadata file (`dataset.csv`, `tables.csv`, `column_dictionary.csv`, `codes.csv`) can be represented as Frictionless resources.

3. **Table Schema**: SDP column definitions can be expressed using Frictionless Table Schema format for validation.

4. **CSV Dialect**: SDP uses the standard Frictionless CSV dialect (comma-delimited, UTF-8, RFC 4180-compliant).

5. **Validation**: SDP packages can be validated using standard Frictionless validation tools, with additional salmon-specific checks layered on top.

### Optional datapackage.json Integration

SDP packages may optionally include a `datapackage.json` file that wraps the SDP metadata for full Frictionless compatibility. This enables:

- Validation using Frictionless tooling (e.g., `frictionless validate`)
- Integration with Frictionless-compatible platforms (e.g., DataHub, CKAN, Open Government Portal)
- Dual compatibility (SDP-native and Frictionless)

**When to include `datapackage.json`:**

- **Not required**: A valid SDP package consists of the CSV metadata files only. You can create and use SDP packages without `datapackage.json`.
- **Recommended for**: Publishing to Frictionless-compatible platforms, using Frictionless validation tools, or when you need programmatic access via Frictionless libraries.
- **Auto-generation**: Tools like the `metasalmon` R package can automatically generate `datapackage.json` from your CSV metadata files, so you don't need to create it manually.

Example `datapackage.json` structure:

```json
{
  "profile": "data-package",
  "name": "sdp-package",
  "title": "Salmon Data Package",
  "description": "A salmon-specific extension of Frictionless Data Package",
  "version": "1.0.0",
  "license": "CC-BY-4.0",
  "resources": [
    {
      "name": "dataset",
      "path": "dataset.csv",
      "profile": "tabular-data-resource",
      "schema": {
        "fields": [
          {"name": "dataset_id", "type": "string"},
          {"name": "title", "type": "string"},
          {"name": "description", "type": "string"}
        ]
      }
    },
    {
      "name": "tables",
      "path": "tables.csv",
      "profile": "tabular-data-resource"
    },
    {
      "name": "column_dictionary",
      "path": "column_dictionary.csv",
      "profile": "tabular-data-resource"
    },
    {
      "name": "codes",
      "path": "codes.csv",
      "profile": "tabular-data-resource"
    },
    {
      "name": "cu_year_index",
      "path": "data/cu_year_index.csv",
      "profile": "tabular-data-resource"
    }
  ],
  "custom": {
    "sdp-version": "0.1.0",
    "ontology-base": "https://w3id.org/gcdfo/salmon"
  }
}
```

### CSV Dialect Specification

SDP packages use the following CSV dialect (aligned with Frictionless defaults and RFC 4180):

- **Delimiter**: comma (`,`)
- **Encoding**: UTF-8
- **Line terminator**: LF or CRLF (both accepted)
- **Quote character**: double quote (`"`)
- **Escape character**: double quote (for embedded quotes; escaped as `""`)
- **Header row**: First row contains column names
- **Empty lines**: Ignored

### Frictionless Table Schema Integration

SDP column definitions can be expressed as Frictionless Table Schema JSON. For example, a column definition in `column_dictionary.csv`:

```csv
dataset_id,table_id,column_name,column_label,column_description,column_role,value_type,term_iri
my_dataset,cu_index,escapement,Escapement,Total spawner count,measurement,integer,https://w3id.org/gcdfo/salmon#EscapementMeasurement
```

Can be represented in Frictionless Table Schema as:

```json
{
  "fields": [
    {
      "name": "escapement",
      "type": "integer",
      "title": "Escapement",
      "description": "Total spawner count",
      "custom": {
        "sdp:role": "measurement",
        "sdp:term_iri": "https://w3id.org/gcdfo/salmon#EscapementMeasurement"
      }
    }
  ]
}
```

This allows SDP packages to be validated using standard Frictionless tools while preserving salmon-specific metadata in the `custom` namespace.

## Validation

SDP packages should be validated at multiple levels: structural validation (Frictionless-compatible), semantic validation (IRI and ontology checks), and domain-specific validation (salmon data rules).

### Structural Validation (Frictionless-Compatible)

These validations align with standard Frictionless Data Package validation:

1. **File Structure**
   - Required files exist: `dataset.csv`, `tables.csv`, `column_dictionary.csv`, `codes.csv`
   - Data files referenced in `tables.csv` exist and are readable

2. **CSV Format**
   - Files are valid CSV (comma-delimited, UTF-8 encoded)
   - Headers match expected columns
   - No empty required columns

3. **Foreign Key Constraints**
   - `dataset_id` in `tables.csv`, `column_dictionary.csv`, and `codes.csv` must exist in `dataset.csv` (case-sensitive match)
   - `table_id` in `column_dictionary.csv` and `codes.csv` must exist in `tables.csv` for the same `dataset_id` (case-sensitive match)
   - `column_name` in `codes.csv` must exist in `column_dictionary.csv` for the same `table_id` and `dataset_id` (case-sensitive match, must match exactly including case)

4. **Required Fields**
   - All required columns have non-empty values
   - No missing required metadata

5. **Data Type Validation**
   - Values match declared `value_type` in `column_dictionary.csv`
   - Dates follow ISO 8601 format where applicable
   - Numeric types are valid numbers

6. **Primary Key Constraints** (if specified)
   - Primary keys declared in `tables.csv` are unique within data files
   - No duplicate rows based on primary key

### Semantic Validation (IRI and Ontology)

These validations check semantic alignment with the DFO Salmon Ontology:

1. **IRI Format Validation**
   - All IRI fields contain valid HTTP/HTTPS URIs
   - IRIs are well-formed (proper encoding, no spaces)

2. **IRI Resolution** (optional, can be disabled for offline validation)
   - `entity_iri` values resolve to valid ontology classes
   - `term_iri` values resolve to valid ontology terms
   - `unit_iri` values resolve to valid unit definitions
   - `vocabulary_iri` values resolve to valid vocabulary endpoints (SKOS concept schemes, REST APIs, etc.)

3. **Ontology Term Type Validation**
   - `term_type` values are valid (e.g., `owl_class`, `owl_object_property`, `skos_concept`)
   - `term_type` matches the actual type of the ontology term at the IRI (if resolution is enabled)

4. **Vocabulary Alignment**
   - `code_value` entries in `codes.csv` align with the declared `vocabulary_iri`
   - Code values match expected values in the controlled vocabulary (if resolution is enabled)
   - `vocabulary_iri` values resolve to valid vocabulary endpoints (SKOS concept schemes, REST APIs, etc.)

### Domain-Specific Validation (Salmon Data Rules)

These validations enforce salmon data domain rules:

1. **Code Value Consistency**
   - For categorical columns, `codes.csv` must contain at least one row with either:
     - A `code_value` that appears in the data, OR
     - A `vocabulary_iri` pointing to the controlled vocabulary
   - Values in `codes.csv` should match actual values present in data files (case-sensitive matching)
   - Undefined codes (codes used in data but not defined in `codes.csv`) generate warnings. If a `vocabulary_iri` is provided for the column, undefined codes are acceptable as they may be valid codes in the external vocabulary.
   - **For machine-readable integration**: `term_iri` should be provided for each `code_value` to enable automated validation and linking. Packages without `term_iri` values are valid but have limited machine integration capabilities.
   - Orphaned codes (codes defined in `codes.csv` but not used in data) are acceptable and may represent valid but unused values in a controlled vocabulary

2. **Column Role Appropriateness**
   - Identifier columns have appropriate value types (typically string or integer)
   - Measurement columns have numeric value types
   - Temporal columns have date/datetime/integer (year) value types
   - Categorical columns have corresponding entries in `codes.csv` when appropriate

3. **Unit Consistency**
   - Measurement columns with `unit_iri` have corresponding `unit_label` for human readability
   - Units are appropriate for the measurement type (e.g., fish counts use count units, not weight units)

4. **Entity Type Consistency**
   - `entity_iri` in `tables.csv` aligns with the types of entities described in the table's description
   - Primary keys are appropriate for the entity type (e.g., CU-year tables should have CU and year in primary key)

### Validation Tools and Implementation

Validation can be implemented using:

1. **Frictionless Tools**: Use `frictionless validate` for structural validation
2. **Custom Validators**: Implement SDP-specific validators in R (metasalmon package) or Python
3. **Ontology Validators**: Use SPARQL queries or ontology reasoners to validate IRI alignment
4. **Integrated Validation**: Combine all three approaches for comprehensive validation

Example validation workflow:

```bash
# Structural validation (Frictionless)
frictionless validate datapackage.json

# SDP-specific validation (custom tool)
sdp-validate --package ./my_sdp_package --check-ontology

# Combined validation
sdp-validate --package ./my_sdp_package --frictionless --ontology --domain-rules
```

### Validation Levels

Validation can be performed at different levels of strictness:

- **Minimal**: Structural validation only (required for valid SDP package)
- **Standard**: Structural + semantic validation (recommended for production use)
- **Strict**: All validations including ontology resolution and domain rules (for quality assurance)

Packages that pass minimal validation are valid SDP packages. Standard and strict validation help ensure high-quality, interoperable packages.
