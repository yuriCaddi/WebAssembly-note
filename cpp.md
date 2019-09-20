
# 使い方

詳しく知りたい人へ -> [Embind](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html#)  
よくつかいそうなところだけまとめ。実質防備録なので、自分が使う場合のやり方しかかいてない。

## インストール

インストールの方法は、[ここ](https://developer.mozilla.org/ja/docs/WebAssembly/C_to_wasm)に書いてある方法で大丈夫。  
aptにもemscriptenが存在するらしいけど、上手く動作しないって色々なところに書いてあった。(試してないけど)  
このインストール方法だと、起動毎にスクリプトを走らせる必要があるので、どうにかすべき。


```
git clone https://github.com/juj/emsdk.git
cd emsdk

# on Linux or Mac OS X
./emsdk install --build=Release sdk-incoming-64bit binaryen-master-64bit
./emsdk activate --global --build=Release sdk-incoming-64bit binaryen-master-64bit
```
```emsdk``` ディレクトリ内で次のコマンドを入力
```
# on Linux or Mac OS X
source ./emsdk_env.sh
```

## C++ソースの作成

```cpp
// quick_example.cpp
#include <iostream>
#include <string>
#include <emscripten/bind.h>
using namespace emscripten;

std::string Hello() {
    return("Hello world from C++");
}

int Add(int a, int b) {
    return(a + b);
}

EMSCRIPTEN_BINDINGS(my_module) {
    function("c_hello", &Hello);
    function("c_add", &Add);
}
```

```EMSCRIPTEN_BINDINGS``` ブロックの中で、 ```Hello()``` と ```Add()``` をエクスポートする。  
```""``` の中で、 ```JavaScript``` からの呼び出しに使う名前を決めることができる。

## コンパイル

```
emcc --emrun --bind -o quick_example.js quick_example.cpp
```

```quick_example.js``` で、グルーコードの名前を指定できる。  
```quick_example.cpp``` には、アプリケーションのコードを指定する。  

```--emrun```をつけるのは、じっこうするときに ```emrun``` コマンドを使うってだけ。  
セキュリティの関係？で、ただただ ```html``` 開いても、実行できない。

## 呼び出し

```html
<!doctype html>
<html>
<script>
  var Module = {
    onRuntimeInitialized: function () {
      console.log(Module.c_hello()) // Hello world from C++
      console.log('hello result:' + Module.c_add(10, 1)); // 11
    }
  };
</script>
<script src="quick_example.js"></script>
</html>
```
これを、 ```sample.html``` とする。

```Module``` オブジェクトで使用可能できる。  
引数も、普通に持たせることができる。

## 実行

```
emrun sample.html
```

## 参考

[Embind - A quick example](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html#a-quick-example)  
[Running HTML files with emrun](https://emscripten.org/docs/compiling/Running-html-files-with-emrun.html#running-html-files-with-emrun)

# クラス

```cpp
class MyClass {
public:
  MyClass(int x, std::string y)
    : x(x)
    , y(y)
  {}

  void incrementX() {
    ++x;
  }

  int getX() const { return x; }
  void setX(int x_) { x = x_; }

  static std::string getStringFromInstance(const MyClass& instance) {
    return instance.y;
  }

private:
  int x;
  std::string y;
};

// Binding code
EMSCRIPTEN_BINDINGS(my_class_example) {
  class_<MyClass>("MyClass")
    .constructor<int, std::string>()
    .function("incrementX", &MyClass::incrementX)
    .property("x", &MyClass::getX, &MyClass::setX)
    .class_function("getStringFromInstance", &MyClass::getStringFromInstance)
    ;
}
```

クラスをエクスポートする場合、まず ```class_``` にクラス名をもたせて、宣言する。

関数をエクスポートするには、下の関数がある。

- constructor()  
  コンストラクタをエクスポートするときに使う。
- function()  
  メンバ関数をエクスポートするときに使う。
- class_function()  
  静的メンバ関数をエクスポートするときに使う。
- property()
  バインディングを定義する。

```html
<script>
  var Module = {
    onRuntimeInitialized: function () {
      var instance = new Module.MyClass(10, "hello");
      /***************
       * x = 10
       * s = "hello"
       ***************/

      instance.incrementX(); // x++
      console.log("instance.x :" + instance.x); // 11

      instance.x = 20;
      console.log("instance.x :" + instance.x); // 20

      console.log(Module.MyClass.getStringFromInstance(instance)); // hello

      instance.delete();
    }
  };
</script>
<script src="quick_example.js"></script>
```

# 構造体・スマートポインタ

多分使えるけど、多少の決まり事があるっぽい？

## 参考
[Embind - Value types](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html#value-types)

# 注意

- 対応しているC++のバージョンが、C++11とC++14だけ。
- 生ポインタを扱う際、```allow_raw_pointers```でマークする必要がある。

# サンプル

[それっぽいもの](https://github.com/yuriCaddi/wasm-cpp-sample)