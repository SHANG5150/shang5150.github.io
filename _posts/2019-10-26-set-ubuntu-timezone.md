---
layout: post
title: 設定 Ubuntu 時區
---

在 Raspberry Pi 安裝完 Ubuntu 19.10 後的默認時區顯示為 UTC 。如果各別 user 沒有另外設定自己的時區，就會套用系統默認時區。重新設定時區以便 rsyslog 在輸出 log 時，能為 Timestamp 套用自己熟悉的時區。

<!--more-->

## 為什麼需要重新設定時區?

在 Raspberry Pi 上安裝 Ubuntu 官方提供的作業系統，將已經設定好的 Image 直接抄寫到記憶卡上，略過了使用 ISO 或可開機 USB 隨身碟進行安裝時的時區設定步驟。所以第一次登入進去時，默認值就直接使用 UTC 了。

## 設定系統默認時區

設定之前先使用命令 timedatectl 確認目前的時區

```
$ timedatectl
              Local time: Fri 2019-10-25 07:37:29 UTC
          Universal time: Fri 2019-10-25 07:37:29 UTC
                RTC time: n/a
               Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
             NTP service: active
         RTC in local TZ: no
```

接著使用命令 ```dpkg-reconfigure tzdata``` ，在互動模式下選擇 Asia/Taipei

```
$ sudo dpkg-reconfigure tzdata

Current default time zone: 'Asia/Taipei'
Local time is now:      Fri Oct 25 15:38:53 CST 2019.
Universal Time is now:  Fri Oct 25 07:38:53 UTC 2019.
```

可以看到 Time zone 變更為 'Asia/Taipei'，並且 Local time 也改成使用 CST 顯示。

不過一些已經被啟動的 service 還沒有套用這項變更，仍然會以 UTC 時間顯示。

```
$ date
Fri Oct 25 15:49:04 CST 2019    # date 的輸出結果已變更
$ logger test                   # 丟一筆測試 log 到 syslog 底下
$ tail -1 /var/log/syslog       # 擷取上面丟的 log
Oct 25 07:44:27 respberrypi shang: test    # log 的 timestamp 仍為 UTC 時間
```

這時候要重新啟動 rsyslog 這個 service，讓它重新載入 timestamp 設定。

```
$ sudo service rsyslog restart    # 重啟 rsyslog
$ logger test again               # 再丟一筆測試 log 到 syslog 底下
$ tail -1 /var/log/syslog         # 擷取最後丟的 log
Oct 25 15:53:49 respberrypi shang: test again
$ date                            # 確認 log 的 timestamp 的顯示時間是否與 date 一致
Fri Oct 25 15:54:23 CST 2019
```

因為不知道還有多少執行中的 service 會有類似的狀況，最好還是在變更過影響整個系統的設定之後重新啟動作業系統。

```
$ sudo reboot
```

## 設定各別 user 的時區

依據 POSIX 環境變數定義，TZ 環境變數代表 Timezone 資訊，並且會被許多與時間相關的函式使用。

但在作業系統實際環境下，在 TZ 環境變數沒有被指定的情況下也能顯示正確的系統默認時區。

```
$ echo $TZ
                                # 空的， TZ 變數沒有任何資料
$ date
Fri Oct 25 17:48:36 CST 2019     # 時間顯示仍然有時區資料
```

因為依據 GNU C Library 的文件說明，當 TZ 環境變數變數沒有被設定的時候，GNU C Library 的預設時區就會去查找 ```/etc/localtime``` 或 ```/usr/local/etc/localtime``` 的時區設定值。如果簡單的從 GNU 的 C Library 文件來看，當管理者與使用者都是使用相同時區的情況下，似乎沒有設置 TZ 變數的必要。

不過這也僅限在所有使用的程式在處理時區相關問題時，都以 GNU C Library 來實作才能保證程式行為是一致的。所以還是把 TZ 環境變數設定完整，避免其它實作行為與 GNU C 不一樣的 Library 造成問題。

開始設定 TZ 環境變數之前，可以先使用命令 ```tzselect``` 來查詢想要設定的時區，以及 TZ 環境變數的設定方法。

```
$ tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
...
3) Antarctica
4) Asia
5) Atlantic Ocean
...
#? 4        # 選擇 4) Asia
Please select a country whose clocks agree with yours.
...
3) Azerbaijan    16) Iran      29) Malaysia           42) Syria
4) Bahrain       17) Iraq      30) Mongolia           43) Taiwan
5) Bangladesh    18) Israel    31) Myanmar (Burma)    44) Tajikistan
...
#? 43        # 選擇 43) Taiwan
The following information has been given:
       Taiwan
Therefore TZ='Asia/Taipei' will be used.
Selected time is now:   Fri Oct 25 18:00:49 CST 2019.
Universal Time is now:  Fri Oct 25 10:00:49 UTC 2019.
Is the above information OK?
1) Yes
2) No
#? 1        # 選擇 1) Yes 離開 tzselect 程式
You can make this change permanent for yourself by appending the line
       TZ='Asia/Taipei'; export TZ
to the file '.profile' in your home directory; then log out and log in again.
```

依據 ```tzselect``` 程式可以查詢到 TZ 環境變數的設定方法。

```
$ TZ='Asia/Taipei'; export TZ
```

並且也提示了可以將上面的設定寫入 user 的 .profile ，讓之後每次登入都自動載入。

```
$ echo "TZ='Asia/Taipei'; export TZ" >> ~/.profile
```

一旦設定好 user 的 TZ 環境變數，在這之後啟動的程式都可以直接取用 TZ 變數指定的 Timezone ，而不是系統默認值。