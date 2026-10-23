---
name: wechat-obsidian-gemini-draft
description: >-
  把公众号长文或贴图配套文字写入 Obsidian「主要来源」，用本机 Antigravity CLI（agy）的 Gemini
  润色写回，再推送到微信公众号草稿箱（长文 draft/add 或贴图 type:image）。用于
  「文字进 Obsidian 再 Gemini 润色推草稿」「贴图文案润色后进草稿箱」「Obsidian→Gemini→草稿箱」
  「用这条流水线发公众号草稿」等请求。只推草稿，不群发。
---

# Obsidian → Gemini → 公众号草稿箱

## 目标

一条可复用流水线：`写进 Obsidian 主要来源 → 备份 → agy Gemini 润色写回 → 组包推公众号草稿箱`。

- 润色必须交给本机 `agy` 的 Gemini，编排者不得用其它模型改写正文。
- 推送只进草稿箱；用户自己在公众号后台发布。
- AppSecret / token 只走环境变量或临时 mode-600 `.env`，推完删除；不进聊天、仓库、Skill 正文或日志。

## 何时用

| 模式 | 触发 | 产物 |
| --- | --- | --- |
| A 长文 | 用户给出正文或已有稿，要进 Obsidian 并润色后推长文草稿 | `00-母稿.md` + 图文草稿 |
| B 贴图 | 用户给出贴图卡 + 配套文字，要润色文案后推图片消息草稿 | `01-贴图文案.md` + `type:image` 草稿 |

用户只说「润色」且不要推草稿时，只跑到 Gemini 写回，跳过推送。

## 依赖

- Obsidian 已打开，目标 Vault 已加载。
- `agy`（Antigravity CLI）可用；`agy models` 能列出 Gemini 完整显示名。
- 推送：本机 `@wenyan-md/cli`，或已验证的官方 `draft/add` 脚本；白名单出口 IP。
- 可选：`$obsidian-gemini-polish` 专责润色子步骤；本 Skill 负责端到端编排与推送。

预检失败就停，报告缺口，不自动改认证、不装软件、不换模型厂商。

## 步骤

### 1. 定位主要来源

- [ ] 用户指定路径优先；否则在 Vault 内容草稿目录下为本次选题建或打开文件夹（如 `30-内容/03-草稿/{slug}/`）。
- [ ] 长文主文件默认 `00-母稿.md`；贴图文案默认 `01-贴图文案.md`。对用户只称「主要来源」，不说内部母稿目录黑话。
- [ ] 写入或更新正文时保留既有 frontmatter；新建时带 `content_stage: source`、`status: draft`。

### 2. 备份并 Gemini 润色

- [ ] 按 `$obsidian-gemini-polish`（或同级 `references/agy-polish.md`）备份为 `*-YYYYMMDD-HHMMSS-pre-gemini.md`，核对 SHA-256。
- [ ] `agy models` 选实际列出的 Gemini 显示名（如 `Gemini 3.8 Flash (High)`）。无头调用按本机需要加 `--dangerously-skip-permissions`。
- [ ] 提示词只传正文：保留事实与结构；去 AI 味（无「待定／待核实」免责、无破折号、少堆句号）；署名保持用户约定文末署名，不擅自改名。
- [ ] 验收 stdout 为完整 Markdown 正文后再写回；合并 `polish: gemini`、`polished_at`、`model`、`source_revision`。Codex／编排者不得自行改措辞。

### 3. 组包推草稿箱

**长文**

- [ ] 用已验收正文生成可推内容（文颜主题或绕开文颜的直调 HTML，以用户当前约定为准）。
- [ ] 封面按用户约定模板准备；缺封面先问或按既有封面 Skill，不虚构。
- [ ] 推 `draft/add` 或文颜草稿模式；记录 media_id／草稿回执。

**贴图（图片消息）**

- [ ] 从主要来源取推荐标题与「配套文字」；frontmatter 设 `type: image`，正文后按顺序嵌入本地图片。
- [ ] 用 wenyan `publish`（支持 `type:image` / `image_list`）或等价官方图片草稿接口。
- [ ] 推送成功后，在对话里交付：发布标题、完整配套文字、图片张数与草稿 Media ID。

### 4. 回执

- [ ] 报告：主要来源路径、备份路径、Gemini 模型显示名、草稿 Media ID、模式 A/B。
- [ ] 失败保留原稿与备份；不声称已润色或已进草稿箱。

## 硬规则

1. 只推草稿，禁止群发／`freepublish`。
2. 润色模型必须是本次 `agy models` 中的 Gemini；列表无 Gemini 则停止。
3. 密钥不落盘持久化；临时 `.env` 用完即删。
4. 推送出口须在公众号 IP 白名单内；40164 时请用户加白名单，不绕过。
5. 对用户说「主要来源」，不说「Obsidian 母稿」等内部叫法。

## 参考

- [references/agy-polish.md](references/agy-polish.md) — 润色调用与失败处理
- [references/draft-push.md](references/draft-push.md) — 长文／贴图推送契约
- [references/frontmatter.md](references/frontmatter.md) — 写回元数据
