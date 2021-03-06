﻿# sorting network
二数の小さくない方と大きくない方を出力する、という機能(比較器)のみを使ってソートする。
比較の結果によってその後の比較する対象が変わることがないため、並列に実行でき、また必ず同じ時間で終わる。
並列性能が十分な時、要素が通過する比較器の最大数がかかる時間にかかわり、段数で表す。

## sorting networkが正しく動作するかの証明

- N!通りの入力が全て整列されて出力されることを確かめる(自明)
- 0-1定理(0,1からなる入力2^N通りだけ調べるだけで問題ない)
- ↑それ、探索に無駄があるよ！(フィボナッチ数F(N)~1.618^N通り程度調べれば網羅できる)(黒田, 1997)

## 汎用アルゴリズム(要素数に関係なく構成できる)
### AKSネットワーク

- 多少のエラーを出すかもしれないネットワークを何回もくぐらせるとエラーしなくなる、という方針で構成
- Ο(NlogN)回の比較、Ο(logN)段でできる
    - ……が、その定数部分はあまりにも大きく、実用的ではない

### バイトニックソート

- バイトニック列(前半分は昇順、後ろ半分が降順)をマージするアルゴリズムを用いたマージソート
- マージにはΟ(NlogN)回の比較とO(logN)段必要なので、結果的にΟ(N(logN)^2)回の比較とΟ((logN)^2)段のアルゴリズム
- バイトニック列を実際に形成するのではなく比較位置を変えた方法は鋸ソートと呼ばれ、ソート済み列が与えられたときに無駄なswapが生じず効率的になるが、メモリアクセスが複雑になる
-要素数がN=2^nの時、n(n+1)/2段で、比較器の数はn(n+1)N/4個

### Batcher奇偶マージソート

- バイトニックソート同様にΟ(NlogN)回の比較とO(logN)段でマージする
- バイトニックソートと比べ、マージ方法が工夫されていて比較回数がやや少ない(段数は同じ)
- 要素数がN=2^nの時、n(n+1)/2段で、比較器の数は(n^2-n+4)N/2-1個

### シェルソート

- 一見普通のソートアルゴリズムのようだが、sorting networkになっている
- 2^p×3^qとなる各数を間隔として採用すると、比較回数はΟ(N(logN)^2)、Ο((logN)^2)段
- 段数最小と比較回数最小が両立しない(?)
### バブルソート・挿入ソート

- 普通のソートアルゴリズムだが、比較結果によって動作が変わることがないのでこれもsorting networkになる
- Ο(N)段、Ο(N^2)回の比較なのでsorting networkとしての性能はよくない

### Zig-Zag sort

- 論文読み中……
- Ο(NlogN)回の比較でソートできるらしい
- シェルソート類似アルゴリズムらしい
- ε-halverを使っている関係でやっぱり定数が大きい

## 各Nに対する既知の最良sorting network
?がついているのは下限であると示されていないもの
「*」は論文中に直接記載がなかったので数えたりしたもの
「@」は既知の最小段数のものを削ることで構成したsorting networkの比較器数

| N | 比較器最小 | 段数最小 | 参考:Batcher奇偶マージソート |
|---|---|---|---|
| 2 | 1比較1段 | 同左 | 同左 |
| 3 | 3比較3段 | 同左 | 同左 |
| 4 | 5比較3段 | 同左 | 同左 |
| 5 | 9比較5段 | 同左 | 同左 |
| 6 | 12比較5段 | 同左 | 12比較 ***6*** 段|
| 7 | 16比較6段 | 同左 | 同左 |
| 8 | 19比較6段 | 同左 | 同左(ここまでは比較器の数はbestと同じ) |
| 9 | 25比較(8段?) | 7段(27比較@) | 26比較8段 |
| 10 | 29比較(9段?) | 7段(31比較?) | 31比較9段 |
| 11 | 35比較?(9段?) | 8段(36比較@) | 37比較10段 |
| 12 | 39比較?(9段?) | 8段(40比較?) | 41比較10段 |
| 13 | 45比較?(10段?) | 9段(47比較@) | 48比較10段 |
| 14 | 51比較?(10段?) | 9段(52比較@) | 53比較10段 |
| 15 | 56比較?(10段?) | 9段(57比較@) | 59比較10段 |
| 16 | 60比較?(10段?) | 9段(61比較?) | 63比較10段 |
| 17 | 71比較?(16段?*) | 10段(79比較?*) | 74比較12段 |
| 18 | 78比較?(14段?*) | 11段?(85比較@) | 82比較13段 |
| 19 | 86比較?(14段?*) | 11段?(94比較@) | 91比較14段 |
| 20 | 92比較?(13段?*) | 11段?(102比較?*) | 97比較14段 |
| 21 | 102比較?(19段?*) | | 107比較15段 |
| 22 | 108比較?(14段?*) | | 114比較15段 |
| 23 | 118比較?(21段?*) | | 122比較15段 |


- [1] THE ART OF COMPUTER PROGRAMMING
    - 古典的な結果を参照した
- [2] http://arxiv.org/abs/1501.06946
    - 最新、N=17で10段、N=20で11段を構成した
    - N=17～20で最低でも10段は必要なことを示した
- [3] http://arxiv.org/abs/1310.6271
    - N=11～16でそれぞれ8段、9段は必要なことを示した
- [4] http://www.genetic-programming.org/hc2011/03-Valsalam/Valsalam-Slides.pdf
- [5] http://nn.cs.utexas.edu/downloads/papers/valsalam.jmlr13.pdf
    - 手つかずだった(?)N=16～の領域でより良いものを構成した
- [6] https://arxiv.org/abs/1405.5754
    - N=9, 10で比較器がそれぞれ25個、29個は必要なことを示した
- [7] http://pages.ripco.net/~jgamble/nw.html
    - N=2^n以外の時でもBatcher奇偶マージソートを構成
    - あっているのか、最適なのか、は未確認





