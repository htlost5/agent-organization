---
agent: SRC
task_id: TASK-001
date: 2026-06-11
status: approved
category: shared
destination: shared/search/decisions/search/SD-001_positioning-tech-survey.md
related:
  - "[TASK-001_positioning-tech-survey](../../../tasks/active/TASK-001_positioning-tech-survey.md)"
  - "[SR-001](../../specs/search-results/SR-001_positioning-tech-comprehensive.md)"
tags:
  - SRC
  - positioning-technology
  - TASK-001
  - survey-plan
---

# SD-001: 位置情報測定技術 調査方針

## 調査目的
SL2_Research プロジェクトの初回タスクとして、位置情報測定技術（positioning/localization technology）に関する既存の全手法を網羅的に調査する。

## カテゴリと優先順位

| 優先度 | カテゴリ | キーワード計画 |
|--------|----------|---------------|
| P0 | 全カテゴリ俯瞰 | "positioning localization technology comprehensive survey", "indoor outdoor positioning techniques survey" |
| P0 | 衛星測位 (GNSS) | "GNSS GPS GLONASS Galileo BeiDou survey", "RTK PPP precise point positioning" |
| P0 | 地上無線測位 | "WiFi positioning BLE UWB RFID ZigBee LoRa survey", "indoor positioning radio technology" |
| P0 | 慣性航法 (INS/IMU) | "pedestrian dead reckoning PDR IMU inertial navigation survey" |
| P1 | 視覚測位 (Visual) | "Visual SLAM visual odometry LiDAR SLAM camera positioning survey" |
| P1 | ハイブリッド/融合測位 | "sensor fusion Kalman filter positioning fusion survey", "multi-sensor fusion localization" |
| P1 | 測位精度向上技術 | "RTK PPP DGPS A-GPS multipath mitigation error correction" |
| P2 | 音響測位 | "acoustic positioning ultrasonic indoor survey underwater acoustic navigation" |
| P2 | 磁気測位 | "magnetic positioning geomagnetic fingerprinting indoor survey" |
| P2 | 新興技術 | "machine learning deep learning positioning fingerprinting", "5G 6G positioning localization", "quantum positioning navigation" |
| P2 | 特殊環境測位 | "underwater positioning, underground positioning, indoor positioning challenges" |
| P3 | 協調測位 | "collaborative positioning crowdsourcing cooperative localization" |
| P3 | VLC/可視光測位 | "visible light positioning VLC indoor positioning LiFi survey" |
| P3 | LPWAN測位 | "LoRa LPWAN positioning localization IoT survey" |

## 使用検索モード

| モード | MCP | 用途 |
|--------|-----|------|
| 研究系 | paper-search (arxiv, semantic, openalex, core) | サーベイ論文・学術論文の網羅的収集 |
| 技術系 | github | オープンソース実装・ツールの発見 |
| 汎用 | web-search | 技術ブログ・業界レポート（失敗時は代替手段） |

## 検索戦略
1. **初回**: 各カテゴリを代表するサーベイ論文を横断検索
2. **深掘り**: 情報不足カテゴリに対して追加キーワードで再検索
3. **GitHub**: 主要オープンソースプロジェクトの発見
4. **上限管理**: 基本5回 + 追加3回、同一キーワード連続2回まで