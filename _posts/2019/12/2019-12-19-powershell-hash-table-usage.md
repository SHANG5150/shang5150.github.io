---
layout: post
title: PowerShell 使用 Hash Table
---

筆記 PowerShell 的 Hash Table 使用方法。

<!--more-->

### Hash Table 說明文件

```PowerShell
PS > help about_hash_tables
```

### Hash Table 物件 Type

1. ```System.Collections.Hashtable```
2. ```System.Collections.Specialized.OrderedDictionary```
  在 PowerShell 3.0 加入的 Ordered Dictionary 物件。Ordered Dictionary 的內容順序會依照加入的順序而定。

### 建立 Hash Table

```PowerShell
# 建立 Hashtable 物件
PS > $Month= @{jan = 1; feb = 2; mar = 3}
PS > $Month
Name                           Value
----                           -----
mar                            3
jan                            1
feb                            2

# 建立 Ordered Dictionary 物件
PS > $Month = [ordered]@{jan = 1; feb = 2; mar = 3}
PS > $Month
Name                           Value
----                           -----
jan                            1
feb                            2
mar                            3
```

Note: 在 <key> = <value> 中的 "=" 不是 assign value 的意思，只是用來分開 Key 和 Value 的識別符號。

### 使用 ConvertFrom-StringData 建立 Hashtable 物件

ConvertFrom-StringData 會自動將以 "=" 分隔的字串建立為 Hashtable。

```PowerShell
# 建立測試用的 String data
PS > $MonthString = @"
>> jan = 1
>> feb = 2
>> mar = 3
>> "@

# 使用 ConvertFrom-StringData 把 String-data 轉換成 Hashtable
PS > ConvertFrom-StringData -StringData $MonthString         
Name                           Value
----                           -----
feb                            2
mar                            3
jan                            1
```

### 檢查 Hashtable 的可用物件成員

```PowerShell
# 檢查 Hashtable 物件成員
PS> @{} | Get-Member

# 檢查 Ordered Dictionary 物件成員
PS > [ordered]@{} | Get-Member
```

Note: Hash Table 的 Key 和 Value 都是 .NET Object，一般是 string 或 interger 型別，但也可以是任意 Object Type。

### 使用 Hash Table

```PowerShell
# 建立範本
PS > $Month = [ordered]@{jan = 1; feb = 2; mar = 3}

# 列出所有 Keys
PS > $Month.Keys
jan
feb
mar

# 列出所有 Values
PS > $Month.Values
1
2
3

# 使用 Key 當作 Property 來取得 Value
PS > $Month.jan
1

# 使用索引子 (Indexer) 取得 Value
PS > $Month["jan"]
1

# 加入新資料到 Hash Table
$hash = @{}
$hash["<key>"] = "<value>"
$hash.Add("Key","Value")    # 會將 key-value 加入 hash 變數
$hash = $hash + @{Time="Now"}

# 使用 "+" 不會改變現有的 hash 變數，會產生新的 hash table

# 從 Hash Table 移除資料
$hash.Remove("Key")
```

### 使用 Hash Table 自訂 Select-Object 顯示欄位

```PowerShell
# Get-Process 列出的表格
Get-Process | Select-Object -First 3
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    565      43    38996        368       0.20  19664   1 3DViewer
    225      21     3192       1576       0.23  30920   1 adb
    239      14     2668       5140       0.23   4440   0 AGMService

# 使用自訂格式列出 VM  Size 並以單位 MB 顯示
Get-Process | Select-Object -First 3 `
>> -Property Handle, @{name = "VM(MB)"; expression = {$_.VM / 1MB}}, ProcessName
Handles        VM(MB) ProcessName
-------        ------ -----------
    565 4939.41796875 3DViewer
    225   91.31640625 adb
    239    63.2265625 AGMService
```