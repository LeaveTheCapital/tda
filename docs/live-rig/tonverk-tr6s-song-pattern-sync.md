# Song transitions

Song-change messages must occur at the end of the outgoing song, the start of
the incoming song, or both. Coordinate transitions across Tonverk, the Python
service in `pimidi`, and TDAwebsite. Tonverk runs the musical sequence and
clock, the Pi controls TR-6S pattern selection, and TDAwebsite holds the song
catalogue and IDs and runs the visuals in the offline browser. Cue format,
sender, routing, and exact timing remain open.

The E16 Lua approach was dropped because its API could not respond to MIDI
Program Change. Moving TR-6S translation to Python gives more flexibility and
fewer memory constraints, and frees E16 Lua for other uses. [Deprecated E16 Lua
source note](source-notes/e16-lua-tr6s-routing-2026-10-25.txt)

## Existing MIDI behaviour

- The [Python TR-6S bridge](https://github.com/LeaveTheCapital/pi-midi-router/tree/main/tr6s_sysex)
  currently responds to mapped CCs and notes. In its [parameter map](https://github.com/LeaveTheCapital/pi-midi-router/blob/main/tr6s-parameter-map.tsv),
  channel 10 notes 0–11 select drum patterns. Program Change handling could be
  added in Python.
- TDAwebsite selects songs by numeric ID on channel 10 Program Change and queues
  by setlist position on channel 10 notes 0–12 in Live Set mode. Reordering the
  setlist changes which song a note queues, while the Pi's drum pattern mapping
  stays fixed. [Song catalogue](https://github.com/LeaveTheCapital/TDAwebsite/blob/main/src/songs.ts)
  · [Browser MIDI handlers](https://github.com/LeaveTheCapital/TDAwebsite/blob/main/src/tda.ts)
- Tonverk's global Program Change output follows pattern changes, and an
  earlier test with a Program Change parameter lock disturbed sync. A song
  can span several Tonverk patterns. [Original test note](source-notes/tonverk-tr6s-song-pattern-sync-2026-10-25.txt)

## Rehearsal check

Run a complete transition on the rig with the browser offline. Verify that
the intended TR-6S pattern and visuals take effect at the song boundary, that
ordinary pattern changes leave the song and drum pattern in place, and that
the drums stay in phase. Repeat after a restart or reconnect.
