# Meeting-to-Text: Local Speaker-Separated Transcription Skill

## The Problem
There is a frequent need to transcribe meeting audio or video recordings into text, especially when using AI assistants like OpenClaw that require structured, machine-readable formats. However:
- Commercial services (like Feishu/Lark Minutes or iFlytek) can be expensive.
- Open-source models are notoriously difficult to deploy.
- Video-to-audio extraction, speech-to-text (ASR), and speaker diarization (Speaker Separation) usually belong to entirely different modules and repositories, making integration frustrating.
- It is difficult for AI agents to call these fragmented tools seamlessly.

## The Solution
**Meeting-to-Text** solves this by packaging these complex capabilities into a single, cohesive **Agent Skill**. By following the setup instructions to deploy the local dependencies and models, you can easily perform local speaker-separated transcription.
Furthermore, the entire system is highly lightweight: after full deployment, the entire system (including all models) takes up **less than 3GB** of disk space.

*Read this document in other languages: [中文(简体)](README_ZH.md)*

## Features
- **True Offline Processing:** 100% local. No data leaves your machine.
- **Auto Audio Extraction:** Uses FFmpeg to extract audio streams from MP4, MKV, MOV, and more.
- **Accurate ASR:** Leverages Alibaba's `SenseVoiceSmall` for fast, accurate multilingual transcription.
- **Speaker Diarization:** Uses `3D-Speaker` to cluster voices and generate separated transcripts with speaker labels (e.g., `Speaker 1`, `Speaker 2`).
- **Universal Agent & CLI Ready:** Structured as a `.md` skill definition and a single Python entrypoint. While we use Codex/OpenClaw as examples, it can be attached to **any** AI agent, copilot, or CLI automation flow.

## Supported Formats
- **Video:** `.mp4`, `.mkv`, `.mov`, `.avi`, `.webm`
- **Audio:** `.wav`, `.mp3`, `.m4a`, `.aac`, `.flac`, `.ogg`

---

## Quick Start (Agent Automated Install)
Instead of manually installing dependencies and models, simply copy and paste the following prompt to your AI assistant (e.g. Codex, OpenClaw, Cursor, etc.):

> **Prompt:** Please follow the instructions in this repository to install this skill for me, including its required models and dependencies: https://github.com/henrCh1/meeting-to-text

---

## Manual Installation Guide

### 1. Prerequisites
- **Python 3.10+**
- **FFmpeg:** Download a pre-compiled Windows build (e.g., from [gyan.dev](https://www.gyan.dev/ffmpeg/builds/)) and extract it. You will need the path to `ffmpeg.exe`.

### 2. Install Python Dependencies
It is highly recommended to use a virtual environment (`venv` or `conda`).
```powershell
# Create and activate your environment
python -m venv envs\asr
.\envs\asr\Scripts\activate

# Install requirements
pip install -r requirements.txt
```

### 3. Clone Required Repositories
You need the `3D-Speaker` repository available locally for speaker diarization.
```powershell
mkdir repos
cd repos
git clone https://github.com/alibaba-damo-academy/3D-Speaker.git
```

### 4. Download Models
The system requires two local models from ModelScope/HuggingFace. Download them into your `models/` directory:
- [SenseVoiceSmall](https://www.modelscope.cn/models/iic/SenseVoiceSmall)
- [fsmn-vad](https://www.modelscope.cn/models/iic/speech_fsmn_vad_zh-cn-16k-common-pytorch)

*(Note: `3D-Speaker` checkpoints will automatically cache to the hub directory on first run).*

---

## Configuration & Usage

The skill relies on absolute paths to function deterministically. You **must** configure the paths before using the skill.

### Standalone Python Setup
You can run the script manually by injecting the paths as Environment Variables, or by modifying the fallback paths in `skills/meeting-to-text/scripts/meeting_to_text.py`:
```powershell
$env:MEETING_TO_TEXT_FFMPEG="C:\path\to\ffmpeg\bin\ffmpeg.exe"
$env:MEETING_TO_TEXT_SENSEVOICE="C:\path\to\models\SenseVoiceSmall"
$env:MEETING_TO_TEXT_VAD="C:\path\to\models\fsmn-vad"
$env:MEETING_TO_TEXT_3D_SPEAKER="C:\path\to\repos\3D-Speaker"

python skills/meeting-to-text/scripts/meeting_to_text.py --input "C:\path\to\video.mp4" --output "C:\path\to\output.txt"
```

### Using as an Agent Skill (Codex, OpenClaw, AutoGPT, etc.)
1. Open `skills/meeting-to-text/SKILL.md`.
2. Find the `<YOUR_CONDA_ENV_PYTHON_PATH>` placeholder and replace it with the absolute path to your virtual environment's `python.exe`.
3. Find the `C:\path\to\your\meeting-to-text\scripts\meeting_to_text.py` placeholder and replace it with the absolute path to the script.
4. Replace `<YOUR_WORKSPACE_TEMP_PATH>` with a temporary directory path (e.g., `C:\temp\meeting_workspace`).
5. Register the `SKILL.md` with your agent platform.

## Output Format
The resulting `.txt` file will look like this:
```text
[00:00:00 - 00:00:05] Speaker 1: Hello everyone, let's start the meeting.

[00:00:05 - 00:00:10] Speaker 2: Yes, I can hear you clearly. Let's begin.
```

## License
Provided under the MIT License. See `LICENSE` for details.
