# つくって、壊して、直して学ぶNewSQL入門

TiDBを使ってみる本

## 環境構築

WSL2を使用

```
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 4796k  100 4796k    0     0  8737k      0 --:--:-- --:--:-- --:--:-- 8736k
Successfully set mirror to https://tiup-mirrors.pingcap.com
Detected shell: bash
Shell profile:  /home/ya/.bashrc
/home/ya/.bashrc has been modified to add tiup to PATH
open a new terminal or source /home/ya/.bashrc to use it
Installed path: /home/ya/.tiup/bin/tiup
===============================================
Have a try:     tiup playground
===============================================
```

以降はtiupコマンドで遊ぶ

## 収穫

- tiup playgroundでかなり気軽にTiDBを試せることが分かったし壊し方も知った
- TiDBのアーキテクチャがわかった
  - コンピューティングレイヤのTiDB、ストレージレイヤのTiKV、スケジューリングやタイムスタンプ発行(Zookeeper的存在)のPD
  - TiKVは内部的にRocksDBを2インスタンスずつ持ち、Raftログと実際のデータ格納に使う。ノード間でRaftによる一貫性と可用性の維持
    - RocksDBはLSMツリーベース。
  - PDもRaftでリーダーを選出、etcdでTSOを単調増加に維持
  - 悲観ロックによる分散トランザクション管理。タイムスタンプベースのMVCCでsnapshot isolationを実現する

