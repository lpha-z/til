学科PCのセットアップ
====
Ubuntu 14.04 LTS
## 日本語入力
システム設定>言語サポート>キーボード入力に使うIMシステム：Mozc

 + `$ ibus-setup` で出てくるポップアップに書いてあった
 + ✔日本語入力ができるようになった
## Dvorak配列に変更する
### Mozcの設定
(手順)

 0. テキスト入力設定から「日本語(Mozc)」を選んでおく
 0. `sudo vi /usr/share/ibus/component/mozc.xml` でlayout要素を`us(dvorak)`にする
 0. 再起動する

(TIPS)

 + [参考サイト](http://kekke59.hatenablog.com/entry/2014/12/29/150649)
 + ✔ 直接入力、日本語入力共にDvorak配列になった
 + × 「半全」キーが【`~】キー扱いになってしまうので使えなくなる
	  Mozcプロパティ>一般>キー設定の選択：ATOKにすると「変換」キーで直接入力と日本語入力が切り替えられるのでこれでしのぐ + ？ PageDownキーを押すとなぜか~が出力されてしまう
 + ？ MozcプロパティのウィンドウがUnityじゃなくてWindowsXPっぽいやつになってしまった
 + 学科PCは、CapsLockがControlキーとして使えるような設定がされているらしいが、それは消えずに使えている(ctrl(nocaps)を読み込んでいるっぽい)

```
$ xkeymap
(略)
control      Control_L (0x25),  Control_L (0x42),  Control_R (0x69)
(略)
```
(キーコード0x25はAの左にあるキー=CapsLock英数)
### 余っているキーの割り当て
 + layout要素を`jp(dvorak)`にすると、「\\|」キーや「＼_」キーが使えるようになるが、記号の配列が大きく異なる
 ++ [参考サイト](https://sites.google.com/site/tetsuroweb/home/software/ubuntu/tips/japanese-keyboard-layout)
 + `key <AB11>`が「＼_」キー、`key <AE13>`が「\\|」キーのようなのでこれらを割り当てる
 + [xkbでキーバインドを変更する](http://blog.cnu.jp/blog/2014/05/12/use-xkb/)と[Ubuntu：「無変換+○」にカーソル移動系ホットキーを設定する（xkb編）](http://did2memo.net/2015/07/20/ubuntu-xkb-muhenkan-hotkey/)を参考に以下のようにキーを変更
	 + 「}]」キーを押すと【＼|】が出力されるのを、【~\`】が出力されるように変更
	 + 追い出された【＼|】を出力する機能を「\\|」キーに変更
	 + 「＼_」キーはとりあえず未設定
	 + 「半全」キーはとりあえず保留(【~\`】を出力するまま)

```
$ mkdir ~/.xkb
$ mkdir ~/.xkb/symbols
$ mkdir ~/.xkb/keymap
$ setxkbmap -print > ~/.xkb/keymap/mykbd
$ vi ~/.xkb/symbols/mykeys (以下のファイルを作成)
$ vi ~/.xkb/keymap/mykbd (xkb_symbolsのincludeの文字列の最後に`+mykeys(addkeys)`を追加)
$ xkbcomp -I$HOME/.xkb ~/.xkb/keymap/mykbd $DISPLAY
```

``` ~/.xkb/keymap/mykbd
partial modifier_keys
  replace key <AE13> {[yen,bar]};
  replace key <AC12> {[grave, asciitilde});
};
```
 + Warningがたくさん出るが、ErrorがなければOK(即反映される)

## DvorakJPの導入
 + Mozcプロパティ>一般>ローマ字テーブル 編集...>編集>インポート でDvorakJPローマ字テーブルを入れる
 + Mozcプロパティ>入力補助>シフトキーでの入力切替をオフ にしないとアルファベットのShift段で半角アルファベット大文字が出てしまう

## マウスポインタが消える
 + 起動中に音量ボタンを押したら(?)マウスポインタが描画されなくなった(描画されないだけで動かせる)
	+ 再起動で解決
	+ この手のはいろいろ相性問題があるらしくて、根本的な原因はわからず

