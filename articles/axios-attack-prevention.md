---
title: "クールダウンで防ぐサプライチェーン攻撃"
emoji: "🛡️"
type: "tech"
topics: ["npm", "セキュリティ", "axios", "nodejs", "サプライチェーン攻撃"]
published: true
---

## はじめに

2026年3月31日、週間1億ダウンロードを誇る「axios」が北朝鮮系ハッカーによるサプライチェーン攻撃を受けました。しかし、`.npmrc`ファイルにたった2行の設定を追加するだけで防ぐことができました。

:::message alert
現在は侵害されたバージョンは削除されていますが、今後同様の攻撃を防ぐための学習として記録します。
:::

## 事件の概要

- **侵害されたバージョン**: `axios@1.14.1` と `axios@0.30.4`
- **攻撃方法**: postinstallスクリプトでマルウェアを自動実行
- **検知から削除まで**: 2〜3時間

重要なのは、**`npm install`するだけで感染する**が、**侵害版は数時間で削除された**という点です。

つまり、**リリース直後の最新版をインストールしなければ、実害はなかった**のです。

## たった2行で防げる対策

```ini:.npmrc
# 7日以内にリリースされたパッケージをインストールしない
min-release-age=7
```

npm v11.10.0（2026年2月リリース）で追加された`min-release-age`設定により、**クールダウン期間**を設けます。

### なぜ有効か

- リリースから7日以内のパッケージはインストールされない
- 侵害版は2〜3時間で削除されたため、7日待てば安全
- **正規パッケージの機能は一切損なわれない**

```bash
$ npm install axios@1.14.1
npm ERR! Package axios@1.14.1 was published less than 7 days ago
npm ERR! min-release-age=7 を満たしていません
```

## 他のパッケージマネージャでの設定

この機能は**npm、pnpm、Yarn、Bun**すべてで利用可能です。

| パッケージマネージャ | 設定名 | 設定例（7日間） |
|------------------|--------|----------------|
| **npm** | `min-release-age` | `min-release-age=7` |
| **pnpm** | `minimumReleaseAge` | `minimumReleaseAge=10080` (分) |
| **Yarn** | `npmMinimalAgeGate` | `npmMinimalAgeGate: 7d` |
| **Bun** | `minimumReleaseAge` | `minimumReleaseAge=604800` (秒) |

:::message
単位が異なる点に注意してください。npmは日、pnpmは分、Bunは秒、Yarnは文字列形式です。
:::

## lockファイルとCI/CDの活用

クールダウン期間だけでは不十分です。**lockファイル**を活用しましょう。

```bash
# ローカルで検証してlockファイルをコミット
$ npm install
$ git add package-lock.json
$ git commit -m "chore: update dependencies"
```

```yaml:.github/workflows/ci.yml
# CI/CDではlockファイルに基づいて厳密にインストール
- name: Install dependencies
  run: npm ci
```

この運用により、**ローカルで検証済みのバージョンのみが本番環境にデプロイ**されます。

## 参考: ignore-scripts

理論上は`ignore-scripts=true`で完全防御できますが、多くの正規パッケージ（`esbuild`、`puppeteer`、`husky`など）が動作しなくなるため実用的ではありません。

```ini:.npmrc
# 実用性に欠ける
ignore-scripts=true
```

## まとめ

### ✅ 推奨する対策

```ini:.npmrc
min-release-age=7
```

- たった1行で実用的な防御が可能
- lockファイルと組み合わせることで、検証済みバージョンのみをデプロイ
- CI/CDでは`npm ci`を使用

### 🎯 重要な考え方

サプライチェーン攻撃は発見から削除まで数時間〜数日で対応されます。**最新版をむやみにインストールせず、クールダウン期間を設ける**ことで、実害が発生する前に侵害版が削除され、安全なバージョンのみが利用可能になります。

## 参考資料

### axios攻撃関連
- [2026年3月31日にaxiosが受けたサプライチェーン攻撃の概要と予防策「クールダウン」機能について | yamory](https://yamory.io/blog/supplychain-attack-on-axios)
- [Mitigating the Axios npm supply chain compromise | Microsoft Security Blog](https://www.microsoft.com/en-us/security/blog/2026/04/01/mitigating-the-axios-npm-supply-chain-compromise/)
- [axios ソフトウェアサプライチェーン攻撃の概要と対応指針 - GMO Flatt Security Blog](https://blog.flatt.tech/entry/axios_compromise)

### クールダウン機能関連
- [pnpm 10.16 Adds New Setting for Delayed Dependency Updates | Socket](https://socket.dev/blog/pnpm-10-16-adds-new-setting-for-delayed-dependency-updates)
- [Yarn 4.10 Adds a Release-Age Gate for Safer Dependency Management | Medium](https://medium.com/@roman_fedyskyi/yarn-4-10-adds-a-release-age-gate-for-safer-dependency-management-765c2d18149a)
- [Package Managers Need to Cool Down | Andrew Nesbitt](https://nesbitt.io/2026/03/04/package-managers-need-to-cool-down.html)
- [Supply Chain Hardening: Release Age Gating | GitHub Gist](https://gist.github.com/eapotapov/ae8c5eebf05776918f46a3f61c56cd43)
