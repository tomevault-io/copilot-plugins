## wmfactory

> First, I need to clarify the definition of World Model in the following text. The World Model I mentioned is like Genie3, which can create a playable world after inputting an image/text. In this world, you can at least input WASD/up down left right to move. World Lab by Li Fei-fei is also such a work. The representative in the industry is HunYuanWorld1.5, and the representative in the academicis Vid2World Crafting Video Diffusion Models to Interactive World Models.

# World Model Unified Frontend

##  Background
First, I need to clarify the definition of World Model in the following text. The World Model I mentioned is like Genie3, which can create a playable world after inputting an image/text. In this world, you can at least input WASD/up down left right to move. World Lab by Li Fei-fei is also such a work. The representative in the industry is HunYuanWorld1.5, and the representative in the academicis Vid2World Crafting Video Diffusion Models to Interactive World Models. 

However, there is no unified frontend framework for the World Model I defined, like I can choose a model and play different World Models on the same website. This kind of model is of great significance to the academic, because it is equivalent to building a unified evaluation framework like LLMArena, and it is convenient for ordinary people to intuitively judge the quality of a world model.

So, the goal of this project is to build a unified World Model frontend, which can support all the World Models I have researched, and can be easily experienced. It is equivalent to this frontend can choose a model, choose a dataset or directly input an image/text, and support WASD movement, and change the perspective by dragging.

## Code Structure

WMFactory
|--data
|--models
|--frontend
|--utils

data: store the dataset of the World Models such as CSGO etc

models: store the models and codes of the World Models such as Diamond, GameFactory etc

frontend: the frontend code of the World Model Factory. This is the core of this project.

utils: the utils code of the World Model Factory.

## Notes 

### Command Conventions
- In this environment, prepend network-sensitive commands with empty proxy variables except for git clone:
  - `git clone <command>`
  - `http_proxy= https_proxy= HTTP_PROXY= HTTPS_PROXY= <other command>`
- For Python package installation, prefer Tsinghua mirror to speed up and stabilize downloads:
  - `-i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn`

### Common Errors and Fixes
1. `git clone ... Failed to connect ...`
- Cause: restricted or unstable network path.
- Fix: re-run clone with network access enabled and keep proxy vars explicit if needed.

2. `conda create -n diamond ... Permission denied: /home/anaconda3/pkgs/cache/...`
- Cause: global conda cache path permission issues.
- Fix: create a project-local conda env and package cache:
  - `CONDA_PKGS_DIRS=/mnt/server/WMFactory/venvs/.conda_pkgs conda create -p /mnt/server/WMFactory/venvs/diamond python=3.10`

3. `No matching distribution found` / proxy connection errors during pip install
- Cause: proxy/network route issue or slow default index.
- Fix: clear proxy env vars and use Tsinghua mirror:
  - `http_proxy= https_proxy= HTTP_PROXY= HTTPS_PROXY= ... pip install -i https://pypi.tuna.tsinghua.edu.cn/simple ...`

4. `requests.exceptions.ConnectTimeout` when downloading from `huggingface.co`
- Cause: HF upstream timeout.
- Fix: use mirror endpoint for model fetch:
  - `HF_ENDPOINT=https://hf-mirror.com`


## TODO



## Finished

DIAMOND

---
> Source: [Rising0321/WMFactory](https://github.com/Rising0321/WMFactory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
