# Repository responsibilities

TDA is organised as a collection of focused repositories connected by this
project-level documentation repository.

| Concern | Source of truth |
| --- | --- |
| End-to-end live-rig topology | `tda` |
| Cross-project protocols and decisions | `tda` |
| Browser visuals and website code | `TDAwebsite` |
| Chataigne modules and show mappings | `tda-chataigne` |
| Oxi Lua scripts for other functions | `oxi-e16-lua-scripts` |
| Raspberry Pi ALSA USB MIDI routing and Python TR-6S translation | `pi-midi-router` |
| Reusable issue-to-PR automation | `agent-workflows` |

When a change crosses repositories, document the system-level decision here
and make the implementation changes in the relevant component repositories.
