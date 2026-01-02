# Documentation Generator Action

<!-- file: README.md -->
<!-- version: 1.0.0 -->
<!-- guid: 9a0b1c2d-3e4f-5a6b-7c8d-9e0f1a2b3c4d -->

A GitHub Actions composite action that generates API documentation from Python
source code and workflow reference documentation from GitHub Actions workflow
YAML files. This action replaces the `docs_workflow.py` script from ghcommon.

## Features

- **API Documentation Generation**: Extracts docstrings from Python modules and
  generates Markdown documentation
- **Workflow Reference Documentation**: Parses GitHub Actions workflows and
  creates a searchable catalog
- **Flexible Version Management**: Supports multiple documentation versions
  (e.g., "latest", "stable-v1.0")
- **Search Index Generation**: Creates a JSON search index for documentation
  site integration
- **Modular Operation**: Generate API docs, workflows, or both independently
- **Embedded Python Logic**: No external dependencies; all logic embedded in the
  composite action

## Usage

### Basic Usage

Generate all documentation with defaults:

```yaml
- name: Generate Documentation
  uses: jdfalk/docs-generator-action@v1
  with:
    output-dir: docs/generated
```

### With Custom Source Directories

```yaml
- name: Generate Documentation
  uses: jdfalk/docs-generator-action@v1
  with:
    source-dirs: 'scripts,src/helpers,tools'
    workflows-dir: '.github/workflows'
    output-dir: docs/generated
    doc-version: 'v1.0.0'
```

### API Documentation Only

```yaml
- name: Generate API Docs
  uses: jdfalk/docs-generator-action@v1
  with:
    source-dirs: 'src'
    output-dir: docs/api
    generate-workflows: 'false'
    generate-search-index: 'false'
```

### Workflow Documentation Only

```yaml
- name: Generate Workflow Docs
  uses: jdfalk/docs-generator-action@v1
  with:
    workflows-dir: '.github/workflows'
    output-dir: docs/workflows
    generate-api: 'false'
```

### In a Complete Documentation Build Workflow

```yaml
name: Build Documentation

on:
  push:
    branches: [main, develop]
    paths:
      - '.github/workflows/**'
      - 'scripts/**'
      - 'docs/source/**'
  workflow_dispatch:

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate API and Workflow Documentation
        uses: jdfalk/docs-generator-action@v1
        with:
          source-dirs: '.github/workflows/scripts,tools'
          workflows-dir: '.github/workflows'
          output-dir: docs/generated
          doc-version: 'latest'

      - name: Upload Documentation Artifact
        uses: actions/upload-artifact@v3
        with:
          name: generated-docs
          path: docs/generated

      - name: Deploy Documentation
        if: github.ref == 'refs/heads/main'
        run: |
          # Your deployment logic here
          echo "Generated files:"
          echo '${{ steps.generate.outputs.generated-files }}'
```

## Inputs

### `source-dirs`

- **Description**: Comma-separated list of source directories containing Python
  modules to document
- **Required**: false
- **Default**: `.github/workflows/scripts`

### `workflows-dir`

- **Description**: Directory containing GitHub Actions workflow YAML files
- **Required**: false
- **Default**: `.github/workflows`

### `output-dir`

- **Description**: Root directory where documentation will be generated
- **Required**: true

### `doc-version`

- **Description**: Documentation version string (e.g., "1.0.0", "latest",
  "stable-v2")
- **Required**: false
- **Default**: Derives from environment or defaults to "latest"

### `generate-api`

- **Description**: Whether to generate API documentation from Python docstrings
- **Required**: false
- **Default**: `true`

### `generate-workflows`

- **Description**: Whether to generate workflow reference documentation
- **Required**: false
- **Default**: `true`

### `generate-search-index`

- **Description**: Whether to generate a search index JSON file
- **Required**: false
- **Default**: `true`

## Outputs

### `build-status`

The build status result: `success`, `partial`, or `failed`

### `generated-files`

JSON array of paths to generated documentation files

### `api-docs-count`

Number of API documentation files generated

### `doc-version`

The resolved documentation version used for this build

### `output-dir`

The absolute path to the documentation output directory

### `search-index-path`

Path to the generated search index JSON file (if enabled)

## Generated Documentation Structure

```
docs/generated/
├── latest/
│   ├── api/
│   │   ├── module1.md
│   │   ├── module2.md
│   │   └── ...
│   └── workflows.md
├── stable-v1.0/
│   ├── api/
│   │   └── ...
│   └── workflows.md
├── versions.json           # List of all documentation versions
└── search-index.json       # Search index for all versions
```

## Output Formats

### API Documentation

For each Python module, generates a Markdown file with:

- Module overview and docstring
- Function signatures and documentation
- Class definitions, docstrings, and methods
- Source file reference

**Example:**

```markdown
# Module `ci_workflow`

**Version:** 1.0.1

CI workflow orchestration utilities.

## Functions

### `build_matrix(...)`

Build a test matrix configuration...

## Classes

### `MatrixConfig`

Configuration for test matrix generation.

#### Methods

- `from_yaml(path: Path)` - Load configuration from YAML file
```

### Workflow Documentation

Generates a comprehensive catalog of all workflows with:

- Workflow name and description
- Trigger events
- Jobs and their configuration
- File references

**Example:**

```markdown
# Workflow Catalog

## CI Tests

Runs test suite on pull requests.

**File:** `.github/workflows/ci-tests.yml` **Triggers:** pull_request, push

### Jobs

- **test-unit**: runs-on `ubuntu-latest`
- **test-integration**: runs-on `ubuntu-latest`
```

### Search Index

JSON file for integration with documentation sites:

```json
[
  {
    "title": "ci_workflow",
    "path": "latest/api/ci_workflow.md"
  },
  {
    "title": "Workflows",
    "path": "latest/workflows.md"
  }
]
```

## Version Management

The action automatically manages multiple documentation versions:

- **Development**: Use "latest" (default)
- **Release**: Use semantic version (e.g., "v1.0.0")
- **Branch-based**: Use branch name if prefixed with "stable-"

The `versions.json` file tracks all available versions for dynamic documentation
site navigation.

## Error Handling

- **Missing directories**: Warnings are issued; execution continues
- **Parse errors**: Individual files are skipped with warnings
- **Malformed YAML**: Workflows are skipped with warnings
- **Overall failure**: Action fails if no documentation could be generated

## Example: Complete Documentation Site

```yaml
name: Documentation

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate documentation
        id: docs
        uses: jdfalk/docs-generator-action@v1
        with:
          source-dirs: 'scripts,src'
          workflows-dir: '.github/workflows'
          output-dir: site/docs
          doc-version: '' # Auto-derive

      - name: Deploy to GitHub Pages
        uses: actions/upload-artifact@v3
        with:
          name: documentation
          path: site/docs

      - name: Log Summary
        run: |
          echo "## Documentation Build Summary" >> $GITHUB_STEP_SUMMARY
          echo "- **Status**: ${{ steps.docs.outputs.build-status }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Version**: ${{ steps.docs.outputs.doc-version }}" >> $GITHUB_STEP_SUMMARY
          echo "- **API Docs**: ${{ steps.docs.outputs.api-docs-count }} files" >> $GITHUB_STEP_SUMMARY
          echo "- **Output**: ${{ steps.docs.outputs.output-dir }}" >> $GITHUB_STEP_SUMMARY
```

## Implementation Notes

- **Python Embedding**: This is a composite action with embedded Python 3 logic
- **No External Dependencies**: Uses only Python standard library and optional
  PyYAML (auto-installed)
- **YAML Support**: Automatically installs PyYAML if needed for workflow parsing
- **Cross-Platform**: Works on any GitHub Actions runner (Linux, macOS, Windows)

## Related

- **Source**: Replaces `ghcommon/.github/workflows/scripts/docs_workflow.py`
- **Original Workflow**: `documentation.yml` in ghcommon repository
- **Documentation Format**: Markdown compatible with static site generators
  (Jekyll, Hugo, Docusaurus, etc.)

## License

Same as parent repository

## Contributing

See parent repository's CONTRIBUTING.md
