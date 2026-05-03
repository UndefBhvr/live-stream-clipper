# Live Stream Clipper - 直播切片工具

[English Version](./README.md)

---

## 项目概述

直播切片工具是一个基于Python开发的自动化工具，用于从直播录像中识别并剪辑精彩片段。它结合了ASR（自动语音识别）、LLM情感分析以及FFmpeg，形成了一套从视频到精剪片段的完整流程。

## 功能特点

- **ASR语音转文字**：使用faster-whisper或GLM-ASR将语音转换为带时间戳的文本
- **LLM情感分析**：利用OpenAI兼容API（DeepSeek等）识别精彩/情绪片段
- **交互式选择**：命令行界面用于查看和选择片段
- **视频剪辑**：使用FFmpeg以原始质量提取所选片段

## 项目结构

```
auto-slice/
├── main.py              # 主入口（完整流程）
├── transcriber.py        # 基于faster-whisper的转录
├── transcriber_new.py    # 基于GLM-ASR的转录
├── clipper.py            # FFmpeg视频剪辑器
├── config.json           # 配置文件
├── config_template.json  # 配置模板
├── llm_client/           # LLM客户端模块
├── mcp_server/           # MCP服务器（片段管理）
├── ui/                  # UI组件
└── requirements.txt     # Python依赖
```

## 安装

```bash
# 克隆仓库
git clone <repository-url>
cd auto-slice

# 从模板创建配置文件
cp config_template.json config.json
# 编辑config.json填入您的API密钥和设置

# 安装依赖
pip install -r requirements.txt

# 安装FFmpeg（系统包）
# Ubuntu/Debian:
sudo apt install ffmpeg
# macOS:
brew install ffmpeg
```

## 配置

编辑 `config.json`:

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

## 使用方法

### 完整流程

```bash
# 运行完整流程：转录 -> 分析 -> 选择 -> 剪辑
python main.py --video video.mp4
```

### 分步执行

```bash
# 步骤1：仅转录
python main.py --transcribe-only --video video.mp4

# 步骤2：仅LLM分析（需要transcript.json）
python main.py --analyze-only --transcript transcript.json -o segments.json

# 步骤3：片段选择（使用演示数据）
python main.py --demo

# 步骤4：视频剪辑
python main.py --clip-only --video video.mp4
```

### 直接使用工具

```bash
# 使用faster-whisper进行转录
python transcriber.py video.mp4 --output transcript.json

# 使用GLM-ASR进行转录
pip install modelscope
python transcriber_new.py video.mp3 --output transcript_glm.json

# 使用FFmpeg进行视频剪辑
python clipper.py -i video.mp4 --segments selected_segments.json
```

## 命令行参数

### main.py

| 参数 | 说明 |
|------|------|
| `--video` | 输入视频文件路径 |
| `--config` | 配置文件路径（默认：config.json） |
| `--transcript` | 转录JSON文件路径 |
| `--demo` | 使用演示转录数据 |
| `--transcribe-only` | 仅运行转录 |
| `--analyze-only` | 仅运行LLM分析 |
| `--select-only` | 仅运行片段选择 |
| `--clip-only` | 仅运行视频剪辑 |
| `--output`, `-o` | 输出文件路径（默认：segments.json） |

### transcriber.py

| 参数 | 说明 |
|------|------|
| `input` | 输入音频/视频文件（位置参数） |
| `--model`, `-m` | 模型大小 (tiny/base/small/medium/large-v3) |
| `--language`, `-l` | 语言代码（默认：zh） |
| `--output`, `-o` | 输出文件路径 |
| `--format`, `-f` | 输出格式 (json/txt/srt/vtt) |

### clipper.py

| 参数 | 说明 |
|------|------|
| `--input`, `-i` | 输入视频文件（必需） |
| `--segments`, `-s` | 片段JSON文件（默认：selected_segments.json） |
| `--output`, `-o` | 输出目录（默认：clips） |
| `--prefix` | 输出文件名前缀 |
| `--dry-run` | 预览模式（不实际剪辑） |

## 工作流程

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   视频      │───>│   转录      │───>│   LLM       │───>│   片段      │
│   文件      │    │   (ASR)     │    │   分析      │    │   选择      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                                │
                                                                v
                                                         ┌─────────────┐
                                                         │   FFmpeg    │
                                                         │   剪辑      │
                                                         └─────────────┘
                                                                │
                                                                v
                                                         ┌─────────────┐
                                                         │   片段      │
                                                         │   输出      │
                                                         └─────────────┘
```

## 环境要求

- Python 3.8+
- ffmpeg
- faster-whisper (pip install faster-whisper)
- httpx (pip install httpx)
- 使用GLM-ASR时：modelscope (pip install modelscope)

## 许可证

MIT