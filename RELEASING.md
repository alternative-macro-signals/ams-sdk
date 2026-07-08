# Releasing ams-sdk to PyPI

Internal notes — this file is NOT part of the published package (Poetry only
ships `ams_sdk/`, `pyproject.toml`, `README.md`, `LICENSE` and the logo; root
files outside `include` are excluded from both sdist and wheel). Verify after
any packaging change with `tar -tf dist/*.tar.gz`.

## When a release is needed

- Any code change under `ams_sdk/`.
- **Any README.md change** — the README is the PyPI project page
  (`readme = "README.md"` in pyproject), and PyPI only re-renders it from a
  newly uploaded version. PyPI hard-rejects re-uploading an existing version.

## Procedure (validated 2026-07-08, spark02)

1. Bump `version` in `pyproject.toml` (patch for docs-only, e.g. 1.0.8 → 1.0.9).
   Single source of truth — there is no `__version__` in the code.
2. Commit the bump together with the content change; push.
3. On the publish box:

```bash
cd ~/programs/ams-sdk
git pull
rm -rf dist/                 # stale artifacts would be re-uploaded by dist/*
poetry build
poetry run twine upload dist/*
```

4. Verify: https://pypi.org/project/ams-sdk/ shows the new version and renders
   the updated README; `pip index versions ams-sdk` lists it.

## Gotchas hit on 2026-07-08

- **Old system twine vs new poetry-core metadata**: recent poetry-core writes
  `Metadata-Version: 2.3+`; an old twine misreports this as
  `InvalidDistribution: Metadata is missing required fields: Name, Version`.
  Fix: use an up-to-date twine.
- **PEP 668** (`externally-managed-environment`) blocks `pip install` into the
  system Python on Debian/Ubuntu. Working pattern (no system changes):
  `poetry run pip install -U twine packaging`, then
  `poetry run twine upload dist/*` — twine lives inside the project's Poetry
  venv. (pipx is the alternative.) Never use `--break-system-packages`.
- Credentials: twine prompts per `~/.pypirc` / `__token__` + API token on the
  publish box.
