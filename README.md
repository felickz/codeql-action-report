# codeql-action-report

A GitHub Action that generates and uploads SARIF HTML reports from CodeQL analysis results.

## Description

This action is designed to run after CodeQL init/analyze steps and will:
1. Install sarif-tools
2. Generate SARIF HTML reports for all .sarif files in the output directory
3. Upload the HTML reports as an artifact

## Usage

### Basic Usage (Recommended)

Simply add this action after the CodeQL analyze step. It will automatically find SARIF files using the `CODEQL_ACTION_SARIF_RESULTS_OUTPUT_DIR` environment variable set by codeql-action:

```yaml
jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        language: ['javascript', 'python']
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3

      - name: Generate SARIF Report
        uses: felickz/codeql-action-report@main
```

### Custom SARIF Input Directory

If you need to specify a custom SARIF directory, you can use the `sarif-input` parameter:

```yaml
      - name: Perform CodeQL Analysis
        id: analyze
        uses: github/codeql-action/analyze@v3

      - name: Generate SARIF Report
        uses: felickz/codeql-action-report@main
        with:
          sarif-input: ${{ steps.analyze.outputs.sarif-output }}
```

## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `sarif-input` | Path to the SARIF output directory. Defaults to `CODEQL_ACTION_SARIF_RESULTS_OUTPUT_DIR` environment variable set by codeql-action. | No |

## Outputs

This action uploads an artifact named `sarif-html-reports` containing HTML reports for all SARIF files found.

## Dependencies

This action relies on the following key dependency:

### [sarif-tools](https://github.com/microsoft/sarif-tools)

**sarif-tools** is a Python package by Microsoft that provides utilities for working with SARIF (Static Analysis Results Interchange Format) files. This action uses sarif-tools to convert SARIF files into human-readable HTML reports.
- **Key Functionality**: All HTML report generation is performed by the `sarif html` command from this package
- **GitHub**: https://github.com/microsoft/sarif-tools
- **PyPI**: https://pypi.org/project/sarif-tools/

### Dependency Graph

Optional - requires enabling GitHub's dependency graph for your repository to generate an SBOM.  The SBOM will be used to attest the html and sarif artifacts.

## Attestations

This action can generate [in-toto](https://in-toto.io/) attestations for the generated SARIF HTML reports and SARIF files. If the dependency graph is enabled for the repository, an SBOM in SPDX format ("https://spdx.dev/Document/v2.3" predicate) will also be generated and attested in the DSSE envelope payload.  Instructions to validate the attestations are provided in the summary.

To use artifact attestations in private or internal repositories, you must be on a GitHub Enterprise Cloud plan.

## Requirements

- Python must be available in the runner environment (pre-installed on all GitHub-hosted runners)
- pip package manager (pre-installed on all GitHub-hosted runners)

## License

MIT