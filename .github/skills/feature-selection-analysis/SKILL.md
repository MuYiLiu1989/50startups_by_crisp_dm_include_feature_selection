---
name: feature-selection-analysis
description: "Use when: performing feature selection for regression analysis, comparing multiple feature selection methods, evaluating model performance across different feature subsets, or implementing CRISP-DM workflow for machine learning projects. This skill applies 10 different feature selection techniques to identify the best features for predicting startup profits."
user-invocable: true
keywords:
  - feature selection
  - machine learning
  - regression analysis
  - CRISP-DM
  - scikit-learn
  - model evaluation
---

# 特徵選擇分析工作流 (Feature Selection Analysis Workflow)

## 技能概述

這個技能實現了一個完整的機器學習工作流，用於對回歸問題進行特徵選擇分析。基於CRISP-DM方法論，包括業務理解、數據準備、特徵選擇、建模、評估和部署6個階段。

**應用場景：**
- 確定回歸問題的最重要特徵
- 比較多個特徵選擇算法的性能
- 評估特徵數對模型性能的影響
- 建立機器學習管道並保存最佳模型

## 工作流程

### 1️⃣ 數據理解 (Data Understanding)

**目標：** 探索數據的基本結構和統計特性

**步驟：**
```python
# 加載數據
df = pd.read_csv(url)

# 檢查
print(f"形狀: {df.shape}")
print(f"列名和類型: {df.dtypes}")
print(f"基本統計: {df.describe()}")
print(f"缺失值: {df.isnull().sum()}")
print(f"分類變數分佈: {df['State'].value_counts()}")
```

**輸出內容：**
- 數據集維度 (行數 × 列數)
- 數據類型確認
- 統計指標 (均值、標準差等)
- 缺失值檢查
- 分類變數分佈

---

### 2️⃣ 數據準備 (Data Preparation)

**目標：** 轉換數據格式、分割數據集、進行特徵標準化

**步驟：**

#### Step 2.1: One-hot編碼 (分類變數)
```python
X_encoded = pd.get_dummies(X, columns=['State'], drop_first=True)
# 使用 drop_first=True 避免多重共線性
```

#### Step 2.2: 訓練/測試集分割
```python
X_train, X_test, y_train, y_test = train_test_split(
    X_encoded, y, test_size=0.2, random_state=42
)
# 使用固定隨機種子確保可重現性
```

#### Step 2.3: 特徵標準化
```python
scaler = StandardScaler()
numerical_cols = ['R&D Spend', 'Administration', 'Marketing Spend']
X_train_scaled[numerical_cols] = scaler.fit_transform(X_train[numerical_cols])
X_test_scaled[numerical_cols] = scaler.transform(X_test[numerical_cols])
```

**重要考量：**
- ✅ 標準化應用於*訓練集擬合*的縮放器
- ✅ 使用 `drop_first=True` 避免編碼陷阱
- ✅ 固定隨機種子確保結果可重現

---

### 3️⃣ 特徵選擇 - 10個方法 (Feature Selection)

每個方法獨立執行，輸出特徵排名。

#### **方法1: SelectKBest (f_regression)**
```python
selector = SelectKBest(score_func=f_regression, k='all')
selector.fit(X_train_scaled, y_train)
scores = selector.scores_
ranking = sorted(zip(feature_names, scores), key=lambda x: x[1], reverse=True)
```
- **原理：** F統計量反映特徵與目標變數的線性關係
- **適用：** 快速篩選與目標強相關的特徵

#### **方法2: SelectKBest (mutual_info_regression)**
```python
selector = SelectKBest(score_func=mutual_info_regression, k='all')
selector.fit(X_train_scaled, y_train)
```
- **原理：** 互信息捕捉特徵與目標的非線性關係
- **優勢：** 不限於線性關係

#### **方法3 & 4: RFE (遞歸特徵消除)**
```python
# 使用Linear Regression或Ridge
rfe = RFE(estimator, n_features_to_select=1, step=1)
rfe.fit(X_train_scaled, y_train)
```
- **原理：** 逐步移除特徵，根據模型係數評估重要性
- **特點：** 考慮特徵間的交互作用

#### **方法5: Linear Regression 係數絕對值**
```python
lr = LinearRegression()
lr.fit(X_train_scaled, y_train)
coef_importance = np.abs(lr.coef_)
```
- **簡單直觀：** 係數大小反映特徵對目標的影響
- **限制：** 只適用於線性模型

#### **方法6: Random Forest Feature Importance**
```python
rf = RandomForestRegressor(n_estimators=100, random_state=42)
rf.fit(X_train_scaled, y_train)
importance = rf.feature_importances_
```
- **優勢：** 捕捉非線性關係和交互作用
- **魯棒性：** 對異常值不敏感

#### **方法7: Permutation Importance**
```python
perm_importance = permutation_importance(model, X_test, y_test, n_repeats=10)
```
- **基於性能：** 直接衡量特徵對預測準確度的影響
- **模型無關：** 適用於任何模型

#### **方法8: Pearson相關係數**
```python
correlations = X_train_scaled.corr()['Profit']
ranking = sorted(zip(feature_names, np.abs(correlations)), reverse=True)
```
- **統計基礎：** 衡量線性相關性
- **快速計算：** 無需訓練模型

#### **方法9: Lasso L1正則化**
```python
lasso = Lasso(alpha=0.01, max_iter=10000)
lasso.fit(X_train_scaled, y_train)
lasso_coef = np.abs(lasso.coef_)
```
- **正則化優勢：** 自動進行特徵選擇
- **稀疏性：** 不重要特徵的係數→0

#### **方法10: XGBoost Feature Importance**
```python
xgb_model = xgb.XGBRegressor(n_estimators=100, random_state=42)
xgb_model.fit(X_train_scaled, y_train)
importance = xgb_model.feature_importances_
```
- **高效梯度提升：** 優秀的預測性能
- **複雜關係：** 捕捉複雜的特徵互動

---

### 4️⃣ 建模 (Modeling)

**目標：** 針對每個方法的特徵排名，訓練Multiple Linear Regression模型

**核心邏輯：**
```python
for method in all_methods:
    ranked_features = feature_rankings[method]
    
    # 逐步增加特徵數 (1-5)
    for n_features in range(1, 6):
        selected_features = ranked_features[:n_features]
        
        # 訓練模型
        X_train_sel = X_train_scaled[selected_features]
        X_test_sel = X_test_scaled[selected_features]
        
        model = LinearRegression()
        model.fit(X_train_sel, y_train)
        y_pred = model.predict(X_test_sel)
        
        # 評估
        r2 = r2_score(y_test, y_pred)
        rmse = np.sqrt(mean_squared_error(y_test, y_pred))
        mae = mean_absolute_error(y_test, y_pred)
```

**結果：**
- 10個方法 × 5個特徵數 = **50個模型組合**
- 每個組合的性能指標 (R², RMSE, MAE)

---

### 5️⃣ 評估 (Evaluation)

**性能指標：**

| 指標 | 公式 | 解釋 |
|------|------|------|
| **R² Score** | $1 - \frac{SS_{res}}{SS_{tot}}$ | 模型解釋方差比例 (0-1，越高越好) |
| **RMSE** | $\sqrt{\frac{1}{n}\sum(y_{pred}-y_{true})^2}$ | 預測誤差的標準差 (越低越好) |
| **MAE** | $\frac{1}{n}\sum\|y_{pred}-y_{true}\|$ | 平均絕對誤差 (越低越好) |

**可視化輸出：**
1. **RMSE vs 特徵數** - 特徵數與誤差的權衡
2. **MAE vs 特徵數** - 絕對誤差趨勢
3. **R² Score vs 特徵數** - 模型性能改進曲線

**分析重點：**
- 最優特徵數：通常3-4個特徵已達到性能平臺期
- 方法對比：某些方法排名相似，可併用
- 過度擬合檢測：測試集性能顯著下降

---

### 6️⃣ 部署 (Deployment)

**輸出文件：**

1. **feature_selection_results.csv**
   ```
   Method, Num_Features, Selected_Features, R2_Score, RMSE, MAE
   ```
   - 50行結果 (10方法×5特徵數)
   - 用於進階分析和報告

2. **feature_priority_ranking.csv**
   ```
   Method, Rank1, Rank2, Rank3, ...
   ```
   - 10個方法的特徵優先順序表
   - 跨方法比較和共識特徵

3. **可視化圖表** (.png)
   - `rmse_comparison.png`
   - `mae_comparison.png`
   - `r2_comparison.png`

4. **訓練的最佳模型** (.pkl)
   ```python
   # 選擇R²最高的模型
   best_model.pkl         # 線性回歸模型
   scaler.pkl             # 特徵標準化縮放器
   best_features.pkl      # 最佳特徵列表
   ```

**模型部署準備：**
```python
import pickle

# 載入保存的模型
with open('best_model.pkl', 'rb') as f:
    model = pickle.load(f)

with open('scaler.pkl', 'rb') as f:
    scaler = pickle.load(f)

# 新數據預測
X_new_scaled = scaler.transform(X_new[best_features])
predictions = model.predict(X_new_scaled)
```

---

## 必要的Python套件

```python
# 數據處理
pandas, numpy

# 機器學習
scikit-learn>=1.0

# 集成方法
xgboost

# 可視化
matplotlib, seaborn

# 模型序列化
pickle (內建)
```

**安裝命令：**
```bash
pip install pandas numpy scikit-learn xgboost matplotlib seaborn
```

---

## 核心配置和最佳實踐

### ✅ 建議做法

1. **固定隨機種子** → 確保結果可重現
   ```python
   random_state=42
   ```

2. **訓練/測試分割** → 避免數據洩漏
   ```python
   test_size=0.2, random_state=42
   ```

3. **特徵標準化** → 統一特徵尺度
   ```python
   scaler.fit(X_train)  # 僅擬合訓練數據
   scaler.transform(X_test)  # 應用到測試數據
   ```

4. **多個方法對比** → 獲得全面洞察
   - 線性方法: LR係數、Pearson相關
   - 樹型方法: Random Forest、XGBoost
   - 通用方法: SelectKBest、Permutation

5. **K值選擇** (每個方法的特徵數)
   - SelectKBest: `k='all'` 獲得所有得分
   - RFE: `step=1` 逐步移除，得到完整排名

### ⚠️ 常見陷阱

1. **中文字型問題** (如使用matplotlib)
   ```python
   plt.rcParams['font.sans-serif'] = ['SimHei', 'DejaVu Sans']
   plt.rcParams['axes.unicode_minus'] = False
   ```

2. **Lasso係數為0** → 減少alpha或增加max_iter
   ```python
   Lasso(alpha=0.01, max_iter=10000)
   ```

3. **Permutation Importance計算慢** → 使用n_jobs=-1並行
   ```python
   permutation_importance(..., n_jobs=-1)
   ```

4. **One-hot編碼維度爆炸** → 使用 `drop_first=True`
   ```python
   pd.get_dummies(..., drop_first=True)
   ```

---

## 預期結果

### 典型輸出示例

```
方法1: SelectKBest (f_regression)
  1. R&D Spend: 4532.5342
  2. Marketing Spend: 2134.2156
  3. Administration: 421.5634
  ...

=== 評估結果 ===
最高R² Score: 0.9498 (5特徵, XGBoost方法)
最低RMSE: 8542.23 (5特徵, Random Forest方法)
最低MAE: 6234.15 (4特徵, Lasso方法)

=== 輸出文件 ===
✓ feature_selection_results.csv (50行結果)
✓ feature_priority_ranking.csv (特徵排名表)
✓ rmse_comparison.png (對比圖表)
✓ best_model.pkl (部署模型)
```

---

## 工作流要點總結

| 階段 | 關鍵活動 | 輸出 |
|------|---------|------|
| **理解** | EDA探索 | 數據統計、分佈、缺失值 |
| **準備** | 編碼、分割、標準化 | 訓練/測試集 |
| **選擇** | 10個排名算法 | 特徵優先順序 |
| **建模** | 50個MLR模型 | 性能指標 |
| **評估** | 指標對比、可視化 | 圖表、分析報告 |
| **部署** | 模型序列化 | .pkl文件、可用於預測 |

---

## 擴展應用

### 1. 調整特徵數範圍
```python
for n_features in range(1, 10):  # 1-9個特徵
    # ...訓練模型...
```

### 2. 交叉驗證
```python
from sklearn.model_selection import cross_val_score
scores = cross_val_score(model, X_train, y_train, cv=5)
```

### 3. 超參數調優
```python
from sklearn.model_selection import GridSearchCV
param_grid = {'alpha': [0.001, 0.01, 0.1, 1.0]}
grid = GridSearchCV(Lasso(), param_grid, cv=5)
```

### 4. 集成多個特徵選擇方法
```python
# 投票制：選擇在多個方法中排名靠前的特徵
consensus_features = [f for f in feature_names 
                     if avg_rank(f) < threshold]
```

---

## 參考資源

- **CRISP-DM**: https://en.wikipedia.org/wiki/Cross-industry_standard_process_for_data_mining
- **Scikit-learn Feature Selection**: https://scikit-learn.org/stable/modules/feature_selection.html
- **XGBoost Documentation**: https://xgboost.readthedocs.io/
- **Model Evaluation Metrics**: https://scikit-learn.org/stable/modules/model_evaluation.html

---

## 適用數據集

✅ **最佳適用：**
- 回歸問題 (連續目標變數)
- 混合特徵 (數值 + 分類)
- 中等規模數據 (100-10000樣本, 5-50特徵)

⚠️ **需要調整：**
- 分類問題 → 使用 `f_classif` 或 `mutual_info_classif`
- 大規模數據 → 考慮特徵抽樣或降維
- 時間序列 → 添加時間依賴特徵

---

**最後更新：** 2026-06-15
