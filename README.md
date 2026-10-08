# TDA

This repository is the project-level home for TDA: live-rig documentation,
cross-project decisions, connection diagrams, and ideas that do not yet belong
to one implementation repository.

It is intentionally not a monorepo. Code and device-specific configuration
remain in focused repositories, while this repository documents how those
parts form the complete live system.

## Live system

- [Audio and MIDI connections](docs/live-rig/audio-midi.md)
- [Repository responsibilities](docs/architecture.md)

## Component repositories

| Repository | Responsibility |
| --- | --- |
| [TDAwebsite](https://github.com/LeaveTheCapital/TDAwebsite) | Browser-based TDA website and live visuals |
| [tda-chataigne](https://github.com/LeaveTheCapital/tda-chataigne) | Chataigne modules, mappings, scripts, and show configuration |
| [oxi-e16-lua-scripts](https://github.com/LeaveTheCapital/oxi-e16-lua-scripts) | Oxi Instruments Lua scripts and device integration |

Shared repository automation is maintained separately in
[agent-workflows](https://github.com/LeaveTheCapital/agent-workflows).

## Capturing ideas

Use this repository for system-wide ideas or work whose destination is not yet
clear. Once an idea belongs to a single component, continue the implementation
in that component's repository and link the issues.

Adding the `agent-pr` label to an owner-authored issue asks the configured
coding agent to prepare a pull request.
