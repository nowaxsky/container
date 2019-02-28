# Docker Network

## 1. FoodTrucks專案

<img src="../resource/foodtrucks.png" alt="foodtrucks" width="80%"/>

* 下面將以FoodTrucks專案為例，後台由Python (Flask)做成，搜尋是使用[Elasticsearch](https://www.elastic.co/products/elasticsearch)(ES)，服務分為兩個部分，說明請參考[Github](https://github.com/nowaxsky/FoodTrucks "FoodTrucks")。請用下列指令下載程式碼：

```
$ git clone https://github.com/prakhar1989/FoodTrucks
$ cd FoodTrucks
$ tree -L 2
.
├── Dockerfile
├── README.md
├── aws-compose.yml
├── docker-compose.yml
├── flask-app
│   ├── app.py
│   ├── package-lock.json
│   ├── package.json
│   ├── requirements.txt
│   ├── static
│   ├── templates
│   └── webpack.config.js
├── setup-aws-ecs.sh
├── setup-docker.sh
├── shot.png
└── utils
    ├── generate_geojson.py
    └── trucks.geojson
```

## 2. Elasticsearch

* 服務拆為兩個部分，所以啟動兩個容器，但從Github上面只能下載到backend的部分，可以透過下列指令搜尋遠端是否有Elasticsearch的專案：
```
$ sudo docker search elasticsearch
```
* 可以發現有官方專案，不過這邊建議還是使用Elastic公司自行維護的[網站](https://www.docker.elastic.co/)來下載：
```
$ sudo docker pull docker.elastic.co/elasticsearch/elasticsearch:6.3.2
``` 
* 下載完成後用下列語法來啟動：
```
$ sudo docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.2
```
* 上述指令中`-e`代表設置環境變量，在一個啟動指令中可以使用多個，
* 啟動完成後查看log：
```
$ sudo docker container logs es
```
* 透過curl指令打API(或是用瀏覽器拜訪localhost:9200)測試完成ES的部分：
```
$ curl 0.0.0.0:9200
{
"name" : "ijJDAOm",
"cluster_name" : "docker-cluster",
"cluster_uuid" : "a_nSV3XmTCqpzYYzb-LhNw",
"version" : {
    "number" : "6.3.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "053779d",
    "build_date" : "2018-07-20T05:20:23.451332Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
},
"tagline" : "You Know, for Search"
}
```

## 3. Flask 

* 建立的Dockerfile如下：
```
# start from base
FROM ubuntu:14.04
MAINTAINER Prakhar Srivastav <prakhar@prakhar.me>

# install system-wide deps for python and node
RUN apt-get -yqq update
RUN apt-get -yqq install python-pip python-dev curl gnupg
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash
RUN apt-get install -yq nodejs

# copy our application code
ADD flask-app /opt/flask-app
WORKDIR /opt/flask-app

# fetch app specific deps
RUN npm install
RUN npm run build
RUN pip install -r requirements.txt

# expose port
EXPOSE 5000

# start app
CMD [ "python", "./app.py" ]
```
* `MAINTAINER`指的是維護者的資訊，`RUN`開頭的指令則是會在建立容器時執行，`ADD`將指定的本機物件添加(複製)到容器中，可以是Dockerfile所在目錄的相對路徑或是一個url，`WORKDIR`是為後續的指令(`CMD`或`RUN`等)指定工作目錄，可以使用多個`WORKDIR`，後續命令如果參數是相對路徑，則會疊加上去，例如下例最後是在路經`/a/b/c`下執行pwd。
```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```
* 建立image(請記得將`nowaxsky`替換為自己Docker hub上的用戶名)：
```
$ sudo docker build -t nowaxsky/foodtrucks-web .
```

* 執行已經建立好的image，`--rm`指令為當容器離開後就立刻清理容器並從檔案系統中移除(If instead you’d like Docker to automatically clean up the container and remove the file system when the container exits, you can add the --rm flag.)：
```
$ sudo docker run -P --rm nowaxsky/foodtrucks-web
Unable to connect to ES. Retying in 5 secs...
Unable to connect to ES. Retying in 5 secs...
Unable to connect to ES. Retying in 5 secs...
Out of retries. Bailing out...
```
* 上述步驟顯示啟動失敗，因為Flask服務啟動時無法取得和Elasticsearch的連線(參考FoodTrucks/flask-app/app.py第8行)，即兩個容器沒有辦法溝通。

## 4. Docker network

* 因為ES對外的接口是`0.0.0.0:9200`，若Flask服務可以連接到這個url就可以成功。請透過下列指令查看docker網路：
```
$ sudo docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
c2c695315b3a        bridge              bridge              local
a875bec5d6fd        host                host                local
ead0e804a67b        none                null                local
```
* 若沒有特別指定，容器預設會在bridge　network上執行，所以ES就是執行在bridge上，請使用下列語法檢查bridge。
```
$ sudo docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "c2c695315b3aaf8fc30530bb3c6b8f6692cedd5cc7579663f0550dfdd21c9a26",
        "Created": "2018-07-28T20:32:39.405687265Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "277451c15ec183dd939e80298ea4bcf55050328a39b04124b387d668e3ed3943": {
                "Name": "es",
                "EndpointID": "5c417a2fc6b13d8ec97b76bbd54aaf3ee2d48f328c3f7279ee335174fbb4d6bb",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```
* 從上面的列表中可以看到容器277451c15ec1出現在containers中，使用的IP是`172.17.0.2`，接著嘗試進入Flask bash並用互動模式來測試，進入後請輸入`curl 172.17.0.2:9200`可以成功取得回傳：
```
$ sudo docker run -it --rm nowaxsky/foodtrucks-web bash
root@35180ccc206a:/opt/flask-app# curl 172.17.0.2:9200
{
"name" : "Jane Foster",
"cluster_name" : "elasticsearch",
"version" : {
    "number" : "2.1.1",
    "build_hash" : "40e2c53a6b6c2972b3d13846e450e66f4375bd71",
    "build_timestamp" : "2015-12-15T13:05:55Z",
    "build_snapshot" : false,
    "lucene_version" : "5.3.1"
},
"tagline" : "You Know, for Search"
}
root@35180ccc206a:/opt/flask-app# exit
```
* 如此就可以讓容器互相通訊，但顯然存在2個問題：
    1. 要如何讓Flask container知道es hostname代表`172.17.0.2`或是其他IP(IP可能會改變)？
    1. bridge network是預設的網路，能夠讓所有的容器拜訪，這樣顯然是不安全的，如何將網路獨立出來？
* 建立自定義network：
```
$ sudo docker network create foodtrucks-net
0815b2a3bb7a6608e850d05553cc0bda98187c4528d94621438f31d97a6fea3c

$ sudo docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
c2c695315b3a        bridge              bridge              local
0815b2a3bb7a        foodtrucks-net      bridge              local
a875bec5d6fd        host                host                local
ead0e804a67b        none                null                local
```
* docker預設不同的網路是不能互通的，就能做到隔離的效果。下面將ES重新啟
動在foodtrucks-net網路中，語法為`--net [network]`。但在這之前請先關閉之前的容器，語法如下：
```
$ sudo docker container stop es
es

$ sudo docker rm es
es

$ sudo docker run -d --name es --net foodtrucks-net -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.2
13d6415f73c8d88bddb1f236f584b63dbaf2c3051f09863a3f1ba219edba3673

$ sudo docker network inspect foodtrucks-net
[
    {
        "Name": "foodtrucks-net",
        "Id": "0815b2a3bb7a6608e850d05553cc0bda98187c4528d94621438f31d97a6fea3c",
        "Created": "2018-07-30T00:01:29.1500984Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "13d6415f73c8d88bddb1f236f584b63dbaf2c3051f09863a3f1ba219edba3673": {
                "Name": "es",
                "EndpointID": "29ba2d33f9713e57eb6b38db41d656e4ee2c53e4a2f7cf636bdca0ec59cd3aa7",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```
* 由上述內容可知ES的容器已經運行在foodtrucks-net網路中，接下來只要再將Flask也運行在foodtrucks-net網路中即可。下面示範bash後透過`curl es:9200`指令成功呼叫到API，這是因為docker會自動解析容器名稱為IP位址，這稱為 ***automatic service discovery*** 。最後在指令中直接啟動服務。
```
$ sudo docker run -it --rm --net foodtrucks-net nowaxsky/foodtrucks-web bash
root@9d2722cf282c:/opt/flask-app# curl es:9200
{
"name" : "wWALl9M",
"cluster_name" : "docker-cluster",
"cluster_uuid" : "BA36XuOiRPaghPNBLBHleQ",
"version" : {
    "number" : "6.3.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "053779d",
    "build_date" : "2018-07-20T05:20:23.451332Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
},
"tagline" : "You Know, for Search"
}
root@53af252b771a:/opt/flask-app# ls
app.py  node_modules  package.json  requirements.txt  static  templates  webpack.config.js
root@53af252b771a:/opt/flask-app# python app.py
Index not found...
Loading data in elasticsearch ...
Total trucks loaded:  733
* Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
root@53af252b771a:/opt/flask-app# exit
```
* 上面的方法是進入bash中啟動，接下來我們正式從外部指令啟動Flask服務，並拜訪`0.0.0.0:5000`確定服務啟動成功：
```
$ sudo docker run -d --net foodtrucks-net -p 5000:5000 --name foodtrucks-web nowaxsky/foodtrucks-web
852fc74de2954bb72471b858dce64d764181dca0cf7693fed201d76da33df794

$ sudo docker container ls
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED              STATUS              PORTS                                            NAMES
852fc74de295        prakhar1989/foodtrucks-web                            "python ./app.py"        About a minute ago   Up About a minute   0.0.0.0:5000->5000/tcp                           foodtrucks-web
13d6415f73c8        docker.elastic.co/elasticsearch/elasticsearch:6.3.2   "/usr/local/bin/dock…"   17 minutes ago       Up 17 minutes       0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   es

$ curl -I 0.0.0.0:5000
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 3697
Server: Werkzeug/0.11.2 Python/2.7.15rc1
Date: Thu, 28 Feb 2019 17:59:45 GMT
```
* 我們可以將上述的步驟製作成bash script，請參考FoodTrucks/setup-docker.sh。
```
#!/bin/bash

# build the flask container
docker build -t prakhar1989/foodtrucks-web .

# create the network
docker network create foodtrucks-net

# start the ES container
docker run -d --name es --net foodtrucks-net -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.2

# start the flask app container
docker run -d --net foodtrucks-net -p 5000:5000 --name foodtrucks-web prakhar1989/foodtrucks-web
```
* 使用指令呼叫bash script：
```
cd FoodTrucks
./setup-docker.sh
```
