# 全公司 OKR 監控系統藍圖（製造業）

> 期間：2026–2030  
> 目標：可計算（Progress）＋可視覺（燈號與 Dashboard）＋可預測（Forecast）

---

## 1) 系統總覽

本方案將 OKR 監控系統拆成 4 層：

1. **資料層（Data Layer）**：ERP/MES/SCADA/QMS/WMS/HRIS/財務系統 + 手動填報。
2. **計算層（Metrics Engine）**：KR 進度計算、燈號判定、預測模型。
3. **視覺層（Dashboard）**：公司級、事業部級、工廠級、部門級四層視圖。
4. **治理層（Governance）**：OKR 週會、月檢、季檢、年度校準。

---

## 2) 可計算（Progress）設計

## 2.1 KR 資料模型（建議最小欄位）

每一筆 KR 建議包含：

- `okr_year`（年度）
- `quarter`（Q1~Q4）
- `org_unit`（部門/工廠）
- `owner`
- `objective_id`, `kr_id`
- `kr_name`
- `baseline_value`
- `target_value`
- `actual_value`
- `direction`（higher_better / lower_better）
- `weight`（0~1）
- `data_source`
- `update_frequency`（daily/weekly/monthly）
- `confidence`（0~1，主責人自評）

## 2.2 進度公式（標準化到 0~100%）

### A. 指標「越高越好」

\[
Progress = \frac{Actual - Baseline}{Target - Baseline}
\]

### B. 指標「越低越好」（如不良率、停機率）

\[
Progress = \frac{Baseline - Actual}{Baseline - Target}
\]

### C. 邊界控制

- 若計算值 < 0，記為 0。
- 若計算值 > 1，可記為 1.2（表示超標 120%）或封頂 1（依公司政策）。

### D. Objective 加權分數

\[
Objective\ Score = \sum_{i=1}^{n}(KR\ Progress_i \times KR\ Weight_i)
\]

### E. 公司總分

\[
Company\ Score = \sum_{j=1}^{m}(Objective\ Score_j \times Objective\ Weight_j)
\]

## 2.3 製造業常見 KR 指標庫（可直接套用）

- **QCDSME 維度**：
  - Quality：FPY、PPM、客訴率、COPQ
  - Cost：單位製造成本、能耗成本、報廢成本
  - Delivery：OTD、Lead Time、排程達成率
  - Safety：TRIR、LTI、未遂事件結案率
  - Morale：離職率、訓練完成率、敬業度
  - Environment：碳排強度、廢棄物回收率、用水強度

---

## 3) 可視覺（燈號 + Dashboard）設計

## 3.1 燈號規則（建議）

以「當期應達進度 vs 實際進度」判定：

- 🟢 綠燈：`actual_progress >= planned_progress - 5%`
- 🟡 黃燈：`planned_progress - 15% <= actual_progress < planned_progress - 5%`
- 🔴 紅燈：`actual_progress < planned_progress - 15%`
- ⚪ 灰燈：資料逾期未更新（超過 SLA）

> 範例：第 8 週計畫應達 40%，實際 30% ⇒ 落後 10% ⇒ 黃燈。

## 3.2 Dashboard 版型（四層）

1. **CEO 戰情室（公司總覽）**
   - 公司 OKR 總分趨勢
   - 各事業群燈號分布
   - Top 10 風險 KR（紅燈 + 高權重）
   - 年底達成率預測（P50/P80）

2. **BU/工廠總經理頁**
   - 廠別比較（良率、交付、成本、安全）
   - 每廠 Objective 熱力圖
   - 產線稼動與瓶頸工序

3. **部門經理頁**
   - KR 週進度燃盡圖
   - 行動項目（Action Register）
   - 問題閉環效率（open vs closed）

4. **一線主管頁**
   - 班次 KPI 即時看板
   - 異常告警（停機、報廢、品質）

## 3.3 關鍵圖表清單

- 趨勢線：KR progress over time
- 紅黃綠矩陣：Impact × Delay
- 熱力圖：工廠/產線/部門達成度
- 瀑布圖：目標差距來源拆解（良率、稼動、速度）
- 預測扇形圖：年底達標機率區間

---

## 4) 可預測（Forecast）設計

## 4.1 預測層次

- **短期（4~12 週）**：線性趨勢 + 季節性校正
- **中期（季度）**：情境模擬（Best/Base/Worst）
- **長期（年度）**：結合產能、人力、需求預測

## 4.2 兩段式預測方法（落地友善）

1. **基線模型（先簡後繁）**
   - 線性回歸 / 指數平滑預估 KR 期末值
2. **風險調整模型**
   - 依資料品質、異常頻率、供應風險做折減係數


a) 期末預估值：
\[
Forecast\_EoP = f(time\_series\ of\ actuals)
\]

b) 達標機率：
\[
P(achieve) = P(Forecast\_EoP \ge Target)
\]

c) 預測進度：
\[
Forecast\ Progress = \frac{Forecast\_EoP - Baseline}{Target - Baseline}
\]

## 4.3 2026–2030 推進節奏（Roadmap）

- **2026（Phase 1）**：
  - 建立指標字典、資料欄位標準、手動+半自動更新
  - 上線基礎燈號與週/月儀表板
- **2027（Phase 2）**：
  - 串接 ERP/MES/QMS 自動取數
  - 導入 KR 期末值預測與達標機率
- **2028（Phase 3）**：
  - 建立跨廠 benchmark 與異常根因模型
  - 啟用紅燈自動派工（CAPA 任務）
- **2029（Phase 4）**：
  - 引入情境沙盤（需求波動、供應中斷、能源成本）
  - 董事會級 ESG + 財務一體化 OKR
- **2030（Phase 5）**：
  - AI Copilot 提示「下一步行動」
  - 形成預測驅動的滾動式 OKR 經營體系

---

## 5) 製造業組織圖（範本）與 OKR 對位

## 5.1 組織圖（Manufacturing Enterprise Org Chart）

```text
董事會
└─ CEO / 總經理
   ├─ COO（營運）
   │  ├─ 工廠A 廠長
   │  │  ├─ 生產部（製一/製二/製三）
   │  │  ├─ 設備維護部（機電/預防保養）
   │  │  ├─ 製程工程部（PE/IE）
   │  │  ├─ 品質部（IQC/IPQC/OQC/客訴）
   │  │  ├─ 倉儲與物流部
   │  │  └─ EHS（安衛環）
   │  └─ 工廠B 廠長
   │     └─ （同上）
   ├─ CSCO（供應鏈）
   │  ├─ 採購
   │  ├─ 計畫（S&OP / 生管）
   │  └─ 供應商品質管理（SQE）
   ├─ CFO（財務）
   │  ├─ 成本會計
   │  ├─ 經營分析（FP&A）
   │  └─ 內控與稽核
   ├─ CHRO（人資）
   │  ├─ 招募
   │  ├─ 培訓發展
   │  └─ 組織發展
   ├─ CIO（資訊）
   │  ├─ 數據平台（BI/數據工程）
   │  ├─ 應用系統（ERP/MES）
   │  └─ 資安
   └─ CTO（技術/研發）
      ├─ 產品研發
      ├─ 製程技術中心
      └─ 自動化與工業AI
```

## 5.2 組織層級 OKR 配置範例

### 公司級 Objective（示意）

- O1：2030 年前成為區域內交付最可靠的製造商
- O2：2030 年前單位製造成本下降 25%
- O3：2030 年前達成零重大職災 + 碳強度下降 40%

### 部門對位（舉例）

- COO：OEE、FPY、OTD、停機時數
- CSCO：供應商準交率、缺料停線時數、庫存週轉
- CFO：毛利率、單位成本、現金轉換週期
- CHRO：關鍵崗位補實率、訓練時數、離職率
- CIO：資料可用率、報表時效、系統穩定度
- CTO：新製程導入週期、良率爬坡速度

---

## 6) 首批 90 天落地計畫（你可以直接執行）

### 第 1–30 天：定義期
- 完成公司/事業部/工廠三級 OKR 樹
- 完成 KR 指標字典（公式、口徑、資料源、更新頻率）
- 定義燈號與例外處理（含灰燈）

### 第 31–60 天：上線期
- 建立 MVP Dashboard（公司頁 + 工廠頁）
- 每週固定節奏更新 KR（週一更新、週三檢討）
- 建立紅燈處置流程（責任人 + 截止日 + 驗證方式）

### 第 61–90 天：優化期
- 加入 Forecast（期末值與達標機率）
- 追蹤預測誤差（MAPE）與模型校準
- 啟動跨廠 benchmark 與最佳實務分享

---

## 7) 你可以直接複製的 KPI/OKR 欄位模板

- `KR_ID`
- `KR_Name`
- `Owner`
- `Plant`
- `Baseline`
- `Target`
- `Actual`
- `Progress%`
- `Planned_Progress%`
- `Signal`（G/Y/R/Gray）
- `Forecast_EoP`
- `P_Achieve`
- `Confidence`
- `Risk`
- `Action`
- `Due_Date`
- `Status`

---

## 8) 建議你下一步（我可接續幫你）

1. 我可以下一步幫你產出「**可直接上 BI 的資料表 schema（SQL）**」。
2. 我可以幫你做「**製造業 3 層 OKR 範例（公司→工廠→部門）100 條 KR 清單**」。
3. 我可以幫你做「**Dashboard 線框圖 + 欄位對應表**」。
4. 我可以幫你做「**Forecast 的 Python 範例（含達標機率）**」。

