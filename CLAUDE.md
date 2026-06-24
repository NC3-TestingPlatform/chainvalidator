# chainvalidator — Project Instructions

## What This Is

DNSSEC chain-of-trust validator. CLI tool and Python library that walks the full trust chain
(IANA root trust anchor → . → TLD → SLD → leaf record) and reports SECURE / INSECURE / BOGUS.

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | Python | ≥ 3.11 |
| CLI | Typer | ≥ 0.12 |
| Terminal | Rich | ≥ 13.7 |
| DNS | dnspython (dnssec extra) | ≥ 2.6 |
| Testing | pytest + pytest-cov | ≥ 8 / ≥ 5 |

## Code Style

- PEP 8 + type annotations on all public functions
- Sphinx-style docstrings (`:param:`, `:type:`, `:returns:`, `:rtype:`)
- `snake_case` for files and functions, `PascalCase` for classes
- Plain dataclasses for models — no Rich or DNS imports in `models.py`
- `# pragma: no cover` only on CLI callbacks and `__main__` blocks

## Testing

```bash
pytest                                     # run all tests + coverage
pytest tests/test_checker.py -v            # single file
pytest --cov=chainvalidator --cov-report=term-missing
```

- Test files mirror source: `tests/test_<module>.py`
- Coverage target: 100% (project standard)
- Avoid `# pragma: no cover` on new code — write the test instead

## Build & Run

```bash
pip install -e ".[dev]"                    # editable install with dev deps

chainvalidator check example.com
chainvalidator check example.com --type MX --timeout 10 --output report.html
chainvalidator info algorithms
```

## Project Structure

```
chainvalidator/
    cli.py          ← Typer CLI; check + info sub-commands
    assessor.py     ← assess() — the only public API entry point
    checker.py      ← DNSSECChecker (chain walking, crypto)
    dns_utils.py    ← low-level DNS query helpers
    dnssec_utils.py ← DS/DNSKEY cryptographic helpers
    models.py       ← pure dataclasses: DNSSECReport, ChainLink, LeafResult, Status
    reporter.py     ← Rich rendering + file export (.txt/.svg/.html)
    constants.py    ← algorithm maps, root servers, pick_root_server()
tests/              ← pytest suite mirroring source structure
```

## Key Conventions

- **Only public API:** `assessor.assess()` — CLI and tests call this, not `DNSSECChecker` directly
- **Exit codes:** 0 SECURE, 1 INSECURE/BOGUS/ERROR, 2 network/connection error
- **Output formats:** inferred from file extension in `reporter.py:_FORMAT_BY_EXT`
- **Commit style:** conventional commits (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`)
- **Root server selection:** `constants.pick_root_server()` uses `secrets.randbelow` — no `random`

## Before Every Commit

Run these checks and update these files as needed — do not skip any step:

```bash
# 1. Verify tests pass and coverage is still 100%
pytest
```

Before pushing, update **CHANGELOG.md**: add your changes under `## [Unreleased]`
using the standard sections (`### Added`, `### Changed`, `### Fixed`, `### Removed`).
When bumping the version, move unreleased items to a new `## [x.y.z] — YYYY-MM-DD`
section and update the comparison links at the bottom of `CHANGELOG.md`.

## Version Bumping

When committing a set of changes, bump the version using semver:
- **patch** (`0.1.x`) — bug fixes, refactor, docs, test-only changes
- **minor** (`0.x.0`) — new algorithm support, new CLI flags, new features
- **major** (`x.0.0`) — breaking API changes

Two files must always be updated together:
- `pyproject.toml` → `version = "x.y.z"`
- `chainvalidator/__init__.py` → fallback `__version__ = "x.y.z"` (the `except` branch)

## GitHub Release

Every version bump **must** be followed by a GitHub release. Do not leave a version tag without a release.

**After bumping the version, committing, and pushing:**

```bash
# Tag the version commit and push
git tag vX.Y.Z
git push origin vX.Y.Z

# Create the GitHub release
gh release create vX.Y.Z \
  --title "vX.Y.Z" \
  --notes "$(cat <<'EOF'
## What's changed

<Copy the ### Added / ### Changed / ### Fixed / ### Removed blocks verbatim
from the [X.Y.Z] section in CHANGELOG.md>

## Impact

<1–3 sentences: what this means for users — what improves, what breaks,
whether the upgrade is urgent (e.g. new DNSSEC algorithm, trust-anchor
update, chain-walk fix, etc.)>

## Migration

<Only for minor/major bumps: list any CLI flags, `assess()` parameters,
or `DNSSECReport` field changes that require user action.
Omit for patch releases.>

---

**Full changelog:** https://github.com/NC3-TestingPlatform/chainvalidator/blob/master/CHANGELOG.md
EOF
)"
```

**Release body checklist:**
- [ ] Changelog entries for this version copied verbatim
- [ ] Impact note written (even one sentence is enough)
- [ ] Migration note present if CLI flags or `assess()` signature changed
- [ ] Full changelog link at the bottom

**Conventions:**
- Tag and title: `vX.Y.Z` — semver, `v`-prefixed, must match `pyproject.toml` version
- Do not mark as draft or pre-release for normal semver releases
