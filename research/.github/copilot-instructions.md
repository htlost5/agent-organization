# Research Domain — Research Agent Rules

このファイルは研究系エージェント（EXD, ANL）に適用されるドメイン固有ルールを定義します。
共通ルールは foundation の `.github/copilot-instructions.md` を参照してください。

---

## 1. 研究系フロー詳細

### 標準フロー
`ORC → (RES) → EXD → ANL → ORC`

### バリエーション
| 種別 | フロー | 適用条件 |
|------|--------|----------|
| **調査のみ** | `RES → ANL → ORC` | 情報収集・分析のみ（実験不要） |
| **完全研究** | `RES → EXD → ANL → ORC` | 実験設計から分析まで |

---

## 2. 研究系エージェント間インターフェース

| from | to | 引き継ぎ成果物 |
|------|-----|---------------|
| RES | EXD | 調査結果（key findings, sources） |
| EXD | ANL | 実験設計書(`EXP-XXX`), 実験結果(`ER-XXX`), 実験ログ |
| ANL | ORC | 分析レポート, 最適解推奨 |

---

## 3. 研究系固有の制約

- 実験の再現性を確保するため、実験条件・環境・バージョン情報を必ず記録する
- 定量評価が困難な場合は、定性評価の基準を事前に EXD が定義する
- ANL は確信度が低い場合、追加データ収集を ORC に推奨する

---

## 4. 研究系固有のエラーハンドリング

詳細は foundation の `config/ops_config.yml` を参照。
研究系に特に関連するセクション: `error_handling.exd_*`, `error_handling.anl_*`