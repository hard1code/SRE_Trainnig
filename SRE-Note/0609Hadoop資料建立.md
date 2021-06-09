# 0609Hadoop 資料建立
EXTRACT提取資料(整裡資料)
LOAD:把整理好的資料上傳到HDFS(Hadoop)

## LOAD資料
$ hdfs dfs -ls /sre
drwxrwxr-x+  - bigred bigboss          0 2021-06-08 16:59 /sre/alpha
$ hdfs dfs -mkdir /sre/alpha/sre102
```
$ hdfs dfs -getfacl /sre/alpha/sre102
# file: /sre/alpha/sre102
# owner: sre102
# group: bigboss
user::rwx
group::r-x
other::r-x
acl command =Acess Cotrol List
superUser為一般使用者創建權限
```
HDFS不行把檔案局部修改,只能把檔案直接置換
- 檔案的Load
```
$ hdfs dfs -put -f radio.txt /sre/alpha/sre102/
-f:是Force 強迫覆蓋
```
- 檔案的Append
```
從本機檔案,添加到HDFS上
$ hdfs dfs -appendToFile apdfile /sre/alpha/sre102/data.txt
```
- 檔案的複製
```
$ hdfs dfs -cp -f /sre/echo/sre301/radio.txt /sre/alpha/sre102
複製到本機
$ hdfs dfs -copyToLocal sre/alpha/sre102/mysys.sh 
```
- 檔案下載
```
$ hdfs dfs -get -f /sre/alpha/sre102/mysys.sh  ~/
```
- 在Hadoop上執行程式(提升安全等級)
$ hdfs dfs -cat /sre/alpha/sre102/mysys.sh | bash
Hadoop是一個多人共享系統(似NFS)