# ChainTrace

> **ChainTrace is a personal multi-chain blockchain fund-flow analysis and investigation project for tracing, validating, and analyzing on-chain asset movements.**

ChainTrace 是我自行開發的**多鏈鏈上金流分析與調查工具**，主要用於整理、分析與追蹤不同區塊鏈上的資產流動。

目前專案聚焦於實際鏈上調查中常見的工作，包括：

- 資金流入／流出分析
- Counterparty 分析
- One-Hop 下游候選追蹤
- Multi-Hop 多層資金路徑追蹤
- 不同鏈資料格式的 Normalization
- 調查證據與 Completeness 管理
- Cross-chain 跨鏈事件解析

---

## 目前支援

| Network | Assets | Flow Analysis | One-Hop | Multi-Hop |
|---|---|---:|---:|---:|
| Ethereum | ERC-20 | ✅ | ✅ | ✅ |
| Arbitrum | ERC-20 | ✅ | ✅ | ✅ |
| TRON | TRC-20 / TRX | ✅ | ✅ | ✅ |
| Solana | SPL Token / SOL | ✅ | ✅ | ✅ |

---

## 核心功能

### 1. 資金流分析

ChainTrace 會先將不同區塊鏈的交易資料轉換成統一的 Canonical Model，再使用同一套分析邏輯進行：

- Inflow
- Outflow
- Net Flow
- Counterparty Analysis
- Native Asset / Token 分離
- 精確資產數量計算

資產數值以原始整數與 decimals 進行轉換，不使用浮點數作為主要計算依據。

---

### 2. One-Hop Tracing

One-Hop 用於分析一筆已知來源資金進入目標地址後，可能的下一層資金移動。

分析會保留：

- Asset Identity
- 時間區間
- Amount Rule
- Transaction Evidence
- Query Bounds
- Completeness
- Stop Reason

ChainTrace 將結果標示為 **downstream candidate**。

候選交易並不代表已經證明為「同一筆資金」的直接延續。

---

### 3. Multi-Hop Tracing

目前的 bounded Multi-Hop engine 支援：

- BFS 路徑展開
- Path-local Cycle Prevention
- Address / Contract / Program Classification
- Query Bounds
- Depth Bounds
- Partial Result Preservation
- Completeness Reporting
- Stop Reason

Ethereum、TRON、Solana 的鏈上執行模型不同，因此 Chain-specific 的資料取得與地址判斷會在 Adapter / Provider 層處理，而不是各自建立不同的 tracing engine。

---

## 多鏈架構

### Ethereum / Arbitrum

目前支援 ERC-20 資產：

- Flow Analysis
- One-Hop
- Multi-Hop
- EVM Contract Classification

---

### TRON

目前支援：

- TRC-20
- Native TRX
- Base58Check Address Normalization
- TRON Account Classification
- Flow Analysis
- One-Hop
- Multi-Hop

TRX 與 TRC-20 使用不同 Asset Identity，不使用 fake contract address 表示 Native Asset。

---

### Solana

目前支援：

- SPL Token
- Token-2022 標準 transfer
- Native SOL
- Versioned Transaction
- Inner Instructions / CPI
- Token Account / Owner 分離
- Owner-aware One-Hop
- Multi-Hop

Solana 的 Token Account 與 Owner 不會被視為同一個地址角色。

ChainTrace 也會區分：

- Owner
- Token Account
- Program
- PDA / Off-curve Address
- System Account

避免在資金追蹤時錯誤進行地址歸因。

---

## Canonical Data Model

不同區塊鏈的資料結構並不相同。

例如：

```text
Ethereum / Arbitrum → Transaction Hash + Block Number
TRON                → Transaction ID + Block Number
Solana              → Signature + Slot
```

ChainTrace 使用統一的 Canonical Model 保存：

- `network_identifier`
- `transaction_id`
- `event_id`
- Chain Position
- Asset Identity
- Native Asset
- Token Asset
- Address Role
- Timestamp
- Raw Amount
- Decimals
- Decimal Amount
- Coverage
- Completeness

同時保留每條鏈原本的資料語意，而不是將所有區塊鏈強制套入 EVM 的資料模型。

---

## Cross-chain Resolution

Cross-chain Resolver 目前正在開發中。

目前已完成 protocol-agnostic 的 Cross-chain Resolution Core，可以表示：

- `RESOLVED`
- `PENDING`
- `AMBIGUOUS`
- `UNRESOLVED`
- `UNSUPPORTED`

調查證據則區分：

- `FACT`
- `INFERENCE`
- `HEURISTIC`

### Cross-chain 證據原則

以下資訊：

- 金額相同
- 金額接近
- 時間接近
- Recipient 相同
- Token 相似

只能作為輔助證據。

**僅有 HEURISTIC evidence 時，不會被標記為 `RESOLVED`。**

跨鏈事件必須具有 protocol-native、可驗證且能建立 Origin Event 與 Destination Event 唯一關係的 FACT evidence，才能被判定為 resolved。

目前正在進行第一個 protocol-specific resolver：

```text
Across Resolver
```

---

## 調查原則

ChainTrace 採取較保守的 Evidence Standard。

### Source Address ≠ Victim

資金來源地址不會因為曾轉入目標地址，就自動被判定為受害者。

### Candidate ≠ Same Funds

下游候選交易不代表已證明是同一筆資金。

### Pending ≠ Resolved

尚缺乏 destination evidence 的 Cross-chain event 不會被當成 resolved。

### Infrastructure ≠ End Recipient

Router、DEX、Bridge、Token Account、Program 等基礎設施地址不會直接被視為最終資金接收者。

### Cross-chain Link ≠ New Transfer

CrossChainLink 只負責連接：

```text
Origin Event
↕
Destination Event
```

它不是額外建立的一筆資金 Transfer，因此不會被重複計入 Flow。

---

## Coverage 與 Completeness

ChainTrace 會區分：

### Execution Completeness

本次查詢是否因為：

- Query Limit
- Page Limit
- Record Limit
- API Error
- Classification Error

而提前停止。

### Query Scope

本次實際查詢了哪些：

- Addresses
- Token Accounts
- Transactions
- Networks

### Historical Coverage

即使已完整執行某次查詢，也不一定代表已取得該地址的完整歷史。

例如 Solana 中：

```text
getSignaturesForAddress(wallet)
```

不等於完整 SPL Token 歷史。

目前已關閉的 historical Token Account 也可能無法透過 current state 找回。

因此 ChainTrace 會保留這類 Coverage Limitation，而不是將部分資料描述成完整歷史。

---

## 開發狀態

目前已完成：

```text
Ethereum
├─ ERC-20
├─ Flow
├─ One-Hop
└─ Multi-Hop

Arbitrum
├─ ERC-20
├─ Flow
├─ One-Hop
└─ Multi-Hop

TRON
├─ TRC-20
├─ TRX
├─ Flow
├─ One-Hop
└─ Multi-Hop

Solana
├─ SPL Token
├─ Token-2022
├─ SOL
├─ Flow
├─ One-Hop
└─ Multi-Hop

Cross-chain
└─ Resolution Core
```

目前開發重點：

```text
Protocol-specific Cross-chain Resolver
```

自動化測試目前已累積超過 **1,200 個 passing tests**，涵蓋：

- Adapter
- Provider
- Normalization
- Canonical Model
- Flow
- One-Hop
- Multi-Hop
- Address Classification
- Coverage / Completeness
- Cross-chain Evidence
- Regression Tests

---

## 專案用途

ChainTrace 目前主要作為：

1. 個人鏈上金流調查工具
2. Blockchain Data 分析與研究工具
3. AML / 虛擬資產調查相關技術實作
4. 多鏈資料結構與資金追蹤方法的學習專案
5. 個人技術作品集

---

## Disclaimer

ChainTrace 是獨立的個人研究與開發專案。

本專案與 Ethereum、Arbitrum、TRON、Solana、Across 或本 Repository 中提及的其他 Blockchain Protocol / Organization 均無隸屬或合作關係。

ChainTrace 所輸出的地址關聯、資金路徑與候選結果應視為**技術分析與調查證據**，不應單獨作為對地址所有權、行為意圖、犯罪活動或法律責任的最終認定。
