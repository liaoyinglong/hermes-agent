# 合并 upstream tag `v2026.4.30` 变更摘要

> 合并提交：`1af2174ed Merge tag 'v2026.4.30'`
> 上游版本：**Hermes Agent v0.12.0 (v2026.4.30)** —— "The Curator" 发布
> 距上一版 v0.11.0：**1,096 commits / 550 PR / 1,270 文件改动 / +217,776 行 / 213 名社区贡献者**
> 本次本地合并影响：**2,065 文件改动 / +472,553 / -32,495**

---

## 一、冲突解决

仅一个文件存在合并冲突：

- `tests/gateway/test_feishu.py`
  - 上游为 [gateway/platforms/feishu.py](../gateway/platforms/feishu.py) 新增了三个事件处理器：
    `register_p2_im_chat_access_event_bot_p2p_chat_entered_v1`、
    `register_p2_im_message_recalled_v1`、
    `register_p2_customized_event("drive.notice.comment_add_v1", ...)`
  - 因合并后 `feishu.py` 已经在调用这些 handler，所以测试侧采纳上游版本，
    在 `_Builder` mock 与 `calls` 期望列表中加入 `p2p_chat_entered`、`message_recalled`、
    `customized:drive.notice.comment_add_v1` 三项。

---

## 二、本次发布主线 —— 自治 Curator（自我维护）

- **`hermes curator` 后台代理**：挂在 gateway 的 cron 上，默认 7 天周期自动评分、合并、清理 skill 库；每次产出 `logs/curator/run.json` + `REPORT.md`。
- **归档分类**：归档的 skill 用模型 + 启发式区分 *已合并 vs 已淘汰*。
- **统一到 `auxiliary.curator`**：在 `hermes model` 选模型，仪表盘可配置。
- **`hermes curator status`**：按使用频次排序，展示最常用 / 最少用 skill。
- **防御**：bundled / hub skill 受保护；pinned skill 拒写。

### 自我改进循环（review fork）大幅升级
- 改为 **rubric/类目优先** 评分（不再自由发挥）。
- **active-update 偏好**：优先更新当前轮刚加载的 skill，且支持 `references/`、`templates/` 子文件。
- 子进程**继承父进程实时 runtime**（provider / model / 凭证真正传过去）。
- 工具集**收紧到 memory + skills**，避免乱发散。
- memory provider 干净退出；review summary 排除前一轮 tool 消息，给到干净上下文。

---

## 三、新接入推理 Provider

| Provider | 说明 |
| --- | --- |
| **LM Studio** | 从 custom-endpoint 别名升级为一等 provider，含 doctor 检查、reasoning 透传、`/models` 实时列表 |
| **GMI Cloud** | 一等 API-Key provider |
| **Azure AI Foundry** | 含自动检测 |
| **MiniMax OAuth** | PKCE 浏览器登录流 |
| **Tencent Tokenhub** | 新增腾讯 Tokenhub 接入 |

模型目录：
- 引入**远程 model catalog manifest**：OpenRouter、Nous Portal 模型清单走远程，新模型免发版即可用。
- 新增 `openai/gpt-5.5`、`gpt-5.5-pro`、`deepseek-v4-pro/flash`、`qwen3.6-plus`。
- `prompt_caching.cache_ttl` 可配（默认 5m，可改 1h）。

---

## 四、新增 / 升级的消息平台（Gateway）

- **Microsoft Teams**（第 19 个平台）—— 通过新的**插件式 platform** 机制接入；网关现在变成 platform plugin host。
- **腾讯元宝 Yuanbao**（第 18 个平台）—— 原生适配器，含文本 + 媒体下发。
- **媒体一致化**：Telegram / Discord / Slack / Mattermost / Email / Signal 多图原生发送；统一音频路由 + FLAC + Telegram 文档兜底。
- **Telegram**：群组 / 论坛白名单、stale 流的 fresh finals、markdown 表格 → 行式 bullets。
- **Discord**：opt-in toolset + ID 注入 + 工具拆分 + Feishu 联通。
- **Slack**：所有 gateway 命令注册成原生 slash；`strict_mention` 防线程误触。
- **飞书 Feishu**：新增 P2P 进入会话、消息撤回、`drive.notice.comment_add_v1` 自定义事件 → 这也是本次 *唯一冲突文件* 的源头。

---

## 五、原生集成

- **Spotify**：7 个原生工具（播放、搜索、队列、歌单、设备）+ PKCE OAuth + 交互式 setup wizard + 内置 skill。
- **Google Meet 插件**：加入会议、转写、发声、跟进；OpenAI Realtime + Node bot server。
- **Vercel Sandbox**：新增为 `execute_code` / terminal 后端。

---

## 六、TUI（Ink + JSON-RPC 网关）

合并新增了完整的 `ui-tui/`、`tui_gateway/` 以及浏览器版仪表盘 `web/`。要点：

- **冷启动 -57%**：lazy agent init、OpenAI/Anthropic/Firecrawl 延迟导入、`load_config` 按 mtime 缓存、`get_tool_definitions` + `check_fn` TTL 缓存、危险命令正则预编译。
- **TUI 追平并超越经典 CLI**：LaTeX 渲染、`/reload` 热加载 .env、可插拔忙碌指示器、可选自动恢复上轮会话、`/resume` 中按 `d` 删除会话、滚轮线性滚动、`/mouse` 关掉 ConPTY 假鼠标注入。
- **Models 仪表盘 Tab + 浏览器内模型配置**：每模型分析、主/辅模型在 dashboard 切换。
- **Web Dashboard `/chat`**：嵌入真实 `hermes --tui`（PTY bridge + xterm.js），不再独立实现聊天。

---

## 七、Skills 生态

**晋升为内置 / 默认绑定：**
- **ComfyUI v5**（CLI + REST + 硬件门控本地安装）—— 从 optional 提升为 built-in。
- **TouchDesigner-MCP**（含 GLSL / postFX / audio / geometry / 9 篇 reference）—— bundled by default。
- **Humanizer**（去 AI 腔）。
- **claude-design**（HTML artifact）、**design-md**（Google DESIGN.md）、**airtable**、**pretext**、**spike**、**sketch**。

**Skill UX：**
- `hermes skills install <url>` 直接装 HTTP(S) 链接的 skill。
- `/reload-skills` 斜杠命令；`hermes skills list` 显示启停状态；`skill_manage` 可编辑 `external_dirs` 内 skill 并拒写 pinned。
- 每个 bundled / optional skill 都生成独立文档页。

---

## 八、Agent / 内核

- **多模态图像路由**：按模型实际视觉能力路由，而非按 provider 默认。
- **delegate `child_timeout_seconds` 默认 600s**；子代理零 API 调用超时时打诊断 dump。
- 修复 `tool_calls` 中 CamelCase / `_tool` 后缀、未转义控制字符、`json.JSONDecodeError` 走重试而非本地错误。
- `_copy_reasoning_content_for_api` 顺序修复（多 provider reasoning 隔离）；DeepSeek/Kimi `tool_calls` 注入空 `reasoning_content`；持久化流式 `reasoning_content`。
- 重命名 `[SYSTEM:` → `[IMPORTANT:`，避开 Azure 内容过滤。
- **Compression**：未知错误时回主模型重试 summary；aux 失败但 main 兜底成功也通知用户。
- **Session/FTS5**：新增三元 trigram FTS5 索引（CJK 检索可用）；索引 `tool_name` + `tool_calls`，含修复 + 迁移。
- **Checkpoints**：启动自动清理孤儿 / 失效 shadow repo。

---

## 九、安全 & 隐私

- **机密红化默认关闭**（`redaction.enabled: false`）—— 避免假阳性 secret 模式破坏 patch 输出，长期顽疾。
- 需要时显式启用。

---

## 十、CLI / DX

- **`hermes -z <prompt>`** 一次性非交互模式（`--model` / `--provider` / `HERMES_INFERENCE_MODEL`）。
- **`hermes update --check`** 升级前 preflight；可选升级前自动备份 `HERMES_HOME`。
- **TTS provider registry + Piper** 本地 TTS 内置。
- **Langfuse 可观测性插件 + hermes-achievements 插件** 内置。

---

## 十一、对本仓库 fork 的提醒

合并带入了大量新顶层目录，新做改动时需注意：

- `tui_gateway/`、`ui-tui/`、`web/`：TUI 与 Web Dashboard 的真正实现；新聊天功能优先扩 Ink，**不要**在 React 里再造一遍主聊天界面。
- `optional-skills/`：重 / 小众 skill 的存放处，不要再塞进 `skills/`。
- `tools/feishu_doc_tool.py`、`tools/feishu_drive_tool.py`、`tools/yuanbao_tools.py`、`tools/discord_tool.py`、`tools/browser_supervisor.py`、`tools/browser_cdp_tool.py`、`tools/path_security.py`、`tools/schema_sanitizer.py` 等为新增工具，开发新工具前先确认无重复。
- 飞书新事件 handler 已经在 `gateway/platforms/feishu.py` 中接入，本仓库后续再加飞书事件应一并加测试断言（参考本次冲突的解法）。

> 详情见仓库根目录的 `RELEASE_v0.12.0.md`（上游随本次合并一并带入）。
