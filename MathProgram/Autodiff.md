# 自動微分と応用例
油層工学をはじめ工学を学ぶ人、エンジニアとして働く人は、多かれ少なかれ微分係数を求めなければならない状況になることがあるのではないだろうか。（個人的意見です）

微分係数を求めるためには一般的に、以下の方法が使用される。
- 手計算で導関数を求め、微分係数を算出する。
- 数値微分を利用する。
- 数式微分を利用し、導関数を求めて微分係数を算出する。
- 自動微分を利用する。

今回注目するのは自動微分である。自動微分は「手計算で導関数を求めなくてよい」、「数値微分のように誤差が生じる心配がない」、「数式微分のように効率が悪くなることがない」方法である。

自動微分は原理は難しくないものの、調べても理解するのに時間がかかるものかと考えている。
そのため、世の中に便利な自動微分ライブラリはあるが、できるだけ簡単なプログラムでの計算例を示す。

なお、今回扱う自動微分はボトムアップ型自動微分（フォワードモード）であり、pythonでの例を示す。（pythonの経験が浅いため、もっと良い書き方があるかもしれません）

## ボトムアップ型自動微分の原理
ボトムアップ型自動微分では通常の計算に加えて微分係数の計算を同時に行う。
パソコンで計算を行う際、$f+g$であれば通常は$f$と$g$の和を計算するのみであるが、$f$と$g$の微分係数が既知であれば、$f+g$の微分係数は$f^\prime+g^\prime$のように簡単に算出できる。自動微分ではこの性質を利用する。

初めに、$f$が$X$の一変数関数でかつ四則演算で表されることを考える。
微分係数が既知であれば足し算した時の微分係数が簡単に求められるという性質を利用するために、
$X$に関数値の計算を行うための変数$x$と微分係数の計算を行うための変数$dx$を持たせる。ここではそれぞれ$X_x$、$X_{dx}$と表記する。
$X$が3の時の計算を行う際は通常は$X=3$とするが、ここでは、$X_x=3$、$X_{dx}=1$とする。
ここで$X_{dx}=1$とするのは、$X$の$X$による微分係数は$1$だからである。

このように定義した$X$に対して以下の表に従って演算を行わせる。

| プログラムでの記述 | 関数値 | 微分係数の計算 |
| --- | --- | --- |
|$$f+g$$|$$f_x+g_x$$|$$f_{dx}+g_{dx}$$|
|$$f-g$$|$$f_x-g_x$$|$$f_{dx}-g_{dx}$$|
|$$f*g$$|$$f_x \times g_x$$|$$f_{dx} \times g_x+f_x \times g_{dx}$$|
|$$\frac{f}{g}$$|$$\frac{f_x}{g_x}$$|$$\frac{f_{dx} g_x - f_xg_{dx}}{f^2}$$|

例えば$y=X+X$を計算する際には、
$$y_x=X_x+X_x$$
$$y_{dx}=X_{dx}+X_{dx}$$
のようにして$y$を求める。

$y=X\times X+X$であれば初めに
$$\left(X\times X \right)_x=X_x \times X_x$$
$$\left(X\times X \right)_{dx}=X_{dx} \times X_x + X_x \times X_{dx}$$
を計算し、
$$y_x=\left(X\times X \right)_x+X_x$$
$$y_{dx}=\left(X\times X \right)_{dx}+X_{dx}$$
のように$y$を算出する。

これをプログラムで実現するために「通常の計算を行う用の変数」と「微分係数の計算を行う変数」の2種類の変数を持つ新たな型を用意する。
その方に対し和、差、積、商の計算を行う関数を作成し、演算子「+ - * /」がそれぞれの関数を呼び出すようにする（演算子多重定義）。
また、$\sin$や$\cos$、$\log$に対しても応用可能であり、本ページのプログラムでは実装しているので参考になるかと思う。
以上がボトムアップ型自動微分の原理である。

ちなみにこの手法は多変数関数の際にも応用可能である。（$n$変数関数であれば単に$dx$を$n$個用意すれば良い）


```python
import numpy as np

class ad: # adというクラスの作成
    def __init__(self, x, dx=1.0):
        self.x = x    # 関数値
        self.dx = dx  # 微分値
    
    # Jupyter labなどで使用する際に値の出力を可能にするためのメソッド
    def __repr__(self):
        return "f = " + str(self.x) + "\n" + "df = " + str(self.dx)
    
    # 関数値だけ取り出すためのメソッド
    def outf(self):
        return self.x
    # 微分値だけ取り出すためのメソッド
    def outdf(self):
        return self.dx
    
    # 和の演算子多重定義
    def __add__(self, other):
        if isinstance(other, ad):
            return ad(self.x + other.x, self.dx + other.dx)
        else:
            return ad(self.x + other, self.dx)
    def __radd__(self, other):
        return ad(self.x + other, self.dx)
    
    # 差の演算子多重定義
    def __sub__(self, other):
        if isinstance(other, ad):
            return ad(self.x - other.x, self.dx - other.dx)
        else:
            return ad(self.x - other, self.dx)
    def __rsub__(self, other):
        return ad(other - self.x, -self.dx)
    
    # 積の演算子多重定義
    def __mul__(self, other):
        if isinstance(other, ad):
            return ad(self.x * other.x, self.x * other.dx + self.dx * other.x)
        else:
            return ad(self.x * other, self.dx * other)
    def __rmul__(self, other):
        return ad(self.x * other, self.dx * other)
    
    # 商の演算子多重定義
    def __truediv__(self, other):
        if isinstance(other, ad):
            return ad(self.x / other.x, (self.dx * other.x - self.x * other.dx) / (other.x * other.x))
        else:
            return ad(self.x / other, self.dx / other)
    def __rtruediv__(self, other):
        return ad(other / self.x, -self.dx * other / (self.x * self.x))
    
    # **の演算子多重定義
    def __pow__(self, other):
        return exp(other * log(self))
    def __rpow__(self, other):
        return exp(self * log(other))
    # -の演算子多重定義（y=-xなどに対応できる）
    def __neg__(self):
        return ad(-self.x, -self.dx)

def sin(self):
    if isinstance(self, ad):
        return ad(np.sin(self.x), np.cos(self.x) * self.dx)
    else:
        return np.sin(self)

def cos(self):
    if isinstance(self, ad):
        return ad(np.cos(self.x), -np.sin(self.x) * self.dx)
    else:
        return np.cos(self)

def tan(self):
    return sin(self)/cos(self)

def exp(self):
    if isinstance(self, ad):
        return ad(np.exp(self.x), np.exp(self.x) * self.dx)
    else:
        return np.exp(self)

def log(self):
    if isinstance(self, ad):
        return ad(np.log(self.x), self.dx / self.x)
    else:
        return np.log(self)

def sqrt(self):
    return self ** 0.5 
```

## 使用例
$\displaystyle x=3$


```python
x = ad(3.0)
```

xと入れるだけでjupyter labなどでは値を出力してくれる。（__repr__で設定しているため）


```python
x
```




    f = 3.0
    df = 1.0



$y=\sin{x^2}$、$y^\prime=2x\cos{x^2}$


```python
y = sin(x ** 2)
```


```python
print(np.sin(3.0 ** 2))
print(2 * 3.0 * np.cos(3.0 ** 2))
print(y)
```

    0.4121184852417566
    -5.466781571308061
    f = 0.41211848524175493
    df = -5.466781571308067
    

## 応用例　ニュートン法
$$\cos{x}+(x+1)e^x = 0$$


```python
x = ad(1.0)
delta_x = 1.0
while abs(delta_x) > 1e-6:
    y = cos(x) + (x + 1) * exp(x)
    f = y.outf(); df = y.outdf()
    delta_x = -f / df
    x = ad(x.outf() + delta_x)
print("x=" + str(x.outf()))
```

    x=-1.4633436590068039
    
