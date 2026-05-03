# Live Stream Clipper - 直播切片工具

[中文版说明](./README_zh.md)

---

## Overview

Live Stream Clipper is a Python-based tool for automatically identifying and clipping highlight segments from live stream recordings. It leverages ASR (Automatic Speech Recognition), LLM-based emotion analysis, and FFmpeg to create a complete pipeline from video to edited clips.

## Features

- **ASR Transcription**: Convert speech to text with timestamps using faster-whisper or GLM-ASR
- **LLM Emotion Analysis**: Identify highlight/emotional segments using OpenAI-compatible APIs (DeepSeek, etc.)
- **Interactive Selection**: CLI interface for reviewing and selecting segments
- **Video Clipping**: Use FFmpeg to extract selected segments with original quality preserved

## Project Structure

```
auto-slice/
├── main.py              # Main entry point (full pipeline)
├── transcriber.py        # faster-whisper based transcription
├── transcriber_new.py    # GLM-ASR based transcription
├── clipper.py            # FFmpeg video clipper
├── config.json           # Configuration file
├── config_template.json  # Template for configuration
├── llm_client/           # LLM client module
├── mcp_server/           # MCP server for segment management
├── ui/                  # UI components
└── requirements.txt     # Python dependencies
```

## Installation

```bash
# Clone the repository
git clone <repository-url>
cd auto-slice

# Create config file from template
cp config_template.json config.json
# Edit config.json with your API keys and settings

# Install dependencies
pip install -r requirements.txt

# Install FFmpeg (system package)
# Ubuntu/Debian:
sudo apt install ffmpeg
# macOS:
brew install ffmpeg
```

## Configuration

Edit `config.json`:

```json
{
  "llm": {
    "base_url": "https://api.deepseek.com",
    "api_key": "your-api-key",
    "model": "deepseek-v4-pro",
    "temperature": 0.7
  },
  "mcp_server": {
    "host": "127.0.0.1",
    "port": 8765
  },
  "transcription": {
    "model": "large-v3",
    "language": "zh",
    "hotwords": []
  }
}
```

## Usage

### Full Pipeline

```bash
# Run complete pipeline: transcription -> analysis -> selection -> clipping
python main.py --video video.mp4
```

### Individual Steps

```bash
# Step 1: Transcription only
python main.py --transcribe-only --video video.mp4

# Step 2: LLM Analysis (requires transcript.json)
python main.py --analyze-only --transcript transcript.json -o segments.json

# Step 3: Segment Selection (with demo data)
python main.py --demo

# Step 4: Video Clipping
python main.py --clip-only --video video.mp4
```

### Direct Tool Usage

```bash
# Transcription with faster-whisper
python transcriber.py video.mp4 --output transcript.json

# Transcription with GLM-ASR
pip install modelscope
python transcriber_new.py video.mp3 --output transcript_glm.json

# Video clipping with FFmpeg
python clipper.py -i video.mp4 --segments selected_segments.json
```

## Command-Line Options

### main.py

| Option | Description |
|--------|-------------|
| `--video` | Input video file path |
| `--config` | Config file path (default: config.json) |
| `--transcript` | Transcript JSON file path |
| `--demo` | Use demo transcript data |
| `--transcribe-only` | Run transcription only |
| `--analyze-only` | Run LLM analysis only |
| `--select-only` | Run segment selection only |
| `--clip-only` | Run video clipping only |
| `--output`, `-o` | Output file path (default: segments.json) |

### transcriber.py

| Option | Description |
|--------|-------------|
| `input` | Input audio/video file (positional) |
| `--model`, `-m` | Model size (tiny/base/small/medium/large-v3) |
| `--language`, `-l` | Language code (default: zh) |
| `--output`, `-o` | Output file path |
| `--format`, `-f` | Output format (json/txt/srt/vtt) |

### clipper.py

| Option | Description |
|--------|-------------|
| `--input`, `-i` | Input video file (required) |
| `--segments`, `-s` | Segments JSON file (default: selected_segments.json) |
| `--output`, `-o` | Output directory (default: clips) |
| `--prefix` | Output filename prefix |
| `--dry-run` | Preview without actual clipping |

## Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Video     │───>│ Transcription│───>│  LLM        │───>│  Segment    │
│   File      │    │  (ASR)       │    │  Analysis   │    │  Selection  │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                                │
                                                                v
                                                         ┌─────────────┐
                                                         │  FFmpeg     │
                                                         │  Clipping   │
                                                         └─────────────┘
                                                                │
                                                                v
                                                         ┌─────────────┐
                                                         │   Clips     │
                                                         │   Output    │
                                                         └─────────────┘
```

## Requirements

- Python 3.8+
- ffmpeg
- faster-whisper (pip install faster-whisper)
- httpx (pip install httpx)
- For GLM-ASR: modelscope (pip install modelscope)

## License

MIT