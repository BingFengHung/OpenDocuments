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