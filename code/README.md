# Code Domain — 実装系エージェントパッケージ

## 概要
コーディング・実装タスクに特化したエージェント群を定義します。
常時配置する foundation に対し、本パッケージは実装タスク発生時にワークスペースに追加します。

## 含まれるエージェント
| ID | 名称 | 役割 |
|----|------|------|
| DEV | DevPlanner | 要件分析・機能仕様決定 |
| ARC | Architect | アーキテクチャ設計・技術スタック選定 |
| IMP | Implementer | コード実装・デバッグ |
| REV | Reviewer | コードレビュー・セキュリティチェック |
| TST | Tester | テスト実行・合否判定 |
| REL | Release Manager | git管理・ビルド・リリース |

## 前提
- foundation パッケージがワークスペースに存在すること
- Orchestrator（foundation）が統制を行う