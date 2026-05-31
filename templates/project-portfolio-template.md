# プロジェクト経歴ポートフォリオ (<yyyy-mm-dd> 時点)

転職サービス / エージェントの「プロジェクト経歴入力欄」 に貼れる粒度・文体で全 PJ を記述したもの。基準日 <yyyy-mm-dd>。氏名表記は <ハンドル>。

## 0. 読み方

PJ は **時系列順** に並べる。 章は以下の構造:

- §1 学生時代 (高等教育機関 / 卒研 等) — 時系列最上位
- §2 本業 (会社別 / フェーズ別)
- §3 副業 / 業務委託
- §4 個人プロジェクト (社会人期)
- §5 OSS 上流コントリビュート (継続)
- §6 ドメイン固有 (競技 / 認定 / 教育 / 同人 等)
- §7 その他 (登壇 / 技術発信 / 確定申告)
- §8 関連ドキュメント

各 PJ は **アピール軸の代表性** で 2 段化:

| 表示 | 対象 | 厚み |
|---|---|---|
| **インライン展開 (メイン)** | 2 軸評価 70%+ / リーダー / 主担当 / 5 年継続 / 大規模数値根拠 / 上流貢献 / **本人特に注力した PJ** | 500-800 字 (STAR + 設計判断 + 学び) |
| `<details>` トグル | メンバー役 / 補完案件 / アルバイト期 / 短期案件 / 軽量 PJ | 200-300 字 (STAR まで) |

各 PJ は **STAR フレーム** (概要 / 課題 / 行動 / 成果) に従う。 メイン PJ には「設計判断の言語化」「学び」も加える。 フォーマット基準は [recruitment-service-formats.md](../../_shared/sources/recruitment-service-formats.md) を参照。

| サービス | アピール pivot |
|---|---|
| Findy / Forkwell | 数値根拠 + GitHub 連携 |
| ビズリーチ / doda X | 戦略的キャリア + 事業貢献 |
| doda / エン転職 | 担当工程 + 規模 + 体言止め |
| LinkedIn | action verb + ATS キーワード |
| レバテック FL | 自走力 + 業務委託特性 |
| LAPRAS / Forkwell | アウトプット + Pinned Works |
| 転職ドラフト | 背景→課題→工夫→結果 + 野望欄 + 言語バージョン明記 |

PJ 重み付けは [<userid>-evaluation-<yyyy-qN>.md](<userid>-evaluation-<yyyy-qN>.md) の 2 軸スコアに従う。 スコア source は [_pj-evaluations/_metadata.yaml](_pj-evaluations/_metadata.yaml)。

---

## 1. 学生時代 (<学校> <YYYY-MM> 〜 <YYYY-MM>)

<N> 年間に <N> 件のコンテスト / 部活動 / チーム制作 PJ + <N> 件の卒業研究 + <継続教材> を運用した。 注力 PJ (<最大 3 件>) を中心にメイン展開する。

> [!IMPORTANT]
> 学生時代は **社会人期の個人プロジェクトと章を分ける** (LL6)。
> 「課題制作 / 部活動 / コンテスト出場 / 卒研」 を含む独立章として扱う。

<details>
<summary>[学年 / 種別] <PJ 名> — <短い説明> (期間)</summary>

- 役割: <個人 / チーム / 個人開発>
- 利用技術: <言語 (バージョン) / FW / ハードウェア>
- 関連: <GitHub リポ URL> / <プレゼン pptx>
- スコア: 設計品質 N% / 学年 / 社会人換算 anchor (任意)

#### 概要 (1-2 文)

#### 課題 (1-2 文)

#### 行動 (2-3 文)

#### 成果 (1-2 文 / 数値根拠 / 受賞 / 後年への波及)

</details>

### [学年 / 注力 PJ] <PJ 名> — <短い説明> (期間)

(注力 PJ はメイン展開 = 500-800 字 / 概要 + 課題 + 行動 + 設計判断 + 成果 + 学び)

---

## 2. <本業 / 会社 A> (<YYYY-MM> 〜 現在)

(本業の概要 1-2 段落 / 関与期間 / 雇用形態の変遷 / ブランクがあれば明示)

> [!IMPORTANT]
> 本業の中で長期 (3 年以上) のブランクを挟む場合は **領域は同じだが時期で分けて記述** (例: 領域 A に集中 → 領域 B 復帰)。 関与モードを明示 (フルコミット期 / メンバー期 / 抜けた期 / PM 兼任期 / LL12)。

<details>
<summary>[アルバイト期 / インターン期] <PJ 名> (期間)</summary>

(STAR で 200-300 字)

</details>

### [雇用形態 / 役割] <PJ 名> (期間)

- 期間: YYYY-MM 〜 YYYY-MM
- 業界 / ドメイン: <業界>
- 職種: <職種>
- 役割: **<リーダー / 主担当 / メンバー>**
- チーム規模: 1-N 人
- フェーズ: <要件定義 / 設計 / 実装 / 運用>
- 利用技術: <言語 (バージョン) / FW / DB / インフラ>
- 関連: <リリース URL> / <技術ブログ> / <OSS リポ>
- スコア: 設計品質 N% / <Senior 等> / X-Y 万円 anchor

#### 概要 (1-2 文 / 50-100 字)
#### 課題 (1-2 文 / 50-100 字)
#### 行動 (3-4 文 / 100-200 字)
#### 設計判断の言語化 (主担当案件のみ / 1-2 段落)
#### 成果 (1-3 文 / 50-150 字、 数値必須)
#### 学び (主担当案件のみ / 1-2 文)

---

## 3. <副業 / 業務委託先> (<YYYY-MM> 〜 / 業務委託)

(該当があれば / 4 段階の温め期間 + 契約後の継続関与)

<details>
<summary>[前史 / 個人] <PJ 名> (期間)</summary>

(契約前の温め期間 / 個人アウトプット)

</details>

### [業務委託 / 役割] <PJ 名> (期間)

(STAR + 設計判断 + 学び)

---

## 4. 個人プロジェクト (社会人期)

社会人期 (<YYYY-MM> 〜) の個人プロジェクトを記述する。 学生時代の PJ は §1 を参照。

> [!IMPORTANT]
> **学生時代の PJ は §1 を参照** と明記 (LL6 / 章を混ぜない)。

<details>
<summary>[個人 OSS] <PJ 名> (期間)</summary>

(STAR で 200-300 字)

</details>

### [個人 PJ オーナー / 注力 PJ] <PJ 名> (期間)

(注力 PJ はメイン展開 = 500-800 字)

---

## 5. OSS 上流コントリビュート (継続)

<details>
<summary>[継続 N 年] <OSS 名> 上流貢献 (期間)</summary>

(STAR で 200-300 字 / PR 数 / merged 率 / 期間)

</details>

### [業務由来 OSS / メンテナンス] <OSS 名> (期間)

(メイン PJ で 500-800 字)

---

## 6. <ドメイン固有活動 A> (任意)

例: 競技 (戦績 / 大会運営 / メディア運営) / 認定 / 教育 / 同人。

### [役割] <PJ 名> (期間)

(STAR + 設計判断 + 学び)

<details>
<summary>[役割] <サブ PJ 名> (期間)</summary>

(STAR で 200-300 字)

</details>

---

## 7. その他 (登壇 / 技術発信 / 確定申告)

<details>
<summary>[コミュニティ] 登壇 / 技術発信 (期間)</summary>

- 媒体: <ブログ N 本> / <Qiita N 本> / <Zenn N 本> / <note N 本>
- 登壇: <イベント名 + 時期>

(STAR で 200-300 字)

</details>

<details>
<summary>[個人事業 / 確定申告] FY<YYYY> 確定申告サマリ (該当があれば)</summary>

| 項目 | 値 |
|---|---:|
| <YYYY> 年分 給与収入 | <X> 円 (<会社>) |
| <YYYY> 年分 営業等 (事業) 収入 | <Y> 円 (<契約先>) |
| <YYYY> 年分 還付額 | <Z> 円 |
| 集計期間 | 個人事業は暦年 (<YYYY>-01-01 〜 <YYYY>-12-31) / 法人は事業年度 |
| 個人事業 開業日 | <YYYY-MM-DD> |
| 個人事業 開業届提出日 | <YYYY-MM-DD> (LL3: 別の日付の可能性に注意) |

詳細は [_intermediate/tax-return-<yyyy>.md](_intermediate/tax-return-<yyyy>.md) を参照。 個人事業は暦年 (1/1-12/31)、 法人は事業年度で集計する (LL17)。

</details>

---

## 8. 関連ドキュメント

- 統合 CV (読み物): [integrated-cv-<yyyy-mm>.md](integrated-cv-<yyyy-mm>.md)
- 2 軸評価レポート: [<userid>-evaluation-<yyyy-qN>.md](<userid>-evaluation-<yyyy-qN>.md)
- 職務経歴書 PDF 写し: [resume-<yyyymmdd>.md](resume-<yyyymmdd>.md)
- フォーマット基準: [recruitment-service-formats.md](../../_shared/sources/recruitment-service-formats.md)
- 領域別リサーチ: `_research-areas/` ([README](README.md) からナビゲート)
- 中間成果物: `_intermediate/` (引用専用集計データ)
- PJ 別 2 軸評価詳細: `_pj-evaluations/` + `_metadata.yaml`
- rubric 適用例: `_rubric-applied-examples/` (技術 / サービス / ML / ドメイン 別)
