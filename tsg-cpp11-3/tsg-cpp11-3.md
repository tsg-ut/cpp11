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

マルチスレッド環境においてスレッドごとに一意な値を保持するらしい。マルチスレッドよくわからない。

```C++
#include <iostream>
#include <thread>

thread_local double pi = 3;

void correct_pi() { pi = 3.14; }

int main() {
    std::thread thread(correct_pi);
    thread.join(); // wait until thread terminates

    std::cout << "pi is " << pi << std::endl; // -> pi is 3
    return 0;
}
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

```C++
```
