# Quick Reference Card

**LiveKit Audio Egress** - Quick developer reference

---

## 🏗️ Build & Run

```bash
# Build
cargo build --release

# Run
cargo run

# Debug mode
RUST_LOG=debug cargo run

# With trace logging
RUST_LOG=trace cargo run
```

---

## 📁 Project Structure

```
livekit-audio-egress/
├── src/
│   ├── main.rs       # Entry point, spawns egress tasks
│   ├── egress.rs     # LiveKit connection & track handling
│   ├── mixer.rs      # Audio mixing & encoding
│   └── speaker.rs    # Per-speaker buffering
├── Cargo.toml        # Dependencies
└── Cargo.lock        # Locked versions
```

---

## 🔧 Key Constants

```rust
// mixer.rs
const SAMPLE_RATE: u32 = 48_000;  // 48kHz
const NB_CHANNELS: u32 = 2;       // Stereo
const DELAY_MS: f64 = 10.0;       // Startup delay
```

---

## 🐛 Known Critical Bugs

| File | Line | Issue | Fix |
|------|------|-------|-----|
| speaker.rs | 26 | Returns None | Implement buffering |
| mixer.rs | 97-103 | Infinite loop | Add `x += 1` |
| egress.rs | 51 | Early break | Remove `break` |

---

## 🔄 Data Flow

```
LiveKit → Egress → Resampler → Channel → Speaker → Mixer → Encoder → HLS
```

---

## 📊 Audio Format

| Stage | Sample Rate | Channels | Format |
|-------|-------------|----------|--------|
| Input | Varies | 1-2 | Varies |
| Resampler | 48kHz | 2 | i16 |
| Mixer | 48kHz | 2 | i16 |
| Encoder | 48kHz | 2 | AAC |
| Output | 48kHz | 2 | HLS |

---

## 🧪 Testing

```bash
# Format code
cargo fmt

# Lint code
cargo clippy

# Check without building
cargo check

# Build with warnings as errors
cargo build --release -- -D warnings
```

---

## 📦 Dependencies

```toml
livekit = "0.7.0"              # WebRTC SDK
ffmpeg-rs-raw = { git = "..." } # FFmpeg bindings
tokio = "1.41.1"               # Async runtime
reqwest = "0.12.9"             # HTTP client
anyhow = "1.0.81"              # Error handling
```

---

## 🎯 Common Tasks

### Adding Configuration
```rust
// Current: Hardcoded
const URL: &str = "wss://nostrnests.com";

// Better: From environment
let url = env::var("LIVEKIT_URL")
    .unwrap_or("wss://nostrnests.com".to_string());

// Best: From config file
let config = Config::from_file("config.toml")?;
```

### Error Handling
```rust
// Current: Propagate with ?
let result = some_function()?;

// Better: Log and continue
if let Err(e) = some_function() {
    warn!("Function failed: {}", e);
    // Recovery logic
}
```

### Adding Logging
```rust
use log::{trace, debug, info, warn, error};

info!("Starting mixer for room {}", room_id);
debug!("Received {} samples from {}", len, sid);
```

---

## 🔍 Debugging

### Enable Logging
```bash
# All debug
RUST_LOG=debug cargo run

# Specific module
RUST_LOG=egress_audio::mixer=trace cargo run

# Multiple modules
RUST_LOG=egress_audio=debug,livekit=info cargo run
```

### Backtrace
```bash
RUST_BACKTRACE=1 cargo run
RUST_BACKTRACE=full cargo run
```

### GDB/LLDB
```bash
# Build with debug symbols
cargo build

# Debug with lldb (macOS)
rust-lldb target/debug/egress-audio

# Debug with gdb (Linux)
rust-gdb target/debug/egress-audio
```

---

## 📝 Code Style

### Naming
```rust
// Types: PascalCase
struct SpeakerChannel { }

// Functions/variables: snake_case
fn record_track() { }
let sample_rate = 48000;

// Constants: SCREAMING_SNAKE_CASE
const SAMPLE_RATE: u32 = 48_000;

// Lifetimes: short, lowercase
fn process<'a>(data: &'a [i16]) { }
```

### Documentation
```rust
/// One-line summary.
///
/// Detailed explanation.
///
/// # Arguments
/// * `arg` - Description
///
/// # Returns
/// Description of return
///
/// # Errors
/// When errors occur
pub fn function(arg: Type) -> Result<Return> { }
```

---

## 🚀 Performance Tips

### Async Best Practices
```rust
// Good: Non-blocking
tokio::spawn(async move { ... });

// Bad: Blocking in async
async fn bad() {
    std::thread::sleep(Duration::from_secs(1)); // DON'T
}

// Good: Async sleep
async fn good() {
    tokio::time::sleep(Duration::from_secs(1)).await;
}
```

### Memory Efficiency
```rust
// Pre-allocate vectors
let mut buffer = Vec::with_capacity(expected_size);

// Reuse buffers
buffer.clear(); // Instead of buffer = Vec::new()

// Use references
fn process(data: &[i16]) { } // Not Vec<i16>
```

---

## 🔐 Security Checklist

- [ ] Validate all inputs
- [ ] Check buffer bounds
- [ ] Sanitize file paths
- [ ] Verify token expiry
- [ ] Rate limit API calls
- [ ] Monitor disk usage
- [ ] Use TLS for connections
- [ ] Don't log sensitive data

---

## 📚 Helpful Resources

### Documentation
- [README.md](README.md) - Project overview
- [ARCHITECTURE.md](ARCHITECTURE.md) - Technical details
- [ANALYSIS.md](ANALYSIS.md) - Known issues
- [CONTRIBUTING.md](CONTRIBUTING.md) - How to contribute

### External
- [Rust Book](https://doc.rust-lang.org/book/)
- [Tokio Docs](https://tokio.rs)
- [LiveKit Docs](https://docs.livekit.io)
- [FFmpeg Docs](https://ffmpeg.org/documentation.html)

---

## 🆘 Common Errors

### "libavcodec not found"
```bash
# Ubuntu/Debian
sudo apt-get install libavcodec-dev

# macOS
brew install ffmpeg
```

### "Failed to connect to room"
- Check network connectivity
- Verify room ID is correct
- Ensure nostrnests.com is accessible
- Check token is valid

### "No such file or directory"
- HLS output directory doesn't exist
- Create directory matching room name
- Check write permissions

---

## 💡 Quick Fixes

### Fix Speaker Buffering
```rust
// In speaker.rs, replace next_samples():
pub fn next_samples(&mut self, next_pts: i64) -> Option<Vec<i16>> {
    let mut samples = Vec::new();
    while let Some(frame) = self.frames.front() {
        if self.pts + frame.data.len() as i64 > next_pts {
            break;
        }
        if let Some(frame) = self.frames.pop_front() {
            samples.extend_from_slice(&frame.data);
            self.pts += frame.data.len() as i64;
        }
    }
    if samples.is_empty() {
        None
    } else {
        Some(samples)
    }
}
```

### Fix Mixer Loop
```rust
// In mixer.rs, fix lines 97-103:
for x in 0..out_samples.capacity() {
    let weight = 1f32 / speaking.len() as f32;
    for speaker in &speaking {
        if x < speaker.len() {
            out_samples.push((speaker[x] as f32 * weight) as i16);
        }
    }
}
```

---

**Version:** 1.0 | **Updated:** Feb 2026 | **License:** MIT
