# 草稿推送契约

## 通用

- AppID 由用户环境提供；AppSecret 仅 `WECHAT_APP_SECRET` 或临时 mode-600 `.env`。
- 推完删除临时 `.env`。
- 只创建草稿；验收 Media ID 或草稿 ID。

## 长文

- 输入：润色后主要来源正文 + 封面图。
- 工具：文颜 CLI 草稿模式，或官方 `cgi-bin/draft/add`（保留原 HTML 主题时优先直调）。
- 失败：40164 → IP 白名单；其它错误保留本地包并报告。

## 贴图（type: image）

- frontmatter：`title`、`type: image`、`cover` 指向首图。
- 正文为配套文字；文末按顺序 `![](./images/NN.png)`。
- 工具：`wenyan publish -f post.md --app-id … --env-file …`（core 对 `type:image` 走图片草稿）。
- 交付：标题、完整配套文字、张数、Media ID。
