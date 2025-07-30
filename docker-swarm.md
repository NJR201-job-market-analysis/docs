## 為什麼要 docker swarm

### 傳統手動用 Docker Compose 部署的問題

實務上，會有很多台主機或虛擬機，如果沒有 Docker Swarm，你需要手動遠端登入每一台主機（SSH、RDP 等），在每台主機上執行 docker-compose up -d 等指令，多台機器間沒集中管理，不容易保持設定一致，若要更新服務，得一台一台操作，容易出錯或版本不一致，缺乏自動調度與容錯能力，例如節點掛了沒自動遷移服務

### 使用 Docker Swarm 的優勢

- 集中管理: 在任一 manager 節點執行 docker stack deploy -c docker-compose.yml mystack 即可部署整個叢集
- 自動調度: Swarm 根據你的 deploy 配置，自動選擇符合條件的節點部署容器
- 高可用性: 若某台節點掛了，Swarm 會自動在其他節點重啟容器
- 滾動更新: 你可以設定滾動更新策略，讓服務平滑升級，不影響使用
- 擴展性: 只要加新節點進入叢集，服務可以自動擴展
- 更細緻的部署控制: 可以用 placement、replicas、resource limits 等在 yml 裡指定，省去手動分派負擔

## Stack 概念

```bash
docker stack deploy --with-registry-auth -c portainer.yml por
```

yml 後面的 `por`、`mysql`、`rabbitmq`、`crawler` 代表 Stack，　Stack 是 Docker Swarm 中的一個邏輯概念，代表一組相關服務的集合。它類似於 Docker Compose 的 "project"，但專為 Swarm 模式設計。

基本概念：
* Stack = 一組相關的服務 (Services)
* Service = 一個或多個相同容器的實例
* Container = 實際運行的應用程式

一個 Stack 可以包含多個 Services，每個 Service 可以運行一個或多個 Containers，例如：

Stack = 整個資料庫系統，一個包含 MySQL 和 phpMyAdmin 的資料庫系統
Service = 資料庫的各個服務，像是 MySQL 資料庫服務、phpMyAdmin 管理介面服務
Container = 實際運行的資料庫程式，MySQL 服務運行一個 MySQL 資料庫程式、phpMyAdmin 服務運行一個 phpMyAdmin 網頁程式

## 指令一覽

刪除 stack
```bash
docker stack rm airflow api crawler mysql rabbitmq
```

刪除網路
```bash
docker network rm network-name
```

建立網路
```bash
docker network create --scope=swarm --driver=overlay --attachable network-name
```