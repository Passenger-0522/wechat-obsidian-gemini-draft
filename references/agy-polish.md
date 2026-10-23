# agy Gemini 润色

## 预检

```bash
command -v agy
agy models
```

只选本次列出的 Gemini 完整显示名。沿用本机 Antigravity 订阅登录；不读取 token，不复制钥匙串。

## 提示词要点

- 只输出完整 Markdown 正文，不要 frontmatter、前言或整篇代码围栏。
- 保留事实、数字、链接、结构。
- 去 AI 味：无待定／待核实免责、无破折号、少堆句号。
- 文末署名按用户约定保留。

## 调用

```bash
agy -p "$prompt" --model "$model" --dangerously-skip-permissions --print-timeout 5m \
  > candidate.md 2> agy.stderr.log
```

退出非零或 stdout 空／像日志：不写回。事实被改：反馈 Gemini 最多两次修正；编排者不私自改措辞。
