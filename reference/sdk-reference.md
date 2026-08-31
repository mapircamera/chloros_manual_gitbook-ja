# Chloros Python SDK リファレンス

**バージョン:**

1.2.0**生成日:**2026-07-29 19:19 ·**改訂日:** 2026-08-30**パッケージ:** `chloros-sdk` (PyPI)**対象:** LLMによる利用に最適化されており、人間が読みやすい形式。**範囲:** `import chloros_sdk` によって公開されているすべてのパブリッククラス、関数、およびヘルパー。画像処理、単一カメラ制御、同期配列、DAQ センサー、プロジェクトの自動化を網羅した、コピー＆ペースト可能な例が含まれています。

要点のみを確認したい場合は、以下へ進んでください：
- [インストールとクイックスタート](#installation)
- [LATTICEアレイ向けSmart-Connect](#smart-connect-for-lattice-cameras)
- [DAQセンサーセッション](#daq-sensor-sessions)
- [プロジェクトの自動化](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## 60秒でわかるアーキテクチャ

SDKは、Chlorosバックエンド（デスクトップGUIやCLIでも使用されているのと同じFlaskサーバー）の上に構築された、Pythonの薄いレイヤーです。自動化を行うには、`chloros_sdk`をインポートして高レベルなメソッドを呼び出します。内部的には、すべての呼び出しがポート5000上のローカルバックエンド（`http://127.0.0.1:5000/api/...` — 意図的にHTTP`::1`とは異なる）へのリクエストに変換されます74とは意図的に異なる名称であり、これはWindows上で最初に`::1`に解決され、IPv4のみのバックエンドに対してはリクエスト1回あたり約2秒の処理時間がかかる）。バックエンドはハードウェアプール — カメラ、DAQセンサー、アライメントプロファイル、フレームバッファ — を管理しているため、SDKスクリプトは、シリアルポートやNICの帯域幅を奪い合うことなく、GUIと共存できます。

使用するサーフェスは3つあります：

1. **`ChlorosLocal` + フリー関数** (`process_folder`, `process_lattice_capture`) — 画像処理パイプライン。1回のPython呼び出しで、フォルダ全体のデータをキャリブレーション／デベイヤー／インデックスエクスポート処理を通します。
2. **Smart-connectハンドル** (`connect_camera`、 `connect_array`, `connect_daq_sensor`) — ライブハードウェア用の永続的なバックエンドセッションを開きます。GUI と同じ「スマートプリペップ」フロー：ネットワークプローブ、ティアの自動選択、PTP、AE シード、GPIO トリガー設定。
3. **`ChlorosProject` / `open_project`** — 保存済みのプロジェクト（`cameras.json` + `sensors.json` + `project.json` を含むフォルダ）を読み込み、すべてを一括で接続し、 名前付きハンドルを使用してキャプチャを実行します。

サーフェス 1 および 2 は、まだリスニング中のローカルバックエンドが存在しない場合、**ローカルバックエンドを自動起動**します（GUI や CLI が起動する、同梱のバイナリと同じものです）。そのため、バックエンドを事前に起動しなくても、新しいシェルからスクリプトをそのまま実行できます。 オプトアウトするには、`auto_start_backend=False` を渡してください（例：リモートバックエンドを指定する場合。リモートバックエンドは決して起動されません）。[バックエンドの自動起動](#backend-auto-start)を参照してくださいを参照してください。Surface 3 の動作は異なります：`open_project()` は `auto_start_backend` パラメータを受け付けず、`connect_all()` はバックエンドを起動しません。代わりに `http://127.0.0.1:5000`を一度プローブし、応答がない場合は、何も表示せずに直接（バックエンドフリー）`lattice_sdk`デバイス制御に静かにフォールバックします。`proj.process()`と`stream(..., overlays=True)`のみが、遅延構築により`ChlorosLocal()`（これはauto-start）を遅延生成するのは、`proj.process()`と`stream(..., overlays=True)`のみです。

これら3つすべてが認証ゲート方式を採用しています。マシン上で `chloros-cli login` を一度実行するか、デスクトップ GUI 経由でサインインする必要があります。有効なセッションがない状態で SDK を呼び出すと、`ChlorosAuthenticationError` が発生します。

要件：
- Python 3.7 以上（パッケージの宣言通り；3.10 ベースで開発・3.10を前提に開発）
- ローカルにChloros Desktopがインストールされていること（バックエンドバイナリはインストーラー内に同梱されています）
- 有効なChloros以上のログイン。SDK / CLI の最低プランは**Copper**ティア以上（Copper / Bronze / Silver / Gold）です。 無料の**Iron**ティアでは、SDK / CLIへのアクセス権がありません。これは**サーバー側**で強制されます：SDK / CLIフラグが付いたすべてのリクエストは、有効なセッションと有料プランの両方を保持している必要があります。そうでない場合、バックエンドは`403`を返し、`error_code: PLAN_UPGRADE_REQUIRED` （`ChlorosLocal` によって `ChlorosLicenseError` として、また `connect_*` ヘルパーによって `ChlorosConnectError` として表示されます）。ログアウトしている呼び出し元には `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) が返されます。これら2つは異なる問題です。なぜなら、`chloros-cli login`を再実行すると前者は修正されますが、後者は修正できないからです。
- プランの猶予期間内であれば、オフラインでの使用がサポートされます。ティア情報は、サーバー検証キャッシュ（5分間）または署名済みでマシンに紐付けられたライセンスキャッシュ（月額プランの場合は30日間、年間プランの場合はサブスクリプションの有効期限まで）から読み取られます。 この猶予期間が終了すると、プランは無料プランに切り替わり、SDK / CLI へのアクセスは、マシンがサーバーに一度接続できるまで停止します。`chloros-cli status` (`GET /api/license-status`) は無料プランでもアクセス可能な状態を維持するため、その理由が確認できます。これは、SDK / CLI ルートのみがティア制限の対象外となっているため、原因が明らかです。
- Windows 10/11 64ビット、**Ubuntu 22.04 LTS以降**、またはJetson （JetPack 6）。Ubuntu 20.04は**サポートされていません**：`.deb`の依存関係は、`libc6 (>= 2.34)`を含め、バックエンドがリンクしているものから派生しており、focalにはglibc 2.31が同梱されているためです。

---

## インストール

Python（SDK）は、Chlorosバックエンドの上に構築された、Pythonの薄いレイヤーです。DAQのみのワークフローを数個超えるすべてのケースにおいて、 **ローカルにChlorosデスクトップパッケージ**（WindowsインストーラまたはLinux`.deb`）をインストールする必要があります。これにより、バックエンドバイナリ、LATTICEカメラ用のArena SDKランタイム、およびキャリブレーションバンドルが提供されます。

最新のダウンロード：[`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### ステップ 1 — 「Chloros」プラットフォームパッケージのインストール

#### Windows (.exe)

1. ダウンロードページから `Chloros-Setup-x.y.z.exe` をダウンロードします。
2. インストーラーを実行し、ウィザードの指示に従います。デフォルトのインストールパスは `C:\Program Files\MAPIR\Chloros\` です。
3. Chlorosを少なくとも1回起動し、Chloros+アカウントでサインインしてください。

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

### ステップ 2 — Python SDK のインストール

**Chlorosインストーラには、対応するSDKホイールが同梱されています。** すべてのWindowsインストーラおよびLinux .debパッケージは、GUI / CLI / バックエンドのバージョンと完全に一致する`chloros_sdk-X.Y.Z-py3-none-any.whl`をディスクに配置します。同期を保つためにPyPIを追いかける必要はありません。

#### Windows

インストーラは自動的に- システムのPythonを使用して、バンドルされたwheelファイルに対して`pip install`を自動的に実行します（`py.exe`ランチャーが推奨されますが、利用できない場合は`python -m pip`にフォールバックします）。特別な操作は不要です — インストールが正常に完了すると、`import chloros_sdk`はPython環境で動作します。対象のマシンにPythonが存在しない場合、インストーラはこの手順を黙ってスキップし、GUIおよびCLIは引き続き動作します。

#### Linux (.deb)

この .deb パッケージは、wheel ファイルを `/usr/lib/chloros/sdk/` に配置します。`postinst` は正確なコマンドを出力します。PEP 668 準拠のディストリビューションはデフォルトでグローバルな pip 書き込みを拒否するため、 そのため、自動インストールは行いません：

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

エアギャップ環境の Jetson へのデプロイの場合、これは完全にオフラインで行われます。wheel ファイルはすでにディスク上に存在しています。

#### パブリック PyPI

pip のみのホスト（Chlorosのデスクトップパッケージがインストールされていないもの、リモートバックエンドまたは DAQ 専用のワークフロー）の場合：

```bash
pip install chloros-sdk
```

PyPIはリリースバージョンのインストーラービルドに合わせて更新されるため、公開されているwheelは最新の安定版リリースと一致します。開発版ビルド（例：`1.1.4.dev1`）は、バンドルされたインストーラーwheelを介してのみ提供されます。

#### 確認

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ のサブスクリプションが必要です。** すべての SDK への呼び出しには、有効な Chloros+ ログインが必要です。1 台のマシンにつき 1 回、`chloros-cli login user@example.com 'YourPassword'` を実行してください。認証情報は `~/.chloros/` にキャッシュされます。

### デスクトップパッケージは必要ですか？

ほとんどのワークフローでは、pipパッケージだけでは**不十分**です。各SDKサーフェスに必要なものは以下の通りです：

| SDKサーフェス | デスクトップパッケージが必要？ | 理由 |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`、`process_lattice_capture` | **はい** | `/usr/lib/chloros/chloros-backend` (Linux) または `C:\Program Files\MAPIR\Chloros\…` (Windows) でバックエンドバイナリを自動起動します。 |
| `connect_camera`、`connect_array`、`connect_daq_sensor`、 `analyze_array_network`、`list_*`、`discover_*` | **はい**（ローカル）**/ いいえ**(リモート) | バックエンド経由の純粋なHTTPクライアント。ローカルバックエンド → デスクトップパッケージが必要。リモートバックエンド →**トンネル経由**の`backend_url=` （「リモートバックエンドモード」を参照 — 同梱のバックエンドはループバックのみにバインドされます）。 |
| `ChlorosProject` / `open_project` | **はい** | 保存されたプロジェクトをバックエンド経由で実行します。 |
| 直接 LATTICE クラス（`LatticeCamera`、`CameraPool`、`Calibration`、`DLS`、…） | **はい** | デスクトップパッケージに同梱されている Arena SDK ネイティブランタイムが必要です。そうでない場合、`CAMERA_AVAILABLE` はインポート時に `False` となります。 |
| ダイレクトDAQクラス（`DAQUSensor`、`DAQMSensor`、`DAQESensor`、`SensorFleet`、`discover_all`) | **なし** | pyserial/bleak/zeroconf 上の純粋な Python。pip のみの環境でも、DAQ をエンドツーエンドで駆動できます。 |

### リモートバックエンドモード (pipのみのホスト、トンネル経由)

> **同梱のバックエンドはLAN経由では到達できません。** 本番
> ビルドはループバックのみ（両方のループバックファミリー）にバインドされ、
> 唯一の非ループバックモード（`CHLOROS_CLOUD_MODE`）を
> 完全に拒否するため、 そのため、
> `backend_url="http://<lan-ip>:5000"` **は、インストール済みの
> Chloros** に対しては動作しません — このパターンは、ソース/dev
> バックエンドに対してのみ機能しました。別のマシン上のバックエンドを駆動するには、そのループバック
> ポートを自分で転送し、SDKをトンネルに向けてください：

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

ヘッドレス／CI／ロボティクスホストでは、1台のマシンを 「Chlorosサーバー」としてフルデスクトップ環境をインストールしたマシンを1台用意し、他のすべてのマシンで`pip install chloros-sdk`を使用できます。ただし、それら間の転送には、上記のユーザーが手配したトンネルを使用し、LAN上の直接接続（URL）ではありません。

> **既知の制限事項 — `ChlorosLocal`はpip専用モードには対応していません**です。** `ChlorosLocal(backend_url=BACKEND)`は現在、URLを調査する*前に*、コンストラクタ内でローカルバックエンドバイナリを解決しており、デスクトップパッケージがインストールされていない場合 — アクセス可能なリモートバックエンドが存在する場合でも — `ChlorosBackendError`（「Chlorosバックエンドが見つかりません…」）というエラーを発生させます。これは、リモートバックエンドに接続可能であっても、デスクトップパッケージがインストールされていない場合に発生します。上記のスマートコネクト機能（`connect_camera` / `connect_array` / `connect_daq_sensor`、 および `analyze_array_network` と `list_*` / `discover_*` ヘルパー）のみが、pip のみのホストから動作します。

### DAQ専用ワークフロー（pip専用ホスト）

DAQセンサーのみが必要で、LATTICEカメラや画像処理を一切行わない場合、pipパッケージだけで一通り揃っています：

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

バックエンドも .deb パッケージも不要で、ハードウェアへの直接DAQ作業を行う際に Chloros+ へのログインも必要ありません。

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
| `create_project(project_name, camera=None)` | 新しいプロジェクトを作成します（オプションで `"Survey3N_RGN"` のようなカメラテンプレートを使用可能）。 |
| `import_images(folder_path, recursive=False)` | RAW/TIF/JPG/DNG 画像 **および `.daq` 光センサーの記録** をインポートします。`count`（画像） および `scan_count`（記録データ）を返します。フォルダにどちらも含まれていない場合のみ警告を表示します。 |
| `export_light_sensor(daq=True, csv=True)` | プロジェクト内の各光センサー記録に対して、キャリブレーション済みの `.daq` および `.csv`を`<project>/Light Sensor/`に書き込みます。[光センサー記録](#light-sensor-recordings--calibrated-daq--csv)を参照してください。 |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | 処理ノブを設定します。 |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | パイプラインを実行します。`{"status": "complete", "async": False}`を返し、バックエンドが提供する場合、`summary`キーも返します — 詳細は [実行後の概要とヒント](#post-run-summary--hints)を参照。 |
| `get_config()` / `get_status()` / `status()` | バックエンドの状態を確認します。 |
| `logout()` | キャッシュされた認証情報をクリアします。 |
| `shutdown_backend()` | バックエンドを終了します （SDKで起動された場合）。 |
| `discover_cameras()` | **このインスタンスのバックエンド経由で** LATTICEカメラを検出する（`/api/camera/discover`）。 辞書のリストを返します（`serial`、 `model`, `ip`, …) — GUI/ CLI で表示されるものと同じ形状のリストを返します。見つからない場合やバックエンドに到達できない場合は空のリストを返します。 |
| `camera_capture(output_dir, format="tiff", **settings)` | 単一の フレームを**バックエンド経由で**キャプチャする（このハンドルによって自動起動される）。これにより、GUI/ CLI と同じ準備が行われる（デフォルトは12ビット、プールの再利用、埋め込みキャリブレーションメタデータ）。ターゲットの解決には `serial=` または `device_index=` を使用してください。 `exposure`/`gain`/`pixel_format`/`preset` を `**settings` として処理します。レガシーメタデータ辞書（`filepath`、`width`、`height`、`pixel_format`、`exposure_time`、`gain`、`timestamp`）。 |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | プールされたカメラからオーバーレイ合成されたプレビューフレームを生成 — バックエンドの `/api/camera/<serial>/stream-annotated` ルート経由の軽量 MJPEG クライアント（ゼブラ／グリッド／クロスヘア／ヒストグラム／ ピーキング／スポットはサーバー側で描画）。`decode=True`はBGR配列を出力し、`False`は生のJPEGバイトを出力する。また、プロジェクト単位で`ChlorosProject.stream(overlays=True)`としても利用可能。 |

クリーンアップを保証するためのコンテキスト マネージャーとして使用し、確実なクリーンアップを実現：

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

### 光センサーの記録 — 校正済み `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E は、そのキャリブレーションバンドルが**ない状態でも**記録可能です。これは、
一般公開されている [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
レコーダー (`record_daq.py`) がデフォルトで行っている動作です。これらは生データのセンサーカウントを書き出し、ファイルにタイムスタンプを付与することで、
Chlorosが**シリアル番号** — ローカルキャッシュ
から取得し、次に MAPIR クラウドから取得して、インポート時に適用します。

Chlorosは、結果を1回の記録につき2つの製品として、
`<project>/Light Sensor/`の下に書き出します：

| 製品 | 内容 |
| --- | --- |
| `<name>_calibrated.daq` | 再処理可能なアーカイブ — ライブ記録と同じスキーマですが、これを生成したバンドルが宣言されています。これを再インポートしても、**再度** 再校正されることはありません。 |
| `<name>_calibrated.csv` | センサー独自の波長グリッドにおける分光放射照度（W/m²/nm）。1行につき1つの測定値に加え、測光列（総電力、明所視/暗所視ルクス、PPFDおよびその青/緑/赤の分割、ピーク波長）。 |
| `<name>_raw.daq` / `<name>_raw.csv` | **バンドルのないセンサーのみ（DAQ-A）。**センサーの生スペクトルカウント —**放射照度**ではありません。以下を参照してください。 |

`process()`は、その処理段階の一つとしてこのエクスポートを実行します。画像データは**不要**です：
単独で飛行する光センサーは独立したワークフローであり、そのようなプロジェクトには
その性質上、画像が一切含まれません。

**DAQ-Aの記録データは生カウントとしてエクスポートされます。** DAQ-Aシリーズは、シリアル番号ごとの
バンドルシステムが導入される以前から存在しており、取得すべきバンドルがありません。代わりに、現場で
反射率ターゲットを用いて校正されるため、バンドルは必要とされなかったのです。これらの記録データは
`_calibrated` ではなく `_raw` というファイル名でエクスポートされます。これは、ファイル内のフラグではなく
別のファイル名を使用しているからです。なぜなら、その主張は、単なるファイル名としてメールで転送されても成立していなければならないからです。 `.csv`ヘッダーには`raw spectral sensor counts (NOT irradiance)`と記載されており、
値はファイル**内**でのみ比較可能である（これはまさにターゲットベースのキャリブレーションが
それらを利用する目的であり）、センサー間で比較できるわけではないという警告が表示されます。電力依存の測光カラム（総電力、
明所視／暗所視ルクス、PPFD）は、カウント値から積分されるのではなく、**NULL**として返されます。

バンドルをまったく取得できなかったDAQ-U / DAQ-M / DAQ-Eは、生データを書き込むのではなく、依然として**スキップ**されます。 その場合はバンドルが存在するため、「再接続して再処理する」というアドバイスが有効です。

レガシーな**v1.01 / v1.02**の記録（DAQ-A-SDがこれを書き出します）には、読み取りごとのエポックは含まれておらず、
ファイルの書き込み時刻のみが含まれています。画像↔ダウンウェル・マッチャーは依然としてこれらを拒否します — フレームを
書き込み時刻と照合することは、目に見えない形で誤りを生じさせるため — しかし、エクスポーターはこれらを読み込み、
CSV は `clock=daq_created_on` を出力するため、この製品はどのクロックを使用しているかを明示します。

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
キャリブレーションがないセンサー）は、**理由を明記して** `skipped` として報告されます。このデータは、生カウントを含む「校正済み」ファイルとして
書き出されることは決してありません。インターネットに接続して
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

### 実行後の概要とヒント

完了時、 `process()`は`GET /api/processing-summary`を取得し、その本文を`result["summary"]`として添付します。この取得はベストエフォート方式であり、正常な返却をブロックすることは決してありません。要約が利用できない場合、`process()`は、単純な`{"status": "complete", "async": False}`形式にフォールバックします。`summary["hints"]`の各エントリ（提案された是正措置を含む完全な文章、例：実行結果がゼロ出力となった理由など ——は、Python`UserWarning`として再出力されるため、辞書を検査しなくても、出力が0だった実行は自己診断が可能です：

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]`は、機械可読形式の半分の部分です：

| キー | カウント対象 |
| --- | --- |
| `models` | 実行に含まれるカメラグループ。 |
| `images_in_groups` | それらのグループにまたがるソース画像。 |
| `targets_found` | 検出された反射率ターゲット。 |
| `images_calibrated` | 実行でキャリブレーションされた画像。 |
| `exported_files` | **実行で生成された画像データファイル。** |
| `daq_recordings_exported` / `daq_recordings_skipped` | 光センサーの記録データ。意図的に別々に集計されています。これらは別の段階から得られたものであり、画像データが全くない実行でも存在するため、これらを含めると、DAQのみの実行でもあたかも 画像がエクスポートされたかのように見えてしまうため。 |

これらに加えて：`summary["output_dirs"]`（書き込みが行われたすべてのディレクトリ）、
`summary["light_sensor_export"]`、`summary["stopped"]`（ユーザーが実行を中断した場合に
真となるため、部分的なカウントは、生産量が不足した完了済み実行として読み取られない）、および
`summary["groups"]`（グループごとの内訳）です。

`exported_files`は、パイプラインが**書き込みを行う際**に記録されるものであり、
プロジェクトのイメージオブジェクトから後からスキャンされるわけではありません。並列およびGPU戦略は独自のイメージ
オブジェクトを構築するため（GPUパスではワーカーサブプロセス内で）、以前のスキャンでは
そのような実行ごとに`0 file(s) written`を報告し、その後、すべてが正常に動作した実行に対して-exportsヒントを出力していました — すべてが正常に動作した実行において
です。この数値に基づいてスクリプトを作成している場合、正常な並列実行では現在、
ゼロ以外のカウントが報告されるようになります。

Light-sensorのスキップ報告では、リーダーが各ファイルに対して実際に特定した理由 —
読み取り不能なスキーマ、バンドルの欠落、書き込みエラー — が**重複排除**されるため、1つの原因でスキップされた20個のファイルは、
その原因が20回繰り返されたのではなく、1つの原因として扱われます。

> **実行で画像が生成されなかった場合、`process()`は発生しません。** これは、SDKと
> CLIが意図的に異なる唯一の点です。`chloros-cli process`は「出力が要求されたが、何も
> 書き込まれなかった」場合を失敗とみなして非ゼロで終了するのに対し、SDKは通常通り終了し、その
> 状況を`summary` / hintsを通じて報告します。パイプラインが空の実行時に停止すべき場合は、 ご自身で
> 確認してください — 例外が発生していないことだけに頼るのではなく、`summary` を確認するか（あるいはプロジェクトフォルダ内のファイル数を数えてください）。
> 一般的な原因としては、入力フォルダが
> キャプチャとして認識されなかった場合や、存在するカメラに対して適用不可としてプロダクトがスキップされた場合 （例：RGB専用
> カメラからのラディアンス）などです。

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

### サポートされる値

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

#### 放射測定出力（LATTICE マルチスペクトルパイプライン）

`process` パイプラインの LATTICE マルチスペクトル（M3C/M3M）エクスポートレベル — `reflectance`（デフォルト）、 `radiance`、`sensor-response`、または `all`（画像ごとに適用可能な各モード） — は、プロジェクトの **「放射測定出力」** 処理設定に対応します。`configure()` には専用のキーワードがあります：

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

高度な回避策 — プロジェクトの `"Radiometric output"` キーを `custom_settings` 経由でプロジェクトの `"Radiometric output"` キーを記述する — 依然として機能しますが、設定ブロック全体が置き換えられることに注意してください（以下の警告を参照）：

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance`（デフォルト）は、カメラの放射輝度を、画像とともに検出された**タイムスタンプが一致するDAQダウンウェルリング**（記録された`.daq` (DAQ-U/M/E)**または、画像と共に検出された DAQ-M ネイティブの `.csv`**によって自動的に解決された、**タイムスタンプが一致した DAQ ダウンウェリング**でカメラの放射度を割ります。ローカルに存在しないカメラ単位または DAQ 単位のキャリブレーションバンドルは、初回使用時に**AWS から自動的に取得**されます。CLIは、これを-タイプの製品トグルとして提供されます：`chloros-cli process`: `--radiance`/`--no-radiance`、`--reflectance`/`--no-reflectance`、`--debayered`、 `--preview`。

> `custom_settings`は、計算された設定ブロック全体を**置き換えます**（設計上、`configure()`の他のキーワードや検証をバイパスします）。これを使用する際は、上記の例のように、関心のあるすべての `Project Settings`キーをすべて含めてください。

---

## LATTICEカメラ用Smart-Connect

ライブハードウェア向けの永続的なバックエンドセッション。 GUIが使用するエンドポイントと同じであるため、SDK / CLI / GUI間で動作は同一です。

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
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICamノードを読み込み、`{nodes, errors, enums, device}`を返します。 |
| `set_settings(**kwargs)` | フレンドリー名（`exposure_time`、`gain`、 `pixel_format`、`width`、`height`、`target_brightness`、`ae_damping`、 `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | **1** フレームをキャプチャします。フレームのメタデータ辞書からなる 1 要素のリストを返します。（バースト／マルチフレームキャプチャは削除されました。一連のフレームが必要な場合は、ループ内で `capture()` を呼び出してください。） |
| `disconnect()` | プールから解放します。すでに開かれているセッションにアタッチされている場合は、何もしません。 |

`capture()` エクスポート制御（配列＋GUI と同じモデル）：

- `processing` / `levels` — `processing="all"` は該当するすべてのエクスポートタイプを保存します。`levels=["raw","radiance"]` はそれらのみ保存します（`processing` を上書きします）。 バックエンドのデフォルト設定にする場合は、両方を省略してください。
- `force_daq=True` — 割り当てられた DAQ/DLS 測定値を、生データのみの取得時であっても `.daq` サイドカーとして保存します。これにより、後でフレームを反射率/屈折率データに再処理することが可能になります。DAQがリンクされていない場合は何もしません。

### 同期アレイ — `ArraySession` (Smart-Prep)

`connect_array`は、**マルチカメラ構成における推奨のエントリポイント** です。内部では、GUI によるスマートプリップの全フローを実行します：

1. **ネットワーク分析** (`/api/camera/array/recommend`) — フレームをドロップすることなく、sim-emit ティアに収まる最大のフレームサイズを検出します。
2. **ティアの自動選択** — 回線が処理可能な場合は `sim-capture-sim-emit`、そうでない場合は `sim-capture-ftd-stagger` または `slip-emit-and-capture` を選択します。
3. **自動縮小**— 回線が要求された解像度を維持できない場合、フレームサイズを自動的に縮小／ビニングを自動的に増加させます。**このセーフティネットは、集計上のオーバーサブスクリプションには対応していません**：回線に対するカメラ数が多すぎる場合、フレームの縮小では解決できません — [オーバーサブスクリプション](#over-subscription-the-per-cam-floor)を参照してください。
4. デフォルトで**PTP有効**— カメラ間のタイムスタンプは、**約1 ms**の精度で1つの共有クロックに同期されます。同時露光はM8ハードウェアトリガーによるものです（**&lt; 100 µs**のモジュール間遅延）、 PTPによるものではありません。PTPは*タイムスタンプ*を同期させるものであり、露光を同期させるものではありません。
5. **カメラごとのピクセル形式の自動選択** — RGBカメラ → `BayerRG8`、マルチスペクトルカメラ → `BayerRG12`。
6. **AEのシード設定** — 各カメラの現在のAE状態をスナップショットとして保存し、接続時に撮影中の露出がリセットされないようにする。
7. **GPIOトリガー設定** — `connect_array`がすべてのカメラ（`TriggerMode=On`、`TriggerSource=Line2`）をアーム状態にするため、マスターからのパルスがM8ケーブルを介してスレーブを駆動します。これはアレイののみのステップです。`LatticeCamera`で1台のカメラのみを開いた場合は、代わりにフリーラン動作になります。

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

`force_tier` 値：
- `"sim-capture-sim-emit"` — 真の同時性（すべてのカメラが同じクロックエッジで発火）。
- `"sim-capture-ftd-stagger"` — 柔軟な時間領域でのスタッガー（カムがわずかにずれたタイミングで送信され、ワイヤ上でパケットが直列化される）。
- `"slip-emit-and-capture"` — カムごとの順次キャプチャ（時間的な同期なし；シミュレーションに適合するフレームサイズがない場合の唯一のオプション）。

`wire_ceiling_mbps` は、**ホストの持続ワイヤ帯域幅**（MB/s）を上書きします — これは、
アレイ全体の割り当てが依存する唯一の数値である。自動検出された
値を使用するには、`None` のままにしておく。アレイが GVSP 破損フレームを報告した場合は、この値を低く設定する。自動値は
NIC が通知するリンクレートから導出されるが、これは USB アダプタ、帯域幅の狭いPCIeレーン、
および負荷の高い共有ファブリックを過大評価してしまうため、この過大評価は、
目に見えるリンク速度の低下ではなく、破損フレームとして現れます。この値はプロジェクトのアレイのキャプチャブロックに永続化されるため、
アレイを再開したり、後で `connect_array` を実行したりすると、他のアレイ設定と同様に復元されます。
[アレイの健全性](#array-health--which-subsystem-is-losing-frames) を参照してください。

#### オーバーサブスクリプション（カメラごとの下限値）

Sim-emit ペーシングでは、各カメラに衝突安全なワイヤ帯域幅の割り当てが行われ、その下限は **カメラあたり 8 MB/s**(`per_cam_floor_bps`)。`N × floor`が衝突防止の上限を超えると、アレイは**ワイヤをオーバーサブスクリプション**状態となります。この場合の障害モードはGVSPパケット損失であり、フレームレートの低下ではありません。また、フレームサイズによる解決策は存在しません。**ビニングやROIはフレームあたりのバイト数を減らすものであり、1秒あたりのペーシングバイト数を減らすものではない**集計チェックが比較する対象はこれです。1 GbEホストにおける実用的なフル解像度の上限：**MTU 1500のカメラ6台、ジャンボフレーム使用時は9台**（分析応答内の`max_cams_collision_safe`が、当該回線の上限値を報告しています）。対策：カメラ台数を減らす、ジャンボフレームをエンドツー、またはより高速なNICの導入。

- `analyze_array_network()`および`/api/camera/array/connect`の応答には、`oversubscribed`、`aggregate_demand_bps`、`collision_safe_ceiling_bps`、`max_cams_collision_safe`、 および `per_cam_floor_bps` を含みます。`oversubscribed` が true の場合、投影処理は **fps フィールドをゼロに設定**します（`achievable_fps_max` / `fps_bright` / `fps_dark`）を**ゼロに設定**し、誤解を招くような「遅いものの動作はしている」というレートを報告することはありません。
- `POST /api/camera/array/connect` は、`pin_resolution` ボディパラメータを受け入れます（**HTTP のみ — SDK の kwarg ではありません**； `connect_array` では公開されない）。ピンニングを行うと、ビニングによるウォークダウンという安全策が外れるため、`pin_resolution` が設定された状態でオーバーサブスクライブされた接続は、すべての是正策を明記したエラーと共に**厳格に拒否** される。ピンニングを行わない場合、接続 はウォークダウンを実行しますが、縮小しても集計値をクリアできないという警告が表示されます。
- ベンチワーク用の回避策：バックエンドの環境変数に `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` を設定すると、拒否を「大きな警告」に格下げできます。これにより、接続は実行されますが、パケット損失を受け入れることになります。

#### アレイの健全性 — どのサブシステムがフレームを損失しているか

`GET /api/camera/array/<array_id>/capability`は、接続されたアレイ上の稼働中の`health`ブロックを
保持しており、**10秒**のローリングウィンドウで再評価されます。これは、フレーム損失を
を、原因を特定しない単一の「不完全」レートとして示すのではなく、
互いに相反する修正を必要とする2つの原因に分類します：

| 項目 | 意味 | 対象サブシステム |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (シリアルごと) | フレームが**到着したが、構造的に不整合だった**— GVSPパケット損失。 |**ネットワーク**：ワイヤ・バジェット、ペーシング、NIC RXリング、MTU |
| `never_arrived_rate_pct`（シリアルごと） | フレームが**まったく到着しなかった**— カメラが起動しなかったか、何も送信されなかった。 |**トリガー／同期**: M8ケーブル、`line=`、`TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 各カメラの最低成功率。 | — |
| `per_cam_rate_pct` | カメラごとの不完全率の合計 （両方の原因を合わせたもの）。 | — |
| `stable_for_seconds` | 各カメラが 0.01 % 未満の状態を維持していた期間。 | — |

`health` と併せて、同じレコードには、割り当て全体が滞留している数値も報告されています：

| フィールド | 意味 |
| --- | --- |
| `wire_ceiling_mbps` | ホストの有効な持続ワイヤ予算（MB/s）。 |
| `wire_ceiling_source` | その数値の出典を （例：`USB-capped 200 MB/s (was theoretical 1062; …)` または `user override 120 MB/s (auto said 200)`） |
| `wire_ceiling_is_user_set` | `true` が `wire_ceiling_mbps=` によって設定されたとき。 |
| `nic_is_usb` | USBイーサネットアダプタの場合は `true`。 |

このエンドポイント用の SDK ラッパーはありません。直接読み取ってください：

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

**読み取り方法：** 0 以外の `gvsp_corrupt_rate_pct` かつ `never_arrived_rate_pct` が 0 の場合、
トリガーと同期ケーブルの状態は完璧であり、損失の100％がネットワーク経路に起因していることを示しています。この場合、
`wire_ceiling_mbps`の値を下げて再接続してください。逆のパターンが見られる場合は、同期ケーブルまたは
トリガーラインに問題があることを示しています。

> **`target_fps`Xは、破損フレームの決定要因ではありません。** GevSCPDのペーシングは
> 接続時に一度だけ設定されるため、トリガーレートを下げてもデューティサイクルは変化しますが、
> 同時送信バーストレートは変化しません。測定上、要求量を5倍削減しても改善は見られませんでしたが、
> ワイヤのシーリングを240 MB/sから200 MB/sに下げることで、同じリグの破損率が10.4 %から
> 0.00 %に低下しました。

> **TRI032Sファームウェアでは、ストリーム途中の自動縮小機能は利用できません。** 稼働中のアレイでは
> これを自動で修正することはできません。接続を解除して再接続し、接続時間ピッカーが
> 新しい上限に基づいて再計画を行うようにしてください。

**USBイーサネットアダプタは、その
銘板の記載にかかわらず、プローブによって**200 MB/s**に制限されます。リンク速度を持続転送速度に変換する効率表は
PCIeに基づいており、USB NICはイーサネットリンク速度を広告しますが、
USBバスとそのドライバによって制限されます。この上限は相対的な割合ではなく絶対値です — USB 1 GbE アダプタは
約 80 MB/s を実現するため、この制限の影響を受けません。

#### `ArraySession` メソッド

| メソッド | 説明 |
| --- | --- |
| `status(timeout=10.0)` | ライブ `{fps, ptp, frame_count, last_error, …}`。 |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | 1つの同期されたキャプチャグループ。`CaptureResult`（フレーム辞書のリスト＋`.skipped`）を返します。エクスポート制御は以下を参照してください。 |
| `capture(..., smart=True)` | **スマートキャプチャ** — すべてのカメラでAEが安定するのを待ってからトリガーします。 |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | 最速キャプチャ：RAWのみ + 割り当てられたDAQ読み取り値 (+ フリーの結合インデックス)。GUIの 「最速キャプチャ」ボタンと同様の動作。 |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | 単発／連続／間隔を1つの境界付きループ内で実行。`list[CaptureResult]`を返す。**終了させるには `count` および／または `duration_s` が必要です**（SDK には Ctrl+C 機能がありません）. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | ライブの複合インデックスビューの録画を動画/GIFとして開始 → `RecorderHandle`。配列ごとに1つの複合レコーダー。 |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | 高fpsのRAWベイヤーバーストを開始 → `RecorderHandle`。`build_video()`を使用してオフライン再処理を行う。 |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | 保存済みのRAWバーストをキャリブレーション済みの動画(s) に変換する。完了するまでブロックし (`wait=True`)、`{outputs, errors, combined}` を返す。 |
| `build_video_status(job_id, timeout=15.0)` | オフラインビルドジョブをポーリングする：`{running, result, error, burst_dir}`。 |
| `disconnect()` | アレイ全体を解放する。 |

`capture()` エクスポート制御（GUI/CLIが使用するのと同じエンドポイント）：

- `processing` / `levels` — `processing="all"` （または `levels=["raw","radiance",…]`）は、カメラごとに適用可能なすべてのエクスポートタイプを保存します。単一の `processing` 値は、そのレベルのみを保存します。
- `aligned=True` — すべてのメンバーの非生データエクスポートを、 配列の [アライメントプロファイル](#array-alignment) に合わせてワープします（コレジスター）。RAWデータはワープされませんが、メタデータに変換情報が含まれます。配列にプロファイルがない場合は、アライメントなしにフォールバックします （結果の `alignment` に警告が表示されます）。
- `render_index=False` — カメラごとの植生インデックスのオーバーレイをスキップします。デフォルトでは、設定された場所にレンダリングされます。
- `force_daq=True` — 選択されたレベルで必要とされない場合でも、割り当てられたDAQ/DLS測定値を`.daq`サイドカーとして保存します。

**TIFF圧縮（HTTP -onlyノブ）：**`ArraySession.capture()`は`compression`キーを送信しないため、バックエンドのデフォルト設定が適用されます — `POST /api/camera/array/capture`は`compression`ボディパラメータを読み取り、デフォルトでは`"deflate"`（ロスレスzlib L1 + 水平予測、フル解像度フレームあたり約4.1 MB）となります。 `"none"`は非圧縮（1フレームあたり約6.3 MB）で書き込みを行い、**書き込み速度は約5倍高速**です。— どちらもロスレスであり、インポート時の読み取り結果は同一です。SDKにはこれに関するkwargが設定されていません。 回避策として、`chloros-cli lattice array-capture --compression none` または生のHTTPを使用します。 DEFLATEもPythonのGILを保持するため、圧縮書き込みはカメラごとの書き込みスレッド間で並列化されません。センサーレートでの8カメラフル解像度キャプチャを継続するには、`compression: "none"`が必要です。詳細： [CLI リファレンス → array-capture](cli-reference.md)。**メンバーごとのエクスポート上書き（HTTPのみ）：**同じエンドポイントは、`exclude_serials`（リスト — 保存されたセットからメンバーを削除；配列は依然として1つの同期グループとしてトリガーされ、除外されたメンバーは`excluded`で返される）、`serial_levels`（`{serial: [level tokens]}` ごとの-camレベルでの上書き）、および `serial_index`（`{serial: bool}` per-cam インデックス・オーバーレイ上書き）も受け付けます。これらはGUIと互換性のある本体パラメータであり、**現時点ではSDKのkwargsではありません**。マップに存在しないメンバーは、配列全体の`levels` / `render_index` にフォールバックします。

##### スキップされたカムの検査 — `CaptureResult.skipped`

`ArraySession.capture()` は `CaptureResult` を返します。これは `list`のサブクラスです。これを反復処理し、 インデックス付けし、`len()`処理を行っても、既存のパターンはすべて正常に動作し続けます。新しいコードでは、`.skipped`属性を検査することで、どのカムが除外されたか、およびその理由を確認できます。 最も一般的なケースは、RGBの混合フィルターアレイ内のカメラで、`processing="radiance"`または`"reflectance"`を要求した場合です。-Bayerラディアンスは広帯域センサーでは意味をなさないため、バックエンドは意味のないデータを生成するよりも、それらのカメラをスキップします。

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

理由トークンは `<level>-not-applicable-to-rgb-cam` というパターンに従います（スキップされたレベルごとに 1 つのエントリがあり、それぞれが `level` を持ちます）。反射率に起因するスキップは `reflectance-skipped-no-fresh-dls` （新しい下向き測定値が利用できない場合）、`reflectance-skipped-bound-daq-unavailable (…)` （バインドされたDAQに到達できなかった）、および `dls-uncalibrated-band-<nm>` — バンドの大部分がDAQの光センサーの放射測定的に校正された範囲（~374–974 nm）外にあるため、 そのため、DAQ に基づく絶対反射率の区分は拒否され、フレームはセンサー応答に基づいた処理へと強制的に降格されます。出荷中の SKU の中では F988 のみがこの現象を引き起こします。このカメラのがサポートするパスは、反射率パネルによるワークフローです。

`processing` レベル：

| レベル | 出力 |
| --- | --- |
| `"raw"` | シングルチャンネル・ベイヤー （モノクロカメラ：単一バンド）をセンサーから直接出力。 |
| `"debayered"` *（SDKのデフォルト）* | 双線形デモザイクによる3チャンネルBGR（モノクロカメラ：1チャンネルグレースケール）。 |
| `"radiance"` | 完全な放射測定チェーンを経由した float32 W/m²/sr/nm。マルチスペクトルのみ — RGB カメラはスキップされます。 |
| `"reflectance"` | uint16 0..32768（Pix4D対応）；絶対基準値を得るには、ライブDAQとのペアリングが必要です。マルチスペクトルのみ。 |
| `"display"` | GUIプレビューと一致するフルチェーン（カメラのプロファイルに基づくCCM + WB + ガンマ）。 |
| `"all"` | 各カメラにつき**適用可能なレベルごとに1ファイル** （GUIの「Capture All」／CLIのデフォルト設定に準拠）。返される`CaptureResult`には、`(cam, level)`ごとに1つのフレームディクショナリが含まれ、各ディクショナリにはレベルが格納されます。適用対象外のレベルは`.skipped`に格納されます。各反射率フレームに使用されたDAQ測定値は、`.daq`サイドカーとして保存されます。 |

> **注 — デフォルト値はCLIとは異なります。** `ArraySession.capture()`のデフォルトは`processing="debayered"`です。 `chloros-cli lattice array-capture`コマンドのデフォルトは`processing="all"`です。CLI/GUIのマルチレベル保存を反映させるには、SDKから`processing="all"`を明示的に渡してください。

### キャプチャモードとレコーダー

アレイ表面は、GUIのキャプチャパネルを反映しています。シングル／連続／インターバル／最速シャッターモードに加え、2つのレコーダー（ライブコンポジットビデオおよび生バースト→オフライン再処理）が用意されています。

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

- **`capture_repeated`**は、SDKの連続/インターバルループです。スクリプトからこれを中断するための`Ctrl+C`が存在しないため、**必ず** `count` および／または `duration_s` を指定する必要があります（いずれかに達すると停止します）。 `interval_s`は、各パス開始時点から計測されます（GUIと同様）。残りのkwargsは、そのまま`capture()`に渡されます。
- **`record`**は*モニタリンググレード*です：表示されている通りのライブ複合インデックス合成画像をキャプチャするため、フレームが受信されるには複合ストリームが開いている必要があります。配列ごとに1つの合成レコーダー（すでに実行中の場合は例外を発生させます）。
- **`burst` → `build_video`** は *分析グレード* です：`burst` は、生フレーム、フレームごとのマニフェスト、および `.daq` 1つ（DLSの読み取りごとに1つ）を、`<output>/bursts/<base>/` 下のグラブループのフルレートで（チェーン処理なし、exiftoolなし、ライブビューなし）。`build_video`は各フレームを最も近い`.daq`と時刻を照合し、インポートパイプラインの放射度／反射率／屈折率チェーンを再実行する。`products`は、`{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`（デフォルト：統合インデックス）のリストである。`burst().stop()`は、ベストエフォート方式による統合インデックスの構築も自動的に開始し、 これは停止結果において `build_job` として返されます。

#### `RecorderHandle`

`ArraySession.record()` および `ArraySession.burst()` によって返されます。スコープの終了時に自動的に停止させるコンテキストマネージャとして使用するか、 または手動で制御するために使用します。

| メンバー | 説明 |
| --- | --- |
| `job_id` | バックエンドジョブID (文字列)。 |
| `kind` | `"composite"`（`record` から）または `"raw"` （`burst` からのもの）。 |
| `start_stats` | `start` の呼び出しによって返される辞書。 |
| `result` | 実行中の `None`；停止後の最終的な 停止結果ディクショナリ。 |
| `stats(timeout=10.0)` | ジョブのリアルタイム統計情報（書き込まれたフレーム数、実測FPS、経過時間）。 |
| `stop(timeout=60.0)` | レコーダーを停止します。最終結果を返してキャッシュします。冪等です（2回目の呼び出しではキャッシュされた結果が返されます）。 |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### すでに接続済みのアレイへの接続 — `attach_array`

アレイがすでに起動している場合（GUIによって開かれたか、以前のSDKセッションで`connect_array`が呼び出された場合）、再接続する代わりに `attach_array` を使用してそのアレイへのハンドルを取得してください。<sn><id>その状況では、</id></sn>`connect_array` は常に「Camera is<sn> already in array<id>」</id></sn>というエラーを返します<sn><id>。 これは、プール内のメンバーに対して `/array/connect` を POST しても冪等ではないためです。`attach_array` は `/api/camera/array/list` を読み取り、array_id または serials のいずれかで照合を行います。

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

パターン：SDKデスクトップGUIと共テナント化するスクリプトは、まず`attach_array`を試み、プールにまだ配列がない場合は`connect_array`にフォールバックするようにしてください。

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **重要 — context-manager の終了時には接続が切断されます。**`ArraySession.disconnect()`は常に`/array/disconnect`をPOSTします。`CameraSession`や`DAQSensorSession`のように、「アタッチ済みだが所有されていない」というガードは存在しません。 GUI と共用しており、スコープ終了時に配列を破棄したくない場合は、**`with` ブロックを使用しないでください** — ハンドルを通常の変数に保持し、明示的な `disconnect()` をスキップしてください:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### ネットワーク解析ヘルパー

配列を開く前に役立つ機能 — 提案された設定が適合するかどうかを予測します：

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

`status`は、`ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip`のいずれかですX （それ以外の場合は `error`）。`auto_capped_fps` は、要求された解像度が、上限トリガーレートでのみ RX リングに適合することを意味します。解像度を維持し、`target_fps=result["recommended"]["recommended_target_fps"]` を `connect_array` に渡す（[例 6](#6-capability-probe-before-connecting-a-4-cam-array) を参照）。

**投影値の読み方**（GUIの「アレイ設定」パネルと同じモデル）：

- **バースト（`frame_bytes_total`）は、各カメラの実際のピクセルフォーマットでカメラごとに合算されます。**モノ**M3M**カメラは、渡された `pixel_format` に関わらず Mono12 (2 B/px) をストリーミングするため、4 台のカメラによるフル解像度のフレームは、3 台のモノクロカメラの場合、**約 25 MB** となり、すべて 8 ビットという仮定から算出される約 12.6 MB とは異なります。 バックエンドは、各カメラのモデルからそのフォーマットを判別します。
- **アドミタンス (`burst_fits_nic_ring`) はドレインを認識する**ものであり、バースト全体対リングという区別ではありません： ホストがRXリングを、カムがそれを満たす速度よりも速く排出する場合、sim-emitが適用されます。10Gホストと1 GbEカムの場合、バーストがリング容量を超えていてもフル解像度を**許容**します； 1 GbEホストの場合はブロックされます（`needs_force_slip` / `auto_shrunk`）。
- **`achievable_fps_max`は、保守的なシリアル取得の上限値**です — `max(readout+emit, N×emit)`は、カメラごとの送信量が1 GbEカメラリンクに制限され、露光時間とは無関係です。例：4台のカメラによるフル解像度12ビットアレイの場合、約2.8 fps（ランタイムで測定された約2.7–3.0）と一致）。完全なモデル：[CLI 参考資料 → アレイのfpsおよびバーストモデル](cli-reference.md#array-fps--burst-model)。
- **オーバーサブスクリプション（`oversubscribed: true`）とは、N × カメラごとの下限値が、衝突安全上限** — fpsフィールド（`achievable_fps_max` / `fps_bright` / `fps_dark`）は0となり、自動縮小／ビニングでは修正できない（これらは1フレームあたりのバイト数を減らすだけで、 1秒あたりのペース調整済みバイト数ではない）。解決策としては、カメラ数の削減、ジャンボフレームの使用、またはより高速なNICの導入がある。`max_cams_collision_safe`は上限値（1 GbE、MTU 1500、 ジャンボフレーム使用時は9台）を示しています。この応答には、`aggregate_demand_bps`、`collision_safe_ceiling_bps`、および`per_cam_floor_bps` （8 MB/s）が含まれます。[オーバーサブスクリプション](#over-subscription-the-per-cam-floor)を参照してください。

### 検出と一覧表示

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## スマートAE / スマートキャプチャ

LATTICEアレイは接続されるとすぐにバックグラウンドで継続的なAEを実行しますが、新たに撮影対象を指定したシーンは収束するまで少し時間がかかります。**スマートキャプチャ** は、この利便性をパッケージ化した機能です。各カメラの露出をポーリングし、ウィンドウ全体でアレイの状態が安定するまで待機した後、キャプチャをトリガーします。これはGUIと同等の機能であり、デスクトップアプリの「スマート」キャプチャボタンも、同じバックエンドエンドポイントを呼び出します。

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

スマートAEポリシーは、デフォルトでは保守的な設定になっています。 厳密な放射測定を行う場合は `exposure_tolerance_pct` を厳しく設定し、変化の激しいシーンで「おおよそ正確」な結果が求められている場合は緩く設定してください。

---

## DAQ センサーセッション

スペクトルセンサー（USB経由のDAQ-U、BLE経由のDAQ-M、イーサネット経由のDAQ-E）用の永続的なバックエンドプール。カメラの表面を反映しています：スマート検出、プールの再利用、冪等なアタッチ。

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

優先順位：イーサネット → BLE → USB。明示的なヒントを1つ指定することで、トランスポートを固定できます。

### トランスポートの固定

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
| `status(timeout=10.0)` | プールエントリの概要（ストリーミング／記録状態、波長 範囲、キャリブレーションSHA、積分時間、frame_avg、AE状態）。 |
| `latest(n=1, timeout=10.0)` | 直近のスペクトルフレームを最大 N 個返す。 |
| `stream_start()` / `stream_stop()` | ストリーミングの再開／一時停止（ハンドルは開いたまま）。 |
| `record_start(output_dir=None, device_name=None)` | .daq ファイルの記録を開始する。ファイルパスを返す。AWS キャリブレーションバンドルのない DAQ-U/M では拒否される（DAQ-E は例外）。 |
| `record_stop()` | 記録を停止する。 `{path, rows}`を返します。 |
| `disconnect()` | プールから解放します。アタッチ済みだが所有されていないハンドルに対してはノーオペレーションです。 |

> **キャップ補正プロファイル (`cap_id`)は、SDKの調整ノブではありません。** `connect_daq_sensor()` / `DAQSensorSession` は、`cap_id` パラメータや `set_cap` メソッドを公開していません。CLI（`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) またはバックエンドの `/api/daq` HTTP ルート（`/api/daq/connect` および `/api/daq/<id>/cap-id` は `cap_id`を受け入れます）。

### ディスカバリー — 接続先のアドレスを検索する

`discover_daq_sensors()`は、USB / BLE / ETH上で*開くことのできる*センサーをスキャンします。これは`discover_lattice_cameras()`に対応するDAQ版であり、**DAQ-MのBLE MAC**を入手する唯一の手段です。DAQ-Eにはホスト名があり、DAQ-UにはCOMポートがありますが、MACアドレスはデバイスに印字されておらず、OSにも表示されません。

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
| `address` | COMポート / BLE MAC / ホスト名 — `connect_daq_sensor` として `port=` に渡す / `mac=` / `eth_host=`。 |
| `display` | 人間が読み取れるラベル。 |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`、 または、スキャンで識別できないポートの場合は `None`（USB シリアルアダプタはプローブなしでは区別できないため、不明なものは非表示にせず表示されます）。 |
| `extra` | 各トランスポートごとの詳細（BLE アドバタイズドネーム、USB メーカー、DAQ-E IP/FW/…）。空の値は省略されます。 |

| パラメータ | デフォルト | 説明 |
| --- | --- | --- |
| `transports` | 3つすべて | スキャンを制限するシーケンス（またはCSV文字列）。目的が明確な場合に指定すると有効です — BLEは処理に時間がかかるため。 |
| `scan_timeout` | 5 | トランスポートごとのスキャン ウィンドウ（秒単位）。バックエンドは1–20に制限します。 |
| `timeout` | 60.0 | 呼び出し全体に対するHTTPの上限（SDKの他の箇所と同様）。 |
| `auto_start_backend` | `True` | ローカルの ローカルバックエンドを起動します。リモートの `backend_url` に対しては決して起動しません。 |

> **プール内で既に開かれているセンサーは表示されません。** 接続済みの BLE 周辺機器はアドバタイズを停止し、開いている COM ポートはプローブできないため、ディスカバリーでは *接続可能な* ものがリストされます。 何かを接続した直後に結果が空になるのは想定内です。すでに保持しているデバイスについては、`list_daq_sensors()` を使用してください。スキャンを実行できないトランスポート（bleak / zeroconf がインストールされていない）は、例外を発生させるのではなくスキップされるため、Bluetooth 未搭載のマシンでも USB および ETH の応答は取得できます。

### リスト

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### GUI／CLIとの共存

GUIですでにセンサーが開かれている場合、 Pythonから`connect_daq_sensor(port="COM3")`を呼び出すと、`already_connected=True`というマークのついたハンドルが返されます。これにより、セッションの`disconnect()`はノーオペレーションとなるため、SDKスクリプトがスコープの終了時にGUIからセンサーを強制的に切り離すことはありません 終了時に、GUIの下からセンサーが引き剥がされることはありません。

### ダイレクト・ハードウェア・クラス（バックエンドなし）

`daq_sdk`は`chloros_sdk`によって再エクスポートされるため、バックエンドを使用せずにプロセス内でセンサーをエンドツーエンドで駆動することも可能です：

> **利用可能性:**`daq_sdk`はChlorosのデスクトップインストール版に同梱されていますが、**PyPIパッケージには含まれていません** — `pip install chloros-sdk`を使用すると`lattice_sdk`が利用可能になりますが、`chloros_sdk.DAQ_AVAILABLE == False`は除外されます。これらのクラスを使用する前にそのフラグを確認してください。pipのみのホスト環境では、代わりに [`connect_daq_sensor()`](#daq-sensor-sessions) を使用してセンサーを制御してください。これにはローカルのトランスポートライブラリは必要ありません。

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

GUI と所有権を共有したい場合は、スマートコネクトパス (`connect_daq_sensor`) を優先してください。センサーを排他的に所有するヘッドレススクリプトには、ダイレクトクラスを使用してください。

---

## プロジェクトの自動化 — `ChlorosProject`

保存されたChlorosプロジェクトは、`cameras.json` + `sensors.json` + `project.json`を含むフォルダです。`open_project`はマニフェストを読み込み、`connect_all`は保存された設定で全ての保存済みデバイスをオンライン状態にします。これは、GUIで操作した場合と同じハードウェア状態となります。

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

### `ChlorosProject` メソッド

| メソッド | 説明 |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | 保存されたすべてのデバイスを検出および接続します。クラスごとの接続レポートを返します。 `127.0.0.1:5000` でリスニングしているバックエンドがある場合はそれを使用し、ない場合は黙って直接（バックエンドなし）の `lattice_sdk` デバイス制御にフォールバックします。バックエンドを起動することはありません。 |
| `disconnect_all()` | すべてを切断します。 |
| `capture_all(output_dir=".")` | すべてのカメラから1フレームずつ、およびすべてのセンサーからスペクトルデータを取得します。 |
| `stream(camera, overlays=False, fps=10.0)` | 指定されたカメラ （またはアレイ）から BGR `numpy` フレームを生成するジェネレータ。`overlays=False` は直接的な `lattice_sdk` グラブループである（アレイは `{serial: frame}` ディクショナリを生成する）。`overlays=True` は `ChlorosLocal.camera_stream()` → バックエンドの `/api/camera/<serial>/stream-annotated` MJPEG フィードを経由し、カメラに保存された `ui.overlay` ブロックがクエリパラメータとして渡されます。バックエンドモードと、**単体のカメラ**が必要です。ダイレクトモードのカメラは `RuntimeError` を発生させます を発生させます（バックエンドはこのプロセスが所有するカメラを取得できません）。また、配列の場合は `NotImplementedError` を発生させます（カメラごとに合成オーバーレイ — メンバーを名前でストリーミング）。ワンショットでの同等操作：`CameraHandle.capture(annotated=True)`。 |
| `align_arrays(align=True, verbose=False)` | 現在接続されているすべての配列に対してアライメントを実行します。 |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | プロジェクトの画像に対してキャリブレーション／インデックス処理パイプラインを実行します（`ChlorosLocal.process`をラップします。これら 4つが**唯一**受け入れられるkwargsである — `indices=`などは`TypeError`を発生させる；インデックスは`ChlorosLocal.configure()`を介して設定する）。`ChlorosLocal()`を遅延生成し、バックエンドを自動起動する。 |

属性:
- `proj.cameras` — `Dict[str, CameraHandle]` は、名前およびシリアル番号をキーとする。
- `proj.arrays` — `Dict[str, ArrayHandle]` は。
- `proj.sensors` — `Dict[str, SensorHandle]` は、名前と slot_id をキーとする。
- `proj.config` — `project.json["config"]` ディクショナリ。

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
トークンを採用しており、チェーンは累積的です。各レベルはそれより上のすべての処理を実行します：

| レベル | 出力 | 備考 |
| --- | --- | --- |
| `raw` | 1チャンネル・ベイヤー、センサーネイティブ | デモザイク処理なし。このレベルではオーバーレイは利用できません。 |
| `debayered` | 3チャンネルBGR（**デフォルト**） | 双線形デモザイク。バックエンドモードなしで動作する唯一のレベル。 |
| `radiance` | float32、W/m²/sr/nm | 完全な放射測定チェーン：デモザイク + 3×3 アンミックス（マルチスペクトル） + DSNU + フラットフィールド + NIST スケール。露出 × ゲインが除算されているため、値は絶対値となる。 |
| `reflectance` | uint16, 32768 = 1.0 | 放射度を下向き放射照度（ρ = π·L/E）で除した値。DLS/DAQの測定値が必要 — 以下の注を参照。 |
| `display` | 8ビット sRGB風 | GUIと同等のレンダリング：カメラのアクティブなカラープロファイルによるCCM＋ホワイトバランス＋ガンマ補正。 |

`debayered` 以外のものはすべてバックエンドモードを必要とする。ダイレクトモードのカメラは
`NotImplementedError`を生成します。`reflectance`には使用可能なダウンウェルリングの測定値が必要です — フレームの終了点は
プールされたDAQをカメラのDLSスロットに自動的に引き込みますが、 しかし、DAQがバインドされていない場合、チェーンは
反射率の出力を拒否し、品質の低い結果を黙って
返すのではなく、返されるメタデータに格下げを明示的に記録します。

> **反射率のDNスケール — ハードコーディングしないでください。** LATTICEの反射率は`32768` = ρ 1.0を使用し、
> XMPに`Chloros:PixelScale=32768`を付与します。Survey3の反射率は`65535` = ρ 1.0 を使用し、
> `Chloros:*` タグは含まれていません。 タグを読み取り、それを除算する。これは uint16 ドメインで定義されているため、
> `32768` として、再スケーリングを行うすべてのフォーマットで維持される （16ビット TIFF、8ビット PNG /JPG、32ビット パーセント）—— まず、
> 保存されたデータ型を uint16 に正規化してください（8ビットからは ×257、float からは ×65535）。唯一の例外：
> 8ビットソースのキャプチャが8ビットのTIFFとして書き込まれた場合は、リスケールされず*クリップ*されるため、それを表すスケールは存在しない
> — Chloros では、その場合、`PixelScale`およびMicaSenseのタプルを完全に省略する。 LATTICE反射率ファイルで
> タグが欠落している場合は、デフォルト値としてではなく、「有効なスケールなし」として扱う。

> **EXIFがエクスポートに引き継がれる。** `process()`は、ソースキャプチャのGPSブロック
> **およびその ExifIFD** をすべての出力データにコピーするため、エクスポートデータには `FocalLength`、`FNumber`、
> `ExposureTime`、`ISO`、`DateTimeOriginal`、および`CameraSerialNumber`に加え、
> 地理参照情報も含まれます。`FocalLength`は、Pix4Dが地上サンプル距離を算出するために使用するデータです。これがないと
> 再構築は著しく誤ったスケールで処理されてしまいます（実測例では、411 mの現場が
> 47.8 kmの現場となってしまいました）。 このコピーは意図的に `-all:all` ではありません。IFD0 の構造タグが
> LATTICE の出力を破損させるためです。また、`ExifImageWidth`/`Height`は、エクスポートされたラスタではなくソースの
> キャプチャを記述しているため除外されています。

キャプチャ段階のサブフラグ（放射レベルに適用 — `radiance`、`reflectance`、`display`）：

| フラグ | デフォルト | 意味 |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + フラット-フィールド + 3x3アンミックス + NIST放射測定スケール。 |
| `apply_white_balance` | `True` | WB LUT。カメラにDAQがバインドされている場合はDLS対応。 |
| `apply_index` | `False` | 植生指数の評価。 |
| `index_expression` | `None` | 計算式のオーバーライド。値が空でない場合 → インデックスが自動有効化。 |
| `annotated` | `False` | GUI 装飾（ゼブラ／グリッド／ピーキング）のオーバーレイ。`raw` では利用不可。 |

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
> `chloros_sdk.CapturePathMap` は `Dict[str, Union[str, List[str]]]` です：単一レベルの
> `processing`は各シリアルに1つのパスを割り当てますが、マルチレベルのもの（`"all"`、または
> 明示的な`levels`リスト）は、その
> カメラ用に保存されたすべての製品の**順序付きリスト** として提供されます。
> カメラごとに保存されたすべての製品の順序付きリストです。ライブの複合合成映像（ストリーミングされている場合）は、シリアル番号の下ではなく、追加の
> `"combined"` キーの下に届き、シリアル番号の下には届きません。`str` を前提としたコードは、
> リスト形式では型チェッカーが警告を出さずにエラーになります — アノテーションには、リスト形式が公開されてからしばらくの間、`Dict[str, str]`
> と記載されていたため、このエイリアスが存在します。フラット形式を使用したい場合は、
> 正規化を行ってください：
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### 配列のアライメント

`ArrayHandle` は、アライメントの全範囲を公開します。プロファイルはデフォルトでセッション限定です — 永続化するには、`export_alignment()`を明示的に呼び出してください。

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

`connect_all(align=...)`は、接続時にすべての配列を自動アライメントできます:

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

## ダイレクトハードウェア（バックエンドなし）

バックエンド （CI、ヘッドレスロボット、組み込み）への依存をゼロにしたい場合は、`lattice_sdk` と `daq_sdk` を直接インポートしてください。これらはいずれも `chloros_sdk` によって再エクスポートされています。`CAMERA_AVAILABLE` に関する注意 / `DAQ_AVAILABLE`: `lattice_sdk`はPyPIパッケージに含まれています（ただし、Arena SDK ランタイムが必要です）。一方、`daq_sdk`はデスクトップ版インストールにのみ同梱されています。

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

4つのプリセットのうち3つは**フリーラン**です：カメラは連続して露光し、
`capture()`が次のフレームを返します。 `triggered`は例外です。これは
ライン2のハードウェアエッジをトリガーとしてカメラを待機状態に設定するため、エッジが検出されるまでは何もキャプチャしません。

| プリセット | トリガー | 使用場面 |
| --- | --- | --- |
| `default` | フリーラン | 一般的な用途 |
| `high_speed` | フリーラン | 8ビット、60 fps 制限、短時間露光 |
| `high_quality` | フリーラン | 12ビット、fps 制限なし — 静止画撮影の一般的な選択肢 |
| `triggered` | **アーム状態、ライン2** | カメラはM8シンクロケーブルに接続されており、別の何かがシャッターを切る |

`triggered` （または手動で`trigger_mode="On"`を設定）し、
ライン2を駆動するものが何もない場合、すべての`capture()`はタイムアウトします — カメラに待機を指示したため、これは正常な動作です。
SDKでは、この現象が発生した際の説明が記載されています； 参照
[キャプチャ中の](#direct-hardware-backend-free)を参照してください。

> **注 — 接続時の「GVSP probe」／`SC_ERR_TIMEOUT -1011`メッセージはエラーではありません。**&gt; 接続時、SDKは、スループット向上のために**ジャンボフレーム**（9000バイトのGVSPパケット）のネゴシエーションを試みます。 ダイレクトなポイント・ツー・ポイントのNICリンク（例：リンクローカルの`169.254.x.x`アドレス）では、通常、ネットワークはジャンボフレームを伝送できないため、 そのため、このプローブはタイムアウトとなり、次のような行がログに記録されます：
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> これは**設計上のフォールバック**です： SDKは自動的に標準の1500バイトパケットに戻り、カメラは通常通り接続を継続します（続く`[chunk-enable …]`の行は、通常の接続シーケンスの一部です）。キャプチャ機能は引き続き動作します。
>
> このプローブはスキップすることもできますが、**これは単にログの出力を抑制するだけのものではなく、ジャンボフレームを無効にするものです。** ネットワークの状態がどれほど良好であっても、カメラは「Don&#x27;t-Fragment」pingに対して最大1500バイトまでしか応答しないため、pingテストだけではジャンボフレームの存在を検出できません。これを検出できるのはこのプローブだけです。これを無効にすると、カメラはどのネットワークにおいても、 どのネットワークでも：
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> ジャンボフレームを*確実に*伝送できないことが分かっているネットワークでのみ有効であり、その場合、カメラ1台あたり接続時間を約1秒短縮できます。これは単なる見かけ上の変更ではなく実質的なトレードオフであるため、SDKではこれを使用する際にその旨が表示されるようになりました：
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **特別な理由がない限り、設定を変更しないでください。** 有効にしたままにすると、接続のたびに実際のネットワーク環境が再測定されます： ジャンボ対応スイッチに接続すれば、次の接続時に自動的にジャンボが有効になり、設定も再起動も不要です。
>
> ジャンボパケットのスループットを*確実に*得たい場合は、エンドツーエンドでジャンボを有効にする（NICのMTUを9000に設定し、ジャンボパケットを通すスイッチを使用する）か、リンクがジャンボパケットに対応していることが分かっている場合は、`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`でサイズを固定してください — ただし、サイズを固定するとプローブがスキップされ、前方のネットワークへの適応が停止してしまうため、恒久的な設定よりもコマンドごとの `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` を優先してください。パス上の**すべての**パス上の**すべての**デバイスがジャンボパケットを転送できる必要があります。これにはPoEスプリッターやインジェクターも含まれます。これらが原因で、本来ジャンボパケットに対応している設定でも転送できないことがよくあります。

> **`SC_ERR_TIMEOUT -1011` が `capture()` / `grab*()` の実行中に発生する場合は別の問題であり、これは実際のエラーです。**&gt; 上記の注記は、**connect-timeプローブ**によって記録された`-1011`についてのみです。**キャプチャ**から発生した同じエラーは、カメラは正常に接続されているものの、画像を送信していないことを意味します：
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> 決定的な手がかりは、*制御*チャネルが正常（検出が機能し、設定や `[chunk-enable …]` の書き込みがすべて成功している）であるにもかかわらず、*すべての*フレームでタイムアウトが発生しているカメラです。
>
> **通常の原因は、カメラがハードウェアトリガー用に武装状態になっていることです。** `trigger_mode="On"` および `trigger_source="Line2"` の場合、M8 同期ケーブルに電気的なエッジが到着するまで、カメラは何も送信しません。そのラインを駆動するケーブルがない場合、すべての取得処理は永遠に待機したままになります。カメラは故障しておらず、ネットワークも 正常です――指示された通りに動作しているのです。
>
> `CameraSettings()` および `default` / `high_speed` / `high_quality` プリセットはフリーランを指定しており、アーム状態でタイムアウトしたグラブは、単に `-1011` と表示されるのではなく、その理由を説明します。`PRESETS["triggered"]`は設計上、Line2をアームします。
>
> 任意のカメラを強制的にフリーラン状態にするには：
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> それでも `trigger_mode="Off"` でタイムアウトする場合は、カメラが実際にデータを送信していないことになります。ログと `ip link show` を送ってください。

#### カラープロファイル（RGBのライブプレビュー） — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)`は、RGBカメラの**ライブプレビュー**用の表示カラープロファイルを選択します（マルチスペックカメラはこの設定を無視します）：

| プロファイル | 意味 |
| --- | --- |
| `raw` | 放射測定チェーンを完全にバイパスします。 |
| `linear` | DSNU + フラット + WB、CCMなし、ガンマなし。 |
| `natural` | リニア + 測定済みCCM + sRGBガンマ、簡易仕上げのみ（彩度平滑化 + ハイライトの彩度低下） — リアルなデフォルト設定。 |
| `enhanced` | `natural`に、フルハブパリティ仕上げ（デフリンジ、ヴィブランス、CLAHEローカルコントラスト）を追加。**1フレームあたりの処理コストが約2倍**となるため、より豊かな画質が得られる反面、LIVEのフレームレートは低下します。 |
| `custom_temp` | `natural` だが、ホワイトバランスを `custom_cct_k` ケルビンに固定 （DLSは無視され、バックエンド側で2000～10000 Kにクリップされます）。 |

このプロファイルは**ライブプレビュー専用**の速度/ルック調整ノブです：保存されたキャプチャは、選択されたプロファイルに関係なく常に豊かで高品質な仕上がりが得られます。したがって、フレーム時間を確保するために `natural` を選択しても、ディスクに書き込まれるデータの品質は低下しません。未知のプロファイルは `ValueError` を引き上げます。クロロス・バックエンド にアクセス可能な場合、変更内容はそちらにもPOSTされるため、次のプレビューフレームに反映されます（バックエンドを持たない direct-SDK ユーザーでも、設定の変更は反映されます）。

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### モノクロ (M3M) カメラと `Calibration`

モノクロ **M3M** カメラ (`M3M-<lens>-F<wavelength>`)はシングルバンドです：グレースケールプレーンが1つで、ベイヤーモザイクも3×3のスペクトルクロストーク行列もありません。`Calibration`はこれを認識し、`is_mono`フラグを割り当てます。反射率は依然としてバンドごとの放射測定マップとして適用されます（アンミックスは単位行列となります）が、単一のカメラに対するマルチバンド演算は、無意味な結果を返すのではなく、適切な値を生み出します：

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

モノクロハードウェアから植生指数を構築するには、異なる波長の複数の M3M カメラをアラインメント済みのマルチバンド・スタックに結合し（[アレイのアラインメント](#array-alignment)を参照）、1 台のカメラではなくそのスタック全体に対して指数を計算します。

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

> **`apply_sensor_settings` で受け入れられるキー**— 具体的には `integration_time_ms`、`frame_avg`、`ae_enabled`、`sunshine_diffuser_installed`（DAQ-E；`cap_id`への移行に伴い非推奨）、`filter_model`（DAQ-M）、および`cap_id`（すべてのDAQ種別；`None`/`""`/`"none"` = センサーのみ、キャパシタンス補正なし）。未知のキーは**黙って無視**されます — 例例：`{"integration_time": 64}`は何も行わない（`integration_time_ms`でなければならない）。`{"applied": [...], "errors": {...}}`を返し、例外は決して発生しない。

`chloros_sdk` は、上記で使用されたコアサーフェスのみを再エクスポートします。完全な `daq_sdk` パブリック API（22 個の名前）には以下が追加されています。これらを `daq_sdk` から直接インポートしてください：

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

「Chlorosで何らかの問題が発生した」場合を処理するために、基底クラスをキャッチします:

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

> `ChlorosAuthenticationError` および `ChlorosConfigurationError` は、他のクラスと同様に最上位レベルでエクスポートされています。 図に示すように、`chloros_sdk.exceptions`からもインポート可能です。

階層：

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

### 1. カスタム進行状況バーを使用したフォルダの処理

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

### 3. プロジェクト主導型キャプチャ・キャンペーン

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

### 4. マルチカメラ・フレームストリーム → NumPy パイプライン

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

### 5. ヘッドレス・ダイレクト・ハードウェア（バックエンドなし）キャプチャスクリプト

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

### 7. キャプチャ・レシピ相当（純粋なPython）

CLIのレシピDSLには、Pythonに相当する直接的な形式があります：

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

スマートコネクトのエントリポイント — `connect_camera`、`connect_array`、`connect_daq_sensor`、および `discover_lattice_cameras` — は、バックエンドが `127.0.0.1:5000`（スマートコネクト・サーフェスのデフォルトの URL）でリスニングしていることを前提とする、HTTPのシンクライアントです。 GUI または CLI がすでに実行中の場合は、そのいずれかが動作しています。スクリプトのみを実行した状態では、動作しているものが存在しない可能性もあるため、これらの関数は、**バンドルされたバックエンドバイナリを自動起動**します（ウィンドウなし、 `ChlorosLocal` と同じ方法）し、最初の呼び出しが行われるまで最大 `backend_startup_timeout` 待機します。

ルール：

- **起動されるのはローカルのURLのみです。** `localhost` / `127.0.0.1` / `[::1]` を指す `backend_url` は対象となります。それ以外のホストは他者のマシンであるとみなされ、決して起動されません。
- **バックエンドは再利用のために実行されたままになります** （CLIと同様）— スクリプトが終了しても暗黙のシャットダウンは行われません。スクリプトを再実行すると、稼働中のバックエンドが再利用されます。
- **これらの呼び出しのいずれかで `auto_start_backend=False` を指定してオプトアウト**（例： リモートバックエンドを指定した場合や、バックエンドのライフサイクルを自身で管理している場合など）。

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

バンドルされたバイナリが見つからないか、起動できない場合、その後のHTTP呼び出しは、単なる「接続拒否」トレースではなく、対処可能な **プラットフォーム対応**な`ChlorosConnectError`を発生させます。単なる接続拒否のトレースではなく、Windowsではデスクトップアプリや`chloros-cli`コマンドを、Linux（GUIなし）では`chloros-cli`コマンド、あるいは`.deb`へと誘導します。

---

## 環境とヘッダー

SDKは、すべてのバックエンドHTTP呼び出しに`X-Chloros-Client: sdk`のフラグを付けます。バックエンドでは、GUIの無料プランではなく、SDK / CLIのライセンスルール（ログイン**および**有料のChloros+プランが必要）が適用されます。これはインポート時に自動的に設定されるため、ユーザー側で何かを行う必要はありません。

`http://localhost` および `http://127.0.0.1` はローカルバックエンドとして検出されます。他のホスト（例：自社アナリティクスサービス）への呼び出しは変更されません。

バックエンド URL を上書きするには、`backend_url=`（または `api_url=` および `ChlorosLocal`）を指定します：

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
- SDK は、動作をバンドルされたバックエンドのバージョンに紐付けます。 古いSDKと新しいバックエンドを混在させても通常は動作しますが（前方互換性のあるエンドポイント）、新しいSDKと古いバックエンドを混在させると、新しいエンドポイントで`404`エラーが発生する可能性があります。この場合は、デスクトップアプリを同等のバージョンにアップグレードしてください。
- スマートコネクトのインターフェース （`connect_camera` / `connect_array` / `connect_daq_sensor`）およびネットワーク分析エンドポイントは、安定したJSONスキーマを返します。新しいフィールドは追加される形となります。

---

## トラブルシューティングのヒント

- **`ChlorosAuthenticationError: Login required`** → このマシンで `chloros-cli login EMAIL PASSWORD` を 1 回実行するか、Chlorosデスクトップアプリ経由でサインインしてください。
- **`ChlorosConnectError: No Chloros backend is running …`** → スマートコネクトの呼び出しによりローカルバックエンドが自動起動されるため、このメッセージが表示されるのは、バンドルされたバイナリが見つからない、または起動できない場合に限られます （例：デスクトップパッケージがなく、pipのみがインストールされているホスト）。このメッセージはプラットフォームに応じて異なります。Windowsでは、デスクトップアプリを開くか、任意の`chloros-cli`コマンドを実行してください。 Linux では、`chloros-cli` コマンドを実行するか（GUI は存在しません）、`.deb` をインストールしてください。リモートバックエンドの場合は、`backend_url=`（および `auto_start_backend=False`）を指定してください。
- **`CAMERA_AVAILABLE == False`** インポート時 → `lattice_sdk`の読み込みに失敗しました （通常、Arena SDK ランタイム DLL がインストールされていないため）。カメラ以外のサーフェスは正常に動作します。
- **Array connect がネイティブ解像度未満を返す**→ バックエンドのスマートプリップ機能が、通信回線に収まるようフレームサイズを自動的に縮小しています。原因を確認するには `analyze_array_network()` を使用し、 その後、リンクをアップグレードするか、縮小を受け入れるか、`force_tier="slip-emit-and-capture"`を指定して順次キャプチャを行ってください。この縮小による安全策は、集計上のオーバーサブスクリプション（`oversubscribed: true`、fpsフィールド0）には**対応していません**： 回線に対してカメラ数が多すぎる場合は、ビニングやROIでは解決できません。カメラ数を減らすか、ジャンボフレームを有効にするか、より高速なNICに切り替えてください（[オーバーサブスクリプション](#over-subscription-the-per-cam-floor)を参照）。
- **`analyze_array_network()` が、NIC の RX リングが極めて小さい（~0.26 MB）と報告する／「FRAMES WILL DROP」というメッセージが表示される** → ホスト NIC のの受信リングがデフォルト設定のままになっている（NICドライバの更新後に32にリセットされることが多い）。Realtek製USB 10GbEアダプタを使用している場合は、`ReceiveBufferLen=256`および`PendingReceives=64`（権限昇格）を設定し、バックエンドを再起動してリングを読み直させるリングを再読み込みするようにしてください。詳細な手順：[CLI 参考 → ホストNICの設定とチューニング](cli-reference.md#host-nic-setup--tuning-lattice-arrays)。
- **再起動／シャットダウン時にホストがハングし、その後WMIで`Invalid class`エラーが発生／NICが有効化されない** → 古い USB 10GbE ドライバが原因で `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`) が発生しています。アダプタドライバを最新バージョン (2026 以上) に更新し、受信リング設定を再適用してください。 [CLI リファレンス → ホスト NIC の設定と調整](cli-reference.md#host-nic-setup--tuning-lattice-arrays)を参照してください。
- **反射率の測定が拒否されました** → 絶対スケールの反射率を取得するには、ライブ DAQ をカメラ（またはアレイ）にバインドする必要があります。GUI 経由でバインドするか、ペアリングされたセンサーを必要としない `processing="radiance"` (W/m²/sr/nm) を使用してください。
- **`smart=True` のキャプチャに予想以上に時間がかかる** → AE の収束はシーンの動的特性に依存します。より高速な （安定性の低い）トリガーを使用したい場合は、`exposure_tolerance_pct`の値を厳しくするか、`stability_window_s`の時間を短縮してください。

---

## 関連項目

- [CLI リファレンス](cli-reference.md) — すべてのCLIサブコマンドは、SDK呼び出しに対応しています。
- [DAQ センサーガイド](../daq/README.md) — センサーごとの配線、校正、および記録に関するルール。
- オンラインドキュメント: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
