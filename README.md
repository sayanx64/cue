# cue

An open-source AI copilot that floats over your screen, sees what you see, hears your meetings, and tries to stay hidden from screen shares.

cue is a self-hosted Electron app. Bring your own AI key: OpenAI, Anthropic, Google Gemini, or Nvidia.

> Important: capture hiding is best-effort, not guaranteed. Electron asks the OS to exclude the overlay from capture, but capture tools, Windows/macOS versions, GPU drivers, and external cameras can bypass that. Do not use cue where hidden assistance violates rules, consent requirements, or law.

## Platform support

- Windows 10/11: primary target
- macOS: supported
- Linux: untested

## What it does

cue combines three inputs:

- Screen: full-resolution screenshot when a feature needs visual context
- Microphone: your side of the conversation
- Meeting/system audio: the other side of the conversation, when OS loopback capture is available

Main actions:

- Assist: `Ctrl+Enter` on Windows, `Cmd+Enter` on macOS
- Solve coding problem on screen: `Ctrl+H` on Windows, `Cmd+H` on macOS
- What should I say?
- Follow-up questions
- Recap
- Ask anything in the input box

## Install on Windows

### Option A: download a build

Download the Windows installer or portable build from Releases when available:

- `cue-*-windows-x64.exe` for the installer
- `cue-*-windows-x64.exe` portable build if published

Windows SmartScreen may warn because this is an unsigned open-source app. If you trust the build, choose More info -> Run anyway.

### Option B: run from source

Install Node.js 18+ first.

```bash
git clone https://github.com/Blueturboguy07/cue.git
cd cue
npm install
npm start
```

Build Windows artifacts:

```bash
npm run dist:win
```

This creates NSIS installer and portable Windows artifacts under `dist/`.

## Install on macOS

Run from source:

```bash
npm install
npm start
```

Build a macOS zip:

```bash
npm run dist:mac
```

macOS requires Microphone and Screen Recording permissions. If macOS says the app is damaged, clear quarantine for a trusted local build:

```bash
xattr -cr /Applications/cue.app
```

## First launch

1. Open Settings from the `...` button, or press `Ctrl+,` / `Cmd+,`.
2. Pick a provider and paste your API key.
3. For listening features, use OpenAI with Whisper/audio access or Gemini. Anthropic is useful for screen/coding help but does not provide speech-to-text.
4. Click the top-bar listen button to start/stop mic and meeting-audio capture.

On Windows, if microphone capture fails, check:

Windows Settings -> Privacy & security -> Microphone -> allow microphone access and allow desktop apps.

## Zoom and capture hiding

cue calls Electron's `setContentProtection(true)`.

- On Windows, Electron uses Windows display-affinity capture protection where supported.
- On macOS, Electron uses the platform window-sharing protection APIs.

For Zoom, use:

Zoom -> Settings -> Share Screen -> Advanced -> Screen capture mode -> Advanced capture with window filtering.

Avoid capture modes without window filtering. Even with the right setting, hiding remains best-effort.

## How it works

cue is an Electron app:

- `main.js`: overlay window, global shortcuts, screen capture, IPC, AI orchestration
- `renderer/`: glass UI, mic capture, system-audio loopback capture, onboarding
- `src/`: provider clients, prompts, settings store, screenshot and audio helpers

Capture pipeline:

```text
main process
  ├─ transparent always-on-top overlay window
  ├─ Electron desktopCapturer screenshots
  ├─ speech-to-text via OpenAI Whisper or Gemini
  └─ LLM streaming via OpenAI, Anthropic, Gemini, or Nvidia

renderer process
  ├─ getUserMedia microphone capture
  └─ getDisplayMedia loopback system-audio capture when available
```

## Troubleshooting

### Listening does nothing

- Add an OpenAI key with Whisper/audio access, or a Gemini key.
- On Windows, allow microphone access for desktop apps.
- System/meeting audio depends on OS loopback support. If no loopback track is available, cue can still transcribe your mic.

### A feature returns 403 or no access to model

Your API key is probably restricted. Enable the required model or audio/transcription access, use an unrestricted key, or add a Gemini key for transcription.

### cue appears in a screen share

Capture hiding is best-effort. In Zoom, enable Advanced capture with window filtering. Some capture tools and OS versions can still show the overlay.

## Privacy

- No cue server
- No telemetry
- API keys are stored locally in `cue-data.json`
- Screenshots and audio are sent only to the AI provider you configured
- cue does not persist screenshots or audio beyond the current in-memory transcript

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
