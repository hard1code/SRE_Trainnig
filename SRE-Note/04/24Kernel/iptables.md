Kernel
![kernel module](https://i.imgur.com/9rbYR4i.png)  
OS(Operating System)  
like Mac IBM-IAX Windows  
Kernle dive  
1. memory  
2. network  
3. disk  
4. CPU  
Shell just a interface  
```
查詢本機有掛載的Module
$ lsmod
```
 
路由判斷:透過查看本機Route -n 選擇封包出路  
封包進入:透過本機網路卡接收到的封包  
Module 為可載入核心模組  
Kernel需要時才掛載  
```
安裝iptables套件
$ sudo apk add tables
查詢目前iptables表
$ sudo iptables -L -n

``` 
![iptable map](https://i.imgur.com/00XpXnO.png) 
![Static Route](https://i.imgur.com/QkMnqEj.png)  
以上兩圖為例
當10.233.0.130要ping至10.233.0.189之設備
透過設定.189 設備的Iptables
```
$ sudo iptables –I INPUT –s 10.233.0.130 –j DROP
$ sudo iptables -L -n
```
可以透過上述設定看到
![iptable -L -n](https://i.imgur.com/NFRuSCi.png)
```
參數
- I
- A

```
[FORWARD 與 NAT 規則](https://web.mit.edu/rhel-doc/4/RH-DOCS/rhel-sg-zh_tw-4/s1-firewall-ipt-fwd.html)  

