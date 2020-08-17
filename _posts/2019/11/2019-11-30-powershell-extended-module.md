---
layout: post
title: 使用 PowerShell 的擴充 Module
---

筆記 PowerShell 安裝、更新和移除 Module 的相關管理方法。

<!--more-->

### 管理 Module 的相關指令

```PowerShell
# Module
Alias   Command
gmo  -> Get-Module
        Get-InstalledModule
inmo -> Install-Module
upmo -> Update-Module
        Uninstall-Module

# Repository
Alias    Command
         Get-PSRepository
         Set-PSRepository
```

### 確認 Module 相關設定

```PowerShell
# 列出 PowerShell 預設的 Module 安裝路徑
PS > $env:PSModulePath.Split(";")
C:\Users\CurrentUser\Documents\WindowsPowerShell\Modules
C:\Program Files\WindowsPowerShell\Modules
C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules

# 指定安裝在 CurrentUser 的 Module 會安裝在
# C:\Users\CurrentUser\Documents\WindowsPowerShell\Modules

# 指定安裝在 AllUser 的 Module 會安裝在
# C:\Program Files\WindowsPowerShell\Modules

# PowerShell 自帶的 Module 會安裝在
# C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
```

### 找出目前環境安裝了哪些 Module

```PowerShell
# 列出所有可使用的 Module
PS > Get-Module -ListAvailable
   目錄: C:\Program Files\WindowsPowerShell\Modules
ModuleType Version    Name                      ExportedCommands
---------- -------    ----                      ----------------
Script     1.0.1      Microsoft.PowerShell.O... {Get-OperationValidation, Invoke...
Binary     1.0.0.1    PackageManagement         {Find-Package, Get-Package, Get-...
Script     3.4.0      Pester                    {Describe, Context, It, Should...}
......(略)

   目錄: C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
ModuleType Version    Name                 ExportedCommands
---------- -------    ----                 ----------------
Manifest   1.0.0.0    AppBackgroundTask    {Disable-AppBackgroundTaskDiagnosticL...
Manifest   2.0.0.0    AppLocker            {Get-AppLockerFileInformation, Get-Ap...
Manifest   1.0.0.0    AppvClient           {Add-AppvClientConnectionGroup, Add-A...
......(略)
```

### 找出已註冊的 Module Repository

```PowerShell
PS > Get-PSRepository
Name         InstallationPolicy   SourceLocation
----         ------------------   --------------
PSGallery    Untrusted            https://www.powershellgallery.com/api/v2
```

### 搜尋 Repository 中的可用模組

```PowerShell
# Find-Module 搭配萬用字元，可以使用較模糊的方式尋找 Module
# 例如，想找出與 Text 處理相關的 Module
PS > Find-Module -Name *text*
Version    Name             Repository    Description
-------    ----             ----------    -----------
0.1.8      ClipboardText    PSGallery     Support for text-based clipboard...
1.0.6      ColoredText      PSGallery     The cutting edge API for text co...
1.0.1      LoremText        PSGallery     Generate random text similar to ...
......(略)

# 不使用萬用字元，就需要輸入一致的名稱才能正確找到 Module
PS > Find-Module -Name Text2Image
Version    Name          Repository    Description
-------    ----          ----------    -----------
1.0.2      Text2Image    PSGallery     Convert Text to Image. Redirect ...

# Find-Module 預設會列出 Module 的最新版本，使用 -AllVersions 可列出全部版本
PS > Find-Module -Name Text2Image -AllVersions
Version    Name          Repository    Description
-------    ----          ----------    -----------
1.0.2      Text2Image    PSGallery     Convert Text to Image. Redirect ...
1.0.1      Text2Image    PSGallery     Convert Text to Image. Redirect ...
```

### 安裝 Module

```PowerShell
# 以安裝 Text2Image 為例
PS > Install-Module -Name Text2Image

未受信任的存放庫
您正在安裝來自未受信任之存放庫的模組。若信任此存放庫，請透過執行
Set-PSRepository 來變更其 InstallationPolicy 值。確定要安裝來自 'PSGallery'
的模組?
[Y] 是(Y)  [A] 全部皆是(A)  [N] 否(N)  [L] 全部皆否(L)  [S] 暫停(S)  [?] 說明
(預設值為 "N"):Y
```

PSGallery 這個 Repository 預設的 InstallationPolicy 是 Untrusted，所以會出現確認安裝的提示。

進行無人職守須安裝時，可在安裝前提前調整 Repository 的 InstallationPolicy，以避免出現安裝詢問。

```PowerShell
PS > Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
PS > Install-Module -Name Text2Image
# 現在不會再出現安裝確認的提示

# 安裝完畢後，可以在把 InstallationPolicy 設回  Untrusted
PS > Set-PSRepository -Name PSGallery -InstallationPolicy Untrusted
```

Install-Module 的幾個選項 :

```PowerShell
-Repository         # 指定 Module 要從哪個 Repository 安裝
-RequiredVersion    # 指定要安裝的 Module 版本，預設是最新版本
-Scope              # 指定要安裝的範圍，預設是 AllUsers

# Scope 使用預設的 AllUser 會需要系統管理員權限
# 如果安裝時指定 -Scope CurrentUser 則不需要系統管理員權限
```

### 確認已安裝模組

命令 Get-Module 的 -ListAvailable 參數可以列出所有可用 Module，如果只想列出自行安裝的 Module 可以改用命令 Get-InstalledModule。

```PowerShell
PS > Get-InstalledModule
Version Name       Repository Description
------- ----       ---------- -----------
1.0.2   Text2Image PSGallery  Convert Text to Image. Redirect your output in...

# 無參數的 Get-InstalledModuel 只會列出最新版本的 Module
# 使用 -AllVersion 參數可以列出相同 Module 的所有版本
PS > Get-InstalledModule -Name Text2Image -AllVersion
Version Name       Repository Description
------- ----       ---------- -----------
1.0.1   Text2Image PSGallery  Convert Text to Image. Redirect your output in...
1.0.2   Text2Image PSGallery  Convert Text to Image. Redirect your output in...

# -AllVersions 只能對應單一 Module 名稱進行運作
# 如果想列出所有 Module 的所有版本時，可以使用 Foreach-Object 組合
PS > Get-InstalledModule |
>> ForEach-Object {Get-InstalledModule -Name $_.Name -AllVersions}
Version Name               Repository Description
------- ----               ---------- -----------
1.0.14  SpeculationControl PSGallery  This module provides the ability to qu...
1.0.1   Text2Image         PSGallery  Convert Text to Image. Redirect your o...
1.0.2   Text2Image         PSGallery  Convert Text to Image. Redirect your o...
```

### 確認模組提供的功能

安裝完模組之後，可以使用 Get-Command 的 -Module 參數列出模組提供的指令。

```PowerShell
PS > Get-Command -Module Text2Image
CommandType Name      Version Source
----------- ----      ------- ------
Function    New-Image 1.0.2   Text2Image

# 找到可用命令之後，再以 Help 調查看看命令，大致上就能了解使用方法了
PS > help New-Image
```

### 更新模組

```PowerShell
# 如果已安裝的 Module 在 Repository 上有新版本，可以使用 Update-Module 進行更新
PS > Update-Module

# Update-Module 預設不會輸出有進行更新的 Module 項目
# 如果想在更新前確認有哪些 Module 會進行更新，可簡易使用 -Whatif 參數
PS > Update-Module -Whatif
WhatIf: 正在目標 "模組 'Text2Image' 的版本 '1.0.1'，正在更新成版本 '1.0.2'" 上執
行 "Update-Module" 操作。

# 或者拆出已安裝的 Module 版本與 Repository 的版本做比較
PS > Get-InstalledModule |
>> Where-Object {$_.Version -lt (Find-Module $_.Name).Version}
Version Name       Repository Description
------- ----       ---------- -----------
1.0.1   Text2Image PSGallery  Convert Text to Image. Redirect your output in...
```

# 反安裝 Module

```PowerShell
# 移除 Text2Image Module
PS > Uninstall-Module -Name Text2Image

# 如果安裝了多個版本的相同 Module， Uninstall-Module 預設只會移除最新版本的那一個
# 使用 -AllVersions 參數移除指定 Module 的所有版本
PS > Uninstall-Module -Name Text2Image -AllVersions

# 如果只是想移除較舊版本的 Module 可以使用 -RequiredVersion 指定要移除的版本
# 確認 Text2Image Module 的所有版本
PS > Get-InstalledModule -Name Text2Image -AllVersions
Version Name       Repository Description
------- ----       ---------- -----------
1.0.1   Text2Image PSGallery  Convert Text to Image. Redirect your output ...
1.0.2   Text2Image PSGallery  Convert Text to Image. Redirect your output ...

# 指定移除 Text2Image 的 1.0.1 版本
PS > Uninstall-Module -Name Text2Image -RequiredVersion 1.0.1

# 確認 Text2Image Module 的 1.0.1 版本已被移除
PS > Get-InstalledModule -Name Text2Image -AllVersions
Version Name       Repository Description
------- ----       ---------- -----------
1.0.2   Text2Image PSGallery  Convert Text to Image. Redirect your output ...
```

經過多次 Update-Module 後，可能存在許多舊版本的 Module。批次移除舊版本 Module 需要比較長一些的 Script 幫助。

```PowerShell
PS > Get-InstalledModule |
>> ForEach-Object {
>> $CurrentVersion = $PSItem.Version
>> Get-InstalledModule -Name $PSItem.Name -AllVersions |
>> Where-Object -Property Version -LT $CurrentVersion
>> } | Uninstall-Module

# 上方 Script 流程說明 :
# 1. 找出所有已安裝的最新版本 Module
# 2. 轉交給 Foreach-Object
# 3. 找出每個已安裝 Module 的所有版本 Module
# 4. 利用 Where-Object 過濾出版本小於最新版本的 Module
# 5. 把過濾出的 Module 全部轉交給 Uninstall-Module
# 6. 由 Uninstall-Module 移除舊版本的 Module
```