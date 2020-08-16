---
layout: post
title: 使用 SSH 與 Ubuntu 連線設定
---

在安裝好 Ubuntu 之後，預設已經安裝並啟用了 SSH 作為遠端連線工具。以下紀錄一些使用 SSH 連線的方法和相關設定。

<!--more-->

使用 SSH Client 連線到 SSH Server 有數種方法，因為剛安裝好的 Ubuntu 預設啟用了 SSH Server，並允許使用密碼當作登入驗證方式。所以在 Ubuntu 安裝好，並確認能夠連上網路後，就能夠用 SSH Client 連線登入 Server，不需要對 SSH Server 做其它設定。

### 使用密碼驗證的方式透過 SSH Client 連線登入 SSH Server

最簡易的登入命令，只要指定使用者名稱 (user) 和主機位置 (hostname) 就可以進行連線。

```
> ssh user@hostname
        # 第一次連線時，會提示 SSH Server 不在 known_hosts 清單之中，並詢問是否要繼續連線。
yes     # 直接回答 yes 讓 ssh 將 SSH Server 加入已知的 host 清單。
        # 接著會要求輸入在上方指定要登入的 user 的密碼。
```

在第一次連線到新 SSH Server 時，SSH Server 會印出 SSH Server 的 ECDSA key fingerprint，並詢問是否要繼續連線。如果輸入 yes ，SSH Client 會將 Server 的 Public Key 寫入 ```~/.ssh/known_hosts``` 檔案。之後再次連線到相同的 Server 時，就不會再次詢問是否要繼續連線。

如果平時慣用的 Server 本身沒有什麼更動，但是在某一次的連線忽然跳出 Unknow host 的連線詢問，那就要注意是不是不小心連上了其它的 Server，或是網路環境發生了資安相關問題。

### 設定 SSH Server 僅限 Key 驗證登入

Server 有了對外 IP 之後，容易招引一些網路上暴力猜密碼的機器人。系統剛裝好三天，```auth.log``` 就記錄了數萬筆嘗試以 root 或奇怪 User name 登入的 log。

```
$ cat /var/log/auth.log | cut -d' ' -f1,2 |uniq -c
   1078 Oct 20
  32843 Oct 21
  14036 Oct 22
```

為了避免 User name 和密碼被暴力破解出來，還是關閉密碼驗證，改用 RSA Key 作為登入驗證比較安全。

設定的步驟順序如下 :

1. [在 Client 產生 RSA Public Key 以及 Private Key](#在-client-產生-rsa-public-key-以及-private-key)
2. [將 Client 的 Public Key 上傳到 Server](#將-client-的-public-key-上傳到-server)
3. [將 Server 端的密碼驗證登入關閉](#將-server-端的密碼驗證登入關閉)
4. [重新啟動 Server 的 sshd Service](#重新啟動-server-的-sshd-service)

#### 在 Client 產生  RSA Public Key 以及 Private Key

如果已經有產生過 RSA Key 的話，也可以直接拿已經建立好的 Key 使用。

```
PS C:\> ssh-keygen
    # 第一步會提示產生 public/private RSA Key。
    # 接著要求輸入 Key 的存放位置。
    # 如果沒有輸入存放位置的話，預設會放在當前 User 的 ~/.ssh/id_rsa。
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\User/.ssh/id_rsa):    # 直接按下 Enter 使用預設位置
    # 接著會要求輸入 Key 的 Passphrase。
Enter passphrase (empty for no passphrase):    # 直接按下 Enter。
Enter same passphrase again:    # 重複驗證輸入的 Passphrase，同樣按下 Enter 即可。
```

如果有設定 RSA Key 的 Passphrase，可以在 Key 外流的時候多一層防護。但是在每次登入時，都會詢問 Key 的 Passphrase。需要另外使用 ssh-agent 來另外管理 Keys。

#### 將 Public Key 導入預期要用來登入的 User 的 ~/.ssh/authorized_keys 檔案

如果 Client 是在 Linux 系統下，會有現成的 ssh-copy-id 可以處理這件事。但在 Windows 底下還是得自己處理。

```
# 先把 Public Key 從 Client 複製到 Server 上。

PS C:\> scp ~/.ssh/id_rsa.pub user@host:~/.ssh/id_rsa.pub

# 進入 Server 端進行設定
PS C:\>  ssh user@hostname

# 把剛才複製到 Server 的 Key 匯入 authorized_keys 檔案
$ cat ~/.ssh/id_rsapub >> ~/.ssh/authorized_keys
```

Ubuntu 安裝已經為使用者建立好 ```~/.ssh``` 目錄，如果是其它的 Linux 發行版，可能會需要自己建立 ```.ssh``` 目錄。

```
$ mkdir ~/.ssh
```

#### 將 Server 端的密碼驗證登入關閉

```
# 修改 SSH Server 的 config 檔案
$ sudo vim /etc/ssh/sshd_config

# 允許使用 Key 驗證登入
PubkeyAuthentication no    # 改為 PubkeyAuthentication yes

# 禁止使用 Password 驗證登入
PasswordAuthentication yes    # 改為 PasswordAuthentication no
```

#### 重新啟動 Server 的 sshd Service

```
# 重新啟動 SSH Server
$ sudo service ssh restart

# 使用 Key 登入 SSH Server
設定完 Server 的 Key 驗證之後 SSH Client 不必再設定任何東西。只需要直接登入 SSH Server 即可。

> ssh user@hostname
```

SSH Client 登入時，預設會在 ```~/.ssh``` 目錄下尋找對應的 Private Key 進行驗證。

### SSH Server Port 設定
將 SSH Server 設定好僅限 Key 驗證後，暫時不必擔心被暴力猜密碼機器人攻破。但 ```auth.log``` 仍然持續被機器人大量刷登入失敗的 log 訊息，可能造成真正重要的 ```auth.log``` 被登入機器人的試錯 log 洗掉。

多數登入機器人只針對預設 Port 22 進行試錯登入。或許是認為使用預設設定的 Server 有比較大的機率，在登入驗證部分也只使用允許密碼登入的預設設定，可能比較容易入侵成功。

所以簡單的因應方式就是直接把 Port 22 改成其它 Port 號碼。

```
# 修改 SSH Server 的 config 檔案
$ sudo vim /etc/ssh/sshd_config

# 變更 Port Number
Port 22    # 改為 Port 2222

# 重新啟動 SSH Server
$ sudo service ssh restart
```

目前 Server 放在 NAT Server 後面的情況也比較普遍，所以不修改 SSH Server 的 Listen Port，而直接使用 NAT Port 轉傳 (Port Forwarding) 功能設定對外開放 Port 號碼也可以。

一般便宜家用路由器有提供 NAT Port Forwarding 功能的也滿街都是了。直接使用路由器的 Web GUI 進行設定即可。

![nat-port-forwarding-settings](/assets/img/post/2019-11-02-ssh-usage-and-config-settings/nat-port-forwarding-settings.png)

設定好 Port Forwarding 之後，使用 SSH Client 就需要明確加入指定 Port 號碼的選項進行連線。

```
$ ssh -p 2222 user@hostname
```

因為指定了其它 Port 號碼進行連線，會因為新連線的 Port 號碼不同，提示 SSH Server 不在 ```known_hosts``` 清單之中，並詢問是否要繼續連線。在回覆 yes 將 SSH Server 加入 ```Known_hosts``` 清單之後，可以將之前使用預設 Port 22 進行連線的 host 資料從 ```Know_host``` 清單中移除。

從經驗上來看，將 Port 從 22 改掉之後，就能避掉多數登入機器人了。如果碰到針對性的攻擊，機器人先掃描 Server 所有的 Port 來確認 SSH Server 正確 Port 號碼的狀況，就得考慮更進一步使用如 ```fail2ban``` 來自動封鎖那些多次試錯失敗的 IP 位址。

### 避免 SSH 斷開連線
使用 SSH 連線到 Server 監看 log，有時候會有長時間沒有 log 的狀況。而 SSH 的連線在一段時間之內沒有封包通過，就會自動切斷 TCP 連線。

可能造成自動中斷的原因很多 :

* 作業系統在 SSH Client 長時間沒有收到資料後 time out。
* 防火牆在長時間沒有收到資料 time out。
* NAT Server 在建立連線後，長時間沒有收到資料 time out。
* SSH Server 長時間沒有收到資料 time out。
* 其它原因。

各種設備都有提供對 time out 回收 Port 佔用機制的設定，但是要逐一設定各個設備並不合適，而且並不是所有 Service 都真的會需要在沒有資料 I/O 的情況下保留連線狀態。

SSH Client 和 SSH Server 對於保持連線狀態都有提供相關設定。兩邊的設定預設都是會在一段時間沒有資料流通後就進行 time out，回收掉對應的網路資源。

在設定規劃的原則上，因為 Server 的網路資源是相對有限的，所以建議不要直接在 ```sshd_config``` 將 Server 設為對所有 Client 保持連線。而是在 Client 連線時，自行加入 KeepAlive 的連線選項。

```
# 使用 -o 指定 ServerAliveInterval 的值為 30 秒
$ ssh -p 2222 -o ServerAliveInterval=30 user@hostname
```
SSH Client 與保持連線的相關設定有三個。

```
TCPKeepAlive
ServerAliveInterval
ServerAliveCountMax
```

---
##### TCPKeepAlive

TCPKeepAlive 選項用來設定是否要對 Server 送出 TCP keepalive 訊息。預設值是 yes，表示開啟。設為 no 表示關閉。

##### ServerAliveInterval

ServerAliveInterval 用來設定在沒有收到資料的多少秒之後，會向 Server 送出一個加密的請求回覆訊息，來確認 Server 是否還活著。預設是 0 秒，也就是不對 Server 送出確認訊息。這也是為什麼 TCPKeepAlive 預設雖然是 yes，但 SSH Client 仍會 time out 的原因。

##### ServerAliveCountMax

ServerAliveCountMax 是用來設定，當 Client 對 Server 送出請求回覆訊息，而沒有收到回覆的最大次數。預設值是 3，表示如果 Client 連續發起 3 次的回覆請求都沒有收到時，會將 Server 狀態判斷為已離線，並結束 SSH Client 的連線。

---

在網路環境比較糟糕，或是路由器可能會暫時關閉的環境下，可能需要斟酌是否要使用保持連線的功能。在這樣的環境下，有可能因為短時間內無法建立與 Server 的連線而造成 SSH Client 一直被斷開的狀況。

### 使用 SSH Client config 簡化登入命令
如果經常需要連線到 SSH Server，每次登入都需要輸入一長串的參數和目的地位址也是挺麻煩的。一種 Command Line 的通用解決方法是將登入指令保存為批次命令，下次連線時，就可以直接執行。而 SSH Client 提供了另一種方法，將登入所需要相關參數寫在 Config 檔案中。當需要連線到指定 SSH Server 時，只要簡單輸入 Host 別名即可。

在 ```~/.ssh``` 目錄下建立一個純文字檔，命名為 ```config```，並將對應參數寫入其中。

```
Host     MyHost             # 連線目標 host 的別名
Hostname host.example.tw    # 目標主機的網域位址或 IP 位址
Port     2222               # SSH Server 使用的 Port 編號
User     UserName           # 預期登入的使用者名稱
ServerAliveInterval 30      # 指定請求 Server 回應的間隔秒數，避免連線 time out
```

config 儲存完畢後，只要使用 Command Line 指定 SSH 要連線的 Host 別名，SSH Client 就會在 config 檔案中找出要使用的對應參數並套用。

```
# 用別名連線到 SSH Server
$ ssh MyHost
```

如果需要在同一台 Client 連線到多部不同的 SSH Server，只要依序將其它 Host 別名的相關設定追加到 config 下方即可。

### 參考資料

* [OpenSSH 文件](https://www.openssh.com/manual.html)
* [Linux使用ssh超时断开连接的真正原因](http://bluebiu.com/blog/linux-ssh-session-alive.html)

##### 資安相關問題

* [How to stop/prevent SSH bruteforce](https://serverfault.com/questions/594746/how-to-stop-prevent-ssh-bruteforce)
