# 0426Linux 開機流程  
1. BIOS  
- 開機前會做自我檢測，會去找第一個可開機的裝置的第一個磁區  
2. MBR  
-  儲存在硬碟的第一個磁區system.d  
3. Boot Loader  
- 開機管理程式  
- 存在MBR  
- 載入Kernel到記憶體當中  
- 在幫忙載入init到記憶體讓kernel可以工作  
4. Kernel  
- 開啟後開始執行第一支程式(System.d)  
5. Init  
- 負責登入前的檢查動作  
![start step](https://i.imgur.com/IRIiOP7.png)  

## 撰寫BIOS檢查程式  
```
#!/bin/bash
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS

```
### MBR主開機紀錄(Master Boot Record)  
- 位於可開機裝置的第一個磁區  
- 大小 512 Bytes  
- 446 Bytes 儲存開機管理程式（Boot Loader）  
- 4 個 16 Bytes 儲存磁區分割表  
- 2 Bytes 儲存檢查標誌  
- 446(boot loader) + (4(磁區分割表)*16) + 2 = 512  
![](https://i.imgur.com/0NTHtUb.png)  

## MBR 磁碟分割表  
partition 1-4 就是C,D,E槽  
系統分割命名只能是sda 1-4 in linux  
若要分割超過4個必須由sda4來做分割  
也就是命名在sda5以後  
ex如果是分割5個 就是sda 1 2 3 4(5 6) sda4用來記錄這延展磁區的log  

## GPT磁區分割表  
還是有MBR,但只是儲存開機管理程序  
GPT把磁區使用(LBA)一個有 512 Bites 共有128個主要磁區  
LBA 2-33 用來當作儲存磁碟分割資訊  
後面當作備份  
為什麼有128個磁區  
(33-2+1(32個LBA))*512(Bite)/128(一個partition需要128個bites) =128(個磁區)  
![](https://i.imgur.com/yZZ5h0a.png)  
### 128 partition實際儲存資訊↓  
![](https://i.imgur.com/ZvG353r.png)  


## fdisk分割指令(檢查)  
```
sudo fdisk /dev/sda -l
```
目前的linux版本都可以使用fdisk的指令來切割gpt格式  

## gdisk分割指令(檢查)  
```
sudo gdisk /dev/sda -l
```
gpt可以支援128個磁區
## parted 分割指令(檢查)
```
sudo parted /dev/sda unit s print
```

## Boot Loader  
- Chain loading（控制權轉交給其他 loader）  
- 負責載入(kernel)到記憶體並移交控制權給 Linux 核心(kernel)  
- 提供開機選單  
- 目前 GRUB2(開機管理程式) 最受歡迎 還是統稱為GRUB  
- initrd.img/initramgfs:當boot loader載入kernel到記憶體的臨時系統檔(快照檔)  
- grub : boot loader 
- initrd : ubuntu放system.d的(不一定每個linux都是systemd)
- vmlinuz: linux kernel名稱,但是不包含module  
- 模組 ls -al lib/modules  
```
$ ls -al /boot
```
![](https://i.imgur.com/u1SopHM.png)

## GRUB2 啟動流程  
BIOS 在 MBR 中啟動名稱為 boot.img 的 loader  
載入init & boot 到記憶體裡面  
```
$ cat /boot/grub/grub.cfg | grep -A10 vml
```
## 只有在GRUB2才有的↓
![](https://i.imgur.com/KkCNsRf.png)

階段 1  
- boot.img 負責載入階段 1.5 的 loader，名稱為 core.img  
階段 1.5  
- core.img 包含檔案系統的驅動程式  
負責載入位於 /boot 目錄中的階段 2 檔案  
階段 2  
- 將 Kernel 解壓縮到主記憶體中  
- 將 initrd/initramfs 載入記憶體，並將其位址傳給 Kernel  
- 將控制權交給 Kernel  

## Kernel  
- 當Kernel初始化時,會檢查I/O裝置(cpu,ram,人機介面)跟bios類似  
- init在任何linux都是壓縮過的檔案
- bootloader解壓縮init到記憶體裡面(記憶體讀取比硬碟快)在kernel讀取initrd  
- 載入linux kernel module driver  
- init 第一個程式

## initrd & initramfs   
是一個在記憶體裡面臨時的系統檔案  

## 找檔案類型指令 file  
```
file initrd.img

file initrd.img-5.4.0-73-generic

```

## init (linux第一個程式)  
```
ls -al /sbin/init
```
- ubuntu現在都稱為systemd  
- alpine :init  
- 第一個開機程式:system.d  
- tinycore : busybox因為功能太少呼叫外援
![](https://i.imgur.com/WH10ivF.png)

建立軟連結  
```
ln --help
cp /boot/initrd.img ~/wk
unmkinitramfs initrd.img src
tree -L 2

```
![](https://i.imgur.com/2RP89r1.png)

--- 
# 自製 TCL 光碟 (Live CD)  
---
- 查詢版本 OS type 網站  
![](https://i.imgur.com/COGhk5K.png)  
![](https://i.imgur.com/i9OBqRe.png)  
![](https://i.imgur.com/GsnF9ak.png)  
## tinycore install  
![](https://i.imgur.com/Vsved27.png)  
![](https://i.imgur.com/BeyvWZ7.png)  

![](https://i.imgur.com/e5XPA6N.png)  
- 解壓縮 core.  
![](https://i.imgur.com/H2vmy6h.png)  
###### zcat = gz = tar = cpio 都是壓縮解壓縮的工具指令  
```
zcat ../iso/boot/core.gz | sudo cpio -i -H newc
-i 執行解壓縮
-H 指定格式
SVR4 格式檔
```
![](https://i.imgur.com/vLMhEJm.png)  
![](https://i.imgur.com/jywEX9W.png)  

skel 建立使用者範本 profile,bashrc...  
cpio 並沒有參數可以承接檔案 所以必須透過資料流導入做壓縮動作所以必須先做zcat讀取資料流  

![](https://i.imgur.com/UqsSaTs.png)  

	kernel /boot/vmlinuz
	initrd /boot/core.gz
	append loglevel=3
驗證上述  
vmlinuz  
core.gz 的內容  
log有分等級1-6  
這邊設定3  
timeout 300 =3秒  
如超過3秒使用預設 F1 boot.msg開機  

```
xorriso -as mkisofs \
-b boot/isolinux/isolinux.bin \
-c boot/isolinux/boot.cat \
-no-emul-boot \
-boot-load-size 4 \
-boot-info-table \
-V 'TinyCore CM Iso' -o ../tcl-20210426.iso .

-as mkisofs 模擬這個mkisofs命令(當原命令資源不夠的時候)
-b 告訴iso電腦開機的binary檔的位置
-c 同上isdo需要開機的categeroy的檔案 (無法查看)
boot-load-size 4 \在這個命令底是規定128bite一個磁區
*4 為512 =MBR
-boot-info-table \ 參數
-V 'TinyCore CM Iso'  光碟機的名稱
-o ../tcl-20210426.iso .輸出現在這個目錄並給予檔名
參考ISO9660 光碟格式
```
