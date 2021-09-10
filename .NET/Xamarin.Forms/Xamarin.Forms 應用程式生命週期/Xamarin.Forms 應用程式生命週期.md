# Xamarin.Forms 應用程式生命週期

要講應用程式生命週期，必須要先從最一開始的 `Application` 類別開始說起，這個基底類別由 App 類別所繼承 (程式的啟動點)， Application 類別具有以下幾項功能：

- `MainPge` 屬性，可用指定 app 的初始頁面。
- 永續 `Properties dictionary` 可用來儲存一些簡單的值，用作不同生命週期時，狀態的改變。
- 靜態 `Current` 屬性，參考到當前 application 物件。

裡面包含了生命週期方法，如 `OnStart`、`OnSleep` 與 `OnResume` 以及 Modal 導覽事件。

## MainPage Property
