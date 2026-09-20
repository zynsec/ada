<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/ada-free-logo-dark.svg">
  <img src="assets/ada-free-logo-light.svg" width="300" alt="Ada (free)">
</picture>

**Dependency security, aggregated.** Ada collects, audits, and consolidates
dependency findings from every package manager and external scanner into one
branded report.

[Install](#installation) · [Commands](#commands)

</div>

---

Ada is a free, open-source dependency security tool from
[Zyenra Security](https://zyenra.com). It is a self-contained Go binary with no
runtime dependencies and an embedded default configuration.

> <picture>
>   <source media="(prefers-color-scheme: dark)" srcset="assets/ada-pro-logo-dark.svg">
>   <img src="assets/ada-pro-logo-light.svg" width="170" alt="Ada Pro">
> </picture>
>
> **Ada Pro** is the complete platform: full SBOMs (CycloneDX, SPDX),
> reachability-aware findings, continuous monitoring, license compliance,
> private registries, CI/CD auto-fix, and it runs inside your AI coding agent.
>
> **[Get Ada Pro](https://zyenra.com/products)**

## Features

- **Multi-project detection**: automatically identifies npm, Composer, and other project types
- **External scanner support**: consumes JSON output from Snyk, npm audit, composer audit, and merges it into a single report
- **Dependency collection**: walks source trees to find and classify dependency manifests (scannable vs vendored)
- **OSV.dev integration**: queries OSV.dev for vulnerabilities in vendored and bundled dependencies
- **Report merging**: consolidates multiple scan results into one unified HTML or JSON report
- **Custom branding**: configurable company theming, logos, and colors via `~/.config/ada.config`
- **Zero dependencies**: self-contained Go binary with embedded default configuration

## Installation

```bash
git clone https://github.com/rvzsec/ada.git
cd ada
go build -o ada ./cmd/ada
sudo mv ada /usr/local/bin/ada
```

**Prerequisites**: Go 1.24+

## Commands

### `ada audit` - Run security audits directly

Navigate to a project directory and run audits. Ada auto-detects project types
(npm, Composer, and more) and runs the appropriate audit tools.

```bash
ada audit              # Generate both JSON and HTML reports
ada audit --json       # JSON only
ada audit --html       # HTML only
```

**Output**: `ada-audit-report.json`, `ada-report.html`

### `ada report --from-json` - Generate reports from external scanner output

Consume JSON output from external scanners (for example Snyk) and generate
consolidated branded reports. Auto-detects the input format.

```bash
ada report --from-json scan-results.json --html             # Single file to HTML
ada report --from-json scan1.json scan2.json --html --json  # Merge multiple to HTML + JSON
```

This is the primary integration point for CI/CD pipelines where scanning is
handled by tools like Snyk, and Ada handles report generation.

**Output**: `ada-report.html`, `ada-audit-report.json`

### `ada collect` - Collect dependency manifests from source

Walks a source directory, finds dependency manifest and lock files, copies them
preserving structure, and classifies each target as scannable or
vendored/bundled.

```bash
ada collect --source ./repo --out ./deps
```

**Output**: `ada-collect-manifest.json` in the output directory

### `ada osv` - Scan vendored dependencies via OSV.dev

Reads a collect manifest and queries [OSV.dev](https://osv.dev) for known
vulnerabilities in vendored libraries. Output is compatible with
`ada report --from-json`.

```bash
ada osv --manifest ada-collect-manifest.json           # JSON report (default)
ada osv --manifest ada-collect-manifest.json --html    # HTML report
```

**Output**: `ada-osv-report.json`

## CI/CD integration

Ada is designed to plug into CI/CD pipelines as the report consolidation layer.
A typical flow:

```
Scanner (Snyk / npm audit / ...)        Ada
----------------------------------      ------------------------------------------
snyk test --json -> result1.json --.
snyk test --json -> result2.json --+--> ada report --from-json *.json --html --json
snyk test --json -> result3.json --'            |
                                                v
                                        ada-report.html        (branded)
                                        ada-audit-report.json  (machine-readable)
```

## Configuration

Create `~/.config/ada.config` to customize branding:

```json
{
  "theme": {
    "primaryColor": "#ff8f1a",
    "headerBackground": "#ff8f1a",
    "headerTextColor": "#fff"
  },
  "company": {
    "title": "Your Company",
    "report_heading": "Dependency Security Analysis Report",
    "logo_link": "https://yourcompany.com/logo.png",
    "favicon_link": "https://yourcompany.com/favicon.ico",
    "website": "https://yourcompany.com"
  },
  "report": {
    "title": "Dependency Security Report",
    "description": "Security vulnerability analysis for dependencies"
  }
}
```

Falls back to embedded defaults if no config file is present.

## Project structure

```
ada/
├── cmd/ada/           # CLI entry point
│   └── main.go
├── internal/          # Core logic
│   ├── audit.go       # Audit execution
│   ├── collect.go     # Dependency file collection
│   ├── config.go      # Configuration management
│   ├── config.json    # Embedded default config
│   ├── osv.go         # OSV.dev integration
│   ├── project.go     # Project type detection
│   └── reports.go     # Report generation (JSON/HTML)
├── go.mod
└── README.md
```

## Credits

Inspired by [snyk-to-html](https://github.com/snyk/snyk-to-html) for report card
structure and styling.

Maintained by [Zyenra Security](https://zyenra.com).
