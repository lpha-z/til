学科PCのセットアップ
====
Ubuntu 14.04 LTS
## 日本語入力
システム設定>言語サポート>キーボード入力に使うIMシステム：Mozc

 + `$ ibus-setup` で出てくるポップアップに書いてあった
 + ✔日本語入力ができるようになった
## Dvorak配列に変更する
[Mozcの設定をus(dvorak)にする](http://kekke59.hatenablog.com/entry/2014/12/29/150649)だけでもいいのだけれど、MacでいうDvorak-Qwerty(コントロールキーを押している時だけqwertyに戻る)にしたい上、日本語キーボードにしかないキーを活用できなくなるので、xkbを用いて実現する

### キーの定義
```
$ mkdir ~/.xkb
$ mkdir ~/.xkb/symbols
$ mkdir ~/.xkb/keymap
$ setxkbmap -print > ~/.xkb/keymap/mykbd
$ vi ~/.xkb/symbols/mykeys (キーボードで何が出力されるかを定義、小文字面・大文字面・Qwerty面・特殊面)
$ vi ~/.xkb/symbols/mytypes (どのモディファイアキーを押しているときにどの面を使うかの定義)
$ vi ~/.xkb/keymap/mykbd (xkb_symbolsのus(dvorak)をmykeys(dvorakquarty)に変更、xkb_typesにmytypes(control_Q)を追加)`
$ xkbcomp -I$HOME/.xkb ~/.xkb/keymap/mykbd $DISPLAY
```

 + Warningがたくさん出るが、ErrorがなければOK(即反映される)

## 変換無変換の活用
自分のPCのBackspaceキーが壊れて以来、無変換キーをその機能に割り当てていたせいで変な手癖がついてしまった。
その時についでに変換無変換(親指)活用宗派に入信してしまったのでそれを実現するやつ。

### モディファイアキーの追加
以下のファイルを.Xmodmapとして保存、`$ xmodmap .Xmodmap`で実行

```
keycode 255 = space
keycode 65 = ShiftL
add Mod3 = Muhenkan
add Mod3 = Henkan
```

ちなみに下みたいにしたらxcapeの方でエラーだった
```
keycode 65 = NoSymbol
add Shift = space
```


###　xcapeでワンショットモディファイアにする
[xcapeのリポジトリ](https://github.com/alols/xcape)の指示に従ってインストール
`$ xcape -e "#102=BackSpace"`みたいにやればOK。設定ミスをしたからといって何回もコマンドを実行すると効果が重複するようになってわけのわからないことになる、そういう時は`$ killall xcape`で一からやり直そう

### 起動時にコマンドを打たないでこれらを有効にする
まず設定を実行するコマンドを集めたシェルスクリプトファイルを作る、`chmod +x`を忘れずに

```
#!/bin/bash
xkbcomp -I$HOME/.xkb ~/.xkb/keymap/mykbd $DISPLAY 2> /dev/null
xmodmap .Xmodmap
killall xcape
xcape -e "#102=BackSpace"
xcape -e "#100=Tab"
xcape - e"#65=Space"
```
 + `/etc/rc.local`を使う方法はうまくいかなかった
 + ~~`/etc/profile.d/`以下にこのスクリプトを実行するスクリプトを置いておけば~~ ダメ、ログインできなくなる
 + 自動起動するアプリケーションに入れてもうまくいかない
 + (うまくいかないので保留中……)
#### 参考文献

 + [xkbでキーバインドを変更する](http://blog.cnu.jp/blog/2014/05/12/use-xkb/)
 + [Ubuntu：「無変換+○」にカーソル移動系ホットキーを設定する（xkb編）](http://did2memo.net/2015/07/20/ubuntu-xkb-muhenkan-hotkey/)
 + [Ubuntu forums : Mac-style Dvorak-Qwerty keyboard](http://ubuntuforums.org/showthread.php?t=774773)
 + [xkbを使って修飾キー有のキーバインドを設定する](http://jou4.hateblo.jp/entry/2015/04/04/191437)
 +[[xkb] Ubuntu 14.04 で Caps Lock を別のキーにする方法](http://ill-identified.hatenablog.com/entry/2014/09/14/143337)(xkbの入門、わかりやすい)
 + [キーバインドの設定 ](http://huyu398.hatenadiary.com/entry/2013/02/22/230033)
 + [UbuntuでSandSをxcape+xmodmapで実現する](http://qiita.com/ychubachi@github/items/95830219f1bdf912280b)


## DvorakJPの導入
 + Mozcプロパティ>一般>ローマ字テーブル 編集...>編集>インポート でDvorakJPローマ字テーブルを入れる
 + Mozcプロパティ>入力補助>シフトキーでの入力切替をオフ にしないとアルファベットのShift段で半角アルファベット大文字が出てしまう
 + ちなみに普通の設定では出せない全角記号を出せるようにしてある
	+ y+htnsで矢印4種 ： (yazirusi)+Qwertyでいうjkl;。GoogleIMEもz+hjklで矢印が打てたよね。交互打鍵になるのもうれしい
	+ 全角アポストロフィの代わりに三点リーダ2つ(……) ：  全角アポストロフィはまず使わない。促音(っ)も考慮したが、単独で打ちたいとき以外は1打で打てるので見送った
(以下シフト段)
	+ C, W で ×, ÷ ： (caceru), (waru)から。
	+ A, B, G で α, β, γ ： (arufa), (be-ta), (g;ma)から。
	+ Sで☆ ： (star)から。
	+ Qで々 ： (onazi)で(on)まで共通だったので。
	+ {}で【】 ： 変換しないと出せないかっこの中では最も使用頻度が高かった
	+ _で～ ： 普通なら~で～を出すところだが、全角アンダースコアはまず使わないし、長音符類が一つのキーにまとまっているとわかりやすいし、中央寄りでうれしい
	+ |~で≠≒ ： それなりに使うのに変換が貧弱だった。どっちだか忘れがちだが、縦棒で関連付けて覚える。~は先の移動で空席だったし、≒の意味で～を使うことがあるので。

## マウスポインタが消える
 + 起動中に音量ボタンを押したら(?)マウスポインタが描画されなくなった(描画されないだけで動かせる)
	+ 再起動で解決
	+ この手のはいろいろ相性問題があるらしくて、根本的な原因はわからず

## うっかりログインできなくなった
 + `/etc/profile.d/`に間違ったスクリプトを入れてしまったせいでログインが不可能に。
 + `Ctrl+Alt+F1`,で呼び出せるレスキューコンソールからもログインできず。
	+ ブートメニューでadvance options for Ubuntu>recovery mode>rootで当該ファイルを削除(readonlyだと無理だが、`dpkg`を実行したらread/writeになった)。これでログイン可能になった。