
在本地端構建和上傳自己的容器(container)時：

1. DOCKER_IMAGE_VERSION 指定構建版本
2. DOCKER_IMAGE_USERNAME 指定自己的帳戶名稱

構建指令
```
DOCKER_IMAGE_VERSION=0.0.0 DOCKER_IMAGE_USERNAME=xxxx docker build -f Dockerfile -t xxx/cake-crawler
```

推送到 Docker Hub
```
docker push your_username/jobmarket-crawler
```

---------------------------------------------------------------------

shared 裡面有寫好的 db 用來將職缺資料寫入資料庫，可參考 crawlers/cake/tasks 的寫法。

---------------------------------------------------------------------

本地開發時，可以先不透過 docker，省去反覆構建的麻煩，不過要確保執行前，相關服務例如 RabbitMQ、MySQL 和 PhpMyAdmin 都已經順利打開。

安裝依賴包
```
pipenv install
pipenv install -e .
```

啟動 worker
```
pipenv run celery -A crawlers.worker worker --loglevel=info --hostname=%h -Q jobmarket-worker-1
```

啟動 producer 爬取資料
```
pipenv run python crawlers.cake.producer
```