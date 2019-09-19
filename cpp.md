
# 使い方

詳しく知りたい人へ -> [Embind](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html#)  
よくつかいそうなところだけまとめ。

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
      console.log(Module.c_hello())
      console.log('hello result:' + Module.c_add(10, 1));
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


# 構造体・スマートポインタ

多分使えるけど、多少の決まり事があるっぽい？

## 参考
[Embind - Value types](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html#value-types)

# 注意

- 対応しているC++のバージョンが、C++11とC++14だけ。
- 生ポインタを扱う際、```allow_raw_pointers```でマークする必要がある。