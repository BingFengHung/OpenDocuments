# Xamarin.Forms Behavior讓控制項具有額外的行為

Behavior 讓你加入新功能到使用者控制項上面，而不用透過繼承控制項來做出新類別，才能夠去完成這個新功能。`透過將功能實作於 behavior 類別上面，再附加到控制項上面`，以此來當作是控制項本身的部分功能一樣。

Behaviors 讓你將平常需要撰寫 code-behind 程式碼 (因為可以直接與控制項的 API 互動)，改為透過 Behavior 附加這樣的方式，可以更簡潔的附加到控制項上面，並且將其包裝來讓其他應用程式使用。

行為的設計可為控制項提供更全面的功能，例如：
- 為 Entry 加入信箱驗證器
- 控制動畫
- 加入效果到控制項上面

除此之外，他們可用於將命令與未設計為與命令互動的控制項相關聯。例如：可以用來呼叫命令來回應事件的觸發。

**Xamarin.Forms 支援兩種不同樣式的行為**
- Xamarin.Forms behaviors：實作一個繼承至 Behavior 或是 Behaviro<T> 的類別，其中 T 為控制項的類型，指定行為可使用在哪種控制項上面。
- Attached behaviors：一個 static 類別，包含一個或多個附加屬性。

### 1. 建立 Attached Behaviors
先從 Attached Behaviors 來看，附加行為是一個具有多個附加屬性的靜態類別。

他們定義在一個類別中，但是可以附加到其他物件上面，且他們可被 XAML 辨識為屬性，如下圖 1 所示。

```xml
<Entry Placeholder="Enter a System"
       local:NumberValidationBehavior.AttachBehavior="true"/>
```
圖1、加入附加行為

附加屬性可以定義一個 propertyChanged 的委派，在當屬性的值改變時，會被呼叫，如同屬性被設定在控制項上面。當 propertyChanged 委派執行，**他會傳遞其有被附加的控制項的參考，並且傳遞舊的值與新的值**。

可以透過此委派將傳入的控制項加入新功能，如下所述：
1. propertyChanged 委派轉換物件參考，轉成控制項類型。
2. propertyChanged 委派修改控制項的屬性、控制項方法的呼叫與註冊事件處理器。

有一個問題是，附加屬性被定義在 static 類別，具有 static 屬性與方法。

基於上述原因，這讓附加屬性很無法保有狀態。**此外，Xamarin.Forms 行為已經取代附加行為作為更適合用來建立附加行為的方法。**

下面的範例將建立一個檢查輸入的數字是否為 double 型態，如果不是會將其標示為紅色字體，程式碼如下圖 2 所示。

```cs
public static class NumberValidationBehavior {
	public static readonly BindableProperty AttachBehaviorProperty = BindableProperty.CreateAttached(
		"AttachBehavior",
		typeof(bool),
		typeof(NumbericValidationBehavior),
		false,
		propertyChanged: OnAttachBehaviorChanged
	);

	public static bool GetAttachBehavior(BindableObject view) {
		return (bool)view.GetValue(AttachBehaviorProperty);
	}

	public static void SetAttachBehavior(BindableObject view, bool value) {
		view.SetValue(AttachBehaviorProperty, value);
	}

	static void OnAttachBehaviorChanged(BindableObject view, object oldValue, object newValue) {
		var entry = view as Entry;

		if (entry == null) 
		   return;

		bool attachBehavior = (bool)newValue;

		if(attachBehavior)
		   entry.TextChanged += OnEntryTextChanged;
		else 
		   entry.TextChanged -= OnEntryTextChanged;
	}

	static void OnEntryTextChanged(object sender, TextChangedEventArgs args) {
		double result;
		bool isValid = double.TryParse(args.NewTextValue, out result);
		((Entry)sender).TextColor = isValid ? Color.Default: Color.Red;
	}

}
```
圖2、製作附加行為

上面根據附加的屬性值，true 或 false 會加入 Text 改變的事件或是取消事件，如果是 true 的話，當 Text 值改變了，就進行 double 的判斷，判斷失敗將 text 改為紅色。

附加行為使用可在 XAML 與 C# 中使用，使用方法如下

在 XAML 中
```xml
<Entry Placeholder="Enter a System"
       local:NumbericValidationBehavior.AttaachBehavior="true">
```
在 C# 
```cs
var entry = new Entry { Placeholder = "Enter a System.Double" };
NumbericValidationBehavior.SetAttachBehavior(entry, true);
```

### 2. 建立 Xamarin.Forms Behaviors
Xamarin.Forms Behaviors 透過繼承 Behavior 或是 Behavior\<T> 來建立。

建立的流程如下：
1. 建立一個繼承至 Behavior 或 Behavior\<T> 的類別，其中 T 是控制項的類型。
2. 覆寫 OnAttached 方法來執行所需的設定。
3. 覆寫 OnDetachingFrom 方法來執行任何清除的需要。
4. 實作行為的核心功能

建立程式碼範例如下圖 3 所示：
```cs
public class CustomBehavior: Behavior<View> {
	protected override void OnAttachedTo(View bindable) {
		base.OnAttachedTo(bindable);
		// Perform setup
	}

	protected override void OnDetachingFrom(View bindable) {
		base.OnDetachingFrom(bindable);
		// Perform cleanup
	}

	// Behavior implementation
}
```
圖 3、建立行為

當行為被附加到控制項時，OnAttachedTo 方法會馬上觸發。這個方法會接收附加此行為的物件的參考，並且可以被使用來加入事件處理器或是執行其他的設定。例如，你可以訂閱一個控制項上的事件。

當行為從控制項上面移除時， OnDetachingFrom 會被觸發。此方法一樣接收附加此行為的物件的參考，並且被使用來執行任何清除的需要。例如，你需要取消控制項事件的訂閱來避免記憶體洩漏。

按照前一個附加屬性，這次用行為來實現，程式碼如下圖 4 所示。
```cs
public class NumbericValidationBehavior: Behavior<Entry> {
	protected override void OnAttachedTo(Entry entry) {
		entry.TextChnaged += OnEntryTextChanged;
		base.OnAttachedTo(entry);
	}

	protected override void OnDetachingFrom(Entry entry) {
		entry.TextChnaged -= OnEntryTextChanged;
		base.OnDetachingFrom(entry);
	}

	void OnEntryTextChanged(object sender, TextChangedEventArgs args) {
		double result;
		bool isValid = double.TryParse(args.NewTextValue, out result);
		((Entry)sender).TextColor = isValid? Color.Default: Color.Red;
	}
}
```
圖 4、型別檢驗行為

- OnAttachedTo 註冊控制項 TextChanged 的事件
- OnDetachingFrom 取消註冊控制項 TextChanged 的事件

使用 Xamarin.Forms Behavior，每個 Xamarin.Forms 控制項都有一個 Behaviors 的集合，可用來新增一個或多個行為，使用方法如下：

在 XAML 中使用
```xml
<Entry Placeholder="Enter a System.Double">
   <Entry.Behaviors>
	    <local:NumbericValidationBehavior/>
	 </Entry.Behaviors>
</Entry>
```

在 C# 中使用
```cs
var entry = new Entry { Placeholder = "Enter a System.Double" };
entry.Behaviors.Add(new NumbericValidationBehavior())/
```

要從控制項中移除行為，除非透過從控制項的 Behaviors 集合來移除，否則無法移除控制項的行為。

移除指定行為
```cs
var toRemove = entry.Behaviors.FirstOrDefault(b => b is NumbericValidationBehavior);

if (toRemove != null) {
	entry.Behaviors.Remove(toRemove);
}
```

將行為全部移除 entry.Behaviors.Clear()
