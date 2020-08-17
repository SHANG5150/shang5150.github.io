---
layout: post
title: 使用 PowerShell Profile
---

PowerShell 的 Profile 檔案提供開啟 PowerShell 時，自動載入一些常用設定或自定義函式的功能。隨著使用 PowerShell 的頻率增加，可以將一些較冗長的 Script 紀錄在 Profile 之內，增加作業效率。

<!--more-->

### PowerShell Profile 的位置
PowerShell 的變數 ```$PROFILE```，包含了各個優先層級的 Profile 檔案位置。

```PowerShell
# 使用 Select-Object 列出全部 Profile 檔案位置
PS > $PROFILE |Select-Object -Property *                                       
AllUsersAllHosts       : C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1
AllUsersCurrentHost    : C:\Windows\System32\WindowsPowerShell\v1.0\Microsoft.P
                         owerShell_profile.ps1
CurrentUserAllHosts    : C:\Users\User\Documents\WindowsPowerShell\profile.ps1
CurrentUserCurrentHost : C:\Users\User\Documents\WindowsPowerShell\Microsoft.Po
                         werShell_profile.ps1

# 各別列出 Profile 檔案位置
PS > $PROFILE.AllUsersAllHosts
C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1
PS > $PROFILE.AllUsersCurrentHost
C:\Windows\System32\WindowsPowerShell\v1.0\Microsoft.PowerShell_profile.ps1
PS > $PROFILE.CurrentUserAllHosts
C:\Users\User\Documents\WindowsPowerShell\profile.ps1
PS > $PROFILE.CurrentUserCurrentHost
C:\Users\User\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1

# 直接使用 $PROFILE，預設會回傳 CurrentUserCurrentHost 的檔案位置
PS > $PROFILE
C:\Users\User\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```

### Profile 檔案的載入順序

PowerShell 載入 Profile 時，如果對應位置的檔案不存在，會自動略過。

載入順序 :

1. AllUsersAllHosts
2. AllUsersCurrentHost
3. CurrentUserAllHosts
4. CurrentUserCurrentHost

### PowerShell Profile 的 Host 差異

PowerShell 依據執行的使用者身份和 Host program 不同，而載入不同路徑底下的 Profile 檔案。

例如在 Console Host 執行 PowerShell 可以取得以下 Profile 路徑

```PowerShell
PS > $PORFILE | Select-Object -Property *
AllUsersAllHosts       : C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1
AllUsersCurrentHost    : C:\Windows\System32\WindowsPowerShell\v1.0\Microsoft.P
                         owerShell_profile.ps1
CurrentUserAllHosts    : C:\Users\User\Documents\WindowsPowerShell\profile.ps1
CurrentUserCurrentHost : C:\Users\User\Documents\WindowsPowerShell\Microsoft.Po
                         werShell_profile.ps1
```

而在  Windows PowerShell ISE Host 可以取得以下 Profile 路徑

```PowerShell
PS > $PORFILE | Select-Object -Property *
AllUsersAllHosts       : C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1
AllUsersCurrentHost    : C:\Windows\System32\WindowsPowerShell\v1.0\Microsoft.P
                         owerShellISE_profile.ps1
CurrentUserAllHosts    : C:\Users\User\Documents\WindowsPowerShell\profile.ps1
CurrentUserCurrentHost : C:\Users\User\Documents\WindowsPowerShell\Microsoft.Po
                         werShellISE_profile.ps1
```

留意 CurrentHost 的兩個 Profile 差異。ConsoleHost 會載入 ```Microsoft.PowerShell_profile.ps1```，而 PowerShell ISE Host 會載入 ```Microsoft.PowerShellISE_profile.ps1```。

### Profile Script Execution Policy 設定

PowerShell 的 Script Execution Policy 預設是 Restricted。表示禁止執行所有的 Script。而 Profile 也是一種 Script，所以為了啟用 Profile 功能，需要先把 Script Execution Policy 設定放行。

```PowerShell
PS > Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
執行原則變更
執行原則有助於防範您不信任的指令碼。如果變更執行原則，可能會使您接觸到
about_Execution_Policies 說明主題 (網址為
https:/go.microsoft.com/fwlink/?LinkID=135170)
中所述的安全性風險。您要變更執行原則嗎?
[Y] 是(Y)  [A] 全部皆是(A)  [N] 否(N)  [L] 全部皆否(L)  [S] 暫停(S)  [?] 說明
(預設值為 "N"):Y

# 輸入 Y 確認更改為 RemoteSigned 規則即可
```

### 建立 Profile 檔案

在 ```$PROFILE``` 變數中記錄的四個檔案位置，預設是沒有檔案的。

```PowerShell
# 在建立 Profile 檔案之前，可以先測試檔案是否存在，以免覆蓋掉舊有的內容
PS > Test-Path -Path $PROFILE.CurrentUserAllHosts
False

# 建立空的 Profile 檔案
PS > New-Item -Type File -Path $PROFILE.CurrentUserAllHosts
    目錄: C:\Users\User\Documents\WindowsPowerShell
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----     2019/11/28  下午 04:17              0 profile.ps1

# 編輯 Profile 檔案
PS > notepad $PROFILE.CurrentUserAllHosts

# 或是使用 PowerShell ISE 來編輯
PS > powershell_ise.exe $PROFILE.CurrentUserAllHosts
```

### 載入 Profile

編輯完 Profile 之後，重新啟動 PowerShell 即可載入 Profile 內容。

### 開啟 PowerShell 但不載入 Profile

如果已經寫好了 Profile，但有特殊狀況，需要開啟沒有經過 Profile 修改的 PowerShell 環境，可以使用以下幾種方法。

* 在 Host program 開啟新的 PowerShell
  ```PowerShell
  # 使用 -noprefile 選項開啟新 PowerShell
  PS > PowerShell.exe -noprefile
  ```

* 使用 Windows 執行對話視窗開啟 PowerShell
  1. [win] + [R] 開啟 windows 的執行對話視窗
  2. 輸入 ```PowerShell -noprofile```
  3. 點選確定，開啟 PowerShell

### Profile 範例

```PowerShell
# PowerShell profile for CurrentUserAllHost.

# Edit PowerShell profile by PowerShell_ISE
function Edit-Profile
{
    powershell_ise.exe $profile.CurrentUserAllHosts
}

#Shot-cut for Edit-Profile
Set-Alias -Name epf -Value Edit-Profile

# Use -ShowWindow option to open help documents by default.
function Get-HelpWindow ($CommandName)
{
    help -Name $CommandName -ShowWindow
}

# Shot-cut for Get-HelpWindow
Set-Alias -Name ghw -Value Get-HelpWindow

Write-Output "PowerShell Profile has been loaded."  
```

### 參考說明文件

```PowerShell
PS >help about_profiles
```