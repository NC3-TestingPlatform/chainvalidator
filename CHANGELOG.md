# Changelog

All notable changes to **chainvalidator** are documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Version numbers follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

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

[Unreleased]: https://github.com/NC3-TestingPlatform/chainvalidator/compare/v0.1.2...HEAD
[0.1.2]: https://github.com/NC3-TestingPlatform/chainvalidator/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/NC3-TestingPlatform/chainvalidator/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/NC3-TestingPlatform/chainvalidator/releases/tag/v0.1.0
