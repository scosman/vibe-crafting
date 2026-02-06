# Pipelines Framework

Asset-backed job pipeline framework for orchestrating complex AI agent workflows. Jobs are idempotent, outputs are atomically committed to storage, and completed work is automatically skipped on re-runs.

## Key Files

- `specs/phase_instructions.md` - How to implement a "phase" of a project. Start here.
- `specs/implementation_plan.md` — Build checklist (track progress here)
- `specs/spec_and_architecture.md` — Full specification and component design

## Project Structure

Code lives in `src/kiln_pipelines/`. Test files use `test_<name>.py` convention in the same directory as the source file.

## Commands & Tools

You have access ot a MCP server to running tools like lint, format, types, test.

We use:
 - ruff for formatting: `uvx ruff format` to fix issues
 - ruff checking: `uvx ruff check --fix` to fix issues
 - ty for typechecking: `uvx ty check` to run

This script will run all checks (lint, format, types, tests):
```bash
uv run ./checks.sh
```

Run tests only:
```bash
uv run pytest .
```

### Limitations

The root path of this project is a sub-project of a larger python project. You can't run `uv sync`, `uv add` or other commands. But the commands above all work in folder scope.
