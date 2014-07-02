# C++11分科会 第5回資料
担当: semiexp

## オーバーロード

だいたい 11 じゃない C++ と同じ．

### オーバーロード可能な宣言，不可能な宣言
シグネチャが異なっても，「戻り値型」「static の有無」だけが異なる場合はオーバーロードできない．
ただし，リファレンス修飾子が異なる場合はオーバーロード可能である．
リファレンス修飾子は，省略の場合は暗黙に lvalue となるが，省略を行った場合はこの違いのみでオーバーロードはできない．

```cpp
struct hoge
{
    void f();
    void f() &&; // 不可能なオーバーロード
    
    void g() &;
    void g() &&; // 可能なオーバーロード
};
```

### オーバーロード解決の規則
候補関数 candidate functions のうち，最適な関数が選択される．
この候補関数となる関数は，「普通に考えて候補になりそうなもの」と思ってよさそうなものである．
最適関数の選択規則は複雑であり，特に目新しい点は見当たらないため，省略する．

### 演算子オーバーロード : ユーザー定義リテラル
C++11 で追加された機能として，「ユーザー定義リテラル」がある．
オーバーロード演算子の形は以下のようになる：

```cpp
operator "" (identifier)
```

"" と identifier の間には，必ず 1 つ以上の空白文字が必要である．
また，identifier は，必ず 1 つ以上のアンダースコア _ で始まらなければならない．

仮引数リストは，以下のいずれかに限られる：
```cpp
const char*
unsigned long long int
long double
char
wchar_t
char16_t
char32_t
const char*, std::size_t
const wchar_t*, std::size_t
const char16_t*, std::size_t
const char32_t*, std::size_t
```

```cpp
// OK
void operator "" _x (unsigned long long int); 

// NG: identifier が _ から始まっていない
void operator "" y (unsigned long long int);

// NG: 引数の型が不適切
void operator "" _z (int); 
```

リテラル演算子はテンプレートとしても定義できるが，仮引数リストが空かつ，テンプレート仮引数は，char 型の仮引数パックである必要がある：
```cpp
template <char ... Chars> void operator "" _x () {}
```
上のようなテンプレート定義については後述．

リテラル演算子は，名前空間スコープで宣言せねばならず，クラスメンバなどであってはいけない．(クラスの friend にはなれる)
グローバル名前空間スコープで宣言されたリテラル演算子は直接使用可能であるが，名前空間スコープで宣言された場合は，using 宣言が必要である．

```cpp
namespace ns {
    void operator "" _x (unsigned long long int) {}
}

int main()
{
    1_x; // NG
    
    {
        using namespace ns;
        1_x; // OK
    }
    
    {
        using ns::operator "" _x;
        1_x; // OK
    }
}

```

上の例でもわかるように，リテラル演算子は必ずしも値を返す必要はなく，void 型でも構わない．

## テンプレート

### alias 宣言テンプレート
クラス，関数，エイリアス宣言に対してテンプレートを用いることが可能である．

typedef はテンプレート化できなかったが，using を用いた alias 宣言はテンプレート化可能である：

```cpp
#include <algorithm>

// NG: typedef はテンプレート化できない
template <typename T>
typedef pair<T, T> spair2<T>;

template <typename T>
using spair = std::pair<T, T>;

int main()
{
    spair<int> p = std::make_pair(3, 4);
}
```

### テンプレート引数
テンプレート仮引数にはクラスまたは値を取ることができる．また，デフォルトテンプレート実引数を指定することができる．

```cpp
#include <cstddef>

template <typename T = int, std::size_t len = 10>
struct minivec
{
...
}
```

これは今までの C++ と同じであるが，C++11 では可変テンプレート仮引数が新たに追加された．

### 可変テンプレート仮引数
識別子の前に ... を記述すると，仮引数パックの宣言となり，可変引数テンプレートの宣言となる：
```cpp
template <typename ... Types>
struct X {};
```
テンプレート仮引数パックは，0 個以上のテンプレート実引数を取る．
可変引数テンプレートにできるものは，型，値，テンプレートなどがある．

```cpp
// 型テンプレート
template <typename ... Types>
struct X {};

// 値テンプレート
template <int ... Ints>
struct Y {};

int main()
{
    X <> x1;
    X <int, double, float> x2;
    
    Y <> y1;
    Y <2, 7, 1, 8, 2, 8> y2;
}

```

関数テンプレートも，同様に可変引数の定義ができる：
```cpp
template <typename ... Params>
void f (Params ... p1) {}

int main()
{
    f();
    f(1, 2, 3);
}

```

仮引数パックは展開しないと使えない．
パックの名前の後に ... を付けると，仮引数パックが展開される．
見た目上は，仮引数パックの指定の文字列が繰り返されたような振る舞いをする．
ただし，展開方法は文脈に依存する．

```cpp
template <typename ... Types>
struct hoge {};

template <typename ... Types>
struct piyo
{
    using type1 = hoge<Types...>;
    
    using type2 = hoge<Types>; // NG: 展開せずに仮引数パックは使えない
    
    using type3 = hoge<Types * ...>;
};

int main()
{
    // hoge<char, int>
    piyo<char, int>::type1 t1;
    
    // hoge<char*, int*>
    piyo<char, int>::type3 t3;
}

```

つまり，「パックの名前があるところは，実引数で置き換えられて，その後で文脈に即した型になる」というのが，実引数の数だけ繰り返される，と思うことができる．
関数仮引数パックの場合も同様である：

```cpp
#include <algorithm>

template <typename ... params>
void f (params ... pack) { }

template <typename ... params>
void g (params ... pack)
{
    f (pack ...); // 値をそのまま f に渡す
    f ((pack * 2) ...); // 各値を 2 倍して f に渡す
    f (std::max(5, pack)...); // 各値と 5 との max をとって f に渡す
}

```

### 可変引数テンプレートの使用
「仮引数パックの先頭を取ってくる」ような処理は，直接には記述できない．
可変引数テンプレートを使って実際に汎用的なコードを作成するには，2 通りの方法がある：

- 固定長引数関数に投げる
- 再帰を使う

固定長引数関数に渡すのは，単にパックを展開するだけである．
しかし，この方法では汎用的なコードにならない．
任意個の引数に対応するには，再帰を用いる：

```cpp
template <typename T>
T min(T a1, T a2)
{
    return a1 < a2 ? a1 : a2;
}

template <typename T, typename ... Types>
T min(T head, Types ... tail)
{
    return min(head, min(tail ...));
}

int main()
{
    min(5, 8);
    min(1, 2, 3, 4, 5);
    min(1.0, 3.0, 2.2, 4.4);
}
```

仮引数パックを使わない関数テンプレートは，仮引数パックを使う関数テンプレートよりも優先順位が高いため，例えば min(5, 8); では
仮引数パックを使わない部分が正しく呼ばれる．

## ライブラリ
C++11 で追加されたライブラリは，あまりにもたくさんあるので，すべてを紹介するのはとても不可能である．
ここでは，面白そう，有用そう，と思ったものをいくつか紹介する．

### 正規表現
正規表現は，C++11 でようやく標準ライブラリに追加された．

正規表現ライブラリを用いるには次を行う：
```cpp
#include <regex>
```

簡単な使用例を次に示す．

このコードは，入力した文字列が leading-zero を持たない整数 (28, 510, 0 など) かどうか判定するはずである．
はずであるというのは，手元ではこのコードを実行すると regex の生成の段階で例外が発生した．
ちなみに，regex のコンストラクタの引数が正規表現として不適切な場合は regex_error 型例外が発生する．

```cpp
#include <iostream>
#include <regex>

using namespace std;

int main() {
    regex re("0|([1-9][0-9]*)");
    string in;
    
    cin >> in;
    
    if (regex_match(in, re)) {
        cout << "An integer without leading-zeros" << endl;
    }
    
    return 0;
}
```

GCC 4.9.0 以降では正しく動くはずである．
参考：[c++ - Is gcc4.7 buggy about regular expressions? - Stack Overflow](http://stackoverflow.com/questions/12530406/is-gcc4-7-buggy-about-regular-expressions)

- regex などはすべて std 名前空間に定義されている．
- regex 型は char 型の文字列を処理するのに使う．wchar_t 型用に，wregex というものも用意されている．
- regex_match は，文字列全体が正規表現にマッチしているかを判定する．文字列としては，イテレータの組，const char *，std::string が使える．
- 検索 regex_search，置換 regex_replace などといった関数も存在する．(ここでは解説は省略する)

### スマートポインタ
C++11 で，unique_ptr, shared_ptr, weak_ptr といったスマートポインタが追加された．
auto_ptr というものも (前から) あるが，これはあまり使うべきでない．

- unique_ptr は，変数の寿命が尽きたら自動で管理している領域を解放するスマートポインタである．
- shared_ptr は，参照カウント式のスマートポインタである．
- weak_ptr は，shared_ptr に付随して使うもので，shared_ptr の指す領域を指すが，参照カウントに影響しない．
これは，循環参照を防ぐために便利である．

これらを用いるには次を行う：
```cpp
#include <memory>
```

unique_ptr についてのみ説明する．
unique_ptr の使い方は簡単で，new した領域をそのまま unique_ptr に渡すだけである．
unique_ptr が管理している領域は，明示的に解放する必要がない．
関数スコープからの離脱，例外が発生した，などのタイミングで，自動で解放される．

```cpp
#include <memory>

struct piyo
{
//...

    int value () { return 5; }
};

void hoge()
{
    std::unique_ptr<piyo> ptr(new piyo);
    
    int v = ptr->value(); // 普通のポインタと同じように使える
    // 明示的な解放処理は不要
}

int main()
{
    hoge();
    
    return 0;
}
```

### スレッド
スレッドを用いるには，次を行う：
```cpp
#include <thread>
```

C++11 のスレッドの基本は，std::thread である．
例えば次のコードを実行すると，1～1000 までを出力する処理が 2 スレッドで並行して行われる．

```cpp
#include <cstdio>
#include <thread>

void f(int id) {
    for (int i = 1; i <= 1000; i++) fprintf(stderr, "%d %d\n", id, i);
}

int main()
{
    std::thread th1 (f, 1);
    std::thread th2 (f, 2);
    
    th1.join();
    th2.join();
    printf("fin.\n");
    return 0;
}
```

std::thread の引数には，関数オブジェクトと，その関数オブジェクトに対する引数を渡す．
join を用いると，その (std::thread が持っている) スレッドが終了するまで待機することができる．

std::thread は，関数の戻り値を捨ててしまう，例外を適切に捕捉できないなどの問題があるため，
他に std::async なども用意されている．

std::async を用いると，次のコードのように，関数の戻り値を取得することができる．
std::async は std::future 型を返し，future 型に対して get を行うと実際の戻り値を取得できる．
次のコードを実行するとわかるように，async の呼び出しの関数オブジェクトの実行は当然別スレッドで行われる．

```cpp
#include <cstdio>
#include <thread>
#include <future>
// std::future の使用にはこの記述が必要

int f(int id) {
    for (int i = 1; i <= 1000; i++) fprintf(stderr, "%d %d\n", id, i);
    return id;
}

int main()
{
    std::future<int> f1 = std::async(std::launch::async, f, 1);

    for (int i = 1; i <= 1000; i++) fprintf(stderr, "main %d\n", i);

    fprintf(stderr, "f1 %d\n", f1.get());

    return 0;
}
```

## 参考
- [C++11の文法と機能\(C++11: Syntax and Feature\)]( http://ezoeryou.github.io/cpp-book/C++11-Syntax-and-Feature.xhtml )
- [標準C++の正規表現: ＜regex＞ ](http://codezine.jp/article/detail/7716)
- [スマートポインタの使い方 その1：unique_ptrAdd Star](http://d.hatena.ne.jp/A7M/20110520/1305894007)
- [C++11時代のthreading](http://d.hatena.ne.jp/fjnl/20111206/1323159773)
