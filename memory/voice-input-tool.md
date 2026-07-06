---
name: voice-input-tool
description: "用户自建的 Windows 语音工具完整交接文档。Caps Lock 语音输入 + F9 一键发送到 Claude Code。设计给\"如果坏了要靠另一个 AI 会话来修\"的场景用。"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

# 语音工具 · 完整交接文档 (2026-07-06)

用户在 Windows 上自建的语音输入工具。跨会话/跨 AI 都要能读懂这份文档就直接开始修 bug。

## 一句话概括

三个脚本, 两输入 + 一输出:
- **`voice_input.py`** (核心) — 处理两个输入热键:
  - **Caps Lock 长按 (>250ms)** → 语音识别 + 粘贴到当前光标（任何输入框都行）
  - **F9 按住** → 自动跳到 Claude Code 窗口 + 语音识别 + Ctrl+V 粘贴 + Enter 发送
  - 短按 Caps Lock（<250ms）保留正常大小写切换功能
- **`voice_reply_watcher.py`** (可选, 手动开) — 后台盯当前 Claude Code 会话的 jsonl 文件, Claude 每次回复自动 TTS 朗读出来
  - 启动时锁死到"当时最活跃的 session", 不再自动切换到别的对话 (用户明确要求"只这一个对话有语音")
  - 只朗读 ≤ 500 字的短回复
  - **不开机自启** (用户手动决定要不要)

## 文件树 (全在 `C:\Users\tianlan\voice-input\`)

```
voice-input/
├── voice_input.py                   # 核心脚本 (Caps Lock + F9 两个输入热键)
├── voice-input-hidden.vbs           # voice_input 静默启动器 (wscript mode 0)
├── voice_reply_watcher.py           # AI 回复朗读器 (2026-07-06 新加)
├── voice-reply-hidden.vbs           # 回复朗读器的静默启动器
├── run.bat                          # 可见启动器 (调试时用)
├── requirements.txt                 # 依赖清单
├── assets/
│   ├── dot-ring.png                 # 曾经想给 assistant 的圆点素材 (已弃用)
│   └── voice-samples/               # edge-tts 语音试听样本 (定型后没用了)
├── .venv/                           # Python 虚拟环境 (Python 3.14, ~2GB)
└── (已删除: assistant.py 及其一切衍生 - F9 subprocess 方案彻底放弃)
```

可写数据目录 (用户明确要求放 D 盘, 见 [feedback-writable-data-on-d-drive]):
```
D:\voice-input-data\
├── f9-persona.md                    # (F9 subprocess 时代的产物, 已弃用但没删)
├── assistant-history.json           # (同上, 已弃用)
├── feishu-config.json               # (飞书方向失败, 已弃用)
├── .assistant-session-main          # (同上)
├── .assistant-session-classify      # (同上)
└── f9-memory/                       # git repo, 镜像 memory 到 GitHub (可能已弃用, 见下)
```

## 开机启动

注册表 Run 键 (HKCU 用户级, 不需要 admin):
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
  → VoiceInput = wscript.exe C:\Users\tianlan\voice-input\voice-input-hidden.vbs
```

`VoiceAssistant` 和 `VoiceReply` 都**没有**加进 Run 键:
- `VoiceAssistant` 已弃用 (F9 subprocess 方案失败)
- `VoiceReply` 用户明确要求手动开 (只针对某个特定会话朗读)

**Startup 文件夹路径写不进去** (Bash tool 沙箱拒绝访问 + 需要 admin), 所以走注册表这条路。schtasks 也拒。

**VBS 里所有中文注释必须删掉** (wscript 用 GBK 解 UTF-8 中文会崩, 报 800A01A8), 只保留 ASCII。

## voice-reply-watcher 详解

启动: `wscript.exe C:\Users\tianlan\voice-input\voice-reply-hidden.vbs` (静默) 或 `python voice_reply_watcher.py` (可见)。

原理:
1. 启动时扫 `~/.claude/projects/C--Users-tianlan/*.jsonl`, 挑"最近改过 + 够大" (`>5000` 字节, 过滤掉 `-p` 一次性调用产物) 的一个作为**pinned session**
2. **不再自动切换** (`PIN_TO_STARTUP_SESSION = True`) —— 只朗读这一个 session 里的 assistant 回复
3. 每 1 秒 tail 一次这个 jsonl, 抽出新的 `{"type": "assistant", ...}` 消息里的 text 内容
4. `_prepare_for_tts()` 清掉 Markdown 符号 / 代码块 / 链接 / 粗体标记等 (免得 TTS 念"星号"、"井号")
5. 只朗读 `3 <= 长度 <= 500` 字的 (`MIN_SPEAK_CHARS` / `MAX_SPEAK_CHARS`)
6. edge-tts (`zh-CN-XiaoyiNeural`, rate=+5%, pitch=-8Hz, volume=-10%) → mp3 → pygame 播放

**要换朗读的 session**: 杀了重启 (会重新扫最新最大 jsonl)
**要临时关闭朗读**: `taskkill /F /IM python.exe` 里挑那个 voice_reply_watcher 的进程杀
**朗读中断不了**: 现在没做打断机制, 长回复一路念完. 想加需求扩展 `_speak()`

## 核心配置常量 (`voice_input.py` 顶部)

```python
HOTKEY = "caps lock"                # 主热键
CHAT_HOTKEY = "f9"                  # 副热键 (F9 发给 Claude Code)
CHAT_WINDOW_KEYWORDS = ["Claude", "Visual Studio Code"]   # 找目标窗口的关键词
MODEL_SIZE = "medium"               # tiny / base / small / medium / large-v3
LANGUAGE = "zh"                     # 锁定中文
DEVICE = "auto"                     # auto = 后台每 5 秒查 GPU 忙不忙, 忙就切 CPU
COMPUTE_TYPE = "int8"               # auto/cpu 用 int8; 纯 GPU 可用 float16
GPU_BUSY_THRESHOLD = 70             # GPU 占用超过 % 就切 CPU
GPU_POLL_INTERVAL = 5               # 秒
SAMPLE_RATE = 16000
MIN_DURATION = 0.25
PTT_THRESHOLD = 0.25                # 短于此秒数当作普通短按 (Caps Lock 切换)
INITIAL_PROMPT = "..."              # whisper 上下文提示 (专有名词字典)
```

## Caps Lock tap/hold 双用机制 (核心难点)

Caps Lock 平时用来切换大小写。要复用来当 PTT 但**不能破坏切换功能**。做法：
- `suppress=False` (不吃掉物理事件, 让 Windows 自己切大小写)
- 按下开一个定时器 (PTT_THRESHOLD = 250ms), 超时还没松开才判定为 PTT 开始录音
- PTT 结束时 `keyboard.send('caps lock')` 撤销物理按下产生的切换
- 用 `_synth_until` 时间窗忽略自己发出的合成事件, 防止 send caps lock 触发自己的回调造成死循环

## 音频处理管线

录音 (sounddevice 16kHz mono float32)
  → `noisereduce.reduce_noise(non-stationary, prop_decrease=0.75)` (背景噪声抑制)
  → 动态压缩 `sign(x) * |x|^0.6` (小声部分相对提升)
  → 峰值归一化到 0.95
  → whisper `transcribe(beam_size=1, vad_filter=False, without_timestamps=True, initial_prompt=INITIAL_PROMPT)`
  → 输出文字

## GPU/CPU 自动切换 (2026-07-06 关键改动)

启动时**同时加载 CPU 和 GPU 两份模型** (多占约 800MB RAM)。后台线程每 5 秒查 `nvidia-smi --query-gpu=utilization.gpu`, 忙 (>70%) 就 `_gpu_busy = True`。`_get_model()` 按状态返回对应模型, 切换零延迟 (模型都在内存)。

**背景**: 用户用 Unity 时 GPU 被吃满到 98%, whisper 在 GPU 上一次识别要 30 秒才出结果。有了 auto 模式后 Unity 一开就自动降级 CPU (1-2 秒), Unity 关了自动升回 GPU (~200ms)。用户不用改任何配置。

## F9 → Claude Code 一键发送 (2026-07-06 新加)

流程 (在 `_on_f9_press` / `_on_f9_release` / `_transcribe_and_send_to_chat`):
1. F9 按下 → 立刻开录音 (`_start_recording()`) + 后台线程用 ctypes EnumWindows 找目标窗口 (`_find_and_focus_chat_window()`)
2. 找到就 `SetForegroundWindow(hwnd)`, 最小化的先 `ShowWindow(hwnd, SW_RESTORE)`
3. F9 松开 → 停录音 → 转写 → 再确保一次窗口在前台 → 发 `Escape` 清 popover → `Ctrl+V` 粘贴 → `Enter` 发送

**用户约束**: 需要用户**先手动点一下 Claude Code 聊天输入框**让它获得焦点, 之后 F9 才能准确落在输入框。因为 VS Code 里 SetForegroundWindow 只保证窗口在前台, 窗口内部的焦点由 VS Code 自己管 (它会回到"最后聚焦的输入框")。

**如果 F9 粘错地方**: 检查用户是不是切到了别的 tab (代码编辑器/终端), Escape+Ctrl+V+Enter 会落到那里去。

## 依赖 (`requirements.txt`)

```
sounddevice>=0.4.6
numpy>=1.24
faster-whisper>=1.0.0
keyboard>=0.13.5
pyperclip>=1.8.2
edge-tts>=6.1.0      # 弃用但没删
pygame-ce>=2.5.0     # 弃用但没删
```

还有几个 pip install 但没写进 requirements 的:
- `noisereduce` (音频降噪)
- `pycaw` (调过 mic 音量, 用完就没管了)
- `pystray`, `Pillow` (给 assistant 圆点用的, 弃用)

Python 版本: **3.14** (从 Windows Store 装的, 路径在 `C:\Users\tianlan\AppData\Local\Microsoft\WindowsApps\python.exe`)。**这是坑**: 3.14 太新, 很多包没 wheel。当时踩过的坑:
- `pygame` 没 3.14 wheel → 换成 `pygame-ce` (社区版, 有 wheel)
- `ctranslate2` (faster-whisper 依赖) 只有 3.14 wheel, 装得上

## GPU 加速的 DLL 加载

`ctranslate2` 需要 CUDA + cuDNN 的 DLL。用户装的是 pip 版:
```
pip install nvidia-cublas-cu12 nvidia-cudnn-cu12
```
DLL 装在 `.venv/Lib/site-packages/nvidia/cublas/bin/` 等目录, 脚本顶部有一段:
```python
_nv = Path(__file__).parent / ".venv" / "Lib" / "site-packages" / "nvidia"
if _nv.exists():
    _paths = [_nv / "cublas" / "bin", _nv / "cuda_nvrtc" / "bin", _nv / "cudnn" / "bin"]
    os.environ["PATH"] = ";".join(str(p) for p in _paths) + ";" + os.environ.get("PATH", "")
    for _p in _paths:
        if _p.exists():
            os.add_dll_directory(str(_p))
```
`os.add_dll_directory` 只影响 Python 层加载 (让 ctranslate2 找 cudnn 加载模型), 但 CUDA 运行时用另一套机制找 cublas, 所以还要改 `PATH` 环境变量。**两个都必须**。

## 常见问题排查

**问: F9/Caps Lock 按了没反应**
- `tasklist | grep python.exe` 有没有 voice_input.py 进程 (占 700MB+ 的那个是加载好的)
- 没有 → 拉起来: `wscript.exe C:\Users\tianlan\voice-input\voice-input-hidden.vbs`
- 有但没反应 → 可能有多个进程抢热键。`taskkill /F /IM python.exe` 全杀掉重启一个

**问: 识别很慢 (10 秒+)**
- 大概率是 GPU 被别的程序占了 (Unity/游戏等)
- 应该会自动切 CPU (1-2 秒), 如果没切说明 nvidia-smi 报的利用率没超阈值, 或后台监控线程挂了
- 应急方案: 改 `DEVICE = "cpu"` 强制 CPU

**问: F9 粘到别的地方去了**
- 用户忘了先点一下 Claude Code 聊天输入框
- 或者 VS Code 窗口没被前台化 (Windows 有焦点抢占限制)
- 应急: 让用户手动 Alt-Tab 到 Claude Code, 再点一下输入框

**问: 一直有几个 python.exe 进程**
- 用户可能同时打开了多次 voice_input, 全杀重启就好

**问: VBS 报 800A01A8**
- VBS 文件有中文注释 → wscript GBK 解不开。用 ASCII 注释重写

**问: 中文识别错了同音字**
- 加到 `INITIAL_PROMPT` 的词库里, 重启 voice_input 就生效
- 目前收录: skill, scale, agent, prompt, workflow, subagent, 目录, 文件夹, 顶视图, 侧视图, 收银台, 棋盘格, 识别率, 识别
- lycheeGame / 求合体 / 案卷九章 / 凌晨两点的便利店 也在

**问: 想改热键**
- `HOTKEY = "..."` 或 `CHAT_HOTKEY = "..."` 支持 `keyboard` 库的键名, 如 `"f8"`, `"pause"`, `"right shift"`, `"caps lock"`

**问: 想彻底关掉工具**
1. `taskkill /F /IM python.exe`
2. `reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v VoiceInput /f`

## 已弃用但代码/文件还在的东西 (要不要清理让用户定夺)

- `D:\voice-input-data\f9-persona.md` (F9 subprocess 时代的 persona 文件)
- `D:\voice-input-data\assistant-history.json` (F9 subprocess 时代的历史)
- `D:\voice-input-data\feishu-config.json` (**里面有真实的 App Secret**, 已废弃, 建议删)
- `D:\voice-input-data\.assistant-session-main` / `.assistant-session-classify` (F9 subprocess session-id)
- `D:\voice-input-data\f9-memory\` (git repo 镜像 memory 到 GitHub 的备份, 可以留)
- `assets/dot-ring.png` (给 assistant 圆点用的图, gpt-image-2 生成的)
- `assets/voice-samples/` (edge-tts 语音试听样本)

## 走过的弯路 (供未来参考, 别再踩)

1. **飞书 Wiki 集成失败** - 用户飞书是免费个人版, 应用+API 全要付费, 权限申请不通过。凭这条卡了 30 分钟
2. **F9 subprocess 版本失败** - 用 `claude -p` 起 subprocess 让 F9 单独跟 Claude 对话, 但每次都是新会话文件 (爆几十个 jsonl), 也听不懂用户的沉淀指令 (给菜单不干活), 而且识别率跟主 Claude 差不多但缺少这一整天累积的上下文。用户嫌它笨。最后放弃, 改成"F9 直接发到主 Claude 聊天框"
3. **`--session-id` 复用会报 "already in use"** - 因为前一个 subprocess 没释放 lock。加了 fallback 重生成 UUID 的重试。但最终也没走这条路 (F9 subprocess 都弃用了)
4. **Persona 让 Claude 认不出"记一下"沉淀触发** - 反复改 persona 三次都不行, Claude 一直给菜单。最后是用 Python 硬编码沉淀路由。但 F9 subprocess 整个方案都弃用了, 这段代码也没用了
5. **Session 文件爆炸** - `claude -p` 每次一个新 session.jsonl 文件。清理: `find "C:/Users/tianlan/.claude/projects/C--Users-tianlan/" -maxdepth 1 -name "*.jsonl" -size -100k -delete`
6. **手机麦克风增益** - pycaw 只能改 level (100% 已满), Boost 开关归特定驱动管, 用户的 USB 麦没这个选项。转向软件侧 (noisereduce + 动态压缩)
7. **pygame 音频卡顿** - 默认 buffer 太小, 改成 `pre_init(freq=24000, size=-16, channels=1, buffer=2048)` 好了 (但 pygame 也随 assistant 一起弃用了)
8. **Python 3.14 兼容性** - `pygame` 无 wheel 换 `pygame-ce`; 别的都装得上

## 用户偏好 (跟工具相关的)

- 可写数据默认放 D 盘 (`D:\voice-input-data\`) — [feedback-writable-data-on-d-drive]
- edge-tts 中文语音选过一轮, 最终定的是 `zh-CN-XiaoyiNeural` + rate=+5% + pitch=-8Hz + volume=-10% (但 assistant.py 弃用后没用了)
- 圆点 UI 曾经做过 (呼吸圆点 + 星尘 + 状态色 + 字幕吐司), assistant 弃用一起没了

## 最后测试通过的状态 (2026-07-06)

- Caps Lock 长按 → 语音输入到光标 ✓
- F9 按住 → 跳到 Claude Code + 发送 ✓
- voice-reply-watcher → Claude 回复自动朗读 (锁死单会话) ✓
- GPU 自动切换 (Unity 开时切 CPU) ✓
- 识别延迟: GPU 100-300ms / CPU 1-2s
- 朗读延迟: <1s 从 Claude 完成回复到开始念
- 中文识别准度: 大部分对, 遇到冷门字词就漏, 加进 INITIAL_PROMPT 修
