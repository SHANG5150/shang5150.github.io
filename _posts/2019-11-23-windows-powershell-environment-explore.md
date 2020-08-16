---
layout: post
title: PowerShell 環境探勘
---

記錄自己學習 Windows PowerShell 的使用方法。先摸索 PowerShell 環境裡面有哪些資訊可以使用，並紀錄一下探勘環境資訊的方法。

<!--more-->

雖然過去一直都在 Windows 上從事軟體開發工作，但幾乎很少使用 Windows 的命令列來處理問題，反而比較熟悉 bash。在調查了目前 (2019) 使用 Windows 命令列比較合適的工具後，選擇跳過 cmd, VBScritp 直接切入 PowerShell 做為學習與應用的工具。作為 PowerShell 入門學習，紀錄一些用來取得 PowerShell 環境內相關資料的管道和方法。

### PowerShell 入口

PowerShell 的開啟入口有以下幾種方式 :

1. 按一下 [Windows] 按鈕，搜尋 key-word "powershell"，直接點選，以一般使用者開啟，或按滑鼠右鍵 -> 以系統管理員身份執行。
2. 使用快捷鍵 [Win] + [x] 開啟 Windows 桌面工具列的 Win 圖示右鍵選單。點選 [i] 以一般使用者開啟，或 [a] 以系統管理員身份執行。
3. 開啟檔案總管，切換到預期使用 PowerShell 工作的目錄。點選左上角 [檔案]->[開啟 Windows PowerShell]。
4. 使用 PowerShell ISE。PowerShell ISE 是 PowerShell 的整合環境，可編輯多行指令與提供相關便利功能。

### 命令補完

PowerShell 提供 [Tab] 和 IntelliSense 兩種命令補完方法。

```PowerShell
# [Tab] 命令補完
PS> get-ch    # 按下 [Tab]
PS> Get-ChildItem    # 自動補完到 Get-ChildItem

# [Tab] 參數補完
PS > Get-ChildItem -    # 按下 [Tab]
PS > Get-ChildItem -Path    # 自動補完參數名稱，繼續按 [Tab] 可輪替更換參數

# [Tab] 參數數值補完
PS > Set-ExecutionPolity -ExecutionPolicy     # 按下 [Tab]
PS > Set-ExecutionPolity -ExecutionPolicy AllSigned    # 自動補完可用的參數

# [Ctrl] + [Space] 使用 IntelliSense 叫出候選項目選單
PS > Set-ExecutionPolicy -    # 按下 [Ctrl] + [Space]
PS > Set-ExecutionPolicy -ExecutionPolicy    # 顯示可用上下左右箭頭選取的候選字清單
ExecutionPolicy      Confirm              WarningAction        InformationVariable
Scope                Verbose              InformationAction    OutVariable
Force                Debug                ErrorVariable        OutBuffer
WhatIf               ErrorAction          WarningVariable      PipelineVariable

# PowerShell IntelliSense 一樣可用在命令和參數數值的補完
# 在中文輸入法環境，[Ctrl] + [Space] 會與 PowerShell IntelliSense 衝突
# 直接切換到英文輸入法即可解決
```

### 獲取 PowerShell 說明文件

在網路上可以找到許多 PowerShell 的相關說明。然而 PowerShell 本身也提供了許多說明文件。

```PowerShell
# 更新說明文件
PS > Update-Help
```

PowerShell 的說明文件存放在 ```C:\Windows\System32\WindowsPowerShell\v1.0\en-US``` 。因為位於 ```Windows/System32``` 目錄之下，執行更新時，需要具有 Administrator 權限。

### 查找 PowerShell 相關說明文件

```PowerShell
# 說明文件命令
Get-Help    # 取得說明文件文字
help        # 取得說明文件，並傳送給 more 指令，一次顯示一頁
man         # help 的 Alias

# 用 key-word 查找相關指令或說明文件
PS > help *keyWord*

# 使用預設網頁瀏覽器開啟線上說明
PS > help help -Online

# 使用 Window GUI 查看說明
PS > help help -ShowWindow
```

使用 ```-ShowWindow``` 選項開啟 help 文件，可使用 Get-Help 內嵌的 GUI 搜尋文件內容，或使用設定介面過濾出想要看的說明區段。

### 說明文件的語法 (SYNTAX) 判讀

```PowerShell
# 以 Get-EventLog 為例
Get-EventLog [-LogName] <String> [[-InstanceId] <Int64[]>] [-After <DateTime>]  [-AsBaseObject ] [-Before <DateTime>] [-ComputerName <String[]>] [-EntryType  <String[]>] [-Index <Int32[]>] [-Message <String>] [-Newest <Int32>] [-Source  <String[]>] [-UserName <String[]>] [<CommonParameters>]

# 參數 [-LogName] <String>
[-LogName]    # 表示 -LogName 這個參數選項名稱可寫或不寫
<String>      # 跟在 -LogName 參數後面表示為這個參數選項的值。沒有中括號表示為必要參數

# 參數 [[-InstanceId] <Int64[]>]
# 整組參數使用中括號包圍，表示這個參數不是必要項目
[-InstanceId]    # 使用中括號包圍，表示這個參數選項名稱可寫或不寫
<Int64[]>        # 內側未使用中括號包圍，表示在有指定 -InstanceId 時，為必要參數
```

SYNTAX 的各別參數說明，在說明文件的參數說明區段。

```PowerShell
# 使用 -detailed 或 -full 查閱
PS > help Get-EventLog -detailed
PS > help Get-EventLog -full
```

相關說明文件。

```PowerShell
PS > help about_Command_syntax
PS > help about_Parameters
```

### 查找可用指令

```PowerShell
# 列出當前環境下，可使用的所有指令
PS > Get-Command *
PS > gcm    # Get-Command 的 alias

# 取得 Command 屬性
PS > Get-Command ls | Format-List *
```

Get-Command 與 Get-Help 不同，前者的資料來源是直接從 Command code(或執行檔) 取得，而 Get-Help 的資料來源是額外下載的文件檔案。


### 查詢 function 內容

有些 Command 是在 PowerShell 內，以 function 的類型存在，並在說明文件部分進行 hlep 文件重導向。在這種狀況下，查看 function 的定義，可能會比查閱對應 hlep 來得有用。

```PowerShell
# function mkdir 會重導向文件到 New-Item
PS > help mkdir -ShowWindow        # 這樣會開啟 New-Item 說明文件
PS > Get-Content function:mkdir    # 取得 mkdir 的 Definition
```

### 別名 (Alias)

查詢別名的方法 :

```PowerShell
# 列出全部的別名
PS > Get-Alias    # 取得當前 Session 可用的 Alias
PS > gal          # Get-Alias 的 alias
PS > alias        # 功能與 Get-Alias 相同

# 查看別名的原始命令
PS > gal cd       # 查看 cd 的原始命令 (cd -> Set-Location)

# 查詢某個命令是否有對應的 alias
PS > Get-Alias -Definition Set-Location
```

命令參數的簡短輸入方法。

```PowerShell
# 使用 Window GUI 查看 Get-ChildItem 的原始命令
PS > help Get-ChildItem -ShowWindow

# 可簡化 -ShowWindow 參數成 -s
PS > help Get-ChildItem -s
```

設定新的別名。

```PowerShell
# 給 Command 設定新的別名
PS > Set-Alias -Name gh -Value Get-Help
```

PowerShell 的 Alias 與 Linux 系統慣用的 Alias 不同。PowerShell 的 Alias 不能在設定時夾帶參數。要製作有預設參數的別名，可以改用 function。

### 查詢環境資訊

PowerShell Providers 提供了共通的資料存取方法，方便以共通的磁碟機結構和指令來操作不同類型的資料。

```PowerShell
# 取得 PowerShell 支援的 Provider 類型
PS > Get-PSProvider

# 列出當前的磁碟機
PS > Get-PSDrive

# 取得磁碟機資料
PS > Get-ChildItem C:/          # 取得 C 磁碟機下的檔案目錄
PS > Get-ChildItem HKCU:        # 取得 WIndows 登錄資料 HKEY_CURRENT_USER 下的內容
PS > Get-ChildItem Env:         # 取得 WIndows 環境變數的內容
PS > Get-ChildItem Variable:    # 取得當前 PowerShell 的有效變數
PS > Get-ChildItem Function:    # 取得當前 PowerShell 的有效函數
```

### 查詢 Command 的 Return Object Type 與 Member

PowerShell 的 Command I/O 是基於物件運作。想要對一個物件進行 Method 呼叫，或存取 Property value 前，要先確認清楚有那些 Member 可以使用。

```PowerShell
# 取得 Get-Process 指令的 Output 物件 Type。
PS > Get-Process | Get-Member
   TypeName: System.Diagnostics.Process
Name                       MemberType     Definition
----                       ----------     ----------
Handles                    AliasProperty  Handles = Handlecount
Name                       AliasProperty  Name = ProcessName
NPM                        AliasProperty  NPM = NonpagedSystemMemorySize64
PM                         AliasProperty  PM = PagedMemorySize64
......(略)

# TypeName 指出 Get-Process 的回傳物件型別是 System.Diagnostics.Process
# MemberType 欄位列出每個 Process 成員的類型

# 取出物件 Property 資料的方法
PS > (Get-Process).Name                       # 列出所有 Process 物件的 Name value
PS > Get-Process | Select-Object Name, CPU    # 列出 Process 物件的 Name 和 CPU property value
```