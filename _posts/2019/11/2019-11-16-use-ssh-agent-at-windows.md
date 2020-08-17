---
layout: post
title: 在 Windows 使用 SSH Agent
---

ssh-agent 用來儲存 Private key 給之後使用 ssh 登入遠端機器時使用。避免經常需要切換 ssh 或是使用自動化腳本時，一再要求輸入密碼的麻煩。

<!--more-->

### Windows 使用 ssh-agent 前的設定

在 Windows 10 (1903) 自帶的 OpenSSH 套件裡， ssh-agent 有 bug，而且遲遲沒有更新到最新版 OpenSSH。所以如果要使用 ssh-agent 則需要自行安裝最新版 OpenSSH，並手動把環境變數 Path 中的自帶 OpenSSH 路徑移除。

Note: [Windows 10 自帶 OpenSSH 的 ssh-agent issue](https://github.com/PowerShell/Win32-OpenSSH/issues/1263)

接著使用 Chocolatey 安裝最新版 ssh-agent 。

```
PS > Choco install -y openssh
```

Note: Chocolatey 是 Windows 用來取得 Open source 的方法。模式類似 OSX 中的 homebrew 或 Linux 系列中的 apt、rpm 等。

安裝完後，確認 PowerShell 有沒有在正確的路徑找到新安裝的 OpenSSH

```
PS > (Get-Command ssh-agent.exe).Source
C:\Program Files\OpenSSH-Win64\ssh-agent.exe
```

Windows 10 自帶的 OpenSSH 預設安裝位置在 ```C:\Windows\System32\OpenSSH```。如果 PowerShell 仍然在這個路徑找到 ssh-agent，需要另外調整環境變數 Path 的內容。

### Windows 啟用 ssh-agent

ssh-agent 在 Windows 是 Service 類型的工具。對 Service 的操作，需要有管理員權限。以下操作都在具有管理員權限的 PowerShell 中進行。

ssh-agent 預設的 Service StartType 是 "Disabled"。在這個狀態下直接執行 ssh-agent 會收到錯誤訊息。

### 取得 ssh-agent service StartType 狀態

```
PS > (Get-Service ssh-agent).StartType
Disabled       # 確認狀態是 Disabled
```

### 在 ssh-agent service StartType 是 Disabled 的狀況下嘗試執行

```
PS > ssh-agent
unable to start ssh-agent service, error :1058
```

接著要將 ssh-agent service 的 StartType 變更為可手動啟用的狀態。

```
# 把 ssh-agent 的 StartType 設定成手動啟動
PS > Set-Service ssh-agent -StartupType Manual

# 再次確認 ssh-agent 的 StartType 狀態
PS > (Get-Service ssh-agent).StartType
Manual        # 確認狀態成功改為 Manual
```

如果想要把 ssh-agent 設定成開機自動啟用，也可以把 Service StartType 設定為 ```Automatic``` 。

接下來，開始手動啟動 ssh-agent。

```
# 啟動 ssh-agent
PS > Start-Service ssh-agent

# 確認 ssh-agent 是否正常啟動
PS > (Get-Service ssh-agent).Status
Running
```

### 將 Private key 加入 ssh-agent

```
PS > ssh-add.exe
Enter passphrase for C:\Users\User/.ssh/id_rsa:    # 輸入 key 的 passphrase
Identity added: C:\Users\UserName/.ssh/id_rsa (UserName@hostname)
```

如果沒有特別將 Private key 存成別的檔名，ssh-add 會找出所有預設位置的 Key ，並依序加入。

Private Key 預設位置 :

```
~/.ssh/id_rsa
~/.ssh/id_dsa
~/.ssh/id_ecdsa
~/.ssh/id_ecdsa_sk
~/.ssh/id_ed25519
```

### 使用 ssh 遠端登入

```
# 一般登入
PS > ssh userName@hostName

# 使用 config 設定登入
PS > ssh configHostId
```