# Chloros Python SDK リファレンス

**バージョン:**

1.2.0**生成日:**2026-07-29 19:19 ·**改訂日:** 2026-08-30**パッケージ:** `chloros-sdk` (PyPI)**対象:** LLMでの利用に最適化されており、 人間が読みやすい形式。**範囲:** `import chloros_sdk` によって公開されているすべてのパブリッククラス、関数、およびヘルパー。画像処理、単一カメラ制御、同期化された配列、DAQ センサー、プロジェクトの自動化を網羅した、コピー＆ペースト可能な例を掲載しています。

要点だけ知りたい場合は、以下へ進んでください：
- [インストールとクイックスタート](#installation)
- [LATTICEアレイ用Smart-Connect](#smart-connect-for-lattice-cameras)
- [DAQセンサーセッション](#daq-sensor-sessions)
- [プロジェクトの自動化](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## 60秒でわかるアーキテクチャ

SDKは、Chlorosバックエンドの上に構築された、Pythonの薄いレイヤーです （デスクトップGUIやCLIが使用するのと同じFlaskサーバー）の上に構築された、の薄いレイヤーです。自動化を行うには、`chloros_sdk`をインポートして高レベルなメソッドを呼び出します。内部的には、すべての呼び出しがポート5000上のローカルバックエンド（`http://127.0.0.1:5000/api/...`）へのHTTPリクエストに変換されます（意図的に`localhost`は意図的に避けています。これはWindowsで最初に`::1`に解決され、IPv4のみのバックエンドに対してはリクエスト1回あたり約2秒の処理時間がかかるためです）。バックエンドはハードウェアプール（カメラ、DAQセンサー、アライメントプロファイル、 フレームバッファなど）を管理しているため、SDKスクリプトは、シリアルポートやNICの帯域幅を奪い合うことなく、GUIと共存できます。

使用するインターフェースは3つあります：

1. **`ChlorosLocal` + フリー関数** (`process_folder`、`process_lattice_capture`) — 画像処理パイプライン。 Pythonを1回呼び出すだけで、フォルダ全体のキャリブレーション／デベイヤー処理／インデックスエクスポートを実行できます。
2. **Smart-connectハンドル** (`connect_camera`、`connect_array`、`connect_daq_sensor`) — ライブハードウェア用の永続的なバックエンドセッションを開きます。GUI と同じ「smart-prep」フロー：ネットワークプローブ、ティアの自動選択、PTP、AE シーディング、GPIO トリガー設定。
3. **`ChlorosProject` / `open_project`** — 保存済みのプロジェクトを読み込みます（`cameras.json` + `sensors.json` + `project.json`）を含むフォルダ）を読み込み、すべてを一度に接続し、名前付きハンドルを使用してキャプチャを実行します。

サーフェス 1 および 2 は、まだリスニング状態になっていない場合、**ローカルバックエンドを自動起動**します（GUI や CLI が起動する、同じバンドルされたバイナリです）。そのため、バックエンドを事前に起動しなくても、新しいシェルからスクリプトをそのまま実行できます。`auto_start_backend=False`を渡してこの機能を無効にできます（例：リモートバックエンドを指定する場合。リモートバックエンドは決して起動されません）。[バックエンドの自動起動](#backend-auto-start)を参照してくださいを参照してください。Surface 3 の動作は異なります。`open_project()` は `auto_start_backend` パラメータを受け付けず、`connect_all()` はバックエンドを起動することはありません — `http://127.0.0.1:5000`に対して一度プローブを行い、応答がない場合は、 黙って（バックエンドを使用しない）直接的な `lattice_sdk` デバイス制御に切り替わります。`proj.process()` と `stream(..., overlays=True)` のみが、`ChlorosLocal()` を遅延生成しますXを遅延生成します（これは自動起動を行います）。

これら3つすべてに認証要件があります：マシン上で`chloros-cli login`を1回実行するか、デスクトップGUI経由でサインインしてください。有効なセッションがない状態でSDKを呼び出すと、`ChlorosAuthenticationError`が発生します。

要件：
- Python 3.7 以上（パッケージの記載通り。3.10 で開発・テスト済み）
- ローカルにChloros Desktopがインストールされていること（バックエンドバイナリはインストーラー内に同梱されています）
- 有効なChloros以上のログイン。SDK / CLIへのアクセスは、**Copper**ティア以上（Copper / Bronze / Silver / Gold）が対象となります。無料の**Iron**ティアでは、SDK / CLIへのアクセス権がありません。これは**サーバー側**で強制されます： SDK / CLI のフラグが設定されたすべてのリクエストは、有効なセッションと有料プランの両方を保持している必要があります。そうでない場合、バックエンドは `403` と `error_code: PLAN_UPGRADE_REQUIRED` を返します（`ChlorosLocal`として表示され、 および `connect_*` ヘルパーによって `ChlorosConnectError` として表示されます）。ログアウトした呼び出し元には、代わりに `401` / `AUTH_REQUIRED` （`ChlorosAuthenticationError`）が返されます。これら2つは異なるものです。なぜなら、- 実行中の `chloros-cli login` は前者を修正しますが、後者を修正することはできません。
- プランの猶予期間内であればオフラインでの利用がサポートされています。この期間中は、サーバー検証キャッシュ（5分間）または署名付きでマシンに- ライセンスキャッシュ（月額プランの場合は30日間、年間プランの場合はサブスクリプションの有効期限まで）から読み込まれます。この猶予期間が終了すると、プランは無料プランに切り替わり、SDK / CLI へのアクセスは、マシンがサーバーに一度接続できるまで停止します。`chloros-cli status` (`GET /api/license-status`) は無料プランでもアクセス可能な状態を維持されるため、その理由が明らかです。これは、SDK / CLI へのルートの中で、プラン制限の対象外となっている唯一のルートだからです。
- Windows 10/11 64ビット、**Ubuntu 22.04 LTS 以降**、または Jetson (JetPack 6)。Ubuntu 20.04 は**サポート対象外**：`.deb`の依存関係は、`libc6 (>= 2.34)`を含め、バックエンドがリンクするライブラリに由来しており、focalにはglibc 2.31が同梱されているためです。

---

## インストール

Python（SDK）は、Chlorosバックエンドの上に構築された、Pythonの薄いレイヤーです。DAQのみのワークフローを数件超えるすべての用途については、**Chlorosデスクトップパッケージをローカルにインストール**する必要があります（WindowsインストーラまたはLinux `.deb`）。これにより、バックエンドバイナリ、 LATTICEカメラ用のArena SDKランタイム、およびキャリブレーションバンドルを提供します。

最新のダウンロード： [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### ステップ 1 — 「Chloros」プラットフォームパッケージのインストール

#### Windows (.exe)

1. ダウンロードページから `Chloros-Setup-x.y.z.exe` をダウンロードします。
2. インストーラーを実行し、ウィザードの指示に従います。デフォルトのインストールパスは `C:\Program Files\MAPIR\Chloros\` です。
3. 「Chloros」を少なくとも1回起動し、Chloros+ アカウントでサインインします。

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### ステップ 2 — Python をインストールする SDK

**Chlorosインストーラーには、対応するSDKホイールが同梱されています。** すべてのWindowsインストーラーおよびLinux .debパッケージは、GUI / CLI / バックエンドのバージョンと完全に一致する`chloros_sdk-X.Y.Z-py3-none-any.whl`をディスクに配置します。同期を保つためにPyPIを追いかける必要はありません。

#### Windows

インストーラーは、システムのPythonを使用して、バンドルされたwheelファイルに対して`pip install`を自動実行します（`py.exe`ランチャーが優先され、それが利用できない場合は`python -m pip`にフォールバックします）。特別な操作は不要です。インストールが成功すると、`import chloros_sdk`は、インストールが正常に完了すれば、Python環境でも動作します。対象マシンにPythonが存在しない場合、インストーラはこの手順を黙ってスキップし、GUIおよびCLIは引き続き動作します。

#### Linux (.deb)

この .deb パッケージは、wheel ファイルを `/usr/lib/chloros/sdk/` に配置します。`postinst` は正確なコマンドを出力します。PEP 668 準拠のディストリビューションでは、デフォルトでグローバルな pip への書き込みが拒否されるため、自動インストールは行われません：

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

エアギャップ環境の Jetson へのデプロイの場合、これは完全にオフラインで行われます。wheel ファイルはすでにディスク上に存在します。

#### パブリック PyPI

pip のみのホスト（Chlorosのデスクトップパッケージがインストールされていない場合、リモートバックエンドまたは DAQ 専用のワークフロー）の場合:

```bash
pip install chloros-sdk
```

PyPIはリリースバージョンのインストーラビルドに合わせて更新されるため、公開されているwheelは最新の安定版リリースと一致します。開発版ビルド（例: `1.1.4.dev1`）は、バンドルされたインストーラwheelを介してのみ提供されます。

#### 確認

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ のサブスクリプションが必要です。** すべての SDK への呼び出しには、有効な Chloros+ ログインが必要です。各マシンで `chloros-cli login user@example.com 'YourPassword'` を一度実行してください。認証情報は `~/.chloros/` にキャッシュされます。

### デスクトップパッケージは必要ですか？

ほとんどのワークフローでは、pipパッケージだけでは**不十分**です。各SDKサーフェスに必要なものは以下の通りです：

| SDKサーフェス | デスクトップパッケージが必要か？ | 理由 |
| --- | --- | --- |
| `ChlorosLocal`、`process_folder`、`process_lattice_capture` | **はい** | `/usr/lib/chloros/chloros-backend` (Linux) または `C:\Program Files\MAPIR\Chloros\…` (Windows) でバックエンドバイナリが自動起動します。 |
| `connect_camera`、`connect_array`、`connect_daq_sensor`、`analyze_array_network`、`list_*`, `discover_*` | **はい**(ローカル)**/ いいえ**(リモート) | バックエンド経由の純粋なHTTPクライアント。ローカルバックエンド → デスクトップパッケージが必要。リモートバックエンド →**トンネル経由**の`backend_url=`（「リモートバックエンドモード」を参照 — 同梱のバックエンドはループバックのみにバインドされます）。 |
| `ChlorosProject` / `open_project` | **はい** | 保存されたプロジェクトをバックエンド経由で読み込みます。 |
| 直接 LATTICE クラス (`LatticeCamera`, `CameraPool`、`Calibration`、`DLS`、…) | **はい** | デスクトップパッケージに同梱されているArena SDK ネイティブランタイムが必要です。そうでない場合、`CAMERA_AVAILABLE`はインポート時に`False`となります。 |
| ダイレクトDAQクラス (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`） | **いいえ** | pyserial/bleak/zeroconf 経由の純粋なPython。pipのみの環境でもDAQをエンドツーエンドで駆動可能。 |

### リモートバックエンドモード（pipのみのホスト、トンネル経由）

> **同梱のバックエンドはLAN経由では到達できません。** 本番
> ビルドはループバックのみ（両方のループバックファミリー）にバインドされ、
> 唯一の非ループバックモード（`CHLOROS_CLOUD_MODE`）を完全に拒否するため、
> `backend_url="http://<lan-ip>:5000"`は **は、インストール済みの
> Chloros** に対しては動作しません — このパターンは、ソース/dev
> バックエンドに対してのみ動作していました。別のマシン上のバックエンドを駆動するには、そのループバック
> ポートを自分で転送し、SDK をトンネルに向ける必要があります：

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

ヘッドレス／CI／ロボティクスホストでは、フルデスクトップ環境がインストールされた1台のマシンを「Chlorosサーバー」として残し、その他のすべてのマシンに`pip install chloros-sdk`を設定することができます。ただし、それらの間の転送は、上記のユーザー-で設定された上記のトンネルを介して行われ、直接の LAN URL 接続ではありません。

> **既知の制限事項 — `ChlorosLocal` は pip のみでの動作には対応していません。** `ChlorosLocal(backend_url=BACKEND)` は現在、コンストラクタ内でローカルバックエンドバイナリを解決してから *URL* を検索しており、 、デスクトップパッケージがインストールされていない場合 — リモートバックエンドに接続可能であっても — `ChlorosBackendError`（「Chlorosバックエンドが見つかりません…」）を発生させます。上記のスマートコネクト機能（`connect_camera` / `connect_array`X / `connect_daq_sensor`、および `analyze_array_network` と `list_*` / `discover_*` ヘルパー）のみが、pip のみのホストから動作します。

### DAQ専用ワークフロー（pip専用ホスト）

DAQセンサーのみが必要で、LATTICEカメラや画像処理を扱わない場合、pipパッケージだけで完結します：

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

バックエンドも.debパッケージも不要で、Chlorosへのログインも不要で、直接ハードウェアに依存しないDAQ作業を行うために、バックエンドも.debパッケージもへのログインも不要です。

---

## クイックスタート

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## トップレベルのAPIインデックス

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## 画像処理 — `ChlorosLocal`

主要なパイプラインクラスです。初回使用時にバックエンドを起動し、プロジェクトを作成・設定し、進行状況を監視し、実行後の要約を返します。

### コンストラクタ

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

### メソッド

| メソッド | 説明 |
| --- | --- |
| `create_project(project_name, camera=None)` | 新しいプロジェクトを作成します（オプションで、`"Survey3N_RGN"` のようなカメラテンプレートを使用することも可能です）。 |
| `import_images(folder_path, recursive=False)` | RAW/TIF/JPG/DNG 画像 **および `.daq` 光センサーの記録** をインポートします。 `count`（画像）および `scan_count`（記録データ）を返します。フォルダにどちらも含まれていない場合にのみ警告を表示します。 |
| `export_light_sensor(daq=True, csv=True)` | プロジェクト内の各光-センサーの記録ごとに、`<project>/Light Sensor/`に書き込みます。[光センサーの記録](#light-sensor-recordings--calibrated-daq--csv)を参照してください。 |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | 処理パラメータを設定します。 |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | パイプラインを実行します。`{"status": "complete", "async": False}`に加え、バックエンドが提供する場合、`summary`キーが返されます（バックエンドが提供している場合）— [実行後の要約とヒント](#post-run-summary--hints)を参照してください。 |
| `get_config()` / `get_status()` / `status()` | バックエンドの状態を確認します。 |
| `logout()` | キャッシュされた認証情報をクリアします。 |
| `shutdown_backend()` | バックエンドを終了します （SDKで起動された場合）。 |
| `discover_cameras()` | **このインスタンスのバックエンド経由で** LATTICEカメラを検出（`/api/camera/discover`）。辞書のリストを返す (`serial`, `model`, `ip`, …) — GUI/CLIで表示されるものと同じ形状。見つからない場合やバックエンドに到達できない場合は空のリストを返す。 |
| `camera_capture(output_dir, format="tiff", **settings)` |**バックエンドを経由して**（このハンドルによって自動起動される）単一のフレームをキャプチャし、GUI/CLI （デフォルトは12ビット、プールの再利用、埋め込みキャリブレーションメタデータ）と同じプリセットが適用されるようにします。ターゲットの解決には`serial=`または`device_index=`を使用します。`exposure`/`gain`/`pixel_format`/`preset` は `**settings` として渡してください。レガシー・メタデータ辞書 (`filepath`、`width`、`height`、`pixel_format`、 `exposure_time`、`gain`、`timestamp`）。 |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | プールされたカメラからオーバーレイ合成されたプレビューフレームを出力する — バックエンドの `/api/camera/<serial>/stream-annotated` ルート上に薄い MJPEG クライアントを配置 （サーバー側で描画されるゼブラ／グリッド／クロスヘア／ヒストグラム／ピーキング／スポット）。`decode=True`はBGR配列を、`False`は生のJPEGバイトを返します。また、-project 経由でも `ChlorosProject.stream(overlays=True)` として利用可能です。 |

確実なクリーンアップを行うためのコンテキストマネージャーとして使用してください：

```python
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

### 光センサーの記録 — キャリブレーション済み `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-Eは、キャリブレーションバンドルを**使用せずに**記録可能です。これは、
公開されている[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
レコーダー（`record_daq.py`）がデフォルトで行っている動作です： 生センサーカウントを書き込み、ファイルにタイムスタンプを付与することで、
Chlorosがそのセンサーの工場出荷時校正データを**シリアル番号ごとに**取得できるようにします（まずローカルキャッシュから、
次にMAPIRクラウドから）し、インポート時にそれを適用します。

Chlorosは、その結果を1回の記録につき2つの製品として、
`<project>/Light Sensor/`の下に書き出します：

| 製品 | 内容 |
| --- | --- |
| `<name>_calibrated.daq` | 再処理可能なアーカイブ — ライブ記録と同じスキーマですが、生成元のバンドルが明記されています。これを再インポートしても、**再度**校正されることはありません。 |
| `<name>_calibrated.csv` | センサー独自の波長グリッドにおける分光放射照度（W/m²/nm）。1行につき1つの測定値に加え、測光列（総電力、明所視/暗所視力ルクス、PPFDおよびその青/緑/赤の分割値、ピーク波長）。 |
| `<name>_raw.daq` / `<name>_raw.csv` | **バンドルのないセンサーのみ（DAQ-A）。**センサーの生スペクトルカウント —**放射照度**ではありません。以下を参照してください。 |

`process()` は、その処理段階の一つとしてこのエクスポートを実行します。 画像データは**不要**です：
単独で飛行する光センサーは独立したワークフローであり、そのようなプロジェクトには
その性質上、画像データが一切存在しません。

**DAQ-Aの記録データは生カウントとしてエクスポートされます。** DAQ-Aファミリーはシリアルごとの
バンドルシステムより前に開発されたものであり、取得すべきバンドルが存在しません。その代わりに、現場で
反射率ターゲットを用いて校正されるため、バンドルは必要とされませんでした。これらの記録データは、
`_calibrated` ではなく `_raw` というステム名でエクスポートされます。これは、 ファイル内のフラグではなく、
ファイル名そのものが異なるのです。これは、ファイル名がそのままの状態でメール転送されても情報が失われないようにするためです。
`.csv`ヘッダーには`raw spectral sensor counts (NOT irradiance)`と記載されており、
値はファイル**内**でのみ比較可能であるという警告が表示されます。これはまさに ターゲットベースのキャリブレーションが
それらを使用する目的——であり、センサー間で比較できるわけではない。 電力依存の測光カラム（総電力、
明所視／暗所視ルクス、PPFD）は、カウント値から積分されるのではなく、**NULL**として返されます。

バンドルを取得できなかったDAQ-U / DAQ-M / DAQ-Eは、**スキップ**され、
生データとして書き込まれることはありません。この場合、バンドルは存在しているため、「再接続して再処理する」ことが有効な対処法となります。

レガシーな**v1.01 / v1.02**の記録（DAQ-A-SDがこれらを書き込みます）には、読み取りごとのエポック情報が含まれておらず、
ファイルの書き込み時刻のみが記録されています。image↔downwellingマッチャーは依然としてこれらを受け付けません — フレームを
書き込み時刻と照合することは、目に見えない形で誤りを生じることになるからです — しかし、エクスポーターはそれらを読み込み、
CSV は `clock=daq_created_on` を出力するため、製品はどのクロックを使用しているかを明示します。

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

キャリブレーション・バンドルを取得できない記録（オフライン、またはファイルに
キャリブレーションがないセンサー）は、`skipped` の下に **理由を明記して** 報告されます。これらは決して
生カウントを保持する「校正済み」ファイルとして生カウントデータが書き出されることは決してありません。インターネットに接続して
再実行すると、エクスポートが完了します。

### 進行状況のコールバック

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### 実行後の要約とヒント

完了時、 `process()`は`GET /api/processing-summary`を取得し、その本文を`result["summary"]`として添付します。この取得はベストエフォート方式であり、正常な返却を妨げることはありません — 概要が利用できない場合、`process()`は通常の`{"status": "complete", "async": False}`形式にフォールバックします。`summary["hints"]`の各エントリ（例：実行結果が 出力がゼロになった理由など）も、Python形式の`UserWarning`として再出力されるため、辞書を検査しなくても、出力ゼロの実行は自己診断が可能です：

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]`は、機械可読な形式の半分の部分です：

| キー | カウント対象 |
| --- | --- |
| `models` | 実行内のカメラグループ数。 |
| `images_in_groups` | それらのグループにまたがるソース画像の数。 |
| `targets_found` | 検出された反射率ターゲット。 |
| `images_calibrated` | 当該実行でキャリブレーションされた画像。 |
| `exported_files` | **その実行で生成された画像成果物ファイル。** |
| `daq_recordings_exported` / `daq_recordings_skipped` | 光センサーの記録データ。 意図的に別々にカウントされています。これらは別の段階から生成されたものであり、画像が全くない実行でも存在するため、これらを統合してしまうと、DAQのみの実行でも画像がエクスポートされたかのように見えてしまいます。 |

これらに加えて： `summary["output_dirs"]`（書き込みが行われたすべてのディレクトリ）、
`summary["light_sensor_export"]`、`summary["stopped"]`（ユーザーが実行を中断した場合に
実行を中断した場合に真となるため、部分的なカウントが、生産量が不足した完了済み実行として認識されないようにするため）、および
`summary["groups"]`（グループごとの内訳）。

`exported_files`は、パイプラインが**書き込みを行う際**に記録されるものであり、
プロジェクトのイメージオブジェクトから事後にスキャンされるものではありません。並列およびGPU戦略は、独自のイメージ
オブジェクトを （GPUパスではワーカーサブプロセス内で）構築するため、従来のスキャンでは
そのような実行ごとに`0 file(s) written`を報告し、その後、すべてが正常に動作した実行に対してゼロ-exportsヒントを出力していました — すべてが正常に動作していた実行
においてもです。この数値に基づいてスクリプトを作成している場合、正常な並列実行では現在
ゼロ以外のカウントが報告されます。

ライトセンサーによるスキップは、リーダーが各ファイルについて実際に特定した理由 —
読み取り不能なスキーマ、バンドルの欠落、書き込みエラー — を**重複排除**して報告します。したがって、1つの原因で20ファイルが
スキップされた場合、その原因は20回繰り返されるのではなく、1つの原因として扱われます。

> **実行で画像が生成されなかった場合、`process()`は発生しません。** これは、SDKと
> CLIが意図的に異なる唯一の点です。`chloros-cli process`は、「製品が要求されたが、 書き込まれなかった
>」という状況を失敗とみなして非ゼロで終了するのに対し、SDKは通常通り終了し、その
>状況を`summary` / hintsを通じて報告します。 パイプラインが空の実行で停止すべき場合は、
> 例外が発生していないことだけに頼るのではなく、ご自身で
> `summary` を確認するか（またはプロジェクトフォルダ内のファイル数を数えてください）。 一般的な原因としては、入力フォルダが
> キャプチャとして認識されなかったことや、使用されているカメラには適用できないとして製品がスキップされたこと（例：RGB -only
> カメラからのラディアンス）などが挙げられます。

### 便利関数

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### サポートされている値

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### 放射測定出力（LATTICE 多スペクトルパイプライン）

`process` パイプラインの LATTICE マルチスペクトル（M3C/M3M）エクスポートレベル — `reflectance` （デフォルト）、`radiance`、`sensor-response`、または `all`（画像ごとに適用可能な各モード） — は、プロジェクトの**「放射測定出力」** 処理設定に対応します。`configure()` には専用のキーワードがあります：

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

高度な回避策 — `"Radiometric output"` キーを `custom_settings` を通じて記述する — は依然として機能しますが、 ただし、これにより設定ブロック全体が上書きされる点に注意してください（以下の警告を参照）：

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance`（デフォルト）は、カメラの放射輝度を**タイムスタンプが一致するDAQダウンウェリング**で除算します。これは、記録された`.daq`（DAQ-U/M/E）**、**または画像と共に存在するDAQ-Mネイティブの`.csv`**から自動的に解決されます。カメラごとやDAQごとの キャリブレーションバンドルがローカルに存在しない場合は、初回使用時に**AWSから自動的に取得**されます。CLIは、これを`chloros-cli process`上のタイプ別製品のトグルとして公開します：`--radiance`/`--no-radiance`、`--reflectance`/`--no-reflectance`、`--debayered`、`--preview`。

> `custom_settings`は、計算された設定ブロック全体を**置き換えます**（設計上、 `configure()`の他のキーワードや検証を意図的にバイパスします）。これを使用する際は、上記の例のように、必要なすべての`Project Settings`キーを含めてください。

---

## LATTICEカメラ用Smart-Connect

ライブハードウェア向けの永続的なバックエンドセッション。GUIが使用するエンドポイントと同じであるため、SDK / CLI / GUI間で動作が同一です。

### 単一カメラ — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()` シグネチャ

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` メソッド

| メソッド | 説明 |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICam ノードを読み込み、`{nodes, errors, enums, device}` を返します。 |
| `set_settings(**kwargs)` | フレンドリー名によるノードの書き込み（`exposure_time`、`gain`、`pixel_format`、 `width`、`height`、`target_brightness`、`ae_damping`、`ae_upper_limit`、 `trigger_mode`, `trigger_source`, …)。 |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | **1** フレームをキャプチャします。フレームのメタデータ辞書からなる 1 要素のリストを返します。（バースト／マルチフレームキャプチャは削除されました。一連のフレームが必要な場合は、ループ内で `capture()` を呼び出してください。） |
| `disconnect()` | プールから解放します。すでに開かれているセッションにアタッチされている場合は、何もしません。 |

`capture()` エクスポート制御 （配列＋GUIと同じモデル）：

- `processing` / `levels` — `processing="all"` は該当するすべてのエクスポートタイプを保存します。`levels=["raw","radiance"]` は （`processing`を上書きします）。バックエンドのデフォルトを使用する場合は、両方を省略してください。
- `force_daq=True` — 割り当てられたDAQ/DLSの測定値を`.daq`サイドカーとして保存します。これにより、後でフレームを反射率/屈折率データに再処理することが可能になります。DAQがリンクされていない場合は何もしません。

### 同期アレイ — `ArraySession` (Smart-Prep)

`connect_array`は、マルチカメラ設定における**推奨のエントリポイント**です。内部でGUIベースのSmart-Prepフロー全体を実行します：

1. **ネットワーク分析** (`/api/camera/array/recommend`) — フレームをドロップすることなく、sim-emitティアに収まる最大のフレームサイズを特定します。
2. **ティアの自動選択** — 回線が対応可能な場合は `sim-capture-sim-emit`、そうでない場合は `sim-capture-ftd-stagger` または `slip-emit-and-capture`。
3. **自動縮小**— 回線が要求された解像度を維持できない場合、フレームサイズを自動的に縮小／ビニングを自動的に増加させます。**この安全策は、集約的なオーバーサブスクリプションには対応しません**： 回線に対してカメラ数が多すぎる場合は、フレームの縮小では解決できません — [オーバーサブスクリプション](#over-subscription-the-per-cam-floor)を参照してください。
4. デフォルトで**PTP有効** — カメラ間のタイムスタンプはマイクロ秒単位で比較可能です。
5. **カメラごとのピクセル形式の自動選択** — RGB カメラ → `BayerRG8`、 マルチスペック → `BayerRG12`。
6. **AEの初期化** — 各カメラの現在のAE状態をスナップショットとして保存し、接続中に露出設定がリセットされないようにします。
7. **GPIOトリガー設定** — `connect_array`がすべてのカメラ（`TriggerMode=On`、`TriggerSource=Line2`）をアームし、マスターのパルスがM8ケーブルを介してスレーブを駆動するようにします。これはアレイ専用 手順です。`LatticeCamera`で単一のカメラを開いた場合は、代わりにフリーラン動作になります。

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` シグネチャ

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier` の値：
- `"sim-capture-sim-emit"` — 完全な同時動作（すべてのカメラが同じクロックエッジで撮影を開始する）。
- `"sim-capture-ftd-stagger"` — 柔軟な時間ドメインのスタッガー（カムがわずかにずれたタイミングで送信されるため、ワイヤ上でパケットが直列化される）。
- `"slip-emit-and-capture"` — カムごとの順次キャプチャ（時間的同期なし；シミュレーションに適合するフレームサイズがない場合の唯一の選択肢）。

`wire_ceiling_mbps`は、**ホストの持続的なワイヤ帯域幅** （MB/s単位）を上書きします。これは、アレイ全体の割り当てが依存する唯一の
数値です。自動検出された
値を使用するには、`None`のままにしておきます。アレイがGVSP破損フレームを報告した場合は、この値を小さくしてください。自動値は
NICが通知するリンクレートから導出されており、USBアダプタ、帯域の狭いPCIeレーン、
負荷の高い共有ファブリックでは過大評価されがちです。この過大評価は、
目に見えるリンク速度の低下ではなく、破損したフレームとして現れます。この値はプロジェクトのアレイキャプチャブロックに永続化されるため、
プロジェクトを再開したり、後で `connect_array` を実行したりすると、他のアレイ設定と同様に復元されます。
[アレイの健全性](#array-health--which-subsystem-is-losing-frames) を参照してください。

#### オーバーサブスクリプション（カメラごとの下限値）

Sim-emit ペーシングでは、各カメラに衝突防止用のワイヤ帯域幅の割り当てが行われ、その下限は **カメラあたり 8 MB/s**(`per_cam_floor_bps`) です。`N × floor` が衝突上限を超えると、アレイは**ワイヤをオーバーサブスクリプション**状態となります。この場合の障害モードはGVSPパケット損失であり、フレームレートの低下ではありません。また、フレームサイズによる解決策は存在しません：**ビニングやROIのフレームあたりのバイト数を減らすこと**であり、集計チェックが比較対象とする**1秒あたりのペース制御されたバイト数**ではありません。1 GbEホストにおける実用的なフル解像度の上限：**MTU 1500のカメラ6台、ジャンボフレーム使用時は9台**（分析応答内の`max_cams_collision_safe`が、当該回線の上限値を報告しています）。対策：CAMの数を減らす、 エンドツーエンドでのジャンボフレームの使用、またはより高速なNICの導入。

- `analyze_array_network()`および`/api/camera/array/connect`の応答には、`oversubscribed`、`aggregate_demand_bps`、`collision_safe_ceiling_bps`、`max_cams_collision_safe`、 および `per_cam_floor_bps` を含みます。`oversubscribed` が true の場合、プロジェクションは **fps フィールドを 0 に設定します**（`achievable_fps_max` / `fps_bright` / `fps_dark`）を**ゼロに設定**し、誤解を招くような「遅いけれど動作している」というレートを報告することはありません。
- `POST /api/camera/array/connect`は、`pin_resolution`のボディパラメータを受け付けます（**HTTP のみ — SDKのkwargではありません**； `connect_array`はこれを公開しません）。ピンニングを行うと、ビニングによるウォークダウンのセーフティネットが取り除かれるため、`pin_resolution`が設定された状態でオーバーサブスクライブされた接続は、**すべての是正策を列挙したエラーメッセージとともに完全に拒否** され、すべての是正策を列挙したエラーメッセージが表示されます。ピンニングを行わない場合、接続はウォークダウン処理を進めますが、縮小しても集計値をクリアできないという警告が表示されます。
- ベンチワーク用の回避策：バックエンドの環境で `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` を設定すると、拒否を「目立つ警告」に格下げできます。これにより、接続は実行されますが、パケット損失を受け入れる必要があります。

#### アレイの健全性 — どのサブシステムでフレームが失われているか

`GET /api/camera/array/<array_id>/capability`は、接続されたアレイ上のアクティブな`health`ブロックを保持しており、
**10秒**のローリングウィンドウで再評価されます。これは、フレーム損失を
を、原因を特定しない単一の「不完全」レートとして扱うのではなく、
互いに相反する修正を必要とする2つの原因に分類します：

| 項目 | 意味 | 対象サブシステム |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (シリアルごと) | フレームが**到着したが構造的に不正だった**— GVSPパケット損失。 |**ネットワーク**: ワイヤ予算、ペーシング、 NIC RXリング、MTU |
| `never_arrived_rate_pct`（シリアルごと） | フレームが**全く到着しなかった**— カメラが起動しなかったか、何も送信されなかった。 |**トリガー／同期**: M8ケーブル、`line=`、`TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 各カメラごとの最悪のレート。 | — |
| `per_cam_rate_pct` | カメラごとの未完了率の合計（両方の原因を合わせたもの）。 | — |
| `stable_for_seconds` | 各カメラが 0.01 % 未満の状態を維持していた期間。 | — |

`health` と併せて、同じレコードには、割り当て全体が滞留している数値が報告されています：

| フィールド | 意味 |
| --- | --- |
| `wire_ceiling_mbps` | ホストの有効な持続ワイヤ帯域幅（MB/s）。 |
| `wire_ceiling_source` | その数値の出典（例：`USB-capped 200 MB/s (was theoretical 1062; …)` または `user override 120 MB/s (auto said 200)`）。 |
| `wire_ceiling_is_user_set` | `true` が `wire_ceiling_mbps=` によって設定された値。 |
| `nic_is_usb` | USB イーサネットアダプタの場合、`true`はUSBイーサネットアダプタ用です。 |

このエンドポイント用のSDKラッパーはありません。直接読み取ってください：

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**読み取り方法：** 0 以外の `gvsp_corrupt_rate_pct` で、`never_arrived_rate_pct` が 0 の場合、
トリガーとケーブルの同期は完璧であり、損失の 100 % がネットワーク経路に起因することを意味します。この値を
`wire_ceiling_mbps`に下げて再接続してください。逆のパターンは、同期ケーブルまたは
トリガーラインに問題があることを示しています。

> **`target_fps`は、フレーム破損の決定要因ではありません。** GevSCPDのペーシングは
> 接続時に一度だけ設定されるため、トリガーレートを下げてもデューティサイクルは変化しますが、
> 同時送信バーストレートは変化しません。測定上、要求量を5倍削減しても改善は見られませんでしたが、
> ワイヤの 上限を240 MB/sから200 MB/sに下げたところ、同じリグで破損率が10.4 %から
> 0.00 %に低下しました。

> **TRI032Sファームウェアでは、ストリーム途中の自動縮小機能は利用できません。** 稼働中のアレイでは
> これを自身で修正することはできません。接続を解除して再接続し、接続時間ピッカーが
> 新しい上限値に基づいて再計画を行うようにしてください。

**USBイーサネットアダプタは、**その
銘板に記載された値にかかわらず、プローブによって **200 MB/s** に制限されます。リンク速度を持続転送速度に変換する効率表は
PCIeに基づいており、USB NICはイーサネットリンク速度を広告しつつも、
USBバスおよびそのドライバによって制限されます。この上限は相対的なものではなく絶対的な値です。つまり、USB 1 GbEアダプタは
約80 MB/sの速度しか得られず、この制限の影響を受けません。

#### `ArraySession` メソッド

| メソッド | 説明 |
| --- | --- |
| `status(timeout=10.0)` | ライブ `{fps, ptp, frame_count, last_error, …}`。 |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | 1つの同期キャプチャグループ。`CaptureResult`を返しますX（フレーム辞書のリスト + `.skipped`）を返します。エクスポート制御については以下を参照してください。 |
| `capture(..., smart=True)` | **スマートキャプチャ** — すべてのカメラで AE が安定するのを待ってからトリガーします。 |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | 最速キャプチャ：RAWのみ + 割り当てられたDAQ読み取り値 (+ フリーの結合インデックス)。GUIの「Fastest Capture」ボタンと同様の動作をします。 |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | 1つの境界付きループ内で、シングル／連続／インターバルを実行します。`list[CaptureResult]`を返します。**終了させるには `count` および／または `duration_s`** が必要です（SDKには Ctrl+C 機能がありません）。 |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | ライブの複合インデックスビューの録画をビデオ/GIFとして開始 → `RecorderHandle`。配列ごとに1つの複合レコーダー。 |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | 高フレームレートの生ベイヤーバーストを開始 → `RecorderHandle`。 `build_video()` を使用してオフラインで再処理します。 |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | 保存された生バーストデータを、キャリブレーション済みの動画にオフラインで再処理します。処理が完了するまでブロックし（`wait=True`）、 `{outputs, errors, combined}`を返す。 |
| `build_video_status(job_id, timeout=15.0)` | オフラインビルドジョブをポーリングする：`{running, result, error, burst_dir}`。 |
| `disconnect()` | 配列全体を解放します。 |

`capture()` エクスポート制御（GUI や CLI が使用するのと同じエンドポイント）：

- `processing` / `levels` — `processing="all"` （または `levels=["raw","radiance",…]`）は、カメラごとに適用可能なすべてのエクスポートタイプを保存します。単一の `processing` 値は、そのレベルのみを保存します。
- `aligned=True` — すべてのメンバーの非-rawエクスポートを配列の[アライメントプロファイル](#array-alignment)にワープ（位置合わせ）します（位置合わせ済み）。rawデータはワープされませんが、メタデータに変換情報が含まれます。配列にプロファイルがない場合は、位置合わせなしにフォールバックします（結果の`alignment`に警告が表示されます） 。
- `render_index=False` — カメラごとの植生インデックスのオーバーレイをスキップします。デフォルトでは、設定された場所にレンダリングされます。
- `force_daq=True` — 選択されたレベルで必要とされない場合でも、割り当てられたDAQ/DLS測定値を`.daq`サイドカーとして保存します。

**TIFF圧縮（HTTP 専用ノブ）：**`ArraySession.capture()` は `compression` キーを送信しないため、バックエンドのデフォルト設定が適用されます — `POST /api/camera/array/capture`は`compression`ボディパラメータを読み込み、デフォルトでは`"deflate"`（ロスレスzlib L1 + 水平予測、 フル解像度フレームあたり約4.1 MB）。`"none"`は非圧縮（約6.3 MB/フレーム）で書き込み、**書き込み速度は約5倍高速**です。— どちらもロスレスであり、インポート時の読み取り結果は同一です。SDKはこの機能に対するkwargを公開していません。 回避策として、`chloros-cli lattice array-capture --compression none` または生のHTTPを使用します。また、DEFLATEはPythonのGILを保持するため、圧縮書き込みはカメラごとの書き込みスレッド間で並列化されません。センサーレートでの持続的な8台のカメラによるフル解像度キャプチャをセンサーレートで持続させるには、`compression: "none"`が必要です。詳細：[CLI リファレンス → array-capture](cli-reference.md)。**メンバーごとのエクスポートオーバーライド（HTTP 限定）：**同じエンドポイントは `exclude_serials`（リスト — 保存されたセットからメンバーを削除；配列は引き続き 1 つの同期グループとしてトリガーされ、除外されたメンバーは `excluded` で返される）や、`serial_levels`（`{serial: [level tokens]}`：カメラごとのレベルオーバーライド）、および `serial_index`（`{serial: bool}`：カメラごとのインデックスオーバーレイオーバーライド）。これらはGUIと整合性のある本体パラメータであり、**現時点ではSDKのkwargsではありません**。マップに存在しないメンバーは、配列全体の`levels` / `render_index`にフォールバックします。

##### スキップされたカムの検査 — `CaptureResult.skipped`

`ArraySession.capture()`は`CaptureResult`を返します。これは`list`のサブクラスです：これを反復処理したり、インデックス操作を行ったり、`len()`を適用したりしても — 既存のパターンはすべて引き続き機能します。新しいコードでは、`.skipped`属性を検査することで、どのカムが除外されたか、およびその理由を確認できます。 最も一般的なケースは、RGBの混合フィルター配列内のカメラで、`processing="radiance"`または`"reflectance"`を要求した場合です。ブロードバンドセンサーではベイヤーごとの放射輝度は意味をなさないため、バックエンドは意味のないデータを生成するよりも、それらのカメラをスキップします。

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

理由トークンは `<level>-not-applicable-to-rgb-cam` というパターンに従います（スキップされたレベルごとに 1 つのエントリがあり、それぞれが `level` を持ちます）。反射率に起因するスキップは、`reflectance-skipped-no-fresh-dls`（新しい下向き測定値が利用できない）、 `reflectance-skipped-bound-daq-unavailable (…)`（バインドされたDAQに到達できなかった）、および `dls-uncalibrated-band-<nm>` — バンドの大部分がDAQ光センサーの放射測定校正範囲 （～374–974 nm）の範囲外にあるため、DAQ に基づく絶対反射率の分割は拒否され、フレームはセンサー応答レベルへと大幅に格下げされます。出荷中の SKU の中では F988 のみがこの現象を引き起こします。このカメラでサポートされているパスは、反射率パネルワークフローです。

`processing` レベル：

| レベル | 出力 |
| --- | --- |
| `"raw"` | センサーから直接出力されるシングルチャンネル・ベイヤー（モノクロカメラ：単一バンド） |
| `"debayered"` *（SDKのデフォルト）* | バイリニア・デモザイクによる3チャンネルBGR（モノクロカメラ：1チャンネルのグレースケール）。 |
| `"radiance"` | フル放射測定チェーンを経たfloat32 W/m²/sr/nm（完全な放射測定チェーン経由）。マルチスペクトルのみ — RGB カメラはスキップされます。 |
| `"reflectance"` | uint16 0..32768（Pix4D対応）；絶対基準値を得るには、ライブDAQとのペアリングが必要です。 マルチスペクトルのみ。 |
| `"display"` | GUIプレビューと一致するフルチェーン（カメラのプロファイルに基づくCCM + WB + ガンマ）。 |
| `"all"` | 各カメラについて**該当するレベルごとに1ファイル**（GUIの 「Capture All」／CLIのデフォルト設定に準拠）。返される`CaptureResult`には、`(cam, level)`ごとに1つのフレーム辞書が含まれ、各辞書には該当するレベルが格納されます。該当しないレベルは`.skipped`に表示されます。 各反射率フレームに使用されたDAQ測定値は、`.daq`サイドカーとして保存されます。 |

> **注 — デフォルト値はCLIとは異なります。** `ArraySession.capture()`のデフォルト値は`processing="debayered"`、`chloros-cli lattice array-capture`コマンドのデフォルト値は`processing="all"`です。`processing="all"`をSDKから明示的に渡すことで、CLI／GUIのマルチレベル保存に対応できます。

### キャプチャモードとレコーダー

アレイ表面は、GUIのキャプチャパネルを反映しています。シングル／連続／インターバル／最速シャッターモードに加え、2つのレコーダー（ライブコンポジットビデオおよび生バースト→オフライン再処理）が利用可能です。

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**は、SDKの連続／インターバルループです。スクリプトからこれを中断するための `Ctrl+C` が存在しないため、**必ず** `count` および／または `duration_s` を指定する必要があります（いずれかに達すると停止します）。`interval_s` は各パス開始時点から計測されます （GUIと一致）から計測されます。残りのkwargsはそのまま`capture()`に渡されます。
- **`record`**は*モニタリンググレード*です： 表示されている通りのライブ複合インデックス合成画像をキャプチャするため、フレームを取り込むには複合ストリームが開いている必要があります。配列ごとに1つの合成レコーダー（すでに実行中の場合は例外を発生させます）。
- **`burst` → `build_video`** は *分析用* です： `burst`は、生フレーム＋フレームごとのマニフェスト＋`.daq`（`<output>/bursts/<base>/`の下で個別のDLS読み取りごとに1つ）を、グラブループのフルレートで書き込みます （チェーンなし、exiftoolなし、ライブビューなし）。`build_video`は各フレームを最も近い`.daq`と時刻を照合し、インポートパイプラインの放射度／反射率／インデックス・チェーンを再実行します。`products`は、`{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`（デフォルト：統合インデックス）のリストです。`burst().stop()`はまた、ベストエフォート方式の統合インデックス構築を自動で実行し、その結果は停止結果として`build_job`として返されます。

#### `RecorderHandle`

`ArraySession.record()` および `ArraySession.burst()` によって返されます。スコープの終了時に自動的に停止させるコンテキストマネージャーとして使用するか、手動で操作してください。

| メンバー | 説明 |
| --- | --- |
| `job_id` | バックエンドジョブ ID (文字列)。 |
| `kind` | `"composite"` （`record` から）または `"raw"` （`burst` から）。 |
| `start_stats` | `start`の呼び出しによって返される辞書。 |
| `result` | 実行中の `None`；停止後の最終的な停止結果ディクショナリ。 |
| `stats(timeout=10.0)` | ジョブのリアルタイム統計（書き込まれたフレーム数、実現された fps、経過時間経過時間）。 |
| `stop(timeout=60.0)` | レコーダーを停止します。最終結果を返してキャッシュします。冪等です（2回目の呼び出しではキャッシュされた結果が返されます）。 |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### すでに接続済みのアレイへの接続 — `attach_array`

アレイがすでに起動している場合（GUIによって開かれたか、以前のSDKセッションで`connect_array`が呼び出された場合）、再接続する代わりに`attach_array`を使用してそのハンドルを取得してください。<sn><id>この状況では、</id></sn>`connect_array` は常に「Camera is<sn> already in array<id>」</id></sn>というエラーを<sn><id>返します。これは、メンバーは冪等ではないため、その状況では常に「Camera  is already in array」というエラーが発生します。`attach_array`は`/api/camera/array/list`を読み取り、array_idまたはserialsのいずれかで照合します。

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

パターン：SDK デスクトップ GUI と共用するスクリプトは、まず `attach_array` を試み、プールにまだ配列がない場合は `connect_array` にフォールバックする必要があります。

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **重要 — context-manager の終了は接続を切断します。**`ArraySession.disconnect()`は常に`/array/disconnect`をPOSTします。`CameraSession` / `DAQSensorSession` のような「アタッチ済みだが所有されていない」というガードはありません。GUI と共テナント構成で、スコープ終了時にアレイを破棄したくない場合は、**`with` ブロックを使用しないでください** — ハンドルを通常の変数に格納し、明示的な `disconnect()` は省略してください：
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### ネットワーク分析ヘルパー

配列を開く前に便利 — 提案された設定が収まるかどうかを予測します：

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` は、`ok` / `auto_capped_fps` のいずれかです / `auto_shrunk` / `needs_force_slip`（それ以外の場合は `error`）のいずれかです。`auto_capped_fps` は、要求された解像度が、上限値のあるトリガーレートでのみ RX リングに適合することを意味します。この場合、解像度を維持し、`target_fps=result["recommended"]["recommended_target_fps"]` を `connect_array` （[例 6](#6-capability-probe-before-connecting-a-4-cam-array) を参照）。

**投影値の読み方**（GUI の「アレイ設定」パネルと同じモデル）：

- **バースト (`frame_bytes_total`) は、各カメラの実際のピクセルフォーマットでカメラごとに合計されます。**モノクロ**M3M** カメラは、渡す `pixel_format` に関係なく Mono12 (2 B/px) をストリーミングするため、4-カメラのフル解像度フレームは、3台のモノクロカメラの場合、**約25 MB**となり、すべて8ビットという仮定から算出される約12.6 MBとは異なります。バックエンドは、各カメラのモデルからそのフォーマットを判別します。
- **アドミタンス（`burst_fits_nic_ring`）はドレインを意識した動作**であり、バースト全体対リングの比較ではありません： ホストがRXリングを、カムがリングを埋める速度よりも速く空にする場合、sim-emitが適用されます。10Gホストと1 GbEカムの場合、バーストがリング容量を超過していてもフル解像度を**許容**します。一方、1 GbEホストの場合はブロックされます（`needs_force_slip` / `auto_shrunk`）。
- **`achievable_fps_max`は、保守的なシリアル・リトリーブの最大値**である — `max(readout+emit, N×emit)`では、カメラごとの送信量が1 GbEカメラリンクに制限され、露光に依存しない。 例：4 台のカメラによるフル解像度 12 ビットアレイの場合、約 2.8 fps（ランタイムで測定された約 2.7～3.0 と一致）。完全なモデル：[CLI 参考 → アレイの fps およびバーストモデル](cli-reference.md#array-fps--burst-model)。
- **オーバーサブスクリプション（`oversubscribed: true`）とは、カメラごとの下限値 N × が、衝突防止の上限値を超えることを意味する** — fps フィールド（`achievable_fps_max` / `fps_bright` / `fps_dark`) は 0 となり、自動縮小／ビニングでは解決できません（これらはフレームあたりのバイト数を減らすだけで、1秒あたりのペース配分されたバイト数を減らすわけではないため）。対策としては、カメラ数の削減、ジャンボフレームの使用、またはより高速なNICの採用があります； `max_cams_collision_safe`は上限値（1 GbE @ 1500 MTUの場合、フル解像度カメラ6台、ジャンボフレーム使用時は9台）を報告します。この応答には、`aggregate_demand_bps`、`collision_safe_ceiling_bps`、および`per_cam_floor_bps`（8 MB/s）も含まれます。 [オーバーサブスクリプション](#over-subscription-the-per-cam-floor)を参照してください。

### 検出と一覧表示

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICEアレイは接続されるとすぐにバックグラウンドで連続AEを実行しますが、新たに撮影対象を合わせたシーンでは、収束するまで少し時間がかかります。 **Smart-Capture** は、この処理をパッケージ化した便利な機能です。各カメラの露出値をポーリングし、ウィンドウ全体でアレイの状態が安定するまで待機してから、キャプチャをトリガーします。これは GUI 上の操作と同等であり、 デスクトップアプリの「スマート」キャプチャボタンも、同じバックエンドエンドポイントを呼び出します。

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

`ChlorosProject`（次のセクション）経由で操作する場合、さらに多くの設定項目が利用可能になります：

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

スマートAEポリシーはデフォルトで保守的です。厳密な放射測定作業を行う場合は`exposure_tolerance_pct`を厳しく設定し、変化の激しいシーンで「おおよそ正確」な値が求められている場合は緩く設定してください。

---

## DAQセンサーセッション

分光センサー用の永続的なバックエンドプール（USB経由のDAQ-U、BLE経由のDAQ-M、イーサネット経由のDAQ-E）。カメラの動作を反映しています：スマート検出、プールの再利用、冪等なアタッチ。

### スマート検出（ゼロコンフィグ）

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

優先順位：イーサネット → BLE → USB。明示的なヒントを1つ指定することで、転送方式を固定できます。

### 固定された転送方式

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession` メソッド

| メソッド | 説明 |
| --- | --- |
| `status(timeout=10.0)` | プールエントリの概要（ストリーミング／記録状態、 波長範囲、キャリブレーションSHA、積分時間、frame_avg、AE状態）。 |
| `latest(n=1, timeout=10.0)` | 直近のスペクトルフレームを最大N個返す。 |
| `stream_start()` / `stream_stop()` | ストリーミングの再開／一時停止 （ハンドルは開いたまま）。 |
| `record_start(output_dir=None, device_name=None)` | .daqファイルの記録を開始する。ファイルパスを返す。AWSキャリブレーションバンドルのないDAQ-U/Mでは拒否される （DAQ-Eは例外）。 |
| `record_stop()` | 記録を停止する。`{path, rows}`を返す。 |
| `disconnect()` | プールから解放する。 所有していない接続済みハンドルに対してはノーオペ。 |

> **キャップ補正プロファイル（`cap_id`）は、SDKの調整項目ではありません。** `connect_daq_sensor()` / `DAQSensorSession` メソッドでは、`cap_id` パラメータ や`set_cap`メソッドは公開されていません。艦隊のキャップ補正プロファイルは、CLI（`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`）またはバックエンドの `/api/daq` HTTP ルート（`/api/daq/connect` および `/api/daq/<id>/cap-id` は `cap_id`を受け付けます）。

### ディスカバリー — 接続先のアドレスを検索する

`discover_daq_sensors()`は、USB / BLE / ETH上で、*開くことのできる*センサーをスキャンします。 これは `discover_lattice_cameras()` に対応する DAQ 版であり、**DAQ-M の BLE MAC** を入手する唯一の手段です。DAQ-Eにはホスト名があり、DAQ-UにはCOMポートがありますが、MACアドレスはデバイス本体に表示されておらず、OSにも一覧表示されません。

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| フィールド | 説明 |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COMポート / BLE MAC / ホスト名 — `connect_daq_sensor` へ、`port=` / `mac=` / `eth_host=` として渡す。 |
| `display` | 人間が読み取れるラベル。 |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`、または `None`（スキャンで識別できないポート用。USBシリアルアダプタはプローブなしでは区別できないため、 そのため、不明なものは非表示にせず表示されます）。 |
| `extra` | トランスポートごとの詳細（BLEのアドバタイズ名、USBメーカー、DAQ-EのIP/ファームウェアなど）。空の値は省略されます。 |

| パラメータ | デフォルト | 説明 |
| --- | --- | --- |
| `transports` | 3つすべて | シーケンス（またはCSV文字列） 。必要な条件が分かっている場合は指定するとよい — BLEは処理に時間がかかる。 |
| `scan_timeout` | 5 | トランスポートごとのスキャンウィンドウ（秒単位）。バックエンドにより1～20に制限される。 |
| `timeout` | 60.0 | 呼び出し全体に対するHTTPの上限（SDKの他の箇所と同様）。 |
| `auto_start_backend` | `True` | ローカルバックエンドが ローカルバックエンドが実行されていない場合、ローカルバックエンドを起動します。リモートの`backend_url`に対しては決して起動しません。 |

> **プール内で既に開かれているセンサーは表示されません。** 接続済みのBLE周辺機器はアドバタイズを停止し、開いているCOMポートはプローブできないため、ディスカバリーでは *接続可能な* デバイスが一覧表示されます。 何かを接続した直後に結果が空になるのは想定内です。すでに保持しているものについては、`list_daq_sensors()`を使用してください。スキャンを実行できないトランスポート（bleak / zeroconfがインストールされていない場合）は、例外を発生させるのではなくスキップされるため、BluetoothのないマシンでもUSBおよびETHの応答を取得できます。

### リスト

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### GUI／CLIとの共存

GUIで既にセンサーが開かれている場合、 Pythonから`connect_daq_sensor(port="COM3")`を呼び出すと、`already_connected=True`というマークが付いたハンドルが返されます。その結果、セッションの`disconnect()`はノーオペレーションとなるため、SDKスクリプトがスコープの 終了時に、GUIの下からセンサーが引き剥がされることはありません。

### ダイレクト・ハードウェア・クラス（バックエンドなし）

`daq_sdk`は`chloros_sdk`によって再エクスポートされるため、バックエンドなしでプロセス内でセンサーをエンドツーエンドで駆動することも可能です：

> **利用可能性：**`daq_sdk`はChlorosのデスクトップインストール版に同梱されていますが、PyPIパッケージには**同梱されていません** — `pip install chloros-sdk` を使用すると `lattice_sdk` が利用可能になりますが、`chloros_sdk.DAQ_AVAILABLE == False` は利用できません。 これらのクラスを使用する前に、このフラグを確認してください。pipのみが利用可能なホストドライブでは、代わりに[`connect_daq_sensor()`](#daq-sensor-sessions)を経由してセンサーを使用してください。これにはローカルのトランスポートライブラリは必要ありません。

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

GUI と所有権を共有したい場合は、スマート接続パス (`connect_daq_sensor`) を優先してください。センサーを排他的に所有するヘッドレススクリプトには、ダイレクトクラスを使用してください。

---

## プロジェクトの自動化 — `ChlorosProject`

保存されたChlorosプロジェクトは、`cameras.json` + `sensors.json` + `project.json`を含むフォルダです。`open_project`はマニフェストを読み込み、`connect_all`は、保存された設定（GUIで生成されるのと同じハードウェア状態）で、保存されたすべてのデバイスをオンラインにします。

### 最小限の例

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

または、コンテキストマネージャーとして：

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject` のメソッド

| メソッド | 説明 |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | 保存されたすべてのデバイスを検出・接続します。クラスごとの接続レポートを返します。`127.0.0.1:5000`でリスニングしているバックエンドがある場合はそれを使用し、ない場合は黙って直接接続（バックエンドなし）`lattice_sdk`デバイス制御に無音でフォールバックします。バックエンドは決して生成しません。 |
| `disconnect_all()` | すべてを切断します。 |
| `capture_all(output_dir=".")` | すべてのカメラから1フレームずつ、およびすべてのセンサーからスペクトルデータを取得します。 |
| `stream(camera, overlays=False, fps=10.0)` | 指定されたカメラ（またはアレイ）からBGR形式の`numpy`フレームを生成するジェネレータ。`overlays=False`は、ダイレクトな`lattice_sdk`グラブループの直接的な実装です（アレイは `{serial: frame}` 辞書を出力します）。`overlays=True`は、`ChlorosLocal.camera_stream()`を経由して、バックエンドの`/api/camera/<serial>/stream-annotated` MJPEGフィードへルーティングされ、カメラの保存された`ui.overlay`ブロックがクエリパラメータとして渡される。バックエンドモードと**スタンドアロンカメラ**が必要：ダイレクトモードのカメラは`RuntimeError`を発生させる（バックエンドはこのプロセスが所有するカメラを取得できない）し、配列は`NotImplementedError`を発生させる （カメラごとに合成オーバーレイ — メンバーを名前でストリーミング）が発生します。ワンショット相当：`CameraHandle.capture(annotated=True)`。 |
| `align_arrays(align=True, verbose=False)` | 現在接続されているすべての配列に対してアライメントを実行します。 |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | プロジェクトの画像に対してキャリブレーション／インデックス処理パイプラインを実行 （`ChlorosLocal.process`をラップする。これら4つが**唯一**受け入れられるkwargsである — `indices=`などは`TypeError`を発生させる。`ChlorosLocal.configure()` 経由でインデックスを設定します。`ChlorosLocal()` を遅延構築し、これによりバックエンドが自動起動します。 |

属性:
- `proj.cameras` — `Dict[str, CameraHandle]` は、名前およびシリアル番号をキーとする。
- `proj.arrays` — `Dict[str, ArrayHandle]`：名前とarray_idをキーとする。
- `proj.sensors` — `Dict[str, SensorHandle]`：名前とslot_idをキーとする。
- `proj.config` — `project.json["config"]` 辞書。

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**処理レベル。** `capture()`、`grab()`、および`frame_stream()`はすべて同じ`processing`
トークンを取り、チェーンは累積的です。各レベルは その上のすべての処理を実行します：

| レベル | 出力 | 備考 |
| --- | --- | --- |
| `raw` | 1チャンネル・ベイヤー、センサーネイティブ | デモザイク処理なし。このレベルではオーバーレイは利用できません。 |
| `debayered` | 3チャンネル BGR (**デフォルト**) | 双線形デモザイク。バックエンドモードなしで動作する唯一のレベル。 |
| `radiance` | float32, W/m²/sr/nm | 完全な放射測定チェーン：デモザイク + 3×3 アンミックス (マルチスペクトル) + DSNU + フラットフィールド + NISTスケール。露光量 × ゲインが除算されており、値は絶対値となる。 |
| `reflectance` | uint16, 32768 = 1.0 | 放射輝度を下向きの放射照度（ρ = π·L/E）で除した値。DLS/DAQの測定値が必要 — 以下の注を参照。 |
| `display` | 8ビット sRGB風 | GUIと同等のレンダリング：CCM + ホワイトバランス + カメラのアクティブカラープロファイルによるガンマ補正。 |

`debayered` 以外のものはすべてバックエンドモードを必要とする。ダイレクトモードのカメラは
`NotImplementedError` を発生させる。`reflectance` には使用可能な下向き放射量の測定値が必要である — フレームの終了点は
プールされたDAQをカメラのDLSスロットに自動的に引き込みますが、DAQがバインドされていない場合、チェーンは
反射率出力を受け入れず、品質の低い結果を黙って返すのではなく、返されるメタデータに格下げを
明示的に記録します。

> **反射率DNスケール — ハードコーディングしないでください。** LATTICEの反射率では、`32768` = ρ 1.0 を使用し、
> XMP `Chloros:PixelScale=32768`として記録します。Survey3の反射率では、`65535` = ρ 1.0を使用し、
> `Chloros:*`タグは含みません。タグを読み取り、それを除算してください。 uint16 ドメインで定義されているため、
> `32768` は、再スケーリングを行うすべてのフォーマット（16 ビット TIFF、8 ビット PNG /JPG、32 ビット パーセント）においてそのまま維持されます。保存されたデータ型を
> 保存されたデータ型をまず uint16 に戻します（8 ビットからは ×257、float からは ×65535）。唯一の例外：
> 8 ビットソースのキャプチャが 8 ビットの TIFF として書き込まれた場合は、リスケールされず *クリップ* されるため、そのスケールは
> — Chloros では、その場合、`PixelScale` および MicaSense タプルを完全に省略します。LATTICE 反射率ファイルでタグが欠落している場合は、
> デフォルト値としてではなく、「有効なスケールなし」として扱います。

> **EXIF はエクスポート時に引き継がれます。** `process()`は、ソースキャプチャのGPSブロック
> **およびそのExifIFD**をすべてのプロダクトにコピーするため、 そのため、エクスポートデータには `FocalLength`、`FNumber`、
> `ExposureTime`、`ISO`、`DateTimeOriginal`、`ISO`、
> `ExposureTime`、`DateTimeOriginal`、`CameraSerialNumber`といったファイル名に加え、
> 地理参照情報も付加されます。`FocalLength`は、Pix4Dが地上サンプル距離を算出するための元となるデータです。これがなければ
> 再構築は著しく誤った縮尺になってしまいます（ある実測事例では、411 mの敷地が
> が47.8 kmの敷地として処理されてしまった）。コピーは意図的に`-all:all`ではない：IFD0の構造タグが
> LATTICEの出力を破損させるためであり、`ExifImageWidth`/`Height`は、エクスポートされたラスターではなくソースの
> キャプチャを記述しているため除外されています。

キャプチャ段階のサブフラグ（放射測定レベルに適用 — `radiance`、`reflectance`、`display`）：

| フラグ | デフォルト | 意味 |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + フラットフィールド + 3x3 アンミックス + NIST 放射測定スケール。 |
| `apply_white_balance` | `True` | WB LUT。DAQがカメラにバインドされている場合はDLSを考慮。 |
| `apply_index` | `False` | 植生指数の評価。 |
| `index_expression` | `None` | 計算式のオーバーライド。値が入力されている場合、指数が自動的に有効になります。 |
| `annotated` | `False` | GUI装飾（ゼブラ／グリッド／ピーキング）のオーバーレイ。`raw`では利用不可。 |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **戻り値の型は `CapturePathMap` であり、`Dict[str, str]` ではありません。**
> `chloros_sdk.CapturePathMap` は `Dict[str, Union[str, List[str]]]` です：単一レベル
> `processing`は各シリアルに1つのパスを割り当てますが、マルチレベルのもの（`"all"`、または
> 明示的な`levels`リスト）は、その
> カメラ用に保存されたすべての製品の**順序付きリスト**を割り当てます。ライブの複合合成映像は、
> ストリーミングが行われていた場合、シリアル単位ではなく、追加の
> `"combined"`キーの下に配置されます。`str`を前提としたコードは、 ストリーミングされている場合、シリアル番号の下ではなく、追加の
> `"combined"` キーの下に届きます。`str` を前提とするコードは、
> リスト形式では型チェッカーによる警告もなく動作しますが — リスト形式がリリースされた後も、注釈には `Dict[str, str]`
> と記載されていたため、このエイリアスが存在します。フラット形式を使用する場合は、
> 正規化してください：
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### 配列のアライメント

`ArrayHandle` は、アライメントの全範囲を公開します。 プロファイルはデフォルトでセッション限定です。永続化するには、`export_alignment()` を明示的に呼び出してください。

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### 接続時のアライメント

`connect_all(align=...)` は、接続時にすべての配列を自動アライメントできます：

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

指定がない場合は、`project.json["config"]["auto_align_on_connect"]` にフォールバックします。

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## ダイレクトハードウェア（バックエンド不要）

バックエンド（CI、ヘッドレスロボット、組み込み）への依存を完全に排除したい場合は、`lattice_sdk` および `daq_sdk` を直接インポートしてください。これらは両方とも、`chloros_sdk` によって再エクスポートされています。`CAMERA_AVAILABLE` / `DAQ_AVAILABLE` に関する注意点：`lattice_sdk` は PyPI パッケージに含まれていますが に含まれていますが（ただし、Arena SDK ランタイムが必要です）、`daq_sdk`はデスクトップ版にのみ同梱されています。

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### プリセットとトリガー

4つのプリセットのうち3つは**フリーラン**です。カメラは連続して露光を行い、
`capture()`が次のフレームを返します。 `triggered`は例外です。これは、
ライン2のハードウェアエッジをトリガーとしてカメラを待機状態に設定するため、エッジが検出されるまでは何も撮影されません。

| プリセット | トリガー | 使用場面 |
| --- | --- | --- |
| `default` | フリーラン | 一般的な用途 |
| `high_speed` | フリーラン | 8ビット、 60 fps 上限、短時間露光 |
| `high_quality` | フリーラン | 12ビット、fps上限なし — 静止画撮影の標準的な選択肢 |
| `triggered` | **待機状態、ライン2** | カメラはM8シンクロケーブルに接続されており、他の何かによってトリガーされる |

`triggered` （または`trigger_mode="On"`を自分で設定）し、ライン2を駆動するものが何もない場合、すべての`capture()`はタイムアウトします — カメラに待機を指示したため、これは正しい動作です。
SDKでは、これが発生した際の説明が記載されています。
[キャプチャ中の SC_ERR_TIMEOUT](#direct-hardware-backend-free) を参照してください。

> **注 — 接続時の「GVSP probe」／`SC_ERR_TIMEOUT -1011` メッセージはエラーではありません。**&gt; 接続時、SDKは、スループット向上のために**ジャンボフレーム**（9000バイトのGVSPパケット）のネゴシエーションを試みます。ダイレクトなポイントツーポイントのNICリンク（例：リンクローカルの`169.254.x.x`アドレス）では、ネットワークは通常ジャンボフレームを伝送できないため、このネゴシエーションはタイムアウトとなり、次のようなログが出力されます：
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> これは**設計上のフォールバック動作**です： SDKは自動的に標準の1500バイトパケットに戻り、カメラは通常通り接続を続けます（続く`[chunk-enable …]`行は通常の接続シーケンスの一部です）。キャプチャ機能は引き続き動作します。
>
> このプローブをスキップすることもできますが、**これは単なるログ-抑制機能に留まらず、ジャンボフレームを無効化します。** ネットワークの状態がどれほど良好であっても、カメラは最大1500バイトまでの「Don&#x27;t-Fragment」pingにのみ応答するため、pingテストだけではジャンボフレームを検出することはできません。これを検出できるのはこのプローブだけです。これを無効にすると、カメラはどのネットワーク上でも、標準の1500バイトのパケットを、どのネットワークでも永久に送信し続けます：
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> ジャンボフレームを処理できないと*確実に*分かっているネットワークでのみ有効であり、その場合、カメラ1台あたり接続時間を約1秒短縮できます。これは単なる見かけ上の変更ではなく実質的なトレードオフであるため、SDKではこれを使用する際に次のように表示されるようになりました：
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **特別な理由がない限り、設定を変更しないでください。** 有効にしたままにしておくと、接続のたびに実際のネットワーク環境が再測定されます。ジャンボパケットに対応したスイッチに接続すれば、次回の接続時には設定や再起動を一切行わなくても、自動的にジャンボパケットが認識されます。
>
> ジャンボパケットのスループットを*確保したい*場合は、エンドツーエンドでジャンボパケットを有効にしてください （NICのMTUを9000に設定し、かつジャンボパケットを通すスイッチを使用）するか、リンクがジャンボパケットに対応していることが分かっている場合は`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`で固定してください。ただし、恒久的に設定するよりは、コマンドごとに指定する`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …`を推奨します。 固定サイズに設定するとプローブがスキップされ、前段のネットワークへの適応が停止してしまうためです。経路上の**すべての**デバイスがジャンボパケットを通過させられる必要があります — PoEスプリッターやインジェクターも含まれます。これらが、本来ジャンボパケットに対応しているはずの環境でもパケットを転送できない主な原因となります。

> **`SC_ERR_TIMEOUT -1011` が `capture()` / `grab*()` の最中に発生するのは別の問題です — そちらは実際のエラーです。**&gt; 上記の注記は、**connect-time プローブ**によって記録された `-1011` についてのみです。**キャプチャ**から発生した同じエラーは、カメラは正常に接続されたものの、画像を送信していないことを意味します：
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> 決定的な手がかりは、*制御*チャネルが正常であるカメラ（検出が機能し、設定および `[chunk-enable …]` への書き込みがすべて成功している）でありながら、*すべての*フレームでタイムアウトが発生していることです。
>
> **通常の原因は、カメラがハードウェアトリガー用に武装状態になっていることです。** `trigger_mode="On"`および`trigger_source="Line2"`の場合、 カメラは、M8同期ケーブルに電気的なエッジが到着するまで、一切の信号を出力しません。そのラインを駆動するケーブルがない場合、すべてのデータ取得は永遠に待機したままになります。カメラは故障しておらず、ネットワークも正常に機能しています — 指示された通りに動作しているのです。
>
> `CameraSettings()`および`default` / `high_speed` / `high_quality`プリセットはフリーランとなり、アーム中にタイムアウトしたグラブは、単に「`-1011`」と表示されるのではなく、その理由が説明されます。`PRESETS["triggered"]`は、設計上、Line2をアームします。
>
> 任意のカメラをフリーラン状態にするには：
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> `trigger_mode="Off"` を使用してもタイムアウトする場合は、カメラが実際にデータを送信していない可能性があります。ログと `ip link show` をお送りください。

#### カラープロファイル （RGB ライブプレビュー） — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` は、RGBカメラの**ライブプレビュー**用の表示カラープロファイルを選択します（マルチスペクトルカメラはこの設定を無視します）：

| プロファイル | 意味 |
| --- | --- |
| `raw` | 放射測定チェーンを完全にバイパスします。 |
| `linear` | DSNU + フラット + WB、CCMなし、ガンマなし。 |
| `natural` | リニア処理 + 測定済みCCM + sRGBガンマ。安価な仕上げ（彩度スムージング + ハイライトの彩度低下）のみ適用 — リアルなデフォルト設定。 |
| `enhanced` | `natural` に、フルハブパリティ仕上げ（デフリンジ、ヴィブランス、CLAHE ローカルコントラスト）を追加。より豊かな画質ですが、**1フレームあたりの処理コストが約2倍**となるため、LIVEのフレームレートは低下します。 |
| `custom_temp` | `natural` と同様ですが、ホワイトバランスが `custom_cct_k` ケルビンに固定（DLSは無視され、バックエンド側で2000～10000 Kにクリップされる）。 |

このプロファイルは**ライブプレビュー専用**の速度／画質調整機能です。保存されたキャプチャは、選択されたプロファイルに関係なく常に完全で豊かな仕上げが適用されるため、フレーム時間を確保するために`natural`を選択しても、ディスクに書き込まれるデータの品質が低下することはありません。 未知のプロファイルが設定されると `ValueError` が上昇します。chloros バックエンドに接続可能な場合、その変更はバックエンドにも POST されるため、次のプレビューフレームに反映されます（バックエンドを持たない direct-SDK ユーザーでも、設定の変更は反映されます）。

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### モノクロ（M3M）カメラと `Calibration`

モノクロ **M3M** カメラ（`M3M-<lens>-F<wavelength>`）はシングルバンドです。グレースケールプレーンが 1 つで、ベイヤーモザイクはなく、 3×3のスペクトルクロストーク行列もありません。`Calibration`はこれを認識し、`is_mono`フラグを公開します。反射率は依然としてバンドごとの放射測定マップとして適用されます（アンミックスは単位行列です）が、単一のカメラに対するマルチカメラ1台でのマルチバンド演算は、無意味な結果を返すのではなく、むしろ次のような結果をもたらします：

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

モノクロハードウェアから植生指数を構築するには、異なる波長の複数の M3M カメラをアラインメント済みのマルチバンド・スタックに結合し（[アレイのアラインメント](#array-alignment) を参照）、1 台のカメラではなくそのスタック全体に対して指数を計算します。

DAQ ダイレクトモード:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` が受け付けるキー**— 具体的には `integration_time_ms`、`frame_avg`、 `ae_enabled`、`sunshine_diffuser_installed`（DAQ-E；`cap_id`に置き換えられ、非推奨）、`filter_model` (DAQ-M)、および `cap_id`（すべての DAQ 種別；`None`/`""`/`"none"` = センサー本体のみ、キャップ補正なし）。未知のキーは**黙って無視**されます — 例えば、`{"integration_time": 64}`は何も行いません （`integration_time_ms`である必要があります）。`{"applied": [...], "errors": {...}}`を返し、例外は決して発生しません。

`chloros_sdk`は、上記で使用されたコアサーフェスのみを再エクスポートします。 完全な `daq_sdk` パブリック API（22 個の名前）には以下が追加されています — `daq_sdk` から直接インポートしてください：

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## 例外

「Chlorosで発生したあらゆる問題」を処理するために、基底クラスをキャッチします：

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

> `ChlorosAuthenticationError` および `ChlorosConfigurationError` は、他のクラスと同様にトップレベルでエクスポートされています。また、示されているように `chloros_sdk.exceptions` からもインポート可能です。

階層:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## エンドツーエンドの例

### 1. カスタム進行状況バーを使用してフォルダを処理する

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. ライブ LATTICE アレイ → 反射率 + DAQ リファレンス

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. プロジェクト主導型のキャプチャキャンペーン

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. マルチカメラ・フレームストリーム → NumPyパイプライン

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. ヘッドレス・ダイレクトハードウェア（バックエンドなし）キャプチャスクリプト

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. 4カメラアレイ接続前の機能プローブ

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. キャプチャ・レシピの同等実装（純粋なPython）

CLIのレシピDSLには、Pythonによる直接的な同等実装があります：

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## バックエンドの自動起動

smart-connectのエントリポイント — `connect_camera`、 `connect_array`、`connect_daq_sensor`、および `discover_lattice_cameras` — は、バックエンドが `127.0.0.1:5000`（スマート・接続面のデフォルトURL）でリスニングしていることを前提としています。GUIまたはCLIがすでに実行されている場合、そのいずれかが動作しています。スクリプトのみを実行している場合は、動作していない可能性があるため、これらの関数は**最初の呼び出しの前に、バンドルされたバックエンドバイナリを**自動起動**し（`ChlorosLocal`と同様にウィンドウなし）、`backend_startup_timeout`までその起動を待機する必要があります。

ルール：

- **起動されるのはローカルのURLのみです。** `backend_url`が`localhost` / `127.0.0.1` / `[::1]`を指す`backend_url`のみが対象となります。それ以外のホストは他者のマシンであるとみなされ、決して起動されません。
- **バックエンドは再利用のために実行されたままになります**（CLIと同様）— スクリプトが終了しても暗黙のシャットダウンは行われません。スクリプトを再実行すると、稼働中のバックエンドが再利用されます。
- **これらの呼び出しのいずれかで**`auto_start_backend=False`** を指定してオプトアウトしてください（例：リモートバックエンドを指定した場合、またはバックエンドのライフサイクルを自身で管理する場合）。

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

バンドルされたバイナリが見つからない、または起動できない場合、その後の HTTP 呼び出しでは、単なる接続拒否のトレースではなく、対処可能な、**プラットフォーム依存の** `ChlorosConnectError` が発生します — Windows ではデスクトップアプリまたは `chloros-cli` コマンドが提示され、Linux（GUIなし）では `chloros-cli` コマンドまたは `.deb` が提示されます。

---

## 環境とヘッダー

SDKは、すべてのバックエンドHTTP呼び出しに`X-Chloros-Client: sdk`を付与します。バックエンドでは、GUIの無料プランではなく、SDK / CLIのライセンスルール（ログイン**および**有料のChloros+プランが必要）が適用されます。これはインポート時に自動的に設定されるため、ユーザー側で何かを行う必要はありません。

`http://localhost` および `http://127.0.0.1` はローカルバックエンドとして検出されます。他のホスト（例：自社アナリティクスサービス）への呼び出しは変更されません。

バックエンドURLを、`backend_url=`（または`api_url=` on `ChlorosLocal`）を渡すことで上書きします：

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

（ループバックではない `backend_url` はソース/dev バックエンドにのみ到達します。同梱のバックエンドはループバックにのみバインドされます。トンネルパターンについては「リモートバックエンドモード」を参照してください。）

---

## バージョン管理と互換性

- SDK バージョンは、`chloros_sdk.__version__` として公開されています。
- SDK は、動作をバンドルされたバックエンドのバージョンに固定します。古い SDK と新しいバックエンドを組み合わせても通常は動作しますが（前方互換性のあるエンドポイント）、新しい SDK と古いバックエンドを組み合わせると、`404`エラーが発生する可能性があります。デスクトップアプリを対応するバージョンにアップグレードしてください。
- スマートコネクトのインターフェース （`connect_camera` / `connect_array` / `connect_daq_sensor`）およびネットワーク分析エンドポイントは、安定したJSONスキーマを返します。新しいフィールドは追加される形となります。

---

## トラブルシューティングのヒント

- **`ChlorosAuthenticationError: Login required`** → このマシンで `chloros-cli login EMAIL PASSWORD` を 1 回実行するか、Chloros デスクトップ アプリ経由でサインインしてください。
- **`ChlorosConnectError: No Chloros backend is running …`** → スマートコネクトの呼び出しによりローカルバックエンドが自動起動されるため、このメッセージは、バンドルされたバイナリが見つからない、または起動できない場合（例：デスクトップパッケージのない pip のみのホスト）にのみ表示されます。 このメッセージはプラットフォームに応じて異なります。Windows ではデスクトップアプリを開くか、任意の `chloros-cli` コマンドを実行してください。Linux では、`chloros-cli` コマンドを実行するか（GUI は存在しません）、`.deb` をインストールしてください。 リモートバックエンドの場合は、`backend_url=`（および `auto_start_backend=False`）を指定してください。
- **インポート時に**`CAMERA_AVAILABLE == False`** → `lattice_sdk`の読み込みに失敗しました（通常、Arena SDK ランタイムDLLがインストールされていないことが原因です）。カメラ以外のサーフェスは正常に動作します。
- **Array connect がネイティブ解像度未満を返す**→ バックエンドのスマートプリップ機能により、ワイヤに収まるようフレームサイズが自動的に縮小されます。原因を確認するには `analyze_array_network()` を使用し、その後、リンクをアップグレードするか、 縮小を受け入れるか、`force_tier="slip-emit-and-capture"`を指定して順次キャプチャを行ってください。この縮小によるセーフティネットは、集計上のオーバーサブスクリプション（`oversubscribed: true`、fpsフィールドが0）には**対応していません**。回線に対してカメラ数が多すぎる問題は、ビニング/ROI では修正できません。カメラ数を減らすか、ジャンボフレームを有効にするか、より高速な NIC に移行してください（[オーバーサブスクリプション](#over-subscription-the-per-cam-floor) を参照）。
- **`analyze_array_network()` が、NIC の受信リングが非常に小さい（約 0.26 MB）と報告する / 「FRAMES WILL DROP」と表示される接続ゲート** → ホスト NIC の受信リングはデフォルト値になっています （NIC ドライバの更新後に 32 にリセットされることが多い）状態です。Realtek USB 10GbE アダプタでは、`ReceiveBufferLen=256` および `PendingReceives=64`（優先度高）を設定し、リングを再読み込みするためにバックエンドを再起動してください。詳細な手順：[CLI 参考 → ホストNICの設定と調整](cli-reference.md#host-nic-setup--tuning-lattice-arrays)。
- **再起動/シャットダウン時にホストがハングし、その後 WMI `Invalid class` エラーが発生する／NIC が有効化されない** → 古い USB 10GbE ドライバが原因で `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)。アダプタドライバを最新バージョン（2026以上）に更新し、受信リング設定を再適用してください。詳細は [CLI リファレンス → ホスト NIC のセットアップと調整](cli-reference.md#host-nic-setup--tuning-lattice-arrays)を参照してください。
- **反射率の取得が拒否されました** → 絶対スケールの反射率を取得するには、稼働中の DAQ をカメラ（またはアレイ）にバインドする必要があります。GUI 経由でバインドするか、`processing="radiance"` (W/m²/sr/nm) を使用してください。
- **`smart=True`のキャプチャに予想以上に時間がかかる** → AEの収束はシーンの動的特性に依存します。より高速な（安定性は低い）トリガーが必要な場合は、`exposure_tolerance_pct`を厳格化するか、`stability_window_s`を短縮してください より高速（安定性は低下）なトリガーを希望する場合は、`exposure_tolerance_pct`を厳密化するか、`stability_window_s`の時間を短縮してください。

---

## 関連項目

- [CLI リファレンス](cli-reference.md) — すべてのCLIサブコマンドは、SDK呼び出しに対応しています。
- [DAQ センサーガイド](../daq/README.md) — センサーごとの配線、校正、および記録に関する規則。
- オンラインドキュメント: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
