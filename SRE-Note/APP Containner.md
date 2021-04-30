# Application Container 軟體貨櫃(容器)
---
![](https://i.imgur.com/Xmx9l55.png)
![](https://i.imgur.com/zJ5I2dv.png)
:Arrow_Up:**ISOLATED隔離:
### 就是把一個Program變成一台電腦擁有專屬的網路、根目錄、Process**
Bins:其他的程式(命令)
Lib:相依檔
.so:System object
.ko:Kernel Object
server:硬體主機
Container 共用Kernel(OS)不用開機程序
![](https://i.imgur.com/u9OASco.png)

## NameSpace
* cgroup
* ipc:Container之間的溝通,InterProCess
* mnt:mount把特殊設備掛載至某一個目錄區
* net:每一個Container都有自己的網路
* pid:
* user:每一個Container都有自己的owner
* uts:每一個Container都有獨立的電腦名稱
* time:
unshare就是namespace
只要是Container的root，就會被限制，不在是無所不能。

## Slirp4netns
--net:只有空間沒有modern(網路設備)

## Chroot
```
1.建立要隔離的FS(File System)
mkdir -p r2d2/rootfs/{bin,lib/x86_64-linux-gnu,lib64,usr/{bin,sbin},sbin,proc}

2.把我們要隔離的程式複製到要隔離的FS資料夾上(舉例busybox)
$ which busybox
```

## ovarlay2
APPContainer 的系統都是用overlay2
webserver 的資料系統，只要修改cdrom就可以更新網站