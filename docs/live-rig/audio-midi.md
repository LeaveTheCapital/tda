# Audio and MIDI connections

Status: working draft derived from the notes captured on 8 October 2026.

This document is the canonical overview of audio, MIDI, clock, control, and
visual routing in the TDA live setup. The untouched input is preserved in
[the dated source notes](source-notes/audio-midi-2026-10-08.txt).

Solid arrows below represent current wiring or a stated requirement. Dotted
arrows represent proposed, optional, or unverified paths. A solid arrow can
still have an explicit verification note where the physical connection is
known but its behaviour needs testing.

## Audio routing

```mermaid
flowchart LR
    subgraph EURO1["Euro 1 · mono synth voice"]
        ATLANTIX["Intellijel Atlantix"]
        SEALEGS["Intellijel Sealegs<br/>stereo delay"]
        ATLANTIX -->|audio| SEALEGS
    end

    subgraph EURO2["Euro 2 · breakbeat sampler voice"]
        EZE_AUDIO["Ezeptocore"]
        EURO2_MIX["4-channel mixer"]
        IKARIE["Bastl Ikarie<br/>stereo filter"]
        FX_AID["Happy Nerding FX Aid<br/>stereo FX"]
        EZE_AUDIO -->|stereo| EURO2_MIX
        EURO2_MIX -->|stereo| IKARIE
        IKARIE -->|stereo| FX_AID
    end

    subgraph BESTIE["Bastl Bestie"]
        BESTIE_IN1["Input 1 · Euro 1"]
        BESTIE_IN2["Input 2 · Euro 2"]
        BESTIE_IN3["Input 3 · normalled feedback"]
        BESTIE_MIX["Stereo mix"]
        BESTIE_IN1 --> BESTIE_MIX
        BESTIE_IN2 --> BESTIE_MIX
        BESTIE_IN3 -->|"performance feedback"| BESTIE_MIX
    end

    subgraph ZOOM["Zoom LiveTrak L6"]
        L6_CH1["Channel 1 · vocals"]
        L6_CH5["Channel 5 · Bestie L/R"]
        L6_CH6["Channel 6 · Tonverk L/R"]
        L6_AUX1["Aux 1 · mono send"]
        L6_MASTER["Master mix"]
        L6_SD["SD multitrack recording"]
        L6_USB["USB-C audio"]
        L6_CH1 --> L6_MASTER
        L6_CH5 --> L6_MASTER
        L6_CH6 --> L6_MASTER
        L6_CH1 -->|"mostly dry vocal send"| L6_AUX1
        L6_CH1 --> L6_SD
        L6_CH5 --> L6_SD
        L6_CH6 --> L6_SD
    end

    SM7B["Shure SM7B"] -->|"XLR · mono"| L6_CH1
    SEALEGS -->|"stereo · minijack"| BESTIE_IN1
    FX_AID -->|"stereo · minijack"| BESTIE_IN2
    BESTIE_MIX -->|"stereo TRS"| L6_CH5
    TONVERK_AUDIO["Tonverk<br/>stereo master"] -->|"L/R"| L6_CH6
    L6_AUX1 -->|"mono · vocal FX send"| TONVERK_INPUT["Tonverk<br/>routable input"]
    TONVERK_INPUT --> TONVERK_AUDIO
    L6_USB -.->|"possible audio input"| PI_AUDIO["Raspberry Pi<br/>audio processing / speech-to-text"]
```

Unused Zoom L6 inputs are channel 2 (mono XLR/jack), channel 3 (stereo), and
channel 4 (stereo). The Bestie input 3 feedback path is musically useful but
changes the overall gain, so a repeatable way to control its level remains an
open performance-design problem.

## MIDI, clock, control, and visuals

```mermaid
flowchart LR
    subgraph CONTROLLERS["Performance controllers"]
        KEYBOARD["MIDI keyboard"]
        MONOLIT["Lightreft Monolit<br/>TRS-A MIDI / USB host"]
        E16["Oxi E16<br/>TRS-A MIDI / USB / Lua"]
        GAMEPAD["Gamepad / Wii controller"]
    end

    TONVERK["Tonverk<br/>brain + master clock"]

    subgraph EURO_CONTROL["Eurorack control"]
        UMIDI["Intellijel uMidi<br/>Euro 1"]
        EURO1_VOICE["Euro 1<br/>Atlantix voice"]
        SEALEGS_CLOCK["Sealegs<br/>syncable LFO"]
        KENTON["Kenton Pro Solo MkII"]
        EZE_CONTROL["Euro 2<br/>Ezeptocore"]

        UMIDI -->|"pitch / gate · MIDI Ch 1"| EURO1_VOICE
        UMIDI -->|"CC + mod CC1 to CV"| EURO1_VOICE
        UMIDI -->|clock| SEALEGS_CLOCK
        UMIDI -->|"reset from transport start · verify"| EZE_CONTROL
        KENTON -->|"Aux 1 clock CV · PPQN configurable"| EZE_CONTROL
    end

    subgraph PI["Raspberry Pi"]
        PI_MIDI["USB MIDI input"]
        CHATAIGNE["Chataigne"]
        VISUALS["HTML / JavaScript visuals"]
        LIGHTING["Lighting / LED control"]
        HDMI["HDMI screen / projector"]

        PI_MIDI --> CHATAIGNE
        PI_MIDI --> VISUALS
        CHATAIGNE -.->|"MIDI / lighting messages"| LIGHTING
        CHATAIGNE -.->|"control bridge · protocol TBD"| VISUALS
        VISUALS --> HDMI
    end

    KNOT["Intech Studio Knot<br/>USB MIDI host · TRS-A I/O"]
    L6_MIDI["Zoom L6<br/>TRS-A MIDI + USB MIDI"]

    KEYBOARD -->|"play Tonverk voices"| TONVERK
    MONOLIT -->|"MIDI CC · physical route TBD"| TONVERK
    E16 -->|"MIDI CC / SysEx · physical route TBD"| TONVERK
    TONVERK -->|"MIDI notes / CC / clock / transport"| UMIDI
    TONVERK -->|"DIN MIDI · split/routing TBD"| KENTON

    MONOLIT -.->|"optional USB MIDI"| KNOT
    E16 -.->|"optional USB MIDI"| KNOT
    GAMEPAD -.->|"optional USB host input"| KNOT
    KNOT -.->|"TRS-A MIDI route TBD"| TONVERK

    MONOLIT -.->|"USB MIDI? · verify"| PI_MIDI
    E16 -.->|"USB MIDI"| PI_MIDI
    L6_MIDI -.->|"USB MIDI route · verify"| PI_MIDI
    GAMEPAD -.->|"keyboard / mouse / gamepad events"| CHATAIGNE
```

## Device roles and current assignments

| Device | Current or intended role |
| --- | --- |
| Tonverk | Central musical brain, sequencer, and master clock; receives keyboard notes and controller CC; provides stereo master audio |
| Lightreft Monolit | Performance MIDI controller; intended to control Tonverk and potentially host a gamepad |
| Oxi E16 | Performance MIDI controller and Lua-capable translator for CC and SysEx |
| Kenton Pro Solo MkII | Converts MIDI clock to stable PPQN clock CV for Euro 2; spare aux outputs may later provide standardised CC-to-CV modulation |
| Intellijel uMidi | Converts Tonverk MIDI to pitch, gate, clock, modulation, and reset signals for Euro 1 and Euro 2 |
| Euro 1 | Atlantix mono voice through Sealegs stereo delay into Bestie input 1 |
| Euro 2 | Ezeptocore through mixer, Ikarie, and FX Aid into Bestie input 2 |
| Bastl Bestie | Performance submixer for both Eurorack voices and its normalled feedback effect |
| Zoom L6 | Main mixer, vocal input, stereo returns from Bestie and Tonverk, optional multitrack recorder, and possible Pi audio/MIDI interface |
| Raspberry Pi | Runs Chataigne and browser visuals; outputs HDMI; potential audio analysis, speech-to-text, and lighting control host |
| Intech Studio Knot | Optional USB MIDI host and TRS-A routing utility |

## Decisions and verification still needed

1. Define the physical MIDI merge/split path that lets the keyboard, Monolit,
   and E16 reach Tonverk while Tonverk drives both uMidi and the Kenton.
2. Identify the installed Intellijel uMidi hardware/firmware version and the
   applicable updater or configuration application.
3. Verify that Tonverk's MIDI transport-start message reliably produces the
   intended reset CV from uMidi to Ezeptocore.
4. Record the Kenton clock division/PPQN setting and confirm whether the clock
   output described as `clock CV out` and `Aux 1` is the same configured output.
5. Confirm Zoom L6 USB audio and MIDI capabilities in the intended Raspberry Pi
   operating mode, including the available channel count.
6. Choose the Raspberry Pi's primary USB MIDI source and determine whether a
   powered hub or the Knot is required.
7. Verify whether Monolit's USB-C port carries MIDI and decide whether its USB-A
   host port or the Knot should host game controllers.
8. Decide how Chataigne communicates with the browser visuals and lighting
   system: MIDI, OSC, WebSocket, standard lighting protocols, or a combination.
9. Design a repeatable gain strategy for the Bestie feedback effect and the
   Zoom L6 vocal-to-Tonverk effects loop.
