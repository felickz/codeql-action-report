# codeql-action-report

A GitHub Action that generates and uploads SARIF HTML reports from CodeQL analysis results, with optional security and compliance attestation features.

## Description

This action is designed to run after CodeQL init/analyze steps and will:
1. Install sarif-tools
2. Generate SARIF HTML reports for all .sarif files in the output directory
3. Upload the HTML reports as an artifact
4. Optionally fetch GitHub Advanced Security status
5. Optionally run OpenSSF Scorecard analysis
6. Optionally generate and attest SBOM
7. Generate comprehensive security and compliance reports with attestations

## Usage

### Basic Usage (Recommended)

Simply add this action after the CodeQL analyze step. It will automatically find SARIF files using the `CODEQL_ACTION_SARIF_RESULTS_OUTPUT_DIR` environment variable set by codeql-action:

```yaml

...

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

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3

      - name: Generate SARIF Report
        uses: felickz/codeql-action-report@main
```

### Custom Configuration

You can customize which features are enabled using the `features` input. By default, all features are enabled:

```yaml
      - name: Generate SARIF Report with Custom Features
        uses: felickz/codeql-action-report@main
        with:
          features: |
            advanced-security: true
            scorecard: true
            sbom: true
```

To disable specific features:

```yaml
      - name: Generate SARIF Report (Scorecard only)
        uses: felickz/codeql-action-report@main
        with:
          features: |
            advanced-security: false
            scorecard: true
            sbom: false
```

### Custom SARIF Input Directory

If you need to specify a custom SARIF directory, you can use the `sarif-input` parameter:

```yaml
      ...

      - name: Perform CodeQL Analysis
        id: analyze
        uses: github/codeql-action/analyze@v3

      - name: Generate SARIF Report
        uses: felickz/codeql-action-report@main
        with:
          sarif-input: ${{ steps.analyze.outputs.sarif-output }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `sarif-input` | Path to the SARIF output directory. Defaults to `CODEQL_ACTION_SARIF_RESULTS_OUTPUT_DIR` environment variable set by codeql-action. | No | (uses env var) |
| `features` | Security and compliance features to enable. Format: YAML multi-line string with feature flags. Available features: `advanced-security`, `scorecard`, `sbom`. | No | All enabled |

### Features

#### `advanced-security`
- **Default**: `true` (enabled)
- **Description**: Fetches GitHub Advanced Security status from the repository, including:
  - Security and analysis features (Advanced Security, Secret Scanning, etc.)
  - Dependabot vulnerability alerts status
  - Dependabot automated security fixes status
- **Output**: JSON report saved to `security-reports/advanced-security-status.json`

#### `scorecard`
- **Default**: `true` (enabled)
- **Description**: Runs [OpenSSF Scorecard](https://github.com/ossf/scorecard) analysis on the repository
- **Requirements**: Requires appropriate permissions (`actions: read`, `contents: read`, `pull-requests: read`, `checks: read`)
- **Output**: JSON and Markdown reports saved to `security-reports/`

#### `sbom`
- **Default**: `true` (enabled)
- **Description**: Generates Software Bill of Materials (SBOM) in SPDX format
- **Requirements**: Dependency graph must be enabled in repository settings
- **Output**: SPDX JSON file used for attestation

## Outputs

This action uploads the following artifacts:
- **SARIF HTML Reports**: HTML visualization of CodeQL scan results
- **SARIF Files**: Raw SARIF files from CodeQL analysis
- **SBOM**: Software Bill of Materials (if dependency graph is enabled)
- **Security Reports**: GitHub Advanced Security status and OpenSSF Scorecard results (if features are enabled)

## Dependencies

This action relies on the following key dependency:

### [sarif-tools](https://github.com/microsoft/sarif-tools)

**sarif-tools** is a Python package by Microsoft that provides utilities for working with SARIF (Static Analysis Results Interchange Format) files. This action uses sarif-tools to convert SARIF files into human-readable HTML reports.
- **Key Functionality**: All HTML report generation is performed by the `sarif html` command from this package
- **GitHub**: https://github.com/microsoft/sarif-tools
- **PyPI**: https://pypi.org/project/sarif-tools/

### Dependency Graph

Optional - requires enabling GitHub's dependency graph for your repository to generate an SBOM. The SBOM will be used to attest the html and sarif artifacts.

### [OpenSSF Scorecard](https://github.com/ossf/scorecard)

Optional - The action can run OpenSSF Scorecard to assess the security posture of your repository. Scorecard evaluates various security best practices and provides a score for each check.
- **GitHub**: https://github.com/ossf/scorecard
- **Features**: Automated security best practice checks, SARIF output support, detailed scoring

## Security and Compliance Features

### GitHub Advanced Security Status

When enabled (default), this action fetches comprehensive security feature status from your repository:

- **Security & Analysis Features**:
  - Advanced Security status
  - Secret Scanning (standard and push protection)
  - Secret Scanning AI Detection
  - Secret Scanning Validity Checks
  - Secret Scanning Non-Provider Patterns
  
- **Dependabot**:
  - Vulnerability Alerts status
  - Automated Security Fixes status
  - Security Updates status

All data is saved as JSON and included in attestations for audit and compliance purposes.

### OpenSSF Scorecard

When enabled (default), runs comprehensive security checks including:
- Binary artifacts detection
- Branch protection review
- CI/CD tests verification
- Code review practices
- Dangerous workflow patterns
- Dependency update tool usage
- License verification
- Maintained project status
- Pinned dependencies check
- SAST tool usage
- Security policy presence
- Signed releases verification
- Token permissions review
- Vulnerability disclosure
- And more...

Results are displayed in the GitHub Actions summary with detailed scoring and recommendations.

## Attestations

This action generates [in-toto](https://in-toto.io/) attestations for all generated artifacts:
- SARIF HTML reports
- SARIF files  
- Security compliance reports (Advanced Security status, Scorecard results)

If the dependency graph is enabled for the repository, an SBOM in SPDX format ("https://spdx.dev/Document/v2.3" predicate) will also be generated and attested in the DSSE envelope payload. Instructions to validate the attestations are provided in the GitHub Actions summary.

To use artifact attestations in private or internal repositories, you must be on a GitHub Enterprise Cloud plan.

## Required Permissions

For full functionality, ensure your workflow has the following permissions:

```yaml
permissions:
  security-events: write  # Required for CodeQL
  contents: read          # Required for checkout and Scorecard
  actions: read          # Required for Scorecard workflow analysis
  pull-requests: read    # Required for Scorecard PR checks
  checks: read           # Required for Scorecard CI/CD analysis
  id-token: write        # Required for attestations
  attestations: write    # Required for attestations
```

## Requirements

- Python must be available in the runner environment (pre-installed on all GitHub-hosted runners)
- pip package manager (pre-installed on all GitHub-hosted runners)

## License

MIT