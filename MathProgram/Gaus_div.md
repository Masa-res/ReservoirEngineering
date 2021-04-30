<script async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js" id="MathJax-script"></script>
# ガウスの発散定理
ガウスの発散定理は連続な1階の偏導関数を有するベクトル場に対して成り立つ定理である。
なお、このホームページではベクトル場の発散は以下のように定義する流儀に従う。
$$\nabla \cdot \boldsymbol{v}=\lim_{V \rightarrow 0}\frac{\int_S \boldsymbol{v} \cdot d\boldsymbol{A}}{V}$$

ベクトル場$\boldsymbol{v}$に対して、曲面$S$で囲まれた体積$V$のある領域$\Omega$の表面上での積分は、
$$\int_S {\boldsymbol{v}\cdot d \boldsymbol{A}}$$
で与えられる。領域$\Omega$を$\Omega_i \ (i=1,2,3,\cdots,n)$という$n$個の小領域に分割し、それぞれの表面を$S_i \ (i=1,2,3,\cdots,n)$
、それぞれの体積を$V_i \ (i=1,2,3,\cdots,n)$とする。

$\Omega_i$と$\Omega_j$に接しているならば、その間の表面積分は打ち消しあうため、
$$\int_S {\boldsymbol{v}\cdot d \boldsymbol{A}}=\sum_{i=1}^n\int_{S_i} {\boldsymbol{v}\cdot d \boldsymbol{A}}=\sum_{i=1}^n \frac{\int_{S_i} {\boldsymbol{v}\cdot d \boldsymbol{A}}}{V_i}V_i$$
となる。$\displaystyle \lim_{n \rightarrow \infty} {\max\{V_i|i=1,2,3,\cdots, n\}}=0$となるように分割を行えば、ベクトル場の発散の定義より、
$$\int_S {\boldsymbol{v}\cdot d \boldsymbol{A}}=\int_\Omega \nabla \cdot \boldsymbol{v}\ dV$$
となる。