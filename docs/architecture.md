# Repository responsibilities

TDA is organised as a collection of focused repositories connected by this
project-level documentation repository.

| Concern | Source of truth |
| --- | --- |
| End-to-end live-rig topology | `tda` |
| Cross-project protocols and decisions | `tda` |
| Song catalogue, song IDs, and offline browser visuals | `TDAwebsite` |
| Chataigne modules and show mappings | `tda-chataigne` |
| Oxi E16 Lua scripts | `oxi-e16-lua-scripts` |
| Raspberry Pi ALSA USB MIDI routing and Python TR-6S translation | `pi-midi-router` (local checkout `~/pimidi`) |
| Reusable issue-to-PR automation | `agent-workflows` |

When a change crosses repositories, document the system-level decision here
and make the implementation changes in the relevant component repositories.
