# HackMeHarder

HackMeHarder is a lightweight combined SAST (Static Application Security Testing) and DAST (Dynamic Application Security Testing) tool focused on Python web applications (Flask-friendly). It performs static analysis to find risky code patterns and then runs targeted dynamic attacks against a live application to confirm which findings are exploitable. The result is higher-confidence vulnerability reports with fewer false positives.

This repository contains:

- A static analyzer based on AST (in `SAST/`) that performs taint-style checks.
- A dynamic scanner (DAST) that crawls a running web app, injects payloads and analyzes responses (under `DAST/`).
- A correlation engine that maps SAST findings to DAST confirmations for high-confidence results.
- A convenient CLI (`cli.py`) to run SAST, DAST or the full correlated scan.

## Table of contents

- Quick start
- Features
- Prerequisites
- Installation
- CLI usage and examples
- Configuration (rules and payloads)
- How it works (architecture)
- Development & tests
- Troubleshooting
- Contributing
- License

## Quick start (recommended)

1. Create and activate a virtual environment (Windows PowerShell):

```powershell
python -m venv .venv; .\.venv\Scripts\Activate.ps1
```

2. Install dependencies:

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
# (optional) for editable install so `cli.py` imports work as package
pip install -e .
```

3. Run a SAST scan on your local source tree:

```powershell
python cli.py sast C:\path\to\your\project
```

4. Run a DAST scan against a running instance (include scheme):

```powershell
python cli.py dast http://127.0.0.1:5000
```

5. Run a full correlated scan (SAST findings validated by DAST):

```powershell
python cli.py full-scan C:\path\to\your\project http://127.0.0.1:5000
```

## Features

- AST-based taint analysis for Python code (SAST).
- Crawler that discovers attackable endpoints and forms (DAST).
- Attack payloads for SQLi, XSS, Path Traversal and Unvalidated Redirect checks.
- Response analysis heuristics to differentiate true positives from noise.
- Correlation engine that uses static findings to prioritize and confirm dynamic attacks.

## Prerequisites

- Python 3.8+ (tested on 3.8–3.11). Use a virtualenv.
- Git (optional, to clone repository).
- A running web application (for DAST and full-scan).

Check `requirements.txt` for Python package dependencies used by the project.

## Installation

Clone the repo and install requirements:

```powershell
git clone https://github.com/ved-bankeshwar/HackMeHarder.git
cd HackMeHarder
python -m venv .venv; .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
pip install -e .  # optional: installs package in editable mode
```

Note: Editable install makes imports work cleanly for local development.

## CLI usage

The CLI entrypoint is `cli.py`. It exposes three subcommands:

- `sast <path>` — Run static analysis on a local directory and print findings.
- `dast <url>` — Run the crawler + attack pipeline against a live URL.
- `full-scan <path> <url>` — Run SAST, then validate SAST findings with targeted DAST attacks; prints confirmed vulnerabilities.

Examples (PowerShell):

```powershell
# SAST only
python cli.py sast C:\path\to\source

# DAST only
python cli.py dast http://127.0.0.1:5000

# Full correlated scan
python cli.py full-scan C:\path\to\source http://127.0.0.1:5000
```

The CLI uses `click` for friendly terminal output. See `cli.py` for argument validation and output formatting.

## Configuration

- `rules.yaml` — Contains SAST taint-analysis rules and patterns used by the `FlaskTaintAnalyzer` (look in the repository root). Edit this file to add/modify rules for your codebase.
- `DAST/attack_payload/payloads.py` — Contains payload definitions used by the DAST engine. Add or tune payloads there.

When adding rules or payloads, prefer small, focused changes and include example test cases in `vul_app/` if you add a new detection type.

## How it works (high level)

- SAST: `SAST_check.py` walks Python files, parses ASTs and runs `SAST.vulnerability_scanner.FlaskTaintAnalyzer` to collect potential taint flows and risky usages (e.g., unsanitized inputs passed to query execution or templates).
- DAST: `DAST/main_controller/dast_cli.py` drives the dynamic pipeline:
  - `DAST.crawlers.index.crawl` discovers endpoints and forms.
  - `DAST.attack_payload.attack_engine.send_malicious_requests` injects payloads from `PAYLOADS`.
  - `DAST.analysis_engine.index` contains per-vulnerability analyzers (SQLi, XSS, Path Traversal, Redirects) to look for signs of successful exploitation.
- Correlation: `correlation_engine.py` takes SAST findings and maps them to discovered endpoints / parameters so DAST can prioritize which parameters to test more aggressively. Confirmed findings are returned by `run_correlated_scan` and printed by the CLI.

## Development & tests

Project layout (important files/folders):

- `cli.py` — CLI entrypoint.
- `SAST/` — Static analysis modules.
- `DAST/` — Dynamic scanning modules (crawler, attack payloads, analyzers).
- `correlation_engine.py` — Correlates SAST and DAST results.
- `vul_app/` — Example vulnerable app for testing (handy for local test runs).

To run unit/integration-like checks quickly:

1. Start the vulnerable app (if you want to test DAST):

```powershell
# inside repository
python vul_app/test.py
```

2. Run DAST against it in another shell:

```powershell
python cli.py dast http://127.0.0.1:5000
```

There are no formal pytest tests included by default. If you add tests, prefer `pytest` and add a simple `make test` or GitHub Actions workflow.

## Troubleshooting

- Invalid URL errors from the CLI: ensure you include scheme (http:// or https://).
- Parser errors during SAST: some dynamic or syntax-variant Python files can fail to parse; SAST gracefully skips files it cannot parse and reports the error.
- DAST finds no endpoints: ensure the target URL is reachable and that the crawler is not blocked by authentication or robots. Consider using a test instance that serves pages with forms.

## Security & safety

This tool performs active attacks when running DAST. Only run against targets you own or have explicit permission to test. Misuse may be illegal and harmful.

## Contributing

1. Fork the repository and create a feature branch.
2. Add tests for new behavior where possible (SAST rule, payload or analyzer).
3. Run manual tests against `vul_app` to demonstrate detection.
4. Open a PR with an explanation and examples.

## License

See `PKG-INFO` inside `hackmeharder.egg-info/` for metadata; add a `LICENSE` file to make the license explicit if this repo is published publicly.

---

If you'd like, I can also:

- Add a short example `rules.yaml` and annotate the fields.
- Create a tiny `tests/` suite with a couple of pytest cases that validate SAST detection and DAST payload analysis using `vul_app`.

Tell me which of those you'd like next and I'll implement it.
Here is a complete README.md file for your project.

HackMeHarder
HackMeHarder is a lightweight, correlated SAST and DAST scanner for Python Flask applications, built during the Code Cortex 2.0 hackathon. It analyzes source code to find potential vulnerabilities and then launches targeted attacks against a live server to confirm them.

The tool combines "white-box" static analysis (SAST) with "black-box" dynamic analysis (DAST) to provide high-confidence results with low false positives.

Installation
You can install HackMeHarder directly from its GitHub repository using pip. Ensure you have Python and Git installed on your system.

Bash

pip install git+https://github.com/ved-bankeshwar/HackMeHarder.git@ved
Usage
The tool provides three main commands for security testing.

Standalone SAST Scan
Analyze a local source code directory for potential vulnerabilities. This method is fast and identifies the exact line of problematic code.

Command:

Bash

hackmeharder sast <path_to_source_code>
Example:

Bash

hackmeharder sast C:\Users\Prasad\Documents\GitHub\Test_hacking
Standalone DAST Scan
Run a crawl-and-attack scan against a live, running web application. This method tests the application from an attacker's perspective.

Command:

Bash

hackmeharder dast <live_application_url>
Example:

Bash

hackmeharder dast https://flask-vulnerable-app.onrender.com
Full Correlated Scan (Recommended)
This is the most powerful feature. It runs the full SAST+DAST pipeline, using the static analysis results to guide the dynamic attacks. This provides a final report of confirmed, exploitable vulnerabilities.

Command:

Bash

hackmeharder full-scan <path_to_source_code> <live_application_url>
Example:

Bash

hackmeharder full-scan C:\Users\Prasad\Documents\GitHub\Test_hacking https://flask-vulnerable-app.onrender.com






