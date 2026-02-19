# LiveKit Audio Egress

A Rust-based audio egress service for recording and streaming audio from LiveKit rooms. Designed for use with Nostr Nests voice chat platform.

## Overview

This application connects to LiveKit WebRTC rooms, subscribes to audio tracks from participants, mixes them in real-time, and outputs an HLS (HTTP Live Streaming) stream for playback or archival.

## Features

- 🎙️ **Real-time Audio Capture** - Connects to LiveKit rooms and captures participant audio
- 🎚️ **Multi-track Mixing** - Mixes multiple audio sources with equal weighting
- 🔊 **Audio Resampling** - Automatically resamples to 48kHz stereo
- 📡 **HLS Streaming** - Outputs HTTP Live Streaming format for broad compatibility
- 🎵 **AAC Encoding** - High-quality audio compression using AAC codec
- ⚡ **Async/Await** - Built on Tokio for efficient async I/O

## Requirements

### Build Requirements

- **Rust** 1.93.0 or later
- **FFmpeg development libraries** (libavcodec, libavformat, libavutil)
- Optional: **libfdk-aac** for enhanced AAC encoding

#### Installing FFmpeg on Ubuntu/Debian

```bash
sudo apt-get update
sudo apt-get install -y \
    libavcodec-dev \
    libavformat-dev \
    libavutil-dev \
    libfdk-aac-dev
```

#### Installing FFmpeg on macOS

```bash
brew install ffmpeg
```

### Runtime Requirements

- Network access to LiveKit server (default: nostrnests.com)
- Writable filesystem for HLS output

## Building

```bash
# Clone the repository
git clone https://github.com/RandyMcMillan/livekit-audio-egress.git
cd livekit-audio-egress

# Build the project
cargo build --release

# The binary will be at ./target/release/egress-audio
```

## Usage

### Basic Usage

```bash
# Run with default settings
cargo run --release
```

### Configuration

Currently, configuration is done by editing `src/main.rs`:

```rust
// Edit the room ID in main.rs
for r in vec!["YOUR-ROOM-ID-HERE".to_string()] {
    // ...
}
```

### Output

The application generates HLS streams in the following format:
```
{room-name}/
  ├── live.m3u8        # HLS playlist
  └── *.ts             # HLS segments (auto-deleted after streaming)
```

## Audio Specifications

| Parameter | Value |
|-----------|-------|
| Sample Rate | 48,000 Hz |
| Channels | 2 (Stereo) |
| Sample Format | 16-bit signed integer |
| Codec | AAC |
| Output Format | HLS |
| Buffering Delay | 10ms |

## Architecture

```
LiveKit Room → Audio Tracks → Resampler → Mixer → AAC Encoder → HLS Muxer
```

### Components

- **Egress** - Manages LiveKit connection and track subscription
- **Mixer** - Combines multiple audio sources and handles encoding
- **SpeakerChannel** - Buffers and manages individual participant audio

## Development Status

⚠️ **This project is currently under active development and may not be fully functional.**

See [ANALYSIS.md](ANALYSIS.md) for a detailed technical analysis and known issues.

### Known Issues

- [ ] SpeakerChannel buffering not fully implemented
- [ ] Only processes first audio track subscription
- [ ] Hardcoded server URLs and room IDs
- [ ] Limited error handling and recovery
- [ ] No configuration file support

## Dependencies

- **livekit** - WebRTC client SDK
- **ffmpeg-rs-raw** - FFmpeg bindings for Rust
- **tokio** - Async runtime
- **reqwest** - HTTP client for authentication

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

### Development Setup

```bash
# Enable debug logging
RUST_LOG=debug cargo run

# Run with specific log levels
RUST_LOG=egress_audio=trace,livekit=debug cargo run
```

## License

MIT License - see [LICENSE](LICENSE) file for details.

Copyright (c) 2024 Nostr Nests

## Acknowledgments

- Built for [Nostr Nests](https://nostrnests.com) voice chat platform
- Uses [LiveKit](https://livekit.io) WebRTC infrastructure
- FFmpeg for audio processing

## Support

For questions or support:
- Open an issue on GitHub
- Contact: [Repository Owner](https://github.com/RandyMcMillan)

---

**Note:** This is a specialized tool for LiveKit audio egress. For general LiveKit recording needs, consider the official [LiveKit Egress](https://docs.livekit.io/egress/) service.
