# JJ automation scaffolding

This ExecPlan is a living document. The sections Progress, Surprises & Discoveries, Decision Log, and Outcomes & Retrospective must be kept up to date as work proceeds.


## Purpose / Big Picture

Provide two automation helpers for the `skills/using-jj` skill so a contributor can detect whether a repository is jj-managed and can generate a structured catalog of available jj commands from the local `jj` binary. Add a terminology package for jj concepts used in the skill and validate it with the terminology-work validator. After this change, someone can run `python3 skills/using-jj/scripts/detect_vcs.py --path . --format json` and receive a machine-readable verdict (jj/git/colocated/none + roots + warnings), can run `python3 skills/using-jj/scripts/catalog_commands.py --format json` to get a JSON catalog from `jj util markdown-help` with a fallback to `jj util completion bash`, and can validate the jj terminology file with `python3 skills/terminology-work/scripts/validate_vocab.py skills/using-jj/references/jj-vocabulary.json`.


## Progress

Use a list with checkboxes to summarize granular steps. Every stopping point must be documented here, even if it requires splitting a partially completed task into two ("done" vs. "remaining"). This section must always reflect the actual current state of the work.

- [x] (2026-01-23 09:50Z) Research: read `skills/using-jj/SKILL.md` and existing script conventions in `skills/exec-plan/scripts`.
- [x] (2026-01-23 09:50Z) Design: define script responsibilities, CLI interfaces, and error handling.
- [x] (2026-01-23 09:59Z) Implemented `skills/using-jj/scripts/detect_vcs.py` and `skills/using-jj/scripts/catalog_commands.py`.
- [x] (2026-01-23 10:00Z) Updated `skills/using-jj/SKILL.md` to use scripted workflow and reference terminology package.
- [x] (2026-01-23 10:02Z) Added `skills/using-jj/references/jj-vocabulary.json` and validated it with `validate_vocab.py`.
- [x] (2026-01-23 10:02Z) Validation: ran detect_vcs and catalog_commands exercises and captured outputs.

Use timestamps to measure rates of progress.


## Surprises & Discoveries

No surprises discovered while updating this plan to match the implemented scripts and terminology package.


## Decision Log

- Decision: Implement scripts in Python 3 using only the standard library.
  Rationale: Existing skill tooling already uses Python; this avoids new dependencies.
  Date/Author: 2026-01-23 (Codex)
- Decision: Prefer `jj util markdown-help` as the command catalog source and fall back to `jj util completion bash` if markdown help fails.
  Rationale: Markdown help provides summaries and usage; bash completion provides command names when markdown is unavailable.
  Date/Author: 2026-01-23 (Codex)
- Decision: Store jj terminology in a dedicated vocabulary file and validate it with `validate_vocab.py`.
  Rationale: Keeps definitions centralized, consistent, and machine-checkable without bloating the skill text.
  Date/Author: 2026-01-23 (Codex)


## Outcomes & Retrospective

The automation scripts and terminology package are in place and validated. `detect_vcs.py` reports colocated repo roots and emits deterministic JSON, `catalog_commands.py` produces a catalog from `jj util markdown-help`, and `jj-vocabulary.json` validates with zero errors or warnings. The skill now references both scripts and the vocabulary file, reducing guesswork without encoding volatile syntax.


## Context and Orientation

This repository is a collection of Codex skills. The relevant skill for this work is `skills/using-jj/SKILL.md`, which documents how to operate in jj-managed repositories. The automation scripts live in `skills/using-jj/scripts/`, and the terminology package lives in `skills/using-jj/references/`.

Key paths:
- `skills/using-jj/SKILL.md`: Skill guidance describing the scripted workflow and terminology reference.
- `skills/using-jj/scripts/detect_vcs.py`: Detects VCS roots using `jj root` and filesystem markers.
- `skills/using-jj/scripts/catalog_commands.py`: Builds a command catalog using `jj util markdown-help` with a bash-completion fallback.
- `skills/using-jj/references/jj-vocabulary.json`: Terminology package for jj concepts used in the skill.
- `skills/terminology-work/scripts/validate_vocab.py`: Terminology validator.

Terms used in this plan:
- VCS (version control system): the tool managing repository history, such as Jujutsu (jj) or Git.
- jj (Jujutsu): a version control system; in this repo it is detected by `jj root` or by a `.jj/` directory.
- git (Git): another VCS; detected by the presence of a `.git/` directory.
- colocated workspace: a repository where both `.jj/` and `.git/` directories exist at the same root.
- command catalog: a structured list (JSON/text) of `jj` command names and summaries parsed from local `jj` output.


## Plan of Work

Ensure the `skills/using-jj/scripts/` directory contains two scripts with explicit, deterministic outputs. `detect_vcs.py` should try `jj root` first (to find the canonical jj root when available) and fall back to walking parent directories for `.jj/` and `.git/` markers. It must emit both JSON and human-readable text, include warning strings when `jj` is missing or unavailable, and distinguish colocated repositories.

`catalog_commands.py` should prefer `jj util markdown-help` for richer metadata (command summaries and usage snippets). If markdown help fails or yields no commands, it should fall back to `jj util completion bash` to at least gather command names. Output must be deterministic in either JSON or text format, and errors must be explicit and include what was tried.

Add a terminology package for jj concepts used in the skill at `skills/using-jj/references/jj-vocabulary.json` and validate it with `skills/terminology-work/scripts/validate_vocab.py`. Keep `skills/using-jj/SKILL.md` aligned with the scripted workflow and terminology reference.


## Concrete Steps

Work from the repository root (`/Users/ethanwickstrom/Developer/work/skills`).

1) Verify `skills/using-jj/scripts/detect_vcs.py` exposes:
- CLI flags `--path` (default `.`) and `--format` (`text` or `json`).
- A `detect_vcs(base_dir: Path) -> DetectionResult` function.
- Output fields in JSON: `vcs`, `path`, `root`, `jj_root`, `git_root`, `jj_root_source`, `git_root_source`, `warnings`.

2) Verify `skills/using-jj/scripts/catalog_commands.py` exposes:
- CLI flag `--format` (`text` or `json`).
- Functions `parse_markdown_help`, `parse_bash_completion`, and `build_catalog`.
- A preference for `jj util markdown-help` with fallback to `jj util completion bash`.

3) Add `skills/using-jj/references/jj-vocabulary.json` and validate it with:

    python3 skills/terminology-work/scripts/validate_vocab.py skills/using-jj/references/jj-vocabulary.json

4) Ensure `skills/using-jj/SKILL.md` references the scripted workflow and the terminology package.


## Validation and Acceptance

Run validations from the repository root.

1) Detect VCS state for this repo (it contains `.jj/` and `.git/`):

    python3 skills/using-jj/scripts/detect_vcs.py --path . --format json

Expected JSON fields include:
- "vcs" is "colocated"
- "jj_root" and "git_root" both point at the repository root
- "jj_root_source" is "jj root" if `jj` is installed, otherwise "walk"

2) Catalog commands (requires `jj` on PATH):

    python3 skills/using-jj/scripts/catalog_commands.py --format json

Expected JSON fields include:
- "source" is "markdown-help" if `jj util markdown-help` succeeds, otherwise "completion:bash"
- "command_count" is greater than 0
- "commands" contains objects with at least "name" set; "summary" and "usage" may be null when using the completion fallback

3) Validate terminology package:

    python3 skills/terminology-work/scripts/validate_vocab.py skills/using-jj/references/jj-vocabulary.json

Expected: Errors: 0, Warnings: 0

Acceptance is met when the scripts produce the outputs described above, `jj-vocabulary.json` validates cleanly, and `skills/using-jj/SKILL.md` references the scripted workflow and terminology file.


## Idempotence and Recovery

The scripts are read-only: they do not mutate repositories or the environment. Running them repeatedly is safe. If validation fails, re-run individual scripts after fixing parsing or data issues. No rollback steps are required beyond reverting the modified files.


## Artifacts and Notes

Example JSON output for a colocated repo:

    {
      "vcs": "colocated",
      "path": "/Users/ethanwickstrom/Developer/work/skills",
      "root": "/Users/ethanwickstrom/Developer/work/skills",
      "jj_root": "/Users/ethanwickstrom/Developer/work/skills",
      "git_root": "/Users/ethanwickstrom/Developer/work/skills",
      "jj_root_source": "jj root",
      "git_root_source": "walk",
      "warnings": []
    }

Example JSON head for catalog commands:

    {
      "source": "markdown-help",
      "generated_at": "2026-01-23T10:01:00.802483Z",
      "command_count": 111,
      "commands": [
        {
          "name": "jj",
          "summary": "Jujutsu (An experimental VCS)",
          "usage": "jj [OPTIONS] [COMMAND]"
        }
      ]
    }

Example validation output:

    Validating: skills/using-jj/references/jj-vocabulary.json
    Errors: 0, Warnings: 0


## Interfaces and Dependencies

Dependencies: Python 3 (standard library only). Use `argparse`, `dataclasses`, `pathlib`, `json`, `subprocess`, and `re`.

`skills/using-jj/scripts/detect_vcs.py` interfaces:
- `detect_vcs(base_dir: Path) -> DetectionResult`
- `DetectionResult` fields: `kind` (`jj|git|colocated|none`), `path: Path`, `root: Path | None`, `jj_root: Path | None`, `git_root: Path | None`, `jj_root_source: str | None`, `git_root_source: str | None`, `warnings: tuple[str, ...]`
- CLI flags: `--path`, `--format text|json`

`skills/using-jj/scripts/catalog_commands.py` interfaces:
- `parse_markdown_help(text: str) -> list[CommandEntry]`
- `parse_bash_completion(text: str) -> set[str]`
- `build_catalog() -> CatalogResult`
- `CommandEntry` fields: `name: str`, `summary: str | None`, `usage: str | None`
- `CatalogResult` fields: `source` (`markdown-help` or `completion:bash`), `generated_at`, `command_count`, `commands`
- CLI flags: `--format text|json`

`skills/using-jj/references/jj-vocabulary.json` schema:
- Validated by `skills/terminology-work/scripts/validate_vocab.py`
- Must include `vocab`, `axes`, `characteristics`, `concepts`, and `relations` arrays

Both scripts must print deterministic, explicit error messages with context (operation, input, constraint) and exit with non-zero status on failure.


---

## Plan Revision Notes

- (2026-01-23 09:50Z) Initial plan created
- (2026-01-23 10:05Z) Updated plan to reflect script implementations and the jj terminology package
