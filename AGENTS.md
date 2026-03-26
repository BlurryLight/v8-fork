# Repository Guidelines

## Project Structure & Module Organization
V8 source lives in `src/` (runtime, compiler, GC, builtins). Public embedder APIs are in `include/`. Tests are under `test/` with major suites such as `mjsunit/`, `cctest/`, `unittests/`, `test262/`, and `wasm-*`. Build and developer tooling is primarily in `tools/` (for example `tools/dev/gm.py`, `tools/run-tests.py`). External dependencies are in `third_party/`.

## Build, Test, and Development Commands
- `python3 tools/dev/gm.py x64.debug d8`: generate/build a debug `d8` in one step.
- `python3 tools/dev/gm.py x64.debug check`: build common test targets and run a standard check set.
- `python3 tools/run-tests.py --outdir=out.gn/x64.debug mjsunit`: run one suite against an existing build.
- `python3 tools/run-tests.py --outdir=out.gn/x64.debug cctest/test-api/*`: run focused tests.
- `gn gen out.gn/x64.debug && autoninja -C out.gn/x64.debug d8`: manual GN/Ninja flow when you need explicit control.

Run commands from the repository root.

## Coding Style & Naming Conventions
Follow repository format rules: `.editorconfig` sets UTF-8, LF, spaces, and default 2-space indentation. C++ style is enforced by `.clang-format`/`.clang-tidy` and presubmit checks. Prefer existing V8 naming in touched files (`UpperCamelCase` for types, `lower_snake_case_` for many internal fields, API names matching existing headers). Keep includes compliant with DEPS/checkdeps.

## Testing Guidelines
Put tests in the matching suite:
- API/embedder behavior: `test/cctest/`
- JavaScript semantics/regressions: `test/mjsunit/`
- Standard conformance: `test/test262/`

Name tests by behavior (for example `JitCodeEventCallbacks`). Run targeted tests first, then broader suites for risky changes.

## Commit & Pull Request Guidelines
Use short, imperative commit subjects, often with a scoped prefix seen in history (examples: `[wasm] ...`, `[turboshaft] ...`, `Version ...`). Keep subject lines specific to one change. Before upload, run relevant tests and presubmit (`PRESUBMIT.py` checks lint/format/status/deps). PR/CL descriptions should include motivation, key design choices, impacted paths, and exact test commands run.
