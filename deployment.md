## 環境準備

刪除所有 container
```bash
docker rm -f $(docker ps -a -q)
```

初始化 Docker Swarm
```bash
docker swarm init
```

建立網路
```bash
docker network create --scope=swarm --driver=overlay --attachable jobmarket-swarm-network
```

## 佈署服務

切換到 services 資料夾，所有的服務配置都在這個目錄
```bash
cd /services
pipenv --python ~/.pyenv/versions/3.8.10/bin/python

pipenv install

pipenv install -e .
```

先在編輯器上搜索 NOTE: 關鍵字，替換所有服務版本

佈署 Portainer

```bash
# 拉取 portainer 和 agent 的映像檔
docker pull portainer/portainer-ce:2.0.1

docker pull portainer/agent

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

以上是基礎服務，如果都沒問題，就可以執行 Worker 來接收和執行任務，和執行 Producer 來發送任務給 Worker 了，這兩個服務會依賴爬蟲，所以要先構建爬蟲的容器。

```bash
cd /crawler-v2
pipenv --python ~/.pyenv/versions/3.8.10/bin/python

pipenv install

pipenv install -e .

docker build -f Dockerfile -t your_username/jobmarket-crawler:{version} .

docker push your_username/jobmarket-crawler:{version}
```

佈署 Celery Worker
```bash
cd /services
# 指定 crawler 容器的版本
DOCKER_IMAGE_USERNAME=xxx DOCKER_IMAGE_VERSION=xxx docker stack deploy --with-registry-auth -c compose.worker.yml crawler
```

佈署 Producer 發送任務
```bash
# 指定 crawler 容器的版本
DOCKER_IMAGE_USERNAME=xxx DOCKER_IMAGE_VERSION=xxx docker stack deploy --with-registry-auth -c compose.producer.yml crawler
```

佈署 Airflow

訪問 `http://localhost:8080` 建立 airflow 資料庫

```bash
cd dataflow
pipenv --python ~/.pyenv/versions/3.8.10/bin/python

pipenv install

pipenv install -e .

docker build -f Dockerfile -t your_username/jobmarket-dataflow:{version} .

DOCKER_IMAGE_USERNAME=xxx DOCKER_IMAGE_VERSION=xxx docker stack deploy --with-registry-auth -c docker-compose-airflow.yml airflow
```

## Docker Swarm 指令

加入新的 Worker 節點
```bash
docker swarm join-token worker

# 輸出
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-xxxxxxxxxxxx-xxxxxxxxxxxx 192.168.1.100:2377

Please note that this token is specific to workers and should be kept secret

# 把上面的指令在對應 worker 節點上執行，該節點就會加入 Swarm 集群。
```


離開 Swarm 模式

如果沒離開 Swarm，你的電腦的 Docker 環境會一直在 Swarm 下，會無法直接執行一些與 Swarm 模式衝突的普通 `docker run`, 'docker start` 之類的命令。

```bash
docker swarm leave --force
```