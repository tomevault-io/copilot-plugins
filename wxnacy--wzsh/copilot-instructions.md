## wzsh

> 这是 `zsh` 环境的自定义配置，新环境中运行 `installer.sh` 即可安装。

## Overview

这是 `zsh` 环境的自定义配置，新环境中运行 `installer.sh` 即可安装。

- `${HOME}/.zsh` 链接到 `$(pwd)`
- `${HOME}/.zshrc` 链接到 `$(pwd)/zshrc`
- `${HOME}/.zshenv` 链接到 `$(pwd)/zshenv`
- `${HOME}/.zprofile` 链接到 `$(pwd)/zprofile`

## bin

`bin` 目录中存放脚本，已经添加到 `PATH` 中。


## Plugins

每个独立软件包封装起来的插件存放位置，插件目录结构如下

```
plugins
    - wzsh-{plugin-name}
        - bin/
        - config/
        - aliases.zsh
        - installer
        - zprofile
        - zshenv
        - zshrc
```

- `bin` 存放脚本 `zsh` 环境加载的时候会添加到 `PATH` 中，如果脚本名称为 `wzsh-go-version` 则可以使用 `wzsh-go-version` 调用，也可以使用 `z go version` 调用
- `aliases.zsh` `zprofile` `zshenv` `zshrc` 在 `zsh` 环境加载的时候会自动加载
- `installer` 存在插件额外安装脚本，调用 `wzsh-install ${plugin-name}` 可以执行

## 规范

- 优先使用中文编写文档。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wxnacy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wxnacy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
