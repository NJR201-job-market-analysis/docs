## 前言: 為什麼要 docker swarm

### 傳統手動用 Docker Compose 部署的問題

實務上，會有很多台主機或虛擬機，如果沒有 Docker Swarm，你需要手動遠端登入每一台主機（SSH、RDP 等），在每台主機上執行 docker-compose up -d 等指令，多台機器間沒集中管理，不容易保持設定一致，若要更新服務，得一台一台操作，容易出錯或版本不一致，缺乏自動調度與容錯能力，例如節點掛了沒自動遷移服務

### 使用 Docker Swarm 的優勢

- 集中管理: 在任一 manager 節點執行 docker stack deploy -c docker-compose.yml mystack 即可部署整個叢集
- 自動調度: Swarm 根據你的 deploy 配置，自動選擇符合條件的節點部署容器
- 高可用性: 若某台節點掛了，Swarm 會自動在其他節點重啟容器
- 滾動更新: 你可以設定滾動更新策略，讓服務平滑升級，不影響使用
- 擴展性: 只要加新節點進入叢集，服務可以自動擴展
- 更細緻的部署控制: 可以用 placement、replicas、resource limits 等在 yaml 裡指定，省去手動分派負擔

## 環境準備

刪除所有 container
```
docker rm -f $(docker ps -a -q)
```

初始化 Docker Swarm
```
docker swarm init
```

建立網路
```
docker network create --scope=swarm --driver=overlay jobmarket-swarm-network
```

拉取 portainer 和 agent 的映像檔
```
docker pull portainer/portainer-ce:2.0.1
docker pull portainer/agent
```

## 佈署服務

所有的服務配置都在 services 倉庫下
```
cd /services
```

佈署 Portainer
```
docker stack deploy -c portainer.yml por
```

佈署 MySQL + PhpMyAdmin
```
docker stack deploy -c services/compose.mysql.yml mysql
```

佈署 RabbitMQ + Flower
```
docker stack deploy -c services/compose.rabbitmq.yml rabbitmq
```