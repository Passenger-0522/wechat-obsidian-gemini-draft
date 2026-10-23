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

一条清晰可复用的标准化流水线：`写入 Obsidian 主要来源 → 备份原稿 → 本机 agy Gemini 润色写回 → 组包推送至公众号草稿箱`。

- 文本润色必须交由本机 `agy` 托管的 Gemini 执行，编排程序不得使用其他模型改写正文。
- 所有内容仅推送到微信公众号草稿箱，由用户在后台自行审核并发布。
- AppSecret 与 Token 仅允许通过环境变量或临时 `mode-600` 的 `.env` 读取，推送完成后立即清除，严禁写入对话记录、代码仓库、Skill 正文或执行日志。

## 何时用

| 模式 | 触发场景 | 交付产物 |
| --- | --- | --- |
| A 长文 | 用户提供正文或已有文稿，需录入 Obsidian 并完成润色后推送长文草稿 | `00-母稿.md` + 图文草稿 |
| B 贴图 | 用户提供卡片图及配套文字，需完成文案润色后推送图片消息草稿 | `01-贴图文案.md` + `type:image` 草稿 |

若用户仅要求「润色」而无需推送，流程在 Gemini 写回主要来源后直接结束，跳过后续推送环节。

## 依赖

- Obsidian 处于运行状态，且已加载目标 Vault。
- 本机已安装 `agy`（Antigravity CLI），执行 `agy models` 可正常列出 Gemini 的完整显示名称。
- 推送环境：本机安装 `@wenyan-md/cli`，或配置已验证的官方 `draft/add` 脚本，且当前出口 IP 已配置公众号白名单。
- 可选组件：`$obsidian-gemini-polish` 专职处理润色子步骤；本 Skill 负责端到端统筹编排与最终推送。

预检阶段若发现依赖缺失或校验失败，立即终止运行并告知缺口，绝不擅自更改认证配置、安装外部软件或切换模型供应商。

## 步骤

### 1. 定位主要来源

- [ ] 优先使用用户指定的文件路径；未指定时，在 Vault 内容草稿目录下为当前选题新建或定位目录（如 `30-内容/03-草稿/{slug}/`）。
- [ ] 长文主文件默认使用 `00-母稿.md`，贴图文案默认使用 `01-贴图文案.md`。与用户沟通时统一称为「主要来源」，避免使用内部术语。
- [ ] 写入或更新正文时完整保留原有 Frontmatter；新建文件时补全 `content_stage: source` 与 `status: draft`。

### 2. 备份并 Gemini 润色

- [ ] 按照 `$obsidian-gemini-polish`（或同级 `references/agy-polish.md`）备份为 `*-YYYYMMDD-HHMMSS-pre-gemini.md`，并核验 SHA-256。
- [ ] 依据 `agy models` 选用实际列出的 Gemini 显示名（例如 `Gemini 3.8 Flash (High)`）。无头调用时按环境需要补充 `--dangerously-skip-permissions`。
- [ ] 提示词仅传入正文内容：严格保留客观事实与结构框架；清除生硬的 AI 腔调（不添加未经证实的推测、不用破折号、避免连续堆砌短句）；署名保持用户约定文末署名，不擅自修改。
- [ ] 验收 stdout 确认为完整 Markdown 正文后再写回文件；同步合并更新 `polish: gemini`、`polished_at`、`model`、`source_revision`。编排程序不得二次改动润色后的措辞。

### 3. 组包推草稿箱

**长文**

- [ ] 基于已验收的正文生成可推送内容（支持文颜排版主题或原生 HTML 直调，具体以用户当前约定为准）。
- [ ] 按照用户约定的模板准备封面图；若缺少封面，先向用户确认或调用既有封面 Skill，不得凭空捏造。
- [ ] 调用 `draft/add` 接口或使用文颜草稿模式提交，并记录返回的 media_id 与草稿回执。

**贴图（图片消息）**

- [ ] 从主要来源提取推荐标题与配套文字；Frontmatter 声明 `type: image`，并在正文末尾按既定顺序嵌入本地图片。
- [ ] 调用 wenyan 的 `publish` 命令（启用 `type:image` 或 `image_list` 参数）或等价的官方图片草稿接口提交。
- [ ] 推送成功后，在对话中完整交付：发布标题、完整配套文字、图片张数与草稿 Media ID。

### 4. 回执

- [ ] 规范汇报：主要来源路径、备份文件路径、Gemini 模型显示名、草稿 Media ID、执行模式（A 或 B）。
- [ ] 流程若执行失败，务必完整保留原稿与备份文件，严禁声称已完成润色或已进入草稿箱。

## 硬规则

1. 严格限制仅推送至草稿箱，禁止调用群发接口或 `freepublish`。
2. 润色必须选用当前 `agy models` 实际返回的 Gemini 模型；若列表无可用 Gemini 则立即停止。
3. 凭据密钥禁止在本地持久化留存；临时创建的 `.env` 必须用后即删。
4. 推送出口 IP 必须处于公众号后台白名单内；遇到 40164 报错时主动提示用户添加，严禁违规绕过。
5. 面向用户统一使用「主要来源」规范称谓，禁止使用「Obsidian 母稿」等内部叫法。

## 参考

- [references/agy-polish.md](references/agy-polish.md)：润色调用与失败处理规范
- [references/draft-push.md](references/draft-push.md)：长文与贴图推送接口规范
- [references/frontmatter.md](references/frontmatter.md)：写回元数据定义
