# 安裝Alpine(Disk)

---
Alpine Linux是一個由社群開發的基於musl和BusyBox的Linux作業系統，該作業系統以安全為理念，面向x86路由器、防火牆、虛擬私人網路、IP電話盒及伺服器而設計。輕巧簡便

下載網址
---
- [官方網站](https://alpinelinux.org/)
- [Download STANDARD Link(X86_64)](https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/x86_64/alpine-standard-3.13.4-x86_64.iso)
![](https://i.imgur.com/h7U60vH.png)
###### tags: `下載` `版本`

安裝流程(IN VM_Player)
---
**架設VM Player(workstation)環境**
- 開啟VM Player 選擇Create a New Virtual Machine
- 選擇I will install later 架設空的虛擬機
- 選擇Linux環境 Kernel版本選擇 [Other Linux 5.x or later X64 ]
![架虛擬機](https://i.imgur.com/WqXinHe.png)
![選環境版本](https://i.imgur.com/Djs99FE.png)

**選擇虛擬機名稱與存放位置**
- 依照虛擬機功能與作業系統和網段代號或編號設名稱,例AR5301[A(Alpine)R(Router)53(NetworkId)01(設備編號)]
- 依照項目,部門、公司架構建置專屬資料夾儲存

![命名](https://i.imgur.com/N4GL3Eh.png)
![位置](https://i.imgur.com/tNGJevg.png)

###### tags: `命名` `位置`

**設定虛擬機內容**
- 檔案存放方式選擇 Store Virtual disk as single file (使用單一檔案儲存)
###### tags: `Spilt...為分割檔案方式儲存 只fit FAT32格式(少人用)` 
- 依照虛擬機目的及營運項目計算預估儲存空間及增加預計成長空間自定義虛擬機硬體、內存配備
- 依照虛擬機目的與架構環境大小增加刪除設備(選擇Customsize Hardware -- ADD)
- 置入CD/DVD(選擇Customsize Hardware - CD/DVD - Use iso image file)
![Seting](https://i.imgur.com/CUzlnOe.png)
![項目](https://i.imgur.com/5o97sic.png)
###### tags: `依情況 ADD 可增加刪除虛擬設備` 
![CD/DVD](https://i.imgur.com/sxVOQHP.png)
- 設定完成選擇剛設定好的虛擬機後開啟Play Virtual Machine(若有設定需要改動可以選下方Edit Virtual Machinec或是左上Player>倒三角>manage>Edit...)
###### tags: `Edit Virtual Machine`


---
安裝Alpine流程
---
**執行安裝、Set root**
- 執行後輸入管理使用者名稱:root
###### tags: `root`
- 執行安裝程序
```
setup-alpine
```
![root](https://i.imgur.com/O4JZuan.png)
![setup](https://i.imgur.com/6fbFpuM.png)
**設定語系、鍵盤配置Set KeyBoardLayout/Varlant**
- 依照配置/區域KeyBoardLayout選擇(tw)
- 依照配置/區域選擇Varlant(tw)
![Layout](https://i.imgur.com/iUuDih6.png)

**設定主機名稱localhost/網路設置eth0**
- 依照虛擬機功能與作業系統和網段代號或編號設主機名稱,如上命名方式ar5301
###### tags: `主機名稱只能使用[a-z],[0-9]or特殊字元 -`
- 網路設置預設(eth0)-虛擬網路卡0 輸入enter
- 若設定完成後要更改設置輸入指令
```
sudo nano /etc/networks/interface
```
- IP位置來源(DHCP或手動設置直接key ip)-預設[DHCP]輸入enter
- 要不要手動設置網路配置(manual net config)[y/n]-預設[n]輸入enter
![localhost/interfaceset](https://i.imgur.com/qJk8Q2V.png)

**設定管理使用者密碼root pwd**
- key root twice
![rootpd](https://i.imgur.com/V0bEidR.png)
**輸入時區/City**
- 依照所在地區輸入- Asia (第一個字大寫First Upper)
- 依照所在城市輸入- Taipei (第一個字大寫First Upper)
![zone/city](https://i.imgur.com/3uwKgn8.png)
###### tags: `timezone` `city`
**設定代理伺服器Proxy時間管理協定NTP**
- 依照公司環境有無代理伺服器輸入Address -預設[none]輸入enter
- 選擇時間管理協定伺服器NTP Server -預設[chrony]輸入enter
[Wiki-NTP](https://zh.wikipedia.org/wiki/%E7%B6%B2%E8%B7%AF%E6%99%82%E9%96%93%E5%8D%94%E5%AE%9A)
![proxy/ntp](https://i.imgur.com/ig3LuJd.png)
###### tags: `proxy` `NTP`
**設定Alpine安裝來源/安裝SSH Server**
- 依照區域與連線設置相近或快速的來源(如tw網站)-輸入6
```
若無法出現列表並出現可以檢查網路設置後輸入f 重新刷新列表
若還無法可以重新安裝Ctrl+C
```
- 選擇安裝SSH Server類型-預設[openssh]輸入enter
![apl/ssh](https://i.imgur.com/A24exjI.png)
**選擇虛擬硬碟規格**
- 選擇安裝的硬碟代號-預設[none]輸入sda
- 選擇硬碟的組態(Sys系統碟,Data資料碟,LVM卷軸管理組態碟)-預設[?]輸入sys
- 是否清除該虛擬硬碟上的資料-預設[n]輸入y
![HD Stats](https://i.imgur.com/mRT62Qk.png)
**安裝完成重啟**
- 安裝完成後輸入
```
重啟虛擬機指令
sudo reboot
```
- 並點選左上Manage>Edit virtual Machine把CD/DVD設定自動讀取(退出光碟)

