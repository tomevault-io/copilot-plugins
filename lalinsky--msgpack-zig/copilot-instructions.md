## msgpack-zig

> * Update CHANGELOG.md - change [Unreleased] to [X.Y.Z] with current date

Release process:
* Update CHANGELOG.md - change [Unreleased] to [X.Y.Z] with current date
* Update version in build.zig.zon
* Update install instructions in README.md
* Commit files with message "Release vX.Y.Z"
* Tag the commit with vX.Y.Z
* Push commit and tags: `git push && git push --tags`
* Create GitHub release: `gh release create vX.Y.Z --title "vX.Y.Z" --notes "<changelog content>"`

---
> Source: [lalinsky/msgpack.zig](https://github.com/lalinsky/msgpack.zig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
