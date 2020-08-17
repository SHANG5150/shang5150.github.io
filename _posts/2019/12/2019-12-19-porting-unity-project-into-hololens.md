---
layout: post
title: 將 Unity 專案 Porting 到 HoloLens 的流程與設定
---

工作上愈到需要把 Unity 上製作給 Vive Focus 使用的的 Prototype 移植到 HoloLens 進行 Demo 的需求。以下紀錄 HoloLens 的設定和移植遇到的問題。

<!--more-->

### HoloLens 硬體操作

* 開機 : 按住機器左後方末端的電源按鈕一秒。
* 關機 : 按住電源四秒。
* 三分鐘沒有使用會進入待機，單按一下電源鍵可喚醒。
* 進入待機四小時，或電源低於 10 % 會自動關機。
* 戴上 HoloLens 時，左方頭帶按鈕調整亮度，右方頭帶按鈕調整音量大小。

### HoloLens 主機設定

照著開機的設定步驟走，基本上沒有特別問題需要處理。

但是在第一次使用需要登入 Microsoft 帳號。因為在測試的這段時間 (2019/12) HoloLens 只支援單一使用者登入，而沒有登出帳號的功能。因此後續開發如果需要將裝置轉手給下一個人使用，需要將裝置全部還原。如果有多人之間共用的需求，可能考慮使用企業帳號登入，或是另外建一個開發用的共用帳號比較方便。

### 安裝 Windows 10 使用 Unity 的開發工具

* 安裝 Unity 的 LTS 版本。(我使用 Unity 2018.4)。
* 安裝 Unity 時，一起安裝平台支援模組 [UWP Build Support (IL2CPP)]。
  不要使用 [UWP Build Support (.NET)]，它在設定 Unity Build Setting 已被標記為 Deprecated ，會在未來 Unity 版本移除。
* 安裝 HoloLens Emulator and Holographic Templates。
  https://go.microsoft.com/fwlink/?linkid=852626
* 下載 Mixed Reality Toolkit (MRTK) 的 Unity package，並匯入專案。
  https://github.com/microsoft/MixedRealityToolkit-Unity/releases
* 安裝 Windows SDK
  https://developer.microsoft.com/zh-tw/windows/downloads/windows-10-sdk
  在測試期間 (2019/12) 使用的是 MRTK v2.2.0 需要配合使用 Windows SDK (10.0.18362.0) 版本。否則會遇到 Unity Editor 和 Remote Play 可以正常執行，但 Building 時，拋出某些元件遺失的 Exception。

### HoloLens 開發環境設定

* 在 App Store 下載安裝 Holographic Remoting Player (HRP)。
HRP 提供 Unity Remote Play 的功能，可以避免每次要在機器裡面測試都要打包的麻煩。
* 在 ```Settings/Update & Security/for Developer``` 啟用開發者模式。
允許安裝 App Store 以外的 App ，以便使用 Visual Studio 將 App 安裝到 HoloLens。

### Unity Project 環境設定

* 匯入 MRTK，其中 MRTK 的 Foundation package 為必要 package。其它 Tools, Examples, Extensions 和 Providers.UnityAR 為可選用項目。
* 第一次匯入 MRTK 會出現 MRTK Project Configurator 視窗，提供快速套用 Project Settings 的方法，直接全選套用。
  其中 [Single Pass Instanced Rendering Path] 選項，要看需求而定。手邊用來 Porting 的專案本來在 VIVE Focus 上就是以 Single Pass Rendering 的流程進行開發，在這裡沒有遇到 Rendering 問題。
* ```Project Settings/Play``` 設定
  * ```Other Settings/Scripting Backend``` 設為 IL2CPP
  * XR Settings
      - Virtual Reality Supported 勾選啟用
      - VR SDK 加入 Windows Mixed Reality
      - Depth Format: 16-bit depth (手邊專案的 Camera Clipping 只在 0.85-100 之間)
      - Enable Depth Buffer Sharing 勾選啟用
* ```Project Settings/Quality``` 依官方建議設定
  - UWP 的 Level: Very Low
  - Shadows: Disable Shadows
  - Shadow Casecades: NoCascade

# Porting Scene 設定
* 加入 MixedRealityToolkit 物件。
  功能位置 : ```Unity Menu Item/Mixed Reality Toolkit/Add to Scene and Configure...```
  點選後會在當前開啟的 Scene root 下，加入 MixedRealityToolkit 和 MixedRealityPlayspace 兩個 GameObject。
* 設定 MixedRealityToolkit。
  在 MixedRealityToolkit 物件下，依自訂需求 Clone 出對應的 Profile 設定檔。
  ![mixed-reality-toolkit-component-clone-example](/assets/img/post/2019/12/19/mixed-reality-toolkit-component-clone-example.png)
* 設定自訂 Camera profile。依官方建議將 Depth Format 設定為 16-bit depth，所以連帶將 Near/Far Clip 縮短，以提高 Depth 精度。
  - Near Clip: 0.85
  - Far Clip: 100
  - Clear Flag: Color
  - Background Color: (0,0,0)
  - Quality Setting: Very Low
* 設定自訂 Diagnostics profile。Build Project 時，可以關閉這項功能避免干擾。
  ![diagnostics-profile](/assets/img/post/2019/12/19/diagnostics-profile.png)
  Enable Diagnostics System : False
* 設定所有需要接收互動事件的 UGUI Canvas
  點選 Canvas 上的 [Convert to MRTK Canvas] 按鈕
  ![convert-to-mrtk-canvas](/assets/img/post/2019/12/19/convert-to-mrtk-canvas.png)
  掛上 NearInteractionTouchableUnityUI Component。
  將 [Event to Receive] Property 設為 Pointer
  ![set-event-to-reveive-as-pointer](/assets/img/post/2019/12/19/set-event-to-reveive-as-pointer.png)

### Build Project

* 設定 Build Settings
  - 把要 Building 的 Scene 加入 Scenes In Build 清單。
  - 取消勾選 MRTK Examples 匯入的 Scenes。(如果有匯入的話)
  - Target Device: HoloLens
  - Target SDK Version: Latest installed (注意沒有安裝 Windows SDK 10.0.18362.0 會在 Building 時，出現元件遺失問題)
  - Minimum Platform Version: 10.0.10240.0 (開發環境的 Windows 10 所安裝的最小版本)
* Building 出 Project 的 IL2CPP 專案。

### 將專案佈署到 HoloLens
* 使用 Visual Studio 開啟由 Unity 打包出來的 IL2CPP 專案。
* 設定 HoloLens Remote 位置。
  開啟 Visual Studio Project Property 視窗 (Menu/PROJECT/Properties)
  切換 Configuration 到 Release 並開啟 Machine Name 欄位的彈出選單
  ![set-hololens-remote-location](/assets/img/post/2019/12/19/set-hololens-remote-location.png)
  選取 HoloLens 裝置。
  ![select-hololens-device](/assets/img/post/2019/12/19/select-hololens-device.png)
  在 HoloLens 與開發環境位於相同網域之下時，可以直接在 Auto Detected 裡面找到機器。
  如果機器沒有出現在 Auto Detected 清單下，需要自己在上方的 Manual Configuration 設定。
* 設定 Visual Studio Building Configuration 為 [Release], [x86], [Remote Machine]
  ![set-building-configuration](/assets/img/post/2019/12/19/set-building-configuration.png)
  按下 Remote Machine 按鈕後，會打包專案並自動佈署到前一步設定的 HoloLens 機器。
* 第一次佈署 Visual Studio 會提示需要輸入 Device Pin Code。
  Device Pin Code 從 HoloLens Settings/Update & Security 的 For Developers 選項進入。
  向下尋找 Device Discovery 功能，選取 Pair 後取得 Device Pin Code。
  ![get-device-pin-code](/assets/img/post/2019/12/19/get-device-pin-code.jpg)

### Resources
* [HoloLens Emulator and Holographic Templates](https://go.microsoft.com/fwlink/?linkid=852626)
* [MRTK GitHub Release page](https://github.com/microsoft/MixedRealityToolkit-Unity/releases)
* [Windows 10 SDK](https://developer.microsoft.com/zh-tw/windows/downloads/windows-10-sdk)
* [Holographic Remoting Player](https://www.microsoft.com/en-us/p/holographic-remoting-player/9nblggh4sv40?activetab=pivot%3aoverviewtab)

### 參考文件

* [Get your HoloLens (1st gen) ready to use](https://docs.microsoft.com/en-us/hololens/hololens1-setup)
  說明 HoloLens 硬體相關功能和操作方式。
* [Unity development overview](https://docs.microsoft.com/en-us/windows/mixed-reality/unity-development-overview)
  說明使用 Unity 搭配 MRTK 的相關建議設定。
* [Mixed Reality Toolkit (MRTK) Readme Page](https://microsoft.github.io/MixedRealityToolkit-Unity/README.html)
  MRTK 的說明文件入口。提供使用設定、Components 說明，以及一些設計範例。
* [Unity UI on the HoloLens](https://forum.unity.com/threads/unity-ui-on-the-hololens.394629/)