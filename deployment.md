## 環境準備

刪除所有 container
```bash
docker rm -f $(docker ps -a -q)
```

初始化 Docker Swarm
```bash
docker swarm init
```

加入新的 Worker 節點
```bash
docker swarm join-token worker

# 輸出
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-xxxxxxxxxxxx-xxxxxxxxxxxx 192.168.1.100:2377

Please note that this token is specific to workers and should be kept secret

# 把上面的指令在對應 worker 節點上執行，該節點就會加入 Swarm 集群。
```

建立網路
```bash
docker network create --scope=swarm --driver=overlay jobmarket-swarm-network
```

拉取 portainer 和 agent 的映像檔
```bash
docker pull portainer/portainer-ce:2.0.1
docker pull portainer/agent
```

離開 Swarm
如果沒離開 Swarm，你的電腦的 Docker 環境會一直在 Swarm 下，會無法直接執行一些與 Swarm 模式衝突的普通 `docker run`, 'docker start` 之類的命令。
```bash
docker swarm leave --force
```

構建爬蟲容器
```bash
docker build -f Dockerfile -t your_username/jobmarket-crawler:0.0.1 .

docker push your_username/jobmarket-crawler:0.0.1
```

## 佈署服務

所有的服務配置都在 services 倉庫下
```bash
cd /services
```

佈署 Portainer
```bash
docker stack deploy --with-registry-auth -c portainer.yml por
```

佈署 MySQL + PhpMyAdmin
```bash
docker stack deploy --with-registry-auth -c compose.mysql.yml mysql
```

佈署 RabbitMQ + Flower
```bash
docker stack deploy --with-registry-auth -c compose.rabbitmq.yml rabbitmq
```

佈署 Celery Worker
```bash
DOCKER_IMAGE_USERNAME=xxx DOCKER_IMAGE_VERSION:xxx docker stack deploy --with-registry-auth -c compose.worker.yml crawler
```

佈署 Producer 發送任務
```bash
DOCKER_IMAGE_USERNAME=xxx DOCKER_IMAGE_VERSION:xxx docker stack deploy --with-registry-auth -c compose.producer.yml crawler
```
