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


