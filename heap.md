﻿いろいろな種類のヒープ
========
ヒープは秩序を持った木構造であり、次の制約が課される(ここでは最小ヒープを考えるが、最大ヒープも全く同じ)。

- ある要素は、(子を持てば)子要素のいずれよりも小さい。

この制約から、木のルートには木の要素の中の最小値がある、ということが導かれる。
それ以外の部分、特にいとこかそれより離れた関係にある要素には全く秩序が入っていないため、二分検索木などと違って任意の値を検索することには使えないが、必要のない秩序を入れていない(後回しにしたともいう)分、特化した機能には軒並み高速な性能を示す。
ヒープと普通言ったときには、

- 最小値要素の検索と取り出し
- 要素の追加

の二つが高速(Ω(logN)以内。ソートに使えることを考えれば、どちらかは最低でもΟ(logN)かかるのは当然)なことが要求される。
オプショナルとして、二つのヒープのマージ、指定要素のキー値の変化(普通は小さくするのみ？)、指定要素の削除、なども高速にできるヒープもある。

## 二分ヒープ

- もっとも単純なヒープ
   - 挿入と最小値削除にΘ(logN)のコスト
   - マージには多大なコストがかかるため、事実上使えない

- 挿入は、末尾に挿入したい要素を付け加え、次の「ヒープの調整(下→上)」を行う
- 削除は、先頭を削除し、そこを末尾から持ってきた要素で埋めてから、「ヒープの調整(上→下)」を行う

- ヒープの調整は、 変更のあったところ周辺がおかしくなっている可能性があるので、適宜要素を入れ替えて部分的な修正をすればOK。もちろん、この修正が他の部分をおかしくする可能性があるのだけれど、その波及の回数は高々ヒープの高さ＝Θ(logN)である。

- 領域は最小量で済ませられる(配列以外に管理情報がいらない)
- 量が前もってわかっていない場合は、挿入にかかるコストがΩ(logN)より小さい動的配列を使う必要があるので、それのために余分な領域を使うことはある。
- もちろん、普通に連結リストで実装してもオーダーは変わらない。

## 二項ヒープ

- ヒープ木を複数持つことで柔軟性が上昇
- ヒープのマージもΘ(logN)でできるようになった！
- ヒープへの要素挿入は最悪Ο(logN)かかることがあるものの、最初に連続して挿入する場合は償却時間としてはΘ(1)で可能！
   - 最悪のパターンは、要素数を2進数で表した時に繰り上がりと繰り下がりが頻発するときで、2^k-1個入っているヒープに要素を挿入しては最小値要素を取り出す、みたいな場合

- ヒープの調整は、ルートがなくなってばらばらになった子ヒープとのマージとして考えるだけ

## フィボナッチヒープ

- ヒープ木の個数は何個でもいいことにしたので柔軟性がさらに上昇
- モットーは「明日できることを今日やらない」、と怠惰に見せかけて「将来のために貯金」、と堅実さも見せる(?)
- キー値の減算(ヒープの上の方に動かす)は上の二つのヒープではΘ(logN)かかっていたのだけれど、それを償却時間としてΘ(1)でできるようにしてしまった！
   - 実は"償却時間"なのがカラクリであり、実際には最悪Ω(N)かけて、しかもヒープを壊しながら行うこともあるのだけれど、これの実行と、壊れたヒープの修正のコストは以前キー値減算した回数より少ないようになっているため、キー値減算するのにかかった時間を余分に計上して、問題(たくさん破壊してしまった)が発生するまでは時間を得している、問題が起きてもプラスマイナス0で損はしていない、ということにしている
      - 理由：マークされたノードはキー値減算ごとに高々一個しか増えない。

   - この「問題が発生するまでは時間を得したことにしておく」という考え方で、挿入を素早く行うことができる(最小値を探すのに時間がかかることが目に見えているが、問題を先送りにしておく)
   - ちなみに要素の削除は、キー値を無限に減らしたあと最小値削除、として実装可能なので、償却時間としてΘ(logN)で実行可能

- ヒープのマージは何の工夫もいらない。なぜなら、ヒープの木の数は特に規定されていないから
- 挿入はその要素だけからなるヒープとのマージと考えればいい(ここは二項ヒープも同じ)
- こんなことをやっているとどんどん木の数が増えていくのだが、いずれ最小値削除をやるはずで、その時に形を整えることになる

- 面倒なことをすべて押し付けられた最小値削除が非常に複雑
   0. まず、最小値要素を探す。これは、挿入時に順次比較していき、最小値要素を記憶しておけばΘ(1)でできる。
   0. 最小値要素をヒープから切り離す。子がd個あったとすると、Θ(d)の時間で切り離しができる。フィボナッチヒープでは、子の数はΩ(logN)で抑えられることになっているから、この操作はΘ(logN)で終了する。
   0. 残ったルート全部(最悪N個近くある)から最小値を探し出さないといけない。得た情報を有効活用するため、同時に木を整形していく。まず、同じ次数のルートノード同士を合わせて新しいヒープ木にし、同じ次数のルートノードがなくなるまで繰り返す(同じ次数のルートノードはどう発見するか？"整っている"ルートノードの個数はΘ(logN)しかないのでこれの発見はΘ(logN)の領域にメモしておけばよい。"整っていない"ルートノードは全部消し去るので、前から見てけばいい)。この操作には最悪Ω(N)の時間がかかるのだが、挿入したときなどに何もしていなかったツケが回ってきているだけであり、その時の"貯金"を消費すればΘ(logN)で作業をやり遂げたことになる。
   0. 最後に、ルートノード(もうΘ(logN)個しかない)の中から最小値を探せばよい。これで次回の最小値検索要求に備えられる。

   -