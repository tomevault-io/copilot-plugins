## pqc-watch-github

> You are an expert DevSecOps engineer and cryptographer. Your task is to generate the codebase for a Proof of Concept (PoC) named `pqc-watch-github`.

# Context
You are an expert DevSecOps engineer and cryptographer. Your task is to generate the codebase for a Proof of Concept (PoC) named `pqc-watch-github`. 

# Goal
Create a system that uses Semgrep to scan GitHub Pull Requests for hardcoded legacy cryptography and automatically opens a single, consolidated remediation ticket using the GitHub REST API.

# Required Files & Specifications

Please generate the code for the following files:

1. `rules/pqc-legacy-crypto.yml` (Semgrep Rules)
   - Write custom rules to detect legacy RSA in Python (e.g., `cryptography.hazmat.primitives.asymmetric.rsa`) and Java (e.g., `java.security.KeyPairGenerator.getInstance("RSA")`).
   - **Critical:** Add path exclusions to ignore all test environments (e.g., `tests/**`, `mocks/**`, `*Test.java`) so we only scan production-bound code.
   - Include remediation messages advising a switch to PQC standards like ML-KEM (Kyber).

2. `.github/workflows/pqc-scan.yml` (GitHub Action)
   - Trigger on pull requests to the main branch.
   - Set up Python 3.11, install Semgrep and `requirements.txt`.
   - Run Semgrep using the custom rule file and output the results to a JSON file (`semgrep-results.json`). Don't let the pipeline crash if findings are found.
   - Run a Python script (`issue_generator.py`) to process the JSON. Ensure the job has `issues: write` and `pull-requests: write` permissions.

3. `scripts/issue_generator.py` (Action Logic)
   - Read the Semgrep JSON output.
   - If legacy crypto is found, use `PyGithub` to aggregate the findings.
   - Open exactly ONE GitHub issue for the PR detailing all files and line numbers where legacy crypto was found. Do not spam an issue per finding.
   - Check existing open issues to ensure you don't open duplicate tickets for the same PR.

4. `scripts/requirements.txt`
   - Include the necessary dependencies (e.g., `PyGithub`).

5. `README.md`
   - Write a brief overview of the PoC, explaining the architecture (Semgrep + GitHub Actions) and how to configure it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abokov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/abokov)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
