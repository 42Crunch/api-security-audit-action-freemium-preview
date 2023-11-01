# GitHub Action: 42Crunch Free Version - REST API Static Security Testing

The REST API Static Security Testing action locates REST API contracts that follow the OpenAPI Specification (OAS, formerly known as Swagger) and runs thorough security checks on them. Both OAS v2 and v3.0.x are supported, in both JSON and YAML format.

This Github action is working in freemium mode: organizations can run 25 audits per month per repository, with a maximum of 3 repositories per organisation.

You can use this action in the following scenarios:
- Add automatic static API security testing (SAST) task to your CI/CD workflows.
- Perform these checks on pull request reviews and/or code merges.
- Flag the located issues in GitHub's Security / Code Scanning Alerts.

## Discover APIs in your repositories

By default, this action will:

1. Look for any `.json` and `.yaml/.yml` files in the repository.
2. Pick the files that use OpenAPI/Swagger 2.0/3.0x schemas.
3. Perform security audit on each OpenAPI definitions.

## Action inputs

### `upload-to-code-scanning` (GitHub Actions only)

Upload the audit results in SARIF format to [Github Code Scanning](https://docs.github.com/en/github/finding-security-vulnerabilities-and-errors-in-your-code/about-code-scanning).  Note that the workflow must have specific permissions for this step to be successful. This assumes you have Github Advanced security enabled.
Default is `false`.

```YAML
...
jobs:
  run_42c_audit:
    permissions:
      contents: read # for actions/checkout to fetch code
      security-events: write # for results upload to Github Code Scanning
...
```

### `enforce-sqg`

If set to `true`, the task with return a failure when security quality gates (SQG) criteria have failed.
If set to `false`, the action reports SQG failures scenarios without enforcing them (i.e. give a grace period to development teams before you start breaking builds).

Default is `false`.  

### `log-level`

Sets the level of details in the action logs, one of: `FATAL`, `ERROR`, `WARN`, `INFO`, `DEBUG`. 
Default is `INFO`.

### `data-enrich`

Enrichs the OpenAPI file leveraging 42Crunch default data dictionary. For each property with a standard format (such as uuid or date-time), patterns and constraints will be added to the OpenAPI file before it is audited.

Default is ` true`.

### `sarif-report`

Converts the audit raw JSON format to SARIF and saves the results into a specified file.
If not present, the SARIF report is not generated.

### `export-as-pdf`

Exports the audit report highlights as PDF.

If not present, the PDF report is not generated.

## Examples

```yaml
- name: 42crunch-static-api-testing
        uses: 42Crunch/api-security-audit-action-freemium@v1
        with:
        	upload-to-code-scanning: true
        	enforce-sqg: false
          sarif-report: 42Crunch_AuditReport_${{ github.run_id }}.SARIF
          export-as-pdf: audit-report-${{ github.run_id }}.pdf
          log-level: info
```

A typical workflow which checks the contents of the repository, runs Security Audit on each of the OpenAPI files found in the project and saves the SARIF file as artifact would look like this:

```yaml
name: "core-workflow"

# follow standard Code Scanning triggers
on: 
  pull_request:
    # The branches below must be a subset of the branches above
    branches: [ main ]

jobs:
  run_42c_audit:
    permissions:
      contents: read # for actions/checkout to fetch code
      security-events: write # for results upload to Github Code Scanning
    runs-on: ubuntu-latest
    steps:
      - name: checkout repo
        uses: actions/checkout@v3
      - name: 42crunch-static-api-testing
        uses: 42Crunch/api-security-audit-action-freemium@v1
        with:
          # Upload results to Github code scanning
          # Set to false if you don't have Github Advanced Security.
          upload-to-code-scanning: true
          log-level: info
          sarif-report: 42Crunch_AuditReport_${{ github.run_id }}.SARIF
          enforce-sqg: true
      - name: save-audit-report
        if: always()        
        uses: actions/upload-artifact@v3
        with:
          name: 42Crunch_AuditReport_${{ github.run_id }}
          path: 42Crunch_AuditReport_${{ github.run_id }}.SARIF
          if-no-files-found: error
```
