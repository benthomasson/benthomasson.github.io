---
title:  "AI-Assisted Hobbies"
date:   2026-02-14 00:00:00 -0500
categories: ai hobbies robotics
---

Saturday project day. Speech recognition, computer vision, game server automation, and planning a robotic camera arm. All with AI assistance.

What surprised me: AI is most helpful in the areas where I'm *not* an expert.

## The Robotic Arm

I wanted to mount a webcam on a pan/tilt mechanism so my AI assistant ([Iris](https://github.com/benthomasson/iris)) could look around the room. I know software. I don't know robotics.

Claude walked me through:
- **Servo selection** — SG90 micro servos for a lightweight webcam, MG996R if I wanted more torque
- **Pan/tilt bracket options** — 3D printed vs off-the-shelf aluminum kits
- **Control approach** — PWM via Raspberry Pi GPIO or a dedicated PCA9685 servo driver board
- **Power considerations** — separate 5V supply for servos, don't draw from the Pi
- **Degrees of freedom** — two servos for pan/tilt, could add a third for roll

I asked questions I didn't know to ask: "What's the difference between analog and digital servos?" "Will USB power be enough?" "How do I calculate the torque needed for my camera weight?"

The AI knew. I learned. The design came together in an hour instead of a weekend of research.

## Software Projects

**[Iris](https://github.com/benthomasson/iris)** — Added webcam vision to a voice assistant. Whisper for speech recognition, macOS `say` for TTS, Claude CLI for the brain. The AI helped design the state machine (active/sleeping/muted) and the function calling protocol.

**[ftl2-servercraft](https://github.com/benthomasson/ftl2-servercraft)** — A TUI dashboard for spinning up Minecraft and Terraria servers on Linode. FTL2 handles the provisioning automation. Textual provides the interface. The AI helped structure the config system and wrote the server lifecycle scripts.

## The Pattern

When I'm in my domain (Python, automation, systems), AI accelerates what I already know how to do.

When I'm outside my domain (robotics, hardware, mechanical design), AI teaches while building. I ask dumb questions. It gives smart answers. I learn the vocabulary, the tradeoffs, the gotchas.

This is the superpower: **AI makes adjacent domains accessible.**

I'm not becoming a robotics engineer. But I can build a camera arm this weekend because the knowledge gap went from "months of learning" to "an hour of conversation."

## The E-Bike Effect

It's like the first time I rode an electric bike. Pedal-assist—not an electric motorcycle. Each push of the pedal had my force doubled by the motor. I could go twice as fast, twice as far.

AI is pedal-assist for the mind. I'm still doing the thinking. The motor just makes each thought go further.

## What I Built Today

- Voice-controlled assistant with webcam vision
- Game server lifecycle automation
- A plan for a robotic camera mount

I ordered the servos and they arrived the same day. Still waiting on the servo controller.
