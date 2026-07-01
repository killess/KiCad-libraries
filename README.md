# KiCad-libraries

Personal KiCad symbol, footprint, and 3D model libraries (killess).

## Structure
- `symbols/` — schematic symbols (.kicad_sym), one file per category
- `footprints/` — PCB footprints (.pretty folders), one folder per category
- `3dmodels/` — 3D models (.3dshapes folders, .step preferred)
- `templates/` — reusable sheet/project templates
- `docs/` — conventions and changelog

## Using this library

### As a submodule in a project
git submodule add https://github.com/killess/KiCad-libraries.git libs/kicad-libraries

Pin to a tagged release rather than main:
cd libs/kicad-libraries
git checkout v1.0.0

### Environment variable
Set KICAD_LIBS_KILLESS in KiCad → Preferences → Configure Paths, pointing at wherever this repo is cloned.

## Conventions
See docs/symbol-guidelines.md.