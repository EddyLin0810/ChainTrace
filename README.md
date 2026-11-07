# ChainTrace

> 多鏈鏈上金流分析調查工具

我是從鏈上金流調查的過程中，多次嘗試各種市面上各相關分析工具，發現某些功能以及特定的數據統整，是我在調查時需要的。但沒有一個平台足以完善至完全符合我在做調查時的需求。因此規劃透過 AI 輔助開發 ChainTrace。它將不同區塊鏈上的交易資料整理成一致的分析格式，用來查看地址的資金流入、流出、交易對手，以及可能的後續資金去向。

研究鏈上案件時，我經常遇到幾個問題：各條鏈的資料格式不同；一筆 Transaction 可能包含多筆資產轉移；資金經過 Swap、Router 或 Bridge 後，也容易在統整時被重複計算。我把這些處理步驟做成可重複使用的工具，並讓每項分析結果都能回頭核對交易跟查詢條件。

目前開發版本包含 CLI、FastAPI 後端、React 前端與 Windows 封裝程式。不同網路的支援範圍仍有差異，詳細列在下方。

**公開範圍**：此 Repository 用於展示專案與技術文件，尚未公開完整原始碼、安裝包與測試環境。

### 操作畫面：調查概覽

![ChainTrace 調查概覽：資料範圍、交易與經濟事件、涉及地址](assets/screenshots/01-investigation-overview.png)

圖 1：① 查看網路、查詢範圍與 Coverage；② 分別呈現交易筆數與經濟事件數；③ 查看涉及地址數量。畫面顯示的是示範地址目前取得的資料，Coverage 為 `Partial`，不代表完整交易歷史。

## 主要功能

### 資金流入與流出分析

- **Inflow / Outflow / Net Flow**：依資產分別計算流入、流出及淨流量。
- **Counterparty Analysis**：整理往來地址、交易次數、資產數量與集中度，協助辨識值得進一步查看的交易對手。
- **Canonical Data Model**：將不同公鏈的資料正規化為共同格式，同時保留各鏈特有的交易資訊。
- **Amount Precision**：以原始整數金額與 `decimals` 計算資產數量。
- **Query Scope / Coverage**：保留查詢範圍、資料量及限制，讓使用者知道分析結果涵蓋哪些紀錄。

![ChainTrace 資產進出：流入、流出、淨流量與待確認數值](assets/screenshots/02-fund-flow.png)

圖 2：① 依欄位比較 Inflow、Outflow 與 Net Flow；② 分開查看 Ethereum ETH 與原生 BNB 的數量；③ 保留 DAI 的待確認資訊。「待確認」不能解讀成零或已確認餘額。截圖來自 `v0.4.0 · build ca40a106`，展示原生資產在畫面上的呈現方式。單跳與多跳的支援範圍以「支援網路」表為準。

### 關聯地址與篩選

![ChainTrace 關聯地址：網路與方向篩選、進階條件及地址清單](assets/screenshots/03-related-addresses.png)

圖 3：① 以 BNB Smart Chain 與 Inflow 篩選；② 顯示地址、交易識別、行為分類及 Contract／Mint 等篩選欄位；③ 列出目前條件下的關聯地址。

### 單跳與多跳候選追蹤

ChainTrace 可從已知來源交易出發，根據資產、時間及其他查詢條件，找出下一層可能相關的轉出紀錄，再以有界的 BFS（Breadth-First Search）延伸多層候選路徑。

每次追蹤會記錄時間窗、深度與查詢上限，並保留相關交易、停止原因及資料完整度。同一路徑會略過已走訪的節點，避免反覆展開。

**如何解讀結果**：系統找到的是 *downstream candidates*。由於代幣具有可替代性，即使兩筆交易在時間與金額上接近，也不足以認定它們移動的是同一批資產。候選路徑必須和已確認的金流證據分開看待。

## 支援網路

下表依本機開發版本 `2f9d677` 整理。✅ 表示已有實作與測試紀錄，資產範圍以表內說明為準。

| 網路 | 資產／標準 | 金流分析 | 單跳追蹤 | 多跳追蹤 | 範圍與限制 |
| --- | --- | :---: | :---: | :---: | --- |
| Ethereum | ERC-20 | ✅ | ✅ | ✅ | Native ETH 與 internal movement 尚未整合為通用追蹤功能；分析結果受查詢範圍限制。 |
| Arbitrum | ERC-20 | ✅ | ✅ | ✅ | 已有真實資料查詢紀錄；跨鏈路徑仍須逐一驗證。 |
| TRON | TRC-20 / native TRX | ✅ | ✅ | ✅ | 支援有界分析，已有歷史樣本；最新版本尚未重新完成全面 live validation。 |
| Solana | SPL Token / native SOL | ✅ | ✅ | ✅ | 支援有界追蹤，Token Account 歷史資料可能不完整。 |
| BNB Smart Chain | BEP-20 | ✅ | 尚未完成 | 尚未完成 | 本版支援 BEP-20 金流分析。原生 BNB、內部交易與通用協定解碼未列入本版支援範圍。 |

### 各鏈資料處理

**Ethereum / Arbitrum**

目前以 ERC-20 分析為主，透過 EVM Provider / Adapter 處理資料查詢、正規化與地址分類。合約地址在最新區塊的狀態，不一定等於歷史交易發生當下的狀態。

**TRON**

分別處理 TRC-20 與 native TRX，包含 Base58Check 地址正規化、帳戶類型判斷，以及兩種資產的識別方式。Native TRX 不會使用虛構的 Token Contract Address。

**Solana**

處理 SPL Token、Token-2022、native SOL，以及 versioned transactions、inner instructions / CPI。分析時會區分 Wallet Owner、Token Account、Program、PDA / Off-curve Address 與 System Account，避免把不同地址角色直接當成同一個控制者。

**BNB Smart Chain**

透過 Alchemy 取得 BEP-20 流入與流出紀錄。工具將轉帳事件轉成統一格式、去除重複紀錄，再在查詢介面呈現金流統計。

## 跨鏈解析

ChainTrace 已建立不依賴單一協定的 Cross-chain Resolution Core，並實作 **Across、Mayan / Swift V2、Symbiosis** 特定路由的 resolver。目前有程式與測試紀錄，但不能用這些結果推定所有協定、鏈別或轉移方向都能解析。

跨鏈事件使用下列狀態記錄處理進度：

`RESOLVED` · `PENDING` · `AMBIGUOUS` · `UNRESOLVED` · `UNSUPPORTED`

證據則區分為 `FACT`、`INFERENCE` 與 `HEURISTIC`。

金額接近、發生時間相近或接收地址相同，都可能是尋找跨鏈交易的線索，但仍屬輔助判斷。要將 Origin Event 與 Destination Event 標記為 `RESOLVED`，還需要符合該協定的可核對證據及唯一性條件。

目前保存了特定 Ethereum → Arbitrum 路由的歷史主網驗證報告；尚未完成所有跨鏈方向、路由與交易情境的最新即時驗證。

## 調查資料與證據

我用 `EconomicEvent` 模型區分原始轉帳紀錄與要分析的資金事件。整理案件資料時，我會用下列原則判讀結果：

| 概念 | 處理方式 |
| --- | --- |
| **Transfer Record vs. Economic Event** | 同一筆交易可能產生多筆 transfer records，紀錄數量不能直接當成獨立資金事件數。 |
| **Source Address vs. Victim** | 某個地址曾轉出資產，不足以證明地址控制者就是受害者。 |
| **Candidate vs. Confirmed Flow** | 下游候選交易和已有充分證據支持的資金連結分開呈現。 |
| **Router / Swap / Bridge** | 對資產交換、路由及跨鏈行為進行分類，避免未經判斷就重複計入外部資金流入。 |
| **Pending vs. Resolved** | 缺少關鍵來源、目的端或歸因證據時，保留未解狀態。 |

目前部分歸因與防重複計算流程仍與特定研究案件綁定，尚未成為適用所有鏈及協定的通用自動化功能。ChainTrace 也無法單靠鏈上紀錄，自動確認受害者身分、地址所有權或最終損失。

## 系統運作方式

```text
Blockchain Data Providers
          ↓
Chain-specific Adapters / Acquisition
          ↓
Normalization & Canonical Models
          ↓
Flow / Candidate Tracing / EconomicEvent Processing
          ↓
Investigation Results & Evidence / Coverage Metadata
          ↓
CLI · FastAPI · React UI · Windows Application
```

跨鏈解析與案件流程會依各自需要的資料和證據接入，並不是每個模組都已在所有公鏈上完整串接。

Canonical Model 會區分網路、交易、事件、資產及地址角色。各鏈原始識別資訊也會保留，例如 EVM 的 Transaction Hash / Block Number、TRON 的 Transaction ID / Block Number，以及 Solana 的 Signature / Slot。資產不會只憑 Token Symbol 合併，以免不同網路或合約的同名代幣被算在一起。

## 既有驗證紀錄

以下為本機開發版本 `2f9d677`（2026/11/05）的驗證摘要。

| 項目 | 紀錄 |
| --- | --- |
| Python 回歸測試 | 3,598 項通過 |
| React 前端測試 | 179 項通過 |
| BNB Smart Chain 實際查詢 | 以 Windows 封裝程式驗證有限範圍內的查詢與操作 |

Ethereum、Arbitrum、TRON、Solana 與特定跨鏈路由也有歷史驗證紀錄，涵蓋的版本與樣本各有不同。

## 調查案例：0x91cd

我曾將 ChainTrace 的部分數據蒐集及其處理流程，應用於 **iToken Wallet 0x91cd** 的鏈上調查。Ethereum 案件資料共保存 **424 筆入金紀錄**，涉及 **422 個不同的交易雜湊**。

這項研究也包含人工判讀、其他分析工具與案件專用腳本。我用這些方法核對資金來源、去除重複的經濟事件，並辨識交換、路由或跨鏈過程中的資產移動。證據不足的紀錄仍保留待確認狀態，受害者身分與最終損失金額也尚未確認。

## 目前限制

- **資料範圍**：結果受資料供應商、時間窗、分頁及查詢上限影響，可能只涵蓋地址的部分交易歷史。
- **協定解碼與估值**：目前僅支援特定協定及路由，尚未提供通用協定解碼或完整的歷史美元估值。

## AI 輔助開發

我在規劃需求、撰寫程式及測試時使用了 Codex 等 AI 輔助工具。ChainTrace 實際分析交易時，依靠的是鏈上資料、程式規則與可核對的證據。**目前沒有在執行流程中呼叫 LLM API，也不是自主執行調查的 AI Agent。**

## 專案狀態

ChainTrace 是我的個人鏈上調查研究與開發專案。
