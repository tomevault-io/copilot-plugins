## openwebui-design

> Dependency management and versioning standards


# Cursor Rules: Dependency Management Standards

rules:
  - id: dependencies.compliance.production_grade
    description: |
      All dependency management must meet federal and government-regulated agency production standards.
      - Dependencies must support production-ready, fully functional, enterprise-grade code
      - No dependencies with known vulnerabilities or incomplete implementations permitted
      - All dependencies must be validated for security compliance (NIST, FISMA, FedRAMP)
      - Dependency updates must be auditable, traceable, and compliant with regulatory requirements
      - Security audits must pass before adding or updating dependencies
      - All dependencies must be documented with rationale and security considerations
      - Dependency management must handle all edge cases and error conditions explicitly
      - Version pinning must ensure reproducible, secure builds
    severity: error
  - id: dependencies.versioning.strategy
    description: |
      Dependency versioning must follow consistent strategy.
      - Pin exact versions for production dependencies
      - Use version ranges for development dependencies when appropriate
      - Document version pinning rationale
      - Use semantic versioning for dependency versions
      - Update dependencies regularly and test updates
      - Document dependency update procedures
    severity: error

  - id: dependencies.versioning.pinning
    description: |
      Critical dependencies must be pinned to specific versions.
      - Pin security-sensitive dependencies to exact versions
      - Pin framework dependencies (FastAPI, Pydantic, etc.) to compatible versions
      - Use version constraints (>=, <=) for non-critical dependencies
      - Document version constraints and compatibility requirements
      - Test with minimum and maximum supported versions
    severity: error

  - id: dependencies.security.updates
    description: |
      Security updates must be applied promptly.
      - Monitor dependencies for security vulnerabilities
      - Apply security patches within 48 hours when available
      - Test security updates before deploying
      - Document security update procedures
      - Maintain security update log
    severity: error

  - id: dependencies.security.audit
    description: |
      Dependency security audits must be conducted regularly.
      - Run dependency vulnerability scans weekly
      - Use automated tools (safety, pip-audit, Dependabot)
      - Review and address vulnerability reports
      - Document audit procedures and tools
      - Maintain audit history
    severity: error

  - id: dependencies.optional.management
    description: |
      Optional dependencies must be clearly documented and managed.
      - Use optional dependency groups (dev, ml, etc.)
      - Document optional dependencies and their purposes
      - Provide clear installation instructions
      - Handle missing optional dependencies gracefully
      - Test with and without optional dependencies
    severity: error

  - id: dependencies.optional.graceful_degradation
    description: |
      Code must handle missing optional dependencies gracefully.
      - Check for optional dependencies before use
      - Provide clear error messages when optional dependencies are missing
      - Document optional dependency requirements
      - Use try/except for optional imports
      - Provide fallback behavior when possible
    severity: error

  - id: dependencies.documentation.requirements
    description: |
      Dependency requirements must be documented.
      - Document all dependencies and their purposes
      - Document minimum Python version requirements
      - Document optional dependencies and installation
      - Document dependency update procedures
      - Keep dependency documentation current
    severity: error

  - id: dependencies.quality.review
    description: |
      New dependencies must be reviewed before addition.
      - Review dependency licenses for compliance
      - Review dependency maintenance status
      - Review dependency security history
      - Review dependency size and impact
      - Document rationale for new dependencies
    severity: error

  - id: dependencies.quality.minimal
    description: |
      Dependencies should be minimal and necessary.
      - Avoid adding dependencies for single-use functionality
      - Prefer standard library over external dependencies
      - Consider maintenance burden of dependencies
      - Review and remove unused dependencies regularly
      - Document dependency removal procedures
    severity: warning

  - id: dependencies.quality.compatibility
    description: |
      Dependency compatibility must be maintained.
      - Test dependency compatibility regularly
      - Document known compatibility issues
      - Maintain compatibility matrix
      - Test with different dependency versions
      - Document upgrade paths for major version changes
    severity: warning

examples:
  - description: "Dependency versioning in pyproject.toml"
    language: toml
    code: |
      [project]
      dependencies = [
          # Core dependencies - pinned for stability
          "fastapi>=0.104.0,<0.105.0",
          "uvicorn[standard]>=0.24.0,<0.25.0",
          "pydantic>=2.0.0,<3.0.0",
          
          # Data processing - compatible versions
          "pandas>=1.5.0",
          "numpy>=1.23.0",
          
          # Web framework
          "flask>=3.0.0",
          "httpx>=0.24.0",
      ]
      
      [project.optional-dependencies]
      dev = [
          "pytest>=7.0.0",
          "black>=23.0.0",
          "ruff>=0.1.0",
          "mypy>=1.0.0",
      ]
      ml = [
          "sentence-transformers>=2.2.0",
          "transformers>=4.30.0",
          "torch>=2.0.0",
      ]

  - description: "Optional dependency handling"
    language: python
    code: |
      # Optional dependency handling
      try:
          import torch
          TORCH_AVAILABLE = True
      except ImportError:
          TORCH_AVAILABLE = False
          torch = None
      
      
      def use_torch_operation(data):
          """Use torch if available, fallback otherwise"""
          if not TORCH_AVAILABLE:
              raise ImportError(
                  "torch is required for this operation. "
                  "Install with: pip install -e '.[ml]'"
              )
          
          # Use torch
          return torch.tensor(data)

  - description: "Dependency security audit script"
    language: python
    code: |
      #!/usr/bin/env python3
      """
      Dependency security audit script
      
      Runs security scans on project dependencies.
      """
      
      import subprocess
      import sys
      
      
      def run_security_audit():
          """Run security audit on dependencies"""
          print("Running dependency security audit...")
          
          # Try safety first
          try:
              result = subprocess.run(
                  ["safety", "check"],
                  capture_output=True,
                  text=True
              )
              if result.returncode != 0:
                  print("Security vulnerabilities found:")
                  print(result.stdout)
                  return False
              print("✓ No vulnerabilities found with safety")
          except FileNotFoundError:
              print("⚠ safety not installed, skipping...")
          
          # Try pip-audit
          try:
              result = subprocess.run(
                  ["pip-audit"],
                  capture_output=True,
                  text=True
              )
              if result.returncode != 0:
                  print("Security vulnerabilities found:")
                  print(result.stdout)
                  return False
              print("✓ No vulnerabilities found with pip-audit")
          except FileNotFoundError:
              print("⚠ pip-audit not installed, skipping...")
          
          return True
      
      
      if __name__ == "__main__":
          success = run_security_audit()
          sys.exit(0 if success else 1)

references:
  - Python packaging: https://packaging.python.org/
  - Safety (security scanner): https://github.com/pyupio/safety
  - pip-audit: https://github.com/pypa/pip-audit
  - Semantic Versioning: https://semver.org/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cassianwolfe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
