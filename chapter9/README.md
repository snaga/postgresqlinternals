# PostgreSQLの拡張

## PostgreSQLを拡張する方法

PostgreSQLの特徴として拡張のしやすさが挙げられます。

* UDF （ユーザ定義関数）
* インデックス拡張（AND GiST）
* Hook
* カスタムバックグラウンドワーカー
* EXTENSION

PostgreSQLの拡張の基本は、UDFと呼ばれるユーザ定義関数を定義する方法です。これはCでも書けますしそれ以外のLL系の言語でも書くことができます。

また、インデックスを拡張するというのも、PostgreSQLでは比較的多く見られる拡張です。

最近、PostgreSQLを拡張する方法として「Hook」呼ばれる機能が入りました。これは、PostgreSQLの内部の機能に対してユーザが作った関数を割り当てることで、内部のオプティマイザやエグゼキュータの処理を一部ユーザ側で乗っ取ることができるような仕組みです。これを使って内部の動きを変えるというのも1つの方法としてあります。

それ以外に、カスタムバックグラウンドライトという拡張も可能です。バックグラウンドワーカーというのは、PostgreSQLの裏側に常駐してバックグラウンドで動くプロセスのことで、カスタムバックグラウンドワーカーはこれをユーザ自身が作成できる機能です。

EXTENSIONというのは、こういったユーザが作成した関数などをモジュール化する機能です。複数の関数を一括してインストールしたりアンインストールしたりする機能を提供しています。


## Hookによる拡張

* PostgreSQLは動的にモジュール（共有ライブラリ）をロード可能
    * shared_preload_libraries
    * local_preload_libraries, LOADコマンド
* 内部には、関数ポインタを用いた Hook が多数存在する
    * ParserSetupHook
    * ExecutorStart_hook
    * ProcessUtility_hook
    * などなど
* この Hook に独自の関数を設定することで、内部のデータにアクセスしつつ、処理を拡張することができる

Hookによる拡張は、Sharedオブジェクトを作成して、それをpreloadするという形で実装します。

使用できるHookとしては、ParserSetupHook、ExecutorStart_hook、ProcessUtility_hookなどがあり、これらにユーザの定義した関数を設定することで、こういったコマンドなどの内部処理を則ることができます。

## GiSTによるインデックスの拡張

* 汎用検索ツリー（GiST：Generalized Search Tree）
    * 任意のインデックスを実装する枠組み（フレームワーク）を提供
* 7つのメソッドを実装することによって新しいインデックスを実装できる
    * same, consistent, union
    * penalty, picksplit
    * compress, decompress

## 参考文献・参考資料

* C言語によるユーザ定義関数の作り方 http://www.slideshare.net/snaga/how-towritepgfunc
* 関数の変動性分類についておさらいしてみる http://www.slideshare.net/toshiharada/lt-41040251
