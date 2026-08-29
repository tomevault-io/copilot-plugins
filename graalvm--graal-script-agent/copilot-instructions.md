## graal-script-agent

> - Read `README.md` for the project overview.

# Graal Script Agent Development

- Read `README.md` for the project overview.
- Treat the published documentation under `docs/` and the source code as the authority for implemented architecture and semantics.

## Public API Javadoc

- Write for public API consumers and describe stable, caller-visible contracts.
- Document every public type in exported packages, including classes, interfaces, records, enums, annotations, exceptions, and public nested types.
- Make the first sentence concise, standalone, and specific. Directly define the type's role; avoid "This class", marketing language, and restating the type name.
- Prefer one sentence when sufficient. Add sentences only when they provide useful information.
- Explain when a type should be used and how it fits with related APIs when that is not obvious.
- Document essential, verified constraints such as statefulness, mutability, thread safety, ownership, persistence, lifecycle, or security boundaries only when relevant.
- For especially important entry points or abstractions, provide additional usage guidance and a concise example.
- Keep `package-info.java` brief. Class-level Javadoc has priority.
- Avoid implementation mechanics, exhaustive member inventories, historical commentary, unsupported promises, and duplication of tutorials or the getting-started guide.
- Use links and code references where they genuinely improve navigation or precision.

## User manual

- Published documentation uses AsciiDoc rendered by the Asciidoctor Maven Plugin. API references are generated from Javadoc by the Maven Javadoc Plugin. The `docs` Maven profile assembles both into one site under `target/manual/`.
- `docs/README.adoc` is the GitHub-rendered documentation source landing page and contains local build instructions.
- `docs/index.adoc` is the canonical single-page manual assembler. It includes the files in `docs/chapters/` with `leveloffset`.
- `target/manual/index.html` is the supported manual rendering. Chapter source files need not render independently on GitHub.
- Keep two-digit chapter filename prefixes aligned with the include order in `docs/index.adoc`.
- Give every chapter root a stable explicit semantic ID. Give a subsection an explicit ID when another section links to it. Use same-document `xref:id[...]` references for intra-manual links; do not add `env-github` link variants.
- Do not duplicate chapter content in `docs/index.adoc` or `docs/README.adoc`.
- Existing Markdown drafts under `docs/` are planning inputs, not published manual sources. Do not publish, delete, or silently supersede them.
- Use AsciiDoc for published manual content. Do not introduce a Markdown-to-AsciiDoc conversion path.

Regenerate and validate the documentation from the repository root:

```bash
mvn -Pdocs -DskipTests verify
python3 docs/check-links.py target/manual
```

The build produces one manual page at `target/manual/index.html`. It must not copy source chapters, Markdown drafts, or other planning files into the generated site. Never edit or commit generated `target/` content.

For a local versioned Pages preview:

```bash
rm -rf target/pages
python3 docs/stage-pages.py --site target/manual --output target/pages --version 0.1.0 --latest
python3 docs/check-links.py target/pages/0.1.0
python3 docs/check-links.py target/pages/latest
```

Release version directories are immutable. Only `preview` or `preview-*` directories may use `--replace`. The `latest` alias follows semantic-version order and must not regress to an older release.

## Theme and API reference

- The GraalVM documentation theme is under `docs/theme/`; its Native Build Tools provenance is recorded in `docs/theme/NOTICE`.
- Shared manual styling, header, footer, fonts, and highlighting are build inputs. Preserve valid HTML head/body placement when changing `docinfo` files.
- `docs/javadoc/graal-api.css` brands generated Javadoc.
- The `docs` Maven profile generates one combined core, LangChain4j, and Spring AI reference under `target/manual/javadocs/api/`.
- Keep navigation between the manual and API reference valid. Use the build-provided `{api-root}` attribute for manual-to-API links; do not add standalone-rendering fallbacks.
- Link a public Graal Script Agent API type at its first meaningful occurrence in prose, normally where the concept is introduced rather than at an earlier incidental mention or code listing.
- Add method-level links when exact lifecycle, security, ownership, or behavioral semantics matter. Keep link labels concise and code-formatted, such as `Script<T>` or `Script.bind(Context)`.
- Do not link every repeated API mention, example-owned types, ordinary JDK types, or non-public implementation classes.
- After adding API links, regenerate the documentation and run `docs/check-links.py` so type and member targets are verified against the generated Javadoc.

## GitHub publication

- `.github/workflows/documentation.yml` builds previews for pull requests and default-branch updates.
- A matching `v<project-version>` tag publishes an immutable version directory to `gh-pages` and updates `latest` only when the version is not older.
- Manual workflow runs may publish replaceable `preview` or `preview-*` directories but must not update `latest` or overwrite release directories.
- The publication workflow preserves existing versions and retries concurrent `gh-pages` updates.
- GitHub Pages must be configured to publish the `gh-pages` branch from `/ (root)` after the repository moves to GitHub.
- Do not push documentation, create tags, change Pages settings, or perform a remote test deployment without explicit approval.

---
> Source: [graalvm/graal-script-agent](https://github.com/graalvm/graal-script-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
