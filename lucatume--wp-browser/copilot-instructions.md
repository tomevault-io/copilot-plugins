## wp-browser

> This project is called "wp-browser" and it's a set of Codeception modules aimed at testing WordPress plugins, themes and sites.

This project is called "wp-browser" and it's a set of Codeception modules aimed at testing WordPress plugins, themes and sites.

## Tests

The diverse nature of the code part of this project demands different approaches to testing: the code can be tested at the unit level, integration level, functional level and end-to-end level.  

Try to write **unit** tests to cover the code when possible.

Unit tests should avoid mocking and partial mocking of both the subject under test and its dependencies.
When writing or reviewing tests point out these bad practices and propose these mitigations:
* file system mocking using virtual file system libraries or other stream-based solutions -> use temporary directories and files
* mocked dependencies created using PHPUnit, Prophecy or Codeception mock factories  -> use real objects building their null version if possible (e.g., `Symfony\Component\Console\Input\StringInput` to replace the required `Symfony\Component\Console\Input\InputInterface`)
* partial mocking using reflection to set values -> set up the input and dependencies of the subject under test correctly
* getting private/protected properties from objects using reflection -> change the assertions to be on output and side-effects (e.g., modified or created files) rather than on the internal state of the subject under test

Use the `lucatume\WPBrowser\Utils\Filesystem::tmpDir()` method to set up temporary directories kand files, file: src/Utils/Filesystem.php.

## Tools

### Testing
To run tests use the `vendor/bin/codecept run` binary. 
Run a suite of tests using `vendor/bin/codecept run <suite>`, e.g., `vendor/bin/codecept run unit`.
Run a single test file or directory using its path relative to the project root directory, e.g., `vendor/bin/codecept run tests/unit/SomeTest.php` or `vendor/bin/codecept run tests/unit/SomeDirectory`

Before claiming a fix or feature is done, run the suite that covers it and show the actual output. Never state a test passes without having run it in this session. For a bugfix, reproduce the failure first, apply the fix, then re-run and confirm green.

The unit suite needs the `uopz` extension enabled; without it the tests using `src/Traits/UopzFunctions.php` error with `Call to undefined function uopz_set_return()`. Toggle extensions before running tests: `uon`/`uoff` enable/disable uopz, `xon`/`xoff` enable/disable Xdebug. Run `uon` before the unit suite.

Run a suite split across N workers with `vendor/bin/codecept parallel-run <suite> --workers=<N>`, e.g. `vendor/bin/codecept parallel-run unit --workers=10`. Workers re-exec `vendor/bin/codecept` through its `#!/usr/bin/env php` shebang, so they inherit whatever the active CLI php has loaded, not a parent `-d extension=uopz`; `uon` must be active for them too.


### Code quality tools

Once you're done with a task, or a phase of a plan, run static code analysis tools to fix code style issues:

Fix then check PHP Code Sniffer issues:
```
composer run cs-fix
composer run cs
```

Run PHPStan to check on the issues and the fix them:
```
composer run stan
```

Check for typos in the code
```
composer run typos
```

Any one of the scripts above can be provided arguments using the `-- ...` syntax of Composer, examples:
* run `cs:fix` on a directory - `composer cs:fix -- directory`
* run `cs` on a file - `composer cs-- file`
* run `stan` on a file - `composer stan -- file`
* run `typos` on a file - `composer typos -- file`
* run `typos` on a directory - `composer typos -- directory`

## Changelog
Any change that affects behavior, the public API, builds or releases must be recorded in `CHANGELOG.md` under the `## [unreleased] Unreleased` section, before you consider the task done. Use the existing `### Added/Changed/Fixed/Removed/Breaking change` subheads and reference the PR or issue, e.g. `(#804)`. Do not wait to be asked.

## When you cannot reproduce a bug
If you cannot reproduce a reported bug after a few honest attempts, stop. Report what you ruled out and two or three hypotheses for next steps instead of investigating indefinitely. A handoff beats a burned session.

## Commenting
Where possible, adding comments should be avoided. The audience of this codebase are skiled developers ang comments should be far and few only where a piece of code is really cryptic.
When writing or reviewing code, remove comments if they are not really necessary.

---
> Source: [lucatume/wp-browser](https://github.com/lucatume/wp-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
