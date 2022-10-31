---
title: "関数のカリー化を Python で書いて理解する"
emoji: "📔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Haskell", "関数型プログラミング", "Python"]
published: false
---

# はじめに

Haskell を勉強していて「カリー化」という概念を知った。これを理解するために Python で再現することを試みた。

使用したコードは下記。
[https://github.com/gnkm/cheetsheet/blob/master/python/currying/\_\_main\_\_.py](https://github.com/gnkm/cheetsheet/blob/master/python/currying/__main__.py)

なお、Python 3.10.1 で実行した。

# カリー化とは

『すごい Haskell』から引用する。

> カリー化関数は、複数の引数を取る代わりに、常にちょうど1つの引数を取る関数です。カリー化関数が呼び出されると、その次の引数を受け取る関数を返します。

ＭｉｒａｎＬｉｐｏｖａｃａ. すごいHaskellたのしく学ぼう！ (Japanese Edition) (Kindle の位置No.1463-1465). オーム社. Kindle 版. 

本記事では Haskell における下記 `sum3` がカリー化されて実行される様子を Python で表現する。

```haskell:currying.hs
sum3 :: (Num a) => a -> a -> a -> a
sum3 x y z = x + y + z
```

# Python で書いてみる

## カリー化なし

カリー化は使わずシンプルに書くと、上記 `sum3` は下記のように定義できる。

```python:__main__.py
def sum3_simple(x: int, y: int, z: int) -> int:
    return x + y + z
```

確認する。

```python:__main__.py
x = 1
y = 2
z = 3
val_simple = sum3_simple(x, y, z)
print(f'{val_simple = }')  # => 6
```

## カリー化してみる

カリー化したコードは下記のとおり。

```python:__main__.py
from typing import Callable

def sum3_curried(x: int, y: int, z: int) -> int:
    def f(_x: int) -> Callable:
        def g(_y: int) -> Callable:
            def h(_z: int) -> int:
                return _x + _y + _z
            return h
        return g

    return f(x)(y)(z)
```

下記のように表現すると、構造がわかりやすいか。

```
sum3_curried(x, y, z) = f(x)(y)(z) = g(y)(z) = h(z) = x + y + z
```

`h(z)` が本質の関数。
カリー化なので内部の関数は引数を 1 つしかとらず、コードで書くと `g(y)`, `h(z)` となるが、
例えば `h(z)` なら `x`, `y` が使えるスコープなので、実質は(数学的に表現すると) $h(x, y, z)$ である。
`h(z)` 内では `x`, `y` が固定されているため、関数定義では `z` のみを引数にとることになる。

確認する。

```python:__main__.py
val_curried = sum3_curried(x, y, z)
print(f'{val_curried = }')  # => 6
```

下記のようにも書ける。

```python:__main__.py
sum3_lambda = lambda _x: (
    lambda _y: (
        lambda _z: _x + _y + _z
    )
)
val_lambda = sum3_lambda(x)(y)(z)
print(f'{val_lambda = }')  # => 6
```

実際 `sum3` は下記のシンタックスシュガーであり、これに近い書き方である。

```haskell:currying.hs
sum3Lambda :: (Num a) => a -> a -> a -> a
sum3Lambda = \x -> (\y -> (\z -> x + y + z))
```

# 部分適用

カリー化の利点は部分適用ができることである。
例えば下記 `sum3_12` は、冒頭の `sum3` の 2 個の引数を、それぞれ、1, 2 に固定した関数である。

```haskell:currying.hs
let sum3_12 = sum3 1 2
print $ sum3_12 3  -- => 6
```

Python では下記のようにすることで部分適用を使える。

```python:__main__.py
from functools import partial

sum3_x = partial(sum3_simple, x)
sum3_x_y = partial(sum3_x, y)
val_partial = sum3_x_y(z)
print(f'{val_partial = }')  # => 6
```

# まとめ

$n$ 個の引数をとるカリー化関数は、内部では 1 個の引数をとり $n - 1$ 個の引数をとる関数を返す関数を再帰的に実行している。

下記 `sum3` のカリー化を Python で再現した。

```haskell:currying.hs
sum3 :: (Num a) => a -> a -> a -> a
sum3 x y z = x + y + z
```

Python では下記のように表現できる。

```python:__main__.py
sum3_lambda = lambda _x: (
    lambda _y: (
        lambda _z: _x + _y + _z
    )
)
```

このとき、下記のように評価する。

```python:__main__.py
val_lambda = sum3_lambda(x)(y)(z)
```

また、`functools.partial` を使い Python で部分適用を実現する方法も例示した。
