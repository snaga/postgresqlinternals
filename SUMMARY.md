# PostgreSQL Internals

本コンテンツは、2014年1月30～31日に筑波大学で開講された「情報システム特別講義D」における講義「Inside PostgreSQL Kernel」の内容を再構成、加筆・修正したものです。 

* [はじめに](README.md)
    * 本コンテンツについて
    * 本コンテンツへのフィードバックについて
* [アーキテクチャ概要](chapter1/README.md)
    * PostgreSQLの構成要素
    * PostgreSQLの基本的なアーキテクチャ
    * SQL文の処理される流れ
* [トランザクション管理](chapter2/README.md)
    * トランザクション処理におけるACID特性
    * 各レコードの可視性の管理
    * Atomicity（原子性）の実装
    * Consistency（一貫性）の実装
    * Isolation（分離性）の実装
    * トランザクション分離レベルの定義
    * Durability（永続性）の実装
    * チェックポイント
* [メタデータ管理](chapter3/README.md)
    * pg_controlファイル
    * OID/XID/TID
    * システムカタログ
