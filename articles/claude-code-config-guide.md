---
title: "Claude Code設定完全ガイド - 5つの設定機能の使い分け"
emoji: "⚙️"
type: "tech"
topics: ["claudecode", "ai", "初心者向け"]
published: true
---

# はじめに

Claude Code（クロード・コード）は、AIがあなたのコーディングを手伝ってくれる便利なツールです。しかし、「設定ファイルが多くてよくわからない...」と感じている方も多いのではないでしょうか？

この記事では、Claude Codeの5つの設定機能（**CLAUDE.md**、**Rules**、**Skills**、**Hooks**、**Subagents**）について、それぞれの役割と使い分けを解説します。

:::message
**この記事の対象読者**
- Claude Codeを使い始めたばかりの方
- 設定ファイルの違いがよくわからない方
- もっと効率的にClaude Codeを使いたい方
:::

![Claude Code設定ファイルの全体概要](/images/claude-code-config-guide/overview.png)
*Claude Codeの設定ファイルとその関係*

---

# CLAUDE.md - プロジェクト全体の設定

## CLAUDE.mdとは？

**CLAUDE.md**は、Claudeに「このプロジェクトで何を守るべきか」を伝える**全体設定ファイル**です。

:::message
**専門用語解説：CLAUDE.md**
プロジェクトのルートディレクトリに置くMarkdownファイル。Claudeが常に参照する「全体設定」として機能します。
:::

## 使い方

```markdown:CLAUDE.md
# プロジェクト設定

## このプロジェクトについて
ECサイトのバックエンドAPIを開発しています。

## 使用技術
- 言語：TypeScript
- フレームワーク：Express.js
- データベース：PostgreSQL

## コーディング規約
- 変数名はキャメルケース（例：userName）を使用
- 関数には必ずJSDocコメントを記載
- エラーハンドリングは必ず行う

## やってはいけないこと
- console.logをプロダクションコードに残さない
- anyの使用は禁止
- 環境変数をハードコードしない
```

## ポイント

| 設定項目 | 説明 | 例 |
|---------|------|-----|
| アイデンティティ | どのような開発者として振る舞うか | TypeScript開発者として振る舞う |
| 目的 | プロジェクトの目標 | APIの実装・バグ修正 |
| 禁止事項 | やってはいけないこと | console.logを残さない |
| スタイル | コーディング規約 | キャメルケースを使用 |

---

# Rules - パス別の詳細ルール

## Rulesとは？

**Rules**は、特定の条件下でのみ適用される**パス別ルール**です。

:::message
**専門用語解説：Rules**
`.claude/rules/`ディレクトリに配置するMarkdownファイル。パス（ファイルの場所）に応じて自動的に読み込まれ、コンテキスト（状況）に追加されます。
:::

## 使い方

```yaml:.claude/rules/frontend.md
---
path: src/frontend/**
---

# フロントエンド開発ルール

## 使用技術
- React 18 + TypeScript
- Tailwind CSS

## コンポーネント作成ルール
- 関数コンポーネントのみ使用
- propsには必ず型定義をつける
- CSSはTailwindのユーティリティクラスを使用
```

```yaml:.claude/rules/backend.md
---
path: src/backend/**
---

# バックエンド開発ルール

## 使用技術
- Node.js + Express
- Prisma ORM

## APIルール
- RESTful設計に従う
- エラーレスポンスは統一フォーマットで返す
- 認証が必要なエンドポイントにはミドルウェアを適用
```

## パスによる自動適用

Rulesの強力な機能は、**ファイルパスに基づいて自動的に適用される**点です：

```
プロジェクト/
├── src/
│   ├── frontend/    ← frontend.mdのルールが適用
│   │   └── App.tsx
│   └── backend/     ← backend.mdのルールが適用
│       └── server.ts
└── .claude/
    └── rules/
        ├── frontend.md
        └── backend.md
```

### settings.jsonでのパス設定

`.claude/settings.json`を使うと、ルールファイルの適用パスを**明示的に指定**できます：

```json:.claude/settings.json
{
  "rules": {
    "paths": {
      ".claude/rules/frontend.md": ["src/frontend/**/*", "src/components/**/*"],
      ".claude/rules/backend.md": ["src/api/**/*", "src/server/**/*"],
      ".claude/rules/testing.md": ["**/*.test.ts", "**/*.spec.ts"]
    }
  }
}
```

:::message
**パス設定の使い分け**

| 方法 | 説明 | 用途 |
|------|------|------|
| ファイル名による自動適用 | `frontend.md`が`frontend/`に自動適用 | シンプルなパスマッチ |
| settings.jsonでの明示指定 | 複数パスや複雑なパターンに適用 | 柔軟なパス指定 |

**glob パターンの例：**
- `src/**/*` - srcディレクトリ以下すべて
- `**/*.tsx` - 全ディレクトリの.tsxファイル
- `src/{api,server}/**/*` - api または server ディレクトリ
:::

## CLAUDE.mdとRulesの違い

| 項目 | CLAUDE.md | Rules |
|-----|-----------|-------|
| 適用範囲 | 常に全体に適用 | 条件に応じて適用 |
| 配置場所 | プロジェクトルート | `.claude/rules/` |
| 用途 | 全体方針・禁止事項 | 状況別の詳細ルール |

:::message
**コンテキストとメモリについて**
- **コンテキスト**：Claudeが今の会話で参照できる情報の全体
- **メモリ**：CLAUDE.mdやRulesに書かれた永続的な設定

Rulesを使うと、必要な時だけコンテキストにルールを追加できるため、効率的です。毎回プロンプトで同じ指示を書く手間が省けます。
:::

---

# Skills - 専門能力・スラッシュコマンド

![Skillsの概念図](/images/claude-code-config-guide/skills.png)
*Skillsはパワーアップのようにクロードに特殊能力を与える*

## Skillsとは？

**Skills**は、Claudeに**専門的な能力**を持たせるための設定です。特定のタスクに対して定型的な手順や制約を与えることで、一貫した動作を実現します。

:::message
**専門用語解説：Skills**
`.claude/skills/`ディレクトリに配置するMarkdownファイル。`SKILL.md`という名前で作成し、Claudeに特定のタスクの実行方法を教えます。
:::

## スキル定義

### スキルの3層アーキテクチャ

スキルは**ディレクトリ形式**で作成し、以下の3層構造を持つことができます：

```
.claude/skills/commit-helper/
├── SKILL.md           # メイン指示（必須）- エントリポイント
├── examples/          # 使用例（オプション）
│   ├── good.md        # 良い例
│   └── bad.md         # 悪い例
├── resources/         # 参考資料（オプション）
│   └── conventions.md # コミット規約など
└── scripts/           # スクリプト（オプション）
    └── validate.sh    # 検証スクリプト
```

:::message
**3層アーキテクチャのメリット**

| 層 | 役割 | 用途 |
|----|------|------|
| **examples/** | 使用例・サンプル | 良い例・悪い例を提示 |
| **resources/** | 参考資料・ドキュメント | 規約や仕様書を参照 |
| **scripts/** | 補助スクリプト | 検証や前処理を自動化 |

単一ファイル（`.claude/skills/commit-helper.md`）でも動作しますが、ディレクトリ形式の方が拡張性があります。
:::

### 基本構造

````markdown:.claude/skills/commit-helper/SKILL.md
---
name: commit-helper
description: Gitのコミットメッセージを生成します。コミットやGitについて聞かれた時に使用します。
---

# コミットメッセージ生成スキル

## 手順
1. `Git diff --staged`で変更内容を確認
2. 変更の種類を判断（feat/fix/docs/refactorなど）
3. 50文字以内の要約行を作成
4. 必要に応じて詳細な説明を追加

## フォーマット
```
<種類>: <要約>

<詳細な説明（任意）>
```

## 例
```
feat: ユーザー登録APIを追加

- POST /api/users エンドポイントを実装
- バリデーション機能を追加
- エラーハンドリングを実装
```
````

### スキルの呼び出し方法

Claude Codeでは、スキルを呼び出す方法が**2つ**あります：

#### 1. 自動発動（Claudeが判断）
Claudeは、あなたの質問やリクエストを見て、適切なスキルを**自動的に選んで発動**します。

```
ユーザー：「このコミットメッセージを書いて」
Claude：「commit-helperスキルを使用してもよいですか？」
```

#### 2. スラッシュコマンドで明示的に呼び出し
```
/commit-helper
```

:::message
**カスタムスラッシュコマンドはスキルに統合されました**

以前は`.claude/commands/`にコマンドを配置していましたが、現在は`.claude/skills/`に統合されています。スキルを作成すると、自動的にスラッシュコマンド（`/スキル名`）として呼び出せるようになります。

既存の`.claude/commands/`ファイルは引き続き動作しますが、スキルの方が推奨されます。
:::

## スキルの高度な機能

### フォーク実行（コンテキストの分離）

:::message
**専門用語解説：フォーク（fork）**
「分岐」という意味。メインの会話から分離した別の環境でスキルを実行することです。
:::

```yaml:.claude/skills/code-review/SKILL.md
---
name: code-review
description: コードレビューを実行します
context: fork
agent: Explore
---

# コードレビュースキル

## 実行内容
- コード品質のチェック
- セキュリティ問題の検出
- パフォーマンス改善点の提案
```

**フォーク実行のメリット：**
- **セッション履歴が汚れない**：レビュー結果だけがメインの会話に返される
- **コンテキストの独立**：他の作業に影響を与えない
- **結果だけを確認**：詳細な過程は隠され、結論だけが報告される

### ツール制限

特定のツールだけを使えるように制限できます：

```yaml:.claude/skills/safe-reader/SKILL.md
---
name: safe-reader
description: ファイルを安全に読み取ります（変更は行いません）
allowed-tools: Read, Grep, Glob
---

# 安全な読み取りスキル

このスキルはファイルの読み取りのみを行います。
編集や削除は一切行いません。
```

### Codexとの連携（シンボリックリンク）

:::message
**専門用語解説：シンボリックリンク**
ファイルやフォルダへの「ショートカット」のようなもの。実体は別の場所にあるが、あたかもそこにあるかのようにアクセスできます。
:::

複数のプロジェクトでスキルを共有したい場合：

```bash
# 共通スキルを格納するディレクトリ
mkdir -p ~/.claude/shared-skills/

# 各プロジェクトからシンボリックリンクを作成
ln -s ~/.claude/shared-skills/commit-helper .claude/skills/commit-helper
```

これにより、一度作成したスキルを複数のプロジェクトで再利用できます。

### スキルの配置場所と優先順位

| 配置場所 | パス | 適用範囲 | 優先順位 |
|---------|------|---------|---------|
| プロジェクト | `.claude/skills/` | このプロジェクトのみ | 高（優先） |
| グローバル | `~/.claude/skills/` | すべてのプロジェクト | 低 |

:::message
**オーバーライドの仕様**
同名のスキルが両方に存在する場合、**プロジェクト > グローバル**の順で適用されます。これにより、グローバルに汎用スキルを置きつつ、特定プロジェクトでは専用スキルで上書きできます。
:::

## スキルの使い分け

| スキル種別 | 用途 | 例 |
|-----------|------|-----|
| 生成スキル | コードやドキュメントを生成 | コミットメッセージ、コンポーネント作成 |
| 検査スキル | コードの問題を検出 | レビュー、脆弱性スキャン |
| 読み取り専用スキル | 安全な調査・探索 | コードベース調査、依存関係確認 |
| 実行スキル | ビルドやデプロイを実行 | テスト実行、本番デプロイ |

---

# スラッシュコマンド

## スラッシュコマンドとは？

**スラッシュコマンド**は、スキルを**ワンボタンで実行**できるようにするショートカットです。

:::message
**カスタムスラッシュコマンドはスキルに統合されました**

以前は`.claude/commands/`ディレクトリに配置していましたが、現在は**スキル（`.claude/skills/`）に統合**されています。スキルを作成すると、自動的に`/スキル名`で呼び出せるようになります。

詳細は[公式ドキュメント](https://code.claude.com/docs/ja/skills)を参照してください。
:::

## 使い方

スキルを作成すると、自動的にスラッシュコマンドとして呼び出せます。

### 単純なスキル

````markdown:.claude/skills/test/SKILL.md
---
name: test
description: プロジェクトのテストを実行します
---

# テスト実行スキル

プロジェクトのテストを実行してください。

```bash
npm test
```
````

使い方：`/test` と入力するだけ

### 引数付きスキル

````markdown:.claude/skills/create-component/SKILL.md
---
name: create-component
description: Reactコンポーネントを作成します
---

# コンポーネント作成スキル

$ARGUMENTSという名前のReactコンポーネントを作成してください。

## 要件
- TypeScriptで作成
- 関数コンポーネント
- propsの型定義を含める
````

使い方：`/create-component Button` → 「Button」コンポーネントを作成

### 特殊条件付きスキル（自動発動の禁止）

````markdown:.claude/skills/deploy/SKILL.md
---
name: deploy
description: 本番環境にデプロイします
disable-model-invocation: true
---

# 本番デプロイスキル

## 前提条件の確認
1. mainブランチにいること
2. すべてのテストが通っていること
3. 未コミットの変更がないこと

## 実行手順
前提条件を確認した上で、デプロイを実行してください。

```bash
npm run build && npm run deploy
```
````

使い方：`/deploy`（条件を満たしていれば実行）

:::message
`disable-model-invocation: true`を設定すると、Claudeが自動的にこのスキルを発動することを防ぎ、ユーザーが明示的に`/deploy`で呼び出した時のみ実行されます。
:::

## スキルの呼び出しパターン

| パターン | 説明 | 用途 |
|---------|------|------|
| `/スキル名` | ユーザーが明示的に呼び出し | 任意のタイミングで実行 |
| 自動発動 | Claudeが判断して呼び出し | 関連タスクで自動適用 |
| `disable-model-invocation: true` | 明示的呼び出しのみ | 危険な操作の誤発動を防止 |

:::message
詳しくは「[スラッシュコマンド](#スラッシュコマンド)」セクションを参照してください。
:::

---

# Hooks - 自動実行フック

![Hooksのタイミング図](/images/claude-code-config-guide/hooks-timeline.png)
*Hooksは特定タイミングで自動発動するイベントハンドラ*

## Hooksとは？

**Hooks**は、特定のイベントが発生した時に**自動的に実行**される処理です。

:::message
**専門用語解説：Hooks（フック）**
「引っ掛ける」という意味。特定のタイミングで自動的に処理を「引っ掛けて」実行する仕組みです。
:::

## フック設定

フックは`.claude/settings.json`で設定します：

```json:.claude/settings.json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo '⚠️ Bashコマンドを実行します'"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint --fix"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '✅ タスク完了'"
          }
        ]
      }
    ]
  }
}
```

## フックのタイミング

| フック名 | 発動タイミング | 用途 |
|---------|--------------|------|
| PreToolUse | ツール実行**前** | 事前チェック、確認ログ |
| PostToolUse | ツール実行**後** | フォーマット、lint |
| Stop | タスク完了時 | 完了通知、状態確認 |

## 実用例

### タスク完了時の通知

```json
{
  "Stop": [
    {
      "hooks": [
        {
          "type": "command",
          "command": "git status && echo '✅ タスク完了！変更を確認してください'"
        }
      ]
    }
  ]
}
```

### ファイル編集後の自動フォーマット

```json
{
  "PostToolUse": [
    {
      "matcher": "Edit",
      "hooks": [
        {
          "type": "command",
          "command": "npm run format"
        }
      ]
    }
  ]
}
```

### デプロイ前の事前チェック

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash(npm run deploy*)",
      "hooks": [
        {
          "type": "command",
          "command": "./scripts/pre-deploy-check.sh"
        }
      ]
    }
  ]
}
```

---

# Subagents - 専門タスクの委譲

![Subagentsの関係図](/images/claude-code-config-guide/subagents.png)
*Subagentsは専門分野を担当するエージェント*

## Subagentsとは？

**Subagents**は、特定の専門分野を持った**別のAIエージェント**です。

:::message
**専門用語解説：Subagent（サブエージェント）**
メインのClaudeから呼び出される、専門的なタスクを担当する別のエージェント。独立したコンテキスト（状況認識）を持ちます。
:::

## サブエージェントの種類

### 組み込みサブエージェント

Claude Codeには、最初から使えるサブエージェントがあります：

| エージェント名 | 役割 | 用途 |
|--------------|------|------|
| Explore | コードベース探索 | ファイル構造やコードの調査 |
| Plan | 実装計画立案 | 複雑なタスクの設計・計画 |
| general-purpose | 汎用タスク | 様々なタスクを柔軟に処理 |

### カスタムサブエージェント

```markdown:.claude/agents/security-expert.md
---
name: security-expert
description: セキュリティの専門家。脆弱性の検出と対策を担当します。
skills: security-check, vulnerability-scan
---

# セキュリティエキスパート

## 役割
- コードのセキュリティ脆弱性を検出
- 修正案の提示
- セキュリティベストプラクティスの適用確認

## 専門分野
- SQL インジェクション
- XSS（クロスサイトスクリプティング）
- 認証・認可の問題
- 機密情報の漏洩
```

### サブエージェントの呼び出し

```
ユーザー：「このコードのセキュリティをチェックして」
Claude：「security-expertサブエージェントを呼び出します」
```

## サブエージェントの利点

### 1. コンテキストの独立

サブエージェントは独立したコンテキストで動作するため、メインの作業に影響を与えません。結果だけがメインに返されます。

### 2. 並列処理

複数のサブエージェントを同時に動かせます：

```
メインClaude
  ├─→ security-expert：脆弱性チェック中
  ├─→ performance-expert：速度計測中
  └─→ docs-expert：README更新中
```

### 3. 専門性の分離

各サブエージェントが自分の得意分野に集中できます。

## 注意点

:::message alert
**コンテキスト量に注意**
サブエージェントを多用すると、全体のコンテキスト量が膨大になることがあります。必要な時だけ使用しましょう。
:::

:::message alert
**ゾンビプロセスに注意**
バックグラウンドで実行したサブエージェントが終了せずに残り続けることがあります。定期的に確認しましょう。
:::

---

# 機能の使い分け早見表

| 機能 | 答える質問 | 用途 |
|-----|-----------|------|
| **CLAUDE.md** | 「常に守るべきことは？」 | プロジェクト全体の方針・禁止事項 |
| **Rules** | 「この場所では何が必要？」 | パス別の詳細設定・コンテキスト追加 |
| **Skills** | 「どうやって実行する？」 | タスクの手順書・スラッシュコマンド |
| **Hooks** | 「いつ自動実行する？」 | 特定タイミングでの自動処理 |
| **Subagents** | 「誰に任せる？」 | 専門タスクの委譲・並列処理 |

:::message
**統合のポイント**
以前は「Commands（コマンド）」と「Skills（スキル）」が別々の機能でしたが、現在は**コマンドがスキルに統合**されています。スキルを作成すると、自動的にスラッシュコマンド（`/スキル名`）として呼び出せます。
:::

---

# 実践：プロジェクト設定例

以下は、実際のプロジェクトでの設定例です：

```
プロジェクト/
├── CLAUDE.md                    # 全体設定
├── .claude/
│   ├── rules/
│   │   ├── frontend.md          # フロントエンドルール
│   │   └── backend.md           # バックエンドルール
│   ├── skills/
│   │   ├── commit-helper/
│   │   │   └── SKILL.md         # コミットスキル（/commit-helper）
│   │   ├── code-review/
│   │   │   └── SKILL.md         # レビュースキル（/code-review）
│   │   ├── test/
│   │   │   └── SKILL.md         # テストスキル（/test）
│   │   └── deploy/
│   │       └── SKILL.md         # デプロイスキル（/deploy）
│   ├── agents/
│   │   └── security-expert.md   # セキュリティエージェント
│   └── settings.json            # プロジェクトのフック設定
└── package.json
```

:::message
**ポイント**
- スキルは`/スキル名`でスラッシュコマンドとして呼び出し可能
- `.claude/commands/`は非推奨（スキルに統合済み）
:::

---

# まとめ

Claude Codeには5つの設定機能があり、それぞれ異なる役割を持っています：

1. **CLAUDE.md** - プロジェクト全体に常に適用される基本設定
2. **Rules** - パスに応じて条件付きで適用されるルール
3. **Skills** - タスクの手順書、スラッシュコマンドとして呼び出し可能
4. **Hooks** - 特定タイミングで自動実行される処理
5. **Subagents** - 専門タスクを委譲する独立エージェント

:::message
**最新の変更点**
以前は「Commands」と「Skills」が別々でしたが、現在は**コマンドがスキルに統合**されています。スキルを作成すると、自動的にスラッシュコマンドとして呼び出せます。
:::

これらを適切に組み合わせることで、プロジェクトに合わせたClaude Codeの振る舞いをカスタマイズできます。

---

# 参考リンク

- [Claude をスキルで拡張する - 公式ドキュメント（日本語）](https://code.claude.com/docs/ja/skills)
- [Claude Code 公式ドキュメント - Memory（日本語）](https://code.claude.com/docs/ja/memory)

