# TDA change impact map

One job: route a proposed live-rig change to its owning source, immediate effects, and verification. This is a small system map; it cites the existing documents instead of copying device facts into cards.

## Inputs

- Reference: `../docs/architecture.md` for repository ownership.
- Reference: `../docs/live-rig/audio-midi.md` for system topology.
- Reference: `../docs/live-rig/tonverk-tr6s-song-pattern-sync.md` for current Pi pattern selection, proposed song cue flow, and its test plan.
- Evidence: `../docs/live-rig/source-notes/` for preserved observations and corrections.

Read only the source relevant to the proposed change. Use the audio/MIDI overview and its dated notes to distinguish confirmed wiring from paths that still need verification. The Python bridge implements MIDI-controlled pattern selection; automatic Tonverk song cue mapping remains a proposal.

## Process

1. Name the path or behaviour being changed and open its source in the table below.
2. Trace one hop in and one hop out through that source's diagram or test plan. Record confirmed paths, proposals, and unknowns separately.
3. Open the implementation repository named by `../docs/architecture.md`; check its current configuration before changing a cross-repository claim.
4. Update the owning document and its diagram together. Preserve the source notes, and link the implementation change from the system-level decision.

## First-order impact routes

| Change | Start here | Check next |
| --- | --- | --- |
| Audio signal, mixer channel, or feedback path | [Audio routing](../docs/live-rig/audio-midi.md#audio-routing) | Adjacent device inputs and outputs in that diagram; the [dated notes](../docs/live-rig/source-notes/audio-midi-2026-10-08.txt) for channel assignments |
| MIDI controller, USB subscription, clock, or transport route | [MIDI routing](../docs/live-rig/audio-midi.md#midi-clock-control-and-visuals) | [Pi router ownership](../docs/architecture.md) and the [ALSA note](../docs/live-rig/source-notes/alsa-midi-router-2026-10-25.txt); downstream TR-6S and Eurorack timing |
| Tonverk song cue or TR-6S pattern selection | [Song change proposal](../docs/live-rig/tonverk-tr6s-song-pattern-sync.md) | [Python TR-6S bridge](https://github.com/LeaveTheCapital/pi-midi-router/tree/main/tr6s_sysex), Pi subscriptions, and the song cue test plan |
| Visual or lighting control | [MIDI and visual routing](../docs/live-rig/audio-midi.md#midi-clock-control-and-visuals) | [Website and Chataigne ownership](../docs/architecture.md); confirm the bridge protocol before treating a dotted path as live |
| Cross-repository decision | [Repository responsibilities](../docs/architecture.md) | The component repository that owns implementation and any affected path above |

## Outputs

- A change in the owning document or component repository, with links between them when both are affected.
- An explicit unresolved question when the path cannot be verified from the sources.

## Check

A person compares the changed path and its immediate effects with the dated notes and current component configuration. Confirm that relative links resolve and any edited Mermaid diagram parses before accepting the change.
