---
agent: SRC
task_id: TASK-001
date: 2026-06-11
status: approved
category: shared
destination: shared/context/glossary.md
related:
  - "[TASK-001_positioning-tech-survey](../tasks/active/TASK-001_positioning-tech-survey.md)"
tags:
  - SRC
  - TASK-001
  - glossary
---

# 位置情報測定 技術用語集

## A
| 用語 | 定義 |
|------|------|
| **A-GPS (Assisted GPS)** | サーバから衛星軌道情報を取得しTTFF（初回測位時間）を短縮する技術 |
| **AoA (Angle of Arrival)** | アンテナアレイで信号到来角度を測定する測位手法 |
| **アンビギュイティ解決** | 搬送波位相測位における整数値不確定性（N）を解決する処理 |

## B
| 用語 | 定義 |
|------|------|
| **BLE (Bluetooth Low Energy)** | 低消費電力Bluetooth規格。ビーコンを用いた近接検出・測位に利用 |
| **BeiDou** | 中国の衛星測位システム（BDS）。35機構成 |

## C
| 用語 | 定義 |
|------|------|
| **CSI (Channel State Information)** | WiFiのチャネル状態情報。RSSIより詳細な伝搬路情報で高精度測位が可能 |
| **CKF (Cubature Kalman Filter)** | キューバチャー則を用いた数値積分近似カルマンフィルタ |

## D
| 用語 | 定義 |
|------|------|
| **DGPS (Differential GPS)** | 基準局が計算した測位誤差補正値を移動局に配信する方式。精度1-3m |
| **DR (Dead Reckoning)** | 既知の位置から速度・方位の積分で現在位置を推定する手法 |
| **DTL (Dynamic Target Localization)** | 移動体（歩行者・ロボット・UAV等）のリアルタイム追跡測位 |

## E
| 用語 | 定義 |
|------|------|
| **EKF (Extended Kalman Filter)** | 非線形システムを線形近似して適用する拡張カルマンフィルタ |

## F
| 用語 | 定義 |
|------|------|
| **フィンガープリンティング** | 事前計測した電波・磁気特性マップと現在値を照合する測位手法 |
| **FGO (Factor Graph Optimization)** | 因子グラフを用いた最適化ベースの状態推定法。SLAMで主流 |

## G
| 用語 | 定義 |
|------|------|
| **GLONASS** | ロシアの衛星測位システム（Globalnaya Navigatsionnaya Sputnikovaya Sistema） |
| **GNSS (Global Navigation Satellite System)** | 全球衛星測位システムの総称（GPS, GLONASS, Galileo, BeiDou等） |
| **GPS (Global Positioning System)** | 米国の衛星測位システム。31機の衛星で運用 |
| **Galileo** | EUの衛星測位システム。民生用高精度サービスを提供 |

## I
| 用語 | 定義 |
|------|------|
| **IMU (Inertial Measurement Unit)** | 加速度計・ジャイロスコープで構成される慣性計測ユニット |
| **IPS (Indoor Positioning System)** | 屋内測位システムの総称 |
| **INS (Inertial Navigation System)** | IMUの積分計算で位置を推定する慣性航法システム |

## K
| 用語 | 定義 |
|------|------|
| **KF (Kalman Filter)** | 線形システムの状態推定に使われる最適な逐次ベイズフィルタ |

## L
| 用語 | 定義 |
|------|------|
| **LEO-PNT** | Low Earth Orbit衛星群を利用した測位航法タイミング技術 |
| **LiDAR SLAM** | レーザスキャナの点群マッチングで自己位置と地図を同時推定 |
| **LPWAN (Low Power Wide Area Network)** | LoRa, NB-IoT等の低消費電力広域通信規格 |

## M
| 用語 | 定義 |
|------|------|
| **マルチパス** | 反射・回折により複数の伝搬経路で信号が到達する現象。測位誤差の主要因 |
| **MLベース測位** | 機械学習を用いてRSSI/CSI等から位置を推定する手法 |
| **mmWave** | ミリ波（30-300GHz）。5G/6Gで高精度測距に利用 |

## N
| 用語 | 定義 |
|------|------|
| **NAVIC** | インドの地域衛星測位システム（旧IRNSS） |
| **NLOS (Non-Line-of-Sight)** | 見通し線外の信号伝搬。測位精度低下の主要因 |

## P
| 用語 | 定義 |
|------|------|
| **PDR (Pedestrian Dead Reckoning)** | 歩行者の歩数・歩幅・方位を推定して位置を算出する手法 |
| **PF (Particle Filter)** | 多数のパーティクルで確率分布を近似する非線形フィルタ |
| **PPP (Precise Point Positioning)** | 精密衛星軌道・時計情報を用いた高精度単独測位。基準局不要 |
| **PPP-AR** | 整数値アンビギュイティ解決を伴うPPP。精度1-10cm |

## Q
| 用語 | 定義 |
|------|------|
| **QZSS** | 日本の準天頂衛星システム。GPS補完・cm級補強信号提供 |
| **量子測位** | 量子センシング技術を用いた超高精度測位。研究段階 |

## R
| 用語 | 定義 |
|------|------|
| **RFID** | 電波によるID読み取り技術。パッシブ/アクティブタグ利用 |
| **RSSI (Received Signal Strength Indicator)** | 受信信号強度。距離推定の基本指標だが変動大 |
| **RTK (Real-Time Kinematic)** | 搬送波位相の二重差を用いたcm級リアルタイム測位。基準局必要 |

## S
| 用語 | 定義 |
|------|------|
| **SLAM (Simultaneous Localization and Mapping)** | 自己位置推定と環境地図構築を同時に行う技術 |
| **センサフュージョン** | 複数センサのデータを統合し、単一センサより高精度・ロバストな推定を実現 |

## T
| 用語 | 定義 |
|------|------|
| **ToA (Time of Arrival)** | 信号の到達時間から距離を測定する手法 |
| **TDoA (Time Difference of Arrival)** | 複数基地局への到達時間差から位置を推定する手法 |
| **TTFF (Time To First Fix)** | GNSS受信機の初回測位時間 |

## U
| 用語 | 定義 |
|------|------|
| **UKF (Unscented Kalman Filter)** | 非線形変換にUnscented Transformを用いるカルマンフィルタ |
| **UWB (Ultra-Wideband)** | 広帯域（500MHz以上）の短パルス信号を用いた高精度測距技術。精度10-30cm |
| **USBL (Ultra-Short Baseline)** | 水中音響測位の一種。送受波器間の超短基線で方位・距離測定 |

## V
| 用語 | 定義 |
|------|------|
| **VLC (Visible Light Communication)** | LED照明光の変調による通信・測位技術 |
| **Visual Odometry (VO)** | 連続画像からカメラ運動を逐次推定する手法 |
| **Visual SLAM** | カメラ画像を主センサとするSLAMシステム（ORB-SLAM, DSO等） |