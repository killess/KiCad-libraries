# KiCad-libraries Setup Document

**Repo:** https://github.com/killess/KiCad-libraries
**Local clone path:** `F:\GitHub\KiCad-libraries`
**KiCad version:** 10.0
**Purpose:** Personal library of KiCad symbols, footprints, and 3D models (Greg Dennis Hughes), used across projects including the STM32H743VIT6 paddleboat controller board.

---

## 1. Repo structure

```
KiCad-libraries/
├── README.md
├── .gitattributes
├── .gitignore
├── symbols/
│   └── hughes_power.kicad_sym
├── footprints/
│   └── hughes_power.pretty/
│       └── BQ34Z100-G1_TSSOP14.kicad_mod
├── 3dmodels/          (not currently used — see note in Section 5)
├── templates/
└── docs/
```

## 2. Naming convention

**Prefix:** `hughes_` on every library nickname and filename.

- Avoids collision with KiCad's built-in libraries (e.g. stock `Power_Management` vs. your `hughes_power`)
- Based on last name (Hughes) rather than initials or an old gaming handle — chosen for it being unambiguous, professional, and not tied to a business name that might change

Category naming is short and functional, not per-part:

| Library file | Covers |
|---|---|
| `hughes_power.kicad_sym` | Power regulation, gas gauge, battery management ICs |
| *(future)* `hughes_mcu-st.kicad_sym` | ST microcontroller parts |
| *(future)* `hughes_connectors.kicad_sym` | Connectors |
| *(future)* `hughes_sensors.kicad_sym` | GPS/IMU and other sensors |
| *(future)* `hughes_misc.kicad_sym` | Everything else |

Nickname in the library table always matches the filename (e.g. nickname `hughes_power` ↔ file `hughes_power.kicad_sym`) to avoid confusion later.

## 3. Environment variable

Set in KiCad `Preferences` (Drop-Down menubar) → **Configure Paths**:

| Name | Path |
|---|---|
| `KICAD_LIBS_HUGHES` | `F:\GitHub\KiCad-libraries` |


## 4. Library table setup

**Preferences → Manage Symbol Libraries → Global Libraries:**

| Nickname | Library Path | Format |
|---|---|---|
| `hughes_power` | `${KICAD_LIBS_HUGHES}/symbols/hughes_power.kicad_sym` | KiCad |

**Preferences → Manage Footprint Libraries → Global Libraries:**

| Nickname | Library Path | Format |
|---|---|---|
| `hughes_power` | `${KICAD_LIBS_HUGHES}/footprints/hughes_power.pretty` | KiCad |

## 5. Symbol/footprint field convention

Every symbol in this library should carry these fields (hidden, not shown on schematic canvas — still export to BOM):

| Field name | Purpose |
|---|---|
| `Value` | Part number, e.g. `BQ34Z100-G1` |
| `Footprint` | e.g. `hughes_power:BQ34Z100-G1_TSSOP14` |
| `Datasheet` | Direct URL to manufacturer datasheet PDF |
| `Digi-Key` | Digi-Key part number |
| `MPN` | Manufacturer part number |

**Global Field Name Templates** (Preferences → Schematic Editor → Field Name Templates) have been set up with `Digi-Key` and `MPN` so they autocomplete when editing a symbol *instance on a schematic*.

> **Known limitation:** this global template does **not** carry into the Symbol Editor when creating a new library symbol from scratch (confirmed KiCad behavior, not a bug on our end). So new library symbols still need `Digi-Key` and `MPN` added manually via the `+` button in Symbol Properties. The reliable workaround: **duplicate an existing fully-fielded symbol** (e.g. BQ34Z100-G1) as the starting point for new parts, rather than always using New Symbol from scratch — the fields carry over automatically.

3D models: for standard/generic packages, we're currently pointing at KiCad's stock 3D model path (e.g. `${KICAD10_3DMODEL_DIR}/Package_SO.3dshapes/TSSOP-14_4.4x5mm_P0.65mm.step`) rather than duplicating the file into our own `3dmodels/` folder. The `3dmodels/` folder in this repo is reserved for custom/non-stock parts only.

## 6. Worked example: BQ34Z100-G1 (battery gas gauge)

Used as the template part for the whole workflow above. Reference details:

- **Package:** TSSOP-14 (PW), body 5.00mm × 4.40mm — *not* VQFN20 as originally assumed early in this process; corrected against the actual TI datasheet.
- **Datasheet:** https://www.ti.com/lit/ds/symlink/bq34z100-g1.pdf
- **Digi-Key PN:** 296-39949-1-ND
- **MPN:** BQ34Z100PWR-G1

**Pin table (TSSOP-14):**

| Pin # | Name | KiCad Electrical Type | Notes |
|---|---|---|---|
| 1 | P2 | Output | LED 2 or NC (tie to VSS if unused) |
| 2 | VEN | Output | Active-high voltage translation enable |
| 3 | P1 | Output | LED 1 or NC (tie to VSS if unused) |
| 4 | BAT | Input | Translated battery voltage input |
| 5 | CE | Input | Chip enable |
| 6 | REGIN | Power Input | Internal LDO input, 0.1µF decouple to VSS |
| 7 | REG25 | Power Output | 2.5V LDO output, 1µF decouple to VSS |
| 8 | VSS | Power Input | Ground |
| 9 | SRP | Input | Coulomb counter sense (nearest BAT–) |
| 10 | SRN | Input | Coulomb counter sense (nearest PACK–) |
| 11 | P6/TS | Input | Thermistor voltage sense |
| 12 | P5/HDQ | Open Collector | HDQ serial (open drain) |
| 13 | P4/SCL | Input | I2C clock (10kΩ pull-up) |
| 14 | P3/SDA | Open Collector | I2C data (open drain, 10kΩ pull-up) |

**Symbol:** `hughes_power:BQ34Z100-G1` — 1 unit, ref designator `U`, 14 pins per table above, all fields populated per Section 5.

**Footprint:** `hughes_power:BQ34Z100-G1_TSSOP14` — copied from KiCad's stock `Package_SO:TSSOP-14_4.4x5mm_P0.65mm` and renamed, rather than drawn from scratch. Pin 1 orientation checked against datasheet mechanical drawing (Section 12 of the datasheet PDF).

**Status as of this document:** symbol and footprint both created, linked, 3D model confirmed visible in Footprint Editor's 3D Viewer. **Still pending:** full round-trip test (place on test schematic → Update PCB from Schematic → confirm clean placement and correct pin 1 orientation in the actual PCB Editor 3D view, not just the standalone Footprint Editor viewer).

## 7. Workflow summary for adding a new part (repeatable process)

1. Pull the exact pinout from the manufacturer datasheet — do not assume package/pin numbering from memory or search snippets; verify from the actual datasheet PDF.
2. In Symbol Editor, **duplicate** an existing fully-fielded symbol (e.g. BQ34Z100-G1) rather than starting from New Symbol, so the field template carries over.
3. Redraw the body/pins for the new part; update Value, Footprint (can be typed before the footprint exists), Datasheet, Digi-Key, MPN.
4. In Footprint Editor, check if a suitable stock footprint exists to copy (search built-in libraries first) before drawing one from scratch.
5. Copy → paste into `hughes_<category>` library → rename appropriately.
6. Verify/add 3D model reference; preview in Footprint Editor's 3D Viewer (Properties → 3D Models tab; view via Alt+3).
7. Verify pin 1 / pad orientation against the datasheet mechanical drawing.
8. Link footprint back to symbol via the Footprint field.
9. Round-trip test: place on a test schematic → Update PCB from Schematic → confirm clean placement.
10. Commit + push in GitHub Desktop with a clear message (e.g. `Add <PART> symbol+footprint (<package>)`).

## 8. Open items / not yet done

- Rename `KICAD_LIBS_HUGHES` env var to something matching the `hughes_` convention (optional cleanup).
- Full round-trip PCB placement test for BQ34Z100-G1 not yet confirmed complete.
- `docs/symbol-guidelines.md` referenced in earlier planning but not yet written as a standalone file — this document currently serves that purpose but could be split out.
- Additional category libraries (`hughes_mcu-st`, `hughes_connectors`, `hughes_sensors`, `hughes_misc`) not yet created — only `hughes_power` exists so far.
- No Git submodule wiring into the paddleboat project repos yet (`STM32H7-paddleboat` / `PCB-Paddleboat`) — this library repo is currently standalone.
