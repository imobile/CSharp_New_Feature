# C#6 の新機能

## 自動プロパティ (auto-property) の強化

### C#5 までの自動プロパティ

```csharp
public string FirstName { get; set; }
public string LastName { get; set; }
```

getter, setter 両方持った自動プロパティを作れる。
下記のようなコードを書くのと同じ

```csharp
public string FirstName
{
    get { return _firstName; }
    set { _firstName = value; }
}

// バッキングフィールドが自動で生成される
private string _firstName;
```

### 読み取り専用の自動プロパティ (Read-only auto-properties)

C#6 で、 readonly な自動プロパティを書けるようになった

#### C#5 まで

* setter を private にしてがまんする (readonly ではない)
    ```csharp
    public string FirstName { get; private set; }
    ```
* 自動プロパティを使わずに、自分で readonly なバッキングフィールドをつくる
    ```csharp
    public string FirstName 
    { 
        get { return _firstName; }
    }
    private readonly string _firstName;
    ```

#### C#6 から

getter のみの自動プロパティが書けるようになった

```csharp
public string FirstName { get; }
```

readonly なフィールドと同様、値をセットできるのはコンストラクタか初期化子のみ

### 自動プロパティの初期化子 (Auto-Property Initializers)

```csharp
public class Student
{
    public ICollection<Class> Classes { get; } = new List<Class>();
}
```

### expression-bodied な関数メンバ (Expression-bodied function members)

本体が 1 つの文しかないようなメンバは式で置き換えられる

#### C#5 以前

C#5 以前は 1 つの式で書けるものでも return 文を書く必要があった

```csharp
public class Student
{
    public string FirstName { get; }
    public string LastName { get; }

    public string FullName 
    {
        get { return string.Format("{0} {1}", this.FirstName, this.LastName); }
    }

    public Student(string firstName, string lastName)
    {
        this.FirstName = firstName;
        this.LastName = lastName;
    }

    public override string ToString()
    {
        return string.Format("{0}, {1}", this.LastName, this.FirstName);
    }
}
```

#### C#6

```csharp
public class Student
{
    public string FirstName { get; }
    public string LastName { get; }

    // expression-bodied なプロパティ (get only)
    public string FullName => $"{this.FirstName} {this.LastName}";

    public Student(string firstName, string lastName)
    {
        this.FirstName = firstName;
        this.LastName = lastName;
    }

    public override string ToString() => $"{this.LastName}, {this.FirstName}";
}
```


## using static

`using static` で指定したクラスの static メソッドを import できる

### 静的メソッドの import

#### C#5 以前

static メソッドは必ずクラス名を指定して呼び出す

```csharp
using System;
```

```csharp
if (string.IsNullOrEmpty(hoge))
{
    throw new ArgumentException("Cannot be blank", "hoge");
}
```

#### C#6

`using static` を使用してクラス名の指定を省略できる

```csharp
using System.String;
```

```csharp
if (IsNullOrEmpty(hoge))
{
    throw new ArgumentException("Cannot be blank", nameof(hoge));
}
```

### 拡張メソッドの using static

たとえば、 `System.Linq.Enumerable` を `using static` する

```csharp
using static System.Linq.Enumerable;
```

```csharp
// Enumerable.Range() はこのように呼び出せる
IEnumerable<int> range = Range(0, 10);

// Enumerable クラスのすべての拡張メソッドが使用できる
int first = range.First();

// 拡張メソッドを static メソッドと同じように呼ぶことはできない
// int first = First(range);
```

## null 条件演算子 (Null-conditional operators)

`null` 条件演算子を使うと `null` チェックを簡単に書ける

### C#5 以前

```csharp
// person は null になるかもしれないので、 null チェック
var firstName = person != null ? person.FirstName : null;
```

C#5 以前では、自分で `null` チェックをかける必要がある

### C#6

null 条件演算子: `?.` を使えば、 null の場合はそのまま null をかえし、 null でないときだけその先が評価される

```csharp
// person が null の場合は firstName も null
var firstName = person?.FirstName;
```

`??` 演算子と組み合わせて最後にデフォルト値を入れられる

```csharp
(null as IEnumerable<int>)?.Select(x => x * 2).Select(x => x + 1) ?? Enumerable.Empty<int>()
```

## 文字列挿入 (String Interpolation)

`string.Format` のかわりに文字列挿入を使える

### string.Format を使った場合 (C#5)

```csharp
public string FullName
{
    get
    {
        return string.Format("{0} {1}", FirstName, LastName);
    }
}
```

### 文字列挿入を使った場合 (C#6)

文字列に `$` をつけると文字列挿入が使える

```csharp
public string FullName => $"{FirstName} {LastName}";
```

### すこし複雑な例

`{}` 内には、 三項演算子以外の C# の式 (expression) を書ける

```csharp
public string GetFormattedGradePoint() =>
    $"Name: {LastName}, {FirstName}. G.P.A: {Grades.Average()}";
```

### フォーマット指定

```csharp
// 小数点以下 2 桁まで
public string GetFormattedGradePoint() =>
    $"Name: {LastName}, {FirstName}. G.P.A: {Grades.Average():F2}"; // Name: Yamada, Hanako. G.P.A: 12.25
```

### 三項演算子の使用

三項演算子はそのままでは書けないので `()` でかこむ

```csharp
// 小数点以下 2 桁まで
public string GetFormattedGradePoint() =>
    $"Name: {LastName}, {FirstName}. G.P.A: {(Grades.Any() ? Grades.Average() : double.NaN):F2}"; // Name: Yamada, Hanako. G.P.A: 12.25
// Grades が空の場合 -> Name: Yamada, Hanako. G.P.A: NaN
```

### ふつうの文字として `{}` を書きたい場合

`{{`, `}}` と続けて書くことでエスケープできる

```csharp
$"Point: {{{Point.X}, {Point.Y}}}" // Point: {10, 20}
```

### 複数行の文字列挿入

`@` (逐語的文字列リテラル) と組み合わせて `$@` を使うと複数行の文字列挿入が可能

```csharp
$@"Name: {LastName}, {FirstName}.
G.P.A: {(Grades.Any() ? Grades.Average() : double.NaN):F2}"
```

エスケープのルールは逐語的文字列リテラルと同じ

参考: [逐語的文字列リテラル](http://ufcpp.net/study/csharp/st_embeddedtype.html#verbatim-string)

## 例外フィルタ (Exception Filters)

### C#5 まで

C#5 までで catch した例外をフィルタしたい場合

```csharp
try
{
    await this.HogeAsync();
}
catch (HogeException e)
{
    if (e.Message.Contains == "fuga") 
    {
        // fuga が含まれてたときだけなにかするとか
        return Foo();
    }
    else 
    {
        throw;
    }

}
// 他の時はそのまま throw する
```

### C#6

`when` で指定した条件を満たす場合だけ catch する

```csharp
try
{
    await this.HogeAsync();
}
catch (HogeException e) when (e.Message.Contains == "fuga")
{
    // fuga が含まれてたときだけなにかするとか
    return Foo();
}
// 他の時はそのまま throw する
```


## `nameof` (`nameof` Expressions)

`nameof` 式の評価結果はそのシンボルの名前になる
変数名、プロパティ名、フィールド名などが必要な場合につかえる

たとえば、 ArgumentException でパラメータの名前を書くとき

```csharp
if (IsNullOrEmpty(hoge))
{
    throw new ArgumentException(message: "Cannot be blank", paramName: nameof(hoge));
}
```

構文解析の対象にできるので、 hoge のリネームの時など直し漏れがおこらない

## インデックス初期化子 (Index Initializers)

コレクション初期化子でインデクサを使えるようになった

```csharp
private Dictionary<int, string> webErrors = new Dictionary<int, string>
{
    [404] = "Page not Found",
    [302] = "Page moved, but left a forwarding address.",
    [500] = "The web server can't come out to play today."
};
```

## その他の機能

* catch, finally 句での await
* オーバーロード解決の改善
* 構造体のコンストラクタ内でのプロパティ初期化
* コンストラクタの循環参照の禁止
* 未初期化変数の判定改善
* enum の基底型 (`System.Int32` とかが書けるように)
* 変数のシャドーイングルールの変更
* 最適化の改善

## 参考

* [What's New in C# 6 - C# Guide | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-6)
* [C# 6 の新機能 - C# によるプログラミング入門 | ++C++; // 未確認飛行 C](http://ufcpp.net/study/csharp/ap_ver6.html)