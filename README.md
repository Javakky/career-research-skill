# career-research — Claude Code Skill

散逸した一次資料 (公開アカウント / 業務リポ / 個人ファイル / 社内通信 / 大会戦績 / 同人活動) から、 エンジニア 1 名の経歴を **事実ベース + 2 軸評価ベース** で統合し、 統合 CV と転職サービス向けプロジェクト経歴ポートフォリオを生成する Claude Code 用 skill。

## 特徴

- **15 件の失敗事例** (LL1-15) を踏まえた防護策が Phase + Constraints の多層配置
- **8 Phase ワークフロー** (identify → 投入要求 → 自律取得 → Layer 1 → Layer 2 → Layer 3 → Layer 4 → 留意点)
- 既存 rubric が無い領域では **新規 rubric 作成** を提案
- **転職サービス別 pivot** (Findy / ビズリーチ / doda / LinkedIn / レバテック / LAPRAS / 転職ドラフト) に対応
- 学生時代と社会人期の **章分離** + コード設計 rubric (学年別) を分離適用

## インストール

### ローカル運用 (project-local symlink)

```bash
# 自分のプロジェクトの .claude/skills/ にこのリポを symlink で配置
mkdir -p path/to/your/project/.claude/skills
ln -s /path/to/career-research-skill path/to/your/project/.claude/skills/career-research
```

### グローバル運用

```bash
# ~/.claude/skills/ 配下に配置
cp -r . ~/.claude/skills/career-research
```

## 使い方

Claude Code (CLI / IDE 拡張) で以下のいずれかを実行:

```
/career-research
経歴をまとめて
統合 CV を作って
プロジェクト経歴を整理して
```

skill が起動したら、 [checklists/data-intake-checklist.md](checklists/data-intake-checklist.md) の内容がユーザーに提示される。 6 系統 (A-F) の所在情報を 1 メッセージで返すと、 Claude が自律的に Phase 2-7 を実行する。

## 出力構造

```
research/career/<userid>/
  README.md                                   # ナビゲーション
  integrated-cv-<yyyy-mm>.md                  # Layer 4: 統合 CV (style-guide 準拠の読み物)
  project-portfolio-<yyyy-mm>.md              # Layer 4: 転職サービス入力欄向け全 PJ 詳細
  <userid>-evaluation-<yyyy-qN>.md            # Layer 3: 2 軸評価レポート
  resume-<yyyymmdd>.md                        # 職務経歴書 PDF 写し (任意)
  _research-areas/<area>.md                   # Layer 1: 領域別リサーチノート
  _intermediate/                              # Layer 2: 中間成果物 (引用専用)
  _pj-evaluations/                            # Layer 3: PJ 別 2 軸評価
    _metadata.yaml                            # 全 PJ メタ集約
    <family>/<pj>-2axis-evaluation.md
  _rubric-applied-examples/                   # rubric × 本人 PJ の項目別マッピング
    <category>/<rubric>-applied-<pj>.md
```

## 必要な前提環境

- Claude Code (CLI v1.x 以上 / IDE 拡張)
- 公開アカウントの自律取得用ツール:
  - `gh` CLI (GitHub PR / リポ取得)
  - `glab` または GitLab API アクセス (内部 GitLab がある場合)
  - WebFetch (公開ブログ / 動画 / SNS)
- 内部通信ソース (任意):
  - Notion MCP server
  - Slack MCP server
  - Discord MCP server

## 含まれる Bundled Resources

| ファイル | 用途 |
|---|---|
| `SKILL.md` | skill 本体 (8 Phase ワークフロー + 15 件の Lessons learned) |
| `checklists/data-intake-checklist.md` | Phase 1 で提示する完全チェックリスト (A-F 6 系統) |
| `templates/integrated-cv-template.md` | Layer 4 の統合 CV (読み物) 章スケルトン |
| `templates/project-portfolio-template.md` | Layer 4 の転職サービス向け portfolio 章スケルトン (8 章 / メイン + トグル 2 段化) |
| `templates/research-area-template.md` | Layer 1 の領域別ノート構造 |
| `templates/month-by-month-template.md` | Layer 2 月別表の構造 + 出典タグ凡例 |

## 関連する外部 rubric / sources

skill は [_shared/rubrics/](../../research/_shared/rubrics/) と [_shared/sources/](../../research/_shared/sources/) を参照する設計。 これらは別管理 (横断 reference 用の別リポ等) に分離することを推奨。

最低限必要な reference:
- `_shared/rubrics/technical/design-quality-rubric.md`
- `_shared/rubrics/technical/engineer-evaluation-rubric.md`
- `_shared/rubrics/technical/student-era-design-quality-rubric.md`
- `_shared/sources/recruitment-service-formats.md` (転職サービス別フォーマット pivot)

## ライセンス

MIT (詳細は [LICENSE](LICENSE) を参照)

## Contributing

LL16+ の失敗事例追加 / 新規 rubric 提案は歓迎します。 PR でお願いします。

## Acknowledgments

本 skill は実際の経歴整理プロジェクトで発生した 15 件の失敗事例 (LL1-15) を踏まえて構築されています。 同じ過ちを繰り返さないため、 Phase ごとに防護策が組み込まれています。
