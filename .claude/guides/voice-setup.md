# Voice Interface Setup Guide

**Version:** 1.0.0
**Last Updated:** 2026-01-31

## Overview

This guide documents the fully open source voice interface for Claude Code using VoiceMode MCP with local Whisper (STT) and Kokoro (TTS). All audio processing runs locally on your machine — no cloud audio services required.

## Architecture

```text
Claude Code <--MCP--> VoiceMode Server --> Whisper.cpp (STT, port 2022)
                                       --> Kokoro (TTS, port 8880)
                                       --> Microphone (sounddevice)
                                       --> Speakers (audio playback)
```

- **Whisper.cpp** — Local speech-to-text with Core ML and Metal acceleration on Apple Silicon
- **Kokoro** — Local text-to-speech (82M params, Apache 2.0), high-quality English voices
- **VoiceMode MCP** — Orchestration layer providing `converse` and `listen_for_speech` tools to Claude Code
- **Silero VAD** — Voice activity detection for automatic silence/speech boundary detection

## Components

| Component | Version | License | Port | Resource Usage |
|-----------|---------|---------|------|----------------|
| VoiceMode MCP | 8.x | MIT | stdio | Minimal |
| Whisper.cpp | 1.8.x | MIT | 2022 | ~300MB RAM |
| Kokoro | 0.2.x | Apache 2.0 | 8880 | ~1.3GB RAM |

## Installation

### Prerequisites

- macOS with Apple Silicon (M1/M2/M3/M4)
- Homebrew
- `uvx` (from `uv` package manager)
- Claude Code CLI

### Step 1: Install VoiceMode

```bash
uvx voice-mode-install --yes
```

This installs:

- VoiceMode MCP server
- portaudio (via Homebrew, for audio capture)
- Whisper.cpp with Core ML model
- Kokoro TTS service

### Step 2: Register MCP Server

```bash
claude mcp add --scope user voicemode -- uvx voice-mode
```

This adds VoiceMode to `~/.claude.json` so all Claude Code sessions can use it.

### Step 3: Start Services

```bash
voicemode service start whisper
voicemode service start kokoro
```

### Step 4: Verify

```bash
voicemode service status
```

Expected output:

```text
WHISPER: Running (port 2022, Core ML enabled, Metal GPU)
KOKORO: Running (port 8880)
```

### Step 5: Restart Claude Code

Restart Claude Code so it picks up the new MCP server. The voice tools (`converse`, `listen_for_speech`) will be available.

## Usage

### Quick Voice Conversation

Type `/voice` in Claude Code to start a voice interaction, or simply ask Claude to listen:

```text
> listen to me
> let's talk
> /voice
```

Claude will speak a prompt, listen for your response, and continue the conversation.

### Voice Parameters

The `converse` tool supports these parameters:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `wait_for_response` | true | Listen for response after speaking |
| `listen_duration_max` | 120s | Maximum listen time |
| `listen_duration_min` | 2.0s | Minimum recording before silence detection |
| `vad_aggressiveness` | 2 | Voice detection strictness (0=permissive, 3=strict) |
| `speed` | 1.0 | Speech rate (0.25-4.0) |
| `chime_enabled` | — | Audio feedback chime when listening starts |

### Noisy Environment Tips

If background noise (kids, music) causes false triggers:

- Increase `vad_aggressiveness` to 3 (strictest)
- Increase `listen_duration_min` to 3-4 seconds
- Use `/voice` to explicitly trigger listening rather than leaving it in continuous mode

## Service Management

### Start/Stop Services

```bash
voicemode service start whisper
voicemode service start kokoro
voicemode service stop whisper
voicemode service stop kokoro
```

### Enable at Login (Optional)

```bash
voicemode service enable whisper
voicemode service enable kokoro
```

### View Logs

```bash
voicemode service logs whisper
voicemode service logs kokoro
```

### Check Status

```bash
voicemode service status
```

## Troubleshooting

### Microphone Not Working

1. **macOS permissions:** System Settings > Privacy & Security > Microphone — your terminal app (Ghostty, iTerm2, Terminal) must have access
2. **First-time prompt:** macOS shows a permission dialog the first time the mic is accessed. If you missed it, toggle the permission manually
3. **Test mic access:**

   ```bash
   uvx --from voice-mode python -c "import sounddevice as sd; rec = sd.rec(int(0.5*16000), samplerate=16000, channels=1); sd.wait(); print('OK:', len(rec), 'samples')"
   ```

### Kokoro Slow to Start

Kokoro loads the TTS model on first start (~1.3GB into RAM). This can take 30-60 seconds. Check progress with:

```bash
voicemode service status
```

Wait until status shows "Running" before using voice.

### Whisper Transcription Inaccurate

The default `base` model is fast but less accurate. Upgrade to `small` or `medium` for better results:

```bash
voicemode whisper install --model small
voicemode service restart whisper
```

Model comparison:

| Model | Size | RAM | Accuracy | Speed |
|-------|------|-----|----------|-------|
| base | 141MB | ~300MB | Good | Fastest |
| small | 466MB | ~600MB | Better | Fast |
| medium | 1.5GB | ~1.5GB | Best | Moderate |

### Services Not Starting

```bash
# Check if ports are in use
lsof -i :2022
lsof -i :8880

# Kill stale processes
voicemode service stop whisper
voicemode service stop kokoro
voicemode service start whisper
voicemode service start kokoro
```

### VoiceMode MCP Not Connecting

Verify the MCP config in `~/.claude.json`:

```json
{
  "mcpServers": {
    "voicemode": {
      "type": "stdio",
      "command": "uvx",
      "args": ["voice-mode"]
    }
  }
}
```

If missing, re-add:

```bash
claude mcp add --scope user voicemode -- uvx voice-mode
```

Then restart Claude Code.

## Multi-Session Notes

- Whisper and Kokoro run as shared local servers — multiple Claude Code sessions connect to them
- Only use voice actively in **one session at a time** (microphone is a single resource)
- Other sessions retain access to voice tools but should not listen simultaneously

## Related

- [VoiceMode MCP GitHub](https://github.com/mbailey/voicemode)
- [VoiceMode Documentation](https://voice-mode.readthedocs.io/)
- [Whisper.cpp](https://github.com/ggerganov/whisper.cpp)
- [Kokoro TTS](https://huggingface.co/hexgrad/Kokoro-82M)

---

## Version History

### 1.0.0 (2026-01-31)

- Initial guide
- VoiceMode MCP + Whisper.cpp + Kokoro stack
- macOS Apple Silicon focus
- Service management and troubleshooting
