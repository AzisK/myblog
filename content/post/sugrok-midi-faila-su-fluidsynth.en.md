---
title: Playing MIDI Files with FluidSynth
description: Playing MIDI files in the terminal using FluidSynth
date: 2026-03-09T08:00:00Z
author: Ąžuolas Krušna
tags: ["MIDI", "Music", "Terminal"]
ShowCodeCopyButtons: true
---

Hello, today we're going to play MIDI files in the terminal!

I decided to revisit my master's thesis with fresh ears and eyes. It's been over seven years! I'm now a seasoned programmer. Back then, I was creating music with recursive neural networks – echo state neural networks. I am proud of my ambition and courage from that time. However, today is not about that. Today is about playing MIDI files!

For this, we'll use the FluidSynth program.

### What is FluidSynth?

FluidSynth is a software synthesizer that reads MIDI files and converts them into sound using SoundFont (`.sf2`) sound banks. A SoundFont is a file that contains recordings of real instrument sounds (samples). When FluidSynth receives a MIDI instruction, for example, "play a C note on a piano," it takes the corresponding recorded C note sound from the SoundFont file and plays it for us.

### How to install FluidSynth?

Since I'm using macOS, I install FluidSynth using the Homebrew program. See the FluidSynth website [here](https://github.com/FluidSynth/fluidsynth/wiki/Download) for installation instructions on other operating systems.

```bash
brew install fluid-synth
```

Let's check if it was installed correctly:

```bash
fluidsynth --version
```

We should see something like this:

```bash
FluidSynth runtime version 2.5.3
```

### How to download a SoundFont sound bank?

Now we need a SoundFont sound bank file. Without it, FluidSynth won't have anything to play, as it only plays sounds but doesn't contain the actual recordings.

A popular free SoundFont is FluidR3 GM, which contains 128 standard instruments: piano, guitar, violin, trumpet, and more.

Here's how to download the SoundFont file:

```bash
mkdir -p ~/soundfonts  # Creates a "soundfonts" folder in the user's home directory
curl -L -o ~/soundfonts/FluidR3_GM.zip "https://keymusician01.s3.amazonaws.com/FluidR3_GM.zip"  # Downloads the SoundFont archive
unzip ~/soundfonts/FluidR3_GM.zip -d ~/soundfonts # Unzips the SoundFont file from the archive
```

Let's check if the file is correct:

```bash
file ~/soundfonts/FluidR3_GM.sf2
```

We should see that the file is a SoundFont/Bank format:

```bash
FluidR3_GM.sf2: RIFF (little-endian) data, SoundFont/Bank
```

### Playing

That's all the preparation we need! Now, play your MIDI file:

```bash
fluidsynth -a coreaudio -ni ~/soundfonts/FluidR3_GM.sf2 your_file.mid
```

What do these FluidSynth parameters mean?

- `-a coreaudio` --- uses the macOS audio system (audio driver)
- `-ni` --- non-interactive mode, i.e., the program plays the file and exits
- first file --- the SoundFont sound bank
- second file --- your MIDI file

If that's all you needed, congratulations! You can close this page and enjoy the music. But if you want to learn more, read on.

### Additional parameters

#### Volume

If it's too quiet or too loud, use the `-g` parameter:

```bash
fluidsynth -a coreaudio -ni -g 2.0 ~/soundfonts/FluidR3_GM.sf2 your_file.mid
```

A value of 1.0 is the default, 2.0 is twice as loud, and 0.5 is twice as quiet.

#### Interactive mode

If you want to control the playback manually, run it without `-ni`:

```bash
fluidsynth -a coreaudio ~/soundfonts/FluidR3_GM.sf2 your_file.mid
```

Then you can use the FluidSynth console. Here are the main commands:

| Command | Description |
|---------|-----------|
| `help` | Show all commands |
| `player_start` | Start playback from the beginning |
| `player_cont` | Continue playback |
| `player_stop` | Show the current position (doesn't stop!) |
| `player_seek num` | Jump `num` ticks forward/backward |
| `gain value` | Change the volume (0–5, e.g., `gain 2.0`) |
| `set synth.reverb.active False` | Turn off reverb |
| `set synth.chorus.active False` | Turn off chorus effect |
| `settings` | Show all settings |
| `quit` | Exit |

Tip: type `help` to see all commands. Volume setting: `gain 2.0` (louder than the default). The default value depends on the FluidSynth version: `1.0` (older) or `0.2` (newer 2.x+).

### How to find out which channels a MIDI file uses?

MIDI files can use different channels (0-15). I couldn't find a simple way to determine which channels are used in a MIDI file. We can use the `uvx` tool and the `mido` library. Then, with this command and Python script, we can find out which channels are being used:

```bash
uvx --from mido python3 -c "
import mido
mid = mido.MidiFile('your_file.mid')
channels = set()
for track in mid.tracks:
    for msg in track:
        if hasattr(msg, 'channel'):
            channels.add(msg.channel)
print('Used channels:', sorted(channels))
"
```

In my case, the output is:

```bash
Used channels: [3]
```

This means that the MIDI file plays only on channel 3.

It's also important to note that a MIDI file may contain channel setting (`program_change`) messages, which override the default instruments. We can check this with the following Python script:

```bash
uvx --from mido python3 -c "
import mido
mid = mido.MidiFile('your_file.mid')
for track in mid.tracks:
    for msg in track:
        if msg.type == 'program_change':
            print(f'Channel {msg.channel} -> Instrument {msg.program}')
"
```

If nothing is printed, it means the file uses the default instruments.

### Changing instruments

To see all available instruments in the sound bank:

```bash
inst 1
```

You'll see a list of 128 General MIDI instruments:

```
000-000 Yamaha Grand Piano
000-001 Bright Yamaha Grand
000-024 Nylon String Guitar
000-040 Violin
000-056 Trumpet
...
```

To see which instruments are playing on each channel:

```bash
channels -verbose
```

Example:

```
chan 0, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 1, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 2, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 3, sfont 1, bank 0, preset 40, Yamaha Grand Piano
chan 4, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 5, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 6, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 7, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 8, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 9, sfont 1, bank 128, preset 0, Standard
chan 10, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 11, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 12, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 13, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 14, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 15, sfont 1, bank 0, preset 0, Yamaha Grand Piano
```

Note: channel 9 is always for percussion (drums).

To change the instrument number for a channel:

```bash
prog <channel> <instrument_number>
```

For example, to change channel 3 to an electric piano (2):

```bash
prog 3 2
```

You can also use the `select` command:

```bash
select <channel> <sfont> <bank> <program>
```

For example, to change channel 3 to a violin:

```bash
select 3 1 0 40
```

### How to record to a WAV file?

If it sounds good, you can record it to a WAV file using the `-F` parameter:

```bash
fluidsynth -ni -F output.wav ~/soundfonts/FluidR3_GM.sf2 your_file.mid
```

Now you can listen to it in any player or convert it to MP3.

### If it doesn't work

If something doesn't work and you get errors like these:

1. `command not found: fluidsynth`

This means FluidSynth is not installed. Make sure that `fluidsynth --version` returns a version number, not an error. Try installing FluidSynth again with `brew install fluid-synth`.

2. `expected RIFF chunk id`

The SoundFont file is corrupted or is not an SF2 file. Check it with `file ~/soundfonts/FluidR3_GM.sf2`.

3. No sound

Check the volume in your system and try increasing it with `-g 2.0`.

---

That's all there is to it. Now you can play MIDI files from the terminal until your neighbors get annoyed. Enjoy the rock and roll!

---

The MIDI format dates back to 1983, when Dave Smith and Ikutaro Kakehashi created a standard that allows different musical instruments to "talk" to each other. They both received a Technical Grammy Award for this work in 2013. Interestingly, the standard has hardly changed over 40 years --- MIDI files created in the 1980s work perfectly well even today.

---

### Sources

- FluidSynth official website https://www.fluidsynth.org/
- FluidR3 GM SoundFont sound bank https://member.keymusician.com/Member/FluidR3_GM/
- FluidSynth documentation https://github.com/FluidSynth/fluidsynth/wiki
