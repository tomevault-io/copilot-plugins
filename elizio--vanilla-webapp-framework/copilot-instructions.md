## devops-rules

> Docker, CI/CD, and environment management conventions

## **3. DevOps & Deployment Rules**
### **Docker**
- **Rule 3.1**: Ensure `Dockerfile` is optimized for production.
- **Rule 3.2**: Use multi-stage builds to reduce image size.

### **CI/CD (GitHub Actions)**
- **Rule 3.3**: Run tests on every push (`pytest` + frontend checks).
- **Rule 3.4**: Auto-deploy to **DigitalOcean** on `main` branch merges.

### **Environment Management**
- **Rule 3.5**: Use `.venv` for Python dependencies (`pip install -e .`).
- **Rule 3.6**: Freeze dependencies with `pip freeze > requirements.txt`.
- **Rule 3.7**: Include `setup.cfg` and `setup.py` for dependency management and project configuration.

---
> Source: [Elizio/vanilla-webapp-framework](https://github.com/Elizio/vanilla-webapp-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
