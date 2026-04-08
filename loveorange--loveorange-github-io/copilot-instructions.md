## loveorange-github-io

> This is a personal blog built with [Hugo](https://gohugo.io/) and the [Stack theme](https://github.com/CaiJimmy/hugo-theme-stack). The content is written in Markdown and the site is deployed to GitHub Pages.

# Project Overview

This is a personal blog built with [Hugo](https://gohugo.io/) and the [Stack theme](https://github.com/CaiJimmy/hugo-theme-stack). The content is written in Markdown and the site is deployed to GitHub Pages.

## Building and Running

To run the blog locally for development, use the following command. This will start a local server with live reloading.

```bash
hugo server
```

To create a new post, use the `hugo new` command. This will create a new Markdown file in the `content/posts` directory with the default front matter.

```bash
hugo new posts/your-new-post.md
```

## Deployment

The blog is automatically deployed to GitHub Pages whenever changes are pushed to the `master` branch. The deployment process is defined in the `.github/workflows/deploy.yml` file.

The workflow builds the site using `hugo --minify --gc` and deploys the generated `public` directory to the `gh-pages` branch.

## Development Conventions

All blog posts are located in the `content/posts` directory. Each post is a separate Markdown file.

The front matter for each post is defined at the top of the Markdown file. The following fields are available:

*   `title`: The title of the post.
*   `slug`: The URL slug for the post.
*   `description`: A brief description of the post.
*   `date`: The publication date of the post.
*   `draft`: Whether the post is a draft. Set to `false` to publish.
*   `image`: The path to the featured image for the post.
*   `math`: Whether to enable MathJax for the post.
*   `hidden`: Whether to hide the post from the main list.
*   `categories`: A list of categories for the post.
*   `tags`: A list of tags for the post.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LoveOrange)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LoveOrange)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
