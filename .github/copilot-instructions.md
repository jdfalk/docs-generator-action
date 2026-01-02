<!-- file: .github/copilot-instructions.md -->
<!-- version: 1.0.1 -->
<!-- guid: d8e9f0a1-2345-6789-abcd-ef1234567890 -->

# AI Agent Instructions (Standard)

- Use VS Code tasks for non-git operations; prefer MCP GitHub tools for git operations.
- Commits must use conventional commits (type(scope): description).
- Include versioned headers in docs/configs and bump versions on changes.
- This is a composite GitHub Action that embeds Python logic directly in action.yml.
- All changes to documentation generation logic should be made in the embedded Python script within action.yml.
- Follow GitHub Actions composite action best practices.
- Provide concise plans and progress updates.

## Repository Context

This repository provides a composite GitHub Action that automatically generates and maintains project documentation including API docs, workflow catalogs, and search indexes.

**Key Files:**

- [action.yml](action.yml) - Main composite action with embedded Python script
- [README.md](README.md) - Comprehensive documentation and examples
- [CHANGELOG.md](CHANGELOG.md) - Version history

**Documentation Generation:**

- API documentation from code comments
- Workflow catalog generation
- Search index creation
- README generation and updates

**Design Pattern:**

- Composite action using `shell: python` to embed the entire Python script
- Single file action.yml contains all logic (no separate src/ directory needed)
- All environment variables passed as action inputs
- Outputs exposed via GitHub Actions output commands
