# C++11分科会 第4回資料
担当: arcturu

## クラス
[class/struct/enum] \(identifier\) \(final\) \(: (access-specifier) base\) { \(members\) };

- structは暗黙的にpublic
- classは暗黙的にprivate
- クラス名は直後から使える

```cpp11
class Foo {} *ptr = static_cast<Foo *>(nullptr);
```


### 1. C++11 での POD \(Plain Old Data\)

#### POD とは
　Cの整数型、文字型、構造体、... のように連続したバイトデータで表現できる型のこと。C++におけるクラスでも、それがPODクラスであればCのバイト操作関数 \(memcpyなど\) が正しく使えることが保証される。つまりCとC++が混在する状況下でも矛盾なく動作することが保証される。（逆にPODでないクラスは連続したメモリ上にデータが存在するとは限らない）

#### あるクラスがPODクラスであるための条件 \(C++11\)
- トリビアルクラスであること
- 標準レイアウトクラスであること
- 非PODな非staticデータメンバーを持たないこと

 #### トリビアルクラス \(trivial class\)
 非トリビアルな（ユーザー定義の）

 - コンストラクタ
 - コピーコンストラクタ
 - ムーブコンストラクタ
 - コピー代入演算子
 - ムーブ代入演算子

 を持たず、トリビアルなデストラクタ を持っているクラスをトリビアルクラスという。

 #### 標準レイアウトクラス \(standard layout class\)

 - 標準レイアウトクラスではない非staticメンバー
 - 標準レイアウトクラスではないクラスの配列、参照
 - 仮想関数
 - 仮想基本クラス

 を持たないクラスを標準レイアウトクラスという。

#### トリビアルにコピー可能なクラス \(trivially copyable class\)
　ユーザー定義のコンストラクタを持っているものの、それ以外のトリビアルクラスの条件は満たしているクラスはトリビアルにコピー可能なクラスといい、memcpy によって正しくコピーできることが保証される。

### 2. レイアウト互換 \(layout-compatible\)
2つのクラスA, Bが以下のうちいずれか1つの条件を満たせば、AとBはレイアウト互換であるという。

- A, Bは同じ型である
- A, Bはともに、標準レイアウトのstructまたはunionで、同じ数の非staticデータメンバーを持ち、対応する非staticデータメンバーがそれぞれレイアウト互換である

### 3. unionのようなクラス \(union-like class\) の注意点
union、もしくは無名unionを直接のメンバーに持つクラスを、unionのようなクラス \(union-like class\) という。


```cpp11
union U{
	std::vector<int> v;
	std::string s;
};

int main(){
	U u; // error
}
```

これはコンパイル通らない。というのは、unionのようなクラスの共用メンバが非trivialなコンストラクター、コピーコンストラクター、ムーブコンストラクター、コピー代入演算子、ムーブ代入演算子、デストラクターを持っているとそれに対応するunionのメンバーが暗黙的にdeleteされるため。上の例のunionは明示的には

```cpp11
union U{
	std::vector<int> v;
	std::string s;
	U() = delete;
	~U() = delete;
};
```

となっている。

```cpp11
union U{
	std::vector<int> v;
	std::string s;
	U(){}
	~U(){}
};

int main(){
	U u; // ok
}
```

このように自分でdeleteされている部分を定義する必要がある。unionのようなクラス以外にもコンストラクタなどがdeleteされる条件があるので気をつける。->後述

## 派生クラス

###1. final 指定子
```cpp11
class Base {
public:
	virtual void foo() final {}
};

class Derived: Base {
public:
	virtual void foo(){} // error
};
```

finalの指定された関数はオーバーライドできない。

```cpp11
class Base final {
public:
	int foo;
};

class Derived: Base { // error
public:
	int bar(){ return foo; }
};
```

finalの指定されたクラスから派生することはできない。

### 2. override 指定子
```cpp11
class Base {
	virtual int foobar(){}
};

class Derived: Base {
	virtual int foobae() override {} // error
};
```

overrideを指定した関数は必ずオーバーライドしなければならないため、タイプミスによるバグをコンパイル時に発見できる。
## メンバーのアクセス指定
特に変わらない。
## 特別なメンバー関数
デフォルトコンストラクター、コピーコンストラクター、コピー代入演算子、ムーブコンストラクター、ムーブ代入演算子、デストラクターを特別なメンバー関数と呼ぶ。

### 1. 暗黙的にdeleteされる場合がある
普通、明示的に特別なメンバ関数を指定しなければこれらは暗黙的に宣言される。しかし、ある条件下ではそれらがdeleteされるため注意が必要である。

```cpp11
class NonTrivial {
	NonTrivial(); // ユーザー定義コンストラクタ
};

union Foo {
	NonTrivial bar; // 非トリビアルクラス
	// Foo() = delete;
	// unionのようなクラスの共用メンバに非トリビアルクラスがあると
	// 対応する特別なメンバ関数がdeleteされる。（上述）
};

class Bar {
	int &ref; // 初期化子のない参照型の非staticメンバ
	// Bar() = delete;
};
```

このようなケースはやや多く煩雑なため詳細は[12章 特別なメンバー関数 \(Special member functions\)]( http://ezoeryou.github.io/cpp-book/C++11-Syntax-and-Feature.xhtml#special )を参照のこと。

### 2. コンストラクタのデリゲート
メンバー初期化識別子にクラス型を書くことでそのコンストラクタへ初期化処理をデリゲートすることができる。
デリゲートするコンストラクタをデリゲートコンストラクタ、
デリゲートされるコンストラクタをターゲットコンストラクタという。

```cpp03
class X
{
public :
	X()
	{
		// 共通の初期化処理
    }

	X( int value ) 
	{
		// 共通の初期化処理
		// 引数を使った追加の処理
	}
} ;
```

これが

```cpp11
class X
{
public :
	X()
	{
		// 初期化処理
	}

	X( int value ): X() // デリゲートコンストラクタ
	{
		// 引数を使った追加の処理
	}
} ;
```

こうできる。

```cpp11
#include <iostream>
using namespace std;

class X {
	int val;
public:
	X(): X(0) { cout << "init without argument" << endl; }
	X(int i): val(i) { cout << "common init" << endl; }
};

int main(){
	X x1;
	X x2(123);
	return 0;
}
```

\(実行結果\)
> common init
> init without argument
> common init

デリゲートコンストラクタと同時にメンバー初期化識別子を書くとコンパイルエラーになる。

```cpp11
class X {
	int val
public:
	X(): X(0), val(0) { cout << "init without argument" << endl; } // error
	X(int i) { cout << "common init" << endl; }
};
```

### 3. 右辺値参照、ムーブセマンティクス
C++11の目玉機能のひとつ。

#### 右辺値 \(rvalue\)
type &&型で表される。右辺値とはリテラルや、非参照型の関数の戻り値などの

```cpp11
int &num = 1; // error
// ok: const int &num = 1;
```

こういうことをすると怒られるものの総称。

#### 右辺値参照 \(rvalue reference\)
今までの普通の参照を左辺値参照と呼ぶこともある。

```cpp11
int &a = pow(2,5);        // error: 右辺値(一時オブジェクト)は左辺値参照できない
const int &b = pow(2,5); // ok: しかしbは変更不可
b = pow(3,5);             // error
int &&c = pow(2,5);       // ok: 右辺値参照型への代入
c = pow(3,5);             // ok
c = b;                    // error: 左辺値参照はそのままでは右辺値参照に代入できない
c = std::move(b);         // ok: ムーブ代入演算子。内部的にはキャスト
// bは"有効だが規定されない状態(valid but unspecified state)"
// 後で説明
```

#### ムーブセマンティクス
（コピー）コンストラクタで大変な処理をするクラスはできるだけコピーしたくない。
　　　　　　　　　　↓
大変な部分のデータはだいたいポインタや配列で保持しているから、そのポインタだけ受け渡すことでコピーのコストを削れないか？
　　　　　　　　　　↓
例えば後で定義するMyDataクラスを\(ユーザー定義ムーブ代入演算子をコメントアウトして\)使うと

```cpp11
MyData foo(){
	MyData tmp;

	// process

	return tmp;
}

int main(){
	MyData x = foo(); // コピーが発生
}
```

MyDataは1KBくらい使い、コピーがもったいない。
　　　　　　　　　　↓
foo()から返ってきたMyData一時オブジェクトは代入の後すぐ用済みになるのだから、そのうちfoo().ptrだけを新しいオブジェクトのptr、つまり現在のx.ptrをdeleteしてfoo().ptrをそこに代入すればcopy()しないですむ。foo().ptrには代わりにnullptrを入れておけばdeleteされても何も起きないから、そちらがいつ消えようがxには関係がなくなる。
　　　　　　　　　　↓
要するに、右辺は破壊されるが高速に移動できるような仕組みがあれば性能の底上げにつながる。
とはいえコピー代入時に右辺値が常に破壊されるのは困るので、コピー代入演算子では対応できない。\(破壊していいリテラルや関数の戻り値などの一時オブジェクトと、そうでない変数などの区別ができない\)
　　　　　　　　　　↓
　　　　　　＿人人人人人人人人＿ 
　　　　　　＞　右辺値参照型　＜
　　　　　　￣ＹＹＹＹＹＹＹＹ￣
#### サンプル

```cpp11
#include <iostream>
#include <cstdio>
#include <cmath>
using namespace std;

class MyData {
public:
	int dummy;
	char *ptr;

	MyData(){
		ptr = new char[1000];
		printf("constructor new %p\n", ptr);
	}
	MyData(char * str){
		ptr = new char[1000];
		strcpy(ptr, str);
		printf("constructor new %p %s\n", ptr, ptr);
	}
	MyData(const MyData& r){
		ptr = new char[1000];
		copy(&ptr[0], &ptr[1000], &r.ptr[0]);
		printf("copy constructor new %p\n", ptr);
	}
	MyData& operator=(const MyData& r){
		printf("copy assignment\n");
		copy(&ptr[0], &ptr[1000], &r.ptr[0]);
		return *this;
	}
	MyData(MyData&& r){
		printf("move constructor\n");
		ptr = r.ptr; // そもそもnewされない
		r.ptr = nullptr;
	}
	MyData& operator=(MyData&& r){
		printf("move assignment: delete %p\n", ptr);
		if(this == &r){
			return *this;
		}
		delete [] ptr;
		ptr = r.ptr;
		r.ptr = nullptr;
		return *this;
	}
	~MyData(){
		printf("destructor delete %p\n", ptr);
		delete [] ptr;
	}
};

MyData foo(){
	MyData tmp("in foo()");
	return tmp;
}

int main(){
	MyData x("in main()");
	x = foo(); // foo()の戻り値はrvalueなのでデフォルトでmove
	x = static_cast<const MyData&>(foo()); // 意図的にコピー代入演算子を呼ぶ
	MyData y("in main() 2");
	MyData z("in main() 3");
	z = y; // yはlvalueなのでデフォルトはコピー
	z = move(y); // std::move()によってyを "右辺値にキャスト"
	// yは "有効だが規定されない状態"
	// つまりy.dummyなどmoveに関係していないメンバやy.ptr自身にはアクセスできるが、
	// y.ptr[10]というようなmoveに関係するポインタの中身はどうなっているかわからないということ
}
```

\(実行結果\)

>
constructor new 0x7fbc7b403940 in main()
constructor new 0x7fbc7b403d30 in foo()
move assignment: delete 0x7fbc7b403940
destructor delete 0x0
constructor new 0x7fbc7b403940 in foo()
copy assignment
destructor delete 0x7fbc7b403940
constructor new 0x7fbc7b403940 in main() 2
constructor new 0x7fbc7b404120 in main() 3
copy assignment
move assignment: delete 0x7fbc7b404120
destructor delete 0x7fbc7b403940
destructor delete 0x0
destructor delete 0x7fbc7b403d30

### 4. コンストラクタの継承

```cpp11
class Base {
public:
	Base(int i){/* process */}
}

class Derived: Base {
public:
	using Base::Base;
	// これは
	// Derived(int i): Base(i) {/* process */}
	// と書いた場合と同等
}
```

#### 注意点
- 基底クラスの暗黙的に宣言されるコンストラクタは継承されない
- 多重継承すると曖昧さが生じる

## 参考
- [C++11の文法と機能\(C++11: Syntax and Feature\)]( http://ezoeryou.github.io/cpp-book/C++11-Syntax-and-Feature.xhtml )

- [Plain Old Data](http://cxx11.blogspot.jp/2011/12/plain-old-data.html)

- [C++98 C++03 の POD](http://tu3.jp/01004)

- [本当は怖くないムーブセマンティクス](http://yohhoy.hatenablog.jp/entry/2012/12/15/120839)

- [右辺値参照とムーブコンストラクタの使い方](http://program.station.ez-net.jp/special/handbook/cpp/syntax/move.asp)

- [本の虫: rvalue reference 完全解説](http://cpplover.blogspot.jp/2009/11/rvalue-reference_23.html)
