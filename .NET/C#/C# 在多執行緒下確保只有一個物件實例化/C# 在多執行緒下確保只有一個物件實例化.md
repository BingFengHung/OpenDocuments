# C# 在多執行緒下確保只有一個物件實例化

會提到這一篇是因為在我們的程式碼中或多或少會使用到 static 物件，這個static 物件可能在程式一開始執行的時候會產生一個實例化，但是在單一執行緒的情況下執行還不會有問題，但是在多執行緒下，同時執行的時候，就有可能會發生建立多個實例化的狀況(使用獨體模式時可能會發生的狀況)，本篇的目的就是要確保 static 物件實例化時，就算在多執行緒下還是可以確保只產生一個實例化物件。

搜尋MSDN 很快就可以找到解決辦法，MSDN是這樣說的， lock 關鍵字可確保當關鍵區段中已有執行緒時，另一個執行緒不會進入程式碼的關鍵區段。 如果另一個執行緒嘗試進入鎖定的程式碼，它將會等候、封鎖，直到釋放物件，範例程式碼如下：

```cs
class Singleton 
{
	private static Singleton single;
	private static readonly object sync = new object();

	private Singleton() {}

	public static Singleton GetInstance() 
	{
		// 加入 lock 確保同一時間僅有一個執行緒可以進入 
		lock (sync) 
		{
			if (single == null) 
			{
				single = new Singleton();
			}
		}

		return single;
	}
}

```

從上面程式碼可能會有一個疑問，為甚麼我們要特地在建立一個 syncRoot 的物件，而不使用instance 我們要產生的物件呢? 因為instance 連有沒有實體化過都還不知道，是不可已確定能不能 lock 的。現在是解決不會產生多個實體化的問題了，但是這個程式碼還可以再提升效能，我們可以在多加入一個判斷式，這樣我們不會一進入就要直接面對 lock 影響效能，修改後程式碼如下：

```cs
class Singleton
{
	private static Singleton single;
	private static readonly object sync = new object();

	private Singleton(){}

	public static Singleton GetInstance()
	{
		if (single == null) 
		{
			// 加入 lock 確保同一時間僅有一個執行緒進入
			lock (sync) 
			{
				if (single == null)
				{
					single = new Singleton();
				}
			}
		}

		return single;
	}
}

```