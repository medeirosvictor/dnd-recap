# Desktop App Development (C++)

A standalone desktop application that records D&D voice sessions, transcribes them to text, and uses an LLM to generate session summaries.

## About

This is a C++ desktop application for the same functionality as the Discord bot, built primarily for learning low-level programming and understanding how software interacts with system resources.

Goals:
- Learn C++ fundamentals and memory management
- Understand system-level programming (audio, files, networking)
- Build a usable tool without external platform dependencies

---

## How It Works

The application can run in two modes:

### CLI Interface

The application runs as a command-line tool:

```bash
./dnd-recorder --start "Campaign Name"   # Start recording
./dnd-recorder --pause                   # Pause recording
./dnd-recorder --resume                  # Resume recording
./dnd-recorder --stop                    # Stop and process
```

### GUI Interface (Optional)

A simple native GUI provides the same functionality with visual controls:
- Start/Pause/Resume/Stop buttons
- Campaign name input
- Live recording status and duration
- Transcript viewer

The GUI is optional and can be built separately from the CLI version.

### Session Flow

1. **Start**: `./dnd-recorder --start [campaign_name]` - Begins audio capture
2. **Live**: Audio streams to buffer
3. **Pause/Resume**: `--pause` and `--resume` as needed
4. **Stop**: `./dnd-recorder --stop` - Processes audio, generates summary
5. **Output**: Summary saved to file

---

## Why C++

| Reason | Explanation |
|--------|-------------|
| Memory management | Learn manual allocation/deallocation |
| System access | Direct control over audio, files, network |
| Bare metal potential | Can run without OS (embedded systems) |
| Performance | Zero runtime overhead |
| ML Inference | Direct integration with whisper.cpp for local transcription |
| Career skills | Systems programming, game engines |
| No API costs | Local transcription means no per-minute Whisper API fees |

---

## Tech Stack

### Core

| Layer | Technology | Notes |
|-------|------------|-------|
| **Language** | C++17 | Modern C++ with good toolchain support |
| **Build System** | CMake | Cross-platform build |
| **GUI** | Simple native GUI | Learning GUI development |
| **ML Inference** | whisper.cpp | Local transcription, no API needed |

### Libraries

| Purpose | Library | Alternative |
|---------|---------|-------------|
| Audio Recording | PortAudio | Native APIs (WASAPI, ALSA, CoreAudio) |
| Audio Encoding | ffmpeg (libav) | Command-line ffmpeg |
| Speech-to-Text | whisper.cpp | OpenAI Whisper API |
| HTTP Requests | libcurl | cpprestsdk (C++ REST) |
| JSON Parsing | nlohmann/json | cJSON, rapidjson |
| Logging | spdlog | printf-style, plog |
| GUI Framework | Qt | Cross-platform, native look, well-documented |

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    dnd-recorder                           │
├─────────────────────────────────────────────────────────┤
│  GUI Layer (Qt)                                          │
│  ├── Main Window (Qt Widgets)                            │
│  ├── Recording Controls                                  │
│  └── Transcript Viewer                                   │
├─────────────────────────────────────────────────────────┤
│  CLI Layer (main.cpp)                                   │
│  ├── Argument parsing                                    │
│  └── Command dispatch                                   │
├─────────────────────────────────────────────────────────┤
│  Core                                                    │
│  ├── SessionManager      - Session state                │
│  ├── AudioEngine         - Audio capture                │
│  └── TranscriptionEngine - whisper.cpp integration      │
├─────────────────────────────────────────────────────────┤
│  External Integration                                     │
│  ├── WhisperService       - Local transcription (cpp)  │
│  ├── SummarizerService    - GPT API (curl)             │
│  └── FileManager          - File I/O                   │
└─────────────────────────────────────────────────────────┘
```

---

## Learning Objectives

This project teaches:

| Topic | What You'll Learn |
|-------|------------------|
| Pointers & References | Raw memory, smart pointers (std::unique_ptr) |
| Memory Allocation | new/delete, RAII pattern |
| Structs & Classes | Data organization, encapsulation |
| File I/O | Reading/writing audio and text files |
| Networking | HTTP requests with libcurl |
| Audio | Microphone access, encoding |
| ML Inference | whisper.cpp integration, model loading |
| GUI Development | Window management, event handling |
| Build Systems | CMake, make, linking |
| Error Handling | Exceptions, return codes |

---

## Project Structure

```
desktop/
├── src/
│   ├── main.cpp
│   ├── gui/
│   │   ├── main_window.ui          # Qt Designer file
│   │   ├── main_window.cpp
│   │   ├── main_window.hpp
│   │   └── recording_controls.cpp
│   ├── cli/
│   │   ├── argument_parser.cpp
│   │   └── command_handler.cpp
│   ├── core/
│   │   ├── session_manager.cpp
│   │   ├── session_manager.hpp
│   │   ├── audio_engine.cpp
│   │   ├── audio_engine.hpp
│   │   ├── transcription_engine.cpp
│   │   └── transcription_engine.hpp
│   ├── services/
│   │   ├── whisper_service.cpp
│   │   ├── whisper_service.hpp
│   │   ├── summarizer_service.cpp
│   │   └── summarizer_service.hpp
│   ├── utils/
│   │   ├── logger.cpp
│   │   ├── file_manager.cpp
│   │   └── json_helper.cpp
│   └── types/
│       ├── session.hpp
│       └── transcript.hpp
├── models/                     # Whisper model files (.bin)
├── include/                    # Header files
├── tests/
├── CMakeLists.txt
└── README.md
```

---

## Implementation Details

### Audio Recording (PortAudio)

```cpp
// Pseudo-code example
class AudioEngine {
private:
    PaStream* stream;
    std::vector<float> audioBuffer;

public:
    void startRecording() {
        Pa_OpenDefaultStream(&stream, 1, 0, paFloat32, 
                           44100, 1024, callback, this);
        Pa_StartStream(stream);
    }
    
    void stopRecording() {
        Pa_StopStream(stream);
        Pa_CloseStream(stream);
    }
};
```

### HTTP Requests (libcurl)

```cpp
// Pseudo-code example
std::string callWhisperAPI(const std::vector<uint8_t>& audio) {
    CURL* curl = curl_easy_init();
    // Set URL, headers, POST data (audio file)
    // Perform request
    // Return response
}
```

### JSON Parsing (nlohmann/json)

```cpp
// Pseudo-code example
#include <nlohmann/json.hpp>

json response = json::parse(llmResponse);
std::string summary = response["summary"];
auto highlights = response["highlights"];
```

### Local Transcription (whisper.cpp)

```cpp
// Pseudo-code example
#include "whisper.h"

class WhisperService {
private:
    struct whisper_context* ctx;

public:
    bool loadModel(const std::string& modelPath) {
        ctx = whisper_init_from_file(modelPath.c_str());
        return ctx != nullptr;
    }

    std::string transcribe(const std::vector<float>& audio) {
        whisper_full_params params = whisper_default_params(whisper_sampling_strategy_t::WHISPER_SAMPLING_GREEDY);
        params.print_realtime = false;
        params.print_progress = false;

        whisper_full(ctx, params, audio.data(), audio.size());
        return whisper_get_full_result(ctx);
    }

    ~WhisperService() {
        if (ctx) whisper_free(ctx);
    }
};
```

---

## Getting Started

### Prerequisites

- C++17 compatible compiler (GCC, Clang, MSVC)
- CMake 3.15+
- ffmpeg (for audio encoding)
- OpenAI API key (for summarization only)
- whisper.cpp model file (downloaded at build or runtime)

### Dependencies Installation

**Ubuntu/Debian:**
```bash
sudo apt install cmake build-essential libcurl4-openssl-dev \
    libportaudio2 libavcodec-dev libjson-c-dev qt5-default
```

**macOS:**
```bash
brew install cmake curl portaudio ffmpeg qt@5
```

**Windows:**
- Visual Studio 2019+ with C++ workload
- vcpkg for dependencies (qt5, portaudio, curl)

### Build

```bash
cd desktop
mkdir build && cd build
cmake ..
make
```

### Run

```bash
./dnd-recorder --start "Curse of Strahd"
# ... play D&D ...
./dnd-recorder --stop
```

---

## GUI Requirements (Gathering Phase)

> **Note:** This section is for requirements gathering only. No implementation yet.

### Overview

A simple native GUI for recording and viewing transcriptions. The GUI should be minimal and focused on the core workflow.

### Required Features

| Feature | Priority | Notes |
|---------|----------|-------|
| Recording Controls | Must | Start/Pause/Resume/Stop buttons |
| Campaign Name Input | Must | Text field for session name |
| Recording Status | Must | Visual indicator (recording/paused/idle) |
| Duration Display | Must | Show elapsed recording time |
| Transcript View | Should | Display final transcription |

### Optional Features (Post-MVP)

- Audio level meter
- Model selection dropdown
- Export buttons
- Session history list

### Technology Options

| Framework | Pros | Cons |
|-----------|------|------|
| **Qt** | Cross-platform, native look, excellent docs, visual designer | Heavier dependency |
| ImGui | Immediate mode, simple | Non-native look |
| Native (Win32/GTK/Cocoa) | Lightweight, native | More code to write |
| FLTK | Simple, lightweight | Limited features |

**Selected: Qt** - Best balance of cross-platform support and developer experience.

### UI Mockup (Text)

```
┌────────────────────────────────────────────┐
│  D&D Session Recorder                       │
├────────────────────────────────────────────┤
│  Campaign: [________________]              │
│                                            │
│  [●RECORD] [⏸PAUSE] [⏹STOP]               │
│                                            │
│  Status: Recording...  ⏱ 00:45:23          │
│                                            │
│  ──── Transcript ────                     │
│  [Transcription will appear here...]       │
│                                            │
│  [Copy] [Save to File]                     │
└────────────────────────────────────────────┘
```

---

## Output

### Console Output

```
[2025-01-15 19:00:00] Session started: Curse of Strahd
[2025-01-15 19:00:05] Recording...
[2025-01-15 19:45:00] Paused
[2025-01-15 19:50:00] Resumed
[2025-01-15 22:30:00] Session ended
[2025-01-15 22:30:05] Transcribing...
[2025-01-15 22:32:00] Generating summary...
[2025-01-15 22:33:00] Done! Saved to sessions/2025-01-15.txt
```

### Saved File Format

```
SESSION TRANSCRIPT
Campaign: Curse of Strahd
Date: 2025-01-15
Duration: 3h 30m

SESSION SUMMARY
[AI-generated summary]

TIMESTAMPED TRANSCRIPT
[Full transcript with timestamps]

KEY HIGHLIGHTS
[Highlights from LLM]

NPCS ENCOUNTERED
[List of NPCs]

LOCATIONS
[List of locations]
```

---

## Known Challenges

| Challenge | Notes |
|-----------|-------|
| Cross-platform audio | Different APIs per OS (WASAPI, ALSA, CoreAudio) |
| Memory leaks | Must properly manage audio buffer lifetime |
| Large file handling | Sessions >1 hour need chunked processing |
| API errors | Implement retry logic with backoff |
| Curl complexity | Verbose API, easy to misuse |

---

## Future Enhancements (Post-MVP)

- System tray recording indicator
- Multiple audio input selection
- Multiple Whisper model sizes (tiny to large)
- Database for session history
- Session search and filtering
- Export to different formats (Markdown, PDF)
