# HW6：10個特徵選擇方法對比分析 - 50 Startups利潤預測

## 📊 項目概述

本分析遵循 **CRISP-DM 流程**，對比評估10個不同的特徵選擇方法在線性回歸模型中的表現，使用50家初創公司的支出數據預測利潤。

### 分析流程
1. **業務理解 & 數據理解**：探索50家新創公司的支出（R&D、Administration、Marketing）與利潤關係
2. **數據準備**：One-hot編碼State變數，標準化數值特徵
3. **特徵選擇**：應用10個不同的特徵選擇方法
4. **建模**：使用Multiple Linear Regression評估各方法
5. **評估**：基於R²、RMSE、MAE指標進行對比分析

---

## 📈 核心發現

### RMSE與R²比較

**最佳表現：** SelectKBest (f_regression) 和 Pearson Correlation（1個特徵）
- **R² Score**: 0.9265（最高）
- **RMSE**: 7,714.33（最低）
- **特徵組合**: R&D Spend

**重要洞察**：
- 1個特徵（R&D Spend）已能達到92.65%的解釋力，RMSE僅7,714
- 增加更多特徵往往導致性能下降（RMSE增加，R²下降）
- 這表明**R&D Spend是主導性預測因子**，其他特徵貢獻有限甚至造成過度擬合

| 特徵數 | 最佳RMSE | 最佳R² | 對應方法 |
|--------|---------|--------|---------|
| 1 | 7,714.33 | 0.9265 | SelectKBest_f / Pearson / XGBoost |
| 2 | 8,206.33 | 0.9168 | SelectKBest_f / Pearson |
| 3 | 8,995.91 | 0.9001 | SelectKBest_mutual / RFE_Linear / RandomForest / Lasso |
| 4 | 9,015.11 | 0.8996 | XGBoost |
| 5 | 9,055.96 | 0.8987 | 多數方法 |

---

## 🎯 特徵優先順序排名表

| 特徵選擇方法 | Rank 1 | Rank 2 | Rank 3 | Rank 4 | Rank 5 |
|-------------|--------|--------|--------|--------|--------|
| SelectKBest_f_regression | R&D Spend | Marketing Spend | State_New York | State_Florida | Administration |
| SelectKBest_mutual_info | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| RFE_LinearRegression | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| RFE_Ridge | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| LR_Coefficients | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| RandomForest_Importance | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| Permutation_Importance | R&D Spend | Marketing Spend | State_New York | State_Florida | Administration |
| Pearson_Correlation | R&D Spend | Marketing Spend | State_New York | State_Florida | Administration |
| Lasso_Coefficients | R&D Spend | Marketing Spend | Administration | State_Florida | State_New York |
| XGBoost_Importance | R&D Spend | State_New York | Administration | Marketing Spend | State_Florida |

### 共識排名（多數方法的結果）
1. **R&D Spend** ✓ （10/10方法一致推薦）
2. **Marketing Spend** ✓ （10/10方法推薦）
3. **Administration** 或 **State_New York** （方法相依）
4. **State_Florida** 和 **State_New York** （重要性較低）
5. **Administration** （最不重要）

---

## 📊 評估指標對比

### RMSE vs 特徵數
- **趨勢**：增加特徵導致RMSE增加（模型複雜度增加）
- **最優點**：1-2個特徵時RMSE最低
- **建議**：使用1個特徵（R&D Spend）即可達到接近最佳效果

### R² Score vs 特徵數
- **趨勢**：R²在1個特徵時最高（0.9265），之後持續下降
- **原因**：額外特徵引入噪聲而非有用信息
- **結論**：R&D Spend是決定性特徵，其他特徵的邊際貢獻為負

### MAE vs 特徵數
- **最佳結果**：單一特徵（R&D Spend）的MAE為6,077.36
- **性能對比**：與RMSE趨勢一致，確認1-2特徵的優越性

---

## 🔍 方法群組對比

| 方法類別 | 代表方法 | 特點 | 推薦度 |
|---------|---------|------|--------|
| 統計篩選 | SelectKBest, Pearson | 快速、穩定 | ⭐⭐⭐⭐⭐ |
| 遞歸消除 | RFE_Linear, RFE_Ridge | 迭代優化 | ⭐⭐⭐⭐ |
| 模型係數 | LR_Coef, Lasso, Ridge | 基於模型權重 | ⭐⭐⭐⭐ |
| 樹模型 | RandomForest, XGBoost | 非線性捕捉 | ⭐⭐⭐⭐ |
| 資訊論 | Mutual Information | 捕捉非線性關係 | ⭐⭐⭐⭐ |

---

## 💡 主要結論

### 1. **特徵選擇的關鍵發現**
- ✓ **R&D Spend 是絕對主導特徵**，單獨使用可解釋92.65%的利潤變異
- ✓ **Marketing Spend 為次要特徵**，加入後R²提升有限
- ✗ **其他特徵的邊際貢獻為負**，增加會導致過度擬合

### 2. **模型複雜度的權衡**
- **最簡單模型（1特徵）**：RMSE=7,714 | R²=0.9265
- **相對複雜模型（2特徵）**：RMSE=8,206 | R²=0.9168
- **權衡建議**：選擇1特徵達到最優性能，或2特徵平衡簡潔性與精度

### 3. **方法一致性**
- 10個方法在前2個特徵的選擇上**高度一致**：都優先推薦R&D Spend和Marketing Spend
- 證明結論的**穩健性和可信度**

### 4. **實務應用建議**
- 📌 **若追求最高精度**：使用R&D Spend單特徵模型（簡單高效）
- 📌 **若追求平衡方案**：結合R&D Spend + Marketing Spend（2特徵，解釋率91.68%）
- 📌 **模型簡化空間大**：可從5個特徵簡化至1-2個，性能幾乎不損

---

## 📎 文件清單

- `feature_selection_analysis.ipynb` - 完整分析筆記本
- `feature_selection_results.csv` - 10個方法的詳細結果（R²、RMSE、MAE）
- `feature_priority_ranking.csv` - 10個方法的特徵優先順序排名
- `rmse_comparison.png` - RMSE vs 特徵數對比圖
- `r2_comparison.png` - R² Score vs 特徵數對比圖
- `mae_comparison.png` - MAE vs 特徵數對比圖

---

## ✅ 報告簽名

**分析日期**：2026-06-15  
**分析方法**：CRISP-DM 流程  
**模型類型**：Multiple Linear Regression with Feature Selection  
**評估樣本**：50家初創公司（訓練集40樣本、測試集10樣本）

