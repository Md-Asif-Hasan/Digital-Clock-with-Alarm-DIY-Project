# Digital Alarm Clock

Hand-built digital alarm clock project: discrete-logic counters and drivers, 6×7-segment displays (HH\:MM\:SS), a buzzer alarm, and a 555-based timebase. This repo contains the schematic scaffolds, a finished KiCad schematic (best-effort), BOMs, and PCB starter files to help you go from breadboard to a manufacturable PCB.

## What's inside

* `docs/` — design notes and the detailed workflow (`Digital_Alarm_Clock_Workflow.md`).
* `schematic/` — generated KiCad schematic files (`digital_clock_finished_schematic.kicad_sch`) and earlier scaffold versions.

## Quick start

1. Download the repo and open the schematic in KiCad (or inspect the `.kicad_sch` file in a text editor).
2. Run ERC (KiCad) and remap any placeholder symbols to your KiCad library parts (see `docs/` for tips).
3. Assign footprints, generate Gerbers, and order PCBs from your preferred fab.
4. Assemble (THT recommended), test the 1 Hz timebase, verify counters, and test alarm behavior.

## Key notes

* The provided KiCad schematic is a best-effort conversion; verify pin mappings and run ERC before committing to PCB layout.
* For higher accuracy replace the NE555 timebase with a crystal divider (CD4060 + 32.768 kHz) or use an RTC (DS3231).

## License

MIT — see `LICENSE` for details.

## Contact

Questions, issues, or feature requests — open an issue or reach out via the repo's issue tracker.
