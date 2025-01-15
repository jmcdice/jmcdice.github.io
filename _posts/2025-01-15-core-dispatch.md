---
title: Core Dispatch – AI-Driven Radio Communication System
date: 2025-01-14
categories: [AI Projects, Radio]
tags: [ai, raspberry pi, openai, radio, dispatch]
description: Core Dispatch is an AI-powered radio communication system that merges handheld two-way radios, a Raspberry Pi, and GPT-based personas to transcribe, analyze, and respond on the airwaves.
---

# Core Dispatch

**Core Dispatch** is an AI-powered radio communication system that transforms classic handheld two-way radios into an intelligent, context-aware presence on the airwaves. By combining **Raspberry Pi**, **OpenAI’s GPT-based** models, and **Text-to-Speech (TTS)**, Core Dispatch can receive, transcribe, process, and respond to radio communications in real time—whether for emergency dispatch, warehouse inventory updates, or just a laid-back “The Dude” persona for fun.

<video width="600" height="338" controls>
  <source src="https://storage.googleapis.com/echo-004/dispatch-alpha-demo.mov" type="video/mp4">
  Your browser does not support the video tag.
</video>

## Overview

Core Dispatch injects AI-driven personas into any channel you choose, using **three** handheld two-way radios (one for receiving, one for transmitting, and one for the operator), plus a **Raspberry Pi**. It listens for transmissions, transcribes them (e.g., via **OpenAI Whisper**), and generates persona-specific responses (e.g., a chill “Dude,” a knowledgeable “Warehouse Worker,” or a “Dispatch” lead for emergencies). It then uses **TTS** to play those responses back over the same channel.

### Key Components

- **Receiver (RX)**: Listens for audio above a certain threshold, transcribes it with Whisper or another STT service.
- **Transmitter (TX)**: Takes AI-generated text, converts it to speech using TTS, and broadcasts via VOX.
- **Multiple Personas**: Easily swap between “Dispatch,” “Fire Analyst,” “Warehouse Worker,” or a custom persona for your scenario.

## Hardware Setup

Below is the minimal hardware list for Core Dispatch:

1. **Raspberry Pi** (4, 5, or similar)
2. **Three handheld two-way radios** (one for RX, one for TX, one for the operator to talk from)
3. **USB Sound Card** (for capturing audio from RX radio)
4. **2.5mm → 3.5mm audio cable** (RX radio speaker out to Pi mic in)
5. **3.5mm → 3.5mm cable** (Pi headphone out to TX radio mic in)
6. **VOX** on TX radio for automatic push-to-talk

Approximate cost is around $200 on Amazon, depending on deals.

### Wiring Diagram

```
[RX Radio] --(Speaker Out)--> [USB Sound Card Mic-In] --(USB)--> [Raspberry Pi]
[Raspberry Pi Headphone Out] --(3.5mm cable)--> [TX Radio Mic-In]
```

1. Set **VOX** on your transmit radio so it keys up when the Pi plays audio.
2. Check device indexes on your Pi with `sounddevice.query_devices()` and update the config accordingly.

### FCC Regulations & Frequency Use

If you’re in the United States:

Unlicensed Operation: Family Radio Service (FRS) channels typically do not require an FCC license, but they do have power/output limitations.
GMRS (General Mobile Radio Service) requires a license, but offers higher power allowances.
Amateur (Ham) Bands: You must have at least a Technician license (or higher) to transmit, and you must identify your callsign according to Part 97 rules.
Always Verify Local Rules: Regulations vary by region or country (e.g., PMR446 in Europe).

Disclaimer: This project does not provide legal advice. Use only frequencies you’re licensed (or license-free) to operate on.


## Software Setup

1. **Clone the Project**
   ```bash
   git clone https://github.com/jmcdice/core-dispatch.git
   cd core-dispatch
   ```
2. **Install Dependencies**
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
3. **Set Environment Variables**
   ```bash
   export OPENAI_API_KEY="sk-xxxx"
   export TRANSCRIPTION_SERVICE_TYPE="openai-whisper"
   export TTS_PROVIDER="openai"
   ```
4. **Launch the Receiver**
   ```bash
   python start_reciever.py
   ```
   - Continuously listens for radio traffic, transcribes speech, and saves JSON transcription files.
5. **Launch the Transmitter**
   ```bash
   python start_transmitter.py --profile emg_response
   ```
   - Monitors transcriptions and responds via **Dispatch**, **Fire Analyst**, or **Logistics** personas.

## Features

- **Real-Time Transcription**: Captures incoming radio transmissions and converts them to text (Whisper, Google Chirp, or a mock STT).
- **AI-Powered Personas**: Load any persona (or multiple personas) with custom system prompts, specialized behavior, and activation phrases.
- **Tool Integration**: GPT-4 can call external “tools” (e.g., inventory lookups) to fetch real data for responses.
- **TTS Playback**: Multiple TTS engines supported (OpenAI, UnrealSpeech). The final speech is piped through the TX radio on VOX.
- **Context-Aware Conversation**: Maintains conversation logs so each persona can provide relevant, context-specific replies.
- **Logging**: Comprehensive logs for debugging and reviewing activity.

## Example Scenarios

1. **Emergency Dispatch**
   - “Dispatch, this is Unit 33—fast-moving fire near Sunset Blvd.”
   - Fire Analyst and Logistics personas can step in with spread rates and water tanker availability.

2. **Warehouse Worker**
   - “Warehouse, do we have organic almond milk in stock?”
   - The AI persona calls an **InventoryLookupTool** and replies with item quantity and aisle number.

3. **Just Having Fun**
   - “Hey Dude, how about a round of bowling?”
   - The Dude persona responds in a chill manner, referencing White Russians and abiding by the lanes.

## Potential Use Cases

- **Incident Management**: Fire/EMS dispatch, situational awareness, quick queries and resource checks.
- **Business Applications**: Warehouse or retail inventory queries, stock notifications, or shipping updates.
- **Novelty or Entertainment**: A “DJ” persona that plays music clips or a “gamer buddy” persona for on-air banter.

## Next Steps

- **Add More Personas**: Build a “Security Bot,” “Translator Bot,” or “DJ Dave” persona.
- **Try Different TTS/STT**: Swap Whisper for Google Chirp, or OpenAI TTS for UnrealSpeech.
- **Custom Tools**: Link your database or API so the AI can answer real-time queries.
- **Hardware Upgrades**: Experiment with better microphones, external speakers, or even multiple Pi setups networked together.

## Closing Thoughts

Core Dispatch bridges the gap between **traditional radio communication** and **cutting-edge AI**. Whether you’re managing emergency scenarios, running a warehouse, or just having fun with an AI “buddy” over the air, the system’s modular design makes it easy to innovate.

Stay tuned for future updates, and **feel free to fork and contribute** on [GitHub](https://github.com/jmcdice/core-dispatch). Got ideas or questions? Let us know!

---

**Thanks for reading!** If you found this project exciting, share it with your fellow radio enthusiasts, AI buffs, or open-source friends. Happy (AI) radioing!
