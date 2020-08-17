---
layout: post
title: PowerShell 資料格式化
---

筆記 PowerShell 的自動資料格式化功能。記錄格式化文件分類、自動格式化流程和幾個常用的格式化指令。

<!--more-->

### 物件資料自動格式化

PowerShell command 輸出的 Object，會在 Pipeline 的最末端傳遞給 ```Out-Default```。預設 ```Out-Default``` 會將物件轉交給 ```Out-Host```。而 ```Out-Host``` 會將 Object 轉入格式化流程，接著再將格式化後的資料送到 Host programe 進行顯示。

Note: 在 PowerShell 中，以 Out 開頭的指令都不具備格式化功能，也都會觸發自動格式化流程。參考 ```Help Out-Host``` 的備註說明。

### 預設格式化規則參照文件

PowerShell 預設在安裝目錄 (```$PSHOME```) 底下提供了兩種類型的 ```.ps1.xml``` 文件。

* format.ps1xml 文件
  命名中含有 format 的 ps1.xml 文件，提供物件的格式化規則。
  內容包含較精確的格式化資訊，提供以下四種格式的定義。
  <TableControl>(表格), <ListControl>(清單), <WideControl>(橫向清單), 和 <CustomControl>(其它自定義)
  例如 : DotNetTypes.format.ps1xml, Diagnostics.Format.ps1xml 等。
* type.ps1xml 文件
  命名中含有 type 的 ps1xml 文件紀錄 PowerShell 給物件的擴充 Member 規則。
  當格式化系統找不到 format 對應規則時，就會查看是否有 type 擴充規則可拿來當作格式化對象。
  例如 : types.ps1xml, getevent.types.ps1xml 等。

### 格式化流程

1. 取出傳入物件的類別。
2. 查詢物件類別是否有對應的格式化規則。(*.Format.ps1xml 紀錄的那些)
3. 如果沒有，再查詢是否有對應的物件擴充成員規則。(*.types.ps1xml 那些)
4. 如果沒有，就使用 Object 的全部屬性。
5. 如果使用了擴充成員規則或物件全部屬性，那麼屬性數量少於等於四種，會使用表格顯示。屬性數量大於四種就以清單顯示。

### 查找預設格式化規則

PowerShell 預設在安裝目錄 (```$PSHOME```) 底下提供了許多的預設格式化規則文件。如果想查詢命令輸出的物件是否有提供預設格式。可以用下面的方法查詢。

```PowerShell
# 以 Get-Service 為例，先查詢 Get-Service 的 Output 物件類別
PS > Get-Service | Get-Member    
  TypeName: System.ServiceProcess.ServiceController
......(略)

# 查找 .ps1xml 文件內是否有物件類別的定義
PS > Select-String 'System.ServiceProcess.ServiceController' $PSHOME/*.ps1xml |
>> Select-Object -Property Filename -Unique
Filename
--------
DotNetTypes.format.ps1xml    # 找到格式化專用規則
types.ps1xml                 # 也找到擴充成員規則
```

### 資料格式化指令

除了讓 PowerShell 選擇預定義的格式化流程以外，也可以自己指定要使用那一種格式輸出內容。送入 Format 命令的物件，在沒有其它參數的情況下，一樣會去找 Format.ps1xml 的格式化規則。

### 查詢格式化指令

```PowerShell
# 查詢可用的格式化指令
PS > Get-Command Format-*

# 查詢格式化指令文件
PS > help Format-*
```

### Format 指令

```PowerShell
Alias     Command
ft     -> Format-Table     # 以表格方式顯示物件資訊
fl     -> Format-List      # 以清單方式顯示物件資訊
fw     -> Format-Wide      # 以橫式清單的方式一次顯示單一物件的 Property
select -> Select-Object    # 使用 Select-Object 過濾 Property 也會觸發格式化流程
```

#### Format-Table

```PowerShell
# Format-Table 範例
PS > Get-Process | Select-Object -First 1 | Format-Table
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    239      18     2560      11444       0.08   4836   0 AGMService

# 自動調整欄位寬度
PS > Get-Process | Select-Object -First 1 | Format-Table -AutoSize
Handles NPM(K) PM(K) WS(K) CPU(s)   Id SI ProcessName
------- ------ ----- ----- ------   -- -- -----------
    239     18  2560 11444   0.08 4836  0 AGMService

# 使用 Property 參數指定欄位
PS > Get-Process | Select-Object -First 1 |
>> Format-Table -Property ProcessName, Id, Handles
ProcessName   Id Handles
-----------   -- -------
AGMService  4836     239

# 其它參數
-Wrap                              # 展開因為欄位資料過長而被截斷的內容
-HideTableHeaders    # 隱藏 Header 列
```

#### Format-List

```PowerShell
# Format-List 範例
PS C:\> Get-Process | Select-Object -First 2 | Format-List

Id      : 5204
Handles : 239
CPU     : 0.0625
SI      : 0
Name    : AGMService

Id      : 5176
Handles : 349
CPU     : 1.3125
SI      : 0
Name    : AGSService

# 因為沒有在 *.Format.ps1xml 中找到對應於 List 的格式化規則
# 所以 Format-List 此時列出的是 types.ps1xml 擴充成員的欄位
```

#### Format-Wide

```PowerShell
# Format-Wide 範例
PS C:\> Get-Process | Select-Object -First 10 | Format-Wide -Column 3

AGMService                         AGSService                         AppleMobileDeviceHelper
AppleMobileDeviceService           ApplicationFrameHost               armsvc
audiodg                            avp                                avpui
Calculator
```

### Select-Object

```Select-Object``` 的物件送到 ```Out-Host``` 時，也會觸發格式化流程。

```PowerShell
# Select-Object 對照組
PS > Get-Process | Select-Object -First 1
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    239      18     2628      11756       0.08   4836   0 AGMService

# Get-Process 輸出的物件是 System.Diagnostic.Process
# 有對應的格式規則在 DotNetTypes.format.ps1xml 文件被定義。
# 因此觸發格式化流程後，會取得最先找到的 Table 格式進行輸出。

# 取出指定的 Property
PS > Get-Process | Select-Object -First 1 -Property Handles, NPM, PM, WS
Handles   NPM      PM       WS
-------   ---      --       --
    239 17984 2621440 12038144

# 指定 Property 後，Select-Object 的輸出物件變成 Selected.System.Diagnostics.Process
# 在 $PSHOME 目錄底下找不到這個 Type 的格式化定義和擴充成員定義
# 所以格式化系統會改為取用所有指定的 Property 進行格式化
# 可以觀察到輸出的 Value 也失去了預先定義的單位轉換格式
# 並且因為欄位數量小於等於 4 個，所以採用 Table 格式輸出

# 取得更多 Property
ps | select -f 1 -Property Handles, NPM, PM, WS, CPU
Handles : 239
NPM     : 17984
PM      : 2621440
WS      : 12001280
CPU     : 0.078125

# 指定的 Property 大於 4 個，所以改用 List 格式輸出
```
