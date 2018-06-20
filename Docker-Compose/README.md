#
docker compose

docker compose，通過docker compose我們可以非常方便的部署複雜的application。即通過在docker-compose.yml文件中定義我們application各個組成部分——service，然後一鍵build，up。

建立docker-compose.yml

> 版本不同要注意，但其實差不多主要是支援的關鍵字多寡，版本3可以用在swarm佈署上

版本2
```
version: "2"

services:
cmm_push:
build: .
ports: ["80"]
```




版本3
```
version: '3'

services:
cmm_push:
image: cmm_push
ports: ["80"]
build:
context: .
dockerfile: Dockerfile
```

佈署

> docker-compose build


前景執行

> docker-compose up

背景執行

> docker-compose up -d


