## ezlocalai

> - ezLocalai is designed to run inside of a docker container.

# AI Agent Guidance for ezLocalai development

- ezLocalai is designed to run inside of a docker container.
- There are 3 versions of the docker-compose files
  - `docker-compose.yml` - for CPU only systems and downloads pre-built image from Dockerhub
  - `docker-compose-cuda.yml` - for systems with NVIDIA GPUs and CUDA installed, builds locally from the `cuda.Dockerfile` in the repo using `cuda-requirements.txt`
  - `docker-compose-local.yml` - for local development, builds the image from the Dockerfile in the repo using `requirements.txt`

The assistant can test changes made by running the following command:

```bash
docker compose -f docker-compose-local.yml down && docker compose -f docker-compose-local.yml up --build -d
```

Then by running the following command to see logs:

```bash
docker compose -f docker-compose-local.yml logs -f
```

There are tests for each endpoint in ezLocalai in the `tests.ipynb` file.

---
> Source: [DevXT-LLC/ezlocalai](https://github.com/DevXT-LLC/ezlocalai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
