---
agent: SRC
task_id: TASK-001
date: 2026-06-11
status: approved
category: shared
destination: shared/search/specs/search-results/SR-001_positioning-tech-comprehensive.md
related:
  - "[TASK-001_positioning-tech-survey](../../tasks/active/TASK-001_positioning-tech-survey.md)"
  - "[SD-001](../search/decisions/search/SD-001_positioning-tech-survey.md)"
tags:
  - SRC
  - positioning-technology
  - TASK-001
  - comprehensive-survey
---

# SR-001: 位置情報測定技術 包括的先行研究調査結果

## 1. 衛星測位 (GNSS)

### GPS (Global Positioning System)
- **基本原理**: 31機の衛星から送信されるL帯信号の到達時間差（TOA）を測定し、3次元位置を三角測量
- **典型的精度**: 商用SPS: 3-5m, マルチバンド: サブm級
- **適用環境**: 屋外（見通し線必須）
- **代表的研究**: Reid et al. (2019) "Standalone and RTK GNSS on 30,000 km of North American Highways" — 公道での性能評価、車線判別98%@RTK [arXiv:1906.08180]
- **長所**: 全球カバレッジ、24時間運用、民生品が安価
- **短所**: 屋内不可、都市部でマルチパス・非見通し劣化、脆弱性（妨害・なりすまし）

### GLONASS (ロシア)
- **基本原理**: GPSと同様のTOA測位、FDMA方式（一部CDMA移行中）
- **精度**: 3-6m
- **適用環境**: 屋外（高緯度帯でGPSより優位）

### Galileo (EU)
- **基本原理**: GPSと互換性ある信号設計、無料公開サービス
- **精度**: 1-4m（民生）、0.2m（有料認証サービス）
- **特徴**: 2024年に完全運用能力（FOC）達成、PRS認証サービス提供

### BeiDou (中国)
- **基本原理**: 35機構成、MEO+GEO+IGSOのハイブリッド星座
- **精度**: 2-5m（民生）、0.1m（PPPサービス対応）
- **特徴**: 短衛星通信メッセージ機能搭載

### QZSS (日本)
- **基本原理**: 準天頂軌道（QZO）衛星4機でGPS補完・補強
- **精度**: GPS+QZSSでcm級補強信号（L6）
- **適用**: 日本・アジア太平洋地域

### NAVIC (インド)
- **基本原理**: GEO+HEO 7機構成、L5/Sバンド
- **精度**: 5-10m（民生）、0.1m（制限サービス）

### キーレファレンス
- Reid et al. (2019) "Standalone and RTK GNSS on 30,000 km of North American Highways" — DOI:10.33012/2019.16914
- Sheikh et al. (2025) "Review of Positioning Technologies and Antenna Designs for Indoor, Outdoor, and Wearable Applications" — DOI:10.1109/ACCESS.2025.3614190

---

## 2. 地上無線測位

### Wi-Fi Positioning
- **基本原理**: RSSIフィンガープリンティング、ToA/TDoA、CSI分析
- **精度**: 2-15m（RSSIフィンガープリント）、1-3m（CSI/FTMベース）
- **適用環境**: 屋内
- **代表的研究**:
  - Bellavista-Parent et al. (2021) "New trends in indoor positioning based on WiFi and machine learning: A systematic review" — DOI:10.1109/IPIN51156.2021.9662521
  - Xia et al. (2017) "Indoor Fingerprint Positioning Based on Wi-Fi: An Overview" — DOI:10.3390/ijgi6050135 (citations: 250)
  - Liu et al. (2020) "Survey on WiFi‐based indoor positioning techniques" — DOI:10.1049/iet-com.2019.1059 (citations: 279)
- **長所**: 既存インフラ活用可、スマホ標準搭載
- **短所**: 信号変動大、環境変化に弱い、精度限界

### BLE (Bluetooth Low Energy) Beacon
- **基本原理**: iBeacon/EddystoneからのRSSI測定による近接検出・フィンガープリンティング
- **精度**: 1-5m（近接）、3-10m（フィンガープリント）
- **適用環境**: 屋内
- **長所**: 低消費電力、ビーコン安価、スマホ標準搭載
- **短所**: 精度限界、信号干渉、キャリブレーション必要

### UWB (Ultra-Wideband)
- **基本原理**: 極短パルスを用いたToA/TDoA測距、cm級測距精度
- **精度**: 10-30cm（測距）、20-50cm（測位）
- **適用環境**: 屋内・産業用
- **代表的研究**:
  - Qiao et al. (2024) "Advancements in Indoor Precision Positioning: A Comprehensive Survey of UWB and Wi-Fi RTT Positioning Technologies" — DOI:10.3390/network4040027 (citations: 11)
  - Kolakowski (2024) "Comparison of Extended and Unscented Kalman Filters Performance in a Hybrid BLE-UWB Localization System" — DOI:10.23919/MIKON48703.2020.9253854
- **代表製品**: Apple U1チップ（AirDrop指向性）、Decawave DW1000/DWM3000
- **長所**: 高精度、マルチパス耐性、信号干渉に強い
- **短所**: 高コスト、カバレッジ狭、規格競争

### RFID
- **基本原理**: パッシブ/アクティブタグのRSSI読み取り
- **精度**: 10cm-3m（リーダー依存）
- **適用**: 倉庫管理、サプライチェーン

### ZigBee
- **基本原理**: IEEE 802.15.4準拠メッシュネットワーク、RSSI測位
- **精度**: 3-10m
- **適用**: ホームオートメーション、センサーネットワーク

### LoRa/LPWAN
- **基本原理**: CSS変調による長距離低消費電力通信のRSSI/TDoA測位
- **精度**: 20-200m（TDoA）~km級（RSSI）
- **適用**: 広域IoT、農業、スマートシティ
- **代表的研究**: Janssen et al. (2023) "A Survey on IoT Positioning Leveraging LPWAN, GNSS, and LEO-PNT" — DOI:10.1109/JIOT.2023.3243207 (citations: 104)

### セルラー基地局測位
- **基本原理**: 携帯基地局のCell ID、OTDOA、UTDOA
- **精度**: 50m-数km（Cell ID）、50-200m（OTDOA）
- **適用**: 屋外広域

---

## 3. 慣性航法 (INS/IMU)

### 基本原理
加速度計・ジャイロスコープ・磁気センサによる3軸運動検出→積分による相対位置推定

### デッドレコニング (DR)
- **精度**: 歩行距離の2-10%の累積誤差（時間経過で発散）
- **適用**: 屋内（補正なしでは持続不可）

### 歩行者デッドレコニング (PDR)
- **基本原理**: 歩行検出→歩幅推定→進行方向→位置更新
- **精度**: 歩行距離の1-5%（補正あり）、数十m誤差/100m（補正なし）
- **代表的研究**:
  - Klein (2024) "Pedestrian Inertial Navigation: An Overview of Model and Data-Driven Approaches" — DOI:10.1016/j.rineng.2025.104077 [arXiv:2407.21676]
  - Chen et al. (2023) "ReLoc-PDR: Visual Relocalization Enhanced Pedestrian Dead Reckoning via Graph Optimization" [arXiv:2309.01646]
- **長所**: 外部インフラ不要、省電力、プライバシー保護
- **短所**: 累積誤差（時間発散）、センサノイズ、歩幅推定誤差

---

## 4. 視覚測位 (Visual)

### Visual SLAM
- **基本原理**: カメラ画像からの特徴点追跡＋同時マップ構築
- **精度**: cm-dm級（環境依存）
- **代表的研究**:
  - Boche et al. (2025) "OKVIS2-X: Open Keyframe-based Visual-Inertial SLAM Configurable with Dense Depth or LiDAR, and GNSS" — DOI:10.1109/TRO.2025.3619051 [arXiv:2510.04612]
  - Labbé & Michaud (2024) "RTAB-Map as an Open-Source Lidar and Visual SLAM Library" — DOI:10.1002/rob.21831 [arXiv:2403.06341]
  - Soares et al. (2025) "VAR-SLAM: Visual Adaptive and Robust SLAM for Dynamic Environments" [arXiv:2510.16205]
- **代表OSS**: ORB-SLAM3, RTAB-Map, OpenVSLAM

### Visual Odometry (VO)
- **基本原理**: 連続画像間のカメラ運動推定（SLAMより軽量）
- **精度**: cm級（短距離）
- **代表**: DSO (Direct Sparse Odometry), SVO (Semi-direct VO)

### LiDAR SLAM
- **基本原理**: レーザスキャナの点群マッチングによる自己位置推定
- **精度**: cm級
- **代表的研究**:
  - Liu et al. (2024) "LVI-Fusion: A Robust Lidar-Visual-Inertial SLAM Scheme" — DOI:10.3390/rs16091524 (citations: 24)
  - FAST-LIVO2 (hku-mars) — GitHub 4173 stars
- **代表OSS**: LOAM, A-LOAM, LeGO-LOAM, FAST-LIO2, FAST-LIVO2

### マーカーベース測位
- **基本原理**: ARマーカ（QRコード等）のカメラ検出・姿勢推定
- **精度**: mm-cm級

### 長所・短所（視覚系全体）
- **長所**: 高精度、豊富な環境情報取得、ハードウェア低コスト
- **短所**: 照明依存、テクスチャレス環境不可、計算負荷大、動的物体に弱い

---

## 5. 音響測位

### 超音波測位
- **基本原理**: 超音波パルスのToA/TDoA測定による三角測量
- **精度**: cm級（1-10cm）
- **適用環境**: 屋内（見通し線必須）
- **代表的研究**: Delabie et al. (2024) "Anchor Layout Optimization for Ultrasonic Indoor Positioning Using Swarm Intelligence" — DOI:10.1109/IPIN57070.2023.10332490 [arXiv:2405.09222]
- **代表的研究**: Ureña et al. (2018) "Acoustic Local Positioning With Encoded Emission Beacons" — DOI:10.1109/JPROC.2018.2819938 (citations: 75)
- **長所**: 高精度、低コスト、部屋単位の信号閉じ込め
- **短所**: 見通し線必須、マルチパス、大気条件影響、干渉

### 可聴音測位
- **基本原理**: スピーカーからの音響信号（人間に聞こえない帯域も含む）の到達時間測定
- **精度**: dm級
- **適用**: 屋内、スマホベース

### 水中音響測位
- **基本原理**: 音響トランスデューサによるLBL/SBL/USBL測位
- **精度**: 1-10m（LBL）、0.1-1度（USBL方位）
- **適用環境**: 水中
- **代表的研究**: Jouhari et al. (2019) "Underwater Wireless Sensor Networks: A Survey" — DOI:10.1109/ACCESS.2019.2928876 (citations: 317)
- **代表**: Jahanbakht et al. (2021) "Internet of underwater things and big marine data analytics" — (citations: 318)

---

## 6. 磁気測位

### 地磁気マップマッチング
- **基本原理**: 鉄骨構造による地磁気異常をフィンガープリントとして利用
- **精度**: 1-5m
- **適用**: 屋内（鉄骨構造のあるビル）
- **代表的研究**: Li et al. (2012) "How feasible is the use of magnetic field alone for indoor positioning?" — DOI:10.1109/IPIN.2012.6418880 (citations: 262)
- **長所**: 特別なインフラ不要、スマホ標準搭載磁力計、省電力
- **短所**: 環境変化に弱い、鉄筋に依存、磁場時間変動、次元数少なさ（3成分のみ）

---

## 7. ハイブリッド/融合測位

### センサフュージョン手法
- **カルマンフィルタ (KF/EKF/UKF)**: 線形/弱非線形システムに適応。EKF/UKFは非線形対応
- **パーティクルフィルタ (PF)**: 非線形・非ガウス対応、高精度だが計算負荷大
- **因子グラフ最適化 (FGO)**: スライディングウィンドウ最適化、SLAMで主流
- **相補フィルタ**: 低周波・高周波成分のハイブリッド合成

### 代表的研究
- Wang & Ahmad (2024) "A Comprehensive Review on Sensor Fusion Techniques for Localization of a Dynamic Target in GPS-Denied Environments" — DOI:10.1109/ACCESS.2024.3519874 (citations: 36)
- Stefanoni et al. (2025) "A Survey on the Main Techniques Adopted in Indoor and Outdoor Localization" — DOI:10.3390/electronics14102069 (citations: 5)
- Guo et al. (2019) "A Survey on Fusion-Based Indoor Positioning" — DOI:10.1109/COMST.2019.2951036 (citations: 262)
- Zhang et al. (2024) "Seamless Outdoor-Indoor Pedestrian Positioning System with GNSS/UWB/IMU Fusion: A Comparison of EKF, FGO, and PF" [arXiv:2512.10480]

### 複数技術組み合わせ例
- **GNSS+IMU**: 自動運転の基本構成（RTK-GNSS + 高精度IMU）
- **UWB+IMU**: 屋内測位の高精度ソリューション
- **Visual+LiDAR+IMU**: FAST-LIVO2 (GitHub 4173 stars)
- **BLE+UWB+Kailman**: Kolakowski (2024) のハイブリッドBLE-UWBシステム
- **GNSS+UWB+IMU+PDR**: 屋内屋外シームレス測位（Zhang et al. 2024）

---

## 8. 特殊環境測位

### 屋内測位
- **課題**: GPS不可、マルチパス、動的環境変化
- **主要技術**: WiFi, BLE, UWB, PDR, VLP, 磁気, 音響
- **代表サーベイ**:
  - Yassin et al. (2016) "Recent Advances in Indoor Localization: A Survey" — DOI:10.1109/COMST.2016.2632427 (高被引用)
  - Kunhoth et al. (2020) "Indoor positioning and wayfinding systems: a survey" — DOI:10.1186/s13673-020-00222-0 (citations: 257)
  - Geok et al. (2020) "Review of Indoor Positioning: Radio Wave Technology" — DOI:10.3390/app11010279 (citations: 187)

### 水中測位
- **技術**: 音響LBL/SBL/USBL、慣性航法、磁気誘導
- **課題**: 電波減衰（水中）、音響のマルチパス・温度依存
- **代表**: Jouhari et al. (2019), Jahanbakht et al. (2021)

### 地下測位
- **技術**: PDR + 磁気マップマッチング、WiFi/BLE（坑道）

### 宇宙測位（LEO-PNT）
- **基本原理**: LEO衛星群（Starlink等）からの信号利用
- **代表**: Janssen et al. (2023) "A Survey on IoT Positioning Leveraging LPWAN, GNSS, and LEO-PNT"

---

## 9. 新興技術

### 5G/6G測位
- **基本原理**: 5G NRの高帯域信号（mmWave）+ MIMOビームフォーミングを活用した高精度測距・測角
- **精度**: サブm級（mmWave）、m級（sub-6GHz）
- **代表的研究**:
  - Zheng et al. (2023) "5G-Aided RTK Positioning in GNSS-Deprived Environments" [arXiv:2303.13067]
  - Charan et al. (2024) "Pixel-Level GPS Localization and Denoising using Computer Vision and 6G Communication Beams" — DOI:10.1109/GLOBECOM52923.2024.10901399
  - Silva et al. (2026) "Overview of Vehicle Localization: The Shift From Egocentric to Cooperative Positioning" — DOI:10.1109/ACCESS.2026.3678845
- **長所**: 通信インフラとの統合、高帯域幅による精度向上
- **短所**: 標準化途上、専用機器必要

### 機械学習ベース測位
- **手法**: CNN/LSTM/GANデータ拡張（SURIMI）、アンサンブル学習、強化学習
- **代表**: Quezada-Gaibor et al. (2022) "SURIMI: Supervised Radio Map Augmentation with Deep Learning and GAN" — DOI:10.1109/IPIN54987.2022.9918146 [arXiv:2207.06120]
- **応用**: WiFiフィンガープリント自動調整、UWB NLOS補正、PDR歩幅学習

### 協調測位（Crowdsourcing/Collaborative）
- **基本原理**: 端末間通信・BLE/WiFi/UWBを用いた相対測位、信念伝播（Belief Propagation）
- **代表**: Pascacio et al. (2021) "Collaborative Indoor Positioning Systems: A Systematic Review" — DOI:10.3390/s21031002 (citations: 157)

### 量子測位
- **基本原理**: 量子センシングによる超高感度慣性計測・原子時計
- **現状**: 基礎研究段階、実験室レベルの概念実証
- **課題**: 実用化まで長期、装置巨大、コスト

### 可視光測位 (VLC/VLP)
- **基本原理**: LED照明の光信号変調（ToA/RSSI/AoA）による測位
- **精度**: cm-dm級（最高1.93cm — Chen et al. 2019）
- **適用**: 屋内（LED照明のある環境）
- **長所**: 高精度、照明と兼用、電磁干渉なし
- **短所**: 見通し線必須、照明必須、屋外不可

---

## 10. 測位精度向上技術

### RTK (Real-Time Kinematic)
- **基本原理**: 基準局と移動局の搬送波位相観測値の二重差から整数値アンビギュイティを解決
- **精度**: 1-5cm（RTK Fix）、10-40cm（RTK Float）
- **代表**: Reid et al. (2019), Zheng et al. (2023)
- **長所**: cm級リアルタイム測位
- **短所**: 基準局必要（10-20km以内）、Fix解が得られないと精度低下

### PPP (Precise Point Positioning)
- **基本原理**: 精密衛星軌道・クロック情報を用いて単独測位の誤差を補正
- **精度**: 1-10cm（PPP-AR）、dm級（標準PPP）
- **長所**: 基準局不要、広域適用可能
- **短所**: 収束時間30分以上、整数値アンビギュイティ解決なしでは精度限界
- **代表**: Arkalı & Atik (2025) UAV-RTK/PPK/PPP-AR比較 — DOI:10.5194/isprs-archives-xlviii-m-6-2025-325-2025

### DGPS (Differential GPS)
- **基本原理**: 基準局が計算した誤差補正値を移動局に配信
- **精度**: 1-3m
- **長所**: 実装容易
- **短所**: カバレッジ限界、補正遅延

### A-GPS (Assisted GPS)
- **基本原理**: サーバから衛星軌道情報を取得し初回測位時間（TTFF）を短縮
- **精度**: 5-15m（通常GPSと同等）
- **適用**: スマートフォン

### マルチパス対策
- **受信機技術**: マルチパス推定ML、二重デルタ相関、ストロボコリレータ
- **環境技術**: 3D都市マップ支援（Shadow Matching）、魚眼カメラスカイビュー解析

### 誤差補正手法
- **アンビギュイティ解決**: LAMBDA法、MLAMBDA
- **サイクルスリップ検出**: TurboEdit, GF/GF法
- **電離層補正**: Klobucharモデル、NeQuick-G、Ionosphere-free Linear Combination
- **対流圏補正**: Saastamoinen, Hopfield, UNB3m

---

## 総括表

| カテゴリ | 技術 | 精度 | 環境 | インフラ依存度 | 主要論文数 |
|----------|------|------|------|---------------|-----------|
| 衛星測位 | GNSS (SPS) | 3-5m | 屋外 | 高（衛星） | ★★★★★ |
| 衛星測位 | GNSS+RTK | 1-5cm | 屋外 | 超高（衛星+基準局） | ★★★★ |
| 地上無線 | UWB | 10-50cm | 屋内 | 中（アンカー） | ★★★★ |
| 地上無線 | WiFi | 2-15m | 屋内 | 低（既存AP） | ★★★★★ |
| 地上無線 | BLE | 1-10m | 屋内 | 低（ビーコン） | ★★★★ |
| 地上無線 | LoRa | 20-200m | 屋外広域 | 中（ゲートウェイ） | ★★★ |
| 慣性航法 | IMU/PDR | 1-5%距離 | 屋内/屋外 | 不要 | ★★★★ |
| 視覚 | Visual SLAM | cm-dm | 屋内/屋外 | 不要 | ★★★★★ |
| 視覚 | LiDAR SLAM | cm | 屋内/屋外 | 不要 | ★★★★ |
| 音響 | 超音波 | 1-10cm | 屋内LOS | 高（アンカー） | ★★★ |
| 音響 | 水中音響 | 1-10m | 水中 | 高（トランスポンダ） | ★★★ |
| 磁気 | 地磁気 | 1-5m | 屋内 | 不要 | ★★★ |
| 新興 | 5G/6G | サブm | 屋内/屋外 | 高（基地局） | ★★★ |
| 新興 | VLC/VLP | cm-dm | 屋内LOS | 中（LED） | ★★★ |
| 新興 | MLベース | 可変 | 各種 | データ依存 | ★★★★★ |
| 融合 | マルチセンサ | 可変 | 全環境 | 混合 | ★★★★★ |

---

## 主要文献リスト

1. Stefanoni et al. (2025) "A Survey on the Main Techniques Adopted in Indoor and Outdoor Localization" — DOI:10.3390/electronics14102069
2. Wang & Ahmad (2024) "A Comprehensive Review on Sensor Fusion Techniques for Localization in GPS-Denied Environments" — DOI:10.1109/ACCESS.2024.3519874
3. Qiao et al. (2024) "Advancements in Indoor Precision Positioning: A Comprehensive Survey of UWB and Wi-Fi RTT" — DOI:10.3390/network4040027
4. Sheikh et al. (2025) "Review of Positioning Technologies and Antenna Designs" — DOI:10.1109/ACCESS.2025.3614190
5. Li et al. (2020) "Location-Enabled IoT (LE-IoT): A Survey of Positioning Techniques, Error Sources, and Mitigation" [arXiv:2004.03738]
6. Guo et al. (2019) "A Survey on Fusion-Based Indoor Positioning" — DOI:10.1109/COMST.2019.2951036 (citations: 262)
7. Yassin et al. (2016) "Recent Advances in Indoor Localization: A Survey" — DOI:10.1109/COMST.2016.2632427
8. Klein (2024) "Pedestrian Inertial Navigation: An Overview of Model and Data-Driven Approaches" — DOI:10.1016/j.rineng.2025.104077 [arXiv:2407.21676]
9. Boche et al. (2025) "OKVIS2-X: Open Keyframe-based Visual-Inertial SLAM" — DOI:10.1109/TRO.2025.3619051
10. Ureña et al. (2018) "Acoustic Local Positioning With Encoded Emission Beacons" — DOI:10.1109/JPROC.2018.2819938
11. Pascacio et al. (2021) "Collaborative Indoor Positioning Systems: A Systematic Review" — DOI:10.3390/s21031002 (citations: 157)
12. Janssen et al. (2023) "A Survey on IoT Positioning Leveraging LPWAN, GNSS, and LEO-PNT" — DOI:10.1109/JIOT.2023.3243207 (citations: 104)
13. Zhang et al. (2024) "Seamless Outdoor-Indoor Pedestrian Positioning with GNSS/UWB/IMU Fusion" [arXiv:2512.10480]
14. Reid et al. (2019) "Standalone and RTK GNSS on 30,000 km of North American Highways" — DOI:10.33012/2019.16914