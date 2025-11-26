# codeql-action-report

A GitHub Action that generates and uploads SARIF HTML reports from CodeQL analysis results.

## Description

This action is designed to run after CodeQL init/analyze steps and will:
1. Install sarif-tools
2. Generate a SARIF HTML report from the analysis results
3. Upload the HTML report as an artifact

## Usage

Add this action to your workflow after the CodeQL analyze step:

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
        id: analyze
        uses: github/codeql-action/analyze@v3
          
      - name: Generate SARIF Report
        uses: felickz/codeql-action-report@main
        with:
          sarif: ${{ steps.analyze.outputs.sarif-output }}
          language: ${{ matrix.language }}
```

## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `sarif` | Path to the SARIF output directory (use `steps.<analyze-step-id>.outputs.sarif-output` from codeql-action/analyze) | Yes |
| `language` | The CodeQL language to generate a report for (e.g., javascript, python, csharp) | Yes |

## Outputs

This action uploads an artifact named `sarif-html-report-{language}` containing the HTML report.

## Requirements

- Python must be available in the runner environment
- The `sarif` input must point to the SARIF output directory from the CodeQL analyze step

## License

MIT