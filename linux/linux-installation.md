# Linux のインストール

Chloros Linux向けには、CLIおよびバックエンドサーバーをインストールする`.deb`パッケージとして配布されています。Python（SDK）は別のpipパッケージです（バージョンが一致したwheelとして`.deb`内にも同梱されています）。

パッケージファイル名にはバージョンとアーキテクチャが含まれています： x86_64 版は `chloros_1.2.0_amd64.deb`、JetPack 6 Jetson ビルド用は `chloros_1.2.0_arm64_jp6.deb` です。以下のコマンドでは、実際にダウンロードしたファイル名に置き換えてください。

***

## Linux amd64 (x86_64)

### システム要件

| 要件 | 最小 | 推奨 |
| --- | --- | --- |
| **ディストリビューション** | Ubuntu 22.04 LTS 以降 / Debian 12 以降 | Ubuntu 24.04 LTS |
| **プロセッサ** | x86_64 (Intel/AMD) | Intel Core i7 以上 |
| **メモリ (RAM)** | 8GB | 16GB 以上 |
| **グラフィックカード** | 不要（CPU処理） | 4GB以上のVRAMを搭載したNVIDIA GPU（12GB以上で`GPU_PARALLEL`が有効化され、7GB以上でシングルイメージパスにおけるTexture Awareが無効のまま維持される） |
| **ストレージ** | 空き容量 2GB | 空き容量 10GB以上のSSD |
| **Python** | Python 3.7以上 (SDK用) | Python 3.10以上 |

> **Ubuntu 20.04 および Debian 11 はサポートされていません。** `.deb` の依存関係リストは、
> Chloros バックエンドが実際にリンクしているものに基づいており、そこには
> `libc6 (>= 2.34)` も含まれています。 Focal と bullseye はどちらも glibc 2.31 を同梱しているため、`apt` は
> 実行時に後で失敗するのを許すのではなく、インストールを
> 最初から拒否します。

### インストール

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i`は依存関係を解決しません。パッケージが欠落していると報告された場合、`sudo apt-get install -f`（または`sudo apt --fix-broken install`）がインストールを完了します。これは正常な流れであり、エラーではありません。
{% endhint %}

インストールを確認してください：

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->***

## Linux arm64 (NVIDIA Jetson)

### システム要件

| 要件 | 最小 | 推奨 |
| --- | --- | --- |
| **プラットフォーム** | JetPack 6 搭載の NVIDIA Jetson | Jetson Orin NX 16GB または AGX Orin |
| **JetPack** | JetPack 6.x | 最新の JetPack 6 |
| **メモリ (RAM)** | 8GB (GPU/CPU 共有) | 16GB 以上 (並列 GPU ワーカーの閾値は 12GB 以上) |
| **ストレージ** | 2GBの空き容量 | 10GB以上の空き容量があるNVMe SSD |
| **Python** | Python 3.7以上（SDK用） | Python 3.10以上 |

### インストール

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

amd64版の`.deb`と同じ構成で、Jetson Orin / Orin NX / Orin Nano向けに最適化されたCUDAビルドです。Jetsonのメモリ、熱、および実環境での動作については、[NVIDIA Jetsonガイド][NVIDIA Jetsonガイド](nvidia-jetson-guide.md)を参照してください。

***

## Python SDK のインストール (すべてのLinux)

SDKは、バックエンド向けの純粋なPythonHTTPクライアントであるため、amd64とarm64の両方で同じパッケージが動作します。入手元は2つあります：

**PyPIから** — 公開されている安定版リリース：

```bash
pip install chloros-sdk
```

**同梱のwheelファイルから** — 先ほどインストールしたCLI/backendと確実に一致します（お使いの`.deb`がPyPI版より新しい場合にこれを使用してください）：

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668 ディストリビューション**（Ubuntu 23.10 以降、Debian 12 以降）では、システム全体への pip によるインストールが拒否されます。 `pip install --user …`、仮想環境、または`sudo pip install --break-system-packages …`を使用してください。パッケージインストーラは、SDKをシステムに自動インストールすることはありません Python — その選択はユーザーに委ねられています。
{% endhint %}

オプションの追加機能：

| 追加機能 | コマンド | 追加内容 |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | ライブ進行状況ストリーミング用の `sseclient-py` |
| `camera` | `pip install chloros-sdk[camera]` | `bleak`BLE (DAQ-M) 転送用  |

SDK を確認してください：

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` は、Chloros（CLI）およびバックエンドをインストールします。 Python SDK は、ローカルの HTTP API (`http://127.0.0.1:5000`) を介してそのバックエンドと通信し、必要に応じてそれを自動起動します。常に `localhost` ではなく、リテラルな IPv4 アドレスを使用してください — `localhost`は`::1`に解決される可能性があり、リクエストごとに約2秒の処理時間がかかります。
{% endhint %}

***

## 初回設定

### 1. サインイン

CLI および SDK へのアクセスには、有料の Chloros+ ティア（**Copper** 以上）が必要です。これはサーバー側で強制されます。ログアウト状態の呼び出し元には `401 AUTH_REQUIRED` が、無料ティア（Iron）の呼び出し元には `403 PLAN_UPGRADE_REQUIRED` が割り当てられます。

```bash
chloros-cli login your@email.com 'your-password'
```

認証情報は `~/.chloros/user_session.json` にキャッシュされます。

{% hint style="warning" %}
**インストールまたはアップグレードのたびに、再度ログインする必要があります。** パッケージの `prerm` スクリプトは、マシン上のすべてのユーザーに対して `~/.chloros/user_session.json` およびキャッシュされたライセンスを意図的にクリアします。これにより、新しいビルドでは、古いキャッシュを信頼するのではなく、常にライセンスの再検証が行われます。
{% endhint %}

### 2. ライセンスの状態を確認する

```bash
chloros-cli status
```

`chloros-cli status`はどのエディション（無料版を含む）でも動作するため、アクセスが可能か不可能かの理由を常に確認できます。

### 3. システム診断の実行

```bash
chloros-cli selftest
```

7つのチェックが順に実行され、いずれかが失敗するとコマンドは 0 以外の値を返します：

| # | チェック項目 | 確認内容 |
| --- | --- | --- |
| 1 | **バージョン** | 「CLI」が自身のバージョン（`v1.2.0`）を報告します。 |
| 2 | **ポートが利用可能** | ポート 5000 が空いているか、*または*正常な Chloros バックエンドによって既に応答されていること（これは合格とみなされます）。 |
| 3 | **バックエンドの起動** | バックエンドバイナリが起動します。 |
| 4 | **「API」テスト (`/api/test`)** | バックエンドが `status: ok` と応答します。 |
| 5 | **システム情報** | `/api/system-info` から `GPU: <name>, CUDA: <bool>, PyTorch: <version>` を出力します。 |
| 6 | **ノイズ除去モデル** | `*.pth.enc`モデルを検出（Linux上では: `/usr/lib/chloros/models`）。 |
| 7 | **CUDA + ノイズ除去モデル**| 「Texture Aware」は実際に使用可能 — CUDA**および** 少なくとも1つのモデルファイルが必要。 |

実行は `N/7 checks passed` で終了し、失敗した項目は名前ごとに一覧表示されます。

### 4. 最初のデータセットを処理する

```bash
chloros-cli process ~/datasets/flight001
```

***

## ファイルとディレクトリ

### ユーザーごとの設定：

Chlorosは、認証情報とCLIの設定を、単一のクロスプラットフォームディレクトリ**`~/.chloros/`**（Windowsでは`%USERPROFILE%\.chloros\`）に保存します。 一方、Linux固有の2つのキャッシュはXDGの規約に従っており、これらは設定されている場合、`XDG_CONFIG_HOME` / `XDG_CACHE_HOME`を優先します。

| パス | 目的 |
| --- | --- |
| `~/.chloros/user_session.json` | `chloros-cli login` によって書き込まれるログインセッションキャッシュ（パッケージのインストール/アップグレードのたびにクリアされる） |
| `~/.chloros/working_directory.txt` | デフォルトのプロジェクトフォルダーの上書き (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | 「CLI」の言語設定 (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Windows GUI と共有される言語設定 — ここで `language` が `cli_language.json` より優先される |
| `~/.chloros/update_cache.json` | Linux／Jetsonの起動時の更新チェック用1時間キャッシュ |
| `~/.chloros/backend.log` | CLIによってバックエンドが起動された際のバックエンドログ |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | シリアル番号およびバンドルハッシュをキーとする、カメラごとのLATTICEキャリブレーションパックのキャッシュ |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | DAQキャップ補正プロファイルに対するオプションのユーザーオーバーライド |
| `~/.config/chloros/system_config.json` | Dynamic Compute Adaptation からのキャッシュされたハードウェアプロファイル — これを削除すると、ハードウェアの再検出が強制されます |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | バックエンドサーバーのログ（起動ごとに1ファイル） |
| `~/Chloros Projects/` | 上書き設定がない場合のデフォルトのプロジェクトフォルダ |

### システム全体

| パス | 目的 |
| --- | --- |
| `/usr/bin/chloros-cli` | ラッパースクリプト — バンドルされたネイティブライブラリに対して `LD_LIBRARY_PATH` を設定し、その後実際のバイナリを実行します |
| `/usr/bin/chloros-backend` | ラッパースクリプト — 上記と同様。さらに `CHLOROS_PRODUCTION=1` を設定し、バックエンドの認証ゲートが黙って自身を無効化できないようにする |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | コンパイル済みバイナリ |
| `/usr/lib/chloros/arena_runtime/` | LATTICEカメラで必要なArena SDK ランタイム |
| `/usr/lib/chloros/models/*.pth.enc` | Texture Awareデベイヤーで使用される暗号化されたノイズ除去モデル |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | このビルドに完全に一致するPythonSDKホイール |
| `/usr/lib/chloros/exiftool` | 同梱のexiftool（システムにexiftoolが存在しない場合に限り、`/usr/local/bin/exiftool`へのシンボリックリンクとして設定） |
| `/etc/chloros/update.conf` | `chloros-cli update` によって読み込まれる Update-channel の設定 |
| `/etc/sysctl.d/60-chloros-ptp.conf` | バックエンドがroot権限なしでPTPポートをバインドできるよう、`net.ipv4.ip_unprivileged_port_start = 319`を設定 |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | ダイナミックローダーを `/usr/lib/chloros/arena_runtime` に向ける |
| `/lib/udev/rules.d/70-chloros-daq.rules` | ログイン中のユーザーにDAQ-U USBシリアルブリッジ（CP2102N、`10c4:ea60`）へのアクセス権を付与 |
| `/lib/systemd/system/chloros-backend.service` | 常時稼働のバックエンドサービスのオプトイン（インストール済み、**有効化されていない**） |
| `/usr/share/applications/chloros-cli.desktop` | ターミナルを開く「Chloros CLI」アプリケーションメニュー項目 |

## バックエンド実行ファイルの場所

CLIおよびSDKは、バックエンドを自動検出します：

| コンポーネント | パス |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| バックエンド | `/usr/lib/chloros/chloros-backend` |

CLIフラグ（`--backend-exe`）またはコンストラクタパラメータ（`backend_exe`、SDK）を使用してバックエンドパスを上書きし、ポートは`--port` （デフォルトは `5000`）でポートを上書きします。

{% hint style="info" %}
`CHLOROS_BACKEND_URL` は、リモートバックエンド上の **`lattice`**、**`project`**、および**`daq pool-*`** コマンドファミリーを、リモートバックエンドで指定します。主要なコマンド （`process`、`login`、`logout`、`status`、 `export-status`、`time-sync`、`selftest`) は、これを意図的に無視し、常に `http://127.0.0.1:<port>` をターゲットにします。
{% endhint %}

***

## Linux 上の LATTICE カメラおよび DAQ 光センサー

live-hardware コマンド群はすべて、Linux（amd64 および Jetson）で動作します：

* **`chloros-cli lattice`** — LATTICEカメラおよび同期アレイの検出、接続、設定、およびデータ取得を行います。`.deb`は、これらに必要なArena SDK ランタイムをバンドルし、動的ローダーに登録します。
* **`chloros-cli daq pool-*`** — バックエンドプール経由で DAQ-U/M/E 光センサーを接続し、校正済みスペクトルをストリーミングし、`.daq` ファイルを記録します。 コンパイル済みのCLIには、`pool-*`ファミリーのみが含まれています：`pool-connect`、`pool-disconnect`、`pool-list`、 `pool-latest`、`pool-stream`、`pool-record`、`pool-set-cap`。
* **`chloros-cli project`** — 保存されたプロジェクト（そのカメラ、センサー、および処理設定）をヘッドレスで実行します。
* **`chloros-cli time-sync`** — ChlorosバックエンドがLATTICEカメラおよびDAQ-Eセンサー向けに実行しているPTPグランドマスターを検査します。

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id`は、`pool-latest`、`pool-stream`、 `pool-record`、および `pool-set-cap` によって必要とされます。`pool-list` は、現在プール内にある ID を示しています。

{% hint style="info" %}
**マルチホームマシンでの最初のDAQ-E接続には、`--eth-host`を優先してください。** 自動検出は mDNS を検索しますが、ARP キャッシュが空の状態ではセンサーのインターフェースを見逃す可能性があるため、センサーが完全に正常であっても、起動直後の最初の `pool-connect --eth` は失敗する可能性があります。センサーの IP アドレスまたはホスト名を指定することで、検出プロセスを完全にスキップできます。
{% endhint %}

**DAQ-Uのシリアル権限**は、インストール済みのudevルール（`uaccess` + グループ `dialout`）によって処理されます。 すでに接続済みのセンサーにアクセスできないままの場合は、ルールを再読み込みするか、センサーを再接続してください：

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

コマンド一式の詳細については、[CLI リファレンス](../CLI.md)を参照してください。

### ヘッドレスホスト向けの常時稼働 PTP

初回インストール時、systemd ユニット `chloros-backend.service` が生成されますが、**有効化はされていません**。 DAQ-EセンサーやLATTICEカメラのためにPTP時刻同期を継続的に実行する必要があるヘッドレスJetsonまたはサーバーでは、これを有効にしてください：

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

これを有効にしないと、PTPはChlorosバックエンドが実行されている間、つまりCLI / SDKセッションがアクティブな間のみ動作します。

このユニットは、バックエンドを `127.0.0.1:5000` にバインドします（ユニット内の `CHLOROS_HOST` / `CHLOROS_PORT` 環境設定； `sudo systemctl edit chloros-backend.service`で上書き可能）にバインドし、失敗した場合は5秒後に再起動します。

**PTPのポート割り当て方法** PTPはUDP 319/320を使用しており、いずれも通常の特権ポートの下限である1024未満です。 このパッケージの `postinst` は、`/etc/sysctl.d/60-chloros-ptp.conf` に `net.ipv4.ip_unprivileged_port_start = 319` を書き込み、これによりバックエンドがユーザーとして実行中にそれらにバインドできるようになります。 また、万全を期すため、バックエンドのバイナリに `setcap cap_net_bind_service,cap_net_raw=+ep` を適用しています。これが、`libcap2-bin` がこのパッケージの宣言済み依存関係となっている理由です。***

## Bash スクリプトの例

{% hint style="info" %}
**スクリプトで扱いやすい終了コード。**`chloros-cli process` は、成功時には `0` を返し、**失敗時には 0 以外の値を返します。これには、画像プロダクトをリクエストしたものの何も書き出さなかった実行も含まれます** （この場合、`Processing finished but wrote no image products.` を出力し、プロジェクトフォルダ名と一般的な原因を明示します）。 正常に実行された場合は、書き出された画像プロダクトの数が報告されます（`Image products written: N`）。終了コード：`0`（成功）、`1`（失敗）、 `2` 引数エラー、`130` 中断。
{% endhint %}

### 複数のデータセットの処理

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

### カスタム設定での処理

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

有効な `--format` の値は正確に 4 つあり、スペースが含まれているため、必ず引用符で囲んでください：

| `--format` 値 | 出力フォルダ |
| --- | --- |
| `TIFF (16-bit)` *(デフォルト)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` は、`standard`（デフォルト）または `texture-aware`（Chloros以上）を受け付けます。

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
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## トラブルシューティング

### インストール後に「CLI」が見つからない

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
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### インストール中に「setcap failed」が発生

`.deb`は、root権限なしでPTPポート319/320をバインドできるように、`cap_net_bind_service`を`/usr/lib/chloros/chloros-backend`に適用します。 インストール時に `libcap2-bin` が存在しなかった場合、この呼び出しはスキップされます。これをインストールしてから、パッケージを再インストールしてください：

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP が起動しない／ポート 319 にバインドできない

非特権ポートの下限値が引き下げられていることを確認し、引き下げられていない場合は、現在のブートに対して再適用してください：

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

その後、グランドマスターを確認してください：

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### 「LATTICE カメラドライバが見つかりません」

Arena SDK ランタイムが解決されていません。パッケージによって書き込まれるローダー設定が存在し、更新されていることを確認してください：

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
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

起動に失敗した際のバックエンドのログは、`~/.cache/chloros/logs/` にあります。

### CUDA が検出されません

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` には、1 行で同じ内容が報告されています: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`。

### 共有ライブラリの欠落

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### SDカード搭載システムでの起動が遅い

コンパイル済みのバイナリは、起動のたびに一時ディレクトリに展開されます。 `/mnt/ssd/tmp` が存在する場合、Chlorosはそれを自動的に使用します。存在しない場合は、`TMPDIR` を高速なファイルシステムに設定してください：

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Linux での Chloros の更新

`update` コマンドは、Linux /Jetson でのみ利用可能です。このコマンドは、`/etc/chloros/update.conf` で設定された更新チャネルに公開されているバージョンを確認し、一致する `.deb` のダウンロードとインストールを提案します:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Linux /Jetson では、CLIも起動のたびにノンブロッキングの更新チェックを実行します（結果は `~/.chloros/update_cache.json` に 1 時間キャッシュされます） 。新しいバージョンが存在する場合、`Update available: vX.Y.Z` が表示されます。設定やプロジェクトは更新後も保持されますが、更新後は再度サインインする必要があります。

## アンインストール

```bash
sudo apt remove chloros
```

アンインストールすると、`chloros-backend.service`が停止し、非特権ポートの下限がデフォルト値（1024）に復元され、バンドルされたexiftoolへのシンボリックリンクとArenaローダーの設定が削除され、キャッシュされた認証情報が消去されます。 プロジェクトおよび `~/.chloros/` データファイルはそのまま残されます。

***

## 次の手順

* [NVIDIA Jetson ガイド](nvidia-jetson-guide.md) — Jetson 向けの最適化とデプロイ
* [CLI : コマンドライン](../CLI.md) — CLI ガイド
* [API : Python SDK](../api-python-sdk.md) — SDK ガイド
* [CLI リファレンス](../reference/cli-reference.md) および [SDK リファレンス](../reference/sdk-reference.md) — バージョン 1.2.0 向けの包括的なコマンド／API一覧
* [動的演算適応](../processing-architecture/dynamic-compute-adaptation.md) — Chlorosがハードウェアにどのように適応するか
