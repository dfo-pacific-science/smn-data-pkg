# Changelog

All notable changes to the Salmon Data Package (SDP) specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.1] - 2026-2026-01-14

### Added

- SDP I-ADOPT component columns (`property_iri`, `entity_iri`, `constraint_iri`, `method_iri`) in `column_dictionary.csv` (the compound variable remains in `term_iri`, and units stay in `unit_iri`).
- Documentation for measurement-required I-ADOPT components in `SPECIFICATION.md` and updated minimal example showing required measurement fields.
- ExecPlan for I-ADOPT adoption across SDP, ontology docs, and metasalmon tooling.
- Initial project structure
- Schema definitions in `schemas/` directory
- Specification document (SPECIFICATION.md)
- Minimal example package

### Changed

- Condensed `SPECIFICATION.md` to normative rules and added `docs/quickstart.md` and `docs/implementation-guide.md` for guidance.

## [0.1.0] - 2025-12-21

### Added

- Initial specification draft
- Core metadata schemas: `dataset.csv`, `tables.csv`, `column_dictionary.csv`, `codes.csv`
- Support for ontology linking via IRIs
- Support for SKOS concept schemes
- Column role classification system
- Frictionless Data Package compatibility and integration
- Comprehensive validation framework (structural, semantic, domain-specific)
- IRI field harmonization with "Recommended" category
- Clarified distinction between `observation_unit_iri` (table-level unit-of-observation) and `term_iri` (column-level variable), and aligned naming with SSN/OMS/OBOE mapping
