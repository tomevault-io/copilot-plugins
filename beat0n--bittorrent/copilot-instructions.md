## bittorrent

> 本文档详细描述了一个基于 Go (Gin) 和 Vue.js 的前后端分离 BitTorrent 客户端的设计方案。

# BitTorrent 客户端设计方案

本文档详细描述了一个基于 Go (Gin) 和 Vue.js 的前后端分离 BitTorrent 客户端的设计方案。

## 1. 整体架构

应用采用前后端分离的架构：

- **后端 (Go + Gin):** 负责实现 BitTorrent 协议的核心逻辑，并通过 RESTful API 和 WebSocket 提供服务。
- **前端 (Vue.js):** 负责提供用户交互界面，通过 API 与后端通信。

### 1.1. 后端 (Go + Gin)

- **核心逻辑:**
  - 解析 `.torrent` 文件 (Bencode 编码)。
  - 与 Tracker 服务器通信，获取 Peer 列表。
  - 实现与 Peer 的 TCP 连接、握手和消息交换 (如 `choke`, `interested`, `have`, `piece` 等)。
  - 管理文件下载、分片 (Piece) 的 SHA-1 校验和磁盘 I/O。
- **API 服务:**
  - 使用 Gin 框架提供 RESTful API 用于任务管理。
  - 使用 WebSocket 提供实时的状态更新，避免前端轮询。

### 1.2. 前端 (Vue.js)

- **用户界面:**
  - 提供一个单页面应用 (SPA) 来管理下载任务。
  - 允许用户上传 `.torrent` 文件。
  - 实时展示任务列表、进度、速度等状态。
  - 提供对任务的开始、暂停、删除等操作。
- **技术实现:**
  - 使用 Vue.js 构建响应式用户界面。
  - 通过 `axios` 或 `fetch` 与 RESTful API 交互。
  - 通过 WebSocket 接收后端的实时状态推送。
  - (推荐) 使用 Pinia 或 Vuex 进行状态管理。

## 2. 开发流程

建议分三步进行开发：

1. **第一阶段：实现核心 BitTorrent 后端逻辑 (纯 Go)**

   - 目标：创建一个功能完备的、可在命令行运行的下载器。
   - 步骤：
     1. 搭建项目结构 (`cmd/`, `internal/`)。
     2. 实现 Bencode 解析模块。
     3. 实现 Tracker 通信模块。
     4. 实现 Peer 通信协议模块 (核心)。
     5. 实现下载任务管理与文件写入模块。
2. **第二阶段：集成 Gin 框架，实现 API 和 WebSocket**

   - 目标：将核心逻辑封装成 Web 服务。
   - 步骤：
     1. 设计并实现用于任务管理的 RESTful API。
     2. 集成 WebSocket，实现状态的实时推送。
3. **第三阶段：开发 Vue 前端界面**

   - 目标：构建一个用户友好的图形界面。
   - 步骤：
     1. 使用 `npm create vue@latest` 初始化项目。
     2. 开发 UI 组件 (`TorrentList.vue`, `UploadModal.vue` 等)。
     3. 创建服务层来封装与后端的 API 和 WebSocket 通信。
     4. 集成状态管理库。

## 3. 接口设计

### 3.1. 前端界面操作

- **主界面:**
  - “添加 Torrent” 按钮。
  - 任务列表，每项包含：
    - 文件名、大小、进度条。
    - 实时下载/上传速度。
    - Peer 数量。
    - 任务状态 (下载中, 暂停, 完成, 错误)。
    - 操作按钮 (开始/恢复, 暂停, 删除)。
- **添加任务流程:**
  - 点击按钮，弹出文件上传框。
  - 用户选择 `.torrent` 文件后，通过 API 上传至后端。
  - 任务出现在列表中。

### 3.2. 后端 API

#### RESTful API

- `POST /api/torrents`

  - **功能:** 上传 `.torrent` 文件，创建新下载任务。
  - **请求:** `multipart/form-data`。
  - **响应:** `201 Created`，返回任务唯一 ID (`infohash`) 和基本信息。
- `GET /api/torrents`

  - **功能:** 获取所有任务的列表及其静态信息。
  - **响应:** `200 OK`，返回任务的 JSON 数组。
- `POST /api/torrents/:infohash/start`

  - **功能:** 开始或恢复一个指定的下载任务。
  - **响应:** `202 Accepted`。
- `POST /api/torrents/:infohash/pause`

  - **功能:** 暂停一个指定的下载任务。
  - **响应:** `202 Accepted`。
- `DELETE /api/torrents/:infohash`

  - **功能:** 停止并删除一个任务。可选 `?delete_data=true` 参数用于删除已下载文件。
  - **响应:** `204 No Content`。

#### WebSocket API

- `GET /ws/status`
  - **功能:** 建立 WebSocket 连接以接收实时状态更新。
  - **协议流程:**
    1. 客户端成功连接后，后端**立即**推送一次所有任务的完整状态。
    2. 此后，每当任何任务的状态发生变化，后端都会主动推送更新。
    3. 如果客户端断开重连，将重复第一步，以确保状态同步。
  - **推送消息格式 (JSON):**
    ```json
    {
      "type": "TORRENT_UPDATE",
      "payload": {
        "infohash": "...",
        "name": "ubuntu-22.04.3-desktop-amd64.iso",
        "progress": 0.65,
        "downloadSpeed": 1250000,
        "uploadSpeed": 50000,
        "peers": 25,
        "status": "downloading"
      }
    }
    ```

## 4. 实施细节

### 阶段 1, 步骤 2: Bencode 编解码器

**文件位置:** `backend/internal/bencode/`

#### 目标 (What)

创建一个 Go 包，用于 Bencode 格式数据与 Go 数据结构之间的相互转换。

1.  **解码 (Unmarshal):** 将 Bencode 编码的数据流解码为 Go 的数据结构。这是解析 `.torrent` 文件的第一步。
2.  **编码 (Marshal):** 将 Go 数据结构编码为 Bencode 格式的字节流。这对于后续步骤至关重要，例如，我们需要将 `.torrent` 文件中的 `info` 字典重新编码，以便计算其 SHA-1 哈希值 (Info Hash)，这是与 Tracker 和 Peer 通信所必需的。

编解码器需要支持 Bencode 的全部四种数据类型：

1.  **Strings:** 格式为 `<length>:<content>`。
2.  **Integers:** 格式为 `i<number>e`。
3.  **Lists:** 格式为 `l<elements>e`。
4.  **Dictionaries:** 格式为 `d<key-value pairs>e`，其中 Key 必须是字符串且按字典序排序。

#### 实现逻辑 (How)

##### 解码 (Unmarshal)

我们将采用**递归下降**的解析策略。创建一个主函数 `Unmarshal(data []byte) (interface{}, error)`，它接收原始字节切片作为输入，并使用 `bufio.Reader` 来逐字节解析。

主解析函数 `decode` 通过检查当前位置的第一个字节来判断即将解析的数据类型，并调用相应的辅助函数 (`decodeString`, `decodeInt`, `decodeList`, `decodeDict`)。这种递归结构能够优雅地处理任意嵌套的数据。

##### 编码 (Marshal)

编码的实现逻辑与解码类似，也是递归的。创建一个主函数 `Marshal(v interface{}) ([]byte, error)`，它接收一个 Go 值作为输入。

该函数内部会调用一个递归的 `encode` 辅助函数，该函数使用 `bytes.Buffer` 来高效地构建最终的字节切片。`encode` 函数通过 `switch` 语句判断传入值的类型，并调用相应的辅助函数进行处理：

-   **string:** 写入 `<length>:<content>`。
-   **int/int64:** 写入 `i<number>e`。
-   **[]interface{}:** 写入 `'l'`，然后递归编码列表中的每一个元素，最后写入 `'e'`。
-   **map[string]interface{}:** 写入 `'d'`。**关键点：** 根据 Bencode 规范，字典的 Key 必须在编码前按**字典序**进行排序。我们会先提取所有的 Key，对它们进行排序，然后按排序后的顺序依次编码 Key 和对应的 Value。最后写入 `'e'`。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Beat0n)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Beat0n)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
