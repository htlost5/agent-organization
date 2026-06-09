# Code Domain — Implementation Agent Rules

このファイルは実装系エージェント（DEV, ARC, IMP, REV, TST, REL）に適用されるドメイン固有ルールを定義します。
共通ルールは foundation の `.github/copilot-instructions.md` を参照してください。

---

## 1. 実装系フロー詳細

### 標準フロー
`ORC → (RES) → DEV → ARC → IMP → REV → TST → REL → ORC`

### フロー短縮（foundation の `ops_config.yml` 参照）
| 種別 | フロー | 適用条件 |
|------|--------|----------|
| **trivial** | `IMP → TST → REL` | タイポ修正・単純設定変更・1行パッチ |
| **simple** | `DEV → IMP → REV → TST → REL` | 小規模ユーティリティ追加・軽微な機能追加 |
| **standard** | `RES → DEV → ARC → IMP → REV → TST → REL` | 通常の実装タスク |

---

## 2. チェーン委譲モード

### 発動条件（全5条件）
1. タスクカテゴリが「実装」である
2. 要件が明確で、DEV の判断余地が小さい（新規設計より既存拡張が中心）
3. 既知の技術スタックを使用する
4. ORC の確信度が 85%（中）以上
5. 研究要素を含まない

### 動作
- ORC は `DEV→ARC→IMP→REV→TST→REL` を 1回のバッチ指示で起動
- 各エージェントは foundation の `handoff_protocol.instructions.md` に従い直接ハンドオフ
- CRITICAL 差し戻し（REV→IMP, TST→IMP）発生時のみ ORC にエスカレーション
- REL 完了後、ORC に最終報告

---

## 3. 実装系エージェント間インターフェース

| from | to | 引き継ぎ成果物 |
|------|-----|---------------|
| DEV | ARC | 要件定義書(`REQ-XXX`), 設計決定(`DD-XXX`) |
| ARC | IMP | IF仕様書(`IF-XXX`), アーキテクチャ決定(`ADR-XXX`) |
| IMP | REV | 実装コード, 実装ログ |
| REV | TST | レビュー済みコード, レビューログ |
| TST | REL | テスト合格コード, テストログ |
| REL | ORC | リリース情報, コミットハッシュ |

---

## 4. 実装系固有のエラーハンドリング

詳細は foundation の `config/ops_config.yml` を参照。
実装系に特に関連するセクション: `implementation`, `error_handling.imp_*`, `error_handling.rev_*`, `error_handling.tst_*`, `error_handling.rel_*`