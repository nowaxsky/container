# docker

## 1. 什麼是容器

VM和容器雖然都屬於虛擬化的技術，目標都是為了將一套應用程式所需的執行環境打包起來，建立一個孤立環境，方便在不同的硬體中移動，但兩者的運作思維截然不同。簡單來說，常見的傳統虛擬化技術如vSphere或Hyper-V是以作業系統為中心，而Container技術則是一種以應用程式為中心的虛擬化技術。docker就是一種容器的實作。

<img src="./resource/virtualization.png" alt="virtualization" width="80%"/>
<img src="./resource/docker.png" alt="docker" width="80%"/>

兩者最明顯的差別是，虛擬機器需要安裝作業系統(安裝Guest OS)才能執行應用程式，而Container內不需要安裝作業系統就能執行應用程式。Container技術不是在OS外來建立虛擬環境，而是在OS內的核心系統層來打造虛擬執行環境，透過共用Host OS的作法，取代一個一個Guest OS的功用。Container也因此被稱為是OS層的虛擬化技術。

容器和VM的差異如下:

特性|容器|虛擬機
--|--|--
特性|容器|虛擬機
啟動|秒級|分鐘級
硬碟容量|一般為 MB|一般為 GB
效能|接近原生|比較慢
系統支援量|單機支援上千個容器|一般幾十個

## 2. 容器的優勢

* 更快速的交付和部署
* 更有效率的虛擬化
* 更輕鬆的遷移和擴展
* 更簡單的管理

## 3. docker基本概念

1. 映像檔(image): 映像檔就是一個唯讀的模板，可以用來建立docker容器。例如：一個映像檔可以包含一個完整的ubuntu作業系統環境，裡面僅安裝了 Apache 或使用者需要的其它應用程式。
1. 容器(container)：Docker 利用容器來執行應用。容器是從映像檔建立的執行實例。它可以被啟動、開始、停止、刪除。每個容器都是相互隔離的、保證安全的平台。
1. 倉庫(repository)：倉庫是集中存放映像檔檔案的場所。有時候會把倉庫和倉庫註冊伺服器(Registry)混為一談，並不嚴格區分。實際上，倉庫註冊伺服器上往往存放著多個倉庫，每個倉庫中又包含了多個映像檔，每個映像檔有不同的標籤(tag)。
倉庫分為公開倉庫(Public)和私有倉庫(Private)兩種形式。

## 4. 安裝docker(ubuntu)

1. 更新apt套件
    ```
    $ sudo apt-get update
    ```
1. 安裝curl
    ```
    $ sudo apt install curl
    ```
1. 透過curl安裝docker(快速安裝法，生產環境中不推薦使用)
    ```
    $ curl -sSL https://get.docker.com/ | sudo sh
    ```
1. 啟動docker
    ```
    $ sudo service docker start
    ```

## 5. 從遠端取得image

1. 用docker pull從遠端取得image(可以用:後面帶版本)
    ```
    $ sudo docker pull ubuntu:12.04
    ```
1. 查看所有本機的image
    ```
    $ sudo docker image ls
    REPOSITORY       TAG      IMAGE ID      CREATED      VIRTUAL SIZE
    ubuntu           12.04    74fe38d11401  4 weeks ago  209.6 MB
    ubuntu           precise  74fe38d11401  4 weeks ago  209.6 MB
    ubuntu           14.04    99ec81b80c55  4 weeks ago  266 MB
    ubuntu           latest   99ec81b80c55  4 weeks ago  266 MB
    ubuntu           trusty   99ec81b80c55  4 weeks ago  266 MB
    ```
    * 其中映像檔的 ID 唯一標識了映像檔，注意到 ubuntu:14.04 和 ubuntu:trusty 具有相同的映像檔 ID，說明它們實際上是同一映像檔。
    * TAG 用來標記來自同一個倉庫的不同映像檔。例如 ubuntu 倉庫中有多個映像檔，通過 TAG 來區分發行版本，例如 10.04、12.04、12.10、13.04、14.04 等。
1. 查看所有本機的容器
    ```
    $ sudo docker ps
    ```
## 6. 運行docker

1. 用docker run來運行docker，若image不存在會自動從遠端獲取。獲取image之後會自動建立一個容器來運行。
    ```
    $ sudo docker run hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    ca4f61b1923c: Pull complete
    Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
    Status: Downloaded newer image for hello-world:latest

    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    ...
    ```
1. 有些服務運行起來會需要進行互動，可以下參數`-i -t`，如下例：
    ```
    $ sudo docker run -i -t ubuntu:12.04 /bin/bash
    root@fe7fc4bd8fc9:/#
    ```
1. 使用docker run若沒有指定tag，預設使用latest。

## 7. 

