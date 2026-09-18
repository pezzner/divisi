# Divisi

An Ableton Live 12 extension that splits a chord progression in a MIDI clip
into separate tracks — one per "voice" of the harmony — so each can carry
its own instrument. Built for orchestration: turning a sketched chord part
into individual lines for strings, winds, brass, or any divisi-style
arrangement.

## What it does

Right-click a MIDI clip and choose **Pezzner - Divisi: Split Chord to
Voices...**. A dialog opens with a live preview of how the clip will be
split; press Apply and Divisi duplicates the source track once per output
voice, strips every device (instrument/effects) from each duplicate so you
get a blank track ready for your own instrument, and overwrites each
duplicate's clip with just that voice's notes. The new tracks are named
`<source track> — <voice name>` and inserted directly after the source.

Chord detection groups notes by time overlap (not just matching start
times), so it correctly keeps a held bass note grouped with a moving upper
voice, not just block chords with identical onsets.

## Two ways to split a clip

**Divisi Presets** (the default view) — six ready-to-go configurations, no
manual setup:

- **Bass line** — the lowest note of every chord, one continuous line.
- **Lead** — the highest note of every chord, one continuous line.
- **Lead + Bass** — both of the above as two tracks.
- **Closed triad (top 3)** — the top three notes of every chord as three
  monophonic lines.
- **4-part chorale (SATB)** — Soprano/Alto/Tenor from the top three notes,
  and Bass from the chord's *true* lowest note (not just "4th from the
  top," so it stays correct even on chords with more than four notes).
- **Counter-melodies (random pair)** — two independent freely-voiced lines
  drawn from the same chords, for quick countermelody ideas. Has a
  Seed/Shuffle control since it's the one preset with a random part.

**Custom** — manual control over two things:

- *Track mode*: **All voices** (one track per rank in the densest chord,
  with an adjustable cap — if a chord has more notes than the cap, the
  overflow stacks onto the last track rather than spawning more tracks or
  getting dropped) or **Extract specific voice(s)** (monophonic — every
  track holds at most one note at a time; notes beyond the track count are
  left out entirely, the same as manually duplicating a track and deleting
  what you don't want). Extract mode also accepts a specific selection
  like "just voice 1" or "1,3" or "1-3".
- *Voicing*: **Bottom-up** / **Top-down** (fixed pitch rank per chord),
  **Voice-leading** (each track moves independently, following the
  nearest note chord-to-chord — real voice leading, not a fixed rank), or
  **Random** (seeded, with a Shuffle button — reproducible so the preview
  always matches what Apply writes).

Both modes share a **Constrain to instrument range** control: fold a
voice's notes by whole octaves (never changing pitch class) into a target
range, so an idea sketched at any octave becomes playable by a specific
instrument. Includes 19 built-in ranges (strings, woodwinds, brass, saxes,
voice parts) plus a custom low/high MIDI number option. Folding is
*smooth* — each note picks whichever in-range octave keeps it closest to
where the line just was, so it doesn't introduce artificial jumps at
octave boundaries.

## Setup

1. `npm install`
2. Create a `.env` file in this folder (not auto-created) with:
   ```
   EXTENSION_HOST_PATH=C:\ProgramData\Ableton\Live 12 Beta\Program\ExtensionHost\ExtensionHostNodeModule.node
   ```
3. In Live, enable Developer Mode for extensions.
4. `npm start` and leave that terminal running — Live's Developer Mode
   doesn't auto-launch its own Extension Host, so nothing appears in any
   context menu until this is connected. Closing the terminal or letting
   it crash makes the menu item disappear immediately; that's expected,
   not a bug.

If a freshly-connected `npm start` (clean log, no errors) still doesn't
show the menu item in Live, do a full clean restart — quit every running
`npm start`, quit Live, relaunch Live, re-confirm Developer Mode, then
start just this one extension before adding any others.

## Known limitations

- **The dialog can't be resized by the user** — this is a hard limitation
  of the SDK (`showModalDialog` takes a fixed width/height with no
  resize API). It's worked around by sizing the dialog to roughly fit the
  clip's voice count before it opens (a simple clip stays compact, a dense
  one opens taller), with the voice-lane list scrolling internally if it's
  still too tall to fit.
- **Undo grouping**: track creation and note-writing each run inside their
  own transaction, but it isn't fully confirmed whether a chain of
  duplicated tracks always collapses into exactly one undo step in Live —
  worst case, reversing a Divisi run takes a couple of extra Ctrl+Z
  presses rather than one.
- **Take-lane comping**: Divisi assumes a clip's track is `clip.parent`
  directly, with a one-level fallback for a `TakeLane`. This fallback path
  is untested against a real take-laned track in Live.

## Notes for anyone editing this extension

See this project's shared `findings-and-project-log.md` doc (in the
"Ableton Extensions SDK" Claude Project) for the full build history,
including why chord clustering uses time-overlap instead of a start-time
tolerance, the reasoning behind the preset/monophonic-mode split, and the
manifest-naming convention (`manifest.json`'s `"name"` is
`"Pezzner - Divisi"`; the registered menu label is deliberately just the
action — `"Split Chord to Voices..."` — since Live auto-prefixes it with
the manifest name).
