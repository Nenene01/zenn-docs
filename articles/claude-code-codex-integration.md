---
title: "記事執筆を加速！Claude CodeとCodexのSkill連携で校閲・画像生成を自動化"
emoji: "🤝"
type: "tech"
topics: ["claudecode", "openai", "codex", "zenn", "ai"]
published: true
---

## はじめに

Claude Codeで技術記事を書いているとき、こんな悩みはありませんか？

- 「別の視点からも文章をチェックしたい」
- 「記事の図解用にいい感じの画像生成プロンプトが欲しい」
- 「セカンドオピニオンとして他のAIにも見てもらいたい」

本記事では、Claude CodeのSkill機能を使ってOpenAI Codex CLIと連携し、記事執筆ワークフローを強化する方法を紹介します。

:::message
**カスタムスラッシュコマンドはスキルに統合されました**

以前は `.claude/commands/` にカスタムコマンドを配置していましたが、現在は `.claude/skills/` に統合されています。既存の `.claude/commands/` ファイルは引き続き動作しますが、スキルの方が以下の追加機能をサポートするため推奨されます：

- サポートファイル用のディレクトリ構成
- フロントマターによる呼び出し制御
- 関連する場合にClaudeが自動的にスキルをロードする機能

詳細は[公式ドキュメント](https://code.claude.com/docs/ja/skills)を参照してください。
:::

![Claude Code × Codex 連携ワークフロー](/images/claude-code-codex-skill/workflow.png)
*Claude CodeとCodexの連携ワークフロー概要*

## 対象読者

- Claude Codeで技術記事を執筆している方
- AIツールを組み合わせて執筆効率を上げたい方
- Codex CLIに興味がある方

## 前提環境

- Claude Code（インストール済み）
- OpenAI Codex CLI（`npm install -g @openai/codex`でインストール）
- OpenAI APIキーの設定

## なぜCodexと連携するのか

Claude Codeは優秀ですが、1つのAIだけで完結させるより、複数のAIを使い分けた方が良い結果が得られることがあります。

**Claude Codeが得意なこと**
- コードの実装・編集
- ファイル操作・Git操作
- 対話的な作業の進行

**Codex（GPT-4o）が得意なこと**
- 別の視点からの文章チェック
- DALL-E用の画像生成プロンプト作成
- OpenAIエコシステムとの親和性

両者を組み合わせることで、**Claude Codeで執筆→Codexで校閲→Claude Codeで修正**という効率的なワークフローが実現できます。

## MCPではなくSkillを選ぶ理由

Codexとの連携方法として、MCP（Model Context Protocol）を使う方法もあります。しかし、MCPには以下の課題があります：

![MCP vs Skill 比較](/images/claude-code-codex-skill/mcp-vs-skill.png)
*MCPとSkillの比較：Skillの方が可視性・デバッグ性に優れる*

| 項目 | MCP | Skill |
|------|-----|-------|
| 進捗可視性 | ❌ 結果が返るまで見えない | ✅ リアルタイムで表示 |
| デバッグ | ❌ ブラックボックス | ✅ エラー出力が見える |
| 呼び出し方法 | △ プロンプトで指示 | ✅ `/codex` で即呼び出し |
| 長時間タスク | ❌ タイムアウトの不安 | ✅ 進捗を見ながら待てる |

特に記事の校閲のような**長時間タスク**では、Skillの方が圧倒的に体験が良いです。

## スコープの違い：グローバル vs プロジェクト

Codexスキルは**2つのスコープ**で設定できます。用途に応じて使い分けましょう。

### スコープ比較

| スコープ | 配置場所 | 用途 | 特徴 |
|---------|---------|------|------|
| グローバル | `~/.claude/skills/codex/SKILL.md` | 全プロジェクト共通 | 汎用的なコードレビュー・設計相談 |
| プロジェクト | `.claude/skills/codex/SKILL.md` | 特定プロジェクト専用 | プロジェクト固有のタスク（本記事の例） |

:::message
**スキルのディレクトリ構成**

スキルは `SKILL.md` をエントリポイントとするディレクトリです：

```
.claude/skills/codex/
├── SKILL.md           # メイン指示（必須）
├── examples/          # 使用例（オプション）
└── scripts/           # スクリプト（オプション）
```

単一ファイル（`.claude/skills/codex.md`）も動作しますが、ディレクトリ形式の方がサポートファイルを追加できて拡張性があります。
:::

### オーバーライドの仕様

同じ名前のスキルが両方に存在する場合、**プロジェクトスコープが優先**されます。

```
/codex を実行した場合の優先順位:

1. .claude/skills/codex/SKILL.md（プロジェクト）← 優先
2. ~/.claude/skills/codex/SKILL.md（グローバル）
```

:::message
**使い分けの例**

- **グローバル**: 汎用的な「コードレビュー」「設計相談」「バグ調査」
- **プロジェクト**: Zenn記事執筆に特化した「校閲」「画像プロンプト生成」

グローバルに汎用Codexスキルを置いておき、特定プロジェクトでは専用スキルでオーバーライドする運用が便利です。
:::

### グローバルCodexスキルとの関係

汎用的なCodexスキルの設定方法は、別記事で詳しく解説しています：

👉 **[Claude Codeでマルチエージェント環境を構築する](https://zenn.dev/nenene01/articles/claude-code-multi-agent-skills)**

本記事では、**Zenn記事執筆に特化したプロジェクトスコープのスキル**を作成します。

---

## Skill設定の実装（プロジェクトスコープ）

### 1. スキルディレクトリの作成

```bash
mkdir -p .claude/skills/codex
```

### 2. SKILL.mdの作成

`.claude/skills/codex/SKILL.md` を作成します：

```markdown:.claude/skills/codex/SKILL.md
---
name: codex
description: OpenAI Codex CLIを使用して記事の校閲、文章改善、画像生成プロンプト作成を実行する
---

# Codex連携スキル

OpenAI Codex CLIを使用して、Zenn記事の品質向上をサポートします。

## 使用方法

ユーザーの依頼内容に応じて、以下のコマンドをBashツールで実行してください：

\`\`\`bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" "<依頼内容>"
\`\`\`

## パラメータ説明

- `--full-auto`: 完全自動モード（確認なしで実行）
- `--sandbox read-only`: 読み取り専用サンドボックス（ファイルの変更を防止）
- `--cd <dir>`: 対象ディレクトリを指定
- `"<依頼内容>"`: 実行したいタスクの説明
```

### 3. 自動検出について

:::message
**settings.jsonへの登録は不要です**

`.claude/skills/` ディレクトリに配置されたスキルはClaude Codeが自動的に検出します。`settings.json` への明示的な登録は必要ありません。

スキルを配置したら、Claude Codeを再起動（または新しいセッションを開始）するだけで `/codex` コマンドが使えるようになります。
:::

## 実際の使用例

![実際の使用シーン](/images/claude-code-codex-skill/usage-scene.png)
*Claude CodeとCodexを組み合わせた執筆ワークスペース*

### 記事の校閲

```
あなた: /codex この記事を校閲して

Claude Code: Codexで校閲を実行します。
（Codexがターミナル出力でリアルタイムに進捗を表示）

Codexの指摘：
1. 「〜だと思います」が多用されています。断定的な表現に変更を検討してください
2. 第3章と第4章の順序を入れ替えた方が論理的な流れになります
3. 「設定」という言葉が30回以上使われています。類語で置き換えを検討してください

Claude Code: Codexの指摘を元に修正しますか？
```

### 画像生成プロンプトの作成

```
あなた: /codex この記事の内容を図解する画像プロンプトを作って

Claude Code: Codexで画像生成プロンプトを作成します。

Codexの出力：
DALL-E Prompt:
"A clean technical diagram showing the workflow between Claude Code and Codex CLI.
Left side shows Claude Code icon with 'Write & Edit' label, right side shows Codex
icon with 'Review & Generate' label. Arrows connecting them in a circular flow.
Modern flat design, blue and purple color scheme, white background."
```

### 構成のレビュー

```
あなた: /codex 記事の構成をレビューして改善案を出して

Claude Code: Codexで構成レビューを実行します。

Codexの提案：
現在の構成：
1. はじめに
2. 設定方法
3. 使い方
4. まとめ

提案する構成：
1. はじめに（課題提示）
2. なぜこの方法なのか（動機付け）
3. 設定方法（ステップバイステップ）
4. 実践例（具体的なユースケース）
5. トラブルシューティング（よくある問題）
6. まとめ（次のアクション）
```

## 実運用のTips

### 1. 安全な実行

`--sandbox read-only` オプションにより、Codexがファイルを変更することはありません。提案を受けて、実際の編集はClaude Codeで行います。

### 2. 使い分けのコツ

| タスク | 推奨ツール |
|--------|-----------|
| 初稿の執筆 | Claude Code |
| 校閲・レビュー | Codex（/codex） |
| 画像プロンプト作成 | Codex（/codex） |
| 修正の実装 | Claude Code |
| Git操作 | Claude Code |

### 3. コスト意識

Codex CLIはOpenAI APIを使用するため、利用には費用がかかります。頻繁な校閲は避け、ある程度書き上げてからまとめてレビューを依頼するのがおすすめです。

## まとめ

Claude CodeとCodexをSkillで連携することで：

- **マルチAI体制**: 異なるAIの視点でクオリティ向上
- **リアルタイム確認**: 進捗が見えるから安心
- **簡単呼び出し**: `/codex` だけでOK
- **安全性**: read-onlyで意図しない変更を防止

記事執筆において、Claude CodeとCodexは補完関係にあります。ぜひお試しください。

---





## 関連記事

### グローバルスコープでの汎用Codexスキル

本記事はZenn記事執筆に特化したプロジェクトスコープのスキルを紹介しましたが、**全プロジェクトで使える汎用Codexスキル**については以下の記事で解説しています：

👉 **[Claude Codeでマルチエージェント環境を構築する](https://zenn.dev/nenene01/articles/claude-code-multi-agent-skills)**

この記事では、Codexに加えてClaude Codeのサブエージェント（Task tool）も活用したマルチエージェント環境の構築方法を紹介しています。

---

## 参考

- [Claude をスキルで拡張する - 公式ドキュメント](https://code.claude.com/docs/ja/skills)
- [Claude CodeのSkills機能で、独自のスラッシュコマンドを作成する](https://zenn.dev/owayo/articles/63d325934ba0de) - @owayo
- [OpenAI Codex CLI](https://github.com/openai/codex)
