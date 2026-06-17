## youbot

> - `app/frontend/` - Vite-powered PWA frontend

# You Bot AI - Skills

## Working with This Project

### Development Commands

```bash
# Install all dependencies
npm install

# Run in development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

### Project Structure

- `app/frontend/` - Vite-powered PWA frontend
- `app/backend/` - Node.js Express API server
- `app/models/` - AI model files (GGUF, ONNX, etc.)
- `app/uploads/user/` - User-uploaded content
- `app/storage/` - App data (SQLite, Vector DB, cache)
- `docker/` - Docker configurations
- `scripts/` - Utility scripts

### Adding Dependencies

```bash
# Frontend dependencies
cd app/frontend && npm install <package>

# Backend dependencies
cd app/backend && npm install <package>
```

### Model Management

Place AI models in `app/models/gguf/` or `app/models/onnx/`. The system will auto-detect and register them.

### Docker Development

```bash
# Build and run
docker compose -f docker/compose/docker-compose.cpu.yml up

# With GPU support
docker compose -f docker/compose/docker-compose.gpu.yml up
```

---
> Source: [amkyawdev/YouBot](https://github.com/amkyawdev/YouBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
