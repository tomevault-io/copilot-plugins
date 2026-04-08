## humanscodes

> AGENTS.md — guidance for agents in this repo

AGENTS.md — guidance for agents in this repo

Tech stack: Jekyll 3.9.1 + Bundler + jekyll-pandoc. Static site with Markdown, Liquid, and SCSS.

Build/serve
- Setup: gem install bundler && bundle install
- Build: bundle exec jekyll build
- Serve (local): bundle exec jekyll serve --livereload
- Optional: Makefile provides dev helpers only: `make build`, `make serve`, `make serve-drafts`.
- Deploy: managed exclusively by CI/CD (GitHub Actions → S3 + CloudFront). Do not use a Makefile or rsync for deployment.
- CI/CD (GitHub Actions → S3 + CloudFront):
  - Workflow: `.github/workflows/build_and_deploy.yml` (name: "CI / CD"). Triggers on push to `master` and can be run manually via `workflow_dispatch` from the Actions tab.
  - Build: runs `bundle exec jekyll build` with `JEKYLL_ENV=production`, then strips `.html` extensions from non-index pages.
  - Deploy target (S3 path): `s3://${{ secrets.AWS_S3_BUCKET_NAME }}` in region `us-west-2`.
    - Sync pass 1 (files with extensions): `aws s3 sync ./_site/ s3://${{ secrets.AWS_S3_BUCKET_NAME }} --exclude "*" --include "*.*" --delete --no-progress`
    - Sync pass 2 (extensionless HTML): `aws s3 sync ./_site/ s3://${{ secrets.AWS_S3_BUCKET_NAME }} --exclude "*.*" --content-type "text/html" --delete --no-progress`
  - CDN cache busting: CloudFront invalidation with `${{ secrets.AWS_CLOUDFRONT_DISTRIBUTION_ID }}` on path `/*`.
  - Required repository secrets: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_S3_BUCKET_NAME`, `AWS_CLOUDFRONT_DISTRIBUTION_ID`. Default region is set via `AWS_DEFAULT_REGION=us-west-2`.

Linting and “tests”
- No unit tests configured. Use html-proofer as site checks (optional dependency):
  - All pages: bundle exec jekyll build && bundle exec htmlproofer ./_site --check-html
  - Single page: bundle exec htmlproofer ./_site/path/to/page/index.html (adjust path)
- Optional: markdownlint for .markdown posts; SCSS lint is not configured.

Code style
- Structure: use _layouts and _includes; avoid large inline HTML; prefer partials.
- Markdown: front matter required; wrap ~100 cols; fenced code blocks with language.
- SCSS: 2 spaces; kebab-case class names; keep partials in _sass and import from main.scss.
- Liquid: snake_case variables; prefer filters over complex logic; keep includes pure.
- Naming: posts YYYY-MM-DD-title.markdown; files/partials kebab-case; images in images/YYYY.
- Error handling: build with --trace when debugging; treat html-proofer failures as blockers; keep Gemfile versions pinned; always use bundle exec.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ezmiller)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ezmiller)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
