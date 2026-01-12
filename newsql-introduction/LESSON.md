# メモと学び

TiDBはMySQLに寄せて作られている

mysqlクライアントで接続可能

## TiDBの機能概要

- オートスケールしない。ノード数はユーザが決める
- 外部キーはサポートしている。サポートしていないNewSQLもある
- レプリカノードがない
- 分離レベルはSnapshot Isolation

## TiDBのアーキテクチャ

- TiDB: SQLを解釈、実行計画を作成するコンピューティングノード
- TiKV: 分散KVSストレージノード
- PD: 各ノードの配置状況やクラスタの状況を管理するノード
  - どのTiKVノードにアクセスすればよいか教えるロードバランサの役割を担う
- TiProxy: TiDBを複数台冗長化している場合のロードバランサ


クエリ実行の流れ
1. TiDBノードでSQL解析・最適化
2. PDノードの情報をもとにやりとりするTiKVノードを決定
3. TiKVへ検索
4. 結果集約・生成


TiKVに格納されるデータは3重冗長化される

TiKVではデータの保存単位としてリージョンというデータ塊を管理する。複数のレコードをPKの一定の範囲で区切ったもの(シャードみたいなもの)。

TiDBはリージョン単位でTiKVにアクセスする

リージョンごとに読み書きを担当するTiKVサーバ(リーダー)が決まっている。

レプリケーションの遅延を無くしデータ一貫性を保つため、デフォルトではリーダーからだけ読み書きする。PDがTiDBへリーダー情報を伝達する。

リージョンフォロワーからも読み込みできるオプションはある


トランザクションの書き込みには2相コミットを採用

TiKV内でのリーダーからフォロワーへのデータ複製にはRaftを使用している

コピー最中に障害が発生したり、ネットワーク不具合などでコピーに問題が発生した場合でもデータの整合性を保つための合意アルゴリズム

PDはデータの更新時に、書き込む対象のTiKVノード(リーダー)を教える役割とタイムスタンプを発行する役割の2つを担当する

更新されるレコードに対して更新時のTSO(Timestamp Oracle)を付与し、レコードの複数のバージョンを保存する。これによってSnapshot isolationを実現する






## TiDBの各種機能

### バックアップ

```
BACKUP DATABASE <database name> to 'local://<保存ディレクトリへの絶対パス>'
```

```
RESTORE DATABASE * FROM 'local://<保存ディレクトリへの絶対パス>'
```

### ノード冗長性

- TiDBサーバはステートレスなので、1台停止してもTiProxy経由で再接続でき新しいサーバを起動すれば状態を復元できる
  - 停止したTiDBサーバ上で実行中だったトランザクションはロールバックされる
- TiKVサーバは相互にハートビートを送っており、TiKVノードがダウンした際にフォロワーがRaftによりリーダーに昇格する
  - TiDBノードは新しいリーダーにリクエストをリトライする
  - 過半数が止まってしまった場合はネットワーク分断と区別がつかないため自動復旧はできず、どのデータを正とするかは人間の判断が必要になる
  - 手動復旧する際はPDのクラスター情報を直接書き換える(停止したノードをクラスタから除外、新しいTiKVノードを加えてスケールアウト)
- PDサーバは基本的に3台構成で、全ての処理をリーダーが担当。etcdベースで作られている。etcdもRaftベースなのでTiKVと同様の冗長性
  - 手動復旧する際はリージョン情報とTSOの最大値が必要になる
  - 一度データを削除し、新規PDサーバにTiKVに保存されているリージョン情報と過去のTSOよりも十分に大きい値を与えれば手動復旧は可能

## コマンドログ

TiDBサーバのポート番号とプロセスIDの対応関係を調べる

```sql
select ddl_id, ip, port from information_schema.tidb_servers_info;
```

リージョンごとのregion_idを調べる

```sql
select region_id, start_key, end_key,table_name, db_name from information_schema.tikv_regi
on_status where table_name = 'Books';
```

```
+-----------+--------------------------------------+--------------------------------------+------------+---------+
| region_id | start_key                            | end_key                              | table_name | db_name |
+-----------+--------------------------------------+--------------------------------------+------------+---------+
|       276 | 7480000000000000FF7000000000000000F8 | 7480000000000000FF7200000000000000F8 | Books      | library |
+-----------+--------------------------------------+--------------------------------------+------------+---------+
1 row in set (0.01 sec)
```

リージョンIDごとにリーダーのストアIDを調べる

```
select * from information_schema.tikv_region_peers where is_leader = 1 order by region_id;
```

リージョンID276のリーダーのストアIDは3と判明

ストアIDとノードの対応関係を調べる

```
select * from information_schema.tikv_store_status;
```

リージョンIDとポート番号の関係を一発で調べる便利コマンド

```sql
use information_schema;
select region_id, store_id, address from tikv_region_status natural join tikv_region_peers
 natural join tikv_store_status where is_leader = 1 and table_name = 'Books';
```

ストアID3に対応するノードは127.0.0.1:20160と判明

`lsof -i:20161 | grep LISTEN` でプロセス51726がポート20160を使っていると判明

プロセス51726を停止しようとすると、tiupでは検証されてエラー
```
tiup playground scale-in --pid 51726
```
```
no endpoint available, the last err was: error requesting http://127.0.0.1:2384/pd/api/v1/store/3, response: "[PD:core:ErrStoresNotEnough]can not remove store 3 since the number of up stores would be 2 while need 3"
```
強制killする


PDサーバの状態を調べる
```
tiup ctl:v8.5.4 pd member show
```
