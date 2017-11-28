# PostgreSQL Internals

本コンテンツは、2014年1月30～31日に筑波大学で開講された「情報システム特別講義D」における講義「Inside PostgreSQL Kernel」の内容を再構成、加筆・修正したものです。 

* [はじめに](README.md)
    * 本コンテンツについて
    * 本コンテンツへのフィードバックについて
* [1. アーキテクチャ概要](chapter1/README.md)
    * PostgreSQLの構成要素
    * PostgreSQLの基本的なアーキテクチャ
    * SQL文の処理される流れ
* [2. トランザクション管理](chapter2/README.md)
    * トランザクション処理におけるACID特性
    * 各レコードの可視性の管理
    * Atomicity（原子性）の実装
    * Consistency（一貫性）の実装
    * Isolation（分離性）の実装
    * トランザクション分離レベルの定義
    * Durability（永続性）の実装
    * チェックポイント
* [3. メタデータ管理](chapter3/README.md)
    * pg_controlファイル
    * OID/XID/TID
    * システムカタログ
* [4. MVCCとストレージ構造](chapter4/README.md)
    * テーブルファイル
    * テーブルのページレイアウト
    * データ型とデータサイズ
    * インデックス（B-Tree）ファイル
    * B-Tree（リーフ）のページレイアウト
    * VACUUM処理
    * インデックスとタプルの可視性
    * TOASTテーブル
    * FreeSpace Map (FSM)
    * FSMの物理レイアウト
    * Visibility Map (VM)
    * Visibility Mapの構造
* [5. メモリ管理](chapter5/README.md)
    * 共有メモリとローカルヒープ
    * Memory Context
    * 共有バッファと管理アルゴリズム
* [6. ロック制御](chapter6/README.md)
    * ロック
    * 2-Phase Locking (2PL)
    * デッドロックの検出と解消
    * SpinLockとLWL
* [7. パース、リライト、オプティマイズ](chapter7/README.md)
    * パース、リライト
    * オプティマイザ統計情報
    * よく利用されるオプティマイザ統計情報
    * 実行コストの計算
    * Seq scan/Index scan実行コスト例
    * GEQO（遺伝的問い合わせ最適化）
    * 参考資料
* [8. エグゼキュータ](chapter8/README.md)
    * エグゼキュータの役割
    * 結合処理
    * Nested Loop Join
    * Merge Join
    * Hash Join
    * インデックスの種類
    * インデックスのアクセスメソッド
    * B-Treeインデックスのタプル取得呼び出し
    * B-Treeインデックスのアクセスメソッド
    * B-Treeインデックスの構造
    * B-Treeデモ
    * GINインデックス
* [9. PostgreSQLの拡張](chapter9/README.md)
    * PostgreSQLを拡張する方法
    * Hookによる拡張
    * GiSTによるインデックスの拡張
    * 参考文献・参考資料
* [参考文献](refs/README.md)
