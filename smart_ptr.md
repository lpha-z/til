# C言語にあるポインタのセマンティクスと対応するスマートポインタ等

1. 実行時に決まるサイズを持つメモリ領域(配列/オブジェクト)を指し示す
2. コンパイル時から分かっているがヒープ領域にとりたいメモリ領域(配列/オブジェクト)を指し示す
→ 使用者が単一ならunique_ptr、使用者が複数いるならshared_ptr
3. 操作してほしいメモリ領域を渡す時(出力用変数)や・通知・監視など
→ T&、なくなる可能性があるならweak_ptr(なくなってたらそれはそれで)
4. コピーするとオーバーヘッドの大きいメモリ領域を、書き換えないことを前提に渡す時(参照)
→ T const&、所有責任者が決まっていないならshared_ptr(最後に使った人が片付ける)
5. 相互に参照し合うメモリ領域を指し示す(連結リストなど)
6. 配列の走査
→ ポインタ自体を書き換える必要があるタイプ。
→ 安全かどうかはアルゴリズムレベルで確認するのが普通？
7. nullable
→optionalを使おうか
8. 重複のないオブジェクトID
→これはちょっとテクニカル

