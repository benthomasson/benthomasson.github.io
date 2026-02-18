---
title:  "Giving Claude Eyes, Ears, and a Voice"
date:   2026-02-16 00:00:00 -0500
categories: ai iris speech-recognition
---

Claude can read code, write essays, debug distributed systems. But it can't hear you talk. It can't look around the room. It can't tell you the weather without you typing the question first.

So I gave it ears, eyes, and a voice. The project is called [Iris](https://github.com/benthomasson/iris).

## What It Is

Iris is a spoken personal assistant that runs on macOS. You talk. It listens via OpenAI Whisper running locally. It thinks via the Claude CLI. It speaks back via macOS text-to-speech. It can see through your webcam. And it can do things--check the weather, set timers, take notes, send iMessages--by calling local functions.

The data flow is simple:

```
Microphone → Whisper → Claude → Function execution → TTS
```

No cloud speech APIs. No wake word service. No app. Just a Python process with a microphone and a connection to Claude.

## Why Build This?

I kept running into the same friction. I'd be walking around, thinking about a problem, and want to talk to Claude. But that meant pulling out my phone, opening an app, typing. The thought was gone by the time I finished the first sentence.

Voice removes that friction entirely. AirPods in, say what you're thinking, get a response in your ears. Hands free. Eyes free. The conversation happens at the speed of thought instead of the speed of typing.

## The Architecture

Iris is ~1200 lines of Python across six modules:

**computer.py** — The main loop and state machine. Listens for audio, sends transcriptions to Claude, executes function calls, speaks responses. Manages transitions between active, sleeping, muted, passive, and dictation modes.

**llm.py** — Wraps the Claude CLI. Starts a conversation with `claude -p`, continues it with `claude -c -p`. Parses Claude's responses to separate spoken text from JSON function calls.

**functions.py** — A registry of 30+ local functions Claude can call. Weather, timers, notes, Wikipedia, camera capture, iMessage, unit conversion. Each function is registered with a decorator:

```python
@register("get_weather", "Get current weather for a location",
          {"location": "City name or zip code"})
def get_weather(location):
    # Geocode via Open-Meteo, fetch forecast
    ...
```

Claude calls functions by embedding JSON in its response:

```
The weather in Raleigh is 45 degrees and partly cloudy.
{"function": "get_weather", "args": {"location": "Raleigh, NC"}}
```

The JSON gets stripped out and executed locally. The spoken text gets read aloud. If a function returns data, Claude gets a follow-up message with the result and responds conversationally.

**voice.py** — Wraps the macOS `say` command. Configurable voice, rate, pitch.

**ui.py** — A full-screen Textual TUI that displays what you said and what Claude said. Dims when sleeping, turns red when muted.

**dictation.py** — Standalone transcription tool. Useful outside of Iris for just capturing speech to text.

## Modes

The interesting part is the state machine. Iris has several orthogonal modes that compose:

**Active/Sleep** — Iris auto-sleeps after about two minutes of silence. Say its name to wake it up. Wake word detection uses Damerau-Levenshtein edit distance--it'll hear you even if Whisper slightly garbles the name.

**Passive mode** — Iris listens to everything but stays quiet. All speech gets buffered locally. When you say its name, it sends the entire buffer to Claude as context and responds. Think of it as an always-on meeting listener that only chimes in when asked.

**Dictation mode** — Continuous transcription to a timestamped file. Crash-safe, flushed per line. Say the assistant's name to ask Claude about what you've been saying. Useful for capturing ideas on walks and querying them later.

**Visual mode** — Captures a webcam frame every 10 seconds and sends it to Claude for narration. Claude describes what it sees. This is the "eyes" part--it can read text on your screen, identify objects, comment on what's happening in the room.

**Message mode** — Polls iMessage for incoming texts from specified contacts. Each contact gets a separate Claude conversation so contexts don't bleed. The same function calling works over text--someone can message you asking about the weather and Iris responds via iMessage.

**Muted** — Microphone off, but visual captures continue. The TUI background turns red so you know at a glance.

These compose naturally. You can be in visual mode and muted (camera narrates but mic is off). You can be in dictation mode and say the wake word to query your transcript. Passive mode with visual mode means it watches and listens but only speaks when addressed.

## The Name

Iris is named for the Greek goddess who was the messenger between gods and mortals--a bridge between worlds. And for the iris of the eye, the part that lets light in. Both felt right for a project about giving AI the ability to perceive and communicate with the physical world.

## What I Learned

**Local Whisper is good enough.** I expected to need a cloud API for accuracy. Whisper at 16kHz handles conversational speech well. It occasionally garbles names and technical terms, but the fuzzy wake word matching compensates.

**Claude CLI as the brain simplifies everything.** No API keys to manage, no token counting, no streaming. Just subprocess calls. The conversation state is managed by the CLI's `-c` flag. It's not the most elegant integration, but it works and it's zero-config.

**Function calling without tool_use works.** Instead of using Claude's formal tool use protocol, Iris puts the function descriptions in the system prompt and asks Claude to emit JSON blocks. Claude does this reliably. The parsing is a regex over the response. It's unsophisticated and it works.

**Modes matter more than features.** The individual functions (weather, timers, notes) are nice but not the point. The modes--passive, dictation, visual--are what make Iris actually useful. Dictation mode during a walk, passive mode during a meeting, visual mode while working at a desk. The same assistant adapts to the context.

## What's Next

A robotic pan/tilt mount so Iris can look around the room instead of staring at a fixed angle. The servos arrived. Still waiting on the controller board.

Multi-speaker diarization so Iris can tell who's talking in a room. WhisperX with pyannote can do this, but it needs enrollment to learn voices.

And eventually, integration with [FTL2](https://github.com/benthomasson/faster-than-light2) so Iris can not just think and speak, but reach out and actually do things on remote infrastructure. Eyes, ears, voice, and hands.

## Try It

```bash
pip install git+https://github.com/benthomasson/iris
iris
```

Requires macOS, a microphone, and the Claude CLI installed and authenticated. Say "Iris" to wake it up. Say "go to sleep" when you're done.
