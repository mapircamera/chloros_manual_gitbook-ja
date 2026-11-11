# API：Python SDK

{% hint style="info" %}
**完全なAPIをお探しですか？** このページは実践的なチュートリアルです。公開されているすべてのクラス、メソッド、正確なシグネチャ、およびそのままコピー＆ペーストできるサンプルは、AIアシスタント向けに最適化された[SDKリファレンス](reference/sdk-reference.md)に掲載されています。**AIアシスタントをご利用ですか？** チャットにこのURLを貼り付けると、最新のChloros 1.3.0の完全なAPI情報が表示されます：

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

このマニュアルの各ページは、小文字のスラグ + `.md` で生のマークダウン形式として利用可能であり、マニュアル全体は `https://mapir.gitbook.io/chloros/llms.txt` で索引化されています。
{% endhint %}

**Chloros Python SDK**（PyPI上の`chloros-sdk`）は、Pythonからデスクトップアプリが実行できるすべての機能（画像のバッチ処理、LATTICEカメラおよびアレイのリアルタイム制御、DAQ光センサーセッション、保存済みプロジェクトの自動化）を制御します。 これは、GUIやCLIが使用するのと同じローカルバックエンド（`127.0.0.1:5000`上のHTTP）の上に構築された薄いレイヤーであるため、これら3つのインターフェースすべてで動作は同一です。

## インストール

このSDKには、Chlorosデスクトップパッケージが同梱されています。このパッケージには処理バックエンドとハードウェアランタイムも含まれており、常にそのバージョンと一致しています。

**手順 1 — Chloros をインストールします。**Windows：[ダウンロード](download.md) ページからデスクトップインストーラー（デフォルトパス：`C:\Program Files\Chloros\`）を実行します。 Linux：`.deb` パッケージを `sudo apt install ./chloros_1.3.0_<arch>.deb` を使用してインストールします（[Linux インストール](linux/linux-installation.md)）。**手順 2 — SDK の確認** (Python 3.7 以上)。通常、インストールする必要はありません：

* **Windows:** インストーラーは、バンドルされた SDK wheel をシステムの Python にインストールします（Python がインストールされていない場合は、この手順をスキップします）。
* **Linux:** `.deb` は、システム SDK（`python3`）を `/usr/lib/python3/dist-packages` にインストールします。 その依存関係である`python3-numpy`と`python3-requests`は、パッケージによって推奨されています。`apt install`はこれらを含んでいますが、`dpkg -i`は含んでいません。
* **virtualenv** はシステムのパッケージを認識しません。同梱の wheel を virtualenv 内にインストールしてください： Linux では `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`、Windows のインストールフォルダ内の `sdk` フォルダにある `chloros_sdk-*.whl`（デフォルトでは `C:\Program Files\Chloros\sdk\`）をインストールしてください。

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Chlorosパッケージが存在せず、そのSDKが別のマシン上のバックエンドを駆動しているマシン（[pipのみでのインストールで得られるもの](#what-a-pip-only-install-gets)を参照）では、PyPIから`pip install chloros-sdk`をインストールしてください。 PyPIはリリースビルドのたびに更新されるため、最新の安定版リリースと一致します。

**ステップ3 — マシンごとに1回ログインする：**

```bash
chloros-cli login user@example.com 'YourPassword'
```

認証情報は `~/.chloros/` にキャッシュされます（両プラットフォーム共通）。Windows では、デスクトップアプリの [ユーザー](<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">) タブから同様にサインインできます。この SDK を利用するには、有料の Chloros+ プランが必要です。詳細は以下の [ライセンス要件](#license-requirement) を参照してください。

| 要件 | 詳細 |
| --- | --- |
| **Chlorosのインストール** | Windows：デスクトップインストーラー；Linux：`.deb`パッケージ（バックエンドバイナリとSDKを提供） |
| **Python** | 3.7 以上（3.10 で開発・テスト済み） |
| **オペレーティングシステム** | Windows 10/11 64 ビット、Ubuntu 22.04 LTS 以降、または NVIDIA Jetson (JetPack 6) |
| **ライセンス** | 有効なChloros+ ログイン、任意の有料プラン（Copper 以上） |

## 60秒で完了

1回の呼び出しで、プロジェクトの作成、フォルダのインポート、処理の設定、パイプラインの実行が行われます。バックエンドがまだ実行されていない場合は、自動的に起動されます：

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

（Linux では、Linux 形式のパスを使用してください：`/home/user/drone_images/flight001`。SDKは両プラットフォームで同様に動作します。）

LATTICE キャプチャフォルダを処理しますか？ LATTICE対応のラッパーを使用してください。これにより、適切なデフォルト設定（パネルターゲットの検出なし、標準のデベイヤー）が適用されます：

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — パイプラインの完全な制御

1行のコマンドを超える処理を行う場合は、`ChlorosLocal`を使用してください。これは、すでに実行中のバックエンド（デスクトップアプリのバックエンド）を使用するか、初回使用時にインストール済みのバックエンドを起動します（`auto_start_backend=True`； Windowsでは、Chlorosのインストールフォルダ内の`backend`フォルダから起動され、デフォルトでは`C:\Program Files\Chloros\backend\`が使用されます）、プロジェクトの作成と設定、進行状況の監視を行い、実行後の概要を返します。

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
`localhost`に置き換えるのではなく、デフォルトの`http://127.0.0.1:5000`のままにしておく — Windows では、`localhost` は最初に `::1` に解決され、IPv4 専用のバックエンドに対してはリクエストごとに約 2 秒のコストがかかります。
{% endhint %}

コンテキストマネージャーとして使用します。ブロックを離れると、このインスタンスによるバックエンドの使用は終了しますが、バックエンド自体は終了しません。バックエンドは次のスクリプトのために稼働し続け、30分間未使用の状態が続くと自動的に停止します。`shutdown_backend()`は、このインスタンスが起動したバックエンドを1つ停止させます。

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` は以下のキーワードを受け付けます: `debayer`、`vignette_correction`、`reflectance_calibration`、`indices`、`export_format`、 `ppk`、`daq_log_path`、`input_level`、`radiometric_output`、`array_alignment`、 `array_alignment_crop`、`array_alignment_interpolation`、および `custom_settings`。主な値：

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

LATTICE 固有のノブ（`input_level`、`radiometric_output`、および `array_alignment*` ファミリー）については、[SDK リファレンス](reference/sdk-reference.md#supported-values)に完全な値テーブルとともに記載されています。

### 進行状況の監視

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### 実行後のサマリーの読み取り — および空の実行の検出

完了時、`process()`はバックエンドの処理概要を`result["summary"]`として添付します。`summary["hints"]`の各エントリは、注目すべき点（例えば、 なぜ実行結果がゼロだったのか——といった点について、完全な文章で説明されています。また、すべてのヒントはPythonの`UserWarning`として再出力されるため、辞書を確認しなくても、空の実行は自動的に診断されます：

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()`は、実行で画像が生成されなかった場合でも例外を発生させません。** これは、SDKとCLIが意図的に異なる唯一の点です： `chloros-cli process`は、「製品が要求されたが、書き込まれなかった」という状況を失敗とみなして非ゼロで終了しますが、SDKは通常通り終了し、`summary`やヒントを通じてその状態を報告します。 パイプラインを空の実行時に停止させる必要がある場合は、例外に依存するのではなく、`summary` を確認するか（またはプロジェクトフォルダ内のファイル数を数えるか）、ご自身でチェックしてください。
{% endhint %}

## Smart Connect — ライブハードウェア

3つのヘルパーが、バックエンドのハードウェアプールで永続的なセッションを開きます。このプールは、CLI、その他のスクリプト、およびGUIのカメラタブが使用するものと同じであるため、SDKスクリプトは、シリアルポートやネットワーク帯域幅を奪い合うことなく、それらと共存できます。3つすべてが、ローカルバックエンドが実行されていない場合、自動的にローカルバックエンドを起動します。 GUIのDAQパネルはこのプールには含まれません。スクリプトからライトセンサーを開く前に、GUIでそのセンサーを閉じてください。

### 単一のLATTICEカメラ — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**相対パスはスクリプトが実行される場所を基準とします（1.3.0で変更されました）。** ローカルバックエンドを使用する場合、相対パス `output_dir`（デフォルトの `"output"` を含む） — およびバックエンドが開くその他のパス （`burst_dir`、`profile_path`、`save_path`、 `mask_path`）は、スクリプトの作業ディレクトリを基準に解決されるため、`cam.capture("output/")`は`<current folder>/output/`に書き込みを行います。 バージョン 1.2.0 では、インストール済みのバックエンドが自身のインストールフォルダ内でこれらを解決しようとし、「アクセスが拒否されました」というエラーが発生していました。リモートバックエンドに対しては、パスは入力されたままの形で送信されます。
{% endhint %}

### 同期化された配列 — `connect_array`

`connect_array` は、マルチカメラリグに推奨されるエントリポイントです。GUI と同じスマート準備フローを実行します。具体的には、ネットワーク分析、同期層の自動選択、PTP 時刻同期、カメラごとのピクセル形式の選択、AE シード、および GPIO トリガーのアーム設定です。 **最初のシリアルがマスター**（ハードウェアトリガーパルスを発射）となり、残りはスレーブとなります。

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

任意の配列キャプチャに `smart=True` を追加すると、トリガーを発火する前に、すべてのカメラで自動露出が安定するのを待つことができます。

バージョン 1.3.0 の新機能として、`ArraySession` を使用すると、ファイルに書き込むことなく、同期されたフレームを直接取得できます：

* `arr.grab()` は、1回のトリガーで各カメラから1フレームずつを numpy 配列（`{serial: array}`）として返します； `.group_id`は共有トリガーであり、`.errors`は除外されたカメラの名称を表します。
* `arr.frame_stream(fps=5.0)`は、1ティックごとにそのようなセットを1つ生成し、独自のパイプラインでのリアルタイム処理に利用できます。
* `arr.continuous("output/")`は、バックエンド独自の連続キャプチャ（GUIの「連続モード」）を実行し、`ContinuousHandle`と`status()`を返します。 `wait()`、および `stop()` を含みます。すべてのトリガーを高レートで保存する必要がある場合は、キャプチャループの代わりにこれを使用してください。

キャプチャモード（シングル／連続／インターバル／最速）、レコーダー、バースト・トゥ・ビデオ、およびアレイ位置合わせについては、[SDKリファレンス](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep)を参照してください。

### DAQ 光センサー — `connect_daq_sensor`

引数を指定しない場合、`connect_daq_sensor()`はトランスポートを自動的に検出します（優先順位：イーサネット → BLE → USB）：

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

各フレームには、135 ポイントの `spectrum`（校正済みの場合は W/m²/nm）、`is_saturated` フラグ、および CIE `x`、 `y`、`z`を含みます。特定のセンサーやトランスポートを指定するには、1つの明示的なヒントを渡します：

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

なお、キャップ補正プロファイル（`cap_id`）は、SDKの調整項目では**ありません**。代わりに、`chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` を使用して選択してください。

### 保存されたプロジェクト — `open_project`

保存されたChlorosプロジェクトは、接続されたハードウェア（`cameras.json` + `sensors.json` および `project.json`）の状態を維持し、 また、`chloros_sdk.open_project(path)`はカメラ、アレイ、光センサーを一度に再接続し、デバイス名でキャプチャを実行できます。 光センサーは、実行中のChlorosバックエンドを介して、保存された設定とキャプチャ能力とともに再接続されます。リファレンスの [プロジェクトの自動化](reference/sdk-reference.md#project-automation--chlorosproject) を参照してください。

## pipのみでのインストールで得られる機能

ハードウェアサーフェスを使用する前に、モジュールレベルの可用性フラグを確認してください：

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

**`pip install chloros-sdk`のみ**がインストールされており、Chlorosデスクトップパッケージがインストールされていないホストでは：

* `ChlorosLocal`、`process_folder`、および `process_lattice_capture` は動作**しません**。これらは、デスクトップインストーラに同梱されているバックエンドバイナリを必要とします。
* スマートコネクトヘルパー（`connect_camera`、`connect_array`、`connect_daq_sensor`）は純粋なHTTPクライアントであるため、別のマシン上のバックエンドに対しては動作します — ただし、同梱のバックエンドはループバックのみにバインドされるため、ポートを自分で転送する必要があります（例：`ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`）。また、`backend_url="http://127.0.0.1:5000"`を`auto_start_backend=False`と共に渡す必要があります。 [リモートバックエンドモード](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)を参照してください。
* ハードウェアに直接アクセスする LATTICE クラス（`LatticeCamera`、 `CameraPool`、…）はインポートできますが、デスクトップパッケージに含まれるArena SDKランタイムが必要です。これがない場合、`CAMERA_AVAILABLE`は`False`となります。
* DAQセンサーは、（トンネル化された）バックエンド（`connect_daq_sensor()`）またはプロジェクトの`proj.sensors`を介してのみ駆動されます。バックエンドは自身のマシン上でセンサーを開きます。

## AIアシスタント — `chloros-mcp`

このSDKには、`chloros-mcp`が含まれています。これは、Claude Desktop、Claude Code、Cursor、およびその他のMCPクライアントがChlorosを制御できるようにするMCPサーバーであり、画像の処理や、LATTICEカメラ、アレイ、DAQ光センサーの実行が可能です。 そのツールは、このSDKの機能を一通り網羅しています。SDKのインストール時に自動的にインストールされるため、Claude Codeでのセットアップは1行で完了します：

```bash
claude mcp add chloros -- chloros-mcp
```

その他のクライアント：[AIアシスタントでのChlorosの使用方法](ai-assistants.md)。 オプションおよびすべてのツール：[MCPサーバー（AIアシスタント）](reference/sdk-reference.md#mcp-server-ai-assistants)。

## ライセンス要件

SDKへのアクセスには、有料プラン（**Copper以上**：Copper / Bronze / Silver / Gold）のいずれかで有効なChloros+ログインが必要です。 無料のIronプランではSDK/CLIへのアクセスはできません。この制限は**サーバー側**で適用されます：すべてのSDKリクエストには、有効なセッションと有料プランの両方が含まれている必要があります。そうでない場合、バックエンドは`403` / `PLAN_UPGRADE_REQUIRED`を返します （`ChlorosLocal` によって `ChlorosLicenseError` として、また `connect_*` ヘルパーによって `ChlorosConnectError` として生成されます）が返されます。 ログアウトした呼び出し元には、代わりに `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) が返されます。`chloros-cli login` を再実行すると、前者のケースは修正されますが、後者のケースは修正されません。

オフラインでの使用は、プランの猶予期間内であれば機能します。ティアは、サーバー検証キャッシュ（5 分）または署名済みでマシンに紐付けられたライセンスキャッシュ（月額プランの場合は 30 日、年間プランの場合はサブスクリプションの有効期限まで）から読み取られます。 猶予期間が終了すると、プランは無料プランに切り替わり、マシンが一度サーバーに接続するまでSDKへのアクセスが停止します。`chloros-cli status`は無料プランでもアクセス可能な状態を維持するため、その理由は常に確認可能です。 [Chloros+ ログイン](chloros+-login.md)を参照してください。

## 例外

「Chlorosで発生したあらゆる問題」を処理するには、基底クラスをキャッチしてください：

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

すべてのパイプライン例外（`ChlorosBackendError`、`ChlorosConnectionError`、`ChlorosLicenseError`、`ChlorosAuthenticationError`、 `ChlorosConfigurationError`、`ChlorosProcessingError`）はすべて`ChlorosError`から派生しています。 注意点：`ChlorosConnectError` — `connect_camera` / `connect_array` / `connect_daq_sensor` — は、単純な `Exception` から派生しており、**`ChlorosError`** からは派生していないため、`except ChlorosError` ではこれを捕捉できません。 完全な階層構造は [SDK リファレンス](reference/sdk-reference.md#exceptions) に記載されています。

## 関連項目

* [SDK リファレンス](reference/sdk-reference.md) — AI アシスタント向けに最適化された完全な API インターフェース。
* [CLIリファレンス](reference/cli-reference.md) — すべてのCLIサブコマンドは、SDK呼び出しに対応しています。
* [AIアシスタントでのChlorosの使用](ai-assistants.md) — `chloros-mcp`のセットアップおよびプロンプトのレシピ。
* [ダウンロード](download.md) — WindowsおよびLinux用のインストーラー。
