## zzzzzhangyanqiu-github-io

> This repository is a Hugo + PaperMod technical blog deployed to GitHub Pages at

# Project Instructions

This repository is a Hugo + PaperMod technical blog deployed to GitHub Pages at
`https://zzzzzhangyanqiu.github.io/`.

## Technical Accuracy

Technical correctness is the highest priority. It is more important than speed,
polish, or rhythm.

- If a mechanism, term, data flow, configuration behavior, or product detail is
  uncertain, verify it from reliable sources before writing.
- Prefer official documentation for technical claims when available.
- Before publishing a technical explanation, check whether nearby concepts have
  been mixed together, such as routing vs allocation, refresh vs flush, or
  primary shard vs master node.
- If a point cannot be verified, do not present it as fact.

## Blog Structure

- Posts live under `content/posts/<category>/`.
- Front matter uses only `title`, `tags`, `date`, and `categories`.
- Articles with images should use a Hugo leaf bundle:
  `content/posts/<category>/<slug>/index.md`.
- Images for a leaf-bundle article should sit beside `index.md` and be
  referenced with relative paths, such as `![](diagram.svg)`.
- Avoid new `/images/...` references for in-progress articles because local
  Markdown preview does not resolve Hugo `static/` paths well.
- Do not use external image hosting for new assets. Existing migrated images are
  under `static/images/oss/`.
- Editable diagram sources live in `.diagrams/`; final exported images should be
  placed with the article that uses them.

## Publishing

- The site deploys from `main` through `.github/workflows/hugo.yaml`.
- The usual workflow is direct commit and push to `main`, without feature
  branches or pull requests.
- After changing image paths, check path casing exactly. macOS is usually
  case-insensitive, but GitHub Pages runs on Linux and will 404 when a referenced
  path differs only by case.

## Draft Material Library

The `.drafts/` directory is a private material library. Hugo does not publish it.
Each file is a material card.

When the user gives a card title directly, treat it as a request to find and read
the matching `.drafts/` file. File names may replace `/` with `-`, while the H1
inside the card may keep the original wording.

Known material cards and status:

- `一致性哈希：加节点为什么只迁一小部分`:
  `.drafts/素材-一致性哈希-加节点只迁一小部分.md`.
- `ES 的 _split / _shrink 为什么巧妙`:
  `.drafts/素材-ES的split和shrink为什么巧妙.md`.
- `ES 查询性能优化实战`:
  `.drafts/素材-ES查询性能优化实战.md`.

## Material-To-Post Workflow

When promoting a `.drafts/` material card into a post:

- Before drafting the full article, first recommend title options, the overall
  article structure, and the intended highlight or hook of the piece.
- After drafting the article, run the `humanize-writing` skill and revise the
  draft to remove AI-like phrasing before treating it as done.
- After the user says the article is OK to commit or publish, clean up the
  corresponding material card from `.drafts/` as part of the same workflow.

Some earlier cards have already been promoted to published posts and removed from
`.drafts/`, including:

- `一份数据怎么被切碎又拼回来`:
  `content/posts/分布式/一份数据怎么被切碎又拼回来/`.
- `实时性 / 海量 / 精准，三选二`:
  `content/posts/分布式/三个都想要只能要两个/`.
- `ES pinned query：原生搜索置顶`:
  `content/posts/分布式/置顶这件事放应用层还是放es/`.

## Draw.io Diagrams

The user likes draw.io sketch-style diagrams for this blog.

Use this style when the user says `手绘风` or `用 RocketMQ 那张图的风格`.
Otherwise, use a clean straight-line style by default.

Node style:

```text
rounded=1;sketch=1;jiggle=2;curveFitting=1;fillStyle=solid;fillColor=#XXXXXX;strokeColor=#000000;fontColor=#000000
```

Important details:

- `fillStyle=solid` is required. The desired look is solid color fill with a
  hand-drawn border, not hatched pencil fill.
- Use black borders: `strokeColor=#000000`.
- Use rounded nodes: `rounded=1`.

Edge style:

```text
sketch=1;jiggle=2;curveFitting=1;endArrow=classic;strokeColor=#000000;strokeWidth=2.5
```

Use dashed secondary or discovery links with:

```text
dashed=1;endArrow=open;strokeColor=#444444
```

Role colors:

- Producer / sender: `#F4A7B9`.
- Server / container: `#9DC3F7`.
- Inner server node: `#BBD9FB`.
- Registry / center: `#97D077`.
- Consumer / receiver: `#FFD45E`.

This machine has draw.io Desktop installed at:

```text
/Applications/draw.io.app/Contents/MacOS/draw.io
```

For SVG export with Chinese text, use `--embed-svg-fonts false`; otherwise the
SVG can become much larger due to embedded Chinese font subsets.

Example export:

```bash
/Applications/draw.io.app/Contents/MacOS/draw.io -x -f svg --embed-svg-fonts false --svg-theme light -o out.svg in.drawio --no-sandbox
```

---
> Source: [zzzzzhangyanqiu/zzzzzhangyanqiu.github.io](https://github.com/zzzzzhangyanqiu/zzzzzhangyanqiu.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
