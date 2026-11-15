# Installation and Verification

## Requirements

- KiCad 9.0 or later with the PCB Editor open and its IPC API server enabled.
- Python plugin dependency `kicad-python==0.8.0`, installed by KiCad from `requirements.txt`.
- A saved PCB project for storing the configuration and generated report.

The planned compatibility targets are KiCad 9.0.9 and KiCad 10.0.8. This package has not yet been launched inside either desktop version in this workspace.

## Install

1. Close KiCad.
2. Create a `returnpath-continuity` folder under the KiCad IPC plugin directory for your installed version (for example, `.../9.0/plugins/returnpath-continuity`).
3. Extract the contents of `returnpath-plugin.zip` directly into that folder. Confirm `plugin.json` is at the folder root, not inside another nested folder.
4. Enable the IPC API server in KiCad Preferences under **Plugins**, then start the PCB Editor and open a saved board.
5. Allow the plugin environment to finish creating and installing its dependency. The toolbar action may take a short time to appear.
6. Run **Analyze Return Path** from the PCB Editor toolbar.

The KiCad developer documentation lists the default plugin root as `${KICAD_DOCUMENTS_HOME}/<version>/plugins`. Typical roots are `~/Documents/KiCad/<version>/plugins` on macOS, `%USERPROFILE%\\Documents\\KiCad\\<version>\\plugins` on Windows, and `~/.local/share/KiCad/<version>/plugins` on Linux. Use the version folder KiCad creates (for example, `9.0` or `10.0`); the installed plugin folder must contain `plugin.json` directly.

## Configure and Run

On the first run, the add-on creates `returnpath_config.json` beside the project and HTML, JSON, and CSV reports. The HTML and JSON reports include the board nets and copper layers exposed by KiCad; the HTML labels inventories as complete or observed-items-only. Copy the contents of `returnpath_config.example.json` into the project file, then replace the example signal and reference net/layer names with exact names from the board. A name that conflicts with a complete board inventory is reported as `NOT_EVALUATED`, not as a zero-coverage review finding. Save and run the action again.

Reports are written beside the project as `returnpath_report.html`, `returnpath_report.json`, and `returnpath_report.csv`. The add-on reads board geometry; it does not change tracks, vias, zones, or other board items. It does write the project configuration and report files.

## Automated Checks

Run from the package folder:

```text
python -m unittest discover -s tests -v
```

The tests cover polygon holes, arc approximation, route sampling, continuous plane coverage, a slot-induced gap, fail-closed cases, via transitions, wrong-net vias, integer and canonical layer names, via-size extraction, board inventory validation, unknown config keys, JSON/CSV exports, and the JSON fixture. They use API-shaped test doubles; they do not exercise KiCad's installed Python bindings.

## Manual Compatibility Matrix

Complete each row on a machine with that KiCad desktop version before describing the package as integration-tested.

| KiCad version | Plugin loads | Board data extracted | Report opens | Board unchanged |
| --- | --- | --- | --- | --- |
| 9.0.9 | Not run | Not run | Not run | Not run |
| 10.0.8 | Not run | Not run | Not run | Not run |

For each version, test one simple two-layer board, one filled reference plane with a deliberately drawn clearance/slot near a configured trace, and a signal-via transition with and without a nearby reference via spanning the mapped reference layers. Confirm that an unfilled or unsupported reference zone produces `NOT_EVALUATED`, not a clear result. Compare the PCB before and after to verify no board items changed.

If the action is missing, check KiCad's warning messages and plugin environment trace logs. The official developer documentation explains how to recreate a plugin environment and enable API tracing.

## Known Prototype Limitations

- Filled-zone shapes that the adapter cannot read are marked unsupported; the relevant coverage result is not evaluated.
- The local geometry check does not prove that a same-net copper island is electrically connected to the intended return network.
- Via proximity is a configurable geometric heuristic, not an impedance or inductance calculation.
- Native `.kicad_pcb` test-board integration and desktop compatibility remain unverified here.
