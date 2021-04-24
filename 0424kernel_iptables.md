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
![iptable map](https://i.imgur.com/00XpXnO.png)  
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
[FORWARD 與 NAT 規則](https://web.mit.edu/rhel-doc/4/RH-DOCS/rhel-sg-zh_tw-4/s1-firewall-ipt-fwd.html)  
