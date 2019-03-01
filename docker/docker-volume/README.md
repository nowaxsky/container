# Docker Volume

<img src="../resource/types-of-mounts-volume.png" alt="types-of-mounts-volume" width="80%"/>

有兩種方式可以管理容器中的資料：
* Data volumes
* Data volume containers

## 1. 掛載 data volume
* __Data Volume features__
    * Data volume是在容器的特殊目錄內。
    * 當容器被初始化的時候建立，且當容器被停止時預設不會被刪除，即便沒有任何的容器參考到volume時，GC也不會刪除。
    * Data volume是獨立的，同時也可以給多個容器共享，可以透過唯獨模式掛載(mounted)。
* 透過指令`-v`可以在建立容器時直接將volume掛載到容器內，下面示範掛載到容器內的/data位置，透過互動模式可以確定資料夾的存在。接下來在/data中建立檔案file1.txt後離開。
    ```
    $ sudo docker run -it -v /data --name container1 busybox
    / # ls
    bin   data  dev   etc   home  proc  root  sys   tmp   usr   var
    / # cd data
    /data # touch file1.txt
    /data # ls
    file1.txt
    /data # exit
    ```
* 檢查container1容器的掛載資訊，可以看到docker自動建立/var/lib/docker/volumes/...給container1內的/data目錄使用，`"RW": true`意思為可以做讀寫(Read and Write)。
    ```
    $ sudo docker inspect container1
    ...
        "Mounts": [
            {
                "Type": "volume",
                "Name": "29fb846e9275258554d9ad0c06375ca24ad235726d398e8eb8a2dfe7fd52db86",
                "Source": "/var/lib/docker/volumes/29fb846e9275258554d9ad0c06375ca24ad235726d398e8eb8a2dfe7fd52db86/_data",
                "Destination": "/data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
    ...
    ```
* 重新啟動服務並進入容器，可以確認剛才建立的目錄依然存在：
    ```
    $ sudo docker restart container1
    $ sudo docker attach container1
    / # ls
    bin   data  dev   etc   home  proc  root  sys   tmp   usr   var
    / # cd data
    /data # ls
    file1.txt
    /data # exit
    ```
* 接下來測試如果將容器移除後，volume是否依然存在：
    ```
    $ sudo docker rm container1
    container1
    $ sudo docker volume ls
    DRIVER              VOLUME NAME
    local               29fb846e9275258554d9ad0c06375ca24ad235726d398e8eb8a2dfe7fd52db86
    ``` 
* 注意到volume並未被移除(可以比對`"Source": "/var/lib/docker/volumes/29fb846...`確認是剛才提供給container1的volume)。所以在移除容器時可以加入參數`-v`連同volume一併移除，如下：
    ```
    $ sudo docker rm -v container1
    ```
* 由於現在container1已經被刪除，故輸入上述指令將會報錯，也可以透過下列指令直接刪除volume：
    ```
    $ sudo docker volume rm 29fb846e9275258554d9ad0c06375ca24ad235726d398e8eb8a2dfe7fd52db86
    29fb846e9275258554d9ad0c06375ca24ad235726d398e8eb8a2dfe7fd52db86
    ```

## 2. 掛載本機端目錄到容器

* 本節將示範如何將本機端已存在的目錄掛載到容器中(Mount an existing host folder in the Docker container)，語法是在啟動容器時，下參數`-v [host folder]:[container volume name]`。下面範例是將本機的`~/container1`目錄掛載到容器的datavol中，若本機端不存在目錄則會自動建立。
    ```
    $ sudo docker run -it --name container1 -v ~/container1:/datavol / 
    # ls
    bin      datavol  dev      etc      home     proc     root     sys      tmp      usr      var
    / # exit
    $ cd ~
    $ ls
    container1  c.txt  docker-curriculum  examples.desktop  FoodTrucks  sinatra ...
    ```
* 接下來進入本機端的container1目錄，並建立test.txt，然後再次進入容器檢查是否會自動新增這個檔案。
    ```
    $ cd container1 
    $ sudo touch test.txt
    test.txt
    sudo docker restart container1
    container1
    $ sudo docker attach container1
    / # ls
    bin      datavol  dev      etc      home     proc     root     sys      tmp      usr      var
    / # cd datavol/
    /datavol # ls
    test.txt
    /datavol # exit
    ```
* 由上面範例可以確定本機端的資料可以同步到容器內，容器內的變化其實也會同步到本機端，下例示範在容器內新增test2.txt後，檢查本機端是否存在該檔案。
    ```
    sudo docker restart container1
    container1
    $ sudo docker attach container1
    / # ls
    bin      datavol  dev      etc      home     proc     root     sys      tmp      usr      var
    / # cd datavol/
    /datavol # ls
    test.txt
    /datavol # touch test2.txt
    /datavol # ls
    test.txt   test2.txt
    /datavol # exit
    $ ls
    test2.txt  test.txt
    ```
* 在啟動容器時可以同時掛載多個目錄到容器中。另外也可以掛載檔案到容器中，但部分編輯工具會導致錯誤，所以建議直接掛載目錄較佳。

## 3. Data volume containers

* 適用在有一些持續更新的資料需要在容器之間共享，最好建立data volume container。主要分為兩個步驟：
    1. 建立Data volume container。
    1. 建立另外一個容器並掛載步驟1建立的Data volume container。
* 下面先刪除之前建立的容器，重新建立容器後在/data中建立兩個檔案file1.txt和 file2.txt。
    ```
    sudo docker rm -v container1
    container1
    $ sudo docker run -it -v /data --name container1 busybox
    / # cd data
    /data # ls
    /data # touch file1.txt
    /data # touch file2.txt
    /data # ls
    file1.txt  file2.txt
    ```
* 接下來啟動另外一個terminal，並下指令`sudo docker ps`檢查container1服務存在，接著透過下列指令檢查剛才建立的檔案是否存在：
    ```
    $ sudo docker exec container1 ls /data
    file1.txt
    file2.txt
    ```
* 接著啟動第二個容器命名為container2，並用語法`--volumes-from`指定container1的volume給container2當容器使用：
    ```
    $ sudo docker run -it --volumes-from container1 --name container2 busybox
    / # ls
    bin   data  dev   etc   home  proc  root  sys   tmp   usr   var
    / # cd data/
    /data # ls
    file1.txt  file2.txt
    /data # 
    ```
* 從上面的測試中可以確定container2的容器和container1是使用同一個。

## 4. Mount

* 基本上所有`-v`都可以替換為`--mount`來使用，在新版的docker中官方建議使用`--mount`，因為操作上可以更詳細且語意更清楚。(In general, `--mount` is more explicit and verbose. The biggest difference is that the `-v` syntax combines all the options together in one field, while the `--mount` syntax separates them. When using volumes with services, only `--mount` is supported.)
* 下列兩個用法等效：
    ```
    $ docker run -d --name devtest --mount source=myvol2,target=/app nginx:latest
    $ docker run -d --name devtest -v myvol2:/app nginx:latest
    ```
* 下列兩個用法等效：
    ```
    $ docker run -d --name=nginxtest --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly nginx:latest
    $ docker run -d --name=nginxtest -v nginx-vol:/usr/share/nginx/html:ro nginx:latest
    ```