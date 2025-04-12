# ai_service_pic# AI商品諮詢服務

一個基於LINE平台的智能客服系統，能夠回答產品相關問題、搜索商品資訊，並提供友善的購物指引。此系統整合 OpenAI 大型語言模型與向量資料庫 FAISS，達到高效率、精準搜尋與聊天服務；並提供管理後台供管理員監控用戶、對話與產品資料。

## 目錄

- [系統概述](#系統概述)
- [功能特色](#功能特色)
- [系統架構](#系統架構)
- [技術棧](#技術棧)
- [檔案架構及說明](#檔案架構及說明)
  - [核心服務](#核心服務)
  - [資料庫相關](#資料庫相關)
  - [AI與向量資料庫](#ai與向量資料庫)
  - [管理後台](#管理後台)
  - [工具與輔助腳本](#工具與輔助腳本)
- [檔案關聯與數據流程](#檔案關聯與數據流程)
- [安裝與設定](#安裝與設定)
- [使用指南](#使用指南)
- [開發者指南](#開發者指南)
- [故障排除](#故障排除)
- [客戶說明](#客戶說明)

## 系統概述

本系統主要提供智能客服功能，藉由 LINE 消息平台接收用戶問題，使用 OpenAI GPT 模型進行自然語言處理並與向量資料庫進行語意檢索，幫助用戶快速找到相關產品資訊。系統支持群組模式、單一聊天以及管理員後台操作，並具備自動備份與還原向量資料庫的功能。

## 功能特色

### 用戶端功能
- **自然語言理解**：處理各種形式的產品詢問與多輪對話  
- **精準產品搜尋**：利用向量檢索技術進行語意比對  
- **產品詳情查詢**：透過產品編號或關鍵詞直接查詢

### 管理功能
- **用戶管理**：查看並管理用戶基本資訊與對話歷史  
- **簡化產品管理**：直觀的產品增刪改查介面，不再需要手動分類  
- **AI智能分類**：使用先進的 LLM 模型自動為商品分類，提高分類準確度  
- **統計數據**：展示用戶數、消息數、產品數量等統計信息  
- **向量資料庫管理**：支持重建、更新、備份與還原向量資料庫
- **安全的帳號管理**：全新登入系統，支持"記住我"功能與連續登入
- **跨設備優化**：後台管理介面完全相容於電腦、平板與手機，支援ngrok遠端訪問

## 系統架構

系統採用分層設計，主要分為四個層次：

1. **前端介面層**
   - LINE 消息介面
   - 管理後台網頁界面（簡化的產品管理介面）
   - 行動裝置適配層（自動調整成行動版，提供最佳使用體驗）

2. **應用邏輯層**
   - 消息處理（由 main.py 和 line_handler.py 負責）
   - 用戶與產品數據管理（由 mariadb_storage.py、import_manager.py、admin_api.py 處理）
   - 安全認證層（提供穩定的認證機制，支援連續操作不再重複登入）

3. **AI與向量資料庫層**
   - AI代理（agent.py 使用 LangChain 框架調用 OpenAI 模型）
   - 智能產品分類（vector_db_manager.py 使用 LLM 進行準確分類）
   - 向量資料庫管理（vector_db_manager.py、create_improved_vectordb.py、reload_vector_db.py）

4. **資料存儲層**
   - MariaDB：存儲用戶、消息與產品數據  
   - FAISS 向量資料庫：存儲產品向量表示以利實現語意搜尋
   - 本地認證資料存儲（支援localStorage和sessionStorage）

## 技術棧

- **後端語言**：Python 3.8+
- **Web框架**：FastAPI
- **AI模型**：OpenAI GPT-4o-mini, LangChain
- **資料庫**：MariaDB（關聯式）、FAISS（向量資料庫）
- **消息平台**：LINE Messaging API
- **前端**：HTML5、CSS3、JavaScript、jQuery、Bootstrap 5
- **部署工具**：Uvicorn、Nginx、Bash 啟動腳本
- **跨設備支援**：ngrok（支援透過手機直接管理，不限於內網訪問）

## 檔案架構及說明

### 核心服務

- **main.py**  
  *啟動 FastAPI 應用，配置路由並整合同步LINE消息與AI處理。*

- **line_handler.py**  
  *負責解析並處理來自 LINE 的各類消息事件，並調用 AI代理生成回覆。*

- **agent.py**  
  *封裝了 AI代理邏輯，利用 LangChain 與 OpenAI 模型進行對話生成。*

### 資料庫相關

- **mariadb_storage.py**  
  *負責與 MariaDB 進行連接、執行 CRUD 操作，並初始化用戶、消息和產品表格。*

- **init_db.py**  
  *執行資料庫初始化，創建表結構並設置初始資料。*

- **import_manager.py**  
  *批量導入產品數據，並在資料庫導入後觸發向量資料庫更新。*

### AI與向量資料庫

- **tools.py**  
  *提供工具函數給 AI代理使用，進行向量資料庫查詢等操作。*

- **vector_db_manager.py**  
  *負責從產品數據生成向量表示，建立與更新 FAISS 向量資料庫；並使用 LLM 進行智能產品分類。*

- **create_improved_vectordb.py**  
  *從 JSON 或資料庫讀取產品數據並生成向量資料庫。*

- **reload_vector_db.py**  
  *提供重新載入向量資料庫的功能，讓系統無需重啟即可更新向量資訊。*

### 管理後台

- **admin_api.py**  
  *提供 RESTful API 接口供管理前端呼叫，包含用戶、消息和產品管理功能以及身份驗證。*

- **static/admin/**  
  *包含各種前端資源:*
  - **index.html** - 管理介面主頁面，自動檢測登入狀態
  - **login.html** - 新增的登入頁面，支援記住我功能
  - **mobile_adapter.js** - 行動裝置適配器，提供跨設備支援
  - **api_test.html** - API診斷工具，協助排解連接問題
  - **users.js, products.js** - 用戶和產品管理邏輯

### 工具與輔助腳本

- **start_service.sh**  
  *檢查依賴、初始化資料庫及向量資料庫，並啟動主服務與管理後台。*

- **.env.example**  
  *提供範例環境變數設定，需複製為 .env 並更新相關金鑰與配置。*

- **requirements.txt**  
  *包含所有 Python 相依套件清單。*

## 檔案關聯與數據流程

1. **用戶對話流程**  
   用戶 → LINE 平台 → main.py → line_handler.py  
   └── 呼叫 agent.py 處理對話並訪問 tools.py 及 vector_db_manager.py → 向量檢索 → 回覆生成  
   └── 同時透過 mariadb_storage.py 記錄用戶訊息與對話

2. **向量資料庫更新流程**  
   create_improved_vectordb.py 生成初始向量數據  
   └── admin_api.py 呼叫 vector_db_manager.py 更新向量庫  
   └── 如需熱重載則觸發 reload_vector_db.py 重新載入內存中的向量資訊
   └── LLM 自動分類產品到適當類別

3. **產品數據導入流程**  
   import_manager.py 從 JSON 或其他來源導入產品數據  
   └── mariaDB 更新後，vector_db_manager.py 快速更新向量資料庫  
   └── LLM 自動對新導入的商品進行分類，不需手動干預  
   └── 管理前端可通過 API 觸發向量庫重建或重載

4. **管理後台操作流程**  
   管理員透過 login.html 進行身份驗證 → 憑證保存到 localStorage/sessionStorage  
   └── 登入後自動跳轉到 index.html，憑證自動添加到 API 請求中  
   └── 與 mariadb_storage.py 及 vector_db_manager.py 互動，返回 JSON 給前端頁面顯示  
   └── 行動裝置透過 mobile_adapter.js 自動適配，提供最佳化的操作體驗

## 安裝與設定

### 環境需求
- Python 3.8 或以上
- MariaDB 10.5+
- 有效的 LINE Messaging API 帳號
- OpenAI API 金鑰
- (選用) Node.js 14+ 用於前端開發
- (選用) Ngrok 用於行動裝置外部訪問

### 基本安裝步驟
1. 下載專案並進入目錄  
   `git clone [repository-url] && cd ai_service`
2. 建立虛擬環境並安裝依賴  
   `python3 -m venv venv && source venv/bin/activate && pip install -r requirements.txt`
3. 複製範例環境變數設定  
   `cp .env.example .env`  
   並編輯 .env 文件以填入正確配置
4. 初始化資料庫  
   `python init_db.py`
5. 建立向量資料庫（初次）  
   `python create_improved_vectordb.py`
6. 啟動服務（使用 start_service.sh）  
   `./start_service.sh`

## 使用指南

### LINE 聊天
- 加好友後直接發送詢問訊息，例如「你們有哪些產品？」  
- 支援多輪對話與編號查詢

### 管理後台
- 訪問 `http://your-domain:8080` 或 ngrok 連結以進入登入介面
- 在登入頁面輸入管理員憑證，可選擇"記住我"確保連續操作
- 可查看用戶列表、對話記錄、產品管理與統計數據  
- 產品管理介面簡化，移除了類別選擇步驟，由AI自動分類  
- 支持通過 API 重建或重新載入向量資料庫
- 支持在手機、平板等行動裝置上流暢操作

## 開發者指南

- 程式碼模組化設計，各檔案清楚分工  
- 修改或擴展功能時，請參考各模組的說明與注釋
- 前端檔案中的 mobile_adapter.js 負責處理行動裝置適配
- 提交變更前請執行測試並更新文檔

## 故障排除

- **連接問題**：檢查 .env 設定是否正確，確認 MariaDB 與 API 金鑰有效
- **登入失敗**：使用 api_test.html 診斷工具測試連接狀態
- **向量資料庫異常**：檢查 vectordb 目錄下檔案，如有問題可使用 reload_vector_db.py 重載
- **管理界面錯誤**：檢查 admin_api.py 與前端資源是否正確部署
- **手機訪問問題**：確保 Nginx 配置正確支持 ngrok，並檢查 mobile_adapter.js 是否正常載入

## 客戶說明

本系統是一個基於 LINE 平台的智能客服服務，主要功能包括：
- 快速回答與產品、購物相關的問題
- 根據用戶查詢提供精準的產品資訊
- 通過向量資料庫技術進行語意檢索，輔助篩選及排序相關商品
- 自動化產品分類，減少人工維護成本並提高分類準確度
- 管理後台跨設備支援，可在任何裝置上進行系統管理


