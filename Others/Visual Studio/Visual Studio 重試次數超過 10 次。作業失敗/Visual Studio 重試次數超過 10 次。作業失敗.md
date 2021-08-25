# Visual Studio 重試次數超過 10 次。作業失敗
Visual Studio 重試次數超過 10 次。作業失敗。

說明：
使用 Visual Studio 2019 建置專案時出現：

> 無法將 “obj\x86\Debug\xxx.exe” 複製到 “bin\Debug\xxx.exe”。重複次數超過 10 次。作業失敗。

> 無法將檔案”obj\x86\Debug\xxx.exe” 複製到 “bin\Debug\xxx.exe”。由於另一個程序正在使用檔案 “bin\Debug\xxx.exe” ，所以無法存取該檔案。

作法：

先到 obj 的資料夾將 xxx.exe 檔案複製到 bin 底下的檔案取代，發現無法取代後，開啟工作管理員檢查是否有執行檔還是開啟的狀態，確認檔案確實被咬住，結束程式後，就可順利執行了。
