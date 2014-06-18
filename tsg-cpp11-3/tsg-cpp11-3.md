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

## static_assert

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
