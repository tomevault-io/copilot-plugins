## openmri

> OpenMRI is a local viewer for a person's own MRI and CT studies. A Node.js

# OpenMRI: notes for coding agents

OpenMRI is a local viewer for a person's own MRI and CT studies. A Node.js
server (vinext, Next.js style) serves the app on `127.0.0.1:4173`; Python
workers convert DICOM and NIfTI archives. Everything stays on the computer.

## Setting it up for a user

When someone asks you to install, run, or open OpenMRI, do exactly this.

1. Make sure these are installed. Install what is missing:
   - Git.
   - Node.js 22.13 or newer. On macOS: `brew install node`. On Linux use
     https://nodejs.org or nvm; distribution packages are usually too old.
   - Python 3.12, 3.13, or 3.14 with `venv`. On macOS:
     `brew install python@3.12`. On Debian or Ubuntu:
     `sudo apt install python3 python3-venv`.
2. Clone and start with the demo study:

   ```sh
   git clone https://github.com/lev1nson/OpenMRI.git
   cd OpenMRI
   npm run demo
   ```

   On the first run this installs the JavaScript dependencies, builds the app,
   creates the Python environment in `.venv/`, and starts the server in the
   background. That downloads about 1 GB and takes a few minutes. Then it
   imports `demo/jane-head-mri.zip` as patient Jane and opens the app in the
   default browser. Jane's study is the first card under Recent studies; one
   click opens it. Running it again is safe and does not duplicate anything.
   Add `-- --no-open` when there is no browser to open.

3. Check that it works:

   ```sh
   npm run status                                # running, with pid and log path
   curl -s http://127.0.0.1:4173/api/health      # {"app":"openmri",...}
   ```

4. Tell the user the address, http://127.0.0.1:4173, and these commands:
   `npm run down` stops the server, `npm run up` starts it and opens the
   browser. The user imports their own scans with **Import MRI** in the app:
   a ZIP of DICOM or NIfTI files, up to 2 GB.

If something fails, read `.openmri-server.log` or run
`bash scripts/server.sh logs`.

| Symptom                                             | Fix                                                                     |
| --------------------------------------------------- | ----------------------------------------------------------------------- |
| `OpenMRI needs Python 3.12, 3.13 or 3.14`           | Install one of them, then `npm run setup`                               |
| `Port 4173 is used by another application`          | `npm run status`, then stop that process                                |
| The page says the viewer module could not be loaded | Reload the page; the app was rebuilt while it was open                  |
| The 3D view stays black                             | The browser needs WebGL 2; try current Chrome, Edge, Firefox, or Safari |

## Rules

- Keep the server on `127.0.0.1`. It has no login. Never bind it to
  `0.0.0.0`, never tunnel it (ngrok, Cloudflare, SSH), never deploy it.
- The data directory `.openmri/` holds medical images and personal details.
  Never commit it, upload it, paste its contents, or send it anywhere. Do not
  open the user's scans or read patient details unless they ask you to.
- Do not import the user's own archives for them unless they ask and give you
  the file.

## Working on the code

```sh
npm run dev          # dev server on 127.0.0.1:4173
npm run format       # oxfmt
npm run check        # lint, typecheck, Node tests
npm run test:import  # Python tests, including the demo import
```

Run all three checks before you finish. The README describes the project
layout; `CONTRIBUTING.md` has the rules for changes. OpenMRI is a viewer: it
must not detect, measure, or diagnose. The demo study is the only real
scan allowed in the repository.

---
> Source: [lev1nson/OpenMRI](https://github.com/lev1nson/OpenMRI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
