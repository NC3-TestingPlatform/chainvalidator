# Changelog

All notable changes to **chainvalidator** are documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Version numbers follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Fixed
- **H1** — `cli.py`: collapsed `Status.ERROR → exit 2` branches in both JSON and non-JSON
  paths to `else → exit 1`; platform convention (global CLAUDE.md) reserves exit 2 for
  transport-level `RuntimeError`, not for a validation-level error status.
- **L2** — `dns_utils.extract_rrsets`: answer section is now scanned first for RRSIG lookup;
  authority and additional sections are a fallback-only second pass, preventing an authority-
  section RRSIG from incorrectly shadowing a more specific one in the answer section.
- **M2** — `checker._handle_insecure_delegation`: corrected `:returns:` docstring to reflect
  the actual return semantics (returns `False` when no nameserver for the child zone is found).
- **M3** — `checker._get_ns_ip_for_zone` / `_get_authoritative_ns`: removed dead second
  parameter (`validated_keys` / `zone_dnskeys`) from both signatures and their call sites.
- **M4** — Tightened bare `dict` annotations throughout `checker.py` to
  `dict[str, dns.rrset.RRset | None]`; module-level constant `_MAX_CNAME_DEPTH = 8`
  replaces the in-function literal.
- **L3** — `assessor.py`: changed `if progress_cb:` guards to `if progress_cb is not None:`
  to avoid false-negative for callable objects that evaluate to falsy.
- **L4** — `constants.py`: corrected module docstring (the module also contains a stateless
  helper function, not only protocol-defined values).
- Updated `chainvalidator/CLAUDE.md` exit code table to match corrected behaviour.

### Changed
- `from typing import Callable` → `from collections.abc import Callable` in `assessor.py`
  (preferred since Python 3.9+).

### Tests
- `test_cli.py`: renamed `test_error_status_exits_2` → `test_error_status_exits_1`;
  added `TestCLICheckJson` class (5 tests covering `--json` paths for all four statuses
  plus JSON output validity).
- `test_checker.py`: added two `TestFollowCname` tests covering the CNAME cross-zone error
  paths (parent zone unsigned; target zone has no validated keys); fixed 6 `TestNsHelpers`
  calls to match the new 1-argument signatures.
- `test_dns_utils.py`: added test for RRSIG fallback from authority section in
  `extract_rrsets`.

---

## [0.1.4] — 2026-06-24

### Fixed
- **HIGH** — `_follow_cname`: replaced `shared_keys[parent]` / `shared_keys[target_zone]`
  direct lookups with `.get()` guards; a cross-TLD CNAME (e.g. `foo.com → bar.net`)
  previously raised `KeyError` because the target TLD's parent zone was never validated
  in the original chain walk.
- **MEDIUM** — `check()`: added `None` guard before calling `_check_final_rrset` when
  the target zone is unsigned (insecure delegation with no DNSKEY records); previously
  raised `TypeError: 'NoneType' object is not iterable` for "island of security" zones.
- `_load_trust_anchor`: wrapped XML parsing loop in `try/except` so a malformed
  `root-anchors.xml` produces a clean error message instead of a raw traceback.
- Improved CNAME error message: "unvalidated parent" → "parent zone is unsigned
  (insecure delegation)" for clarity.

### Changed
- All type annotations migrated from `Optional[X]` to `X | None` (PEP 604); removed
  all `from typing import Optional` imports.
- Test `_capture` helper updated to inject the console via keyword argument instead
  of patching the module-level `_console` attribute.

---

## [0.1.3] — 2026-05-15

### Added
- `pytest-mock>=3.12` added to dev extras.

### Changed
- Exit codes corrected: `INSECURE` now exits `1` (validation result, not a
  network error); `ERROR` now exits `2` (network/connection failure);
  `RuntimeError` propagated from `assess()` also exits `2`.
- `reporter`: console instance renamed to private `_console` with a public
  `console` alias; `Console` now created with `highlight=False` for
  consistent terminal output.
- `print_full_report`, `print_verdict`, `print_trust_anchor`, `print_chain`,
  and `print_leaf` now accept an optional `*, console: Console | None = None`
  parameter so callers can inject a custom console. `save_report()` always
  writes from `_console`.

---

## [0.1.2] — 2026-05-15

### Added
- `--json` flag on `check` command — prints a JSON representation of the
  `DNSSECReport` to stdout; exit codes are preserved (0 SECURE, 2 INSECURE,
  1 BOGUS/ERROR), consistent with the platform-wide `--json` convention.

---

## [0.1.1] — 2026-04-27

### Changed
- Repository moved to the
  [NC3-TestingPlatform](https://github.com/NC3-TestingPlatform) GitHub
  organisation; all internal URLs updated.

---

## [0.1.0] — 2026-03-11

### Added
- Initial release of **chainvalidator**.
- Full DNSSEC chain-of-trust validation (IANA root → TLD → target).
- Status reporting: `SECURE` / `INSECURE` / `BOGUS`.
- CLI: `chainvalidator check <domain>` with `--type`, `--timeout`,
  `--output` flags.
- `chainvalidator info algorithms` — lists supported DNSSEC algorithm OIDs.
- Report export to `.txt`, `.svg`, `.html`.
- 100% test coverage via pytest.

---

[Unreleased]: https://github.com/NC3-TestingPlatform/chainvalidator/compare/v0.1.4...HEAD
[0.1.4]: https://github.com/NC3-TestingPlatform/chainvalidator/compare/v0.1.3...v0.1.4
[0.1.3]: https://github.com/NC3-TestingPlatform/chainvalidator/compare/v0.1.2...v0.1.3
[0.1.2]: https://github.com/NC3-TestingPlatform/chainvalidator/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/NC3-TestingPlatform/chainvalidator/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/NC3-TestingPlatform/chainvalidator/releases/tag/v0.1.0
