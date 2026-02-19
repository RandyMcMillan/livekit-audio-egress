# LiveKit Audio Egress - Repository Analysis

**Date:** 2026-02-15  
**Repository:** RandyMcMillan/livekit-audio-egress  
**Language:** Rust 2021 Edition  
**Total Lines of Code:** 277 lines across 4 source files

---

## Executive Summary

This is a **Rust-based audio egress service** that records and mixes audio from LiveKit rooms, specifically designed for Nostr Nests (nostrnests.com). The application connects to LiveKit WebRTC rooms as a guest, subscribes to audio tracks, mixes them in real-time, and outputs an HLS (HTTP Live Streaming) stream.

**Current Status:** 🚧 **Work in Progress / Incomplete Implementation**

---

## Project Overview

### Purpose
The application serves as an audio recording and streaming service for LiveKit-based voice chat rooms. It:
- Connects to LiveKit rooms via WebRTC
- Subscribes to remote audio tracks from participants
- Mixes multiple audio sources together
- Encodes the mixed audio using AAC codec
- Outputs an HLS stream for playback/distribution

### Technology Stack
- **Language:** Rust (Edition 2021)
- **Runtime:** Tokio (async/await)
- **WebRTC:** LiveKit SDK v0.7.0
- **Audio Processing:** FFmpeg (via ffmpeg-rs-raw)
- **Audio Codec:** AAC (using libfdk_aac encoder)
- **Output Format:** HLS (HTTP Live Streaming)
- **Logging:** pretty_env_logger
- **HTTP Client:** reqwest v0.12.9

---

## Architecture

### Component Breakdown

#### 1. **main.rs** (34 lines)
- Entry point of the application
- Initializes logging system
- Displays FFmpeg version information
- Spawns Egress tasks for multiple rooms (currently hardcoded to one room ID)
- Uses Tokio async runtime

**Key Details:**
- Hardcoded room ID: `"c9987fa0-c21f-4fba-b2b9-332694120498"`
- Supports spawning multiple room egress tasks concurrently
- Logs warnings if egress tasks fail

#### 2. **egress.rs** (84 lines)
- Manages LiveKit room connection and track subscription
- Handles authentication via Nostr Nests API
- Resamples and forwards audio data to the mixer

**Key Features:**
- **Authentication:** Fetches guest token from `nostrnests.com/api/v1/nests/{room}/guest`
- **WebRTC Connection:** Connects to `wss://nostrnests.com`
- **Audio Processing:**
  - Subscribes to audio tracks when participants join
  - Resamples audio to 48kHz, 2 channels (stereo)
  - Uses `audio_resampler::AudioResampler` for format conversion
  - Sends resampled data to mixer via unbounded channel

**Issues Identified:**
- ⚠️ Only processes the **first** audio track subscribed, then breaks out of event loop
- Missing handling for track unsubscription/participant leaving
- No error handling for mixer channel send failures (uses `?` operator)

#### 3. **mixer.rs** (131 lines)
- Core audio mixing and encoding component
- Manages multiple speaker channels
- Encodes mixed audio to AAC
- Outputs to HLS stream

**Configuration:**
- Sample Rate: 48,000 Hz
- Channels: 2 (Stereo)
- Codec: AAC (attempting to use libfdk_aac)
- Output Format: HLS with segment deletion enabled
- Audio Delay: 10ms (480 samples)

**Critical Issues Found:**
1. 🐛 **Infinite Loop in mix()**: The condition `while x < out_samples.len()` never advances `x`, causing an infinite loop (lines 97-103)
2. 🐛 **SpeakerChannel.next_samples() Always Returns None**: This prevents any audio from being mixed
3. ⚠️ **Commented Code**: "TODO: this isnt working" for libfdk_aac codec selection (line 34)
4. ⚠️ **No Error Handling**: `try_recv()` in a tight loop without delay could peg CPU
5. ⚠️ **Output Path Hardcoded**: Uses `{id}/live.m3u8` which requires directory to exist

**Audio Mixing Logic:**
- Collects samples from all active speakers
- Applies equal weight mixing (1/n where n = number of speakers)
- Should mix to match encoder frame size requirements

#### 4. **speaker.rs** (28 lines)
- Represents a single participant's audio channel
- Buffers incoming audio frames
- **INCOMPLETE IMPLEMENTATION**

**Critical Issue:**
- 🔴 **next_samples() Stub**: Always returns `None`, meaning no audio is ever mixed (line 26)
- Has infrastructure for buffering (VecDeque) but no logic to use it
- Missing PTS (presentation timestamp) management logic

---

## Dependencies Analysis

### Core Dependencies
1. **livekit (0.7.0)** - WebRTC client for LiveKit
   - Features: `rustls-tls-native-roots`
   
2. **ffmpeg-rs-raw** (Git dependency)
   - Source: `https://git.v0l.io/Kieran/ffmpeg-rs-raw.git`
   - Commit: `bde945fe887dfdb38fff096bbf1928b9e8e8469f`
   - ⚠️ **Custom fork**, not official FFmpeg bindings
   
3. **tokio (1.41.1)** - Async runtime
   - Features: `rt`, `rt-multi-thread`, `macros`, `signal`

4. **reqwest (0.12.9)** - HTTP client
   - Features: `json`

### Supporting Dependencies
- `anyhow` - Error handling
- `bytes` - Byte buffer utilities
- `libc` - C library bindings
- `serde` - Serialization/deserialization
- `log` + `pretty_env_logger` - Logging
- `futures-util` - Async utilities

---

## Critical Bugs & Issues

### 🔴 HIGH PRIORITY

1. **speaker.rs:26** - `next_samples()` returns `None` unconditionally
   - **Impact:** No audio is ever mixed or output
   - **Fix Required:** Implement sample buffering and retrieval logic

2. **mixer.rs:97-103** - Infinite loop in mixing logic
   - **Impact:** Would hang the application if audio data reached this point
   - **Issue:** Variable `x` is never incremented
   - **Fix Required:** Add `x += 1` or use iterator

### ⚠️ MEDIUM PRIORITY

3. **egress.rs:51** - Only handles first audio track
   - **Impact:** Multi-participant rooms won't be fully recorded
   - **Fix Required:** Remove `break` statement, continue processing events

4. **mixer.rs:35** - libfdk_aac codec not working
   - **Impact:** May fall back to less optimal AAC encoder
   - **Fix Required:** Debug codec selection or use alternative

5. **mixer.rs:65** - Tight loop with try_recv()
   - **Impact:** Potential CPU spinning
   - **Fix Required:** Use blocking recv() or add delays

6. **No README.md**
   - **Impact:** Poor documentation for users/contributors
   - **Fix Required:** Add comprehensive README

### 📋 LOW PRIORITY

7. **Hardcoded values** throughout
   - Room ID in main.rs
   - WebSocket URL in egress.rs
   - API endpoint in egress.rs

8. **No unit tests** - No test coverage

9. **No CI/CD configuration** - No automated testing or builds

10. **Missing .cargo/config.toml** - No build optimization flags

---

## Data Flow

```
LiveKit Room (nostrnests.com)
    ↓
[Guest Token Authentication]
    ↓
[WebRTC Connection via livekit SDK]
    ↓
[Audio Track Subscription] → RemoteAudioTrack
    ↓
[AudioResampler] → 48kHz, 2ch, i16 samples
    ↓
[UnboundedChannel] → MixerData{sid, data}
    ↓
[SpeakerChannel Buffer] → VecDeque<MixerData>
    ↓
[Mixer.mix()] → Weighted sum of all speakers
    ↓
[AAC Encoder] → Compressed audio packets
    ↓
[HLS Muxer] → {room_id}/live.m3u8 + segments
```

---

## Configuration & Constants

| Parameter | Value | Location |
|-----------|-------|----------|
| Sample Rate | 48,000 Hz | mixer.rs:13 |
| Channels | 2 (Stereo) | mixer.rs:12 |
| Audio Delay | 10ms (480 samples) | mixer.rs:60 |
| Codec | AAC (AV_CODEC_ID_AAC) | mixer.rs:36 |
| Sample Format | S16 (16-bit signed int) | mixer.rs:39 |
| Output Format | HLS | mixer.rs:44 |
| HLS Flags | delete_segments | mixer.rs:44 |

---

## Build & Runtime Requirements

### Build Requirements
- **Rust:** 1.93.0+ (2021 edition)
- **Cargo:** 1.93.0+
- **FFmpeg development libraries** (for ffmpeg-rs-raw)
  - libavcodec
  - libavformat
  - libavutil
  - libfdk-aac (optional, for better AAC encoding)

### Runtime Requirements
- Network access to nostrnests.com (HTTPS + WSS)
- Writable filesystem for HLS output
- FFmpeg runtime libraries

### Platform Support
- Linux (primary target, based on libc usage)
- Potentially macOS/Windows (Rust + FFmpeg are cross-platform)

---

## Recommendations

### Immediate Actions Required
1. ✅ **Implement `speaker.rs:next_samples()`** - Critical for functionality
2. ✅ **Fix infinite loop in `mixer.rs:mix()`** - Add loop increment
3. ✅ **Remove early break in `egress.rs`** - Support multiple tracks
4. ✅ **Add README.md** - Document usage and requirements

### Short-term Improvements
5. 📝 Add configuration file (TOML/YAML) for:
   - WebSocket URL
   - API endpoints
   - Room IDs
   - Audio parameters
6. 🧪 Add unit tests for mixer logic
7. 📊 Add metrics/monitoring (participant count, buffer levels, bitrate)
8. 🔒 Add proper error recovery and reconnection logic

### Long-term Enhancements
9. 🎛️ Dynamic mixing (adjust weights based on audio levels)
10. 🔇 Silence detection (skip encoding silent frames)
11. 📦 Container image (Docker) for easy deployment
12. 🌐 Support for multiple output formats (not just HLS)
13. 🎚️ Audio effects (normalization, noise gate, compressor)
14. 📡 Webhooks for recording events (start/stop/error)

---

## Security Considerations

1. **Authentication:** Uses guest tokens from Nostr Nests API
   - No token validation or expiry checking
   - Tokens might be logged (check logging configuration)

2. **Network:** 
   - Hardcoded nostrnests.com endpoint
   - No TLS certificate pinning
   - Uses rustls-tls-native-roots (good)

3. **File System:**
   - Writes to filesystem without permission checks
   - No disk space monitoring
   - HLS segments could accumulate if cleanup fails

4. **Dependencies:**
   - Custom FFmpeg fork (potential supply chain risk)
   - Should audit git.v0l.io dependency

---

## License & Copyright

- **License:** MIT License
- **Copyright:** 2024 Nostr Nests
- **File:** LICENSE (22 lines)

---

## Conclusion

This is a **promising but incomplete** audio egress implementation for LiveKit. The architecture is sound, using appropriate async patterns and established audio processing libraries. However, **critical functionality is missing** in the speaker buffering and mixing logic, preventing the application from working end-to-end.

**Estimated Completion:** With the critical bugs fixed, this could be production-ready with ~200-300 additional lines of code focusing on:
1. Proper sample buffering in SpeakerChannel
2. PTS (timestamp) synchronization
3. Configuration management
4. Error handling and recovery
5. Documentation

**Primary Use Case:** Recording and streaming audio from Nostr Nests voice chat rooms for archival or redistribution purposes.
