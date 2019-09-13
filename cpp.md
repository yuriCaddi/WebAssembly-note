# C++でWebAssembly

# 大まかな流れだけ

## インストール

インストールの方法は、[ここ](https://developer.mozilla.org/ja/docs/WebAssembly/C_to_wasm)に書いてある方法で大丈夫。  
aptにもemscriptenが存在するらしいけど、上手く動作しないって色々なところに書いてあった。(試してないけど)  
このインストール方法だと、起動毎にスクリプトを走らせる必要があるので、どうにかすべき。

## C++ソースの作成

```cpp
// quick_example.cpp
#include <emscripten/bind.h>

using namespace emscripten;

float lerp(float a, float b, float t) {
    return (1 - t) * a + t * b;
}

EMSCRIPTEN_BINDINGS(my_module) {
    function("lerp", &lerp);
}
```

```EMSCRIPTEN_BINDINGS``` ブロックの中で、 ```JavaScript``` に ```lerp() function()``` を公開する。

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
      onRuntimeInitialized: function() {
        console.log('lerp result: ' + Module.lerp(1, 2, 0.5));
      }
    };
  </script>
  <script src="quick_example.js"></script>
</html>
```
これを、 ```sample.html``` とする。

## 実行

```
emrun sample.html
```

## 参考

だいたい、[ここ](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html#a-quick-example)に書いてある流れと同じ。  
```emrun``` についての記述のみ、[こっち](https://emscripten.org/docs/compiling/Running-html-files-with-emrun.html#running-html-files-with-emrun)に書いてある。(私は読んでない)  



# なにをしているの？

## あらかじめ

調べて書いてるつもりだけど、間違っている可能性が有ります。

## ながれ

1. 

## 注意

- 対応しているC++のバージョンが、C++11とC++14だけ。
- 生ポインタを扱う際、```allow_raw_pointers```でマークする必要がある。