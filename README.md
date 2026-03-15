# prompt-review x cc-company — YourOS Design Document

[YourOS](https://github.com/masaki-sigoto/YourOS) の設計原本。
ChatGPT DeepResearch によって生成された、`prompt-review` と `cc-company` を統合した個人向け「開発・業務 OS」の拡張設計ドキュメント。

## What is this?

このリポジトリは、2 つの Claude Code ツールを統合して 1 つの OS（YourOS）を構築するための **設計書** です。

### 統合対象

| ツール | 役割 |
|--------|------|
| **[prompt-review](https://github.com/masaki-sigoto/YourOS/tree/main/skills/prompt-review)** | AI 対話ログを分析し、技術理解度・プロンプティングパターン・AI 依存度を推定するレポート生成スキル |
| **[cc-company](https://www.npmjs.com/package/cc-company)** | Claude Code 上に仮想会社組織（秘書→CEO→部署）を構築し、日常運営を回すプラグイン |

### なぜ統合するのか

- **prompt-review** は「分析」に強いが、日常の運用フロー（タスク管理・意思決定・ナレッジ整理）を持たない
- **cc-company** は「運営」に強いが、品質ゲート・記憶整理・振り返りの仕組みが不足
- 統合することで「**思いつき → 記録 → 整理 → 実装 → 検証 → 記憶**」の完全なサイクルが回る

## Document Structure

設計ドキュメント（`prompt-review × cc-company_DeepResearch.md`）の構成:

| セクション | 内容 |
|-----------|------|
| **A. 2つのリポジトリの比較** | 目的・強み・弱み・向いている用途・シナジー分析 |
| **B. フォルダ構造** | 11 ディレクトリ構成、AI アクセス制御（readable / writable / BLOCKED） |
| **C. スキル設計** | 10 スラッシュコマンドの仕様（テンプレート・フロー・引数） |
| **D. 運用サイクル** | 日次・週次・月次の運用フロー |
| **E. セキュリティ** | PreToolUse フック、.gitignore、AI 境界ポリシー |
| **F. 成長パス** | Week 1 → Week 2 → Month 1 のロードマップ |

## Built Result

この設計書をもとに構築された実装は **YourOS** リポジトリにあります:

**[masaki-sigoto/YourOS](https://github.com/masaki-sigoto/YourOS)**

YourOS には以下が含まれます:

- 11 ディレクトリのフォルダ構造
- 10 個の Claude Code Skills（`/inbox`, `/triage`, `/spec`, `/task` など）
- cc-company 統合（`.company/` + dual-write ルール）
- セキュリティフック（`AI-blocked/` / `Private/` 保護）
- CLAUDE.md（AI ルール定義）

セットアップ手順は [YourOS の README](https://github.com/masaki-sigoto/YourOS#setup%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97%E6%89%8B%E9%A0%86) を参照してください。

## How It Was Built

### エージェントアーキテクチャ

7 日間の構築プロセスで以下のマルチエージェント体制を採用:

```text
Codex MCP (最上位レビュアー)
  └─ Claude Code (PM / オーケストレーター)
       └─ Sub-agents (task-worker × N, 並列実装)
```

- 各 Day の成果物は **Codex MCP** のレビューと GO 承認を経て確定
- 実装はサブエージェント（task-worker）に並列委任
- 全 7 Day で Codex GO を取得（Final Grade: B+）

### 構築タイムライン

| Day | 内容 | Codex |
|-----|------|-------|
| 1 | OS ルート構造 + CLAUDE.md | GO |
| 2 | cc-company 統合 + dual-write | GO |
| 3 | prompt-review スキル移植 | GO |
| 4 | 入口スキル 3 点（inbox / triage / handoff） | GO |
| 5 | 開発フロースキル 4 点（spec / task / next / decide） | GO |
| 6 | 品質ゲート（review-diff）+ セキュリティフック | GO |
| 7 | 週次レビュー（weekly）+ 総合検証 | FINAL GO |

## Prerequisites

YourOS を自分の環境に構築するには:

| 項目 | 要件 |
|------|------|
| Claude Code | v1.0 以上 |
| cc-company | プラグインインストール済み |
| Python | 3.10+（prompt-review 用） |
| Git | 2.x |
| OS | macOS / Linux |

## Quick Start

```bash
# 1. YourOS をクローン
cd ~
git clone https://github.com/masaki-sigoto/YourOS.git

# 2. スキルをインストール
mkdir -p ~/.claude/skills
cp -r ~/YourOS/skills/* ~/.claude/skills/

# 3. パスを自分の環境に置換（macOS）
find ~/.claude/skills/ -name "SKILL.md" -exec sed -i '' "s|/Users/apple|${HOME}|g" {} +
sed -i '' "s|/Users/apple|${HOME}|g" ~/YourOS/CLAUDE.md

# 4. セキュリティフックを設定
chmod +x ~/YourOS/SOP/security/block-ai-write.sh
# ~/.claude/settings.json にフック設定を追加（YourOS README 参照）

# 5. Claude Code で cc-company をセットアップ
cd ~/YourOS
# Claude Code セッション内で /company を実行
```

詳細は [YourOS README](https://github.com/masaki-sigoto/YourOS#setup%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97%E6%89%8B%E9%A0%86) を参照。

## Related

- [masaki-sigoto/YourOS](https://github.com/masaki-sigoto/YourOS) — 実装リポジトリ
- [cc-company](https://www.npmjs.com/package/cc-company) — Claude Code 仮想会社プラグイン
- [Claude Code Skills](https://docs.anthropic.com/en/docs/claude-code/skills) — Skills の仕様

## License

MIT
