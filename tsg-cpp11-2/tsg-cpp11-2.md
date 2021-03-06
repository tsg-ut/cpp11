# C++11分科会第二回資料 
## 式
### 式
* 演算子とオペランドを組み合わせたもの
* オペランドの型が一致しない時は型変換が行われる  
このあたりはたぶん今まで通り
* ただし、scoped enumは変換されない

### scoped enum型
書き方

    enum class ScopedEnum : long {
    	A, B, C
    };

* C++11から導入
* 整数型への暗黙的な型変換はできない(明示的なキャストが必要)
* 基盤となる型(underlying type)が指定できる
* 各enumeratorは型名で修飾しないと参照できない
* 参照する時は
`ScopedEnum e = ScopedEnum::A;`
* enum classと書いてもenum structと書いても同じ

### unscoped enum型
* 従来のenum

### ラムダ式
    [ /*ラムダキャプチャー*/ ] // ラムダ導入子
    ( /*仮引数リスト*/ ) // 省略可能
    -> void // 戻り値の型、省略可能。autoで推測させる事もできるが{ return 式 ; }の形のみ
    {} // 複合文
最小: `[]{}`

    // 通常の関数
    auto f() -> void {}
    
    int main()
    {
    	f(); // 関数の呼び出し
    
    	// ラムダ式
    	auto g = []() -> void {};
    	g(); // ラムダ式の呼び出し
    
    	// ラムダ式を直接呼び出す
    	[]() -> void {}();
    }

ラムダ式は、例外指定できる。

    []() noexcept {} ;

### 変数のキャプチャー
* ラムダ式の`[]`の中に変数のキャプチャーを記述する。
* キャプチャーした変数はラムダ式の中で利用できる。
* ラムダ式がキャプチャーできるのは、ラムダ式が記述されている関数の、最も外側のブロックスコープ内。
* グローバル変数はキャプチャーしないでアクセスできる(キャプチャーしようとするとエラー)。

#### コピーキャプチャー
変数の値をコピーして保持する。

#### リファレンスキャプチャー
変数への参照を保持する(変数の寿命に注意)。

例

    int main()
    {
    	int x = 0 ;
    
    	[=]
    	{ // コピーキャプチャー
    		int y = x ; // OK、読むことはできる
    		x = 0 ; // エラー、書き換えることはできない
    	} ;
    
    	[&]
    	{ // リファレンスキャプチャー
    		int y = x ; // OK
    		x = 0 ; // OK
    	} ;
    }

#### thisのキャプチャー
    struct C
    {
    	int x ;
    	void f()
    	{
    		[=]{ x ; } ; // OK、ただし、これはキャプチャーではないことに注意
    	}
    } ;
非staticなメンバー関数のラムダ式では、データメンバーを使うことができる。
これは実際にはthisがキャプチャーされている。
thisは必ずコピーキャプチャーされる。
つまり(`[this]`はOKだが、`[&this]`はエラー)

#### デフォルトキャプチャー
* `[=]` コピー
* `[&]` リファレンス

変数ごとに、キャプチャー方法を指定することもできる。

デフォルトキャプチャーと同じキャプチャー方法を、個々のキャプチャーで指定することはできない。

    int main()
    {
    	int a = 0, b = 1, c = 2;
    	[=, &a]{};	//aのみリファレンスキャプチャー、他はコピー
    	[a]{};	//aをコピーキャプチャー、他は参照できない
    }

ラムダキャプチャーを使わないラムダ式は関数ポインターに変換できる。

    void (*ptr1)(void) = []{} ;
    auto (*ptr2)(int, int, int) -> int = [](int a, int b, int c) -> int { return a + b + c ; };

#### ネスト
ラムダ式はネストできる。
内側のラムダ式は、外側のラムダ式のブロックスコープから見える変数しか、キャプチャーすることはできない。

    int main()
    {
    	int a = 0 ; int b = 0 ;
    	[b]{ // 外側のラムダ式
    	int c = 0 ;
    		[=]{ // 内側のラムダ式
    			a ; // エラー、aはキャプチャーできない。
    			b ; // OK
    			c ; // OK
    		} ;
    	} ;
    }



### 疑似デストラクター呼び出し（Pseudo destructor call）
デストラクタは明示的に呼び出せる。

    // このコードは、疑似デストラクター呼び出しの文法を示すためだけの例である
    struct C {} ;
    
    int main()
    {
    	C c ;
    	c.~C() ; // 疑似デストラクター呼び出し
    	C * ptr = &c
    	ptr->~C() ; // 疑似デストラクター呼び出し
    }

placement newと組み合わせるなどの使い道があるらしい

### alignof
オペランドの型のアライメント要求を返す。
結果はstd::size_t型の定数。

### noexcept
オペランドの式が、例外を投げる可能性のある式を含むかどうかを返す。
結果の型はboolの定数。

### コンマ演算子
* 「左から右」に評価される。
* 結果の値は右のオペランドを評価した結果。

    x = (f(), g());

はまず`f()`が評価され、次に`g()`が評価され、`x`に代入されるのは`g()`の結果
(括弧は`(x = f()), g()`と解釈させないためのもの)

### その他些細な事(C++11とはあまり関係ない)
* 関数の実引数は評価の順番は決まっていない。
* main関数は再帰できない。
* 配列の`x[1]`と`1[x]`は同じ。

## 文
しっかりとは読んでないけど、目新しいのはrange-based for文くらい。
### range-based for文（The range-based for statement）
`for ( for-range-宣言 : for-range-初期化子 ) 文`

    int main()
    {
    	int a[] = { 1, 2, 3 } ;
    	for ( int i : a ) ; // 各要素をint型のコピーで受ける
    	for ( int & ref : a ) ; // 各要素をリファレンスで受ける
    	for ( auto i : a ) ; // auto指定子を使った例
    }
