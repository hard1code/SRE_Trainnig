# 04/27 Alpine.iso改裝 TCL  
---
### 查詢指令是否存在  
```
$ command -v {命令名稱}
```
![](https://i.imgur.com/FVdDsO4.png)
### 改裝alpine.iso  
- mkdir /wk/alpine/{backup,iso,initramfs}
- ![](https://i.imgur.com/R53uwnt.png)

![](https://i.imgur.com/rftZMT9.png)
- syslinux grub是bootloader 
- bootloader載入vmlinuz-lts(kernel),initramfs-lts再把這個系統記憶體的位置傳給kernel  
- kernel再去看有沒有這個檔案  

```
$ file iso/boot/initramfs-lts
```
![](https://i.imgur.com/FiojHlR.png)

alpine 開機後再/media會有光碟片  

# Alpine  
- 第一個程式是busybox  
- inittab第一個程式是呼叫openrc  
- Alpine 的init會新增一層目錄並把權限轉移，所以在  initramfs -lts裡建立或修改資料夾，在開機後無法移植成功，但是init會掛載我們預設的cdrom/usb/flpy資料夾  
- 必須透過修改init的系統檔案程式,執行我們埋在cdrom裡的程式  

### 修改/etc/init檔案 呼叫cdrom我們預埋的程式  
- 修改Alpine/initramfs/etc/init  
![](https://i.imgur.com/fj9NMwU.png)  


- Boot loader設定檔  
![](https://i.imgur.com/SRMZoSl.png)  
![](https://i.imgur.com/aRdWHpK.png)

## 設置Alpine開機選單  
在~/wk/alpine/iso/boot/syslinux裡面修改syslinux.cfg  
- 在alpine裡面呼叫kernel與bootloader  
- 需要把要增加  
![](https://i.imgur.com/NHakIPa.png)

## 設置tinycore開機選單  
- 在~/wk/alpine/iso/boot/isolinux裡面修改  isolinux.cfg  
- 查看iso/boot/core.gz/isolinux 版本  
```
$ file isolinux
```
## 要在網站syslinux下載4.05版的GUI menu  
[syslinux](https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux/Testing/)

- 放到~/wk/tinycore/iso/boot  
```
$ pwd
/home/desktop/wk

$ mkdir isolinux
$ 7z x syslinux-4.05-pre7.zip -osyslinux

```
- 把上步驟下載的GUI檔案vesamenu.c32複製至boot/isolinux  
- 修改bootloader開機檔isolinux.cfg  
- 添加 UI vesamenu.c32  
```
cp ~/wk/isolinux/bios/com32/menu/vesamenu.c32 boot/isolinux
nano boot/isolinux/isolinux.cfg
```
![](https://i.imgur.com/foc0jvY.png)
