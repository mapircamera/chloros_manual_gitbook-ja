# API : Python SDK

{% hint style="info" %}
**APIの完全版をお探しですか？** このページは実践的なチュートリアルです。 すべてのパブリッククラス、メソッド、正確なシグネチャ、およびコピー＆ペースト可能な例は、AIアシスタント向けに最適化された[SDKリファレンス](reference/sdk-reference.md)に記載されています。**AIアシスタントをご利用ですか？** チャットにこのURLを貼り付けて、完全かつ最新のChloros 1.2.0 APIを利用できるようにしてください：

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

このマニュアルの各ページは、小文字のスラッグ + `.md` の形式で生のマークダウンとして利用可能であり、マニュアル全体は `https://mapir.gitbook.io/chloros/llms.txt` にインデックス化されています。
{% endhint %}

**Chloros Python SDK** （PyPI上の`chloros-sdk`）は、Pythonからデスクトップアプリが実行できるすべての機能――画像の一括処理、LATTICEカメラおよびアレイのリアルタイム制御、DAQ光センサーセッション、保存済みプロジェクトの自動化――を駆動します。 これは、GUI および CLI が使用するのと同じローカルバックエンド（`127.0.0.1:5000` 上の HTTP）の上に構築された薄いレイヤーであるため、3つのインターフェースすべてで動作は同一です。

## インストール

インストールは 2 つのステップで行います。まず、Chloros デスクトップパッケージ（処理バックエンドとハードウェアランタイムを提供します）をインストールし、次に Python パッケージをインストールします。

**手順 1 — Chloros をインストールします。** Windows：[ダウンロード](download.md)ページからデスクトップインストーラ（デフォルトパス：`C:\Program Files\MAPIR\Chloros\`）を実行します。 Linux: `.deb` パッケージをインストールします（[Linux インストール手順](linux/linux-installation.md)）。**手順 2 — SDK のインストール** (Python 3.7 以降):

```bash
pip install chloros-sdk
```

pip すら必要ない場合があります。すべてのインストーラには、対応する SDK wheel が付属しています。Windows インストーラは、これをシステムの Python に自動インストールします。 Linux `.deb` は、これを `/usr/lib/chloros/sdk/` に配置し、正確な `pip install --user` コマンドを出力します。 PyPIはリリースビルドごとに更新されるため、`pip install chloros-sdk`は最新の安定版リリースと一致します。

**手順3 — マシンごとに1回ログインする:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

認証情報は `~/.chloros/` にキャッシュされます（両プラットフォームとも）。Windows では、デスクトップアプリの [ユーザー](<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">) タブから同様にサインインできます。 SDK では、有料の Chloros+ プランが必要です。詳細は以下の [ライセンス要件](#license-requirement) をご覧ください。

| 要件 | 詳細 |
| --- | --- |
| **Chloros がインストールされていること** | Windows：デスクトップインストーラー； Linux: `.deb` パッケージ（バックエンドバイナリを提供） |
| **Python** | 3.7 以降（3.10 で開発・テスト済み） |
| **オペレーティングシステム** | Windows 10/11 64ビット、Ubuntu 22.04 LTS以降、またはNVIDIA Jetson (JetPack 6) |
| **ライセンス** | 有効な Chloros+ ログイン、任意の有料プラン（Copper 以上） |

## わずか60秒で完了

1回の呼び出しで、プロジェクトの作成、フォルダのインポート、処理の設定、パイプラインの実行が行われます。バックエンドがまだ実行されていない場合は、自動的に起動されます：

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Linux では、Linux のパスを使用してください：`/home/user/drone_images/flight001`。SDK は両プラットフォームで同様に動作します。)

LATTICEのキャプチャフォルダを処理しますか？ LATTICE対応のラッパーを使用してください。これにより、適切なデフォルト設定（パネルターゲットの検出なし、標準のデベイヤー）が適用されます：

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — パイプラインの完全な制御

1行のコマンドを超える処理を行う場合は、`ChlorosLocal`を使用してください。 これは、初回使用時にバックエンド（`auto_start_backend=True`）を起動し、プロジェクトの作成と設定を行い、進行状況を監視し、実行後の要約を返します。

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
`localhost` に置き換えるのではなく、デフォルトの `http://127.0.0.1:5000` を維持してください。Windows では、 `localhost`は最初に`::1`に解決され、IPv4専用バックエンドに対してはリクエストあたり約2秒のコストがかかります。
{% endhint %}

確実にクリーンアップを行うためのコンテキストマネージャーとして使用してください：

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` は以下のキーワードを受け付けます： `debayer`、`vignette_correction`、`reflectance_calibration`、`indices`、`export_format`、 `ppk`、`daq_log_path`、`input_level`、`radiometric_output`、`array_alignment`、 `array_alignment_crop`、`array_alignment_interpolation`、および`custom_settings`。主な値：

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

LATTICE固有のノブ （`input_level`、`radiometric_output`、および `array_alignment*` ファミリー）については、完全な値表とともに [SDK リファレンス](reference/sdk-reference.md#supported-values) に完全な値の表とともに記載されています。

### 進行状況の確認

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### 実行後の要約の読み取り — および空の実行の検出

完了時、`process()`はバックエンドの処理概要を`result["summary"]`として添付します。`summary["hints"]`の各エントリは、注目すべき点（例えば、 なぜ実行結果がゼロになったのか — といった点について、完全な文で説明されています。また、すべてのヒントは Python `UserWarning` として再出力されるため、辞書を確認しなくても、空の実行は自己診断が可能です：

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()`は、実行で画像が生成されなかった場合には発生しません。** SDKとCLIが意図的に異なる点はここだけです： `chloros-cli process` は「出力ファイルが要求されたが、何も書き込まれなかった」状態を失敗と見なし、ゼロ以外の値で終了しますが、SDK は通常通り終了し、その状態を `summary` やヒントを通じて報告します。 パイプラインが空の実行時に停止すべき場合は、例外に依存するのではなく、ご自身で `summary` を確認するか（またはプロジェクトフォルダ内のファイル数を数えるか）してください。
{% endhint %}

## Smart Connect — ライブハードウェア

3つのヘルパーが、バックエンドのハードウェアプールで永続的なセッションを開きます。これはGUIが使用するプールと同じであるため、SDKスクリプトは、シリアルポートやネットワーク帯域幅の競合を起こすことなく、デスクトップアプリと共存します。3つすべてが、ローカルバックエンドが実行されていない場合、自動的にローカルバックエンドを起動します。

### 単一のLATTICEカメラ — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### 同期化されたアレイ — `connect_array`

`connect_array`は、マルチカメラリグ向けの推奨エントリポイントです。GUIと同じスマート準備フロー（ネットワーク分析、同期ティアの自動選択、PTP時刻同期、カメラごとのピクセルフォーマット選択、AEシード、GPIOトリガーのアーム）を実行します。 **最初のシリアルがマスター**（ハードウェアトリガーパルスを発射）となり、残りはスレーブとなります。

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

`smart=True`を任意のアレイキャプチャに追加すると、トリガーを発動する前に、すべてのカメラで自動露出が安定するのを待機します。 キャプチャモード（シングル／連続／インターバル／最速）、レコーダー、バースト・トゥ・ビデオ、およびアレイの位置合わせについては、[SDK リファレンス](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep) を参照してください。

### DAQ 光センサー — `connect_daq_sensor`

引数を指定しない場合、`connect_daq_sensor()`は通信プロトコルを自動検出します（優先順位：イーサネット → BLE → USB）：

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

各フレームには、135ポイントの`spectrum`（校正済みの場合はW/m²/nm）、`is_saturated`フラグ、およびCIE `x`、 `y`、`z`が含まれます。特定のセンサーやトランスポートを指定するには（複数のネットワークインターフェースを持つホストでは、イーサネットの自動検出が最初の試行で正常なDAQ-Eを見逃す可能性があるため、これが確実な選択肢となります）、1つの明示的なヒントを渡します：

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

なお、キャップ補正プロファイル（`cap_id`）は、**SDK** ノブとは異なります。代わりに、`chloros-cli daq pool-connect --cap-id …` または `pool-set-cap` 経由で選択してください。

### 保存されたプロジェクト — `open_project`

保存された Chloros プロジェクトは、接続されたハードウェア（`cameras.json` + `sensors.json` および `project.json`）の状態を維持し、 `chloros_sdk.open_project(path)`はこれらすべてを一度に再接続し、デバイス名でキャプチャを実行できます。リファレンスの[プロジェクトの自動化](reference/sdk-reference.md#project-automation--chlorosproject)を参照してください。

## pipのみでのインストールで得られるもの

ハードウェアサーフェスを使用する前に、モジュールレベルの可用性フラグを確認してください：

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

**`pip install chloros-sdk`**のみがインストールされており、Chlorosデスクトップパッケージがインストールされていないホストでは：

* `ChlorosLocal`、`process_folder`、および `process_lattice_capture` は動作**しません**。これらは、デスクトップインストーラに含まれているバックエンドバイナリを必要とします。
* スマートコネクト・ヘルパー（`connect_camera`、`connect_array`、 `connect_daq_sensor`）は純粋な HTTP クライアントであるため、別のマシン上のバックエンドに対しては動作します。ただし、同梱のバックエンドはループバックにのみバインドされるため、ポートの転送は自身で行う必要があります（例： `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`）し、`backend_url="http://127.0.0.1:5000"`を`auto_start_backend=False`と共に渡す必要があります。[リモートバックエンドモード](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)を参照してください。
* ハードウェアに直接アクセスする LATTICE クラス（`LatticeCamera`、`CameraPool`、 …）はインポートできますが、デスクトップパッケージに含まれる Arena SDK ランタイムが必要です。これがない場合、`CAMERA_AVAILABLE` は `False` となります。
* `daq_sdk`（ダイレクトDAQクラス）は、PyPIパッケージではなくデスクトップインストールに同梱されているため、 そのため、pipのみのホスト上では`DAQ_AVAILABLE`は`False`となります。代わりに、（トンネル経由の）バックエンドに対して`connect_daq_sensor()`を介してDAQセンサーを駆動してください。

## ライセンス要件

SDKへのアクセスには、有料プラン（**Copper以上**（Copper / Bronze / Silver / Gold））のいずれかで有効なChloros+ログインが必要です。 無料の Iron プランでは、SDK/CLI へのアクセスはできません。 この制限は**サーバー側**で適用されます。すべてのSDKリクエストには、有効なセッションと有料プランの両方が含まれている必要があり、そうでない場合、バックエンドは`403` / `PLAN_UPGRADE_REQUIRED`を返します （`ChlorosLocal` によって `ChlorosLicenseError` として、また `connect_*` ヘルパーによって `ChlorosConnectError` として生成されます）。 ログアウトした呼び出し元には、代わりに `401` / `AUTH_REQUIRED` （`ChlorosAuthenticationError`）が返されます。`chloros-cli login`を再実行すると、前者のケースは修正されますが、後者のケースは修正されません。

オフラインでの使用は、プランの猶予期間内であれば機能します。ティアは、サーバー検証キャッシュ（5 分）または署名済みでマシンに紐付けられたライセンスキャッシュ（月額プランの場合は 30 日、年間プランの場合はサブスクリプションの有効期限まで）から読み取られます。 猶予期間が終了すると、プランは無料プランに切り替わり、SDK へのアクセスは、そのマシンがサーバーに一度接続するまで停止します。`chloros-cli status` は無料プランでもアクセス可能な状態を維持するため、その理由は常に確認できます。 [Chloros+ ログイン](chloros+-login.md)を参照してください。

## 例外

「Chlorosで何か問題が発生した」場合を処理するために、基底クラスをキャッチします：

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

すべてのパイプライン例外（`ChlorosBackendError`、`ChlorosConnectionError`、`ChlorosLicenseError`、`ChlorosAuthenticationError`、 `ChlorosConfigurationError`、`ChlorosProcessingError`）はすべて、`ChlorosError`に由来します。 注意点：`ChlorosConnectError` — `connect_camera` / `connect_array` / `connect_daq_sensor` — は、単純な `Exception` から派生しており、`ChlorosError` からは**派生していない**ため、`except ChlorosError` ではこれを捕捉できません。 完全な階層構造は、[SDK リファレンス](reference/sdk-reference.md#exceptions) に記載されています。

## 関連項目

* [SDK リファレンス](reference/sdk-reference.md) — AI アシスタント向けに最適化された、API の完全なサーフェス。
* [CLI リファレンス](reference/cli-reference.md) — すべての CLI サブコマンドは、SDK 呼び出しに対応しています。
* [ダウンロード](download.md) — Windows および Linux のインストーラー。
