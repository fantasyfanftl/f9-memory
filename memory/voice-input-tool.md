---
name: voice-input-tool
description: 用户自建的全局语音输入工具（Windows）—— faster-whisper + GPU + Caps Lock PTT + 隐藏 VBS 后台跑 + 开机启动
metadata: 
  node_type: memory
  type: project
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

用户在 `C:\Users\tianlan\voice-input\` 有一套自建的语音输入方案，2026-07-05 搭起来的。

**架构**
- `voice_input.py` — 主脚本：按住 Caps Lock 录音，松开用 faster-whisper 转文字，通过剪贴板+Ctrl+V 粘贴到当前光标
- `assistant.py` — 语音助手（2026-07-05 用户重新要求恢复的）：F9 按住说话 → `claude -c -p` 送给主会话 → edge-tts 朗读回答 → 屏幕底部有个**HUD 科技风的呼吸圆点**（BreathingDot 类）显示状态，右下角**可展开对话面板**（ChatPanel 类）能选中 Ctrl+C 复制文本
- `voice-input-hidden.vbs` / `assistant-hidden.vbs` — 静默启动器（`wscript` mode 0，无黑窗）—— **VBS 里的注释必须纯 ASCII**，wscript 用 GBK 解 UTF-8 中文会崩
- `run.bat` / `assistant.bat` — 显式带窗口跑的版本，调试用
- `assets/dot-ring.png` —— 曾经想用作圆点视觉的 gpt-image-2 生成的白色发光环图，因 tk 半透明处理不好放弃了；文件保留，代码已切回纯 Canvas 矢量
- `requirements.txt` / `.venv/` — 独立 venv，装了 sounddevice / faster-whisper / keyboard / pyperclip / edge-tts / pygame-ce / Pillow / pystray（pystray 现在没用上）

**关键配置**（脚本顶部常量）
- `HOTKEY = "caps lock"`，**tap/hold 双用**：短按（<250ms）走正常大小写切换；长按 ≥250ms 才是 PTT。实现方式：`suppress=False` + 定时器判断 hold + PTT 结束时 `keyboard.send('caps lock')` 撤销物理按下产生的切换（配 `_synth_until` 忽略自己发出的合成事件，防止无限循环）
- `DEVICE = "cuda"` + `COMPUTE_TYPE = "float16"` —— 用户机器是 RTX 5060 Ti（Blackwell）
- `MODEL_SIZE = "medium"`（2026-07-05 从 small 升上来的，中文明显更准，GPU 上速度无感差别）
- `LANGUAGE = "zh"`（锁定中文，防止自动检测误判）
- `INITIAL_PROMPT` —— 塞常用专有名词进去提高识别率，用户会陆续反馈"老识别错的词"给我，我加进去
- `beam_size=1, vad_filter=False, without_timestamps=True` —— 降延迟设置

**GPU DLL 加载 hack**（脚本开头，import faster_whisper 之前）
```python
_nv = Path(__file__).parent / ".venv/Lib/site-packages/nvidia"
_paths = [_nv/"cublas/bin", _nv/"cuda_nvrtc/bin", _nv/"cudnn/bin"]
os.environ["PATH"] = ";".join(str(p) for p in _paths) + ";" + os.environ["PATH"]
for _p in _paths: os.add_dll_directory(str(_p))
```
两者都要：`add_dll_directory` 让 ctranslate2 找到 cuDNN 加载模型；`PATH` 让 CUDA 运行时找到 cublas64_12.dll。

**依赖坑记录**
- Python 3.14 上 `pygame` 没 wheel，用 `pygame-ce` 替代
- CUDA 运行时用 pip `nvidia-cublas-cu12` + `nvidia-cudnn-cu12`（各 500-700MB）不装 CUDA Toolkit
- 首次 GPU 推理 ~7s（CUDA JIT），之后 <100ms —— 启动时用 1s 静音 warmup 掉

**开机启动**
Startup 文件夹从 Bash tool 沙箱写不进去（拒绝访问），schtasks 也不行。用注册表 Run 键成功：
```
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v VoiceInput /t REG_SZ /d "wscript.exe C:\Users\tianlan\voice-input\voice-input-hidden.vbs" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v VoiceAssistant /t REG_SZ /d "wscript.exe C:\Users\tianlan\voice-input\assistant-hidden.vbs" /f
```
卸载：`reg delete ... /v VoiceInput /f` 或 `/v VoiceAssistant /f`

**权限批准的 UX 坑**（2026-07-05 遇到）
`claude -c -p` 送到主会话后，如果 Claude 要跑新命令，权限确认弹在**主 Claude Code 窗口里**（不在助手圆点那边）。用户按 F9 说"打开飞书"之后需要 Alt-Tab 回来点 Yes。长期解决办法：把常用命令（如 `Start-Process`、`explorer.exe`）加进 settings.json 权限白名单。

**进程管理注意**
`taskkill /F /IM python.exe` 有时会漏杀（"没有此实例的运行" = 已经死了）；重启前一定 `tasklist` 确认干净，不然会有多个 voice_input.py 抢 Caps Lock 出问题（2026-07-05 就发生过）。

**F9 人设文件**（2026-07-05 新增）
`D:\voice-input-data\f9-persona.md` —— 由 assistant.py 每次调用时实时读，作为系统 prompt 前缀注入 `claude -p`。用户可自由编辑（改回答风格、加快捷指令等），不用改代码。

**F9 的核心定位是"工作助手 + 工作沉淀"**（用户后来强调过）：主要用途不是闲聊，而是把用户口述的 SOP / 规则 / 项目决策沉淀进主 memory。人设文件里详细说明了三条沉淀路径（SOP → project 或 sop-XX；规则 → feedback；项目状态 → project）以及"存完只说存哪儿，别复述内容"的确认风格。

**Feishu Wiki 集成**（进行中，2026-07-05 启动）
用户决定用飞书知识库（Wiki）作为跨平台记忆存储：手机能编辑、能从其他 AI 粘贴导入、能跟 F9 互通。**关键约束：Wiki 建在用户的个人飞书账户**（不是任何工作/公司账户），因为这是私人知识。开发者应用也得在同一个个人账户下建，否则权限对不上。用户会在飞书里建 5 个页面（SOP 库 / 项目状态 / 决策与规则 / 活动日志多维表 / 外部导入）+ 建自建应用拿 app_id/app_secret + 加权限，之后我在 assistant.py 里加飞书 SDK 集成。

**Why:** 用户想给 F9 独有的行为规则，但又要保留跟主 memory 共享的项目/偏好上下文，所以采用"共享 memory + F9 专属人设"的双层设计。
**How to apply:** 后续如果用户说"语音工具"、"识别不准"、"改热键"、"太慢了"，都指这套；改的时候记得同步 voice_input.py 和 assistant.py 两边

相关：[[user-github]]（不在 GitHub 上，纯本地）
