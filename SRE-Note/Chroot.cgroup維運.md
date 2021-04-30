# 0429 chroot/cgroup linux維運
curl (砍)
類似爬蟲(可抓網站與檔案)for http
```
$ curl http://www.microsoft.com
```
![](https://i.imgur.com/JDtRAoy.png)

```
$ mkdir rootfs; cd rootfs; curl -s -o alpine.tar.gz http://dl-cdn.alpinelinux.org/alpine/v3.13/releases/x86_64/alpine-minirootfs-3.13.4-x86_64.tar.gz; 
org:分營利機構

```
.tar:打包檔(把很多檔案打包成為一個檔案)
.gz:壓縮(指令是gzip)
所以是先打包成為一個檔案再進行壓縮

linux ps命令一定從/proc裡面讀資料(都叫做process)
diff指令:找出與原檔案不同的設定

## Chroot 
#### hroot，即 change root directory (更改 root 目錄)。在 linux 系統中，系統預設的目錄結構都是以 /，即以根 (root) 開始的。而在使用 chroot 之後，系統的目錄結構將以指定的位置作為 / 位置。
- 為什麼要使用 chroot 命令
    - 增加了系統的安全性，限制了使用者的權力：

        在經過 chroot 之後，在新根下將訪問不到舊系統的根目錄結構和檔案，這樣就增強了系統的安全性。一般會在使用者登入前應用 chroot，把使用者的訪問能力控制在一定的範圍之內。

    - 建立一個與原系統隔離的系統目錄結構，方便使用者的開發：

        使用 chroot 後，系統讀取的是新根下的目錄和檔案，這是一個與原系統根下檔案不相關的目錄結構。在這個新的環境中，可以用來測試軟體的靜態編譯以及一些與系統不相關的獨立開發。

    - 切換系統的根目錄位置，引導 Linux 系統啟動以及急救系統等：

        chroot 的作用就是切換系統的根位置，而這個作用最為明顯的是在系統初始引導磁碟的處理過程中使用，從初始 RAM 磁碟 (initrd) 切換系統的根位置並執行真正的 init，本文的最後一個 demo 會詳細的介紹這種用法。

    - 通過 chroot 執行 busybox 工具

        busybox 包含了豐富的工具，我們可以把這些工具放置在一個目錄下，然後通過 chroot 構造出一個 mini 系統。簡單起見我們直接使用 docker 的 busybox 映象打包的檔案系統。先在當前目錄下建立一個目錄 rootfs：
首先下載一個系統鏡像minifootfs並解壓縮後執行
```
sudo chroot rootfs/ /bin/sh
執行rootfs下的chroot 指令更改原系統的根目錄
rootfs/
```
![](https://i.imgur.com/lbWf0Fj.png)
```
sudo chroot --userspec=bigred:bigred  rootfs/ /bin/sh
這裡的user必須是要原系統/etc/passwd的User或是rootfs下etc/passwd的user
```
![](https://i.imgur.com/jqxbvgG.png)

### 測試2
```
 sudo chroot $J /bin/bash
 sudo chroot rootfs/ /bin/sh
```
- 把bin/bash,bin/ls鎖進Jail根目錄裡面
- chroot 命令 改变其当前目录，并将根目录变为指定目录,然后如果提供了命令则运行命令，也可以运行一个用户的交互式shell的副本（译注：即bash等。）。请注意并不是每一个程序都可以使用 chroot 命令。
Chroot 用途
> 建立開發、測試環境：使用 chroot 建立的環境中測試軟體，避免直接部署到完整的生產環境當中。
修復系統：當一系統故障時，可以透過 chroot 在其他目錄開啟系統，最後再回到故障系統中進行修復。
相依性控制：不同軟體可能產生的相依性可以透過 chroot 建立的不同根目錄環境來進行分隔。
權限分離：將特定使用者允許存取的檔案放入 chroot 的目錄當中，當使用者使用 ssh 登入後限制其所能存取的環境僅限於該 chroot 的目錄下。

## cgroup 用途
可以分配、限制和監控一組任務（進程）使用的物理資源（CPU、內存、磁碟I/O、帶寬或這幾種資源的組合）。  

```
ls /sys/fs/cgroup/
檢視可以分配的硬體及配置
```
![](https://i.imgur.com/FenBynE.png)
- 在cgroup資料夾下內的資料夾a下創建一個資料夾b,b會繼承a資料夾裡面所有的資料  
![](https://i.imgur.com/upiwP02.png)
### 實作檢測控制memory分配資源  
```
$ nano memlimit.sh 
#!/bin/bash

[ ! -d /sys/fs/cgroup/memory/demo ] && sudo mkdir /sys/fs/cgroup/memory/demo

echo "100000000" | sudo tee /sys/fs/cgroup/memory/demo/memory.limit_in_bytes &>/dev/null

echo $$ | sudo tee /sys/fs/cgroup/memory/demo/tasks &>/dev/null

cat /dev/zero | head -c $1 | tail

``` 

- cat /dev/zero →讀取/dev/zero 一直丟asc碼的0出來
- Head –c → 一次顯示一個字
- cat /dev/zero | head -c $1 | tail 利用tail需要有acs 13(enter)才會換行的特性  
- 100000000近似100MB 
- 此程式是往在/sys/fs/cgroup/memory/demo/memory.limit_in_bytes &>/dev/null設定記憶體使用大小標準

- 占用echo $$ | sudo tee /sys/fs/cgroup/memory/demo 下的資料夾的記憶體

![](https://i.imgur.com/Ie4N2VB.png)
- 記憶體使用量超過100mb時就會報錯
## 自定 CPU控制群組    
![](https://i.imgur.com/fZA6osK.png)
1024 可運用資源




