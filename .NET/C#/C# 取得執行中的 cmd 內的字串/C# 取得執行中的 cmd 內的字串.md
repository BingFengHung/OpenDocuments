# C# 取得執行中的 cmd 內的字串

如需擷取執行中 cmd 內的字串出來進行結果分析要如何做呢? 本篇將介紹一個簡單的方法將 cmd 執行的內容擷取出來。

下面函式會去呼叫一個 cmd 程式起來執行，然後回傳 cmd 執行過程中的所有結果，如下圖 1 所示：

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

	string strOupt = null;

	try {
		p.Start();
		p.StandardInput.WriteLine(commandText);
		p.StandardInput.WriteLine("exit");

		strOutput = p.StandardReadToEnd();

		p.WaitForExit();
		p.Close();
	} catch (Exception e) {
		strOutput = e.Message;
	}

	return strOutput;
}
```

首先，先使用 System.Diagnostics.Process 的類別來叫起一隻 cmd，然後再這個類別裡面，要將 StartInfo.RedirectStandardOutput 設定為 true，為甚麼要設定為 true 呢?

來看看 DotNetPerls 對這個參數的解釋
[`RedirectStandardOutput` eliminates the need for output files. It allows us to use a console program directly inside a C# program - witouht havning to rply on ouput file being written to the disk.]

簡單的說，就是在處理 cmd 輸出的字串時，不需要再另外存成一個檔案，再進行操作，而是直接可以在執行的程式中，獲取結過進行字串操作。

執行程式的時候，該程式會呼叫 cmd 起來，然後就會開始記錄此握城中所有在 cmd 中出現的字串，最後再使用一個 StandardOutput.ReadToEnd() 的函式，就可以得到 cmd 執行的所有結果了。