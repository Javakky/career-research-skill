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
    tax-return-<yyyy>.md (任意 / 個人事業は暦年 = 1/1-12/31 / 法人は事業年度)
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
- **tax / 副業 / 個人事業 / 業務委託 context が含まれる場合** (LL3 強化): F-1 (開業日 / 開業届提出日 / 廃業日 / 廃業届提出日) は **必須追加質問**。 partial-provision で F-1 が空 / 未記載なら、 「無し / 該当外」 明示済でない限り 1 回追加質問する (推測では確定できないため)

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
| `tax-return-<yyyy>.md` (任意) | 確定申告サマリ (副業 anchor 用) — **個人事業は暦年 (1/1-12/31)**、 法人は事業年度。 `fy` 接頭辞は誤誘導なので使わない (LL17) |

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

#### 5.4 単独 PJ 採点 / 簡易採点モードの注意 (LL18)

軽量検品 (注力 PJ 1-2 件のみ / Layer 3 をフル展開しない場合) で portfolio.md にスコアを書くとき、 anchor 不明確のまま % を出すと誤読される。 次のガイドに従う:

- **% は比較 anchor を必ず明示**: 「設計品質 75%」 だけでは読めない。 「(本人の本業 Scala PJ を 80% とした場合の) 75%」 / 「(WordPress 案件 median を 60% とした場合の) 75%」 のように **比較対象を明示**する。 anchor が組めないなら数値を出さない
- **年収 anchor は実態原則**: 副業実収入 (例: 200-300 万 / 年) を main anchor として書く。 フルタイム換算 (例: 400-600 万 / 年) を出す場合は **補助として並列表記** し、 「(実態は月 N h 程度 / 換算は機会値)」 と注記する。 換算値だけを書くと過大評価になる
- **質的表現で代替する選択肢**: anchor が組めない単独 PJ では、 数値の代わりに「Senior 中堅相当」 等の質的表現 + 強み / 弱みの列挙で代替する

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
- **一次資料の要点転記** (LL19): PDF / .md / pptx 等の一次資料は assets/ にコピーした上で、 **要点 (≦20 行程度 / 表 / 手順 / 固有名詞 / 原文の立ち位置宣言)** を該当 research-area ノート本文に転記する。 assets/ への保全だけでは narrative が組めない (例: 12 ステップフロー PDF は本文に転記しないと §観察 「権限スコープの段階拡張」 が書けない / 改善提案書の「(例) 当時は外部視点である」 宣言は原文引用すべき)
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

過去の skill 利用で起きた失敗パターンと対策は [lessons-learned.md](lessons-learned.md) に集約。 各 Phase の冒頭で該当グループを必ず確認する。 LL 番号は発見順 (= 他文書からの参照互換性のため固定)、 並び順は **対応 Phase 別** に整理されている。

Phase 別の概要 index:

| Phase | LL | 主題 |
|---|---|---|
| Phase 0 / 1 (identify / 投入要求) | LL3 | 日付イベントは種別を必ず確認 (開業 / 廃業 / 提出 / 適用) |
| | LL12 | 期間 / 役職の境界は入社時期 + リーダー登用日 + 案件継続期間で確定 |
| | LL13 | prompt に投入情報が含まれている場合は Phase 1 スキップ (tax/副業 context では F-1 必須追加質問) |
| | LL16 | 契約前のアウトプット系譜は §前史 として独立章にする |
| Phase 2 (自律取得 + 本人 user_id 確定) | LL1 | 本人 user_id を推測で使ってはいけない |
| | LL5 | GitHub 公開リポは全件網羅 (gh repo list で確認) |
| | LL11 | 議事録作成者 ≠ 本人主担当 (Created by フィルタの限界) |
| | LL19 | 一次資料の要点を research-area 本文に転記 (assets/ はバックアップ) |
| Phase 4 (中間成果物) | LL17 | 個人事業の確定申告は暦年 (`fy` 接頭辞は誤誘導) |
| Phase 5 (rubric 選択 / 適用範囲) | LL7 | 学生期 PJ にもコード設計の rubric が必要 |
| | LL8 | 同ドメイン内の役割差を捉える新規 rubric を躊躇わない |
| | LL14 | rubric 適用範囲を絞る (全 PJ 採点は不要 / LL5 と対極) |
| | LL18 | 単独 PJ 採点 / 簡易採点モードでは anchor を必ず明示 |
| Phase 6 (Layer 4 出力) | LL6 | 学生期 PJ は章を分離 (個人プロジェクトと混ぜない) |
| | LL9 | フォーマット基準は外部リサーチを先に |
| | LL15 | 成果物の優先順位を明示する |
| Phase 7 (検証 / 整合性) | LL2 | 役職は本人確認後に書く (推測で格上げしない) |
| | LL4 | 集計数値は範囲を明示 (master ブランチ単一 / 全ブランチ / 期間) |
| | LL10 | 整合性ファイル波及確認 |

詳細 (失敗の経緯 + 対策) は [lessons-learned.md](lessons-learned.md) の該当セクションを参照。
