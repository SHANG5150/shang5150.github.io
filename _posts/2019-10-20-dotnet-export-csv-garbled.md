---
layout: post
title: 說明解決 Excel 讀取用 .NET 輸出的 CSV 檔案出現亂碼的狀況
---

在 Windows 作業系統上，Excel 開啟 CSV 檔案，會先判斷檔首是否有 UTF-8 的 BOM 記號來判斷是否使用 UTF-8 來解碼文件內容。如果檔首沒有 UTF-8 BOM 記號，則改用當前 Windows 作業系統的 ANSI 來解碼。所以 CSV 檔案用 UTF-8 編碼儲存，但沒有加上用來標記 UTF-8 編碼文件的 BOM 記號，就會造成 Excel 開啟 UTF-8 編碼的 CSV 文件顯示亂碼。

<!--more-->

### 使用 System.IO.StreamWriter 物件輸出 CSV 檔案

建立 ```StreamWriter``` 物件時，如果沒有在 ```constructor``` 上指定 encoding，內部實作預設會使用 internal 的 UTF8NoBOM 做為 encoding 選項。UTF8NoBOM 的 encoding 不適合建立能讓 Excel 辨識的 UTF-8 編碼內容。所以需要明確指定帶有 BOM 記號的 UTF-8 encoding。

```cs
Encoding encoding = Encoding.UTF8;
// 或是
Encoding encoding = new UTF8Encoding(true);

using (var writer = new StreamWriter("Data.csv", false,  encoding))
{
   writer.Write("資料一,資料二,資料三");
}
```

### 使用 System.IO.File.WriteAllText() 方法輸出 CSV 檔案

File.WriteAllText() 方法有兩個 Overloading :

```cs
public static void WriteAllText(string path, string contents);
public static void WriteAllText(string path, string contents, Encoding encoding);
```

未指定 Encoding 的 WriteAllText 方法，內部實作使用了 StreamWriter 來做輸出，並且指定了 StreamWriter.UTF8NoBOM 做為 Encoding 選項，因此不適合拿來做為需要 BOM 記號的 CSV 輸出。需要明確指定帶有 BOM 記號的 UTF-8 Encoding。

```cs
Encoding encoding = Encoding.UTF8;
// 或是
Encoding encoding = new UTF8Encoding(true);

File.WriteAllText("Data.csv", "資料一,資料二,資料三", encoding);
```

### 指定 UTF-8 Encoding 的方法有兩個 :

1. [自己建立 System.Text.UTF8Encoding 物件](#自己建立-systemtextutf8encoding-物件)
2. [使用 System.Text.Encoding.UTF8 Property](#使用-systemtextencodingutf8-property)

#### 自己建立 System.Text.UTF8Encoding 物件

UTF8Encoding 類別有三個 Constructor

```cs
public UTF8Encoding();
public UTF8Encoding(bool encoderShouldEmitUTF8Identifier);
public UTF8Encoding(bool encoderShouldEmitUTF8Identifier, bool  throwOnInvalidBytes);
```

第一個無引數的 UTF8Encoding constructor 內部實作指定了 encoderShouldEmitUTF8Identifier: false ，因此使用無引數建立的 UTF8Encoding 物件不適合用來編碼檔首需要 BOM 記號的 UTF-8 CSV 文件。

使用 UTF8Encoding 物件做為輸出 CSV 文件的解碼，應使用第二或第三個 constructor 並明確指定 encoderShouldEmitUTF8Identifier 為 true，讓編碼時加入 UTF-8 的 BOM 記號。

#### 使用 System.Text.Encoding.UTF8 Property
Encoding.UTF8 Property 內部實作建立了 UTF8Encoding ，並指定 encoderShouldEmitUTF8Identifier: true，因此使用 Encoding.UTF8 Property 做為編碼選項，能夠輸出帶有 BOM 記號的 UTF-8 CSV 文件。