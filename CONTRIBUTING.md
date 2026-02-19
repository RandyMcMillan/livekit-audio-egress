# Contributing to LiveKit Audio Egress

Thank you for your interest in contributing to this project! This guide will help you get started.

## Code of Conduct

Be respectful, collaborative, and constructive. This project follows standard open-source etiquette.

## Getting Started

### Prerequisites

1. **Rust** 1.93.0 or later
2. **FFmpeg development libraries**
3. **Git**
4. **Basic understanding of:**
   - Rust async/await
   - Audio processing concepts
   - WebRTC basics (helpful but not required)

### Setting Up Development Environment

```bash
# Clone the repository
git clone https://github.com/RandyMcMillan/livekit-audio-egress.git
cd livekit-audio-egress

# Install FFmpeg development libraries (Ubuntu/Debian)
sudo apt-get install -y libavcodec-dev libavformat-dev libavutil-dev

# Build the project
cargo build

# Run with debug logging
RUST_LOG=debug cargo run
```

## How to Contribute

### 1. Finding Something to Work On

Check the [ANALYSIS.md](ANALYSIS.md) file for:
- 🔴 High priority bugs
- ⚠️ Medium priority improvements
- 📋 Low priority enhancements

Or browse the GitHub issues for tagged items:
- `good first issue` - Great for newcomers
- `help wanted` - Need community assistance
- `bug` - Something that needs fixing
- `enhancement` - New features

### 2. Before You Start

1. **Check for existing work** - Search issues and PRs
2. **Comment on the issue** - Let others know you're working on it
3. **Ask questions** - Don't hesitate to ask for clarification

### 3. Development Workflow

```bash
# Create a feature branch
git checkout -b fix/speaker-buffering

# Make your changes
# Edit files...

# Build and test
cargo build
cargo test  # When tests exist

# Run the application
cargo run

# Check your code
cargo fmt     # Format code
cargo clippy  # Lint code

# Commit your changes
git add .
git commit -m "Fix speaker buffering implementation"

# Push to your fork
git push origin fix/speaker-buffering

# Open a Pull Request on GitHub
```

### 4. Coding Standards

#### Rust Style
- Follow standard Rust formatting (`cargo fmt`)
- Fix clippy warnings (`cargo clippy`)
- Use meaningful variable names
- Add comments for complex logic

#### Code Quality
- **Keep changes minimal** - Only change what's necessary
- **One issue per PR** - Don't bundle unrelated changes
- **Write tests** - When test infrastructure exists
- **Document public APIs** - Use doc comments (///)

#### Example of Good Code
```rust
/// Buffers audio samples for a single speaker.
///
/// # Arguments
/// * `data` - Resampled audio data from the track
///
/// # Example
/// ```
/// let mut speaker = SpeakerChannel::new("track-123".to_string());
/// speaker.put(mixer_data);
/// ```
pub fn put(&mut self, data: MixerData) {
    self.frames.push_back(data);
}
```

### 5. Pull Request Guidelines

#### PR Title Format
```
<type>: <short description>

Examples:
fix: Implement speaker buffering logic
feat: Add configuration file support
docs: Update README with installation steps
refactor: Simplify mixer loop logic
```

#### PR Description Template
```markdown
## Description
Brief description of what this PR does.

## Related Issue
Fixes #123

## Changes Made
- Implemented SpeakerChannel::next_samples()
- Added PTS tracking
- Fixed infinite loop in mixer

## Testing
- [ ] Built successfully
- [ ] Ran with sample data
- [ ] No clippy warnings
- [ ] Formatted with cargo fmt

## Checklist
- [ ] Code follows project style
- [ ] Comments added for complex logic
- [ ] Documentation updated if needed
- [ ] No unnecessary changes included
```

## Priority Issues to Work On

### 🔴 Critical (Blocking Basic Functionality)

1. **Implement `SpeakerChannel::next_samples()`**
   - File: `src/speaker.rs`
   - Difficulty: Medium
   - Impact: High
   - Description: Needs to return buffered audio samples

2. **Fix infinite loop in `Mixer::mix()`**
   - File: `src/mixer.rs` lines 97-103
   - Difficulty: Easy
   - Impact: High
   - Description: Variable `x` is never incremented

3. **Remove early break in `Egress::run()`**
   - File: `src/egress.rs` line 51
   - Difficulty: Easy
   - Impact: Medium
   - Description: Should handle multiple tracks

### ⚠️ High Priority (Quality Improvements)

4. **Add configuration file support**
   - Files: Multiple
   - Difficulty: Medium
   - Impact: Medium
   - Description: Replace hardcoded values with config

5. **Add error recovery logic**
   - Files: Multiple
   - Difficulty: Hard
   - Impact: High
   - Description: Handle disconnections gracefully

### 📋 Medium Priority (Nice to Have)

6. **Add unit tests**
   - Files: New test files
   - Difficulty: Medium
   - Impact: Medium
   - Description: Test mixer and speaker logic

7. **Add metrics/monitoring**
   - Files: Multiple
   - Difficulty: Medium
   - Impact: Low
   - Description: Track performance and health

## Testing Your Changes

### Manual Testing Checklist

```bash
# 1. Build succeeds
cargo build --release

# 2. No compiler warnings
cargo build 2>&1 | grep warning

# 3. Clippy passes
cargo clippy -- -D warnings

# 4. Code is formatted
cargo fmt -- --check

# 5. Application runs
RUST_LOG=info cargo run

# 6. Check output
ls -la {room-name}/  # Verify HLS files created
```

### When Tests Exist
```bash
# Run all tests
cargo test

# Run specific test
cargo test test_speaker_buffering

# Run with output
cargo test -- --nocapture
```

## Documentation

### What to Document

1. **Public functions** - Use doc comments
2. **Complex algorithms** - Explain the approach
3. **Edge cases** - Note special handling
4. **Breaking changes** - Highlight in PR

### Documentation Style

```rust
/// Short one-line description.
///
/// Longer explanation if needed. Can span
/// multiple paragraphs.
///
/// # Arguments
/// * `arg1` - Description of arg1
/// * `arg2` - Description of arg2
///
/// # Returns
/// Description of return value
///
/// # Errors
/// When this function might error
///
/// # Examples
/// ```
/// let result = function(arg1, arg2);
/// ```
pub fn function(arg1: Type1, arg2: Type2) -> Result<ReturnType> {
    // implementation
}
```

## Common Development Tasks

### Adding a New Dependency

1. Add to `Cargo.toml`
2. Justify why it's needed in PR
3. Check for security advisories
4. Prefer well-maintained crates

```bash
# Check for security issues
cargo audit

# Update dependencies
cargo update
```

### Debugging

```bash
# Run with full debug output
RUST_LOG=trace cargo run

# Run with backtrace on panic
RUST_BACKTRACE=1 cargo run

# Use debugger (lldb on macOS, gdb on Linux)
rust-lldb target/debug/egress-audio
```

### Performance Profiling

```bash
# Build with release + debug info
cargo build --release --features debug

# Use perf (Linux)
perf record -g target/release/egress-audio
perf report

# Use Instruments (macOS)
instruments -t "Time Profiler" target/release/egress-audio
```

## Getting Help

### Resources
- **ARCHITECTURE.md** - Detailed technical documentation
- **ANALYSIS.md** - Known issues and recommendations
- **README.md** - Basic usage information

### Where to Ask Questions
1. Open a GitHub issue with `question` label
2. Comment on relevant existing issues
3. Reach out to repository maintainers

### Common Questions

**Q: Where do I start?**
A: Pick a `good first issue` from the issues page or fix one of the critical bugs listed in ANALYSIS.md.

**Q: How do I test without a LiveKit server?**
A: Currently requires access to nostrnests.com. Mock/test infrastructure would be a great contribution!

**Q: Why use a custom FFmpeg fork?**
A: Historical reasons. Consider migrating to official bindings as a contribution.

**Q: Can I add a new feature?**
A: Yes! But please open an issue first to discuss the design.

## Release Process

(For maintainers)

1. Update version in `Cargo.toml`
2. Update CHANGELOG.md
3. Create git tag: `git tag v0.2.0`
4. Push tag: `git push origin v0.2.0`
5. Create GitHub release
6. (Future) Publish to crates.io

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

## Thank You!

Every contribution, no matter how small, helps improve this project. We appreciate your time and effort!

**Happy coding! 🦀🎵**
