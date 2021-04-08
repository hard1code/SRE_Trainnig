# SSH to someone PC without passwd
**Step1**
```
$SUDO APT install  SSH
會同時安裝SSH Client 及 OpenSSH  Server 
$SUDO APT install  openssh-server
只會裝 OpenSSH server 並未安裝SSH Client

```
**Step2**
```
sudo nano /etc/ssh/ssh_config
```
   ```
 ---------------------
 # StrictHostKeyChecking ask
  # IdentityFile ~/.ssh/id_rsa
  # IdentityFile ~/.ssh/id_dsa
  # IdentityFile ~/.ssh/id_ecdsa
---------------------  
```
把StrictHostKeyChecking ask 去掉#、ask在後面加上no
![ssh_config_ask](https://i.imgur.com/8nPwQPk.png)
             ****************改成****
![ssh_config_no](https://i.imgur.com/CX1Xn2Y.png)


```
StrictHostKeyChecking no
```
就不用在SSH的時候輸入Yes/No
![SSH_Link](https://i.imgur.com/j48x9P8.png)

**Step3在client端產生公/私鑰後傳給**


