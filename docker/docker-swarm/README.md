# Docker Swarm

## 1. Container Orchestration System

想像今天有上百個容器正在執行，我們需要確認每個容器的狀態和管理每個容器是非常複雜的，下列幾項事情是我們所關心的：

1. 每個容器的健康程度
1. 對於一個特定image啟動了多少的容器
1. 根據流量做scale up或scale down
1. 如何對每個容器進行更新
1. ...

Docker Swarm就是要解決容器管理的問題。

## 1. Docker Machine

* Docker Machine是安裝Docker Engine在VM上的工具，但[VM不支援巢狀VM](https://superuser.com/questions/1138980/this-computer-doesnt-have-vt-x-amd-v-enabled-enabling-it-in-the-bios-is-mandat)，所以無法在一個VM中運行另一個VM，故如果是將Linux安裝VM中則不能在上面安裝建立machine(但依然可以安裝docker-machine)。下面範例是將Docker Machine安裝在Windows環境中。
* 先安裝Docker Machine，請輸入下列指令：
    ```
    base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
    ```
* 接下來在ubuntu安裝virtualbox，建立一個machine稱為default：
    ```
    $ sudo apt-get install virtualbox 
    $ sudo docker-machine create --driver virtualbox default
    ```
* 下列語法可以查看所有的machine：
    ```
    $ docker-machine ls
    NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER   ERRORS
    default   *        virtualbox   Running   tcp://192.168.99.187:2376           v1.9.1
    ```