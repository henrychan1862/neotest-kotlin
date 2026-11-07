# Copilot instructions for neotest-kotlin

## Project overview

This repository is a Neovim `neotest` adapter for Kotlin/Kotest projects. The plugin is implemented in Lua under `lua/neotest-kotlin/` and is designed to discover Kotlin test positions, run the matching Gradle tests, and translate Gradle output back into Neotest results.

The main architecture is:

- `lua/neotest-kotlin/init.lua`: adapter entry point used by Neotest (`root()`, `filter_dir()`, `is_test_file()`, `discover_positions()`, `build_spec()`, `results()`).
- `lua/neotest-kotlin/treesitter/`: Tree-sitter-based detection for Kotlin package names, classes, and Kotest namespaces/tests. This is the canonical source for test discovery and IDs.
- `lua/neotest-kotlin/command.lua`: builds the Gradle command used to run a filtered set of tests, including the custom init script and `kotest_filter_*` environment variables.
- `lua/neotest-kotlin/output_parser.lua`: scans Gradle output and converts test lines like `ClassName > namespace > test FAILED` into Neotest-friendly result objects and errors.
- `tests/example_project/`: a sample Gradle/Kotlin project used by the test suite to validate detection and parsing behavior.
- `test-logging.init.gradle.kts`: custom Gradle init script that adds the structured test output needed by the adapter.

The plugin intentionally assumes a Gradle project rooted by a local `gradlew` file; it is not a general Kotlin test runner for Maven or arbitrary directories.

## Build, test, and lint commands

Use the project commands defined in `Makefile` and CI workflow:

- Full test suite: `make test`
- Formatting check: `make check`
- Format/lint rewrite: `make format` (requires StyLua)

For a single test file/spec, run the Plenary test runner directly with Neovim headless mode:

- Single spec file:
  `nvim --headless --noplugin -u tests/bootstrap_init.lua -c "PlenaryBustedFile tests/output_parser_spec.lua" -c "qa"`
- Single directory or suite subset:
  `nvim --headless --noplugin -u tests/bootstrap_init.lua -c "PlenaryBustedDirectory tests/treesitter { minimal_init = './tests/minimal_init.lua', timeout = 180000 }" -c "qa"`

CI runs the equivalent setup in `.github/workflows/test.yml`: Neovim nightly, Java 21, and `make test`.

## Key conventions and patterns

- Keep the project root detection tied to `gradlew`; this repo expects a Gradle project, not a plain source tree.
- `lua/neotest-kotlin/filter.lua` explicitly ignores generated and non-test directories such as `build`, `out`, `.gradle`, `main`, `target`, and similar folders to avoid scanning irrelevant code.
- Test discovery is Tree-sitter-driven. If new Kotlin/Kotest syntax needs support, add a matching example under `tests/example_project/` and update the relevant Tree-sitter tests instead of hard-coding special cases.
- The repo follows the style configuration in `stylua.toml`: LuaJIT syntax, 2-space indentation, double quotes, Unix newlines, and width 80.
- Gradle output is parsed semantically; the adapter does not treat raw Gradle logs as the final truth. It turns Gradle lines such as `ClassName > namespace > test FAILED` into Neotest result IDs and error details.
- The custom init script (`test-logging.init.gradle.kts`) is required for structured test logging; do not assume plain Gradle output is enough.
- When updating test detection or parsing, prefer adding/adjusting tests in `tests/` over ad hoc validation in the editor; the repo is organized around Plenary+Busted specs.

## Relevant project docs

- `README.md`: overview and plugin installation example.
- `.github/CONTRIBUTING.md`: commands for tests and formatter, plus guidance for adding new Treesitter detection patterns.
