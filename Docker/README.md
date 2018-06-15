# Docker 語法

建立image

> docker build -t pushproxy:1.0 .

查看image

> docker images

建立container

> docker container create pushproxy:2.1

執行服務

> docker container run --rm -d -p 800:80 pushproxy:2.1

背景執行 -d 

設定對外port -p {外部}:{內部}


查看container\(已建立\)

> docker container ls -a

移除container

> docker container rm 3b57d04fa1aa

移除image

> docker image rm e6a909e4d294

匯出image

> docker save -o pushproxyV21.tar pushproxy:2.1

寫入image

> docker load --input pushproxyV21.tar

查網路狀態

> docker container inspect {CONTAINER ID}