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

また、constexprおよびconstなマクロのみで生成されたconstはconstexprとなる。
以下はコンパイルを通る。

```C++
#include <iostream>

#define ONE 1

int main() {
    const int TWO = 2;
    constexpr double CHIHAYA = 72;
    const double PI = ONE + TWO;
    constexpr double RADIUS_OF_CHIHAYA = CHIHAYA / (PI * 2);

    std::cout << RADIUS_OF_CHIHAYA << std::endl;

    return 0;
}
```

### constexpr関数

constexprな値を生成する関数。constexpr内にはreturn文の1文のみが記述されていなければならない。
また、returnする値は
