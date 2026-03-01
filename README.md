# D&D Recap Tools

Tools for automatically generating D&D session summaries through voice recording and AI transcription.

## Overview

This repository contains two implementations of the same core functionality:

1. **Discord Bot** - A bot that runs in Discord and records voice channels
2. **Desktop App** - A standalone C++ application for recording and transcription

Both versions:
- Record voice sessions
- Transcribe audio using OpenAI Whisper
- Generate summaries with key highlights using an LLM
- Output formatted text files

---

## Choose Your Version

### [Discord Bot](DEV-BOT.md)

Run within Discord. Ideal for groups already using Discord for their sessions.

**Quick start:**
```bash
cd bot
npm install
cp .env.example .env
npm run dev
```

### [Desktop App (C++)](DEV-DESKTOP.md)

Standalone desktop application. Ideal for learning C++ and offline flexibility.

**Quick start:**
```bash
cd desktop
mkdir build && cd build
cmake ..
make
./dnd-recorder --campaign "My Campaign"
```

---

## Feature Comparison

| Feature | Discord Bot | Desktop App |
|---------|-------------|-------------|
| Platform | Discord | Windows/Mac/Linux |
| Recording | Discord voice | System audio/microphone |
| AI Transcription | OpenAI Whisper | OpenAI Whisper |
| LLM Summary | OpenAI GPT-4o | OpenAI GPT-4o |
| Learning Curve | Lower | Higher (C++) |

---

## Documentation

- [Discord Bot Development](DEV-BOT.md)
- [Desktop App Development](DEV-DESKTOP.md)
- [Test Cases](TEST-CASES.md)

---

## License

MIT License
