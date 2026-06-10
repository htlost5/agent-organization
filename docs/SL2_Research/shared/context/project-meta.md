---
agent: SRC
task_id: TASK-001
date: 2026-06-11
status: approved
category: shared
destination: shared/context/project-meta.md
related:
  - "[TASK-001_positioning-tech-survey](../tasks/active/TASK-001_positioning-tech-survey.md)"
tags:
  - SRC
  - TASK-001
  - project-meta
---

# SL2_Research — プロジェクトメタ情報

## 基本情報
- **プロジェクト名**: SL2_Research
- **作成日**: 2026-06-11
- **ドメイン**: research + search
- **管理エージェント**: ORC (Orchestrator)

## 目的
位置情報測定（positioning/localization）技術に関する先行研究の包括的調査。
初回タスクとして、衛星測位・地上無線・慣性航法・視覚・音響・磁気・ハイブリッド融合・特殊環境・新興技術・精度向上技術の10カテゴリを網羅する。

## ステークホルダー
- ORC: 進捗管理・完了判定
- SRC: 調査実行（本タスク）
- EXD (将来的): 実験設計
- ANL (将来的): 分析

## 成果物一覧
| ファイル | パス | 説明 |
|----------|------|------|
| SD-001 | shared/search/decisions/search/SD-001_positioning-tech-survey.md | 調査方針 |
| SR-001 | shared/search/specs/search-results/SR-001_positioning-tech-comprehensive.md | 包括的調査結果 |
| 検索ログ | logs/search/2026-06-11_SRC_positioning-tech-survey.md | 検索過程の全記録 |
| project-meta | shared/context/project-meta.md | 本ファイル |
| glossary | shared/context/glossary.md | 用語集 |

## 拡張予定
- 実装タスク（位置情報測定システム構築）時の code ドメイン追加
- 実験タスク（性能評価・比較）時の research ドメイン追加