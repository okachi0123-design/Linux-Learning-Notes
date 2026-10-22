# systemd 学習ノート

## 1. systemdとは

systemdは、Linux上で動作するサービスなどのUnitを管理する仕組み。

サービスの起動・停止・再起動、異常終了時の自動再起動などを管理できる。

### Unitとは

systemdが管理する対象の単位。

主な種類：

| Unit | 役割 |
|---|---|
| `.service` | サービス・プロセスの管理 |
| `.socket` | ソケットの管理 |
| `.timer` | 指定した時刻や間隔でUnitを起動 |
| `.mount` | ファイルシステムのマウント管理 |
| `.target` | 複数のUnitをまとめるための目標・同期ポイント |

ServiceはUnitの種類の一つ。

---

## 2. サービスファイルとは

サービスの起動方法や運用ルールを定義する設定ファイル。

主に以下の3つのセクションから構成される。

| セクション | 役割 |
|---|---|
| `[Unit]` | Unitの説明や依存関係、起動順序を定義 |
| `[Service]` | サービスの起動方法やプロセスの管理方法を定義 |
| `[Install]` | enable/disable時のシンボリックリンクの設定を定義 |

※ `.service`ファイルでは`[Service]`は必須。`[Unit]`と`[Install]`は省略可能。

設定ファイルでは、systemdがあらかじめ定義したディレクティブを使用する。

例：`ExecStart=`、`Restart=`、`WantedBy=`

---

## 3. [Unit]

自身のUnitの説明、依存関係、起動順序などを設定する。

### 主なディレクティブ
`Descriptionm, After / Before, Requires / Wants`

### Description

Unitの説明 


### After / Before

Unitの起動順序を指定する。

- `After=A.service`：Aの起動処理が先
- `Before=B.service`：Bより先に自身を起動

ただし、After/Beforeだけでは依存先を自動起動しない。

### Requires / Wants

Unitの依存関係を設定する。

- `Requires`：依存先の起動も要求する。`After`と組み合わせた場合、依存先の起動に失敗すると自身も起動できない。
- `Wants`：依存先の起動も要求するが、依存先の起動に失敗しても自身は起動できる。

※ `Requires`を使用しても、依存先が後から異常終了した際に必ず自身も停止するわけではない。

### 設定例

```ini
[Unit]
Description=FastAPI Application
After=network.targe
Wants=network.target
```

- `Description`：サービスの説明
- `After`：network.targetの後に起動
- `Wants`：network.targetの起動も要求する

---

## 4. [Service]セクション

サービスの起動・停止・再起動、実行環境、セキュリティなどを設定する。

### ① 起動・停止に関する設定

| ディレクティブ | 役割 | 設定例 |
|---|---|---|
| `Type` | サービスの起動完了を判断する方式 | `Type=exec` |
| `ExecStart` | サービス起動時に実行するコマンド | `ExecStart=/usr/bin/python3 /opt/app/main.py` |
| `RemainAfterExit` | プロセス終了後もactiveとして扱うか | `RemainAfterExit=yes` |

**Typeについて**

主な設定値は以下の8種類。

- `simple`：プロセスの生成直後に起動したと判断
- `exec`：指定したプログラムの実行開始に成功したら起動したと判断
- `forking`：親プロセスが終了したら起動完了と判断
- `oneshot`：一度だけコマンドを実行し、完了を待つ
- `dbus`：指定したD-Bus名が取得されたら起動完了
- `notify`：アプリからの準備完了通知を待つ
- `notify-reload`：notify方式に加えて再読み込みの完了通知にも対応
- `idle`：他のジョブの処理が落ち着いてから実行する

**RemainAfterExitについて**

`RemainAfterExit=yes`はプロセスを動かし続ける設定ではなく、プロセスが正常終了してもサービスをactiveとして扱う設定。

特に`Type=oneshot`で活用される。

### ② 自動再起動・タイムアウトに関する設定

| ディレクティブ | 役割 | 設定例 |
|---|---|---|
| `Restart` | 自動再起動の条件 | `Restart=on-failure` |
| `RestartSec` | 自動再起動までの待機時間 | `RestartSec=5s` |
| `TimeoutStartSec` | 起動処理のタイムアウト | `TimeoutStartSec=30s` |

**Restartの代表的な設定**

- `no`：自動再起動しない
- `always`：原則として終了時に再起動
- `on-failure`：異常終了した場合に再起動

※ systemctlによる明示的な停止では、通常、自動再起動されない。

### ③ 実行環境に関する設定

| ディレクティブ | 役割 | 設定例 |
|---|---|---|
| `User` | サービスを実行するユーザー | `User=appuser` |
| `WorkingDirectory` | コマンドを実行する作業ディレクトリ | `WorkingDirectory=/opt/app` |
| `Environment` | 環境変数を設定 | `Environment=PYTHONUNBUFFERED=1` |

**補足**

`User`を設定することで、root以外のユーザー権限でアプリケーションを実行できる。

`WorkingDirectory`は、アプリケーションの相対パスの基準にもなる。

### ④ セキュリティに関する設定

| ディレクティブ | 役割 | 設定例 |
|---|---|---|
| `NoNewPrivileges` | プロセスが新たな権限を獲得することを制限 | `NoNewPrivileges=yes` |
| `PrivateTmp` | サービス専用の一時ディレクトリ環境を用意 | `PrivateTmp=yes` |
| `ProtectSystem` | システムの一部を書き込み禁止にする | `ProtectSystem=full` |

これらはサービスの権限やアクセス範囲を制限するための設定。

アプリケーションによって必要なファイルアクセスが異なるため、設定後は正常に動作するか確認する必要がある。

---

## 5. [Install]セクション

`systemctl enable`や`systemctl disable`を実行した際に、シンボリックリンク（symlink）をどのように作成・削除するかを定義する。

- `enable`：設定に従ってsymlinkを作成
- `disable`：該当するsymlinkを削除

`[Install]`セクション自体がsymlinkを作るわけではない。

### 主なディレクティブと設定例

| ディレクティブ | 役割 | 設定例 |
|---|---|---|
| `WantedBy` | 指定UnitからWants依存されるようにする | `WantedBy=multi-user.target` |
| `RequiredBy` | 指定UnitからRequires依存されるようにする | `RequiredBy=example.target` |
| `Alias` | Unitに別名を設定する | `Alias=web-backend.service` |
| `Also` | enable/disable時に別のUnitも同時に処理する | `Also=helper.service` |

※ 上記は各ディレクティブの独立した設定例。`example.target`と`helper.service`は仮のUnit名。

### WantedByの具体例

```ini
[Install]
WantedBy=multi-user.target
```

この設定でサービスをenableすると、以下のようなシンボリックリンクが作成される。

```text
/etc/systemd/system/
  └── multi-user.target.wants/
       └── fastapi.service
```

これにより、`multi-user.target`が起動するときに、`fastapi.service`も起動対象になる。

### [Unit]のWantsと[Install]のWantedByの違い

- `[Unit]`の`Wants=B.service`：自身がBに依存する
- `[Install]`の`WantedBy=B.service`：Bが自身に依存するように、enable時にsymlinkを作る

また、`Also`は別のUnitを一緒にenable/disableするための設定で、サービスを同時に起動するという意味ではない。

---

(https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html)
- [systemd.exec 公式マニュアル](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html)
