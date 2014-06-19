C++11分科会第3回資料
====================

担当: hakatashi

# 宣言

## 変数及び関数の宣言

従来と変わらず。

```C++
#include <iostream>

int main() {
    int a, b, c, d;
    int const unsigned ONE = 1;
    const unsigned int TWE = 2;
    unsigned const int PI = 3;
    double number = 0.1;

    return 0;
}

int func (int arg) {
    // hogehoge
}
```

## static_assert (宣言式)

従来のassertの静的なやつ。コンパイル時に実行して括弧内が偽ならエラーを吐いてくれる。

```C++
#include <iostream>
#include <limits>

#define MAX_X 10000

int main () {
    char x;

    static_assert(std::numeric_limits<decltype(x)>::max() > MAX_X, "x overflows... :(");

    return 0;
}
```

decltypeは後で出てくる。

結果

```
C:\Users\Hakatashi\Documents\GitHub\cpp11\tsg-cpp11-3>g++ -std=c++11 test.cpp
test.cpp: In function 'int main()':
test.cpp:9:2: error: static assertion failed: x overflows... :(
  static_assert(std::numeric_limits<decltype(x)>::max() > MAX_X, "x overflows... :(");
```

## thread_local (指定子)

マルチスレッド環境においてスレッドごとに一意な値を保持する。

```C++
#include <iostream>
#include <thread>
#include <string>
#include <windows.h>

thread_local std::string name = "undefined";

void in_school() {
    name = "Misumi Nagisa";
    Sleep(2000);
    std::cout << "She is " << name << "." << std::endl;
}

void in_battle() {
    name = "Cure Black";
    Sleep(1000);
    std::cout << "She is " << name << "." << std::endl;
}

int main() {
    std::thread school(in_school);
    std::thread battle(in_battle);

    school.join(); // wait until thread terminates
    battle.join(); // wait until thread terminates

    std::cout << "No, she is " << name << "." << std::endl;
    return 0;
}
```

結果

```
C:\Users\hakatashi\Documents\GitHub\cpp11\tsg-cpp11-3>a
She is Cure Black.
She is Misumi Nagisa.
No, she is undefined.
```

## constexpr (指定子)

言わずと知れたコンパイル時実行。

### constexpr変数

constexprで宣言した変数はコンパイル時変数になる。宣言時に定数を代入する。

```C++
#include <iostream>

#define THREE 3

int main() {
    constexpr double CHIHAYA = 72;
    constexpr double PI = THREE;
    constexpr double RADIUS_OF_CHIHAYA = CHIHAYA / (PI * 2);

    std::cout << RADIUS_OF_CHIHAYA << std::endl; // -> 12

    return 0;
}
```

constexprな値を生成するには式中の値が全てconstexprでないといけない。
よって以下はエラーになる。

```C++
#include <iostream>

#define THREE 3

int main() {
    constexpr double CHIHAYA = 72;
    double PI = THREE;
    constexpr double RADIUS_OF_CHIHAYA = CHIHAYA / (PI * 2);

    std::cout << RADIUS_OF_CHIHAYA << std::endl;

    return 0;
}
```

```
C:\Users\Hakatashi\Documents\GitHub\cpp11\tsg-cpp11-3>g++ -std=c++11 test.cpp
test.cpp: In function 'int main()':
test.cpp:8:56: error: the value of 'PI' is not usable in a constant expression
  constexpr double RADIUS_OF_CHIHAYA = CHIHAYA / (PI * 2);
                                                        ^
test.cpp:7:9: note: 'PI' was not declared 'constexpr'
  double PI = THREE;
         ^
```

また、constexprおよびプリプロセス可能なマクロのみで生成されたconstはconstexprとなる。
以下はたぶんコンパイルを通る。

```C++
#include <iostream>

#define ONE 1

int main() {
    const int TWO = 2;
    constexpr double CHIHAYA = 72;
    const double PI = ONE + TWO;
    constexpr double RADIUS_OF_CHIHAYA = CHIHAYA / (PI * 2);

    std::cout << RADIUS_OF_CHIHAYA << std::endl; // -> 12

    return 0;
}
```

### constexpr関数

constexprな値を生成する関数。constexpr内には基本的にreturn文1文しか記述できない。厳しい。

C++14でだいぶ緩和されるらしい。

```C++
#include <iostream>

constexpr double RADIUS_OF(double CIRCLE) {
    return CIRCLE / (3.14 * 2);
}

int main() {
    constexpr double CHIRUNO = 9;

    std::cout << RADIUS_OF(CHIRUNO) << std::endl; // -> 1.43312

    return 0;
}
```

constexprなコンストラクタなどもあるが、だいたいいっしょである。

## auto (型指定子)

初期化時に適当に型推論してくれる。初期化式を指定しないとコンパイルエラーになる。

```C++
#include <iostream>
#include <vector>

int main() {
    auto integer = 10; // int
    auto number = 1e3l; // long
    auto character = 'a'; // char
    auto pointer = &integer; // int *
    const auto NOT_FOUND = 404; // const int

    std::vector<std::string> vector;
    auto iterator= vector.begin(); // std::vector<std::string>::iterator

    return 0;
}
```

## decltype (型指定子)

括弧内に指定した式の型になる。よって、`auto variable = hogehoge;`とするのは
`decltype(hogehoge) variable = hogehoge;`とするのとだいたい同じことである。(等価かどうかは知らない)

## 名前空間とか

めぼしい新機能はない。

## 属性(attribute)

属性は非常に汎用的な概念で、ソースコードに付加的な情報を加える。文法は角括弧2つを用いて`[[ attribute-list ]]`とする。

### align属性

変数を指定した数値の倍数のアドレスに配置することを強制するアライメントを実施する。

`[[ align() ]]`で指定するとされていたが、`alignas()`と型指定子のように書くよう修正された。

```C++
#include <iostream>

int main() {
    alignas(0x1000) int hanshin = 334;

    std::cout << &hanshin << std::endl; // -> 0x23a000

    return 0;
}
```

### noreturn属性

関数がreturnしないことを明示する。使い所がよくわからない。

関数がreturnしうる場合、コンパイラが警告を出す。

```C++
#include <iostream>

void throw_error [[ noreturn ]] () {
    throw "error";
}

int main() {
    throw_error();
    return 0;
}
```

### deprecated属性

deprecatedな関数、変数、クラスをマークする。G++は未対応の模様?

[教科書](http://ezoeryou.github.io/cpp-book/C++11-Syntax-and-Feature.xhtml)を参考に

```C++
#include <stdio.h>

[[ deprecated ]] char *gets(char * str);

int main() {
    char buf[1000];
    gets(buf);
}
```

# 宣言子

## \_\_func\_\_

C99では関数内で定数\_\_func\_\_を参照することにより関数名を得ることができる。
C++11においてC++でもこれが使えるようになったが、その中身は実装依存とされ仕様上は特に何の意味もない。

## default/delete定義

ちょっとよくわかりませんでした＞＜

## 初期化

真新しいのはリスト初期化(uniform initialization, list initialization)くらいである。

リスト初期化は、これまでリテラルごとに異なっていた初期化のシンタックスを統一するための記法で、波括弧を使って表現する。

以下の3つの初期化は同一。

```C++
type variable = {value};
type variable{value};
type variable({value});
```

よって、例えば以下のように使える。

```C++
#include <iostream>
#include <vector>

int main() {
    int number{3};
    char string[]{'t', 'h', 'r', 'e', 'e'};
    std::vector<double> vector{3, 3.0};

    return 0;
}
```

どういう原理で初期化が行われるかは煩雑なので省略するが、
ここで使われている波括弧の初期化子を*初期化リスト*と呼び、初期化時以外にもいろいろと使うことができる。

初期化リストはstd::initializer_listクラスで実現されており、これをコンストラクタの引数に取ると
ユーザー定義のクラスでも同一の初期化記法を取ることができる。

```C++
#include <iostream>
#include <initializer_list>
#include <sstream>

class magical_girls {
    public:
        std::string members;

        magical_girls(std::initializer_list<std::string> girls) {
            std::stringstream ss;

            for (auto girl: girls) {
                ss << girl << " ";
            }

            members = ss.str();
        }
};

int main() {
    magical_girls madomagi = {"madoka", "sayaka", "mami", "kyouko", "homura"};
    magical_girls nanoha({"nanoha", "fate", "hayate"});
    magical_girls iriya{"iriya", "miyu"};

    std::cout << madomagi.members << std::endl; // -> madoka sayaka mami kyouko homura
    std::cout << nanoha.members << std::endl; // -> nanoha fate hayate
    std::cout << iriya.members << std::endl; // -> iriya miyu

    return 0;
}
```
