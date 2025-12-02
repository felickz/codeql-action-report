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

## Requirements

- Python must be available in the runner environment

## License

MIT