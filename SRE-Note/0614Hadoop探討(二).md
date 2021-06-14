# 06/14Hadoop探討(二)

Hadoop由四個模塊組成

Common
Hadoop Common就是Client,機器名稱就是在k8s中的Adm機器

HDFS
Hadoop的檔案系統就是HDFS,HDFS由Active namenode,secondNamenode(這兩個在M31上一定要搭配W51-W59上的datanode),standbyNamenode,Datanote這一堆所組成的分散儲存系統

YARN
Hadoop的運算系統Yarn,由resource Manager,node Manager組成,主要的功能就是幫我們跑mapreduse所寫的program
resource Manager 再M32上跑配合W51-59上的node manager 所組成的Yarn系統

MapReduce



---

![](https://i.imgur.com/PvxvRfO.png)
這圖(流程)在Hadoop Client(ADM)上發生
1. 我們會打Wget命令上網下載需要分析的檔案
2. 我們要把這個檔案放進(put)HDFS系統,一定要先在ADM這台電腦上,以每128MB切成一塊,
   以範例來說切成5分


---
![](https://i.imgur.com/pdcaRRN.png)
3. Client會去找NameNode,詢問BlockA,B,C要放在哪個機器上,在rack是機櫃,範0例A Blk放,R1N1,R5Node,5,6
4. 橘色的線就是Client 的流程 先儲存倒R1N1.R5N5,N6,後面是透過從前面傳過來的Client→Node1→Node5→Node6


---
![](https://i.imgur.com/yneAl4E.png)
1. Clinet 要讀取檔案的時候會去問HDFS的name node:block分邊放在哪
2. Namenode會回答給Client每一個Block的多位置
3. Client就會name node回答的第一順位去找Data node1 拿Block1,去data node 5找c block ,去Data node 8 找B block
4. 同學的所裝的Client,都是透過(XML)老師的M31去讀取,雖然清單都一樣但是順序會不一樣No1同學從(1,8,5)(5,1,8)…撈資料
1. No2,會是(5,1,6),(6,2,9)…多人讀取一個檔案會透過[Round-robin](https://en.wikipedia.org/wiki/Round-robin_scheduling)循環式讀取做操作


---
```
$ hdfs dfs -put /etc/passwd /sre/alpha/sre102/
$ hdfs fsck  /sre/alpha/sre102/passwd  -files  -blocks  -locations

```
```
Connecting to namenode via http://m31:50070/fsck?ugi=bigred&files=1&blocks=1&locations=1&path=%2Fsre%2Falpha%2Fsre102%2Fpasswd
FSCK started by bigred (auth:SIMPLE) from /120.96.143.106 for path /sre/alpha/sre102/passwd at Mon Jun 14 11:49:44 CST 2021
/sre/alpha/sre102/passwd 2093 bytes, 1 block(s):  OK
0. BP-1466361734-120.96.143.31-1622864518101:blk_1073754986_14176(這是Block名稱) len=2093 Live_repl=3 [DatanodeInfoWithStorage[120.96.143.53:50010,DS-b86eedcd-1795-455d-9fb5-629b98350644,DISK], DatanodeInfoWithStorage[120.96.143.55:50010,DS-110d99e8-c65e-4d15-9de0-7174141bf38a,DISK], DatanodeInfoWithStorage[120.96.143.51:50010,DS-16f0a5c0-1703-40af-891c-ada823b5cdbc,DISK]]
這是三台存放的位置
```


---

![](https://i.imgur.com/WJpeSa0.png)
1. 土黃色代表檔案已經存在HDFS裡面
2. Map是Hadoop截取資料命令
3. shuffle是Hadoop分類/排序資料命令
4. Reduce是Hadoop分析資料命令
- map做截取/過濾
- Shuffle分類/排序
- Reduce分析/上載


---

![](https://i.imgur.com/xaQUIGc.png)
- Hadoop的Mapreduce可以寫一個程式,讓三台不同電腦並行執行截取/排序每一個map處理的都是獨特的資料(不同Block)
1. 淡藍色的框是一台電腦(W51)
2. Split 0 =Block A...1:B,2:C
3. 白色正方為Disk:w53某一個暫存的硬碟空間大小不會超過128MB
4. 紅色實心線為Shuffle(分類/排序)把每個不同截取/過濾後的block資料放在本機暫存硬碟中
5. 右邊此白方框為W55為51-53的分類排序後的資料組合成一個空白方塊(這裡也做分類排序)這裡需要兩個完整暫存空間,若原始資料大小為100TB/年,這套系統至少600TB/年來做計算(*6(是因為
6. HDFS存三份Block,並每台電腦都需要佔存空間))
7. reduce分析上載至本機電腦反饋給Client查看
8. Shuffle就是yarn.nodemanager這張圖就有4個shuffle,左邊為分類排序,右邊shuffle為分類排序彙整



---
```
cat /opt/hadoop-2.10.1/etc/hadoop/yarn-site.xml
```



