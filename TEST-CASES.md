# Test Cases

## Overview

This document outlines the test cases for the Discord D&D Recap Bot.

---

## Session Management Tests

### Start Session

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| SM-01 | User in voice channel runs `/dnd-recap start-session` | Bot joins voice channel, enters recording mode, sends confirmation |
| SM-02 | User NOT in voice channel runs `/dnd-recap start-session` | Bot responds with error: "Join a voice channel first" |
| SM-03 | User runs `/dnd-recap start-session` while already recording | Bot responds with error: "Already recording. End the current session first" |
| SM-04 | User provides campaign name: `/dnd-recap start-session "Curse of Strahd"` | Bot records campaign name in session data |

### Pause Session

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| SM-05 | User runs `/dnd-recap pause-session` while recording | Bot pauses recording, status changes to "paused", sends confirmation |
| SM-06 | User runs `/dnd-recap pause-session` while already paused | Bot responds: "Session is already paused" |
| SM-07 | User runs `/dnd-recap pause-session` with no active session | Bot responds: "No active session to pause" |

### Resume Session

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| SM-08 | User runs `/dnd-recap resume-session` while paused | Bot resumes recording, status changes to "recording", sends confirmation |
| SM-09 | User runs `/dnd-recap resume-session` while recording | Bot responds: "Session is already recording" |
| SM-10 | User runs `/dnd-recap resume-session` with no active session | Bot responds: "No active session to resume" |

### End Session

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| SM-11 | User runs `/dnd-recap end-session` while recording | Bot leaves voice, starts processing, sends "Processing..." message |
| SM-12 | User runs `/dnd-recap end-session` while paused | Bot leaves voice, starts processing |
| SM-13 | User runs `/dnd-recap end-session` with no active session | Bot responds: "No active session to end" |

### Session Status

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| SM-14 | User runs `/dnd-recap status` during active session | Bot shows: campaign name, duration, status (recording/paused), participants |
| SM-15 | User runs `/dnd-recap status` with no active session | Bot responds: "No active session" |

---

## Audio Handling Tests

### Recording Quality

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| AH 5-minute session-01 | Record | Audio file captured, transcription succeeds |
| AH-02 | Record 30-minute session | Audio file captured, transcription succeeds |
| AH-03 | Record 2-hour session | Audio captured, transcription succeeds (may take longer) |
| AH-04 | Session with long silence | Transcription handles silence gracefully |

### Multiple Speakers

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| AH-05 | Session with 2 speakers | Both speakers transcribed |
| AH-06 | Session with 5+ speakers | All speakers transcribed |
| AH-07 | Speaker identification | (Nice to have) Different speakers identified |

### Cross-talk

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| AH-08 | Moderate overlapping speech | Transcription captures content |
| AH-09 | Heavy overlapping speech | Partial transcription, no crash |

### Edge Cases

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| AH-10 | Bot kicked from voice mid-session | Session ends gracefully, partial audio saved |
| AH-11 | Bot loses network mid-session | Attempt reconnection or end session gracefully |
| AH-12 | User mutes themselves | Audio captured when unmuted |
| AH-13 | User deafens themselves | Audio still captured (deafen doesn't affect voice) |

---

## Error Handling Tests

### Dependencies

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| EH-01 | ffmpeg not installed | Bot detects and sends: "ffmpeg required but not found" |
| EH-02 | OpenAI API key missing | Bot sends: "OpenAI API key not configured" |

### API Failures

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| EH-03 | Whisper API returns error | Retry logic triggers, then error message with retry option |
| EH-04 | GPT API returns error | Retry logic triggers, then error message with retry option |
| EH-05 | OpenAI rate limit hit | Backoff and retry, inform user of delay |
| EH-06 | Both APIs fail | Error message, partial audio preserved |

### Data Handling

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| EH-07 | Empty audio (everyone muted) | Bot responds: "No audio detected" |
| EH-08 | Corrupted audio file | Error handling, no crash |
| EH-09 | Disk full | Bot handles write error gracefully |

---

## Output Validation Tests

### Summary Generation

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| OV-01 | Valid session produces summary | JSON with: summary, highlights, characters, locations |
| OV-02 | Highlights contain timestamps | Each highlight has HH:MM:SS format |
| OV-03 | Empty session | Summary says "No significant events captured" |

### File Output

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| OV-04 | Session file created | File exists at `sessions/session_{id}.txt` |
| OV-05 | File contains transcript | Full text present |
| OV-06 | File contains summary | AI summary present |
| OV-07 | File contains highlights | Key moments present |

### Discord Embed

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| OV-08 | Embed displays correctly | Formatted with fields: summary, highlights, NPCs, locations |
| OV-09 | Embed within Discord limits | Under 6000 characters |
| OV-10 | Timestamp formatting | HH:MM:SS format readable |

### Validation Feature

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| OV-11 | User reacts to validate | Highlight marked as "validated" |
| OV-12 | User provides correction | Highlight updated with correction |

---

## Permission Tests

### Bot Permissions

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| PM-01 | Bot has Connect + Speak | Recording works |
| PM-02 | Bot missing Connect permission | Error: "I need permission to join your voice channel" |
| PM-03 | Bot missing Speak permission | Error: "I need permission to speak in the voice channel" |

### User Permissions

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| PM-04 | Any user runs command | Should work (no restrictions for MVP) |
| PM-05 | User with role restriction (future) | Only allowed roles can start |

---

## Privacy & Consent Tests

| ID | Test Case | Expected Result |
|----|-----------|-----------------|
| PC-01 | Recording starts | Confirmation message sent to channel |
| PC-02 | Session ends | "Processing" message shown |
| PC-03 | Audio files deleted after processing | Temp files cleaned up |

---

## Priority Matrix

| Priority | Test IDs |
|----------|----------|
| P0 - Must Pass | SM-01, SM-02, SM-05, SM-08, SM-11, AH-01, AH-02, OV-01, OV-04 |
| P1 - Critical | SM-03, SM-13, SM-14, AH-03, AH-04, EH-01, EH-02, OV-02, OV-08 |
| P2 - Important | SM-04, SM-06, SM-07, SM-09, SM-10, SM-15, AH-05, AH-06, OV-03, OV-05, OV-06, OV-07 |
| P3 - Nice to Have | AH-07, AH-08, AH-09, AH-10, AH-11, AH-12, EH-03, EH-04, OV-11, OV-12, PM-04, PM-05 |

---

## Test Environment Setup

### Required for Testing

1. Discord server with voice channels
2. Bot with required permissions
3. OpenAI API key (with credits)
4. ffmpeg installed
5. At least 2 test users

### Test Session Data

- Campaign: "Test Campaign"
- Duration: 5-10 minutes
- Participants: 2-3 users
- Content: Prepared script with highlights
