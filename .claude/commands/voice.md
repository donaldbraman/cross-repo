# Voice Command

**Version:** 1.0.0
**Last Updated:** 2026-01-31

Start a voice conversation with Claude Code. Use `$ARGUMENTS` as an optional message to speak first.

## Usage

| Command | Action |
|---------|--------|
| `/voice` | Start listening immediately |
| `/voice How does the auth module work?` | Speak the message, then listen for response |

## Instructions

When the user runs `/voice`, initiate a voice conversation using the VoiceMode MCP tools.

### Prerequisites Check

First, verify voice services are available by checking the MCP tools. If `converse` or `listen_for_speech` tools are not available, inform the user:

```text
Voice services not connected. Set up with:
  uvx voice-mode-install --yes
  claude mcp add --scope user voicemode -- uvx voice-mode
  voicemode service start whisper
  voicemode service start kokoro
Then restart Claude Code. See .claude/guides/voice-setup.md for details.
```

### Start Conversation

**If `$ARGUMENTS` provided:**

Use the `converse` tool to speak the provided message and listen for a response:

- `message`: The provided arguments
- `wait_for_response`: true
- `listen_duration_min`: 3
- `vad_aggressiveness`: 2

**If no arguments:**

Use the `converse` tool to prompt the user and listen:

- `message`: "I'm listening. Go ahead."
- `wait_for_response`: true
- `listen_duration_min`: 3
- `vad_aggressiveness`: 2

### Continue the Conversation

After receiving the user's spoken input:

1. Process the transcribed text as a normal prompt
2. Respond using `converse` with `wait_for_response: true` to continue the voice loop
3. Keep the conversation going until the user says "stop", "done", "exit", "quit", or types a text input

### Voice Settings

Use these defaults for natural conversation:

| Parameter | Value | Reason |
|-----------|-------|--------|
| `listen_duration_min` | 3 | Prevents cutting off mid-sentence |
| `vad_aggressiveness` | 2 | Balanced noise filtering |
| `speed` | 1.0 | Natural speech rate |

If the user is in a noisy environment, increase `vad_aggressiveness` to 3.

---

## Related

- [voice-setup guide](../guides/voice-setup.md) - Full installation and configuration guide
- [VoiceMode MCP](https://github.com/mbailey/voicemode) - Upstream project

---

## Version History

### 1.0.0 (2026-01-31)

- Initial release
- Voice conversation loop with VoiceMode MCP
- Automatic prerequisites check
- Configurable VAD and speech parameters
