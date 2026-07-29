# agent-skills

Claude Code / Claude.ai / Cursor / Codex など、Agent Skills 仕様
（[agentskills.io](https://agentskills.io/)）に対応したツールで使える
スキル集です。

## スキル一覧

- [`skills/self-documenting-code`](./skills/self-documenting-code) —
  コードのコメント・命名・JSDoc/TSDoc/API ドキュメントに関する規約

## インストール方法

### Claude Code（`/plugin install`）

このリポジトリは Claude Code のプラグインマーケットプレイスとして
登録できます（`.claude-plugin/marketplace.json` を参照）。

1. マーケットプレイスを登録する:

   ```
   /plugin marketplace add qaynam/agent-skills
   ```

2. プラグインをインストールする:

   ```
   /plugin install self-documenting-code@qaynam-agent-skills
   ```

   または `Browse and install plugins` → `qaynam-agent-skills` →
   `self-documenting-code` → `Install now` から対話的にインストールも可能です。

### Claude.ai

[Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
の手順に従い、`skills/self-documenting-code` フォルダをそのまま
カスタムスキルとしてアップロードしてください。

### Cursor / Codex など他ツール

各ツールのスキル／ルール読み込みディレクトリに
`skills/self-documenting-code` フォルダをコピー（またはシンボリックリンク）
してください。`SKILL.md` の YAML frontmatter（`name` / `description`）と
本文をそのまま解釈できるツールであれば追加設定なしに利用できます。

## スキルの作り方

各スキルは `SKILL.md`（YAML frontmatter + 指示文）を含むフォルダです。
詳細な参照ドキュメントは `references/` 以下に置き、必要になったときだけ
読み込む構成にしています。
