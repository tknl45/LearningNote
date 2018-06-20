# Docker Swarm 

## 目的即管理多台節點的 Docker 上應用程式與節點資源的排程等，並提供標準的 Docker API 介面當作前端存取入口。

Docker Swarm 具備基本叢集功能，能讓多個 Docker 組合成一個網路群組，來提供容器服務。讓多台Docker Host所建立出來container可以在網路上認識。

Docker Swarm 一般分為兩個角色Manager與Worker，兩者主要工作如下：

Manager: 主要負責排程 Task，Task 可以表示為 Swarm 節點中的 Node 上啟動的容器。同時還負責編配容器與叢集管理功能，簡單說就是 Manager 具備管理 Node 的工作，除了以上外，Manager 還會維護叢集狀態。另外 Manager 也具備 Worker 的功能，當然也可以設定只做管理 Node 的職務。
Worker: Worker 主要接收來自 Manager 的 Task 指派，並依據指派內容啟動 Docker 容器服務，並在完成後向 Manager 匯報 Task 執行狀態。


ref:https://kairen.github.io/2016/11/16/container/docker-swarm/

Manager 節點建置
# 建立3個節點，1個manager，2個worker
docker-machine create swarn-manger
docker-machine create swarn-worker1
docker-machine create swarn-worker2


# 查看節點，1個manager，2個worker
docker-machine ls
NAME ACTIVE DRIVER STATE URL SWARM DOCKER ERRORS
swarn-manger - virtualbox Running tcp://192.168.99.100:2376 v18.05.0-ce
swarn-worker1 - virtualbox Running tcp://192.168.99.101:2376 v18.05.0-ce
swarn-worker2 - virtualbox Running tcp://192.168.99.102:2376 v18.05.0-ce

# 進入manager做節點管理
docker-machine ssh swarn-manger

# 啟動swarm 設定到swarn-manger的IP
docker swarm init --advertise-addr 192.168.99.100

## 指令執行後會回傳一串cmd 給其他worker node去執行
## ex.
docker swarm join --token SWMTKN-1-0q0ohnexs40lb9z4kmvqb6zcrmp22hul9tmh6zpfztxzv5cv61-73yubitun1ufm0yhwx7h38p85 192.168.99.100:2377

Worker 節點建置

# 進入worker做節點管理
docker-machine ssh swarn-worker1
docker swarm join --token SWMTKN-1-0q0ohnexs40lb9z4kmvqb6zcrmp22hul9tmh6zpfztxzv5cv61-73yubitun1ufm0yhwx7h38p85 192.168.99.100:2377


範例1:透過指令建立nginx服務
# 進入manager節點
docker-machine ssh swarn-manger

# 拉下nginx 當作image
docker pull nginx

# 建立nginx服務 指定name為 web
docker service create --name web -p 80:80 nginx

# 列出服務
docker service ls
ID NAME MODE REPLICAS IMAGE PORTS
r57e7i6ku7pa web replicated 1/1 nginx:latest *:80->80/tcp

指令會列出服務有幾份，網路情況

# 查看服務
docker service ps web
ID NAME IMAGE NODE DESIRED STATE CURRENT STATE ERROR PORTS
qicsci6uyzha web.1 nginx:latest swarn-worker1 Running Running 32 seconds ago

指令顯示該服務建立在worker1節點上

# 測試服務是否啟動（＠manager）
在manager節點，連到127.0.0.1可以正確讀取到worker1上建立的服務。證明cluster成功。
docker@swarn-manger:~$ curl 127.0.0.1
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
body {
width: 35em;
margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif;
}
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


# 新建立nginx服務 指定name為 web2，並指定對外port為81
docker service create --name web2 -p 81:80 nginx


# 列出服務
docker service ls
ID NAME MODE REPLICAS IMAGE PORTS
r57e7i6ku7pa web replicated 1/1 nginx:latest *:80->80/tcp
hfdaisi2hqy1 web2 replicated 1/1 nginx:latest *:81->80/tcp

# 查看服務
新建立的服務建立在work1上
docker@swarn-manger:~$ docker service ps web2
ID NAME IMAGE NODE DESIRED STATE CURRENT STATE ERROR PORTS
0tyf4jz7whcb web2.1 nginx:latest swarn-manger Running Running about an hour ago
docker@swarn-manger:~$ docker service ps web
ID NAME IMAGE NODE DESIRED STATE CURRENT STATE ERROR PORTS
qicsci6uyzha web.1 nginx:latest swarn-worker1 Running Running about an hour ago

# 停止服務
docker service rm web
docker service rm web2


範例2:透過建立叢集服務

# 進入manager節點
docker-machine ssh swarn-manger

# 拉下whoami 當作image
Whoami 這服務會顯示你在哪一台機器上
docker pull jwilder/whoami

# 拉下visualizer 當作image
visualizer是一個視覺化顯示swarm cluster啟用哪個服務，又部署在哪一台的服務
docker pull dockersample/visualizer

# 開啟服務
docker run -it -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock dockersamples/visualizer

# 建立服務 名稱為helloworld
docker service create --name helloworld -p 80:8000/tcp jwilder/whoami

# 列出服務
docker service ls
ID NAME MODE REPLICAS IMAGE PORTS
70vebscyp41k helloworld replicated 1/1 jwilder/whoami:latest *:80->8000/tcp

# 查看服務
docker service ps helloworld
ID NAME IMAGE NODE DESIRED STATE CURRENT STATE ERROR PORTS
os7zv68213at helloworld.1 jwilder/whoami:latest swarn-worker1 Running Running 5 minutes ago

# 擴展服務為3台 [SERVICE=REPLICAS...]
docker service scale helloworld=2
helloworld scaled to 2
overall progress: 2 out of 2 tasks
1/2: running [==================================================>]
2/2: running [==================================================>]
verify: Service converged

# 列出服務
docker service ls
ID NAME MODE REPLICAS IMAGE PORTS
70vebscyp41k helloworld replicated 2/2 jwilder/whoami:latest *:80->8000/tcp


# 視覺化觀看部署情況
http://192.168.99.100:8080/

# 測試服務
每次回傳的編碼都不同
docker@swarn-manger:~$ curl 127.0.0.1
I'm b11c666ab873
docker@swarn-manger:~$ curl 127.0.0.1
I'm 30f466106f98


# 停止服務
docker service rm helloworld