# 新世代司法公開書類查詢與 AI 知識平台建置提案草案

## 1. 提案目的

本提案旨在規劃一套可取代傳統公開書類查詢網站、並可支援未來 AI 應用的「司法公開書類查詢與 AI 知識平台」。

系統設計目標如下：

1. 承載既有約 1,000 萬筆司法相關公開書類資料。
2. 支援每年約 40 萬筆新增資料的穩定匯入與索引。
3. 提供傳統條件查詢、全文檢索、語意搜尋與 AI 問答能力。
4. 建立資料治理、個資遮隱、版本控管、稽核紀錄與安全控管機制。
5. 避免過度依賴社群版 open database，改以企業級商用資料庫、託管式服務或具長期支援合約的技術組合作為核心。
6. 建立可支援未來 10 至 20 年擴充的資料與 AI 架構。

## 2. 核心設計原則

### 2.1 不做單純網站複製，而是重建資料平台

既有公開書類查詢系統的精髓不在於畫面，而在於：

- 條件式查詢。
- 全文檢索。
- 書類分類。
- 文件公開流程。
- 個資遮隱。
- 查詢結果可追溯。
- 法律資料的可信度與穩定性。

因此，本提案建議將新系統定位為「司法文件知識基礎平台」，而非單純的網頁查詢系統。

### 2.2 避免社群版 open DB 成為核心風險

若系統需服務 10 至 20 年，核心資料庫不建議完全依賴缺乏商業支援或維運責任歸屬不明的社群版 open DB。建議採用以下方向：

- 企業級商用資料庫。
- 雲端託管式關聯式資料庫。
- 具有原廠支援、長期支援版本與 SLA 的資料平台。
- 搜尋與 AI 索引層可使用商業版或託管式服務。
- 核心交易資料與原始文件必須保留於可長期治理的主資料平台。

此設計可降低未來升級、資安修補、備援、權責、維護人力與技術斷層風險。

### 2.3 AI-ready，但不讓 AI 成為唯一真實來源

AI 應用必須建立在可信資料與檢索結果之上。系統應採用 Retrieval-Augmented Generation，簡稱 RAG，讓 AI 回答必須根據檢索到的文件、片段與引用來源生成。

AI 不應直接連接資料庫，也不應繞過權限控管。正確流程應為：

```txt
使用者 → 身分驗證 → 權限過濾 → 搜尋與檢索 → AI 回答 → 引用來源 → 稽核紀錄
```

## 3. 建議整體架構

```txt
                使用者入口
       查詢系統 / AI 問答 / 文件閱讀器 / 分析儀表板
                         │
                         ▼
                  API Gateway
       身分驗證 / 權限控管 / 流量限制 / 稽核紀錄
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
  Metadata API      Search API        AI/RAG API
  案件欄位查詢       全文與語意搜尋       摘要/問答/類案
        │                │                │
        ▼                ▼                ▼
企業級主資料庫     企業級搜尋平台      AI Gateway
Oracle/SQL Server  Elastic/Cloud Search LLM/Embedding/Rerank
        │                │                │
        └───────────────┬┴────────────────┘
                        ▼
                 Object Storage
       原始文件 / 遮隱後文件 / 版本 / PDF / JSON
                        │
                        ▼
                 Data Pipeline
       匯入 / 清洗 / 個資遮隱 / 審核 / 索引 / AI 切片
                        │
                        ▼
                  來源系統
       法務、司法、內部資料源、批次匯入、API 匯入
```

## 4. 技術選型建議

### 4.1 主資料庫選型

主資料庫負責保存案件 metadata、書類狀態、發布狀態、遮隱狀態、審核流程、版本紀錄、權限與稽核資料。

建議優先考慮以下企業級方案：

| 方案 | 適用情境 | 優點 | 注意事項 |
|---|---|---|---|
| Oracle Database / Oracle Exadata / Oracle Autonomous Database | 政府、大型金融、長期核心系統 | 成熟、穩定、高可用、企業支援完整 | 授權與維運成本較高 |
| Microsoft SQL Server Enterprise / Azure SQL Managed Instance | 微軟生態、內部系統整合、政府常見環境 | 管理工具成熟、維運人才較多、企業支援完整 | 大規模全文與 AI 索引仍建議外掛搜尋平台 |
| IBM Db2 | 大型機關、既有 IBM 生態 | 穩定、交易能力強、企業支援 | 技術人才與整合成本需評估 |
| Cloud Spanner / AlloyDB Enterprise Plus / Aurora with Enterprise Support | 雲端原生、高可用、多區部署 | 託管式、擴充性高、維運負擔低 | 需考慮資料主權、採購與雲端政策 |

本提案建議：

- 若偏政府機房或嚴格資料主權：優先評估 Oracle Database 或 SQL Server Enterprise。
- 若可採雲端託管：優先評估 Azure SQL Managed Instance、Oracle Autonomous Database、Cloud Spanner 或具企業支援的雲端資料庫。
- 若未來需要跨區域高可用與自動擴充：可評估 Cloud Spanner 或同級分散式關聯資料庫。

### 4.2 搜尋平台選型

主資料庫不應承擔全部全文檢索與語意檢索壓力。建議獨立建置搜尋平台。

| 方案 | 適用情境 | 優點 | 注意事項 |
|---|---|---|---|
| Elastic Stack Enterprise / Elastic Cloud | 全文檢索、hybrid search、企業支援 | 成熟、功能完整、可商業支援 | 授權成本需評估 |
| Azure AI Search | 微軟雲端、文件搜尋、RAG | 託管式、與 Azure OpenAI 整合佳 | 雲端綁定度較高 |
| Google Vertex AI Search / Discovery Engine | Google Cloud 生態、AI 搜尋 | AI 原生搜尋能力強 | 需評估資料主權與雲端政策 |
| AWS OpenSearch Service with Enterprise Support | AWS 生態、託管搜尋 | 託管式、擴充性佳 | 技術治理需規劃 |

若明確不希望使用社群版 open database，建議採用「商業支援版或雲端託管式搜尋服務」，不要自行維護社群版搜尋叢集。

### 4.3 AI 平台選型

AI 層不應綁死單一模型供應商，建議建立 AI Gateway。

AI Gateway 負責：

- 模型路由。
- Prompt 版本管理。
- Embedding 模型管理。
- Reranking 模型管理。
- 成本控管。
- Token 使用紀錄。
- AI 回答稽核。
- 權限過濾。
- 引用來源檢查。
- 幻覺風險檢查。

可支援模型來源：

- OpenAI API。
- Azure OpenAI。
- Google Gemini。
- Anthropic Claude。
- 私有化部署大型語言模型。
- 未來新增模型。

### 4.4 文件儲存

文件本體不建議全部存放在主資料庫欄位內。建議使用企業級物件儲存：

- AWS S3。
- Azure Blob Storage。
- Google Cloud Storage。
- Oracle Object Storage。
- 企業機房 S3-compatible object storage。

需啟用：

- 版本控管。
- 加密。
- Object lock 或 WORM 保護。
- 存取稽核。
- 生命週期管理。
- 冷熱分層。

## 5. 資料庫與資料模型設計

### 5.1 主資料表

```sql
CREATE TABLE documents (
  id BIGINT PRIMARY KEY,
  document_uid VARCHAR(64) NOT NULL UNIQUE,
  source_system VARCHAR(50) NOT NULL,
  agency_code VARCHAR(20) NOT NULL,
  agency_name NVARCHAR(100) NOT NULL,
  case_year INT,
  case_type NVARCHAR(50),
  case_number_start INT,
  case_number_end INT,
  document_type NVARCHAR(50) NOT NULL,
  case_reason NVARCHAR(200),
  decision_date DATE,
  publication_date DATE,
  visibility_status VARCHAR(30) NOT NULL,
  redaction_status VARCHAR(30) NOT NULL,
  language VARCHAR(10) DEFAULT 'zh-TW',
  checksum_sha256 CHAR(64),
  created_at DATETIME2 NOT NULL,
  updated_at DATETIME2 NOT NULL
);
```

### 5.2 文件內容表

```sql
CREATE TABLE document_contents (
  id BIGINT PRIMARY KEY,
  document_id BIGINT NOT NULL,
  content_version INT NOT NULL,
  raw_text_uri NVARCHAR(1000),
  redacted_text_uri NVARCHAR(1000),
  structured_json_uri NVARCHAR(1000),
  created_at DATETIME2 NOT NULL,
  UNIQUE(document_id, content_version)
);
```

### 5.3 AI 切片表

```sql
CREATE TABLE document_chunks (
  id BIGINT PRIMARY KEY,
  document_id BIGINT NOT NULL,
  chunk_index INT NOT NULL,
  chunk_text_uri NVARCHAR(1000),
  section_type VARCHAR(50),
  token_count INT,
  embedding_model VARCHAR(100),
  embedding_version VARCHAR(50),
  embedding_vector_id VARCHAR(200),
  created_at DATETIME2 NOT NULL,
  UNIQUE(document_id, chunk_index, embedding_model, embedding_version)
);
```

### 5.4 稽核紀錄表

```sql
CREATE TABLE audit_logs (
  id BIGINT PRIMARY KEY,
  actor_id VARCHAR(100),
  actor_type VARCHAR(50),
  action VARCHAR(100) NOT NULL,
  resource_type VARCHAR(100),
  resource_id VARCHAR(100),
  request_ip VARCHAR(50),
  user_agent NVARCHAR(1000),
  metadata_json NVARCHAR(MAX),
  created_at DATETIME2 NOT NULL
);
```

## 6. 資料流設計

### 6.1 文件匯入流程

```txt
來源資料產生
  │
  ▼
資料接收區 Landing Zone
  │
  ▼
格式驗證與 checksum 建立
  │
  ▼
原始文件封存
  │
  ▼
文字抽取 / OCR / 格式轉換
  │
  ▼
Metadata 解析
  │
  ▼
個資與敏感資訊偵測
  │
  ▼
遮隱版本產生
  │
  ▼
人工抽驗 / 審核流程
  │
  ▼
發布狀態確認
  │
  ▼
主資料庫寫入
  │
  ▼
全文索引建立
  │
  ▼
AI chunk 切片
  │
  ▼
Embedding 產生
  │
  ▼
向量索引建立
  │
  ▼
前台可查詢與 AI 可引用
```

### 6.2 事件驅動架構

建議使用 Kafka、Redpanda、Azure Event Hubs、Google Pub/Sub 或 AWS EventBridge 等企業級事件平台。

核心事件：

- document.received
- document.validated
- document.extracted
- document.redacted
- document.reviewed
- document.approved
- document.indexed
- document.embedded
- document.published
- document.withdrawn

事件化設計的好處：

- 任務可非同步處理。
- 單一步驟失敗不影響整體系統。
- 可重跑 pipeline。
- 可追蹤文件生命週期。
- 未來 AI 模型或搜尋引擎更新時可批次重建索引。

## 7. AI 功能規劃

### 7.1 第一階段：AI 輔助搜尋

功能包含：

- 自然語言轉查詢條件。
- 關鍵字擴展。
- 同義詞建議。
- 查詢語句改寫。
- 搜尋結果摘要。
- 相似文件推薦。

### 7.2 第二階段：AI 文件閱讀

功能包含：

- 單篇文件摘要。
- 犯罪事實整理。
- 涉及法條整理。
- 證據清單整理。
- 時間線整理。
- 人物與組織關係整理。
- 金流資訊整理。

### 7.3 第三階段：多文件 RAG 問答

功能包含：

- 多文件比較。
- 類案檢索。
- 趨勢整理。
- 法條與案由歸納。
- 研究報告初稿生成。
- 具引用來源的回答。

### 7.4 第四階段：AI 資料治理

功能包含：

- 個資遮隱建議。
- 文件分類。
- 案由標準化。
- 法條抽取。
- OCR 品質偵測。
- 異常文件偵測。
- 重複文件偵測。
- 搜尋品質改善建議。

## 8. AI 安全與治理

AI 回答必須遵守以下原則：

1. 所有回答必須附引用來源。
2. AI 不得宣稱查不到資料時仍給出確定結論。
3. AI 不得繞過資料權限。
4. AI 回答應明確標示非法律意見。
5. AI 摘要、分類與推論需保存模型版本與生成時間。
6. 高風險功能需人工覆核。
7. 需建立測試集與回歸測試，避免模型更新後品質下降。
8. 需建立 prompt injection 防護與輸入輸出過濾。

## 9. 容量與效能估算

### 9.1 資料成長估算

目前假設：

- 現有資料：約 1,000 萬筆。
- 每年新增：約 40 萬筆。
- 10 年後：約 1,400 萬筆。
- 20 年後：約 1,800 萬筆。

若每份文件平均純文字大小為 8KB 至 30KB，考量原始版本、遮隱版本、結構化資料、全文索引與 AI embedding，整體平台資料量可能達數 TB 至數十 TB。

### 9.2 效能目標

| 項目 | 目標 |
|---|---:|
| Metadata 條件查詢 | 300ms 至 1 秒 |
| 一般全文搜尋 | 1 至 3 秒 |
| 複雜查詢 | 3 至 5 秒 |
| AI 摘要 | 3 至 15 秒 |
| 多文件 AI 問答 | 5 至 30 秒 |
| 報告生成 | 30 秒至數分鐘 |

## 10. 資安與法遵設計

建議納入：

- 身分驗證與多因素驗證。
- 角色權限控管。
- 文件層級權限控管。
- API rate limit。
- 查詢行為稽核。
- 資料加密傳輸與靜態加密。
- 金鑰管理。
- 弱點掃描。
- 滲透測試。
- 個資遮隱規則。
- 敏感文件撤下流程。
- AI 使用紀錄。
- 資料存取最小權限。
- 備份與災難復原演練。

## 11. 工期規劃

### Phase 0：需求盤點與 PoC

工期：4 至 6 週。

工作內容：

- 現行系統功能拆解。
- 資料格式盤點。
- 文件樣本分析。
- 個資遮隱規則盤點。
- 使用者角色訪談。
- 查詢需求分析。
- AI 使用情境設計。
- 技術選型 PoC。
- 資安與法遵需求盤點。

預估費用：新台幣 150 萬至 400 萬。

### Phase 1：MVP 原型

工期：3 至 4 個月。

工作內容：

- 前台查詢頁。
- 文件列表與詳情頁。
- 主資料庫 schema。
- 全文搜尋索引。
- 少量資料匯入。
- 基本遮隱流程。
- 基本 AI 摘要。
- 基本語意搜尋。
- 管理後台雛形。
- 初步部署與測試。

建議資料量：10 萬至 100 萬筆。

預估費用：新台幣 800 萬至 1,800 萬。

### Phase 2：正式版 V1

工期：6 至 9 個月。

工作內容：

- 完整資料模型。
- 企業級主資料庫部署。
- 企業級搜尋平台部署。
- Object storage 與版本控管。
- 資料匯入 pipeline。
- 個資遮隱 workflow。
- 審核後台。
- 搜尋品質後台。
- AI RAG 問答基礎版。
- 權限管理。
- 稽核紀錄。
- 監控告警。
- CI/CD。
- 效能測試。
- 資安測試。
- 1,000 萬筆資料 migration。

預估費用：新台幣 3,000 萬至 8,000 萬。

若包含政府等級資安驗收、無障礙規範、滲透測試、正式教育訓練與完整維運文件，可能提高至新台幣 8,000 萬至 1.2 億。

### Phase 3：AI 強化版 V2

工期：6 至 12 個月。

工作內容：

- 多文件問答。
- 類案搜尋。
- 法條抽取。
- 法律知識圖譜。
- 自動摘要。
- 結構化抽取。
- 趨勢分析 dashboard。
- AI 遮隱輔助。
- AI 回答品質評測。
- Prompt 與模型版本管理。
- RAG 回歸測試。
- 人工回饋學習機制。

預估費用：新台幣 2,000 萬至 6,000 萬。

### Phase 4：平台化與長期擴充

工期：12 至 24 個月，持續演進。

工作內容：

- 多資料源整合。
- 多租戶架構。
- API 平台。
- 研究資料集。
- 開放資料介面。
- 高階統計分析。
- AI agent workflow。
- 法律知識圖譜深化。
- 資料品質自動化。
- 跨機關資料交換。
- 內外部權限分層。

預估費用：新台幣 5,000 萬至 2 億以上，依範圍而定。

## 12. 年度維運預算

| 類型 | 年度預估費用 |
|---|---:|
| 小型 MVP 維運 | 新台幣 100 萬至 300 萬 |
| 中型正式服務 | 新台幣 500 萬至 1,500 萬 |
| 高可用與 AI 查詢服務 | 新台幣 1,500 萬至 5,000 萬以上 |

年度維運需包含：

- 雲端或機房資源。
- 資料庫授權。
- 搜尋平台授權。
- AI API 或 GPU 成本。
- 監控與資安工具。
- 備份與災難復原。
- SRE / DevOps。
- Backend / Data / AI 工程維護。
- 資安與法遵顧問。
- 定期弱掃與滲透測試。

## 13. 建議團隊配置

### MVP 團隊

- Project Manager：1 人。
- Solution Architect：1 人。
- Backend Engineer：2 人。
- Frontend Engineer：1 至 2 人。
- Data Engineer：1 人。
- AI Engineer：1 人。
- DevOps Engineer：1 人。
- QA Engineer：1 人。
- UX/UI Designer：0.5 至 1 人。

### 正式版團隊

- Project Manager：1 人。
- Product Owner：1 人。
- Solution Architect：1 至 2 人。
- Backend Engineer：3 至 5 人。
- Frontend Engineer：2 至 3 人。
- Data Engineer：2 至 3 人。
- AI Engineer：2 至 3 人。
- DevOps / SRE：2 人。
- QA Engineer：2 人。
- UX/UI Designer：1 至 2 人。
- Security Consultant：0.5 至 1 人。
- Legal / Domain Expert：1 至 2 人。

## 14. 建議採購與技術策略

### 14.1 若偏好政府機房或私有雲

建議組合：

- 主資料庫：Oracle Database / SQL Server Enterprise。
- 搜尋平台：Elastic Enterprise 自建叢集。
- 文件儲存：企業級 object storage。
- 事件平台：Kafka enterprise support 或商用等級事件平台。
- AI：私有化 AI Gateway，外接雲端或私有模型。
- 部署：Kubernetes / OpenShift。

### 14.2 若可採用公有雲或混合雲

建議組合 A，Microsoft 生態：

- 主資料庫：Azure SQL Managed Instance。
- 搜尋：Azure AI Search。
- AI：Azure OpenAI。
- 儲存：Azure Blob Storage。
- 事件：Azure Event Hubs。
- 監控：Azure Monitor。

建議組合 B，Oracle 生態：

- 主資料庫：Oracle Autonomous Database。
- 搜尋：Elastic Cloud 或 Oracle Cloud Search 類服務。
- AI：OCI Generative AI 或外接 AI Gateway。
- 儲存：Oracle Object Storage。
- 事件：OCI Streaming。

建議組合 C，多雲可攜式：

- 主資料庫：企業級關聯式資料庫。
- 搜尋：Elastic Enterprise。
- AI：AI Gateway 抽象層。
- 儲存：S3-compatible object storage。
- 事件：Kafka。
- 部署：Kubernetes。

## 15. 主要風險與因應

| 風險 | 說明 | 因應方式 |
|---|---|---|
| 資料品質不足 | 舊資料格式不一致、缺欄位、OCR 錯誤 | 建立資料清洗與品質分級 |
| 個資遮隱不完整 | 司法文件包含高度敏感資訊 | 規則 + AI + 人工抽驗 |
| AI 幻覺 | AI 可能產生無依據回答 | 強制引用來源與回答檢查 |
| 成本失控 | AI 查詢與授權成本可能快速增加 | AI Gateway 成本控管與快取 |
| 技術綁定 | 過度綁定單一廠商 | 使用抽象層與可替換架構 |
| 查詢效能不足 | 全文與 AI 檢索負載大 | 搜尋叢集、快取、索引分層 |
| 維運人力不足 | 大型資料平台需要專業維運 | 採託管服務與原廠支援 |
| 法遵與資安風險 | 涉及公開司法資料與個資 | 完整稽核、權限、資安測試 |

## 16. 結論

本系統不建議以「複製既有網站」為目標，而應重新設計為可長期擴充的司法文件知識平台。

核心建議如下：

1. 使用企業級主資料庫管理 metadata、狀態、流程、權限與稽核。
2. 使用商業支援或託管式搜尋平台承擔全文檢索與語意搜尋。
3. 使用 object storage 保存原始文件、遮隱版本與結構化版本。
4. 使用事件驅動 pipeline 處理匯入、遮隱、索引與 AI embedding。
5. 建立 AI Gateway，避免直接綁死單一模型或讓 AI 直接接觸資料庫。
6. 以 RAG 為 AI 問答核心，所有 AI 回答必須可引用、可追溯、可稽核。
7. 透過資料治理、版本控管與可重建索引能力，確保系統可支援未來 10 至 20 年使用。

若以正式平台為目標，建議採分階段推進：

- 4 至 6 週完成需求盤點與 PoC。
- 3 至 4 個月完成 MVP。
- 6 至 9 個月完成正式版 V1。
- 6 至 12 個月導入 AI 強化功能。
- 12 至 24 個月進入平台化與長期擴充。

整體預算建議以新台幣 3,000 萬至 8,000 萬作為正式版 V1 的初步規劃區間；若目標是 AI 原生司法知識平台，則應以新台幣 8,000 萬至 2 億以上作為中長期建置與擴充預算評估基準。
