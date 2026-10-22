
# sudo

# sudo -s
- root権限のシェルを起動する
- 使う場面 : root権限が必要なコマンドを連続して実行したいときや、一般ユーザーではアクセスできないディレクトリにcdで移動したいとき

例：
```bash
sudo -s
cd /opt/webquiz
exit
```

# sudo bash -c 'コマンド'
- root権限でBashを起動し、指定したコマンド文字列を実行する
- 使う場面 : 自身に権限のないディレクトリ内のワイルドカード（*.py）などをroot権限で展開したいとき　ファイル単独指定なら必要ない

例：
```bash
sudo bash -c 'chown root:webquiz /opt/webquiz/*.py'
```


# sudo cdが実行できない理由
- cdはシェルの組み込みコマンドであり、sudoから直接実行できない
- 使う場面 : root権限が必要なディレクトリに移動したい場合は、sudo -sでrootシェルを起動してからcdを実行する

