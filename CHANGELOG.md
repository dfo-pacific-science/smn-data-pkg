# Changelog

All notable changes to the Salmon Data Package (SDP) specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial project structure
- Schema definitions in `schemas/` directory
- Specification document (SPECIFICATION.md)
- Minimal example package

## [0.1.0] - 2025-01-XX

### Added
- Initial specification draft
- Core metadata schemas: `dataset.csv`, `tables.csv`, `column_dictionary.csv`, `codes.csv`
- Support for ontology linking via IRIs
- Support for SKOS concept schemes
- Column role classification system
- Frictionless Data Package compatibility and integration
- Comprehensive validation framework (structural, semantic, domain-specific)
- IRI field harmonization with "Recommended" category
- Clarified distinction between `entity_iri` (table-level) and `term_iri` (column-level)
