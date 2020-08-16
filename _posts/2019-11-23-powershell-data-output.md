---
layout: post
title: PowerShell 資料輸出
---

PowerShell 預設會把物件經過格式化後，輸出到 Console 或 PowerShell ISE 等 Host programe 讓使用者查看執行結果。這裡紀錄輸出流程和方法。

### 命令輸出流程

PowerShell 命令的輸出流程如下 :

1. 主要的 Command 執行完畢，把輸出物件放入 Pipeline。
2. 在 Pipeline 最後由 Out-Default 來接收物件。
3. 在 Console 環境下，預設 Out-Defualt 會把物件傳送給 Out-Host。
4. Out-Host 收到物件後，將物件交給格式化系統處理。
5. 格式化系統依照格式化規則產生格式化指令，再傳送給 Out-Host。
6. Out-Host 按照格式化指令輸出內容。

Out-Default 不會做物件格式化或輸出，只是個 Placeholder。其目的是用來給客製化的 Out-Defualt function 或 cmdlet 使用。參照 help Out-Default。

以 Out 動詞開頭的命令不會格式化物件，只會依照格式化指令 Render Object 並送到特定的目的地。參照 help Out-Host。

### 查詢可用的輸出指令

```PowerShell
# 輸出指令多以 Out 或 Export 開頭
PS > Get-Command Out*

# 查詢格式化和格式化輸出指令文件
PS > help Out*
```

### 輸出指令

```PowerShell
Out-Host         # 預設的輸出選項
Out-Null         # 把 Output stream 的物件全部刪除
Out-File         # 把 Output stream 的物件寫入到指定檔案
Out-GridView     # 把 Output stream 的物件傳入 Grid GUI 顯示
```

#### Out-Host

把指令的 Output 送到 PowerShell Host 顯示。例如 Console, PowerShell ISE 或其它 Host program。

#### Out-Null

```PowerShell
# Out-Null 會把 Output stream 的物件資料直接刪除
PS > &{
>> Write-Output "Output Message"      # 寫入 Output stream
>> Write-Warning "Warning Message"    # 寫入 Warning stream
>> } | Out-Null
警告: Warning Message                 # 只剩 Warning 訊息
PS >
```

#### Out-File

```PowerShell
# 把 Output stream 的物件寫入指定的文字檔案中
PS > Get-Process | Out-File C:/proc.txt

# 把傳入的 Output stream 內容接續(append) 在指定的文字檔案下方
PS > Get-Process | Out-File -Append C:/proc.txt
```

#### Out-GridView

```PowerShell
# 把 Get-Process 的物件傳入 GridView GUI 顯示
PS > Get-Process | Out-GridView
```

![powershell-gridview-example](/assets/img/post/2019/11/23/powershell-data-output/powershell-gridview-example.png)

### 格式化輸出指令

除了 Out 開頭的指令使用預設格式輸出以外，當使用者需要以其他方式蒐集指令執行結果，或是做更進一步的處理時，就需要使用其它輸出方法。

```PowerShell
Export-Csv       # 把物件格式化成 Csv 後寫入到指定檔案
Export-Clixml    # 把物件格式化成 Clixml 後寫入到指定檔案
```

#### Export-Csv

```PowerShell
# 把 Service 物件匯出成 csv 檔案
PS > Get-Service | Export-Csv serv.csv

# 匯入 Service 的 csv 檔案
PS > $var = Import-Csv serv.csv
PS > $var | Select-Object -First 3
Name                : AarSvc_51910
RequiredServices    : System.ServiceProcess.ServiceController[]
CanPauseAndContinue : False
......(略)

Name                : AdobeARMservice
RequiredServices    : System.ServiceProcess.ServiceController[]
CanPauseAndContinue : False
......(略)

Name                : AGMService
RequiredServices    : System.ServiceProcess.ServiceController[]
CanPauseAndContinue : False
......(略)
```

#### Export-Clixml

```PowerShell
# 把 Service 物件匯出成 xml 檔案
PS > Get-Service | Export-Clixml serv.xml

# 匯入 Service 的 xml 檔案
PS > $var = Import-Clixml serv.xml
PS > $var | Select-Object -First 3
Status   Name               DisplayName
------   ----               -----------
Stopped  AarSvc_51910       AarSvc_51910
Running  AdobeARMservice    Adobe Acrobat Update Service
Running  AGMService         Adobe Genuine Monitor Service
```

### 使用重導向運算子輸出

除了明確使用輸出命令以外，也可以使用重導向運算子，直接將物件儲存到指定檔案。

```PowerShell
PS > "Message" > result
PS > Get-Content result
Message
```

重導向運算子使用 Unicode encoding。需要指定其它 Encoding 時，可以用 Out-File 代替重導向運算子。
