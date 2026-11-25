# Changelog

## 1.0.0 — 2026-11-25
- First release.
- `foamFV` CLI: `build`, `list`, `search`, `show`, `example`, `add` (`--insert`, `--zone`, `--fields`, `--system`, `-region`), `html`.
- Detects `addToRunTimeSelectionTable(option, ...)` and templated `makeFvOption(...)` registrations; finite-area options are excluded.
- Per-version databases in `~/.local/share/foamFV`, bundled database for OpenFOAM-v2606.
- Offline `catalog.html` with ready-to-edit entries.
- Claude Code skill `openfoam-fvoptions` and `install.sh`.
