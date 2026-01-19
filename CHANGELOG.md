# Changelog

<!-- file: CHANGELOG.md -->
<!-- version: 1.0.0 -->
<!-- guid: 9a0b1c2d-3e4f-5a6b-7c8d-9e0f1a2b3c4d -->
<!-- last-edited: 2026-01-19 -->

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-12-29

### Added

- Initial release as composite GitHub Action
- API documentation generation from Python docstrings
  - Extracts function signatures and docstrings
  - Generates class documentation with methods
  - Version extraction from module headers
- Workflow reference documentation generation
  - Parses GitHub Actions YAML workflows
  - Catalogs jobs and trigger events
  - Links to source workflow files
- Search index generation for documentation sites
- Version management for multi-version documentation builds
- Support for multiple source directories
- Embedded Python logic (no external dependencies)
- Comprehensive error handling and warnings
- GitHub Actions output integration
- Full documentation and examples

### Changed

- Ported from `ghcommon/.github/workflows/scripts/docs_workflow.py`
- Converted to composite action format for direct GitHub Actions integration
- Improved error messages with GitHub Actions annotations

### Removed

- Command-line only interface (now integrated as action)
- Dependency on workflow_common module (all logic embedded)
