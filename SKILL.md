---
name: "career-research"
description: "Invoke ONLY when the user explicitly asks to build / 統合する / 構造化する a personal career portfolio (経歴書 / 統合 CV / 職務経歴書 / プロジェクト経歴) from scattered sources — GitHub / GitLab / SNS / ブログ / 個人ファイル / 社内通信 / 大会戦績 / 同人活動 等. Triggers: `/career-research`, 『経歴をまとめて』『統合 CV を作って』『職務経歴書を 1 本にして』『キャリアを構造化して』『プロジェクト経歴を整理して』. Workflow: (1) skill 起動時にデータ投入チェックリストを提示し、 URL / リポジトリ名 / ファイルパス / Notion DB / 社内 user_id を収集 → (2) 自律取得 (gh / glab / git log / WebFetch / MCP / Read) + 本人 user_id を notion-get-users self で確定 → (3) Layer 1 (領域別 _research-areas/<area>.md) → Layer 2 (中間成果物 _intermediate/) → Layer 3 (2 軸評価 / 既存 rubric が無い場合は新規作成) → Layer 4 (integrated-cv-<yyyy-mm>.md + project-portfolio-<yyyy-mm>.md) を生成 → (4) 2 軸評価のラダー × 年数 × 経験を踏まえた重み付けで読み物化. Do NOT auto-invoke on 単発の git log 集計 / 単一 PR 一覧化 / 単一ブログ記事の解析 / 一時的なキャリア相談."
---

# career-research — 統合 CV + プロジェクト経歴ポートフォリオ 構築 skill

## Overview

散逸した一次資料 (公開アカウント / 業務リポ / 個人ファイル / 社内通信 / 大会戦績 / 同人活動) から、エンジニア 1 名の経歴を **事実ベース + 2 軸評価ベース** で統合し、`research/career/<userid>/` 配下に最終成果物 4 種を出力する。前提知識のない実行者でも `/career-research` 1 回で必要な投入が要求され、 最終的に 13 章構成の統合 CV + 転職サービス向けプロジェクト経歴ポートフォリオまで到達する。

## 出力構造 (最終)

```
research/career/<userid>/
  README.md                                   # ナビゲーション
  integrated-cv-<yyyy-mm>.md                  # Layer 4: 統合 CV (style-guide 準拠の読み物)
  project-portfolio-<yyyy-mm>.md              # Layer 4: 転職サービス入力欄向け全 PJ 詳細
  <userid>-evaluation-<yyyy-qN>.md             # Layer 3: 2 軸評価レポート (主成果物)
  resume-<yyyymmdd>.md                        # 職務経歴書 PDF 写し
  _research-areas/<area>.md                   # Layer 1: 領域別リサーチノート (N 本)
  _intermediate/                              # Layer 2: 中間成果物 (引用専用)
    career-timeline.md
    _month-by-month-draft.md
    github-contributions.md
    gitlab-<host>-contributions.md
    public-profiles.md
    card-game-activity-profile.md (任意)
    fy<yyyy>-tax-return-summary.md (任意)
  _pj-evaluations/                            # Layer 3: PJ 別 2 軸評価
    _metadata.yaml                            # 全 PJ メタ集約 (1 ファイル / ファミリ別 yaml は使わない)
    <family>/<pj>-2axis-evaluation.md
  _rubric-applied-examples/                   # rubric × 本人 PJ の項目別マッピング + 一次資料保管
    <category>/<rubric>-applied-<pj>.md
```

## 役割境界 (重要)

| ファイル種別 | 役割 | 書く対象 |
|---|---|---|
| `_research-areas/<area>.md` | 領域別リサーチノート | 一次資料調査結果 / 時系列 / 主要案件詳細 |
| `_intermediate/<topic>.md` | 中間成果物 | PR 集計 / SNS / 月別表 / 戦績テーブル (引用専用) |
| `_shared/rubrics/<cat>/<name>-rubric.md` | 汎用採点軸 (横断 reference) | 採点項目の **抽象定義** のみ。 本人実数値は書かない |
| `_rubric-applied-examples/<cat>/<rubric>-applied-<pj>.md` | rubric × PJ の項目別マッピング | 各項目への ✅/🟡/⚪/❌ + 確度ラベル + 本人実数値 + 一次資料パス |
| `_pj-evaluations/<family>/<pj>-2axis-evaluation.md` | PJ 単位の集約スコア | 8-13 カテゴリ統合 % + 強み弱み + 年収 anchor + 推奨アクション |
| `_pj-evaluations/_metadata.yaml` | 全 PJ メタ集約 | quadrantChart + ファミリ別統合スコアの元データ |
| `integrated-cv-<yyyy-mm>.md` | 読み物 (style-guide 準拠) | 9 章 + 2 軸評価統合 + Mermaid + 表 / 段落 4-5 行 / 視覚要素 20-30% |
| `project-portfolio-<yyyy-mm>.md` | 転職サービス入力欄向け | 全 PJ を STAR + 設計判断 + 学び で詳細記述 (メイン 500-800 字 / トグル 200-300 字) |

## Workflow (8 Phase)

### Phase 0: identify (1 回の質問でまとめて聞く)

`AskUserQuestion` で次を取得:
- **userid**: 出力ディレクトリ名に使う識別子 (短い slug)
- **基準日**: 統合 CV の基準日 (`yyyy-mm-dd`)
- **言語**: 出力言語 (デフォルト: 日本語)
- **context tags**: work / side / hobby のうち取り扱う範囲 (CLAUDE.md の context separation に従う)
- **対象転職サービス**: project-portfolio.md のフォーマット pivot (Findy / ビズリーチ / doda / LinkedIn / レバテック / LAPRAS / 転職ドラフト 等)

`research/career/<userid>/` を作成。既存なら追記モードを確認。

### Phase 1: 投入要求 (Data Intake Checklist)

`checklists/data-intake-checklist.md` の内容を提示し、 「思いつく限り URL / リポジトリ名 / ファイルパス / Notion DB URL を並べてください」 と促す。 6 系統 (公開アカウント / ローカル git / 内部 GitLab / Downloads / MCP / 自己申告ファクト) を 1 メッセージで受け取り、 Claude 側で構造化する。

#### Phase 1 のスキップ条件 (検品 LL13: prompt 内に投入情報が含まれる場合)

`/career-research` 起動時の prompt に **A-F 全てが含まれている場合は、 チェックリスト提示をスキップ** して直接 Phase 2 へ進む。

判定方針:
- prompt 内に「### A. 公開アカウント」「### F. 自己申告ファクト」 のような明示的なセクション分けがあれば skip
- 部分的に含まれている場合 (例: A / B / F のみ提供) は **不足カテゴリのみ追加質問** (深掘りループは禁止 / 1 回まで)
- prompt に「無し / 該当外」 と明示されているカテゴリは追加質問しない

例:
```
prompt: "/career-research ... ### A. GitHub: alice ... ### F. 学校: 〇〇大学 ..."
→ Phase 1 skip → Phase 2 へ直接進む
```

### Phase 2: 自律取得 + 本人 user_id 確定

**重要**: 社内 Notion / Slack の本人 user_id を最初に確定する。

```
1. notion-get-users で user_id="self" → 本人の user_id 取得
2. 取得した user_id を後の created_by フィルタに使う
3. 推測した user_id は使わない (前例: 参加者頻出 user_id を本人と誤判定して PM 業務追記したが、 実は別人だった)
```

| ソース | ツール |
|---|---|
| GitHub 公開 PR / リポ | `gh repo list` / `gh search prs` / `gh api /users/<user>` |
| ローカル git log | `git log --author=<name>` (+ 多人数チームの場合は寄与判定) |
| 内部 GitLab | `glab mr list` / `glx` MCP / GitLab API |
| 公開ブログ / 動画 | `WebFetch` (Qiita / Zenn / note / 業務ブログ / YouTube / X) |
| Notion DB | `notion-fetch` で DB schema 取得 → `notion-query-data-sources` で SQL モード `WHERE 作成者 = ?` |
| Slack | MCP slack tools |
| ローカルファイル | `Read` + `assets/` にコピー保全 (揮発性 dir 依存禁止) |

### Phase 3: Layer 1 — 領域別リサーチノート

取得結果を領域別に分類して `_research-areas/<area>.md` を生成。

#### 領域の分割基準 (抽象)

**抽象基準**:
1. **雇用形態 / 関与形態が異なるか** (本業正社員 / 副業業務委託 / 個人 / 学術 / ボランティア)
2. **同じ「組織」 内で 1 年以上継続するか** (1 年未満の単発案件は領域として独立しない)
3. **ロール / レイヤーが安定しているか** (実装中心 / リーダー中心 / 研究中心 / 制作中心)
4. **ドメイン特化があるか** (ゲーム / 決済 / ML / コンテンツ / セキュリティ)

**よくある領域パターン (典型)**:
- `employment-<company>.md` — 本業 (1 社につき 1 本)
- `side-<contract>.md` — 副業 / 業務委託 (契約先につき 1 本)
- `personal-oss.md` — 個人 OSS / 上流コントリビュート
- `writing-creative.md` — 執筆 / 発信 / 創作
- `community.md` — コミュニティ / 登壇 / 認定
- `<domain>-activity.md` — ドメイン固有 (例: 競技ゲーム / 同人創作 / e スポーツ 等)
- `student-era.md` — 学生時代 (高等教育機関 / 大学院 等。 社会人と別ファイルが筋)

**不足分のチェック観点**:
- 本業の中で長期 (3 年以上) のブランクを挟むか? (例: 領域 A に集中 → 領域 B 復帰) → 領域は同じだが時期で分けて記述
- 学術活動 (卒研 / 論文) は別ファイルか? (publish 未達なら個人 OSS に統合 / 達成なら独立)
- メディア運営 (YouTube / Twitch / ブログ) は別ファイルか? (規模により判断)
- **契約前のアウトプット系譜** (副業 / 業務委託の場合 / LL16): 契約成立前に本人側で蓄積した提案書 / プロト / 個人実装 / 企画書 PDF が複数あれば、 領域ノートに **§前史 (pre-contract output lineage)** として独立章を立てる。 「提案 → プロト → 企画書 → 契約」 の系譜は narrative の核になる (報酬・依頼なしの一方向アウトプットだけで契約化したケースは強い差別化軸)

### Phase 4: Layer 2 — 中間成果物

`_intermediate/` 配下に集計データを生成。 これらは「引用専用」 で、 統合 CV / portfolio から参照されるが、 読み物として読むものではない。

| ファイル | 内容 |
|---|---|
| `career-timeline.md` | 時系列年表 (種別タグ付き) |
| `_month-by-month-draft.md` | 月別 1 行ハイライト + 出典タグ |
| `github-contributions.md` | アカウント別 × 投稿先別の PR / merged テーブル |
| `gitlab-<host>-contributions.md` | プロジェクト別 MR + ローカル git log で全期間補完 |
| `public-profiles.md` | SNS / ブログ / 動画統合プロフィール |
| `<domain>-activity-profile.md` (任意) | ドメイン固有 (競技ゲーム / 同人創作 / e スポーツ 等) |
| `fy<yyyy>-tax-return-summary.md` (任意) | 確定申告サマリ (副業 anchor 用) |

### Phase 5: Layer 3 — PJ 別 2 軸評価 (キモ / 必須)

> **このフェーズが skill の核心**。 設計品質 × 人材レベルの 2 軸で各 PJ を採点し、 ラダー × 年数 × 経験を踏まえた重み付けで Layer 4 を書く。

#### 5.1 採点フロー

1. 各 PJ について `research/_shared/rubrics/{service,technical,ml,domain}/` から **該当する rubric を選択** (複数可)
2. **既存 rubric が無い場合は新規作成** (詳細は §5.2)
3. rubric の項目を本人 PJ に適用し、 `_rubric-applied-examples/<cat>/<rubric>-applied-<pj>.md` を生成
4. 集約スコアを `_pj-evaluations/<family>/<pj>-2axis-evaluation.md` に出力
5. 全 PJ のメタを `_pj-evaluations/_metadata.yaml` に集約

#### 5.1.x 採点範囲 (検品 LL14: rubric 適用範囲の明示)

**全 PJ に rubric を適用するのではない**。 採点対象は次に絞る:

- **本人が F-4 で「注力 PJ」 として指定したもの** (最大 3 件 / 学生期 / 社会人期 別々)
- **2 軸評価 70%+ になりそうなメイン PJ** (リーダー / 主担当 / 長期継続)
- **特殊事例 (8 年スパン波及 / 上流 OSS 5 年継続 等)** で個別採点が価値あるもの

軽量 PJ (1 commit のスニペット / 学年標準の課題作品) は **rubric 適用不要**。 portfolio.md のトグルに 100-200 字で言及するだけで OK。

#### 5.2 rubric 新規作成工程

既存 rubric に該当するものがない場合、 **新規作成を提案** (skill が勝手に作らない / 必ずユーザー確認)。

新規 rubric の構造:
```
# <rubric-name>-rubric
## 0. 採点方針 (採点記号 / 確度ラベル / 採点単位)
## 1-N. カテゴリ別採点軸 (各カテゴリで 4-8 項目)
## N+1. 集計方法
## N+2. レベル換算 (任意)
## N+3. 採点の限界
## 参考文献
```

採点軸の選び方:
- **既存 rubric との互換性**: design-quality-rubric / engineer-evaluation-rubric などと並ぶ採点記号 (✅/🟡/⚪/❌/N/A) + 確度ラベル (観測事実 / 外部比較 / 推定 / 未確認) を踏襲
- **学年別 / 役職別の到達点**: 学生期は学年別、 社会人期はジュニア / Senior / Staff の段階で評価軸を分ける
- **業界 / ドメインの特性**: 競技プレイヤーなら大会出場 / 入賞 / ランキング / メディア出演 など、 ドメイン特有の軸を立てる

新規作成時の注意:
- **抽象採点軸のみを書く** (本人実数値は書かない / それは applied-example へ)
- **匿名化**: 採点軸は **他人の同種 PJ でも採用可能** な汎用化を保つ
- **参考文献**: 外部の業界標準 / カリキュラム / 論文との対応を明示

#### 5.3 ラダー × 年数 × 経験の重み付け

2 軸評価で **「より評価できる部分」 に分量を集中する**:

- **Senior 上位 / Staff 境界判定が出た PJ**: 統合 CV / project-portfolio で厚く書く (メイン展開 / 500-800 字)
- **Senior 中堅 / Junior 判定の PJ**: トグル (200-300 字) で網羅性を担保
- **「シニア標準で当たり前」 の項目 (PSR / Conventional Commits / Hexagonal namespace 等)**: N/A 除外で採点 (ボーナス点ではなく設計判断の実体を測る)

### Phase 6: Layer 4 — integrated-cv + project-portfolio

#### 6.0 必須 / 任意の階層 (検品 LL15: 成果物の優先順位)

| 成果物 | 必須 / 任意 | 出力条件 |
|---|---|---|
| `project-portfolio-<yyyy-mm>.md` | **必須** | 全ケース |
| `integrated-cv-<yyyy-mm>.md` | **任意** | Layer 3 (2 軸評価) をフル実行した場合のみ (= 注力 PJ + 主要 PJ 5 件以上採点済) |
| `<userid>-evaluation-<yyyy-qN>.md` | **任意** | 2 軸評価レポートを単独ファイル化する場合のみ。 統合 CV 内 §7 で代替可 |
| `resume-<yyyymmdd>.md` | **任意** | 本人が PDF を持っていて、 写しを残したい場合のみ |

軽量検品 (注力 PJ 3 件のみ採点) では portfolio.md だけで完結する。 フル実行 (10+ PJ 採点 + 跨領域分析) のときに integrated-cv を生成する。

#### 6.1 integrated-cv-<yyyy-mm>.md

**style-guide 準拠の読み物** として 9-13 章構成で書く。 視覚要素 (Mermaid + 表) を 20-30% 確保。

各章は対応する Layer 1-3 ファイルへリンク (重複記述しない、 §関連ドキュメント で集約)。

#### 6.2 project-portfolio-<yyyy-mm>.md

**転職サービス入力欄向け** に全 PJ を STAR + 設計判断 + 学び で記述。

##### フォーマット基準

詳細は `_shared/sources/recruitment-service-formats.md` を参照。 共通最小公倍数フォーマット (Findy / ビズリーチ / doda / LinkedIn / レバテック / LAPRAS / 転職ドラフト の横断):

```
### [カテゴリ / 役割] PJ タイトル
- 期間: YYYY-MM 〜 YYYY-MM
- 業界 / ドメイン / 職種 / 役割 / チーム規模 / フェーズ
- 利用技術: 言語 (バージョン) / FW / DB / インフラ
- 関連: リリース URL / 技術ブログ / OSS リポ
- スコア: 設計品質 N% / 人材レベル / 年収 anchor (Layer 3 から)

#### 概要 (1-2 文 / 50-100 字)
#### 課題 (1-2 文 / 50-100 字)
#### 行動 (3-4 文 / 100-200 字)
#### 設計判断の言語化 (任意 / メイン PJ のみ)
#### 成果 (1-3 文 / 50-150 字、 数値必須)
#### 学び (任意 / メイン PJ のみ)
```

##### メイン / トグルの 2 段化

| 表示 | 対象 | 厚み |
|---|---|---|
| **インライン展開** | 2 軸評価 75%+ / リーダー / 主担当 / 5 年継続 / 大規模数値根拠 / 上流貢献 | 500-800 字 (STAR + 設計判断 + 学び) |
| `<details>` トグル | メンバー役 / 補完案件 / アルバイト期 / 短期案件 / 学生時代 / 軽量 PJ | 200-300 字 (STAR まで) |

##### サービス pivot 表 (§0 に必ず置く)

| サービス | アピール pivot |
|---|---|
| Findy / Forkwell | 数値根拠 + GitHub 連携 |
| ビズリーチ / doda X | 戦略的キャリア + 事業貢献 |
| doda / エン転職 | 担当工程 + 規模 + 体言止め |
| LinkedIn | action verb + ATS キーワード |
| レバテック FL | 自走力 + 業務委託特性 |
| LAPRAS / Forkwell | アウトプット + Pinned Works |
| 転職ドラフト | 背景→課題→工夫→結果 + 野望欄 + 言語バージョン明記 |

##### 時系列順 + 章構造

時系列順 (古い → 新しい) に並べる:
- §1 学生期 (高等教育機関 / 卒研 等) — 時系列最上位
- §2 本業 (会社別 / フェーズ別)
- §3 副業 / 業務委託
- §4 個人プロジェクト (社会人期)
- §5 OSS 上流
- §6 ドメイン固有 (競技 / 認定 / 同人 等)
- §7 その他 (登壇 / 確定申告 等)
- §8 関連ドキュメント

### Phase 7: 検証 / 留意点記録

最終 CV / portfolio に次を明記:

- API 取得範囲の制約 (例: 「GitLab API は最新ページ依存、 古い時期は git log で補完」)
- 個人寄与割合の不確実性 (チーム PJ で本人担当が不明な範囲)
- 自己申告とアウトプットのトーン差 (謙遜表記など)
- 開業 / 廃業 / 契約終了など、 ステータス変化の正確な日付
- 役職の正確な範囲 (例: 「上位役職 (主催 / 統括) ではなく運営メンバー」)

## Inputs (skill 起動時に必須)

- userid (識別子)
- 基準日 (yyyy-mm-dd)
- 対象転職サービス (任意)

それ以外は Phase 1 でチェックリスト経由で収集する。

## Bundled resources

- `checklists/data-intake-checklist.md` — Phase 1 で提示する完全チェックリスト
- `templates/integrated-cv-template.md` — Layer 4 の統合 CV (読み物) 章スケルトン
- `templates/project-portfolio-template.md` — Layer 4 の転職サービス向け portfolio 章スケルトン (8 章 / メイン + トグル 2 段化)
- `templates/research-area-template.md` — Layer 1 の領域別ノート構造
- `templates/month-by-month-template.md` — Layer 2 月別表の構造 + 出典タグ凡例

## Constraints

- **言語**: 日本語アウトプット (Global Instructions)
- **Commit**: Conventional Commits (commit するかは別途確認)
- **Context separation**: work / side / hobby を混在させない
- **揮発性ディレクトリ依存禁止**: `~/Downloads/` 等のファイルは必ず `<userid>/assets/` にコピー
- **secret 出力禁止**: 月次面談 Notion / Slack DM 等から取得しても、 社外秘事項 (顧客名 / 売上数値 / NDA 情報) は CV / portfolio に書かない。 マスク対応 (社内クラス名 → 機能名 / 社内アプリ名 → 「特定の社内アプリ」 等)
- **事実ベース**: 各記述は出典に紐付ける。 推測は「(推定)」 マークを付け、 別 §「留意点」 にも書く
- **本人寄与判定**: チーム PJ で本人 user_id を **必ず確定** (`notion-get-users self` / `git log --author`) してから採点する。 参加 ≠ 担当 / 推測 ID で採点しない
- **役職の正確性**: 「就任」「主担当」 等のステータスは **本人確認後に記述**。 推測で格上げしない

## Non-goals

- 履歴書の体裁 (氏名 / 印鑑欄 / 規定フォント) 生成 — 別 skill
- 公的書類 (確定申告 / 雇用契約書) の文面起こし — 別 skill
- 採用面接 Q&A の自動生成 — 別 skill
- 給与交渉文書 — 別 skill

## Lessons learned (失敗事例集)

過去の skill 利用で起きた失敗パターンを記録する。 各 Phase の冒頭で該当グループを必ず確認する。 LL 番号は発見順 (= 他文書からの参照互換性のため固定)、 並び順は **対応 Phase 別** に整理。

### Phase 0 / 1 で防ぐ (identify / 投入要求)

#### LL3: 日付イベントは **種別を必ず確認** (開業 / 廃業 / 提出 / 適用)

**失敗**: 確定申告書に書かれた「個人事業 開業日: YYYY-MM-DD」 を「廃業日」 と推測。 portfolio.md / metadata に「個人事業廃業日 / 廃業後も契約継続」 と書いた。 ユーザー指摘で「廃業ではなく開業届提出日 / 実際の開業日はその数ヶ月前」 と判明。 関連ファイルを意味反転で修正する羽目に。

**対策**:
- 日付ファクトは「**何の日付か**」 を必ず本人確認: 開業日 / 開業届提出日 / 廃業日 / 廃業届提出日 / 契約成立日 / リリース日 / 退職日 等
- 確定申告書 / 会社書類の日付は **書類上の日付** であって、 実際のイベント日と異なる場合がある

#### LL12: 期間 / 役職の境界は **入社時期 + リーダー登用日 + 案件継続期間** で確定

**失敗**: 「ある時期以降は当該案件に join していない」 という本人申告を確認していなかったため、 離脱後の期間の Notion 議事録 (= 別人の業務) を本人業務として採点していた。

**対策**:
- 雇用形態の切り替え日 + 案件ごとの開始 / 終了日 を **明示的に確認**
- 「フルコミット期 / メンバー期 / 抜けた期 / PM 兼任期」 のような **関与モード** も時期別に確認

#### LL13: prompt に投入情報が含まれている場合は Phase 1 スキップ

**失敗 (検品で発見)**: skill 起動時の prompt に既に「### A. GitHub: ... ### F. 学校: ...」 のような形で全カテゴリが含まれているケースで、 Claude が再度チェックリストを提示する挙動が冗長だった。

**対策**:
- prompt 内に明示的セクション分け (A-F) があれば Phase 1 を skip
- 部分提供の場合は不足カテゴリのみ 1 回追加質問
- 「無し / 該当外」 明示済カテゴリは追加質問しない

### Phase 2 で防ぐ (自律取得 + 本人 user_id 確定)

#### LL1: 本人 user_id を **推測で使ってはいけない**

**失敗**: 社内 Notion DB の view から取得した「参加者」 フィールドに頻出していた user_id を「本人だろう」 と推測し、 SQL クエリの `WHERE 作成者 = ?` に使用。 結果として **別人の業務を「本人主担当」 として portfolio.md に追記**。 ユーザー指摘で削除する羽目に。

**対策**:
- 必ず `notion-get-users` に `user_id: "self"` を渡して本人 user_id を確定してから query
- 「参加者」 や「メンション先」 の頻出 user_id は **別人の可能性が高い** (組織によってはメンバー全員が常時参加する定例が多いため)

#### LL5: GitHub 公開リポは **全件網羅** (gh repo list で確認)

**失敗**: 本人 GitHub の公開リポ数十件のうち、 主要 PJ (アピール材料になる代表 PJ) は触れたが、 **軽量 PJ (学生期最終課題 / 社会人 1 年目スニペット 等)** に触れず。 ユーザー指摘で初めて分析対象に追加。

**対策**:
- `gh repo list <user> --limit 100` で全公開リポを取得して **規模感を一覧表に整理**
- 軽量 PJ (1 commit / 数 KB) も最低限「(レベル感の証跡 / アピール材料ではない)」 として把握しておく
- 学生期初期の課題作品など、 **当時のレベル感を示す材料** はマトリクス採点で活用できる

#### LL11: 議事録作成者 ≠ 本人主担当 (Created by フィルタの限界)

**失敗**: 本人 user_id の Created by で Notion query して N 件取得。 これを「本人主担当 PJ」 と判断するのは正しい方向だが、 「議事録作成 = 主担当」 ではなく「議事録作成 = 主催 or 議事録係」 の場合もある (1on1 / 雑談記録 / 純粋な進捗確認会など)。

**対策**:
- Created by + 定例議事録テンプレ + 専用タグ で「定常的な業務」 を判定
- 「単発の進捗確認 / 雑談」 は除外して採点する
- 議事録の AI 要約から「設計判断 / 数値根拠 / 失敗事例の言語化」 が読み取れるものを優先

### Phase 5 で防ぐ (rubric 選択 / 適用範囲)

#### LL7: 学生期 PJ にも **コード設計の rubric が必要**

**失敗**: 学生期の複数 PJ をひとまとめのバッチ評価で済ませていた。 ユーザー指摘「注力 PJ (最大 3 件) は当時の設計判断と利用技術を評価する最適な標本なので、 git 分析と中のコード設計についても rubric すべき」 で `student-era-design-quality-rubric.md` を新規作成。

**対策**:
- 学生期 PJ に対しては `student-era-design-quality-rubric.md` を適用 (10 カテゴリ / 学年別)
- git 分析 (コミット数 / ファイル数 / クラス構成) + コード本文の確認 (パッケージ階層 / 命名 / enum 使用 / リファクタ明示) で採点

#### LL8: 同ドメイン内の役割差を捉える **新規 rubric を躊躇わない**

**失敗**: ニッチドメインで既存 rubric を流用したが、 同ドメイン内の別役割視点 (制作側の rubric しか無く、 利用者 / プレイヤー側の rubric が欠落) をカバーできず、 ユーザー指摘で新規 rubric を作成する羽目に。

**対策**:
- 既存 rubric でカバーできないドメインや役割があれば **新規作成** (skill 内で勝手に作らず、 ユーザー確認 → 新規作成)
- 「制作者 / 認定者 / 利用者 (プレイヤー / ユーザー)」 のように同ドメイン内で **役割別 rubric** が必要なケースがある

#### LL14: rubric 適用範囲を絞る (全 PJ 採点は不要 / LL5 と対極)

**失敗 (検品で発見)**: skill 仕様で「全 PJ について rubric を選択」 と書いていたため、 軽量 PJ (1 commit のスニペット / ゲームスコア記録系 PJ 等) にも採点を試みる挙動が想定されてしまっていた。 実際は採点不要。

**対策**:
- 採点対象は注力 PJ (F-4 指定 / 最大 3 件) + メイン PJ (70%+ 想定) + 特殊事例 のみ
- 軽量 PJ は portfolio.md のトグルに 100-200 字で言及するだけ
- **LL5 (網羅取得) と両立**: 取得は全件、 採点は絞る

### Phase 6 で防ぐ (Layer 4 出力)

#### LL6: 学生期 PJ は **章を分離** (個人プロジェクトと混ぜない)

**失敗**: portfolio.md §3「個人プロジェクト」 に学生期の教材系 PJ / 学校コンテスト出展作品 / Web 創作 / 学術系卒研 を全部混ぜていた。 ユーザー指摘「個人開発と学生時代の課題制作・部活動におけるコンテスト出場は章を分けたい」 で章分離。

**対策**:
- 章構成は **時系列順**: §1 学生期 (高等教育機関 / 卒研 等) → §2 本業 → §3 副業 → §4 個人 PJ (社会人期)
- 学生期は「課題制作 + 部活動 + コンテスト出場 + 卒研」 を含む独立章

#### LL9: フォーマット基準は **外部リサーチを先に**

**失敗**: portfolio.md 初版を「本人の resume PDF の形式」 を踏襲して書いた。 ユーザー指摘「ポートフォリオの記述基準は各社転職サービスの推奨する書き方を丁寧にリサーチしてから着手していますか？」 で 7 サービス (Findy / ビズリーチ / doda / LinkedIn / レバテック / LAPRAS / 転職ドラフト) を WebFetch リサーチ → `recruitment-service-formats.md` に集約 → portfolio.md を STAR 構造で書き直し。

**対策**:
- portfolio.md フォーマットは **対象転職サービスを Phase 0 で確認** + `_shared/sources/recruitment-service-formats.md` (既存リサーチ集) を参照
- 該当サービスが既存リサーチに無ければ Phase 6 で **追加 WebFetch リサーチ** を実施してから書く

#### LL15: 成果物の優先順位を明示する

**失敗 (検品で発見)**: 軽量検品 (注力 PJ 3 件のみ採点) のケースで integrated-cv も生成しようとして時間を浪費した。

**対策**:
- `project-portfolio-<yyyy-mm>.md` のみ必須
- `integrated-cv-<yyyy-mm>.md` は Layer 3 フル実行時 (5+ PJ 採点済) のみ生成
- `resume-<yyyymmdd>.md` は本人が PDF を持っている場合のみ写しを残す

### Phase 7 で防ぐ (検証 / 整合性)

#### LL2: 役職は **本人確認後に書く** (推測で格上げしない)

**失敗**: 議事録に上位ロールとして本人名が記載されていたため、 portfolio.md に「上位役職就任 (大規模統括相当)」 と書いた。 ユーザー指摘「そのロールには就かない、 運営チームのメンバーとして参加するだけ」 で全削除 + 格下げ。

**対策**:
- 役職 (主担当 / 主催 / リーダー / 就任 / マネージャー) は **本人申告ベースで確定**
- 議事録の参加者リスト ≠ 役職割当。 「議事録に名前があるから上位ロール」 のような推測は危険

#### LL4: 集計数値は **範囲を明示** (master ブランチ単一 / 全ブランチ / 期間)

**失敗**: 業務リポのコミット数を暫定集計値のまま記載。 ブランチを含めた over-count を含んでおり、 ユーザー指摘で再計測したところ master ブランチ単一の値とは数十 % のズレがあった。

**対策**:
- コミット数 / PR 数 / マージ率 などの数値は「集計範囲」 を必ず明示
  - `master ブランチ単一` / `全ブランチ含む` / `特定期間 (YYYY-MM 〜 YYYY-MM)` / `アカウント別`
- 暫定値は「(推定 / 再計測待ち)」 マークを付ける

#### LL10: 整合性ファイル波及確認

**失敗**: portfolio.md の特定項目を修正した時、 関連する `_research-areas/<area>.md` / `_intermediate/career-timeline.md` / `_metadata.yaml` / `integrated-cv-<yyyy-mm>.md` の整合性を取り忘れ、 数 PJ で記述齟齬が発生。

**対策**:
- ある情報を 1 箇所変更したら、 **下記 5 ファイルへの波及を必ずチェック**:
  - `portfolio.md` (PJ 詳細)
  - `integrated-cv-<yyyy-mm>.md` (読み物 / 11 章 統合)
  - `_research-areas/<area>.md` (領域別詳細)
  - `_intermediate/career-timeline.md` (時系列年表)
  - `_pj-evaluations/_metadata.yaml` (PJ メタデータ集約)
- 役職変更 / 期間変更 / 数値変更は特に注意
