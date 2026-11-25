# foamFV — search OpenFOAM fvOptions and generate entries

English | [日本語](README.ja.md)

OpenFOAM (ESI, openfoam.com) ships about 50 fvOption classes, or about 70 selectable types once the templated ones are expanded (sources, constraints and corrections). Every time you use one, you have to dig through headers and tutorials to find the type name, the mandatory entries and how to write `selectionMode`.
**foamFV** parses the doc comments (`Description`, `Usage`, `\verbatim` examples, `\table` parameter tables) of every class registered as an `fv::option`, plus the `fvOptions` files in the tutorials. It builds a database from them so you can search, read the documentation, generate entries and write them into a case.

It is the sibling of [foamFO](../functionObjects), which does the same for function objects.

| File | Content |
|---|---|
| `foamFV` | CLI (Python ≥ 3.8, standard library only) |
| `catalog.html` | Searchable offline catalog in a single file; open it in a browser |
| `fvDatabase.json` | Bundled database (built from OpenFOAM-v2606) |
| `catalog_template.html` | Template for the catalog |
| `skill/openfoam-fvoptions/` | Claude Code skill |
| `install.sh` | Installer: links the skill and sets up PATH |

## Install

```bash
./install.sh            # link the skill into ~/.claude/skills and print the PATH line
./install.sh --bashrc   # also append the PATH line to ~/.bashrc (makes a backup first)
./install.sh --uninstall
```

For an OpenFOAM version other than v2606, source that environment and build its database. This takes a few seconds:

```bash
source /usr/lib/openfoam/openfoam2412/etc/bashrc
foamFV build            # -> ~/.local/share/foamFV/fvDatabase-openfoam2412.json
```

The database is looked up in this order: `$FOAMFV_DB`, then `~/.local/share/foamFV/fvDatabase-<version>.json`, then the bundled `fvDatabase.json`. If the bundled database does not match your environment, foamFV prints a warning. Set `FOAMFV_QUIET=1` to silence it.

## Usage

| Command | Description |
|---|---|
| `foamFV list [-g sources]` | List entries. Flags: T = used in tutorials, Z = cell selection via selectionMode, R = inter-region |
| `foamFV search heat source` | Full-text search over names, descriptions, parameter names and aliases (Japanese aliases work too) |
| `foamFV show limitTemperature --inherited` | Show the description, example, parameter table, inherited entries, notes and tutorials |
| `foamFV example explicitPorositySource -m 3` | Print real entries from the tutorials |
| `foamFV add scalarSemiImplicitSource --zone heater --fields h` | Print a ready-to-edit entry |
| `foamFV add limitTemperature -n limT --zone all --insert` | Append the entry to `constant/fvOptions`. The file is created if missing; otherwise a `.bak` backup is made first |
| `foamFV add ... --full / --system / -region solid` | `--full` adds optional entries as comments; `--system` writes `system/fvOptions`; `-region solid` writes `constant/solid/fvOptions` |
| `foamFV build [--src DIR] [-o FILE]` | Build the database |
| `foamFV html` | Regenerate `catalog.html` |

How the generated entries are built:
- Mandatory entries come from the section comments in the header example and the Reqd column of the parameter table.
- Options derived from `cellSetOption` also get `selectionMode`/`cellZone`. Inter-region options also get the mandatory entries of their base classes, such as `nbrRegion` and `nbrModel`.
- Templated types accept their real names, e.g. `vectorSemiImplicitSource`.
- Some header examples do not work as a minimal entry (SemiImplicitSource, explicitPorositySource). For these, foamFV uses a minimal entry written from the header and the tutorials. When a header has no example at all (e.g. rotorDisk), foamFV starts from a tutorial entry instead.
- With `-case`, the cellZone names and the fields in `0/` are listed as hints, and an unknown `--zone` gives a warning.

## Notes
- OpenFOAM reads `constant/fvOptions`. It reads `system/fvOptions` **only if** `constant/fvOptions` does not exist (`src/finiteVolume/cfdTools/general/fvOptions/fvOptions.C`).
- To confirm that an option is active, look for `Selecting finite volume options type ...` and `selected N cell(s)` in the solver log.
- foamFV supports ESI OpenFOAM only. It does not handle the Foundation `fvModels`/`fvConstraints` or finite-area `faOptions`.
- Tested so far: database built from v2606 (source tree) and v2412 (package); generated entries run with v2412 simpleFoam (pitzDaily with a topoSet cellZone, `vectorSemiImplicitSource` + `limitVelocity`).

## License
GPL-3.0-or-later ([LICENSE](LICENSE)). The bundled `fvDatabase.json` and `catalog.html` contain text from the OpenFOAM doc comments (GPL-3.0-or-later, © OpenCFD Ltd. and others).

OPENFOAM® is a registered trade mark of OpenCFD Limited, producer and distributor of the OpenFOAM software via www.openfoam.com. This offering is not approved or endorsed by OpenCFD Limited.
