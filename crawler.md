# 爬蟲執行指南

本文檔提供了在不同環境中設置和執行爬蟲的說明。

## 先決條件

在開始之前，請確保你已完成以下操作：

- 根據 `how-to-manage-python-virtualenv.md` 文件中的說明，建立並啟動你的 Python 虛擬環境。
- 確保 RabbitMQ 和 MySQL 等所有必要的服務都已啟動並正在運行。

## 本地開發

在本地環境中，你可以直接執行爬蟲以進行快速測試和開發，從而無需 Docker 即可避免重複構建。

### 1. 產生環境變數檔案

首先，你需要產生 `.env` 環境變數檔案：

```bash
ENV=DOCKER python3 genenv.py
```

### 2. 執行爬蟲

你可以直接執行每個爬蟲的 `main.py` 腳本：

```bash
# 執行 CakeResume 爬蟲
pipenv run python -m crawlers.cake.main

# 執行 104 爬蟲
pipenv run python -m crawlers.104.main

# 執行 1111 爬蟲
pipenv run python -m crawlers.1111.main

# 執行 Yourator 爬蟲
pipenv run python -m crawlers.yourator.main
```

## 分散式爬蟲

對於分散式環境，你需要先部署好基礎服務。

### 1. 啟動 Producer

啟動 producer 來爬取資料。以 CakeResume 為例：

```bash
pipenv run python -m crawlers.cake.producer
```

### 2. 驗證執行結果

你可以透過以下方式來確認執行結果：

1.  **檢查資料庫:** 訪問 `http://localhost:8080`，確認資料是否已成功寫入。
2.  **查看 Worker 日誌:** 檢查 worker 的日誌以確認沒有錯誤發生。

    ```bash
    docker logs jobmarket-worker-1
    ```
3.  **從 Portainer 查看日誌:** 訪問 http://127.0.0.1:9000/#!/2/docker/services，檢查 worker 日誌。