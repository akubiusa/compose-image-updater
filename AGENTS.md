# AGENTS.md

Guidance for AI coding agents and contributors. `CLAUDE.md` imports this file, and `.github/copilot-instructions.md` adds the review checklist.

## Language

- All deliverables are written in **English**: commit messages, PR/Issue titles and bodies, review comments, code comments and docstrings, and documentation (`README.md`, `AGENTS.md`, etc.).
- Exceptions (keep as is): quotations, technical identifiers, generated files, and external source text.
- Runtime messages (logs, errors, Discord notification text) are English.
- Do not rewrite Git history to change the language of past commits.

## Overview

`compose-image-updater` is a small Python CLI plus shell wrappers. For a Docker Compose project it runs `docker compose pull`, compares the version of every image tagged `latest` before and after, and, when something changed, optionally restarts the project and posts the result to a Discord webhook.

- Only images whose tag in the compose file is `latest` are tracked; other tags are not updated.
- The version of an image is the `org.opencontainers.image.version` label, falling back to `Created`.
- A project is restarted (`docker compose down`, then `up --build -d`) only if its directory name is listed in `restartable_projects.json`. After a successful restart, error lines in the logs are counted in the notification (see the README).
- The only runtime dependency is `requests`. Docker with the Compose plugin must be available.

## Commands

```bash
./wrapper.sh /path/to/project                  # update one project (creates ./venv, runs pip install, then python3 -m src)
./update-all.sh /path/to/projects/ /path/to/logs/   # run wrapper.sh for every compose project found (cron use)
./generate-restartables.sh /path/to/projects/  # regenerate restartable_projects.json (requires jq)
python3 -m src /path/to/project                # run without the wrapper (needs requests installed; defaults to the current directory)
```

There are no automated tests. CI (`.github/workflows/`, reusable workflows from `book000/templates`) runs the following; run the same locally before opening a PR:

```bash
python3 -m compileall -q .
ruff check .
ruff format --check .
shellcheck --severity=warning ./*.sh
```

The Python CI also runs `pytest`, which passes when no tests are collected. Verify behavior changes manually against a real or throwaway Compose project and describe the check in the PR.

## Structure

- `src/__main__.py`: entry point. `update_images()` pulls and diffs versions, `main()` handles restart and notification, `is_restartable_project()` reads `restartable_projects.json` from the repository root.
- `src/__init__.py`: helpers that call `docker` through `subprocess` (list arguments, no shell), image version lookup, and `send_to_discord()`.
- `wrapper.sh`: changes to its own directory, loads `discord_webhook_url.txt` into `CIU_DISCORD_WEBHOOK_URL` (the file takes precedence over an existing variable), prepares `venv`, installs `requirements.txt`, and runs `python3 -m src`. It is also used from systemd `ExecStartPre`.
- `update-all.sh`: finds `docker-compose.yml`, `compose.yaml` and `compose.yml` in the given directory and its immediate subdirectories (`find -maxdepth 2`) and runs `wrapper.sh` per directory, teeing output to `<log dir>/<timestamp>.log`.
- `generate-restartables.sh`: writes the names of those directories to `restartable_projects.json` and shows a diff against the previous file.

## Conventions

- Call Docker through `subprocess` with list arguments, as the existing code does. Do not add dependencies beyond `requests` for what the standard library covers.
- Python is formatted and linted with ruff using its defaults (there is no ruff configuration file). Shell scripts must pass ShellCheck.
- When making a non-obvious design decision, state the choice, the rejected alternatives and the assumptions in the PR description.
- Update `README.md` and this file when arguments, configuration files or environment variables change.
- Commits follow [Conventional Commits](https://www.conventionalcommits.org/); branches follow [Conventional Branch](https://conventional-branch.github.io) short form (`feat/...`, `fix/...`).
- Do not commit to Renovate PRs; open a new PR instead.

## Security

- Never commit the Discord webhook URL. It is read from `CIU_DISCORD_WEBHOOK_URL` or `discord_webhook_url.txt`; the file and `restartable_projects.json` are in `.gitignore`.
- Do not print secrets in code or logs.
