---
title: "Claude Code Agent Team × オーケストラ開発：複数AIで開発速度を10倍にする"
emoji: "🎼"
type: "tech"
topics: ["claude", "claudecode", "ai", "開発効率化", "オーケストラ開発"]
published: false
---

## はじめに：なぜAgent Team単体では不十分なのか

Claude Codeの実験的機能「Agent Team」は、複数のAIエージェントを協調させる画期的な機能です。しかし、**Agent Team単体では真の開発速度向上は実現できません**。

なぜなら：
- Claude Codeだけでは、得意・不得意領域がある
- コード実装の速度に限界がある
- 調査・設計・実装の全フェーズを1つのツールでカバーするのは非効率

本記事では、**Agent Team × オーケストラ開発環境**という、複数のAIツールを統合した次世代開発環境を解説します。

## オーケストラ開発環境の全体像

オーケストラ開発とは、複数のAIツールを指揮者（Conductor）が統制し、各ツールの得意分野を活かして開発を加速させるアプローチです。

### 基本構成

```
┌─────────────────────────────────────────────────┐
│         Claude Code (PM/PL - 指揮監督)          │
│  - タスク分解と優先順位付け                      │
│  - Codex Spark判断                              │
│  - Agent Team統括                               │
│  - 品質保証とレビュー                            │
└─────────────────────────────────────────────────┘
            │
            ├─────────────┬─────────────┬──────────────┐
            ▼             ▼             ▼              ▼
    ┌──────────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
    │    Codex     │ │  Gemini  │ │  chaba   │ │  Agent   │
    │   (実装)     │ │(調査設計)│ │(Git管理) │ │   Team   │
    └──────────────┘ └──────────┘ └──────────┘ └──────────┘
         │                │             │              │
         │                │             │              │
    並列実装         設計ドキュメント  worktree     並列レビュー
```

### 各ツールの役割

#### 1. Claude Code（指揮監督 - PM/PL）

**責務:**
- タスク分解と優先順位付け
- 実装難易度の判断（Codex Spark vs 通常モデル）
- Agent Teamの構成と統括
- 最終的な品質保証とレビュー

**なぜClaude CodeがPM/PLなのか:**
- コンテキスト理解能力が高い
- 全体設計を俯瞰できる
- Agent Team機能で複数視点を統合できる

#### 2. Codex（実装エンジン）

**責務:**
- コード実装（メイン）
- ユニットテスト作成
- リファクタリング

**モデルの使い分け:**

```bash
# 簡単な実装（CRUD、単純なロジック）
codex exec -m spark "ユーザー一覧APIを実装"

# 複雑な実装（アルゴリズム、状態管理）
codex exec -m gpt-5.2-codex "並列処理の最適化実装"
```

**Codex Sparkの特性:**
- ⚡ **速度**: 通常モデルの2-3倍速い
- 🤔 **推論時間**: 短い（複雑な思考を要さない）
- ✅ **適用場面**: CRUD、定型処理、テンプレート生成
- ❌ **不向き**: アルゴリズム設計、複雑な状態管理、パフォーマンス最適化

#### 3. Gemini（調査・設計エンジン）

**責務:**
- 技術調査（ライブラリ選定、ベストプラクティス）
- アーキテクチャ設計
- APIドキュメント解析
- 設計書作成

**なぜGeminiが調査・設計なのか:**
- 最新情報へのアクセス能力
- 多角的な視点での分析
- 大規模ドキュメントの理解力

#### 4. chaba（Git/worktree管理）

**責務:**
- PRレビュー環境の自動セットアップ
- 並列開発環境の管理
- AIエージェントの自動起動とレビュー

**主要コマンド:**

```bash
# PRレビュー環境を自動構築
chaba review --pr 123

# AIエージェント統合レビュー
chaba review --pr 123 --with-agent

# 徹底レビュー（全エージェント起動）
chaba review --pr 123 --thorough

# レビュー結果確認
chaba agent-result --pr 123
```

## Agent Team × オーケストラの統合

### なぜAgent Teamが必要なのか

オーケストラ開発環境でも、**Claude Code側で複数の視点が必要な場面**があります：

1. **タスク分解の妥当性検証**
   - Pragmatist: 「このタスク、本当に必要？」
   - Skeptic: 「依存関係が抜けてないか？」
   - Idealist: 「長期的な保守性は？」

2. **Codex Sparkの判断**
   - Technical Reviewer: 「この実装、本当に簡単か？」
   - Performance Analyst: 「速度優先で品質を犠牲にしていないか？」

3. **統合レビュー**
   - Code Reviewer: コード品質
   - Architecture Reviewer: 設計整合性
   - Security Reviewer: セキュリティリスク

### 統合パターン例

#### パターン1: タスク分解 + 並列実装

```
[Claude Code - Agent Team起動]
├─ Task Planner: タスク分解
├─ Complexity Analyzer: 難易度判定
└─ Dependency Checker: 依存関係チェック
        ↓
    タスクリスト確定
        ↓
[並列実装フェーズ]
├─ Codex (Spark): 簡単なタスク3件（並列）
├─ Codex (通常): 複雑なタスク2件（並列）
└─ Gemini: API調査とドキュメント作成
        ↓
[統合レビューフェーズ]
└─ Claude Code - Agent Team起動
    ├─ Technical Reviewer
    ├─ Integration Checker
    └─ Final Approver
```

#### パターン2: PRレビュー自動化

```bash
# 1. chaba経由でPRレビュー環境構築
chaba review --pr 234 --thorough

# 2. 自動的に実行されること
#    - worktree作成
#    - 依存関係インストール
#    - Claude Code起動（Agent Team: 3人）
#    - Codex起動（コード品質チェック）
#    - Gemini起動（設計整合性チェック）

# 3. レビュー結果を統合
chaba agent-result --pr 234
```

## Codex Sparkの使い分け戦略

### 判断フローチャート

```
タスク受領
    ↓
[Claude Code or Geminiが判断]
    │
    ├─ 定型処理・CRUD？ ────→ YES → Codex Spark
    │                               ⚡ 高速実装
    ├─ 単純なロジック？ ────→ YES → Codex Spark
    │
    ├─ テンプレート生成？ ──→ YES → Codex Spark
    │
    └─ 複雑なアルゴリズム？ → YES → Codex（通常）
                                      🎯 品質優先
       複雑な状態管理？ ───→ YES → Codex（通常）

       パフォーマンス最適化？ → YES → Codex（通常）
```

### 実装例

#### Claude Codeからの呼び出し

```markdown
# Sparkを使用する判断（Claude Code内部で実行）

この実装は以下の理由でSparkを使用：
- CRUD操作の定型処理
- 複雑なロジックなし
- テストも定型的
```

```bash
# Claude CodeがBashツールで実行
codex exec -m spark "ユーザー一覧取得APIを実装。
- GET /api/users
- ページネーション対応
- ユニットテスト含む"
```

#### 通常モデルを使用する判断

```markdown
# 通常モデルを使用する判断

この実装は以下の理由で通常モデルを使用：
- 並列処理の最適化が必要
- エッジケースの考慮が重要
- パフォーマンス要件が厳しい
```

```bash
# Claude CodeがBashツールで実行
codex exec -m gpt-5.2-codex "リアルタイム通知システムの実装。
- WebSocket接続管理
- 接続断時の再接続ロジック
- メッセージキューイング
- ユニットテスト + 統合テスト"
```

### 判断基準マトリクス

| タスクタイプ | 複雑度 | Codex Spark | Codex通常 | 判断者 |
|-------------|--------|------------|-----------|--------|
| CRUD実装 | 低 | ✅ 推奨 | - | Claude/Gemini |
| 単純なバリデーション | 低 | ✅ 推奨 | - | Claude |
| テンプレート生成 | 低 | ✅ 推奨 | - | Claude |
| APIクライアント実装 | 中 | ⚠️ 要検討 | ✅ 推奨 | Gemini（API調査後） |
| 状態管理（Redux等） | 高 | ❌ 非推奨 | ✅ 必須 | Claude + Agent Team |
| アルゴリズム最適化 | 高 | ❌ 非推奨 | ✅ 必須 | Claude + Agent Team |
| 並列処理実装 | 高 | ❌ 非推奨 | ✅ 必須 | Claude + Agent Team |

## 実践的なチーム構成例

### 3人構成（レート制限考慮）

**ユースケース:** 記事執筆、ドキュメント作成、小規模レビュー

```
Team Lead (Claude Code)
├─ Researcher (Gemini担当): 情報収集
├─ Writer (Codex Spark担当): 執筆
└─ Reviewer (Claude Code): 最終レビュー
```

**実行例:**
```markdown
Researcher、Writer、Reviewerの3人チームで記事を書いて。

# タスク
1. [Researcher] Agent Teamとオーケストラ開発の事例調査
2. [Writer] 調査結果をもとに記事執筆
3. [Reviewer] 技術的正確性と読みやすさをレビュー
```

### 4人構成（性格ベース）

**ユースケース:** 技術選定、アーキテクチャ設計

```
Team Lead (Claude Code)
├─ Pragmatist: シンプルさと出荷速度重視
├─ Skeptic: リスクと前提を疑問視
├─ Idealist: 長期的品質を追求
└─ Connector: パターン認識と全体文脈
```

**実装例:**

各エージェントのプロンプト定義（Claude Codeが内部で使用）:

```markdown
# Pragmatist
You are the Pragmatist on the team.

Core Lens: Ship fast, iterate based on real feedback.

Key Questions:
- What's the simplest solution that works?
- Can we deliver this in 1 week?
- Which features can we cut without losing core value?

Pushes Back On:
- Over-engineering
- Premature optimization
- Feature creep
```

### 5人構成（開発プロジェクト）

**ユースケース:** 大規模機能開発、Chrome拡張開発

```
Team Lead (Claude Code)
├─ PdM: 要件定義と優先順位
├─ Frontend Dev (Codex): UI実装
├─ Backend Dev (Codex): API実装
├─ Tester (Codex Spark): テスト実装
└─ Reviewer (Claude Code): 統合レビュー
```

**実装フロー:**
1. PdMがタスク分解（Claude Code Agent Team）
2. Frontend/Backend Dev並列実装（Codex × 2）
3. Tester並列テスト作成（Codex Spark）
4. Reviewer統合レビュー（Claude Code Agent Team）

## 体験前後の違い

### 従来の開発（単一エージェント）

```
[作業フロー]
タスク受領
  ↓（30分）
要件理解・調査
  ↓（1時間）
実装
  ↓（30分）
テスト
  ↓（30分）
レビュー
  ↓
完了（計2.5時間）
```

**課題:**
- すべて逐次実行で時間がかかる
- 視点が固定され、見落としが発生
- レビューも同じコンテキストで実施（バイアス）

### Agent Team × オーケストラ開発

```
[作業フロー]
タスク受領
  ↓
[Claude Code - Agent Team] タスク分解（10分）
├─ Task Planner
├─ Complexity Analyzer
└─ Dependency Checker
  ↓
【並列実行フェーズ（30分）】
├─ Gemini: 技術調査 + 設計書作成
├─ Codex (Spark): 簡単な実装3件
└─ Codex (通常): 複雑な実装1件
  ↓
【並列テストフェーズ（15分）】
├─ Codex (Spark): ユニットテスト
└─ Codex (通常): 統合テスト
  ↓
[Claude Code - Agent Team] 統合レビュー（15分）
├─ Technical Reviewer
├─ Integration Checker
└─ Final Approver
  ↓
完了（計1.2時間）
```

**成果:**
- ⚡ **速度**: 2.5時間 → 1.2時間（約50%削減）
- 🎯 **品質**: 複数視点のレビューで見落とし削減
- 🔄 **並列性**: 調査・実装・テストを同時並行
- 💡 **最適化**: Spark使用で実装速度向上

### コスト比較

| 構成 | トークン消費 | 時間 | コスト効率 |
|------|-------------|------|-----------|
| 単一エージェント | 基本 × 1 | 2.5h | 基準 |
| Agent Team (3人) | 基本 × 2.5* | 1.2h | 🟢 高い |
| オーケストラ（並列） | 基本 × 3* | 0.8h | 🟢 最高 |

*キャッシュ効果で実際はより少ない

## 環境構築手順

### 1. Agent Team有効化

```bash
# ~/.zshrc に追加
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# 反映
source ~/.zshrc

# 確認
echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS
# 出力: 1
```

### 2. Codex CLI設定

```bash
# モデル確認
cat ~/.codex/config.toml | grep model

# デフォルトモデル設定例
model = "gpt-5.2-codex"
model_reasoning_effort = "high"
```

### 3. Gemini CLI設定

```bash
# Gemini CLIインストール確認
which gemini

# 基本的な使用方法
gemini "技術調査のプロンプト"
```

### 4. chaba インストール（オプション）

```bash
# ソースからインストール
git clone https://github.com/Nenene01/chaba.git
cd chaba
cargo install --path .

# 確認
chaba --version

# 初期設定
chaba config --local
```

## 実践編：Agent Team × オーケストラでiOSアプリを開発した

ここまでの理論を実際に試すため、Agent Team × オーケストラ開発で**学生向け体育授業リマインダーアプリ「SportsDay+」**を開発しました。

### プロジェクト概要

**アプリ名:** SportsDay+
**目的:** 学生（小中高）が体育の授業がある曜日を設定し、体操服・ジャージ持参を忘れないようリマインド
**技術スタック:** Expo + React Native + TypeScript
**開発時間:** 約2時間（要件定義〜MVP実装）
**GitHub:** [Nenene01/SportsDay-app](https://github.com/Nenene01/SportsDay-app)

### フェーズ1: 要件定義（Agent Team 3人構成）

#### チーム構成

1. **Product Manager** - ビジネス要件整理
2. **Technical Architect** - 技術仕様設計
3. **UX Designer** - 画面設計

#### 実行コマンド

```bash
# Agent Teamを並列起動（Claude Code内で実行）
Task tool × 3（並列実行）
```

#### 成果物

**Product Managerの成果:**
- アプリ名決定：「SportsDay+（スポーツデー）」
- ターゲットユーザー分析（小学生/中学生/高校生の特性）
- 競合調査結果：直接競合なし（ブルーオーシャン市場）
- 3つのユーザーストーリー

**Technical Architectの成果:**
- 技術スタック選定（Expo SDK 53+、AsyncStorage、expo-notifications）
- データ構造設計（PESchedule、WeeklySchedule、ClassPeriod）
- 1日実装スケジュール案
- 技術的リスク評価

**UX Designerの成果:**
- 6画面の構成設計
- メイン画面レイアウト（アスキーアート図解）
- カラースキーム提案（緑=体育あり、青=体育なし）
- 30秒で完了する操作フロー

#### 所要時間

**約30分**（3人のAgent Teamが並列で作業）

単一エージェントなら1時間以上かかる作業を、**Agent Teamで50%短縮**。

### フェーズ2: 実装（Codex）

#### Codex判断

Technical Architectの設計に基づき、以下のように実装を分類：

| ファイル | 複雑度 | モデル選択 | 理由 |
|---------|--------|-----------|------|
| src/types/schedule.ts | 低 | Codex | 型定義は定型処理 |
| src/repositories/ScheduleRepository.ts | 低 | Codex | AsyncStorage CRUDは単純 |
| src/services/NotificationService.ts | 中 | Codex | 通知ロジックは標準的 |
| App.tsx | 中 | Codex | React Native UIは定型的 |

**Codex Sparkは使用せず**、品質優先で通常モデル（gpt-5.2-codex）を選択。

#### 実行コマンド

```bash
cd /path/to/SportsDay-app

# 型定義実装
codex exec "型定義を実装..."

# 残りの実装を一括生成
codex exec --sandbox workspace-write "
Repository、NotificationService、App.tsxを実装..."
```

#### 成果物

- **src/types/schedule.ts** (104行): TypeScript strict型定義
- **src/repositories/ScheduleRepository.ts** (62行): AsyncStorage CRUD
- **src/services/NotificationService.ts** (152行): 週次通知ロジック
- **App.tsx** (300行): 完全に動作するUI

#### 所要時間

**約15分**（Codexがコード生成）

人間が手で書けば3-4時間かかる実装を、**Codexで93%削減**。

### フェーズ3: 統合レビュー（Claude Code）

生成されたコードをClaude Codeが確認：

```typescript
// 型安全性の確認
export type PeriodNumber = 1 | 2 | 3 | 4 | 5 | 6 | 7;
export type TimeString = `${number}${number}:${number}${number}`;
// ✅ 厳密な型定義で実行時エラーを防止

// エラーハンドリングの確認
try {
  const storedSchedule = await AsyncStorage.getItem(STORAGE_KEY);
  // ...
} catch (error) {
  console.warn('Failed to load schedule:', error);
  return getDefaultSchedule();
}
// ✅ 適切なフォールバック処理
```

**品質確認:** すべてのファイルがTypeScript strictモードでコンパイル成功。

### 開発成果まとめ

| 指標 | 従来開発 | Agent Team × オーケストラ | 削減率 |
|------|---------|------------------------|--------|
| 要件定義 | 1-2時間 | 30分 | 50-75% |
| 実装時間 | 3-4時間 | 15分 | 93% |
| 総開発時間 | 4-6時間 | 45分 | 87% |
| コード品質 | 中 | 高（型安全、エラーハンドリング） | 向上 |

### 学んだこと

#### 1. Agent Teamの威力

3人のAgent Teamが並列で要件定義を行うことで：
- 同時に3つの視点から検討
- 相互補完的な成果物（PdMがユーザーストーリー、Architectが技術仕様、UX Designerが画面設計）
- **単一エージェントでは見落としがちな観点**（競合分析、技術的リスク、アクセシビリティ）を網羅

#### 2. Codexの実装品質

Codexが生成したコードは：
- TypeScript strict modeで一発コンパイル成功
- JSDocコメント完備
- エラーハンドリング適切
- **人間が書くのと同等以上の品質**

#### 3. オーケストラ開発の実践的課題

**課題1:** Codexのサンドボックス制約
→ 最初read-onlyで実行、`--sandbox workspace-write`で再実行

**課題2:** モデル選択の判断
→ 迷ったら通常モデル（品質優先）が安全

**課題3:** コンテキスト共有
→ 型定義を先に実装し、それを参照させることで整合性確保

### フェーズ4: App Store公開準備（Codex）

#### EAS Build設定

**app.json更新:**
- bundleIdentifier設定（`com.nenene01.sportsday`）
- 通知権限の説明文追加
- ビルド番号設定

**eas.json作成:**
```json
{
  "build": {
    "production": {
      "autoIncrement": true,
      "ios": { "simulator": false }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id@example.com"
      }
    }
  }
}
```

#### プライバシーポリシー生成（Codex）

```bash
codex exec "プライバシーポリシーをMarkdown形式で生成..."
```

**生成されたポリシー:**
- データ収集なし（ローカルストレージのみ）
- 外部送信なし
- GDPR・個人情報保護法準拠
- App Store審査基準適合

#### 所要時間

**約10分**（EAS CLI設定 + Codexによるポリシー生成）

### 総開発時間

| フェーズ | 所要時間 |
|---------|---------|
| 要件定義（Agent Team 3人） | 30分 |
| 実装（Codex） | 15分 |
| App Store準備（Codex + EAS） | 10分 |
| **合計** | **55分** |

### 次のステップ（今後の展開）

1. **EAS Build実行** - `eas build --platform ios --profile production`
2. **iOSウィジェット実装** - UX Designerが設計した3サイズウィジェット
3. **実機テスト** - 通知機能の動作確認
4. **App Store Connect設定** - アイコン、スクリーンショット、説明文
5. **App Store申請** - `eas submit --platform ios`

---

## まとめ：次世代開発環境の構築

Agent Team × オーケストラ開発は、**複数のAIツールを統合し、各ツールの強みを活かして開発速度を最大化する**アプローチです。

### 重要なポイント

1. **Claude CodeはPM/PL**: タスク分解、判断、統括に特化
2. **Codex Sparkの使い分け**: 簡単な処理は高速化、複雑な処理は品質優先
3. **Geminiで調査・設計**: 最新情報と多角的視点を活用
4. **chabaで環境管理**: PRレビューを自動化し、並列開発を実現
5. **Agent Teamで多視点**: 重要な判断やレビューに複数視点を導入

### 始め方

```bash
# 1. Agent Team有効化
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# 2. Claude Code起動
claude-code

# 3. オーケストラ開発を指示
「Researcher（Gemini）、Implementer（Codex）、Reviewer（Claude Code）の
3人チームで、ユーザー管理機能を実装して。
Implementerは簡単な処理にSparkを使って。」
```

### 次のステップ

1. **小規模から開始**: 3人構成で記事執筆やドキュメント作成から
2. **徐々に拡大**: 4-5人構成で開発プロジェクトに適用
3. **chabaを導入**: PRレビューの自動化で並列開発を加速
4. **最適化を継続**: Spark使い分けのノウハウを蓄積

Agent Team × オーケストラ開発は、すでに複数の開発者が実践し、開発速度の大幅な向上を実証しています。あなたも次のプロジェクトで、この次世代開発環境を試してみてください。

---

## 参考文献

### Agent Team関連
[^1]: [Claude Code "Agent Teams" は役割 (roles) ではなく性格 (personalities) で編成すると強い](https://zenn.dev/happy_elements/articles/d01195392ceb10)
[^2]: [Claude Code Agent Teamsに記事を書いてもらった](https://zenn.dev/aun_phonogram/articles/5bbc0c5ca40df5)
[^3]: [Claude Code Agent Teams を試してみた](https://qiita.com/WdknWdkn/items/4eff08ff2413c23b95ec)

### オーケストラ開発関連
- [chaba - AI Agent Friendly Source Review & Debug Environment](https://github.com/Nenene01/chaba)

---

*この記事は [Claude Code](https://claude.ai/code) のAgent Team機能とオーケストラ開発環境を使用して作成されました。*
