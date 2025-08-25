# 環境設置與服務部署指南

本文檔將引導您完成設置本地開發環境、部署基礎服務以及在 Docker Swarm 上部署分散式爬蟲的完整流程。

## 1. 環境準備

在開始部署任何服務之前，您需要先準備好您的 Docker 環境。

### 1.1. 清理現有的 Docker 容器 (可選)

如果您想從一個乾淨的環境開始，可以刪除所有現有的 Docker 容器。

```bash
docker rm -f $(docker ps -a -q)
```

### 1.2. 初始化 Docker Swarm

我們將使用 Docker Swarm 來管理我們的容器化服務。

```bash
docker swarm init
```

### 1.3. 建立網路

建立一個可用於所有服務的 overlay 網路。

```bash
docker network create --scope=swarm --driver=overlay --attachable jobmarket-swarm-network
```

## 2. 部署基礎服務

現在，我們將部署應用程式所需的核心服務。

**重要提示:** 在執行以下指令之前，請切換到存放所有服務配置的 `services` 資料夾。

```bash
cd services
```

### 2.1. 建立 Python 虛擬環境

請參考 [如何管理 Python 虛擬環境](how-to-manage-python-virtualenv.md) 的說明來建立和啟用虛擬環境。

### 2.2. 安裝依賴

安裝所有必要的 Python 依賴。

```bash
# 同步依賴
pipenv sync

# 或者，手動安裝
pipenv install
pipenv install -e .
```

### 2.3. 平台特定說明

在您的編輯器中搜索 `NOTE:` 關鍵字。根據您的作業系統 (Windows 或 macOS)，使用對應的服務和指令。

### 2.4. 部署 Portainer (服務監控)

Portainer 提供了一個視覺化的界面來監控和管理您的 Docker 環境。

```bash
# 拉取 Portainer 映像檔
docker pull portainer/portainer-ce:2.0.1
docker pull portainer/agent

# 部署 Portainer
docker stack deploy --with-registry-auth -c portainer.yml por
```

部署完成後，您可以訪問 `http://localhost:9000` 來查看 Portainer 的儀表板。

### 2.5. 部署 MySQL + phpMyAdmin

部署 MySQL 資料庫以及用於管理的 phpMyAdmin。

**注意:** 請將 `DOCKER_IMAGE_USERNAME` 和 `DOCKER_IMAGE_VERSION` 替換為您的實際值。

```bash
DOCKER_IMAGE_USERNAME=your_username DOCKER_IMAGE_VERSION=latest docker stack deploy --with-registry-auth -c compose.mysql.yml mysql
```

### 2.6. 部署 RabbitMQ + Flower

部署 RabbitMQ 消息隊列以及用於監控的 Flower。

```bash
docker stack deploy --with-registry-auth -c compose.rabbitmq.yml rabbitmq
```

當所有基礎服務都成功部署後，您就可以開始部署分散式爬蟲了。

## 3. 部署分散式爬蟲

### 3.1. 構建爬蟲 Docker Image

首先，切換到 `crawler-v2` 目錄並構建爬蟲的 Docker 映像檔。

```bash
cd crawler-v2
```

**a. 建立 Python 虛擬環境與安裝依賴**

與 `services` 目錄中的流程相同。

**b. 構建並推送 Docker Image**

**注意:** 請將 `your_username` 和 `{version}` 替換為您的 Docker Hub 用戶名和映像檔版本。

```bash
# 構建映像檔
docker build -f Dockerfile -t your_username/jobmarket-crawler:{version} .

# 推送映像檔到 Docker Hub
docker push your_username/jobmarket-crawler:{version}
```

### 3.2. 部署 Worker

返回 `services` 目錄，並部署爬蟲的 Worker。

```bash
cd ../services

# 部署 Worker
# 注意: 請指定您剛剛建立的爬蟲映像檔版本
DOCKER_IMAGE_USERNAME=your_username DOCKER_IMAGE_VERSION={version} docker stack deploy --with-registry-auth -c compose.worker.yml crawler
```

### 3.3. 部署 Producer (發送任務)

最後，部署 Producer 來發送爬取任務。

```bash
# 部署 Producer
# 注意: 請指定您剛剛建立的爬蟲映像檔版本
DOCKER_IMAGE_USERNAME=your_username DOCKER_IMAGE_VERSION={version} docker stack deploy --with-registry-auth -c compose.producer.yml crawler
```

## 4. 部署 Airflow (可選)

如果您需要使用 Airflow 來進行數據流程管理，請按照以下步驟操作。

### 4.1. 構建 Airflow Docker Image

**a. 準備環境**

```bash
cd ../dataflow
# 建立虛擬環境並安裝依賴
pipenv sync
```

**b. 構建並推送 Docker Image**

根據您的平台選擇對應的指令。

```bash
# 標準環境
docker build -f Dockerfile -t your_username/jobmarket-dataflow:{version} .
docker push your_username/jobmarket-dataflow:{version}

# macOS M1
docker build -f Dockerfile -t your_username/jobmarket-dataflow:{version}.arm64 .
docker push your_username/jobmarket-dataflow:{version}.arm64

# Google Compute Engine (GCE)
docker build -f gce.with.env.Dockerfile -t your_username/jobmarket-dataflow:{version}.gce .
docker push your_username/jobmarket-dataflow:{version}.gce
```

### 4.2. 部署 Airflow

```bash
# 注意: 請替換為您的用戶名和版本
DOCKER_IMAGE_USERNAME=your_username DOCKER_IMAGE_VERSION={version} docker stack deploy --with-registry-auth -c docker-compose-airflow.yml airflow
```

部署後，訪問 `http://localhost:8080` 來初始化 Airflow 資料庫。

## 5. Google Cloud Platform (GCP) 部署

此部分為針對 GCP 環境的特定指令。

### 5.1. 設定 gcloud 專案

```bash
gcloud config set project airflow-466005
```

### 5.2. 上傳 DAGs 到 Composer

```bash
gcloud composer \
    environments storage \
    dags import --environment airflow  \
	    --location us-central1 \
	    --source "src/dataflow"
```