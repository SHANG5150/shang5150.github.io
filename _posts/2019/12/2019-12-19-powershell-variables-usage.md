---
layout: post
title: PowerShell 變數
---

筆記 PowerShell 的變數相關資料。
<!--more-->

### 取得說明

```PowerShell
# PowerShell 的變數使用方法說明文件
PS > help about_Variables

# PowerShell 自動變數的說明。文件中也列出並解釋各個自動變數的功能
PS > help about_Automatic_Variables

# 系統環境變數的說明。
PS > help about_Environment_Variables

# 對用來客製化 PowerShell 行為的相關變數的說明
# 文件中也列出了各個 Preference 變數的預設值
PS > help about_Preference_Variables
```

Note: 雖然使用 Update-Help 更新過 Local 的說明文件，但使用 -Online 選項開啟的網頁版文件內容比較豐富。

### PowerShell 變數特性

* 變數的名稱不區分大小寫。
* 可以包含空白字元，但實務上應該避免。
* 預設值為 ```$null```。
* 可存放任意類型的物件。
* 自行建立的變數預設會放入 Variable Drive。

### 變數類型

* User-Created variables
  由使用者建立，只存活在 PowerShell window 開啟的期間。
* Automatic variables
  由 PowerShell 建立和維護，應被視為 Read-only，不要變動其內容。
* Preference variables
  由 PowerShell 建立，存有環境上的預設變數，可修改這些變數來變更 PowerShell 的行為。

### 確認 Variable 相關 Alias

```PowerShell
PS >Get-Alias -Definition (Get-Command -Noun variable)
CommandType     Name
-----------     ----
Alias           clv -> Clear-Variable
Alias           gv -> Get-Variable
Alias           nv -> New-Variable
Alias           rv -> Remove-Variable
Alias           set -> Set-Variable
Alias           sv -> Set-Variable
```

### 取得變數容器的方法

```PowerShell
# 使用 Get-Variable
PS > Get-Variable PSVersionTable
Name                           Value
----                           -----
PSVersionTable                 {PSVersion, PSEdition, PSCompatibleVersions, ...

# 或是使用 variable 磁碟機
PS > Get-Item Variable:PSVersionTable
Name                           Value
----                           -----
PSVersionTable                 {PSVersion, PSEdition, PSCompatibleVersions, ...
```

### 取用變數內容 Value 的方法

```PowerShell
# 使用 ($) 前綴符號
PS > $PSVersionTable
Name                           Value
----                           -----
PSVersion                      5.1.18362.145
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
......(略)

# 使用 $ 前綴符號加上變數名稱，可以取得 Variable 磁碟機內的變數 Value
# 等同於使用 $variable:PSVersionTable

# 使用前綴符號 ($) 加上變數名稱的調用方法，只能取得 Variable 磁碟機內的資料
# 要取得環境變數 (env 磁碟機) 資料，則需要加上磁碟機名稱
PS > $env:windir
C:\WINDOWS

# 使用 {} 使用那些含有特殊符號或空格的變數名稱
PS > ${env:CommonProgramFiles(x86)}
C:\Program Files (x86)\Common Files
```

### 使用 Get-ChildItem 列出磁碟機中的變數

```PowerShell
# 列出 Variable 磁碟機儲存的變數
PS > Get-ChildItem -Path variable:

# 列出 Environment 磁碟機儲存的環境變數
PS > Get-ChildItem -Path env:
```

### 儲存資料到變數

```PowerShell
# 儲存簡單 Value
$var = 1

# 儲存陣列 {1, 2, 3}
$array = 1, 2, 3

# 儲存指令的 Return value
$process = Get-Process

# 儲存 Hashtable
$hashtable = @{jan = 1; feb = 2; mar = 3}

# 使用 Visual Studio Code 編輯 Profile，宣告常用變數時，改用 Set-Variable 可避免
# "The variable 'ValueName' is assigned but never used. 的警告
Set-Variable -Name variableName -Value "value"
```

### 清除變數內容

```PowerShell
PS > Clear-Variable var
# 或是
PS > $var = $null

# Clear-Variable 會清除變數 var 的內容，但 var 這個變數容器仍然存在
# 清除過後的 var 內容為 $null
```

### 移除變數

```PowerShell
PS > Remove-Variable var

# Remove-Variable 會移除 var 變數的容器和內容
PS > Get-Variable var
gv : 找不到名稱為 'var' 的變數。
......(略)
```

### 變數型別

PowerShell 的物件 Value 預設是鬆散型別 (loosely typed)，可以儲存任意型別的資料。但也可以使用強制轉型符號來限制變數的型別。

```PowerShell
# 限制變數僅能儲存 int 型別資料
PS > [int]$var = 1

# 非 int 型別資料會嘗試轉型為 int 型別，純數值字串可以正確轉型
PS > $var = "123"

# 非數字字串轉型失敗會印出 Error 訊息
PS > $var = "hello"
無法將 "hello" 值轉換為 "System.Int32" 型別。錯誤: "輸入字串格式不正確。"
......(略)
```

### 變數的有效範圍

```PowerShell
# 建立測試資料
PS > $var = "var"

# 執行的 function 可以看到上層建立的變數
PS > function Out-Var{$var}
PS > Out-Var
var

# 在 function 內部重複宣告的變數不會影響上層建立的變數
PS > function Out-LocalVar {$var = "local var"; $var}
PS > Out-LocalVar
local var
PS > $var
var

# 使用 Scope modifier 把變數宣告在 Global 範圍
PS > function Set-GlobalVar{$Global:var = "global var"}
PS > Set-GlobalVar
PS > $var
global var

# 使用 (.) 執行，使 function 或 script 宣告的變數加入或覆蓋掉 Global 範圍
PS > function Set-LocalVar {$var = "local var"}
PS > . Set-LocalVar
PS > $var
local var
```

### Automatic Variables

除了自訂變數以外，PowerShell 也定義了一些自動變數，其內容會依照使用 PowerShell 的狀況自動變更變數值。

以下是比較常用的自動變數。更多的變數可參考 about_Automatic_Variables 文件。
* \$_
  等同 ```$PSItem```，變數值是當前存在於 Pipeline 的物件。
* \$?
  代表最後一次呼叫的命令執行執行成功 (True) 或是失敗 (False)
* \$\^ 和 \$\$
  代表當前 Session 收到的 first token (``$^``) 和 last token (```$$```)。
  例如:
  ```PowerShell
  PS > Write-Output This is a test message.
  PS > $^    # 以字串輸出 "Write-Output"
  PS > $$    # 以字串輸出 "message."
  ```
* \$Error
  收集 Error 訊息的 Array，最近一次的 Error 訊息，可使用 ```$Error[0]``` 取得。
Error array 的 size 由 ```$MaximumErrorCount``` 控制。
* \$true 和 \$false
  代表 True 和 False，用來配合比較運算子進行運算。
* \$null
  代表 null 或 empty value。將 ```$null``` 指定給一個變數表示清空該變數的值。
* \$PROFILE
  其內容包含 Profile 的檔案位置。分別使用各別的 Property 取得。
  ```PowerShell
  PS >$PROFILE.AllUsersAllHosts
  PS >$PROFILE.AllUsersCurrentHost
  PS >$PROFILE.CurrentUserAllHosts
  PS >$PROFILE.CurrentUserCurrentHost
  ```
  如果沒有指定 Property，直接使用 ```$PROFILE``` 會取得 ```CurrentUserCurrentHost``` 的值。
* \$PWD
  其值包含當前所在資料夾的完整位址。

### Environment Variables

PowerShell 的 Env: Drive 提供環境變數的設置。與 Variable: Drive 存放的變數有些差異。

* 與 Variable: 的變數有效範圍不同。
  在 function 內部變更境變數的內容，在回到 function 的 call stack 後仍然留存，行為與 Global variable 相似。
* 統一使用 System.Collections.DictionaryEntry 類別作為容器。
  ```PowerShell
  # 確認方法
  PS > Get-Item env:* | Get-Member
  ```

### 儲存變數

變數的建立和修改只在 Session 開啟的期間之內。一旦關閉 Session，建立的變數就會銷毀，修改過的變數在下次開啟時，也會還原回預設值。

使用 PowerShell 的 Profile 載入機制，可以在 Session 建立時，自動將固定需要使用的變數建立或修改。

```PowerShell
# 例如將變數 var 的初始化加入倒 Profile 的 CurrentUserAllHosts 路徑
PS > Add-Content -Path $Profile.CurrentUserAllHosts -Value '$var = "example"'
```