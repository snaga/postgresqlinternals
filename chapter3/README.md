# メタデータ管理

## pg_controlファイル

* PostgreSQLが起動した時に最初に読み込まれるファイル
* PostgreSQLインスタンスの状態は各種IDの値などが保存される（変更があると更新される）
* 前回は綺麗にシャットダウンされたのか？
    * クラッシュしていたらリカバリ（REDO）を実行する
* 次に使うべき、XIDやOID（後述）は何か？
* 前回のチェックポイントはいつだったか？
* など。。。

pg_controlファイルは、PostgreSQLを起動した時に一番最初に読み込まれるファイルで、PostgreSQLのインスタンスの状態や、前回シャットダウンしたときの各種IDの値、といった内部の情報が保存されているファイルになります。

つまり、前回は正しくシャットダウンされたかどうか、もしクラッシュしてたらリカバリしないといけないので、どこからリカバリを行うか、前回のチェックポイントはいつだったか、次に使うべき内部のID（XIDやOID）は何か、といった情報が保存されているファイルになります。

以下はpg_controlファイルのサンプルです。

```
$ pg_controldata /var/lib/pgsql/9.3/data
pg_control version number:            937
Catalog version number:               201306121
Database system identifier:           5922567728546267133
Database cluster state:               in production
pg_control last modified:             Sat 21 Dec 2013 04:20:24 PM JST
Latest checkpoint location:           185/944492D8
Prior checkpoint location:            185/94447E70
Latest checkpoint's REDO location:    185/944492D8
Latest checkpoint's REDO WAL file:    000000010000018500000094
Latest checkpoint's TimeLineID:       1
Latest checkpoint's PrevTimeLineID:   1
Latest checkpoint's full_page_writes: on
Latest checkpoint's NextXID:          0/76141
Latest checkpoint's NextOID:          90863
Latest checkpoint's NextMultiXactId:  1
Latest checkpoint's NextMultiOffset:  0
Latest checkpoint's oldestXID:        1799
Latest checkpoint's oldestXID's DB:   1
Latest checkpoint's oldestActiveXID:  0
Latest checkpoint's oldestMultiXid:   1
Latest checkpoint's oldestMulti's DB: 1
Time of latest checkpoint:            Sat 21 Dec 2013 04:20:23 PM JST
Fake LSN counter for unlogged rels:   0/1
Minimum recovery ending location:     0/0
Min recovery ending loc's timeline:   0
Backup start location:                0/0
Backup end location:                  0/0
End-of-backup record required:        no
(略）
```

上記のサンプルを見ると「Latestなんとか」という情報がいくつもあるのが分かります。つまり、前回シャットダウンしたときの最後の状態をここに保存しておいて、次に起動するときに、またこの情報を使って立ち上げるというしくみになっています。

例えば、「Latest checkpoint's REDO location」とか「Latest checkpoint's REDO WAL file」という項目がありますが、これは、前回チェックポイント時のトランザクションログの位置、あるいはその時のWALファイルのファイル名、といった情報を示しています。

> pg_controlファイル補足。
> 
> pg_controlファイルはチェックポイントの際に書き出されています。
> 
> ファイルサイズは512バイト程度で一回のディスクオペレーションで書き出せるサイズになっています。（部分破損がないように）。また、データそのものはテキスト形式ではなくバイナリ形式となっています。上記のサンプルはpg_controldataコマンドを使って人間が読める形で出力した情報になります。


## OID/XID/TID

データベースのオブジェクトやとトランザクションなどを管理するために、PostgreSQLの内部にはOID、XID、TIDといったIDが何種類か存在しています。

* OID – オブジェクトID
    * データベース内部で作成されるオブジェクト（テーブル、インデックス、関数、データ型など）に付与される一意な値
    * 昔は、レコード単位で付与されていた（今は違う）
    * unsigned int（32bit）、単調増加
* XID – トランザクションID
    * 参照トランザクションも含め、トランザクションごとに付与される一意な値
    * この値を見ることによって、レコードの新旧を判断する（前述）
    * unsigned int（32bit）、単調増加（3～）
    * 周回を防止するためのVACUUMとFrozenTransactionId（2）
* TID – タプルID
    * ブロックIDと（ブロック内）オフセット番号
    * レコードの物理的な位置を表す（インデックスなどで使用）
    * unsigned short（16bit） + unsigned short（16bit） + unsigned short（16bit）

OIDというのは「オブジェクトID」とも呼ばれていて、データベース内部で作成されるオブジェクト、テーブルやインデックスや関数などに一意に付与される値になります。昔はレコード単位で付与されていたんですが、今は変わってきています。unsigned intで、単調増加する値です。

2つ目はXID、「トランザクションID」とも呼ばれるものです。「XID」というキーワードはPostgreSQLでよく出てくるのですが、参照トランザクションも含めて、トランザクションを始めるごとに1個ずつ付与されていくという数字になります。beginを実行すると、そのタイミングでXIDを1つ払いだして、もう1回beginってやると、また次のXIDが払い出される、という仕組みになります。

PostgreSQLの内部では、このXIDの値を見ることによって、トランザクションやレコードの新旧、つまりどちらのトランザクションが先に始まったのか、どちらのトランザクションの方が遅いものか、といった前後関係を判断しています。

PostgreSQLでは、タプルのヘッダにINSERTしたトランザクションIDとか、削除したトランザクションIDとかを保持していることを2章で解説しましたが、つまり、各レコードのヘッダにこのXIDを保持しているということになります。

ただし、トランザクションIDは32bitのunsigned intなので、これをずっと使い続けると、どこかでwrap around（周回）します。そのため、PostgreSQLの内部ではXIDの周回をうまくハンドリングするような仕組みが実装されています。

最後のTIDは、「タプルID」と呼ばれるIDで、特定のタプル（行）がどこのブロックにあって、そのブロックの中の何番目のレコードとして存在しているのか、という情報を保持しています。内部的にはブロックIDとブロック内のオフセット番号で表現されるデータです。

また、TIDはインデックスアクセスを理解する時に非常に重要な値です。


## システムカタログ

PostgreSQLには、システムカタログとよばれる特殊なテーブルやビューがあります。

* データベースオブジェクト（テーブル、インデックス、関数等）を管理するためのシステムテーブル（＆ビュー）
    * 97のシステムテーブル＆ビュー （PostgreSQL 9.3）
* initdb実行時にpg_catalogスキーマに作成される
* これが壊れると、データベースオブジェクトを正しく見つけられなくなる

これはデータベース内のオブジェクト（テーブルやインデックス、関数など）を管理するためのテーブルやビューで、PostgreSQL 9.3だと97個くらいのシステムテーブルやシステムビューから構成されています。

このシステムカタログの中で、実際にどのようなテーブルがあるか、どのようなユーザがいるか、どのようなデータ型があるか、といった情報を管理しています。

よって、このシステムカタログが壊れると、データベースオブジェクトを正しく見つけられなくなってしまいます。システムカタログというのは、簡単に言えば「データを管理するためのデータ」ということであり、それほどこのシステムカタログというのは重要である、ということになります。

以下はシステムカタログの例です。

```
postgres=# \dS
                        List of relations
   Schema   |              Name               | Type  |  Owner
------------+---------------------------------+-------+----------
 pg_catalog | pg_aggregate                    | table | postgres
 pg_catalog | pg_am                           | table | postgres
 pg_catalog | pg_amop                         | table | postgres
（略）
 pg_catalog | pg_database                     | table | postgres
（略）
(97 rows)

postgres=# \d pg_database
    Table "pg_catalog.pg_database"
    Column     |   Type    | Modifiers
---------------+-----------+-----------
 datname       | name      | not null
 datdba        | oid       | not null
 encoding      | integer   | not null
 datcollate    | name      | not null
 datctype      | name      | not null
 datistemplate | boolean   | not null
 datallowconn  | boolean   | not null
 datconnlimit  | integer   | not null
 datlastsysoid | oid       | not null
 datfrozenxid  | xid       | not null
 datminmxid    | xid       | not null
 dattablespace | oid       | not null
 datacl        | aclitem[] |
Indexes:
    "pg_database_datname_index" UNIQUE, btree (datname), tablespace "pg_global"
    "pg_database_oid_index" UNIQUE, btree (oid), tablespace "pg_global"
Tablespace: "pg_global"

postgres=#
```

「¥ds」を実行するとシステムカタログの一覧を見ることができます。

pg_aggregateというテーブルが一番上にありますが、pg_aggregateというのは集約関数の情報を保持しているテーブルです。その次に、pg_amというテーブルがあります。「am」はアクセスメソッドの略ですが、いろいろなインデックスへのアクセスメソッドが一覧で管理されています。

また、pg_databaseには、このPostgreSQL内部でどういったデータベースが作成・管理されているのか、といった情報が管理されています。

さらに、このpg_datebaseシステムテーブルを詳細に見てみると、データベースの名前、所有者のオブジェクトIDやデータベースエンコーディングなど、その他諸々、さまざまな情報が維持・管理されていることが分かります。

このようなさまざまな管理情報を保持しているのがPostgreSQLのシステムカタログです。
