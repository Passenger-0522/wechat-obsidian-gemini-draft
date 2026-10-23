# wechat-obsidian-gemini-draft

把公众号长文或贴图配套文字写入 Obsidian「主要来源」，用本机 Antigravity（agy）Gemini 润色写回，再推送到微信公众号**草稿箱**。

## 安装到 Codex

```bash
# 克隆后链到 ~/.codex/skills/
ln -sfn "$(pwd)" ~/.codex/skills/wechat-obsidian-gemini-draft
```

或把本目录复制进 `~/.codex/skills/wechat-obsidian-gemini-draft/`。

## 依赖

- Obsidian（Vault 打开）
- `agy`（Antigravity CLI）+ 可用的 Gemini 模型
- `@wenyan-md/cli` 或官方 draft API；公众号 IP 白名单

## 安全

不把 AppSecret 写入仓库。推送用环境变量或一次性 `.env`（mode 600），用完删除。
