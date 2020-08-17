---
layout: post
title: 資料輸出重導向
---

簡短紀錄 PowerShell 資料重導向使用方法和範例組合組合。

<!--more-->

PowerShell 預設最後會把資料全部寫到 Host programe，例如 Console 或 PowerShell SE。而重導向功能提供方法將資料寫到檔案裡保存起來。

### 重導向測試資料

先定義用來測試重導向的資料 :

```PowerShell
PS > function Write-Message{
>> Write-Output "Output message."      # 寫入資料到 Output stream
>> Write-Warning "Warning message."    # 寫入資料到 Warning stream
>> Write-Error "Error message."        # 寫入資料到 Error stream
>> }
```

### 重導向方法

達成重導向的幾個方法 :

```PowerShell
PS > Write-Message | Out-File message.txt    # 使用 Out-* 系列指令處理
PS > Write-Message > message.txt             # 使用重導向運算子處理
PS > $var = Write-Message                    # 使用變數接收 Output stream 資料
PS > Write-Message | Tee-Object              # 使用 Tee 分叉處理
```

### 重導向 Stream 代號與符號

```PowerShell
# Output Stream 代號
   *    All output
   1    Success output
   2    Errors
   3    Warning messages
   4    Verbose output
   5    Debug messages
   6    Informational messages

# Redirection Operators
   n>    把指定的 Stream 重導向到檔案
   n>>    把指定的 Stream 附加在指定檔案尾端
   n>&1    把指定的 Stream 重導向到 output stream
```

### 輸出重導向範例

```PowerShell
# 建立測試用 Write-Message function
PS > function Write-Message{
>> Write-Output "Output message."
>> Write-Warning "Warning message."
>> Write-Error "Error message."
>> }

PS > Write-Message > result               # Output 寫到檔案
PS > Write-Message 2> result              # Error 寫到檔案
PS > Write-Message 2>&1 > result          # Error & Output 寫到檔案
PS > Write-Message 2>&1 3>&1 > result     # Error, Warning & Outpu 寫到檔案
PS > Write-Message *> result              # 全部寫到檔案
PS > Write-Message 1>$null *> result      # 寫 Output 以外的 Stream 到檔案
PS > Write-Message | Tee-Object result    # Output 寫到檔案，同時也顯示在 Host program
```