# Research Domain — 研究系エージェントパッケージ

## 概要
研究・実験・分析タスクに特化したエージェント群を定義します。
常時配置する foundation に対し、本パッケージは研究タスク発生時にワークスペースに追加します。

## 含まれるエージェント
| ID | 名称 | 役割 |
|----|------|------|
| EXD | Experiment Designer | 実験設計・評価指標定義 |
| ANL | Analyst | 調査/実験結果の分析・最適解導出 |

## 前提
- foundation パッケージがワークスペースに存在すること
- Orchestrator（foundation）が統制を行う