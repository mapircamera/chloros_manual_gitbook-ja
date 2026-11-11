# Linuxへのインストール

Chlorosは、Linux向けに`.deb`パッケージとして配布されており、このパッケージにはCLI、バックエンドサーバー、およびPython SDKが含まれています（システム用`python3`に加え、virtualenvs用のバージョン対応wheelも含まれます）。

パッケージファイル名にはバージョンとアーキテクチャが含まれています。x86_64用は`chloros_1.3.0_amd64.deb`、 arm64（NVIDIA Jetson、JetPack 6）の場合は `chloros_1.3.0_arm64_jp6.deb` です。以下のコマンドでは、実際にダウンロードしたファイル名に置き換えてください。

{% hint style="info" %}
**

1.3.0の新機能：プログラムは解凍済みの状態で提供されます。** バックエンドとCLIは、単一の自己解凍ファイルではなく、フォルダ（`/usr/lib/chloros/backend/` および `/usr/lib/chloros/cli/`）としてインストールされます。 単一の自己解凍ファイルではなくなりました。起動のたびに一時フォルダに解凍されることがなくなったため、起動が大幅に高速化されました（x86_64 では、以前はバックエンドが起動のたびに約 100 秒かけて解凍を行っていました） また、`/tmp`内に数GBの空き容量を必要としなくなりました。amd64版の`.deb`も、以前の約半分ほどのサイズになっています。
{% endhint %}

***

## Linux amd64 (x86_64)

### システム要件

| 要件 | 最小 | 推奨 |
| --- | --- | --- |
| **ディストリビューション** | Ubuntu 22.04 LTS 以降 / Debian 12 以降 | Ubuntu 24.04 LTS |
| **プロセッサ** | x86_64 (Intel/AMD) | Intel Core i7 以上 |
| **メモリ (RAM)** | 8GB | 16GB 以上 |
| **グラフィックカード** | 不要（CPU処理） | 4GB以上のVRAMを搭載したNVIDIA GPU（12GB以上で`GPU_PARALLEL`が有効化され、7GB以上でシングルイメージパスにおけるTexture Awareがオフのまま維持される） |
| **ストレージ** | 空き容量 2GB | 空き容量 10GB以上のSSD |
| **Python** | Python 3.7以上（SDK用） | Python 3.10以上 |

> **Ubuntu 20.04 および Debian 11 はサポートされていません。** `.deb` の依存関係リストは、
> Chloros バックエンドが実際にリンクしているものに基づいており、これには
> focal および bullseye に同梱されている glibc 2.31 よりも新しい `libc6` が含まれているため、`apt` は
> 実行時に後で失敗するのを許すのではなく、インストールを最初から拒否します。

### インストール

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**`dpkg -i` ではなく、`apt install ./…` を使用してください。** `apt` は、パッケージの依存関係を解決し、推奨されるもの（Python SDK の要件である `python3-numpy`および`python3-requests`。`dpkg -i`はこれらをインストールしません。 CLIとバックエンドは独立しており、これらなしでも動作しますが、`import chloros_sdk`はこれらを必要とします。インストール出力には、不足しているものがすべて表示されます。`./`を残しておいてください。 `apt`が同名のパッケージを検索するのではなく、このファイルをインストールできるようにしてください。また、このパッケージは`libgl1`も宣言しており、同梱のOpenCVがこれをロードしますが、最小構成のサーバーイメージには含まれていない場合があります。
{% endhint %}

インストールを確認してください：

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### システム要件

| 要件 | 最小 | 推奨 |
| --- | --- | --- |
| **プラットフォーム** | JetPack 6 搭載の NVIDIA Jetson | Jetson Orin NX 16GB または AGX Orin |
| **JetPack** | JetPack 6.x | 最新の JetPack 6 |
| **メモリ (RAM)** | 8GB (GPU/CPU 共有) | 16GB 以上 (共有) (並列処理のしきい値は 12GB 以上) |
| **ストレージ** | 2GBの空き容量 | 10GB以上の空き容量があるNVMe SSD |
| **Python** | Python 3.7以上（SDK用） | Python 3.10以上 |

### インストール

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

amd64版の`.deb`（Python SDK、受信バッファおよびバックエンドサービスを含む）と同じ構成で、**JetPack 6**（L4T R36）上のJetson Orin / Orin NX / Orin Nano向けです。 含まれる PyTorch は、Orin 向けの JetPack 6 の CUDA ビルドであるため、GPU 処理（Texture Aware デベイヤーを含む）は Jetson の GPU 上で実行されます。 このパッケージにはCUDA自体は含まれていません。JetPack 6がインストールするCUDAおよびcuDNNライブラリを使用します（これらが含まれていないイメージ上では`sudo apt install nvidia-jetpack`が使用されます）。 CUDA GPUが使用できない場合、処理はCPU上で実行され、`--debayer texture-aware`はStandardデベイヤーを使用します（バックエンドには、その理由を説明する1行の`[DEBAYER] … needs a CUDA GPU …`ログが出力されます）。Jetsonのメモリ、熱、および現場展開時の動作については、[NVIDIA Jetsonガイド](nvidia-jetson-guide.md)を参照してください。***

## Python SDK のインストール（すべての Linux）**`.deb` によって SDK が自動的にインストールされます。** 両方のパッケージ（amd64 および Jetson）はシステム向けにインストールされるため、インストール直後から pip やネットワーク接続がなくても動作します。 そのパッケージ（`chloros_sdk`、`lattice_sdk`、`lattice_cam`、 `arena_api`、およびプロジェクトローダー）は、そのパッケージが所有する`/usr/lib/python3/dist-packages`に配置されます。アップグレードを行うとこれらは置き換えられ、パッケージを削除するとこれらも削除されます。 バージョンは、常にインストールしたCLIおよびバックエンドと一致します。

SDK独自の要件である`python3-numpy`および`python3-requests`は、**推奨されるものであり、必須ではありません**：`sudo apt install ./chloros_<version>_<arch>.deb`はこれらをインストールしますが、`dpkg -i`はインストールしません。インストール出力には、SDKバージョン`python3`がインポートする内容、不足している要件、 および `python3` のパス上の他の場所に存在し、パッケージのコピーを上書きしている `chloros_sdk`（例：以前に `pip install --user` でインストールされたもの）についても報告されます。**virtualenv内**では、システムのパッケージを認識しないため、同梱のwheelをその中に（完全にオフラインで）インストールします：

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**PyPI から** — Chloros パッケージがインストールされていないマシンで、その SDK が別のマシン上のバックエンドを駆動する場合にのみ（[リモートバックエンドモード](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel) を参照）：

```bash
pip install chloros-sdk
```

pip インストール（virtualenv または PyPI）用のオプション追加項目：

| 追加項目 | コマンド | 追加内容 |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | リアルタイム進行状況ストリーミング用の `sseclient-py` |
| `camera` | `pip install chloros-sdk[camera]` | BLE (DAQ-M) 転送用 `bleak` |

SDK を確認してください:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python SDKは、`.deb`がインストールするバックエンドと、ローカルのHTTP API（`http://127.0.0.1:5000`）を介して通信します。 `ChlorosLocal` および接続ヘルパーは、必要に応じてこれを自動起動します。保存されたプロジェクトの `connect_all()` は決して起動しません。 。そのため、ヘッドレスマシンでは、バックエンドを [サービス](#always-on-ptp-for-headless-hosts) として実行してくださいとして実行してください。常に `localhost` ではなく、リテラルな IPv4 アドレスを使用してください。`localhost` は `::1` に解決される可能性があり、リクエストごとに約 2 秒のコストがかかります。
{% endhint %}

***

## 初回セットアップ

### 1. サインイン

CLIおよびSDKへのアクセスには、有料のChloros+プラン（**Copper**以上）が必要です。これはサーバー側で強制されます。ログアウト状態の呼び出し元には`401 AUTH_REQUIRED`が、無料プラン（Iron）の呼び出し元には`403 PLAN_UPGRADE_REQUIRED`が割り当てられます。

```bash
chloros-cli login your@email.com 'your-password'
```

認証情報は `~/.chloros/user_session.json` にキャッシュされます。

{% hint style="info" %}
**アップグレード時にはログイン情報が保持されますが、削除すると消去されます。** 既存のインストール（`chloros-cli update`経由を含む）の上に新しい`.deb`をインストールしても、`~/.chloros/user_session.json`とキャッシュされたライセンスは保持されるため、ヘッドレスCLI/SDKジョブは引き続き動作します。 パッケージ（`sudo apt remove chloros`）を削除すると、そのマシン上のすべてのユーザーについて、保存されたログイン情報とライセンスキャッシュが消去されるため、新規インストールには再度 `chloros-cli login` が必要になります。
{% endhint %}

### 2. ライセンスの状態を確認する

```bash
chloros-cli status
```

`chloros-cli status`はどのティア（無料版を含む）でも動作するため、アクセスが可能か不可能かの理由をいつでも確認できます。

### 3. システム診断の実行

```bash
chloros-cli selftest
```

7つのチェックが順に実行され、いずれかが失敗するとコマンドは0以外の値を返します：

| # | チェック項目 | 確認内容 |
| --- | --- | --- |
| 1 | **バージョン** | CLIが自身のバージョン（`v1.3.0`）を報告します。 |
| 2 | **ポートの利用可能性** | ポート 5000 が空き状態であるか、*または* 正常な Chloros バックエンドからすでに応答がある（これは合格とみなされる）。 |
| 3 | **バックエンドの起動** | バックエンドバイナリが起動する。 |
| 4 | **APIテスト (`/api/test`)** | バックエンドが`status: ok`を返す。 |
| 5 | **システム情報** | `GPU: <name>, CUDA: <bool>, PyTorch: <version>` から `/api/system-info` を出力します。 |
| 6 | **ノイズ除去モデルの検出** | `*.pth.enc` モデルを検出します （Linuxの場合：`/usr/lib/chloros/models`）。 |
| 7 | **CUDA + ノイズ除去モデル**| Texture Awareは実際に使用可能 — CUDA**および** 少なくとも1つのモデルファイルが必要。 |

実行は `N/7 checks passed` で終了し、失敗したものは名前ごとに一覧表示されます。

### 4. 最初のデータセットを処理する

```bash
chloros-cli process ~/datasets/flight001
```

***

## ファイルとディレクトリ

### ユーザーごとの設定

Chlorosは、認証情報とCLI設定を、単一のクロスプラットフォームディレクトリ **`~/.chloros/`**（Windowsでは`%USERPROFILE%\.chloros\`）に保存します。一方、Linux固有の2つのキャッシュはXDGの規約に従っており、設定された場合は`XDG_CONFIG_HOME` / `XDG_CACHE_HOME`が使用されます。

| パス | 目的 |
| --- | --- |
| `~/.chloros/user_session.json` | `chloros-cli login` によって書き込まれるログインセッションキャッシュ（パッケージのアップグレード後も保持され、パッケージの削除時にクリアされる） |
| `~/.chloros/working_directory.txt` | デフォルトのプロジェクトフォルダの上書き（`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`） |
| `~/.chloros/cli_language.json` | CLIの言語設定 (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Windows GUI と共有される言語設定 — ここで `language` を指定すると、`cli_language.json` より優先されます |
| `~/.chloros/update_cache.json` | Linux/Jetsonの起動時の更新チェック用1時間キャッシュ |
| `~/.chloros/backend.log` | CLIによってバックエンドが起動された際のバックエンドログ |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | シリアル番号およびバンドルハッシュをキーとする、カメラごとの LATTICE キャリブレーションパックのキャッシュ |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | DAQ キャップ補正プロファイルに対するオプションのユーザーによる上書き |
| `~/.config/chloros/system_config.json` | ダイナミック・コンピュート・アダプテーション（DCA）からのキャッシュされたハードウェアプロファイル — これを削除すると、ハードウェアの再検出が強制されます |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | バックエンドサーバーのログ（起動ごとに1ファイル） |
| `~/Documents/MAPIR/Chloros Projects/` | オーバーライドが設定されていない場合のデフォルトのプロジェクトフォルダ |

### システム全体

| パス | 目的 |
| --- | --- |
| `/usr/bin/chloros-cli` | ラッパースクリプト — バンドルされたネイティブライブラリ用に `LD_LIBRARY_PATH` を設定し （また、そのフォルダが存在する場合は `TMPDIR` を `/mnt/ssd/tmp` に設定し）、その後、実際のバイナリを実行 |
| `/usr/bin/chloros-backend` | ラッパースクリプト — バックエンドについても同様で、これをインストール済みの本番用ビルドとしてマークする |
| `/usr/lib/chloros/cli/`、`/usr/lib/chloros/backend/` | コンパイル済みのCLIおよびバックエンド。解凍した状態でインストールされます（1.3.0からの新機能）。 プログラムは `cli/chloros-cli` および `backend/chloros-backend` です |
| `/usr/lib/chloros/chloros-cli`、`/usr/lib/chloros/chloros-backend` | 上記の実際のプログラムを起動する2行のランチャースクリプト — SDK、CLI、および古いスクリプトがここでこれらを探すため、これらのパスに配置されています |
| `/usr/lib/chloros/arena_runtime/` | LATTICEカメラで必要とされるArena SDKランタイム |
| `/usr/lib/chloros/models/*.pth.enc` | Texture Awareデベイヤーで使用される暗号化されたノイズ除去モデル |
| `/usr/lib/python3/dist-packages/chloros_sdk/`（および `lattice_sdk`、`lattice_cam`、`arena_api`：プロジェクトローダー） | システム用 `python3` にインストールされ、本パッケージが所有する Python SDK |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | 仮想環境（virtualenv）用の、このビルドと完全に一致する Python SDK wheel |
| `/usr/lib/chloros/exiftool` | バンドルされた ExifTool ランチャー（システムに exiftool が存在しない場合にのみ、`/usr/local/bin/exiftool` へのシンボリックリンクが作成される）。バージョン 1.3.0 以降、ExifTool スクリプトには `/usr/lib/chloros/exiftool-dist/` に独自の Perl ライブラリが同梱されているため、システムパッケージ `libimage-exiftool-perl` なしで実行可能です |
| `/etc/chloros/update.conf` | `chloros-cli update` によって読み込まれる Update-channel の設定 |
| `/etc/sysctl.d/60-chloros-ptp.conf` | バックエンドがroot権限なしでPTPポートにバインドできるよう、`net.ipv4.ip_unprivileged_port_start = 319`を設定 |
| `/etc/sysctl.d/60-chloros-gige.conf` | LATTICEカメラのストリームに対して、カーネルのソケット受信バッファ (`net.core.rmem_max`、`net.core.rmem_default`)を32 MBに増やし、インストール時に適用します （すでに高い値が設定されている場合は変更しない）。カーネルのデフォルト設定では、カメラパケットがカーネル内でドロップされ、アレイはフレームやフレームレートを失う。`chloros-cli lattice network`は、バッファが少なくなった際にその状態を報告する。 [Linux ホスト — ソケット受信バッファ](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) を参照 |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | ダイナミックローダーを `/usr/lib/chloros/arena_runtime` に向ける |
| `/lib/udev/rules.d/70-chloros-daq.rules` | ログイン中のユーザーに DAQ-U USB シリアルブリッジへのアクセス権を付与する (CP2102N、`10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | 指定したユーザーとしてバックエンドを実行する、オプトイン式の常時稼働バックエンドサービス（インストール済み、**有効化されていない**） — 有効化すべきものは、[Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | バックエンドを**root**として実行する同じサービス（インストール済み、**有効化されていない**）。これを残しておくことで、有効化したマシンが継続して動作するようになります |
| `/var/log/chloros/` | root サービスの詳細ログ。`adm` グループが読み取り可能 （パッケージが削除されると削除されます） |
| `/usr/share/applications/chloros-cli.desktop` | ターミナルを開く「Chloros CLI」アプリケーションメニュー項目 |

**ファイアウォール（1.3.0で新規追加）。** インストール時に`ufw`または`firewalld`が有効になっている場合、このパッケージはUDPポート`5002` （DAQ-E データ）、`5353`（mDNS 検出）、`319` および `320`（PTP）を開き、パッケージが削除されるとそれらのルールを削除します。 ファイアウォールがインストールされているが有効になっていない場合、何も追加せず、後で有効にした際に許可すべきポートを表示します。TCP ポートは開かれません。 DAQ-Eのファームウェア更新時、センサーが画像を取得するためにTCP経由でこのコンピュータに接続しようとすると、ファイアウォールが有効な状態でその接続がブロックされるため、`… device never connected back on TCP <port> …`のエラーが発生します。

## バックエンド実行ファイルの場所

CLIおよびSDKはバックエンドを自動検出します：

| コンポーネント | パス |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| バックエンド | `/usr/lib/chloros/chloros-backend` (`/usr/lib/chloros/backend/chloros-backend`のランチャー) |

`--backend-exe` CLIフラグまたは `backend_exe` SDKコンストラクタパラメータを使用してバックエンドのパスを上書きし、 ポートは `--port`（デフォルトは `5000`）で上書きします。

{% hint style="info" %}
`CHLOROS_BACKEND_URL` は、リモートバックエンド上の **`lattice`**、**`project`**、および**`daq pool-*`** コマンドファミリーをリモートバックエンドで実行します。 コアコマンド（`process`、`login`、`logout`、`status`、 `export-status`、`time-sync`、`selftest`）は、これを意図的に無視し、常に `http://127.0.0.1:<port>` を対象とします。
{% endhint %}

***

## Linux 上の LATTICE カメラおよび DAQ 光センサー

live-hardware コマンド群はすべて Linux (amd64 および Jetson) で動作します:

* **`chloros-cli lattice`** — LATTICEカメラおよび同期アレイの検出、接続、設定、およびキャプチャを行います。`.deb`は、これらに必要なArena SDKランタイムをバンドルし、ダイナミックローダーに登録します。
* **`chloros-cli daq pool-*`** — バックエンドプール経由で DAQ-U/M/E 光センサーに接続し、校正済みスペクトルをストリーミングし、`.daq` ファイルを記録してキャッププロファイルを交換します。また、DAQ-E については、ネットワーク設定の変更、再起動、およびファームウェアの更新を行います。 `chloros-cli daq discover`は接続可能なセンサーをスキャンし、`chloros-cli daq --help`は`pool-*`のサブコマンド一覧を表示します。
* **`chloros-cli project`** — 保存されたプロジェクト（カメラ、センサー、および処理設定）をヘッドレスで実行します。
* **`chloros-cli time-sync`** — ChlorosバックエンドがLATTICEカメラおよびDAQ-Eセンサー用に実行しているPTPグランドマスターを検査します。

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id`は、`pool-latest`、 `pool-stream`、`pool-record`、および`pool-set-cap`によって必要とされます。`pool-list`は、現在プール内にあるIDを表示します。

{% hint style="info" %}
**`--eth-host`は検出をスキップします。** `pool-connect --eth`はmDNSによってDAQ-Eを検出し、ARPキャッシュが空の状態でも機能するARPフォールバックを使用します。`--eth-host`にセンサーのIPアドレスまたはホスト名を指定すると、そのセンサーに直接接続します。
{% endhint %}

**DAQ-Uのシリアルアクセス権限**は、インストール済みのudevルール（`uaccess` + グループ `dialout`）によって処理されます。すでに接続済みのセンサーにアクセスできない場合は、ルールを再読み込みするか、センサーを再接続してください：

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

コマンド一式の詳細については、[CLIリファレンス](../CLI.md)を参照してください。

### ヘッドレスホスト向けの常時稼働 PTP

このパッケージは、バックエンド用に 2 つの systemd サービスをインストールしますが、いずれも有効化されていません。 DAQ-EセンサーやLATTICEカメラ向けにPTP時刻同期を常時実行する必要があるヘッドレスJetsonやサーバー、あるいはSDKスクリプトが保存されたプロジェクトに接続する環境（`connect_all()`はバックエンドを起動しません）では、**ご自身のユーザーとして**サービスを有効にしてください：

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service`は、そのユーザーとしてバックエンドを実行します。つまり、書き込まれるキャプチャデータはそのユーザーのものであり、詳細なログはそのユーザーの`~/.cache/chloros/logs`に記録され、そのユーザーの`chloros-cli login` （`~/.chloros`）を使用するため、まずそのユーザーとして一度サインインしてください。パッケージのインストール時には、ユーザー名が埋め込まれたこのコマンドが表示されます。

サービスがない場合、バックエンド（およびそれに伴うPTPグランドマスター）は、`chloros-cli`コマンドやSDKスクリプトなど、何かによって起動されている間のみ実行されます。

このユニットは、バックエンドを`127.0.0.1:5000` （ユニット内の `CHLOROS_HOST` / `CHLOROS_PORT` 環境設定；`sudo systemctl edit chloros-backend@.service` で上書き可能）にバインドし、障害発生時には 5 秒後に再起動します。

**root サービス。**`chloros-backend.service` は、バックエンドを**root** として実行します。これを有効にしたマシンが稼働し続けるよう、依然としてインストールされていますが、 ただし、rootバックエンドが書き込むすべてのデータはrootに属するため、ユーザーは自身のキャプチャを削除することはできません。そのログは`/var/log/chloros`にあり、`adm`グループが読み取り可能です。自分のユーザーに切り替えるには：

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**サービスのバックエンドは共有されており、クライアントによって停止されることはありません。** `chloros-cli`およびSDKスクリプトは、見つけたままの状態でこれを使用します。`process`はこれ上で実行され、最後のプロジェクトの代わりに自身のプロジェクトを読み込みます。 通常であれば、CLIがバックエンドを再起動する場面（`--restart`、またはインポートで何も見つからなかった`process`の場合）、 このコマンドはユニットの名前を指定し、`sudo systemctl restart <unit>`と表示して終了します（`1`）。これは、この方法で停止されたバックエンドは再起動しないためです。`sudo systemctl restart chloros-backend@$USER.service`を使用して手動で再起動してください。 サービスのバックエンドはアイドル状態でも決して自ら停止することはなく、`chloros-cli backend stop`はこれを拒否し、`systemctl stop`コマンドを指定します。**PTP がポートを取得する方法** PTP は UDP 319/320 を使用しますが、これらはいずれも通常の特権ポートの下限である 1024 未満です。 このパッケージの `postinst` は、`/etc/sysctl.d/60-chloros-ptp.conf` に `net.ipv4.ip_unprivileged_port_start = 319` を書き込みます。これにより、バックエンドはユーザーの権限で実行されている間、これらのポートにバインドできるようになります。 バージョン 1.3.0 以降、このパッケージはバックエンドに対してファイル権限 (`setcap`) を設定せず、`libcap2-bin` も依存関係ではなくなりました。展開されたバックエンドでは、ファイル権限を設定すると起動できなくなるため、 そのため、自分で追加しないでください。***

## Bash スクリプトの例

{% hint style="info" %}
**スクリプトに適した終了コード。**`chloros-cli process`は成功時に`0`を終了させ、**失敗時には0以外の値を返します — 画像プロダクトを要求したものの、何も書き出されなかった実行も含まれます** （`Processing finished but wrote no image products.` を出力し、プロジェクトフォルダ名と一般的な原因を表示します）。成功した実行では、書き出された画像プロダクトの数が報告されます（`Image products written: N`、マスクが書き出された場合は `(plus M clamp masks)` も追加されます）。終了コード：`0`（成功）、`1`（失敗）、`2`（引数エラー）、`130`（中断）。
{% endhint %}

### 複数のデータセットを処理する

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### カスタム設定で処理する

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

有効な `--format` の値は正確に 4 つあり、スペースが含まれているため、必ず引用符で囲んでください：

| `--format` の値 | 出力フォルダ |
| --- | --- |
| `TIFF (16-bit)` *(デフォルト)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` は、`standard`（デフォルト）または `texture-aware`（Chloros+）を受け付けます。

### Cron による自動処理

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK の例

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## トラブルシューティング

### インストール後に CLI が見つからない

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### アクセス拒否

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### ファイル権限を追加した後、バックエンドが起動しない

バックエンド上で `setcap` を実行しないでください。 バージョン 1.3.0 以降、このパッケージはファイル機能（file capabilities）を設定しません。PTP のポート 319/320 は、[PTP のポートの割り当て方法](#always-on-ptp-for-headless-hosts) で説明されている `sysctl` の設定に基づいています。 ファイルカパビリティを設定すると、展開済みのバックエンドはセキュア実行モードとなり、自身のライブラリを見つけられなくなるため、起動しなくなります。設定した場合は、それを削除してください：

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP が起動しない / ポート 319 にバインドできない

非特権ポートの下限が引き下げられていることを確認し、引き下げられていない場合は、現在のブートに対して再適用してください：

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

その後、グランドマスターを確認してください：

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice` コマンドが「LATTICE カメラドライバが見つかりません」というエラーで失敗する

バックエンドが、インストール時に付属していた LATTICE カメラランタイムを読み込めませんでした。すべての `lattice` コマンドはバックエンド上で実行されます。CLI 自体にはカメラドライバは含まれていません。Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`) を再インストールしてください：`.deb`は、`/usr/lib/chloros/arena_runtime`内のバックエンドの隣にランタイムをインストールし、ダイナミックローダーに登録します。その登録を確認するには：

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### バックエンドの起動に失敗しました

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

起動に失敗したバックエンドのログは `~/.cache/chloros/logs/` にあります。ポート 5000 がバックエンドサービス (`systemctl status chloros-backend@$USER.service`) に属している場合は、そのプロセスを強制終了するのではなく、サービスを再起動してください。

### CUDA が検出されませんでした

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` には、`GPU: <name>, CUDA: <bool>, PyTorch: <version>` という 1 行で同じ内容が報告されています。

### 共有ライブラリが見つからない

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### SDカード搭載システムでの起動が遅い

バージョン1.3.0以降、バックエンドとCLIは解凍済み状態でインストールされるため、起動のたびに一時ディレクトリに自動的に展開されることはなくなりました。 一時ファイルは、`/mnt/ssd/tmp` フォルダが存在する場合、自動的にそこに保存されます。存在しない場合は、高速なファイルシステムを指定することができます：

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Linux での Chloros の更新 Linux

`update` コマンドは Linux/Jetson 専用です。このコマンドは、`/etc/chloros/update.conf` で設定された更新チャネルに公開されているバージョンを確認し、一致する `.deb` のダウンロードとインストールを提案します：

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Linux/Jetson では、CLI も起動のたびにノンブロッキングの更新チェックを実行し （結果は `~/.chloros/update_cache.json` に 1 時間キャッシュされます）し、新しいバージョンが存在する場合は `Update available: vX.Y.Z` を出力します。設定、プロジェクト、ログイン情報はアップデート後も保持されます。

パッケージのアップグレード中は、インストール中にすべてのバックエンドサービスが停止するため、実行中のスクリプトはカメラやセンサーへの接続を失いますが、その後、有効になっている各サービスが再起動されます（手動で起動されたものの、有効化されていないサービスは、再起動後と同様に停止したままになります）。

## アンインストール

```bash
sudo apt remove chloros
```

アンインストールすると、すべてのバックエンドサービスが停止および無効化され、特権ポートの下限がデフォルト値（1024）に復元されます。また、受信バッファの設定ファイル（実行中の値は再起動まで保持されます）、バンドルされた exiftool のシンボリックリンク、 Arenaローダーの設定、パッケージによって追加されたファイアウォールルール、およびインストールされたPythonSDKを削除し、キャッシュされた認証情報をクリアします。プロジェクトおよび`~/.chloros/`データファイルはそのまま残されます。 `sudo apt purge chloros` は、`/var/log/chloros` も削除します。

***

## 次の手順

* [NVIDIA Jetson ガイド](nvidia-jetson-guide.md) — Jetson 向けの最適化とデプロイ
* [CLI : コマンドライン](../CLI.md) — CLI ガイド
* [API : Python SDK](../api-python-sdk.md) — SDK ガイド
* [CLIリファレンス](../reference/cli-reference.md) および [SDKリファレンス](../reference/sdk-reference.md) — バージョン1.3.0のコマンドおよびAPIの完全な一覧
* [動的コンピューティング適応](../processing-architecture/dynamic-compute-adaptation.md) — Chlorosがハードウェアに適応する方法
