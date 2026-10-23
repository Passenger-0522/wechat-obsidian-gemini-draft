# 我怎么把「Obsidian → Gemini → 公众号草稿箱」收成 Skill

目标 Skill：`wechat-obsidian-gemini-draft`  
流水线：文字写进 Obsidian「主要来源」→ 本机 agy Gemini 润色写回 → 推微信公众号草稿箱（长文或贴图）。只草稿，不群发。

## 1. 为什么要做成 Skill

口述每次都要重复：路径叫法、备份、模型必须是 Gemini、密钥只走 env、贴图用 `type:image`、对用户只说「主要来源」。做成 Skill 后，触发语一句话就能跑完整条，少漏步骤。

## 2. 目录怎么摆

照 Codex / Agent Skill 惯例：

```
wechat-obsidian-gemini-draft/
  SKILL.md              # name + description + 步骤
  README.md             # 人读的安装说明
  LICENSE
  agents/openai.yaml    # Codex 展示名与默认 prompt
  references/
    agy-polish.md       # 润色调用
    draft-push.md       # 长文／贴图推送
    frontmatter.md      # 写回元数据
```

`description` 写「什么时候用」，把口语触发都塞进去，例如「文字进 Obsidian 再 Gemini 润色推草稿」。

## 3. 怎么接到已有能力

- 润色：复用 `obsidian-gemini-polish` 的规则（备份、`agy models` 选 Gemini、验收后再写回）。编排 Skill 不自己改措辞。
- 推送：长文走文颜草稿或官方 `draft/add`；贴图走 wenyan `type: image` + 本地有序图片。
- 密钥：`WECHAT_APP_SECRET` 或临时 mode-600 `.env`，推完删除。

## 4. 本机预检

1. Obsidian 开着，Vault 已加载  
2. `agy models` 能列出 Gemini 显示名  
3. 出口 IP 在公众号白名单（报 40164 就先加白）  
4. 本机有 `@wenyan-md/cli` 或已验证的 draft 脚本  

缺哪项停哪项，不偷偷换 Claude 润色，不装软件改认证。

## 5. 装进 Codex（C 点）

```bash
# 仓库克隆后
ln -sfn "$PWD" ~/.codex/skills/wechat-obsidian-gemini-draft
```

或把目录复制进 `~/.codex/skills/wechat-obsidian-gemini-draft/`。重启／刷新后应能被 Skill 列表扫到。

## 6. 推 GitHub

```bash
gh repo create Passenger-0522/wechat-obsidian-gemini-draft --public --source=. --remote=origin --push
```

不要提交 `.env`、publish 日志里的 secret。

## 7. 端到端验收

- 主要来源有 `polish: gemini`、`model`、`source_revision.sha256`  
- 备份文件哈希与润色前一致  
- 草稿箱出现对应稿，贴图能看到全部图片  
- 回执含标题、Media ID（或草稿 ID）

## 8. 常见失败怎么停

| 情况 | 动作 |
| --- | --- |
| `agy models` 无 Gemini | 停止，保留原稿 |
| 写回前主文件哈希变了 | 停止覆盖，报告冲突 |
| 40164 | 请加白名单，保留本地包 |
| stdout 空或像日志 | 不写回，最多让 Gemini 修两次 |

## 9. 和「只润色」Skill 的分工

`obsidian-gemini-polish` 只管润色写回。  
`wechat-obsidian-gemini-draft` 管端到端：写入主要来源 → 润色 → 推草稿。用户只要润色时，只跑到写回即可。

产出路径：本文件  
大约字数：约 900  
还缺：Codex 另写的过程文档合并进 README、仓库首次 push 的实际 URL 回填。
