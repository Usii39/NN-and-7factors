# 📈 神經網路結合多因子預測報酬－非線性量化交易策略

**(Deep Learning for Asset Pricing: Multi-Factor Neural Network Strategy in Taiwan Stock Market)**

![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)
![PyTorch](https://img.shields.io/badge/PyTorch-2.0+-ee4c2c.svg)
![Pandas](https://img.shields.io/badge/Pandas-Data_Processing-150458.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

## 📖 專案簡介 (Introduction)
本專案旨在運用深度學習（Deep Learning）技術解決量化財務中的橫截面資產定價問題。
有別於傳統線性定價模型（如 CAPM、Fama-French），本專案建構了一套深層前饋神經網路（Deep Feed-forward Network），直接學習個股微觀特徵（規模、價值、動能、流動性）與總體市場環境之間的非線性交互作用（Nonlinear Factor Interactions），並在樣本外的台灣股票市場中，實作出一套每週再平衡的純多頭（Long-only）量化交易策略。

## ✨ 核心特色 (Key Features)
* **🚫 零未來數據引入 (No Look-ahead Bias)**：嚴格的日曆對齊（Calendar Alignment）機制，確保訓練與回測階段的特徵與目標標籤（ $Y_{t+1}$ ）在時間軸上完美對齊。
* **🧠 幾何金字塔神經網路 (Geometric Pyramid NN3)**：參考頂級財務期刊文獻，建構 `32 -> 16 -> 8` 的特徵壓縮架構，並結合 `BatchNorm`、`Dropout` 與 `AdamW` 權重衰減，有效抵抗金融市場的低訊噪比。
* **🚀 動態 DataLoader 與 Parquet 儲存**：為解決金融面板資料（Panel Data）每週股票數量變動的問題，客製化 PyTorch DataLoader 動態讀取時間序列 Parquet 檔案，大幅降低記憶體消耗並提升訓練極速。
* **📊 嚴謹的樣本外回測 (Out-of-sample Backtesting)**：捨棄傳統市值加權大盤，採用「全市場等權重投資組合（EW Benchmark）」作為公平對照組，精準衡量模型的超額選股能力（Alpha）。

## 🛠️ 底層資料倉儲建置 (Data Infrastructure)
本專案的底層資料來源為台灣經濟新報（TEJ+），涵蓋台灣上市櫃公司長達十年的日頻率價量與財務因子資料（原始大小約 258MB，共計超過 500 萬列）。
為解決 Pandas 直接讀取全量 Excel 檔案所導致的記憶體溢位（OOM）問題，並支援高頻率的滾動式特徵萃取，本專案建構了一套自動化的 ETL 流程，將靜態檔案轉化為高效能的本地端 MySQL 關聯式資料庫。

* **資料清洗與正規化 (ETL & Normalization)**：
  * 使用 Pandas 與正則表達式（Regex）處理原始資料中的雜訊（如日期格式不一、代碼夾雜中文、財報缺漏值等）。
  * 採用**第一正規化（1NF）**原則，將龐大的單一表格切分為 `tej_price_daily`（價量表）與 `tej_factor_daily`（微觀財務因子表），並透過 `股票代碼` 與 `年月日` 作為主鍵（Primary Key）進行關聯。
* **高精準度儲存與效能優化 (Storage Optimization)**：
  * **自訂 Data Dictionary**：捨棄 Python 預設的 Float 型態，透過 SQLAlchemy 嚴格定義 MySQL 底層的 `Numeric(10, 2)`、`Numeric(10, 6)` 與 `BIGINT`，徹底消除浮點數誤差，並大幅縮小硬碟與緩衝池（Buffer Pool）的空間佔用。
  * **B-Tree 複合高速索引**：針對 `(股票代碼, 年月日)` 建立複合索引（Composite Index），將特徵矩陣的 JOIN 查詢速度從全表掃描的分鐘級縮短至**毫秒級 (Millisecond-level)**，完美支撐深度學習模型每週海量的特徵調取需求。
* **全市場因子對齊 (Market Factor Integration)**：
  * 獨立建置 `tej_market_factors_daily`，儲存 Fama-French 市場溢酬與動能等系統性風險因子。
  * 實測階段可透過極速的 `INNER JOIN`，在資料庫底層瞬間將「個股微觀特徵」與「當日總體市場環境」拼合，輸出完整的強化學習狀態矩陣（State Features）。
 
<img width="1189" height="590" alt="image" src="https://github.com/user-attachments/assets/77d8ec1c-427d-44b5-880a-537021b3097c" />

