# Discord Bot Development

A Discord bot that records voice sessions, transcribes them to text, and uses an LLM to generate session summaries with timestamps and key highlights.

## About

This bot records D&D voice sessions directly from Discord and generates automated summaries:

1. DM or player starts a session in a voice channel
2. Bot joins the voice channel and records the audio
3. Pause/resume as needed during the session
4. End session triggers transcription and summarization
5. LLM generates a formatted summary with timestamps and key highlights
6. Players can review and validate highlights

---

## How It Works

### Session Flow

1. **Start**: `/dnd-recap start-session [campaign_name]` - Bot joins voice, starts recording
2. **Live**: Voice audio streams to buffer
3. **Pause/Resume**: `/dnd-recap pause-session` and `/dnd-recap resume-session` as needed
4. **End**: `/dnd-recap end-session` - Bot leaves voice, processes audio
5. **Output**: Summary posted to channel, saved to file

### Recording Modes

| Mode | Behavior |
|------|----------|
| **Idle** | Bot not in voice |
| **Recording** | Actively capturing audio |
| **Paused** | Audio buffer frozen |
| **Processing** | Transcribing and summarizing |

---

## Tech Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| **Language** | TypeScript | Type safety |
| **Bot Framework** | Discord.js v14 | Voice support, slash commands |
| **Runtime** | Node.js 18+ | |
| **Audio Processing** | ffmpeg | Convert opus to wav/mp3 |
| **Speech-to-Text** | OpenAI Whisper API | Transcription with timestamps |
| **LLM Summarization** | OpenAI GPT-4o | Summary generation |
| **Storage** | Local filesystem | Temp files, output files |

### Architecture

```
Discord Bot
├── Commands (/dnd-recap *)
├── Voice Manager
├── Services
│   ├── Recording Service
│   ├── Transcription Service (Whisper)
│   └── LLM Summarizer
└── Output (files, Discord embeds)
```

---

## Feasibility Analysis

### Complexity: Medium-High

| Challenge | Approach |
|-----------|----------|
| Discord voice capture | Discord.js voice + ffmpeg |
| Audio to text | Batch transcription at session end |
| LLM summarization | OpenAI API |
| API costs | ~$0.30-0.80 per session |

### Time Estimate

| Phase | Duration |
|-------|----------|
| Bot setup + voice connection | 2-3 days |
| Audio recording + ffmpeg | 2-3 days |
| Whisper integration | 1-2 days |
| LLM summarization | 1-2 days |
| Summary formatting + output | 1-2 days |
| Testing + polish | 2-3 days |

**Total: ~9-15 days**

---

## Commands

### Session Commands

| Command | Description |
|---------|-------------|
| `/dnd-recap start-session [campaign]` | Join voice and start recording |
| `/dnd-recap pause-session` | Pause recording |
| `/dnd-recap resume-session` | Resume recording |
| `/dnd-recap end-session` | End session, start processing |
| `/dnd-recap status` | Show current session info |

### Post-Session Commands

| Command | Description |
|---------|-------------|
| `/dnd-recap generate-summary` | Regenerate summary |
| `/dnd-recap validate` | Mark highlights as validated |

### Utility Commands

| Command | Description |
|---------|-------------|
| `/dnd-recap help` | Show available commands |
| `/dnd-recap ping` | Check bot latency |

---

## Data Flow

### Recording Phase

```
Voice Channel → Discord Voice Stream (Opus) → ffmpeg → Audio Buffer
```

### Transcription Phase

```
Audio Buffer → Temp File → ffmpeg → OpenAI Whisper → Transcript
```

### Summarization Phase

```
Transcript → Prompt → OpenAI GPT-4o → JSON Response → Format → Output
```

### LLM Prompt Structure

```
You are a D&D session transcriber. Given this transcript:

{transcript}

Generate a JSON response with:
1. "summary": 2-3 paragraph session overview
2. "highlights": array of key moments with timestamps
3. "characters": array of NPCs/PCs mentioned
4. "locations": array of places mentioned
5. "quests": array of quest developments
```

---

## Privacy & Legal

### Considerations

- Get explicit consent from all participants before recording
- Review Discord's terms regarding voice recording
- Delete audio files immediately after transcription
- Notify users when recording starts

---

## Getting Started

### Prerequisites

- Node.js 18 or higher
- ffmpeg installed
- OpenAI API key (Whisper + GPT)

### Installation

```bash
cd bot
npm install

# Install ffmpeg
# Windows: choco install ffmpeg
# Mac: brew install ffmpeg
# Linux: sudo apt install ffmpeg

cp .env.example .env
```

### Environment Variables

```env
DISCORD_TOKEN=your_bot_token
CLIENT_ID=your_client_id
OPENAI_API_KEY=sk-...
GUILD_ID=your_test_guild_id
SESSION_OUTPUT_DIR=./sessions
LOG_LEVEL=info
```

### Bot Setup

1. Go to Discord Developer Portal
2. Create application + bot
3. Enable Server Members Intent
4. Enable Voice States Intent
5. Invite with Connect, Speak permissions

---

## Output Example

### Discord Embed

```
SESSION RECAP
Campaign: Curse of Strahd
Date: January 15, 2025
Duration: 3 hours 24 minutes
Participants: 5

SESSION SUMMARY
The party arrived at the village of Barovia and met LadyWatcher
at the blood of the vite. After a tense confrontation...

KEY HIGHLIGHTS
00:23:15 - Dragonborn paladin Thoradin receives a vision
01:12:43 - Party discovers the werewolf den
02:45:30 - Critical hit - Rogue deals 34 damage
03:02:18 - Found entrance to Castle Ravenloft

NPCS ENCOUNTERED
- LadyWatcher - Vistani fortune teller
- Ismark Kolyanovich - Burgomaster's son
- Ireena Kolyana

LOCATIONS
- Village of Barovia
- Blood of the Vite
- Castle Ravenloft
```

### Saved File Format

```
SESSION TRANSCRIPT
Campaign: Curse of Strahd
Date: 2025-01-15
Duration: 3h 24m

SESSION SUMMARY
[AI-generated summary]

TIMESTAMPED TRANSCRIPT
[Full transcript with timestamps]

KEY HIGHLIGHTS
[Highlights from LLM]
```

---

## Project Structure

```
bot/
├── src/
│   ├── commands/
│   │   ├── session.ts
│   │   ├── summary.ts
│   │   └── util.ts
│   ├── events/
│   │   ├── ready.ts
│   │   ├── voiceStateUpdate.ts
│   │   └── interactionCreate.ts
│   ├── services/
│   │   ├── VoiceService.ts
│   │   ├── TranscriptionService.ts
│   │   ├── SummarizerService.ts
│   │   └── SummaryFormatter.ts
│   ├── utils/
│   │   ├── logger.ts
│   │   ├── fileManager.ts
│   │   └── promptBuilder.ts
│   ├── types/
│   │   └── index.ts
│   └── index.ts
├── sessions/
├── scripts/
├── .env.example
├── package.json
└── tsconfig.json
```

---

## Known Challenges

| Challenge | Notes |
|-----------|-------|
| Discord voice packets | Opus codec requires ffmpeg |
| Large files | Sessions over 1 hour may need chunking |
| API rate limits | Implement retry logic |
| Bot disconnection | Handle unexpected voice state changes |
