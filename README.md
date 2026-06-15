# 🏢 50 Startups 特徵選擇方法對比分析

這是一個關於利用不同特徵選擇方法來預測 50 家初創公司利潤的機器學習分析專案。

## 📋 項目概述

本分析基於 **CRISP-DM** 流程，對比評估 **10 個不同的特徵選擇方法**在線性回歸模型中的表現，幫助找出最優的特徵組合以預測初創公司利潤。

### 核心目標
- 探索初創公司的支出數據（R&D、Administration、Marketing）與利潤的關係
- 評估 10 種特徵選擇方法的優劣
- 找出最優特徵組合以實現最佳預測性能
- 分析各方法的特徵優先順序排名

---

## 📊 數據集資訊

**資料來源**：50 家初創公司的財務數據

**特徵**：
- **R&D Spend**（研發支出）
- **Administration**（行政支出）
- **Marketing Spend**（市場營銷支出）
- **State**（州份：California、Florida、New York）

**目標變數**：Profit（利潤）

---

## 🔧 分析流程

### 1️⃣ 業務理解 & 數據理解
- 載入並探索 50 家初創公司的支出數據
- 分析各特徵與利潤的相關性

### 2️⃣ 數據準備
- One-hot 編碼 State 類別變數
- 標準化數值特徵
- 分割訓練集與測試集

### 3️⃣ 特徵選擇
應用以下 10 種特徵選擇方法：

| 方法 | 類型 | 說明 |
|------|------|------|
| SelectKBest (f_regression) | 統計方法 | 基於 F 統計量選擇特徵 |
| SelectKBest (mutual_info) | 統計方法 | 基於互信息選擇特徵 |
| RFE with LinearRegression | 遞歸方法 | 線性回歸遞歸特徵消除 |
| RFE with Ridge | 遞歸方法 | Ridge 回歸遞歸特徵消除 |
| Linear Regression Coefficients | 模型係數 | 基於回歸係數的重要性 |
| Random Forest Importance | 樹模型 | 基於隨機森林的特徵重要性 |
| XGBoost Importance | 樹模型 | 基於 XGBoost 的特徵重要性 |
| Permutation Importance | 模型無關 | 基於特徵排列的重要性 |
| Correlation with Target | 統計方法 | 基於與目標變數的皮爾遜相關係數 |
| Lasso Coefficients | 正則化方法 | 基於 Lasso 的特徵選擇 |

### 4️⃣ 建模
- 使用 **Multiple Linear Regression** 評估各方法
- 測試 1～5 個特徵的不同組合

### 5️⃣ 評估
基於以下指標進行對比：
- **R² Score**（決定係數）- 越高越好
- **RMSE**（均方根誤差）- 越低越好
- **MAE**（平均絕對誤差）- 越低越好

---

## 🎯 核心發現

### 🏆 最佳表現

**最優特徵組合**：**R&D Spend**（單個特徵）

| 指標 | 數值 |
|------|------|
| R² Score | **0.9265**（92.65% 解釋力） |
| RMSE | **7,714.33** |
| MAE | **6,077.36** |

### 💡 關鍵洞察

1. **R&D 支出是主導性預測因子**
   - 僅使用 1 個特徵（R&D Spend）就能達到 92.65% 的解釋力
   - 其他特徵貢獻有限甚至造成過度擬合

2. **特徵增加會降低性能**
   - 增加更多特徵往往導致 RMSE 增加、R² 下降
   - 這表明存在過度擬合現象

3. **特徵數量與性能的關係**

| 特徵數 | 最佳 RMSE | 最佳 R² | 對應方法 |
|--------|----------|---------|---------|
| 1 | 7,714.33 | 0.9265 | SelectKBest_f / Pearson / XGBoost |
| 2 | 8,206.33 | 0.9168 | SelectKBest_f / Pearson |
| 3 | 8,995.91 | 0.9001 | SelectKBest_mutual / RFE_Linear |
| 4 | 9,015.11 | 0.8996 | XGBoost |
| 5 | 9,055.96 | 0.8987 | 多數方法 |

---

## 📈 特徵優先順序排名

各方法推薦的特徵排名（Top 5）：

| 特徵選擇方法 | Rank 1 | Rank 2 | Rank 3 | Rank 4 | Rank 5 |
|-------------|--------|--------|--------|--------|--------|
| SelectKBest (f_regression) | R&D Spend | Marketing Spend | State_New York | State_Florida | Administration |
| SelectKBest (mutual_info) | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| RFE (LinearRegression) | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| RFE (Ridge) | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| Linear Regression Coef | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| Random Forest Importance | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| Permutation Importance | R&D Spend | Marketing Spend | State_New York | State_Florida | Administration |

**結論**：所有方法都一致性地排名 **R&D Spend** 為最重要的特徵。

---

## 📁 項目文件說明

| 檔案名稱 | 說明 |
|---------|------|
| `feature_selection_analysis.ipynb` | 完整的 Python 分析筆記本，包含所有 10 種特徵選擇方法的實現 |
| `feature_selection_results.csv` | 詳細的分析結果，包含各方法的 R² Score、RMSE、MAE 指標 |
| `feature_priority_ranking.csv` | 各特徵選擇方法的特徵優先順序排名表 |
| `hw6.md` | 項目的詳細文檔和分析報告 |
| `README.md` | 本檔案 |

---

## 🛠️ 技術棧

- **Python 3.x**
- **Pandas** - 數據處理
- **Scikit-learn** - 機器學習
- **NumPy** - 數值計算
- **Matplotlib & Seaborn** - 數據可視化
- **XGBoost** - 梯度提升模型

---

## 🚀 使用方法

### 執行分析

1. 確保已安裝所需的 Python 套件：
   ```bash
   pip install pandas scikit-learn numpy matplotlib seaborn xgboost
   ```

2. 打開 Jupyter Notebook：
   ```bash
   jupyter notebook feature_selection_analysis.ipynb
   ```

3. 執行所有單元格以完成完整分析

### 查看結果

- 開啟 `feature_selection_results.csv` 查看詳細的模型性能指標
- 開啟 `feature_priority_ranking.csv` 查看特徵排名結果
- 查閱 `hw6.md` 獲取深入的分析解釋

---

## 💬 主要結論

1. **簡約模型優於複雜模型** - 單一的 R&D Spend 特徵提供最佳預測性能
2. **避免過度擬合** - 添加無關特徵會降低泛化能力
3. **一致的特徵排名** - 10 種不同的方法都認可 R&D 支出的主導重要性
4. **實用建議** - 在實際應用中，只需使用 R&D Spend 就能有效預測初創公司利潤

---

## 📝 分析方法學

本分析嚴格遵循 **CRISP-DM (Cross-Industry Standard Process for Data Mining)** 流程：

```
業務理解 → 數據理解 → 數據準備 → 建模 → 評估 → 部署
```

採用多種特徵選擇方法確保結果的穩健性，並通過量化指標（R²、RMSE、MAE）進行客觀評估。

---

## 📧 聯繫方式

如有任何問題或建議，歡迎提出。

---

**最後更新**：2026 年 6 月  
**專案狀態**：✅ 完成分析
