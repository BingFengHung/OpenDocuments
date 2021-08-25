# C# 取得執行中命令提示字元的字串內容

如果要擷取執行中的命令提示字元裡面的字串內容出來，該怎麼做呢?

本篇將介紹一個簡單的方法取得執行內容的回傳。

下面函式會透過 cmd 執行一個命令提示字元，然後回傳該命令提示字元執行過程中的所有內容。

```cs
public static string CommandOutput(string commandText) {
	System.Diagnostics.Process p = new System.Diagnostics.Process();

	p.StartInfo.WorkingDirectory = @"C:\Windows\system32\";
	p.StartInfo.FileName = "cmd.exe";
	p.StartInfo.UseShellExecute = false;
	p.StartInfo.RedirectStandardInput = true;
	p.StartInfo.RedirectStandardOutput = true;
	p.StartInfo.RedirectStandardError = true;
	p.EnableRaisingEvents = true;
	p.StartInfo.CreateNoWindow = false;

	string strOutput = null;

	try {
		p.Start();
		p.StandardInput.WriteLine(commandText);
		p.StandardInput.WriteLine("exit");

		strOutput = p.StandardOutput.ReadToEnd();

		p.WaitForExit();
		p.Close();
	} catch(Exception e) {
		strOutput = e.Message;
	}

	return strOutput;
}
```

首先，使用 System.Diagnostics.Process 的類別來叫起一支 cmd，然後這個類別裡面，需要設定 StartInfo.RedirectStandardOutput 設為 true，為什麼要設定為 true 呢? 來看看 DotNetPerls 對此參數的解釋 \[RedirectStandardOutput eliminates the need for output files. It allows us to use a console program direcrtly inside a C# program - without having to rely on output files being written to the disk.]

簡單來說，在處理 cmd 輸出的字串時，不需要另外存成一個檔案，再進行操作，直接可以在直行的城市獲取結果進行所需的字串操作。

執行程式的時候，該程式會呼叫 cmd 起來然後就會開始記錄此過程中所有再 cmd 中出現的字串，最後在使用一個StandardOutput.ReadToEnd() 的函式，可以得到 cmd 執行的所有結果了。