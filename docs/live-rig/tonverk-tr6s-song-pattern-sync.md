# Tonverk song changes and TR-6S pattern selection

Status: the Pi Python bridge implements MIDI-triggered pattern and variation selection from notes. Automatic one-pattern-per-Tonverk-song selection remains a proposal requiring live-rig testing.

The original problem report is preserved in [the 25 October source note](source-notes/tonverk-tr6s-song-pattern-sync-2026-10-25.txt). The [ALSA router note](source-notes/alsa-midi-router-2026-10-25.txt) and [E16 Lua note](source-notes/e16-lua-tr6s-routing-2026-10-25.txt) describe the earlier setup. They are historical evidence, not the current TR-6S translation path.

## Goal and observed problem

A Tonverk song can contain several pattern changes. Tonverk's global Program Change output sends on each pattern change, which is too frequent for selecting one TR-6S pattern per song. A Program Change parameter lock on a Tonverk MIDI-machine trig was tried and disturbed synchronisation. The cause has not been isolated, and DIN Program Change has not been tested.

## Current Pi implementation

The [Pi router repository](https://github.com/LeaveTheCapital/pi-midi-router) contains two separate services:

- The [class-compliant router](https://github.com/LeaveTheCapital/pi-midi-router/blob/main/router-class-compliant.sh) maintains ALSA subscriptions. Tonverk sends MIDI directly to the TR-6S and E16; E16 sends MIDI directly to the TR-6S and Tonverk. This direct Tonverk → TR-6S subscription forwards clock and transport when Tonverk sends them.
- The [Python TR-6S bridge](https://github.com/LeaveTheCapital/pi-midi-router/tree/main/tr6s_sysex) receives E16 and Tonverk control messages, converts mapped CC, NRPN, and notes into Roland DT1 SysEx, and writes to TR-6S USB MIDI 1. It taps Tonverk clock to pace SysEx but does not forward or generate clock or transport. Its [service unit](https://github.com/LeaveTheCapital/pi-midi-router/blob/main/tr6s-sysex.service) is separate from the router unit.

The E16 TR-6S Lua translator has been replaced by this Python bridge. The router's E16 Lua autoload setting should be off for this setup, so both implementations do not write the same parameters. The repository shows the intended configuration; confirm installed units, settings, and ALSA subscriptions on the live Pi.

The current [parameter map](https://github.com/LeaveTheCapital/pi-midi-router/blob/main/tr6s-parameter-map.tsv) binds channel 10 Note On 0–11 to TR-6S pattern selection and channel 10 Note On 120–127 to variations A–H. The pattern note also updates the bridge's kit target. This is a MIDI-controlled pattern selection mechanism, not yet a verified song-change detector. Tonverk's other pattern changes could still send Program Change over the direct route if its global output is enabled.

~~~mermaid
flowchart LR
    TONVERK["Tonverk<br/>clock, transport, control notes"]
    E16["Oxi E16<br/>MIDI controls"]
    ROUTER["Pi ALSA router<br/>USB subscriptions"]
    BRIDGE["Pi Python tr6s-sysex<br/>CC, NRPN, note to DT1"]
    TR6S["Roland TR-6S"]

    TONVERK --> ROUTER
    E16 --> ROUTER
    ROUTER -->|"clock and transport; direct MIDI"| TR6S
    ROUTER -->|"Tonverk and E16 controls"| BRIDGE
    TONVERK -.->|"clock tap for SysEx pacing"| BRIDGE
    BRIDGE -->|"DT1 SysEx on USB MIDI 1"| TR6S
~~~

## Proposed song cue

For one drum pattern per song, place a dedicated cue once at a Tonverk song boundary and map it to the desired TR-6S pattern in the Python bridge. First test whether the existing channel 10 pattern notes can serve as those cues. If more than twelve patterns, a different channel, or explicit song-to-pattern mapping is needed, change the bridge's parameter map or Python code in the Pi repository.

Choose cue messages with the browser visuals in mind: the current [website setlist code](https://github.com/LeaveTheCapital/TDAwebsite/blob/main/src/setlist.ts) also treats channel 10 notes 0–12 as setlist selections. A Tonverk cue reaching both destinations could select both a song and a drum pattern. Record the intended shared mapping or separate the messages before relying on it in a set.

Tonverk's ordinary pattern changes must not select TR-6S patterns. Check its global Program Change output and the TR-6S Program Change receive settings. A direct Tonverk → TR-6S ALSA subscription forwards whole MIDI events; it cannot selectively suppress Program Change. If channel separation is insufficient, filtering belongs in a Pi ALSA client, with the single clock route preserved.

## Timing tests

1. Capture installed service status, the router environment, Tonverk and TR-6S MIDI settings, and the live ALSA graph. Confirm the E16 TR-6S Lua translator is off and that there is one intended clock path.
2. While stopped, send two channel 10 pattern notes from Tonverk. Check the TR-6S display and the Python bridge log to establish numbering and whether each cue causes one change.
3. Play through ordinary Tonverk pattern changes without a song cue. Confirm the TR-6S does not change patterns and the drum clock stays aligned.
4. Send one cue at a song boundary, then try a cue one bar early. Record whether selection is immediate or queued and whether the first step stays in phase.
5. Sweep E16 controls at performance tempo and verify that Python SysEx writes do not cause flams, drift, or tempo wobble. Follow the [Pi hardware test plan](https://github.com/LeaveTheCapital/pi-midi-router/blob/main/test-plan.md).
6. Reboot, reconnect the E16 and TR-6S, and repeat a complete song transition. Capture any failure before changing routing or timing.

If live selection shifts the TR-6S phase, select patterns while stopped for the first performance version. A targeted Stop → selection → Start on a known bar boundary is a further proposal and needs an audible transition test. The TR-6S does not receive MIDI Song Position, so restart recovery must be tested explicitly.

## Acceptance criteria

- Each Tonverk song cue selects the intended TR-6S pattern once.
- Ordinary Tonverk pattern changes do not select another TR-6S pattern.
- TR-6S clock and step phase remain stable through the complete song.
- Stopping, restarting, and reconnecting recover predictably.
