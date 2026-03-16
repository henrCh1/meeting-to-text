# Meeting-to-Text: 本地化会议语音转写与发言人分离Skill

## 核心痛点
在日常工作与学习中，我们总是有把会议录音或者录屏转成文字的需求。尤其是有了 OpenClaw 这种 AI 助手之后，把会议的录音或者录像转换为 AI 能直接阅读的格式变得尤为重要。然而，目前市面上存在诸多痛点：
- **成本高昂**：使用“飞书妙记”或“讯飞听见”等商业化的语音转文字功能比较昂贵。
- **部署困难**：网上开源的模型（如 Whisper 等）通常都有不小的部署难度。
- **模块碎片化**：视频抽离音频、语音识别 (ASR)、以及说话人分离 (Speaker Diarization) 通常分属完全不同的项目和模块，搭建一整套流水线十分麻烦。
- **智能体难以调用**：这些散落的工具带有复杂的接口，难以直接被智能体 (Agent) 顺畅调用。

## 解决方案
**Meeting-to-Text** 正是为了解决以上问题而生。我们将这些分散且复杂的功能打包成了一个标准的 **Skill (技能)**。您只需要根据说明部署本地的基础依赖和模型，就可以很容易地在本地轻松完成上述所有功能。
而且，完全部署后整个系统**十分轻量化**，所有的模型和代码部署完毕后，整个系统占用磁盘空间**不到 3GB**。

*Read this document in other languages: [English](README.md)*

## 核心特性
- **完全纯本地、保护隐私**：100% 离线运行，会议机密数据绝对不上传云端。
- **音视频自动提取**：内置 FFmpeg 调用，支持直接传入 MP4、MKV 等视频文件，系统会自动进行真实的音频轨道抽离。
- **高精度 ASR 转写**：底层接入阿里开源的 `SenseVoiceSmall`，识别速度极快，中英混合识别准确率极高。
- **智能说话人分离**：使用 `3D-Speaker` 提取声纹特征并进行聚类，输出带有时间戳和发言人标签 (如 `说话人1`、`说话人2`) 的逐字稿。
- **通用智能体与 CLI 支持**：以通用 Skill `.md` 格式和单一入口 Python 脚本形式提供封装。无论是 Codex、OpenClaw，还是其他任意大模型 Agent 或 CLI 自动化脚本，均能开箱即用无缝接入。

## 支持的输入格式
- **视频:** `.mp4`, `.mkv`, `.mov`, `.avi`, `.webm`
- **音频:** `.wav`, `.mp3`, `.m4a`, `.aac`, `.flac`, `.ogg`

---

## 快速开始（智能体一键安装）
您可以免去手动的繁杂配置，直接将下面这段话复制发送给您的 AI 智能体（如 Codex, OpenClaw, Cursor 等等）：

> **Prompt:** 请根据这个网站中的指引为我安装这个 skill 还有其对应需要的模型和依赖：https://github.com/henrCh1/meeting-to-text

---

## 手动部署与安装指南

### 1. 前置准备
- **Python 3.10+**
- **FFmpeg:** 请下载 Windows 的预编译版本（例如从 [gyan.dev](https://www.gyan.dev/ffmpeg/builds/) 下载）并解压。后续需要用到 `ffmpeg.exe` 的路径。

### 2. 安装 Python 依赖
强烈建议使用虚拟环境（`venv` 或 `conda`）。
```powershell
# 创建并激活您的虚拟环境
python -m venv envs\asr
.\envs\asr\Scripts\activate

# 安装所有依赖
pip install -r requirements.txt
```

### 3. 克隆必需的算法库仓库
说话人分离功能依赖官方的 `3D-Speaker` 仓库代码。
```powershell
mkdir repos
cd repos
git clone https://github.com/alibaba-damo-academy/3D-Speaker.git
```

### 4. 下载核心模型
本系统依赖两个基于 ModelScope 下载的本地模型。请将它们下载并完整放置在您的 `models/` 目录下（保持原文件夹名）：
- [SenseVoiceSmall](https://www.modelscope.cn/models/iic/SenseVoiceSmall)
- [fsmn-vad](https://www.modelscope.cn/models/iic/speech_fsmn_vad_zh-cn-16k-common-pytorch)

*(注：`3D-Speaker` 所需的具体权重文件在首次运行时会自动下载缓存到对应 hub 目录，您只需要保证代码库存在即可)。*

---

## 环境变量配置与使用

为了保证确定性，本技能依赖绝对路径来寻找模型。您**必须**在运行前配置好您的本地环境路径。

### 方式一：独立运行 Python 脚本
您可以通过设置环境变量，或直接修改 `skills/meeting-to-text/scripts/meeting_to_text.py` 前几行的占位符路径，在命令行中直接调用提取脚本：
```powershell
$env:MEETING_TO_TEXT_FFMPEG="C:\你的路径\ffmpeg\bin\ffmpeg.exe"
$env:MEETING_TO_TEXT_SENSEVOICE="C:\你的路径\models\SenseVoiceSmall"
$env:MEETING_TO_TEXT_VAD="C:\你的路径\models\fsmn-vad"
$env:MEETING_TO_TEXT_3D_SPEAKER="C:\你的路径\repos\3D-Speaker"

python skills/meeting-to-text/scripts/meeting_to_text.py --input "C:\会议录像.mp4" --output "C:\输出逐字稿.txt"
```

### 方式二：作为智能体技能 (Codex, OpenClaw 等任意 Agent) 接入
1. 打开 `skills/meeting-to-text/SKILL.md` 文件。
2. 找到 `<YOUR_CONDA_ENV_PYTHON_PATH>` 占位符，将其替换为您配置好的虚拟环境 `python.exe` 的绝对路径。
3. 找到 `C:\path\to\your\meeting-to-text\scripts\meeting_to_text.py` 占位符，将其替换为该 Python 脚本的实际绝对物理路径。
4. 将 `<YOUR_WORKSPACE_TEMP_PATH>` 替换为一个用于存放临时文件的目录（例如 `C:\temp\meeting_workspace`）。
5. 按照您的 Agent 平台说明，将修改后的 `SKILL.md` 安装或注册进您的助手即可。

## 输出样例
最终生成的 `.txt` 逐字稿内容格式如下：
```text
[00:00:00 - 00:00:05] 说话人1：大家好，我们现在开始开会。

[00:00:05 - 00:00:10] 说话人2：好的，声音很清楚，听得到。
```

## 许可证
本项目采用 MIT License 开源许可协议。详情请查阅 `LICENSE` 文件。
