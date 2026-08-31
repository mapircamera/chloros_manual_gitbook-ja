# Chloros CLI リファレンス

**バージョン:**

1.2.0**作成日:**2026-07-29 19:19 ·**改訂日:** 2026-08-30**対象読者:** LLMによる利用に最適化されていますが、人間が読める形式です。**適用範囲:** `chloros-cli`のユーザー向けサブコマンドすべて。オプションおよびコピー＆ペースト可能な例を含みます。

本書は、MAPIR Chloros に同梱されているコマンドラインツール「`chloros-cli`」の完全なリファレンスです。LLM（あるいは人間）が、ソースコードを確認することなく、以下のリストからサポートされているあらゆるワークフローを構築できるよう、意図的に網羅的に作成されています。

要点のみを確認したい場合は、以下へ進んでください：
- [5分クイックスタート](#five-minute-quickstart)
- [LATTICE カメラの初回接続ワークフロー](#lattice-camera-first-connect-workflow)
- [DAQ センサーの初回接続ワークフロー](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [キャプチャモード、レコーダー、およびオフライン再処理](#capture-modes-recorders--offline-reprocess)

---

## 表記規則

- すべてのコマンドには「`chloros-cli`」という接頭辞が付きます。Windows ではバイナリ名は「`chloros-cli.exe`」となり、Linux /Jetson では `chloros-cli` となります。
- オプションの引数は `--flag` の形式で示されます。必須の位置指定引数は角括弧なしで示されます。
- デフォルト値が指定されている場合、 フラグを省略するとその値が使用されます。
- CLI は、Chloros バックエンド（`127.0.0.1:5000` 上の Flask サーバー）上の軽量な HTTP クライアントです。バックエンドはほとんどのコマンドによって自動起動されます。 `CHLOROS_BACKEND_URL=<url>`は、リモートバックエンド上の**`lattice`**、**`project`**、および**`daq pool-*`** コマンド群をリモートバックエンドに指向します。これらはコアコマンド（`process`、 `login`、`logout`、`status`、`export-status`、`time-sync`、 `selftest`）は、意図的に `http://127.0.0.1:<port>` を固定し、これを無視します（IPv4 リテラルにより、Windows&#x27; が回避されます `localhost`→`::1` によるリクエストあたり約2秒のペナルティを回避します）。[環境変数]（#environment-variables）を参照。
- すべての SDK / CLI への呼び出しには、Chloros+ アカウントでのログインが必要です（各マシンで `chloros-cli login` を1回実行してください； `~/.chloros/`にキャッシュされます）。
- 例ではLinuxのパスを使用しています。Windowsでは、`/home/user/...`を`C:/Users/.../...`に置き換えてください。

---

## トップレベルの概要

```
chloros-cli [global options] COMMAND [command options]
```

### グローバルオプション

| フラグ | 説明 |
| --- | --- |
| `--backend-exe PATH` | 自動検出されたバックエンド実行ファイルを上書きします。 |
| `--port N` | バックエンドのHTTPポート（デフォルト： `5000`）。 |
| `-v, --verbose` | 詳細出力を有効にする。 |
| `--restart` | バックエンドを強制再起動する (実行中の `backend_server.py` をすべて終了)。 |
| `--version` | バージョンを表示 (`Chloros CLI 1.2.0`)。 |
| `--help` | トップレベルのヘルプを表示。 |

### コマンド索引

| コマンド | 目的 |
| --- | --- |
| [`process`](#chloros-cli-process) | 「Survey3」または「LATTICE」のキャプチャが保存されたフォルダをエンドツーエンドで処理します。 |
| [`login`](#chloros-cli-login) | 「Chloros+」アカウントを使用して、このマシンを認証します。 |
| [`logout`](#chloros-cli-logout) | キャッシュされた認証情報をクリアします。 |
| [`status`](#chloros-cli-status) | 現在のライセンス／認証ステータスを表示します。 |
| [`export-status`](#chloros-cli-export-status) | `process`の実行中のLive Thread-4エクスポートの進行状況。 |
| [`language`](#chloros-cli-language) | CLIの表示言語を設定または一覧表示します（38言語に対応）。 |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | デフォルトのプロジェクトフォルダ（GUIと共有）。 |
| [`update`](#chloros-cli-update) | CLIの更新を確認してインストールします（Linux /Jetson）。 |
| [`selftest`](#chloros-cli-selftest) | システム診断およびスモークテスト。 |
| [`time-sync`](#chloros-cli-time-sync) | PTPグランドマスターの状態確認／制御。 |
| [`lattice`](#chloros-cli-lattice) | LATTICEカメラの制御およびキャプチャ（45以上のサブコマンド）. |
| [`daq`](#chloros-cli-daq) | DAQスペクトルセンサー制御（DAQ-U / DAQ-M / DAQ-E）。 |
| [`project`](#chloros-cli-project) | 保存済みのChlorosプロジェクト（カメラ＋DAQ）を開いて実行する。 |

---

## インストール

`chloros-cli`は、サポートされているすべてのプラットフォームにおいて、Chlorosデスクトップインストーラ内に同梱されています。CLIの別途ダウンロードは必要ありません。プラットフォームパッケージをインストールすると、デスクトップアプリおよび それが駆動するバックエンドバイナリが、`PATH`に追加されます。

最新のダウンロード：[`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> インストーラーには、`Chloros_CLI.bat` / `Chloros_CLI.ps1`、`Launch_CLI.*`、`chloros-cli.sh`) も同梱されており、これらはすぐに使える CLI シェルを起動します。これらの詳細については、 [CLI ユーザーガイド](../CLI.md)で解説されており、ここでは重複して説明しません。

### Windows (.exe)

1. ダウンロードページからWindowsインストーラをダウンロードします。
2. `Chloros-Setup-x.y.z.exe`を実行し、ウィザードの指示に従います。デフォルトのインストールパスは`C:\Program Files\Chloros\`です（CLIは、インストーラによってPATHに追加される`C:\Program Files\Chloros\cli\`に配置され、インストーラーによって PATH に追加されます)。
3. 新しいターミナル（`cmd.exe`、PowerShell、またはWindowsターミナル）を開き、更新された`PATH`が認識されるようにします。

```powershell
chloros-cli --version
```

インストーラーは自動的に`chloros-cli.exe`をシステムの`PATH`に追加し、LATTICEカメラに必要なArena SDKランタイムをバンドルします。

### Linux amd64 (.deb)

Ubuntu 22.04 LTS 以降、または Debian ベースの x86_64 ワークステーション向け。

> **Ubuntu 20.04 はサポートされていません。** パッケージの依存関係リストは、
> バックエンドが実際にリンクしているものに基づいており、これには `libc6 (>= 2.34)` が含まれます。
> focal は glibc 2.31 を同梱しています。`apt`は、実行時に失敗するのではなく、
> インストールを拒否します。

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

この.debパッケージは以下をインストールします：
- `chloros-cli` を `/usr/bin/chloros-cli` に
- コンパイル済みのバックエンドを `/usr/lib/chloros/chloros-backend` に
- Arena SDK ランタイム（LATTICEカメラ用）
- ノイズ除去モデル、キャリブレーションバンドル、および更新チャネルの設定

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

amd64版.debと同じ構成ですが、Jetson Orin / Orin NX / Orin Nano向けに最適化されたCUDAビルドが含まれています。

### マシンごとに1回認証

すべてのプラットフォームにおいて、SDK / CLI への呼び出しが機能するには、Chloros+ への1回限りのログインが必要です：

```bash
chloros-cli login user@example.com 'YourPassword'
```

認証情報は `~/.chloros/user_session.json` にキャッシュされます。

### インストールの確認

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Chloros+ のサブスクリプションが必要です。**CLI を利用するには、有効な Chloros+ プランが必要です。**Copper**はエントリーレベルの Chloros+ プランです — 有料のChloros+各プランには、CLI / SDKへのアクセス権が含まれます。無料の**Iron**プランのみ、このアクセス権がありません。（プランID対応表：`0`=Iron/無料、`1`=Copper、`2`=Bronze、`3`=Silver、`4`=Gold。）アップグレードは [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing) から行えます。
>
> この下限値は、CLIだけでなく、バックエンドによっても強制されます。有料プランに加入していない状態で、SDK / CLI フラグ付きのリクエストは、そのリクエストが `chloros-cli` から来たか、Python から来たかに関わらず、`403 PLAN_UPGRADE_REQUIRED` として拒否されます。 そのリクエストが `chloros-cli`、 SDK、あるいは独自に開発された HTTP クライアントから送信されたものであるかどうかにかかわらずです。ログアウト状態の呼び出し元には、代わりに `401 AUTH_REQUIRED` が返されます。プランの猶予期間中（月額プランの場合は30日間、年間プランの場合は有効期限まで）はオフラインでもアクセスが可能 が経過すると停止します。`chloros-cli status`は引き続き動作するため、その理由が確認できます（これは、ティア制限の対象外となっているSDK / CLIルートです — `GET /api/license-status`）。

---

## 5分で始めるクイックスタート

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

画像が格納されたフォルダを、Chlorosの完全なパイプライン（ターゲット検出 → キャリブレーション → ビネット → 反射率 → インデックスエクスポート）で処理します。

### 概要

```
chloros-cli process INPUT [OPTIONS]
```

### 位置引数

| 引数 | 説明 |
| --- | --- |
| `INPUT` | `.raw + .jpg`（Survey3）、`.tif/.tiff`（LATTICE）、または`.dng`ファイルが含まれる入力フォルダのパス。 |

### 共通オプション

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `-o, --output PATH` | デフォルトのプロジェクトパス下に、タイムスタンプ付きの新しいフォルダを作成します（設定がない場合は `~/Chloros Projects`）。 | 作成または再利用するプロジェクトフォルダ。 そのフォルダにすでに `project.json` が存在する場合、上書きするのではなく、`_1`/`_2` という名前の同階層のフォルダが作成されます。 |
| `-n, --project-name NAME` | 自動（タイムスタンプ） | プロジェクト名。 |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware`はChlorosを使用します+ ニューラルデベイヤーを使用しています。処理速度は遅くなりますが、画質は向上します。 |
| `--vignette / --no-vignette` | `--vignette` | ヴィネット補正。 |
| `--reflectance / --no-reflectance` | `--reflectance` | 反射率キャリブレーション（パネルターゲットが見つかった場合はそれを使用、LATTICEの場合はNISTのシリアルごとのキャリブレーションを使用）。 LATTICEマルチスペクトルでは、これは反射率**プロダクト**の切り替え機能も兼ねています — [プロダクトごとのエクスポート切り替え](#per-product-export-toggles-lattice-multispectral)を参照。 |
| `--ppk` | off | サイドカーファイルからのPPK GNSS補正を適用します。 |
| `--exposure-pin-1 MODEL` | off | 「Survey3」デュアルカメラリグの「pin-1」モデルを固定する。 |
| `--exposure-pin-2 MODEL` | off | 「pin-2」モデルを固定する。 |
| `--recal-interval SECONDS` | 0 | キャプチャ時間の N 秒ごとに、キャリブレーション計算を強制的に再実行する。 |
| `--timezone-offset HOURS` | local | 出力メタデータに組み込まれたタイムゾーンオフセットを上書きする。 |
| `--format FORMAT` | `TIFF (16-bit)` | `TIFF (16-bit)`、`TIFF (32-bit, Percent)`、`PNG (8-bit)`、`JPG (8-bit)`のいずれか。 |
| `--indices NAME [NAME ...]` | なし | 植生指数（`NDVI`、`NDRE`、 `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …）。 |
| `--input-level {auto,raw,debayered,processed}` | `auto` | LATTICE TIFF 用のパイプラインのエントリポイントを強制します（Survey3 .raw には影響しません）。また、**raw データがない**キャプチャを処理できるようにするエスケープハッチもあります — [キャプチャフォルダの外観](#what-a-captures-folder-looks-like)を参照。 |
| `--debayered / --no-debayered` | on | 線形デベイヤー処理された出力を生成 (`Debayered_Images`) を出力します。詳細は [製品ごとのエクスポート切り替え](#per-product-export-toggles-lattice-multispectral) を参照してください。 |
| `--preview / --no-preview` | オン | 表示プレビューを出力 (`Preview_Images`): RGB = ホワイトバランス (利用可能な場合はDAQ光源、それ以外はグレイワールド) + ガンマ; multispec = 偽色伸張。 |
| `--radiance / --no-radiance` | on | float32 ラディアンスを出力 (`Radiance_Images`, W/m²/sr/nm）。 |
| `--reflectance-source {daq,target,auto}` | `auto` | LATTICE反射率プロダクトの基準：`auto` = フレーム単位でQAを通過した-フレームターゲットが絶対基準、DAQ下向き放射（ρ = π·L/E）によるフォールバック； `target` = 厳格（DAQによる置換なし）； `daq` = DAQを優先。 [Per-製品エクスポート切り替え](#per-product-export-toggles-lattice-multispectral)を参照。 |
| `--target-reflectance-dir DIR` | なし | ユニットごとの**測定済み**ターゲット反射率スキャンのディレクトリ（`<serial>.csv`）；見つからない場合は、公称の T3/T4P スペクトルにフォールバックする。 |
| `--array-alignment / --no-array-alignment` | on | LATTICEアレイ：各キャプチャの`Chloros:Alignment*` XMPに記録されたモジュール間アライメントを、すべての処理済みプロダクト（デベイヤー処理済み／プレビュー／ 放射輝度／反射率／屈折率）に適用する。タグのない画像に対しては何も行わない。 |
| `--array-alignment-crop / --no-array-alignment-crop` | トリミング | アライメント済みのエクスポートデータを、アレイの共通オーバーラップ領域にトリミングし、すべてのモジュールが同一のフットプリントを共有するようにする。`--no-…`は、フル センサーキャンバスを保持する（ソースの外側は黒で塗りつぶす）。 |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | アライメントによる歪み補正のためのリサンプリング。`nearest`はソースのDN値を正確に保持する （放射測定値の画素間混合なし）。 |

### ターゲット検出オプション

| フラグ | 説明 |
| --- | --- |
| `--min-target-size PIXELS` | 検出器用の最小パネル・ターゲットサイズ（px）。 |
| `--target-clustering 0-100` | クラスタリング感度。 |
| `--target / --targets` | 入力フォルダをターゲットパネルのみとして扱う（サーベイ検出をスキップ）。 |

### 例

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### 製品ごとのエクスポート切り替え (LATTICE マルチスペクトル)

LATTICE 処理は、**1回のパスで該当するすべての製品**に展開されます*。4つのタイプ別トグル（`--debayered`、`--preview`、`--radiance`、`--reflectance`）はすべて**デフォルトでON** **ON**に設定されています**。**1つを無効にするには、`--no-<type>`形式を使用してください。RGBのマスターカメラは、デベイヤー処理済みデータとプレビューデータのみを出力し（バンドごとの放射輝度や反射率）のみを出力するため、`--radiance`/`--reflectance`はこれらに対してはノーオペとなります。Survey3`.raw`（標準の反射率／ターゲットパスに従うもの）については、これらのトグルは無視されます。*(旧来の`--radiometric-output {reflectance,radiance,sensor-response}`フラグは**削除**され、これらのトグルに置き換えられました。`sensor-response`レベルはもはや存在しません。）*

| 製品 | 出力 | DAQダウンウェルリングが必要？ |
| --- | --- | --- |
| `--debayered` | 線形デモザイク (`Debayered_Images`)。 | いいえ |
| `--preview` | プレビュー表示 (`Preview_Images`)：RGBはWB + ガンマ補正；multispecは偽色伸張。 | No. |
| `--radiance` | フル放射測定チェーンからの float32 W/m²/sr/nm（完全な放射測定チェーンからの値（`Radiance_Images`））。 | No. |
| `--reflectance` | uint16 反射率 ρ（`32768` = 1.0）、Pix4D対応。 | **はい**。ただし、QAに合格したフレーム内ターゲットによって固定されている場合は除く（下記参照）。 |

`--reflectance-source`は反射率の基準を選択します：**`auto`**（デフォルト）は、QAに合格したフレーム内のターゲットを**絶対基準**とする — ターゲットにアンカーされた経験的ラインチェーンは、除外パネル上でクロススコアされ、測定された勝者が適用される — ターゲットが存在しないかQAに失敗した場合は、DAQのダウンウェル分割 (ρ = π·L/E) にフォールバックします；**`target`**は厳格な動作（DAQによる置換なし）です；**`daq`**はDAQ優先の動作を選択します。ターゲットの幾何学的設定（ArUco / 固定ROI / ストリップ）はプロジェクトのターゲット設定から取得される；`--target-reflectance-dir DIR`はユニットごとの**測定済み** スキャン（`<serial>.csv`）を保持し、ターゲットユニットのシリアル番号/QRによって検索され、フォールバックとして公称T3/T4Pスペクトルが使用される。

DAQ反射率パスは、記録された**`.daq`**（DAQ-U/M/E）**または、画像データと共に存在するDAQ-Mネイティブの`.csv`**から、**タイムスタンプが一致するダウンウェリング**を自動的に特定します。カメラごとまたはDAQごとのキャリブレーションバンドルがローカルにキャッシュされていない場合、パイプラインは初回使用時に**AWSから自動的に取得**します （初回利用時にインターネット接続が必要；`~/.chloros/`としてキャッシュされます）。

#### 反射率ピクセルの読み取り（Pix4D / Metashape / 独自のスクリプト）

反射率は整数のDNとして保存されており、**ρ = 1.0 に対応するDNはソースカメラによって異なります**：

| ソース | ρ = 1.0 に対応する値 | 判別方法 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (ρ 2.0までの余裕あり) | ファイルに XMP `Chloros:PixelScale=32768` のスタンプが押されている。 |
| Survey3 | `65535` (ρ 1.0でクリップ) | `Chloros:*` XMPタグ — その欠如こそが*シグナル*である。 |

**`Chloros:PixelScale`を読み取り、それを除算する** 定数であると仮定するのではなく。このタグは uint16 ドメインで定義されているため、再スケーリングを行う出力形式間でも `32768` のまま維持されます — `TIFF (16-bit)`、`PNG (8-bit)`、`JPG (8-bit)`、および `TIFF (32-bit, Percent)` はすべて自己記述型です （保存されたデータ型をまず uint16 に正規化します：8 ビットからは ×257、float からは ×65535）。

> **設計上、スケールが適用されないケースが 1 つあります。** 8ビットソースのキャプチャ（BayerRG8）が8ビットTIFFとして書き込まれる場合、パイプラインは再スケーリングを行わずに0..255に*クリップ*され、再スケーリングは行われないため、ρ≈0.008を超えるすべての値は255に平坦化され、そのファイルにはスケール情報が記述されません。Chlorosは、`Chloros:PixelScale`および`MicaSense:RadiometricCalibration`のタプルを意図的に省略し を意図的に省略し、その理由をログに記録します。**LATTICE反射率ファイルにこのタグがない場合、スケールが設定されていると仮定しないでください。分割不可能なピクセルを分割するのではなく、16ビットまたは32ビットで再エクスポート**してください。決して割り切れないピクセルを無理に分割してはいけません。

#### エクスポート時に引き継がれるEXIF

`process`は、ソースキャプチャの**GPSブロックとそのExifIFD**をすべての出力ファイルにコピーするため、
エクスポート時には`FocalLength`、`FNumber`、`ExposureTime`、`ISO`、`DateTimeOriginal`、および
`CameraSerialNumber`が地理参照情報と共にエクスポートされます。

**`FocalLength`は写真測量において必須です。** Pix4Dは、
焦点距離と高度から地上サンプル距離を算出します。このタグがない場合、著しく誤ったスケールで計算されてしまいます。ある
49回の撮影を行ったオレンジ農園の飛行では、このタグの欠如により、411 m × 160 mの敷地が、再構築時には
47.8 km × 13 kmの敷地 — 大部分が「nodata」となる455 MPのオルソ画像となり、GSDを確認するまでは、
タイリングの問題やBigTIFFの問題と誤認されていました。もしオルソ画像が不自然な
スケールで出力された場合は、まずエクスポートされた成果物に`exiftool -FocalLength`を適用してください。

このコピーは意図的に **`-all:all`** ではありません。IFD0の構造タグは、
、また `ExifImageWidth` / `ExifImageHeight` は、
*ソース*のキャプチャを記述しているため除外されています。これらが含まれていると、サイズ変更されたエクスポートデータは、
XMPはコピーではなく直接書き込まれます。これは、ExifToolが
XMPブロックのコピー時に同一呼び出しのXMPタグを破棄してしまうためです（これにより、MAPIRの
キャリブレーションタグが失われてしまいます）。

### 出力の保存先

出力ファイルは、**プロジェクトフォルダ内に、カメラごとにグループ分けされ、さらにファイル形式ごとに分類されて**保存されます：

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE用のカメラフォルダは`LATT-<sensor>-<lens>-F<filter>`（キャプチャのEXIF
`Model`）で、Survey3の場合は `<model>_<filter>` となります。センサーとフィルターを共有するものの、
レンズが異なる2台のカメラは、ヴィネット、視野、歪みが異なるため、別々のツリーに分けられます。フォーマット
フォルダは `--format`に準拠しています： `tiff16`、`tiff8`、`png8`、`jpg8`、または`tiff32` の形式で
`TIFF (32-bit, Percent)`となります。

> **エクスポートされたすべての製品は、SOURCEファイルの名前を保持します。** radianceによる
> `capture_…_raw.tif`のエクスポートも、依然として`capture_…_raw.tif`と呼ばれます — 単に
> `tiff32/Radiance_Images/` 内に存在します。**ファイル名ではなくフォルダが成果物を識別します**。そのため、
> `*radiance*.tif` をグロブ検索しても何も見つかりません。代わりにディレクトリを基準に照合してください。

### 光センサーの記録 — キャリブレーション済みの `.daq` および `.csv`

`process` は、入力フォルダ内の `.daq` 記録も処理しますが、その際、**必要**
ありません。DAQ-U / DAQ-M / DAQ-Eを単独で飛行させただけでも完全な
キャプチャとなり、`.daq`ファイルのみを含むフォルダも有効な入力となります。

DAQは、キャリブレーション**なしで**記録することができます。これは、一般公開されている
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)レコーダーが
(`record_daq.py`)がデフォルトで行っている動作です。これらは生のセンサーカウントを書き出し、ファイルにタイムスタンプを付与することで、
Chlorosが**シリアル経由で** （まずローカルキャッシュから、
次に MAPIR クラウド）から取得し、適用します。`process`は結果を書き戻します：

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv`は、1回の読み取りにつき1行を格納します：UTC時刻、積分時間、総電力、
明所視／暗所視ルクス、PPFD（およびその青／緑／赤の分割値）、ピーク波長、そして
センサー独自の波長グリッドに基づく全スペクトル。`.daq`は、
再校正を行わずに再インポートします。

成功した場合、実行結果は `Light-sensor products written: N (calibrated .daq + .csv)` と報告されます。
括弧内の記述は実際に書き込まれた内容を表しており、具体的には
バンドルのないセンサーの場合は `(RAW COUNTS — this sensor has no calibration bundle)`、
両方を格納するフォルダの場合は `(N calibrated, M raw counts)` となります。バックエンドの独自の
見出し「`[DAQ-EXPORT]`」および「`[RUN-SUMMARY]`」も同様の方法で表現が導き出されます。これら
3つはいずれも、キャリブレーション済みの生データをエクスポートしたとは見なされません。

キャリブレーション・バンドルを取得できない DAQ-U / DAQ-M / DAQ-E の記録 — オフラインであるか、
そのセンサーのキャリブレーションデータがファイルに存在しない場合 — は、**理由を明記してスキップ**され、
`[DAQ-EXPORT]` 行に記録されます。生カウントを含む「キャリブレーション済み」ファイルとして書き出されることはありません。
インターネットに接続し、再実行してください。スキップ理由は、リーダーが実際に
そのファイルに対して特定したものです （スキーマが読み取れない、バンドルがない、書き込みエラーなど）であり、実行
サマリーには**個別の**理由がリストされます。つまり、1つの原因で20ファイルがスキップされた場合でも、
20回の繰り返しとしてではなく、1つの原因として扱われます。

#### DAQ-Aの記録データは生カウントとしてエクスポートされる

**DAQ-A** シリーズは、シリアル番号ごとのバンドルシステムが導入される以前に開発されたものであり、取得すべきキャリブレーションバンドルが
存在しません。その代わりに、現場で反射率ターゲットを用いてキャリブレーションが行われますが、これが
、そもそもキャリブレーション・バンドルが必要なかった理由です。これらの記録を拒否すると、
数値を抽出する手段が全くなくなってしまうため、**別の名前**でエクスポートされます：

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

ファイル内のフラグではなく別のファイル名となっているのは、この指定が
単なるファイル名としてメールで転送されても失われないようにするためです。 `.csv`ヘッダーには
`raw spectral sensor counts (NOT irradiance)`と記載されており、値はファイル**内**でのみ比較可能である
という警告が表示されています。これはまさに、ターゲットベースのキャリブレーションがそれらを目的として使用している点であり、
センサー間で比較できるわけではないのです。 電力依存の測光カラム（総電力、明所視および
暗所視ルクス、PPFD） は、カウントから積分されるのではなく**NULL**として書き込まれており、実行
サマリーには`RAW COUNTS`と記載されているため、ログに「エクスポート」されたものは放射照度として読み取ることができません。

レガシー**v1.01 / v1.02**の記録（DAQ-A-SDがこれらを書き込みます）には、読み取りごとのエポック情報がなく、
ファイルの書き込み時刻のみが含まれています。画像↔ダウンウェリング・マッチャーは依然としてこれらを拒否しますが — フレームを
書き込み時刻と照合すると、目に見えない形で誤りが生じるため — ですが、エクスポーターはこれらを読み込み、
CSVには「`clock=daq_created_on`」と出力されるため、製品側ではどのクロックが使用されているかが明示されます。

### 注意事項

- `process`は、 フォルダがSurvey3、LATTICE、または混合のいずれであるかを自動的に検出します。
- 進行状況はServer-Sent Eventsを介して配信されます。CLIでは、スレッドごとのリアルタイムの進行状況（検出中、分析中、処理中、エクスポート中）が表示されます。
- Linux /Jetson の場合、CLIはスワップ領域を確認し、大容量のフォルダを処理する前に警告を表示することがあります。また、テクスチャ対応のデベイヤーは、低消費電力の Jetson （Nano、Orin Nano）に対して、GPUの周波数制限を自動的に適用します。
- 処理が成功すると、実行レポートに書き出された画像プロダクトの数が表示されます（`Image products written: N`）。

#### 画像を書き出さない実行は失敗とみなされます

画像の出力を指定したにもかかわらず、実行結果が**なし**（`project.json`および
`calibration_data.json`のみ）だった場合、`process`はこれを 失敗とみなします。つまり、
`Processing finished but wrote no image products.` を出力し、**非ゼロで終了**するため、スクリプトで
これを検知できます。メッセージにはプロジェクトフォルダ名と、一般的な原因が記載されています：

- 入力フォルダがキャプチャとして認識されなかった（レイアウトと `--input-level` を確認してください）、または
- 要求されたすべてのプロダクトが、それらのカメラには適用できないとしてスキップされた（例：
  RGB専用カメラからの放射輝度／反射率を要求した場合など）。

`--verbose`を指定して再実行し、バックエンドのログで`[LATTICE-EXPORT]` / `[EXPORT-CHECK]`の行を確認してください。
これらは、カメラごとのスキップが説明されています。これらは、通常であればCLIの出力には反映されません。

意図的にメタデータのみの処理（すべてのプロダクトをオフにし、`--indices`を生成しない）でも、
**成功**とみなされます。これは、このケースでは空の画像出力が正しい結果だからです。**光センサーのみの処理**も同様です： `.daq`の記録が格納されたフォルダには、定義上エクスポートすべき画像が存在せず、
その実行結果は、代わりに書き込まれたキャリブレーション済みの`.daq` / `.csv`に基づいて評価されます。

---

## `chloros-cli login`

このマシンを Chloros+ のクラウドアカウントで認証してください。認証情報は `~/.chloros/user_session.json` に安全にキャッシュされます。

```
chloros-cli login EMAIL PASSWORD
```

### 例

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$`（パスワードの一部を削除または複製）。401エラーが発生した場合、CLIは自動的に`$$`Xを再度付加した形式で自動的に再試行し、次にパスワードの重複を排除した半分の文字列で再試行します。再試行が成功するとログインが完了し、次回使用するべき正しいシングルクォート構文が表示されます。

> **ヘッドレス／スクリプトでの使用：キャッシュされたセッションがないため、即座に失敗するのではなく、対話型プロンプトが表示されます。** バックエンド生成コマンド（`process`、`status`、`export-status`、`time-sync`、…）は、キャッシュされたライセンスやセッションなしで実行されると、処理を進める前に標準入力（stdin）で対話型の `Email:` / `Password:` プロンプトが表示されます。したがって、キャッシュされたセッションがない無人ジョブは、入力を待つ状態でハングします。ヘッドレス作業をスケジューリングする前に、マシンごとに 1 回 `chloros-cli login EMAIL PASSWORD` を実行してください 実行してください。

---

## `chloros-cli logout`

キャッシュされたセッションをクリアし、次回の呼び出し時に新規ログインを強制します。

```bash
chloros-cli logout
```

---

## `chloros-cli status`

現在のライセンス階層（Iron/Copper/Bronze/Silver/Gold）、認証済みユーザー、およびデバイスバインディング数を表示します。

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Thread-4のエクスポート進行状況をリアルタイムでポーリングします。別のシェルから`process`が実行されている**最中**でも安全に呼び出すことができます。

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

CLIの表示言語を設定します（CJK、RTL、インド系文字を含む38言語に対応）。スクリプトをレンダリングできないレガシーコンソールでは、英語にスムーズに切り替わります。

```
chloros-cli language [LANG_CODE] [--list]
```

### 例

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## プロジェクトフォルダコマンド

これらは、デフォルトのプロジェクトフォルダの場所を管理します （GUIと共有）を管理します。

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Jetsonのみ。 `version_url`と`/etc/chloros/update.conf`を比較し、一致する`.deb`のダウンロードとインストールを提案します。

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linux/JJetson上のCLIも、**起動のたびに自動更新チェック**を実行します（ノンブロッキングで、コマンドの実行を遅らせることはありません）： `/etc/chloros/update.conf`を読み取り、結果を1時間`~/.chloros/update_cache.json`にキャッシュし、新しいバージョンが存在する場合は`Update available: vX.Y.Z / Run: chloros-cli update`を出力します。エラーが発生した場合やWindowsでは、何も表示せずにスキップされます。

---

## `chloros-cli selftest`

7段階のスモークテストを実行します：バージョン、ポートの可用性、バックエンドの起動、`/api/test`、 `/api/system-info`（GPU/CUDA/PyTorch）、ノイズ除去モデルの有無、CUDA＋ノイズ除去の準備状況。

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTPグランドマスターのステータスおよび制御。ChlorosホストがPTPグランドマスターを実行し、LATTICEカムおよびDAQ-Eユニットは、デバイス間のタイムスタンプ同期のためにこれにスレーブとして接続されます。

| サブコマンド | 説明 |
| --- | --- |
| `status` | グランドマスターの状態、BMCA優先度、クロックIDを表示します。 |
| `peers` | Delay_Reqを介して検出されたスレーブ（カメラおよびDAQ-Eセンサー）を一覧表示します。 |
| `cameras` | カメラごとのPTPヘルス状態（`PtpStatus`、`PtpOffsetFromMaster`、 `PtpMeanPathDelay`）。 |
| `restart` | グランドマスタープロセスを再起動する。 |
| `set-priority --priority1 N --priority2 N` | BMCAの優先順位を上書きする。 |

### 例

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE カメラ制御。すべてのサブコマンドは Chloros バックエンドを経由します。このバックエンドはカメラプールを管理しているため、その後の CLI 呼び出しでは同じオープンハンドルが再利用されます。

### 共通オプション（ほとんどのサブコマンドで共有）

| フラグ | 説明 |
| --- | --- |
| `-d, --device N` | カメラインデックス（デフォルト：0）。 |
| `-s, --serial SN` | 特定のシリアル番号。 `--device`を上書きします。 |
| `--serials SN1,SN2,…` | マルチカメラ操作用の、コンマ区切りのシリアル番号。 |
| `--all` | 検出されたすべてのカメラに対して操作を実行します。 |
| `--exposure US` | 露光時間（マイクロ秒単位）。 |
| `--gain DB` | ゲイン（dB）。 |
| `--pixel-format FMT` | 例：`BayerRG8`、`BayerRG12`。 |
| `--width N` / `--height N` | 画像の寸法。 |
| `--preset {default,high_quality,high_speed,triggered}` | 設定プリセットを適用します。すべてのフリー実行されますが、`triggered`は例外です。これは、2番目のラインのハードウェアエッジを検知するとカメラが作動する設定になっており、そのラインに信号が入力されない限り、撮影を行わずに永久に待機し続けます。 |
| `-o, --output DIR` | 出力ディレクトリ（デフォルト：`output`）。 |
| `--packet-size {auto,jumbo,standard,N}` | GVSPパケットサイズ。 `auto` は ICMP+GVSP プローブを実行します。`jumbo` = 9000、`standard` = 1500。 |

### LATTICE カメラの初回接続ワークフロー

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### サブコマンドリファレンス

#### 検出と情報

| サブコマンド | 目的 |
| --- | --- |
| `lattice info` | 接続済みのカメラを一覧表示 （ベンダー、モデル、シリアル番号、IP、MAC）。 |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | ホストシステムを分析し、最適なカメラ設定を決定する。`--no-discover`はカメラの検出をスキップし （高速化、NICのみの分析）。 |
| `lattice network [--fix] [--estimate] [--cameras N]` | NIC設定の確認・修正；帯域幅/FPSの推定。 |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | 安定スキーマのバックエンドのネットワーク機能 + アレイの推奨 (`status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`）。`auto_capped_fps`は要求された解像度を維持するが、目標fps — `recommended.recommended_target_fps`を読み取り、接続ターゲットとして渡す。エラーではなく成功として扱う。 |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | カメラを開かずに実施する「What-if」分析。**`--n-active` は、この配列のみではなく、回線上のカメラの総数である**— スタンドアロンのカメラが同時にストリーミングする場合、またはカメラの数を過小評価した需要に対して回線予算が計算された場合にこれを発生させる（デフォルト：`len(--models)`）。常に集計値である `Wire budget:`（要求値 vs 衝突防止上限）と `Max cameras:` の行を出力し、アレイがワイヤーの容量を超過している場合は `** OVER-SUBSCRIBED**` をフラグとして表示します — [アレイのfpsおよびバーストモデル](#array-fps--burst-model)を参照。 |
| `lattice gpu` | GPUステータスを表示します。 |
| `lattice firmware [--update] [--force] [-y\|--yes]` | カメラのファームウェアを確認または更新します。ローカルの `.fwa` 選択は固定されています：`firmware/<MODEL_PREFIX>/` 内のビルドに一致するファイル`MIN_FIRMWARE_VERSION`に一致するファイルが存在する場合、フラッシュされます （フォールバックとして最高バージョンのみ）、そのためディスク上にステージングされた新しいベンダーイメージは、そのピンがバンプされるまで無効なままとなります。意図的に新しいリリースは、署名付きAWSマニフェストを介してユニットに配信され、新しいバージョンの場合はこちらが推奨されます。 |
| `lattice presets [--apply NAME]` | カメラのプリセットを一覧表示または適用します。 |
| `lattice status` | カメラのライブステータス。 |

#### キャプチャ

| サブコマンド | 目的 |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | 単一フレーム。**デフォルトですべてのエクスポート形式で保存されます** (`--processing all`)。[キャプチャのエクスポートレベル](#capture-export-levels-the-all-default)を参照してくださいを参照。`--levels`は明示的なサブセットを保存します（`--processing`を上書きします）。`--force-daq`は、割り当てられたDAQの読み取り値を、RAWのみのキャプチャであってもXPとして書き込みます。`--jpeg-quality` = JPEG 画質 1–100（デフォルト 95）。 |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Ctrl+C が入力されるまでディスクにストリーミングします。 |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | ブラウザベースのライブ MJPEG プレビュー。`--ae-damping` は自動露出のダンピングを設定します (0.4–100)。 |

#### センサーの調整

| サブコマンド | 目的 |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | 任意の GenICam ノードの読み取り/書き込み。 |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | 露出および AE。 |
| `lattice gain [--auto] [--off] [--set DB]` | ゲインおよびオートゲイン。 |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | センサーのROIおよびビニング。 |
| `lattice format [--set FMT] [--list]` | ピクセルフォーマット。 |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | ハードウェア／ソフトウェアトリガー。 |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (フラグなし = ワンショットWB) | WB操作。RGB /ベイヤー方式カメラのみ。モノクロM3Mではノーオペレーション（スキップ）となる。 |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB 表示カラーパイプライン。`natural` （デフォルト）は低コストなライブ処理です。`enhanced`は、デフリンジ＋ヴィブランス＋CLAHEローカルコントラストを追加し、ハブ・パリティの完全な外観を実現しますが、1フレームあたりの処理コストは約2倍となり、 そのため**ライブ**時のフレームレートは低下します — 保存されたキャプチャには、いずれの場合でも完全な処理が適用されます。RGB /Bayerカメラのみ；モノクロM3Mではスキップされます。 |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | 表示の彩度/コントラスト（RGBフィルター搭載カメラ）。モノクロM3Mではスキップされます。 |
| `lattice filter [--set NAME] [--list]` | カメラのフィルターモデルを設定します（`RGN-IMX265`、`OCN`、`NGB`、…）。 |
| `lattice power [--sleep]` | プローブの電源／熱ノードを測定し、低電力アイドル状態を切り替えます。 |

#### キャリブレーションとセンサー

| サブコマンド | 目的 |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | 反射率ターゲットを用いたキャリブレーション。 |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | 内蔵の降下光センサーコマンド。 |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | 既存の画像にヴィネット補正を適用する。 |

#### マルチカメラ （一時セッション）

| サブコマンド | 目的 |
| --- | --- |
| `lattice multi-info` | 同期役割を持つすべてのカメラを一覧表示する。 |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | 各カメラから1フレームずつ同期した画像を取得します。永続的な配列が接続されている場合、**デフォルトですべてのエクスポート形式**が保存されます。一時的な配列なしのフォールバックは**デバイヤー処理のみ**（残りのフレームについては、まず `array-connect` を実行してください）。 |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | 同期されたフレームをストリーム配信（トランジェント）。 |
| `lattice multi-test [--count N]` | GPIO同期タイミングテスト。 |
| `lattice multi-detect [--line LINE] [--json]` | GPIOマスター/スレーブ配線の自動検出。 |

#### アライメント

| サブコマンド | 目的 |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — 検出器/マッチング調整ノブ `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`、RANSAC 調整ノブ `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`、マルチフレーム結合 `[--averaging mean\|median\|inlier_weighted]`、幾何学的制約 `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`、空間的制約 `[--roi X0,Y0,X1,Y1] [--mask PATH]`、および-スレーブ上書き `[--per-cam-override SN:KEY=VALUE]`（再現可能） | ライブカメラからアライメントプロファイルを算出します。`--prefilter` のデフォルトは `gradient` です（エッジマップ；GUI/アレイアライナーと同様に — エッジはスペクトルバンドをまたいで保持される）。`--matcher flann`は、特徴点が約5000以上ある場合に効果を発揮する；`--averaging median`は1回の不良撮影に対して頑健である；`inlier_weighted`は一致数に応じて重み付けを行う；`--lock-scale`は、最も近い回転（スケールなし）に投影する。`--lock-axis`は、1つの並進成分をゼロにする。`--mask`は、すべてのカメラに適用される（カメラごとの設定には`--per-cam-override`を使用カメラごとの設定には`--per-cam-override`を使用、例：`--per-cam-override 214701292:method=phase`）。`--rms-threshold-px`は、再投影のRMSがゲート値を超えるキャリブレーションの保存を拒否する。 |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | アライメント済みのマルチバンドフレームを1枚キャプチャする。`--bit-depth`はデフォルトでカメラに合わせる；`--no-crop`はフルフレームを維持する（黒でパディング）；`--interpolation`（デフォルトは`linear`）および`--border-mode`/`--border-value`（デフォルトは `constant`/0）は CPU ワープを制御します — GPU パスはいずれの場合も双線形です。 |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | ストリームアラインされたマルチバンドフレーム（`align-apply`と同じワープ設定）。 |
| `lattice align-info --profile PATH [--json]` | プロファイルの詳細を表示。 |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | レイヤーの順序を変更。 |

#### インデックス / 植生計算

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

フラグの完全なセット：`--input PATH | --live --profile PATH`、`--preset NAME`（NDVI / NDRE / EVI / SAVI / GNDVI /…)、`--formula EXPR`、`--channel SYM=BAND`（繰り返し可能）、 `--capture-level raw|debayered|radiance|reflectance|unknown`（ソースのTIFFに記録されたキャプチャレベルを上書きします。デフォルト：TIFFのメタデータから読み取ります）、`--output PATH`、`--output-format all|raw|tif|colorized|lut|png`、`--gradient NAME|JSON`、`--vmin/--vmax/--percentile LO,HI`、`--bg-mode clip|transparent|indexColor|backgroundColor`、`--colorize`、`--list-presets`、`--list-gradients`。`--live` では、アライメント・ワープ・ノブも適用されます：`--save-multiband`、`--gpu/--no-gpu`、`--no-crop`、 `--bit-depth 8|12|16`、 `--vignette`、`--interpolation nearest|linear|cubic|lanczos`、`--border-mode constant|replicate|reflect|wrap`、`--border-value N`。

> **`--channel`のシンボルは大文字と小文字が区別されます。** シンボル側は、プリセットのチャンネル名と完全に一致している必要があります（プリセットでは小文字が使用されます。例：NDVI = `red`、`nir` — `--list-presets`を確認してください）。また、バンド側は、アラインされたスタック内のバンド名と一致している必要があります（オフラインモードの場合は、0を起点とするバンドインデックスでも可）。 `--channel red=Red_660 --channel nir=NIR_850`は正常に動作しますが、`--channel RED=660`は`channel_map missing entries`エラーで失敗します。

#### 持続的接続（Smart-Prep、GUIと同等のフロー）

これらのコマンドは、CLIの呼び出しをまたいで、バックエンドプール内のカメラ接続を維持します。

| サブコマンド | 目的 |
| --- | --- |
| `lattice cam-connect [--serial SN]` | プールに 1 台のカメラを追加します（単一カメラ、アレイなし）。 |
| `lattice cam-disconnect [--serial SN] [--all]` | 解放。 |
| `lattice cam-list` | プール内のカメラを一覧表示。 |
| **`lattice array-connect`**|**永続的な同期アレイを接続 （推奨されるエントリポイント）**。GUI によるスマートプレップフロー全体を実行します。 |
| `lattice array-disconnect [--array-id ID] [--all]` | アレイを解放します。 |
| `lattice array-list` | 接続済みのアレイを一覧表示します。 |
| `lattice array-status [--array-id ID]` | ライブFPS、PTP、最後のエラー。 |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | ライブアレイからの同期キャプチャ 1 回 — 単発 / 連続 / インターバル / 最速。 **デフォルトは `all`** （カメラごとに、該当するエクスポート形式ごとに1ファイル）。スキップされたカメラ（例：RGBは放射輝度／反射率から除外）は `Skipped: SN:<serial> (<reason>)` で報告されます。反射率に使用された DAQ 測定値は併せて保存され、`DAQ: <path>` で報告されます。 [キャプチャモード、レコーダー、およびオフライン再処理](#capture-modes-recorders--offline-reprocess)を参照してください。 |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | ライブの複合インデックスビューをビデオ/GIFとして記録 （モニタリング用；複合ストリームが開いている必要があります）。 |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | 高フレームレートの生ベイヤーバースト （分析用；オフラインで再処理）。 |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | 保存済みのRAWバーストを、キャリブレーション済みの動画に再処理する。 |

##### `array-connect` オプション

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--serials SN1,SN2,…` | すべての LATTICE カメラを自動検出（2台以上必要） | 最初のシリアル番号がマスターとなります。省略された場合、検出は LATTICE (`TRI032*`) モデルに絞り込まれ、それらすべてが接続されます。 |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO 同期ライン。 |
| `--target-fps F` | 自動 | マスタートリガーの発射レート。 |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | ティアピッカーを上書きします。 |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | **ホストの持続的なワイヤ帯域幅（MB/s） — アレイ全体の割り当てはこの数値に基づいて決定されます。** アレイがGVSP破損フレームを報告した場合は、この値を下げてください。自動設定値はNICが通知するリンクレートに基づいて算出されますが、USBアダプタ、帯域の狭いPCIeレーン、負荷の高い共有ファブリックでは実際の値より過大評価される傾向があります。この値はプロジェクトのアレイキャプチャブロックに永続化されるため、reopen / CLI / SDK による再接続で復元されます。詳細は [アレイの健全性](#array-health--which-subsystem-is-losing-frames)を参照してください。|
| `--binning {1,2,4}` | auto | ハードウェア・ビニング。 |
| `--no-recommend` | off | ネットワーク分析ステップをスキップします。 |
| `--no-ptp` | off | PTPを無効化（この場合、カメラ間のタイムスタンプは比較**できません**）。 |

### Smart-AE / Smart-Capture

LATTICEアレイは、接続されるとすぐにバックグラウンドで継続的なAEを実行しますが、新たに設定されたシーンは収束するまで少し時間がかかります。`array-capture --smart`は、**便利なパッケージ機能**として提供されています：アレイ内のすべてのカメラで AE が安定するのを待ち、その後キャプチャをトリガーします。セッションの途中でシーンを変更する際に使用してください。

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

デフォルトの安定化ポリシーは保守的です：タイムアウト5秒、安定性ウィンドウ1.5秒、露光ばらつき許容範囲±5％。自動化とは異なる動作が必要な場合は、SDK（`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`）で調整してください。

### キャプチャのエクスポートレベル（`all`のデフォルト）

このリリース以降、`lattice capture`、`lattice multi-capture`、および`lattice array-capture`は、**デフォルトで`--processing all`*に設定されます。* — 各カメラに適用されるエクスポートタイプごとに 1 つの保存ファイルが生成され、GUI の「すべてキャプチャ」の動作と一致します。レベルは以下の通りです：

| レベル | 出力 | 適用対象 |
| --- | --- | --- |
| `raw` | センサーから直接出力されるシングルチャンネル・ベイヤー（モノクロカメラ：単一バンド）。 | すべてのカメラ。 |
| `debayered` | 3チャンネルBGRデモザイク（モノクロカメラ：1チャンネルグレースケール）。 | すべてのカメラ。 |
| `radiance` | 完全な放射測定チェーンを経由したfloat32 W/m²/sr/nm。 | マルチスペクトル（M3C/M3M） のみ — **RGB-filterカメラの場合はスキップ**。 |
| `reflectance` | uint16 ρ（`32768` = 1.0）、Pix4D対応。 | マルチスペクトルのみ、かつ**DAQがバインドされ、かつカメラがキャリブレーション済み**の場合のみ**；それ以外の場合はスキップされる。 |
| `preview` / `display` | 完全なGUIプレビューチェーン（カメラのプロファイルに基づくガンマ補正）。`lattice capture`はこの処理を`preview`と命名する；`array-capture`/`multi-capture`は`display`を使用します。 | すべてのカム。 |

単一のレベルを指定すると、そのレベルのみが保存されます（`--processing debayered`）。`all`を要求した場合、指定されたカムに適用されないレベルはスキップ（および報告）され、エラーとはなりません。接続されていない、または校正されていないカムでも、`raw` / `debayered` / `preview`が返されます。

どの反射率フレームについても、実際に使用されたDAQのダウンウェル測定値は、画像の横にある**`.daq`** サイドカーに書き込まれ（これにより、キャプチャを後で再処理できるようになります）、`DAQ:` 行で報告されます。

### captures フォルダの構成

各エクスポートタイプは、`-o`の下にある**独自のサブフォルダ**に格納されるため、多階層のキャプチャでもタイプが混在することはありません：

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>`はキャプチャのタイムスタンプ、`<serial>`はカメラのシリアル番号であるため、1つの同期グループ内では
カメラ間でタイムスタンプが共通となります。**1つの非対称性に注意してください：** `display` レベルは
「`preview/`」という名前のフォルダに保存されますが、ファイル名自体には `_display` が保持されます。つまり、フォルダ名とファイル拡張子は
。未知のレベルの場合は、そのファイル名と同じ名前のフォルダに保存され、サブフォルダが
作成できない場合でも、ファイルが失われることなく出力ルートに書き込まれます。

**キャプチャフォルダを再処理する場合：**`chloros-cli process` を**キャプチャのルート**
(`output/`) を指定します。 `process`は通常、指定したフォルダのみをインポートしますが、そのフォルダに
画像が含まれておらず、サブフォルダが存在する場合、自動的に階層を辿ります。そのため、ルートのレベルにあるサブフォルダと
ルート上の`.daq`がすべて一度に取得されます。キャプチャの各レベルは、レベルごとに1つの画像としてではなく、
1つの画像としてインポートされ、他のレベルはモードとして利用可能です。

**レベルサブフォルダ**を直接指定する（例： `output/raw/`）ことも可能です。この場合、ルートフォルダ
`.daq`は残ったままになります。そのため、再読み込みを行う際には、DAQ読み取りデータをコピーするか、その場所を指定してください`raw/`から放射測定
プロダクトを導出する際は、DAQの読み取りデータをコピーするか、その場所を指定してください。そうしないと、タイムスタンプの一致を照合する対象がなくなります。

**処理は常に`raw`から開始されます。** 各キャプチャ内では、生フレームがパイプラインのソースとなります。
`debayered`、`radiance`、`reflectance`、および `preview` は表示モードとして提供されますが、パイプラインに
フィードバックされることはありません。派生したプロダクトを再処理すると、そのピクセルにすでに焼き込まれているヴィネット、CCM、および
放射輝度の計算が再度適用されてしまうため、Chlorosは二重処理を避けるために
これを拒否します。知っておくべき2つの結果があります：

- `index/` および `composite/` のレンダリングは **一切** 処理されません。これらは出力であり、キャプチャではありません —
  NDVIのLUTレンダリングには、意味のある放射輝度の解釈がありません。
- **以下を含めずに**エクスポートされたキャプチャフォルダ* `raw`（例： `array-capture --processing reflectance`）は、
  正当なパイプラインソースを持ちません。これらのキャプチャは通常通りインポートおよび表示されますが、`process`は
  それらをスキップし、次のように表示します：

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  もし、派生製品（`demosaic`が有効な状態でキャプチャされたハブセッションや、レガシーフォルダなど）を
  どうしても通過させる必要がある場合、`--input-level {raw,debayered,processed}`はエントリポイントを強制し、
  スキップ設定を上書きします。このフラグは意図的に設けられた回避策であり、 `auto`（デフォルト）は
  RAWデータのないキャプチャを処理することはありません。

### 混合フィルターアレイにおけるスキップされたキャプチャ

RGBとマルチスペクトルカメラを1つのアレイに混在させる場合、`array-capture --processing radiance`（または`reflectance`）はマルチスペクトルフレームを保存し、**スキップ** RGBカメラをスキップします。これは、広帯域センサーにおいてベイヤー単位ごとの放射輝度が意味を持たないためです。CLIは、保存された各ファイル（そのエクスポートレベルを含む）、書き込まれた各`.daq`、および各スキップを明示的に出力するため、ファイル数が予想外になることはありません：

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

スキップ理由のトークンは、`<level>-not-applicable-to-rgb-cam`というパターンに従います。 反射率も、`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` でスキップされることがあり、 また、バンドの大部分がDAQ光センサーの放射測定校正範囲（~374–974 nm）外にある場合は、`dls-uncalibrated-band-<nm>`でスキップできます。出荷されているSKUの中では、F988のみがこれに対応しており、そのサポートされるパスは反射率パネルワークフローです。

フィルターの種類に関係なくすべてのカメラを含めるには `--processing debayered`（または `display`）を使用し、カメラごとに適用可能なすべてのレベルを一度に取得するにはデフォルトの `all` を使用してください。

---

## キャプチャモード、レコーダー、およびオフライン再処理

これらはすべて**永続配列**上で動作します（最初に `array-connect` を実行してください）。これらは GUI のキャプチャパネルを反映しています。

### `array-capture` モード

`array-capture` は、4つのシャッターモードと一連のエクスポート切り替えオプションを備えた単一のコマンドです：

| モード | フラグ | 動作 |
| --- | --- | --- |
| **シングル** *(デフォルト)* | (なし) | 1つの同期キャプチャグループを実行した後、終了します。 |
| **連続** | `--continuous` | `Ctrl+C`、`--count N`、または`--duration S`が指定されるまで、連続してパスを実行します。 |
| **間隔** | `--interval S` | `S`秒ごとに1回のパス（各パスの開始時点から計測）、範囲は同じ。 |
| **最速** | `--fastest` | 生データのみ + 割り当てられたDAQ測定値 + 複合インデックスを組み合わせたもの。放射度/反射率/表示の計算を省略するため、フレームのレンダリングが高速化されます。`--processing raw --force-daq`を意味します。保存された`.daq`を後で校正済み製品に再処理します。 |

エクスポート切り替え（任意のモードと組み合わせ可能。すべてGUI/SDKエンドポイントを共有）:

| フラグ | 効果 |
| --- | --- |
| `--processing LEVEL` | 単一のエクスポートレベル、または `all`（デフォルト）. |
| `--levels L1,L2,…` | エクスポートタイプの明示的なサブセット（例：`raw,radiance,reflectance`）；**`--processing`を上書き**します。 |
| `--aligned` / `--no-aligned` | 配列の [アライメントプロファイル](#alignment) に基づいて、各メンバーの非生（non-raw）エクスポートをワープ（共登録）します。 生データはワープされませんが、メタデータに変換情報が保持されます。配列にプロファイルがない場合は、アライメントなしの状態にフォールバックします（警告が表示されます）。 |
| `--index` / `--no-index` | 設定されている場合、カメラごとの植生インデックス・オーバーレイを保存／スキップします。デフォルト：レンダリングします。 |
| `--force-daq` | 選択されたレベルで必要とされない場合でも、割り当てられたDAQ/DLS測定値を`.daq`サイドカーとして保存します （例：RAWのみのキャプチャ）であっても、割り当てられたDAQ/DLSの測定値を`.daq`サイドカーとして保存し、オフラインでフレームを反射率/指数に再処理できるようにする。 |
| `--smart` | トリガーする前に、すべてのカメラでAEが安定するのを待つ（[Smart-AE / Smart-Capture](#smart-ae--smart-capture)を参照）。 |
| `--compression {deflate,none}` | TIFF ピクセル圧縮。`deflate`（デフォルト）= ロスレス zlib L1 + 水平予測、フル解像度フレームあたり約4.1 MB；`none` = 非圧縮、書き込み速度は約5倍高速で1フレームあたり約6.3 MB — ディスク容量に余裕がある場合は、最大持続転送速度を得るために使用してください。どちらも無損失であり、インポート時の読み取り結果は同一です。 |

> **シングル書き込み TIFF + 持続転送速度モデル。**キャプチャデータは、ピクセル＋XMP＋IFD0の「メーカー/モデル」情報を含む**1回の**TIFFファイル書き込みパスで保存されます（フル解像度のMono12で測定：圧縮時36 ms／非圧縮時6.5 ms、従来の「書き込み→ExifToolによる再書き込み」方式の約148 msと比較）。残りのExifTool処理（EXIFサブIFDの微調整）は非同期のバックグラウンドワーカーで実行され、たとえそれが実行されなくても、フレームは完成し、インポート可能な状態になります。 なお、DEFLATE圧縮はPythonのGILを保持するため、圧縮書き込みはカメラごとのライタースレッド間で**並列化されません**。センサーレート（約10.4 fps）での8台カメラによるフル解像度連続撮影には、`--compression none`**および** NVMeクラスのディスク（持続書き込み速度約500 MB/s）が必要です。同じ設定項目は、`POST /api/camera/array/capture`上で`compression`として公開されています。

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — 統合インデックス動画/GIF（監視用グレード）

**ライブ複合インデックスビュー**に表示されている内容を、`.avi`（およびオプションで`.gif`）に記録しますに記録します。ライブコンポジットからデータを取得するため、フレームが記録されるには、コンバインドストリームが開いている必要があります（例：GUIで配列がプレビューされている状態）。 2 秒ごとに進行状況をポーリングし、`--duration`、`Ctrl+C` で停止するか、レコーダーが自動的に終了した時点で停止します。

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--array-id ID` | アレイのみ | ターゲットアレイ （1つしか接続されていない場合は省略）。 |
| `-o, --output DIR` | `output` | 出力ディレクトリ（バックエンドローカル）. |
| `--fps F` | `10` | 録画フレームレート。 |
| `--duration S` | Ctrl+C まで | `S` 秒後に自動停止。 |
| `--gif` | オフ | アニメーションGIFも書き込み。 |
| `--gif-only` | オフ | GIFのみを書き込み（`.avi`なし）。 |

### `array-burst` — 生ベイヤー高フレームレート連写（分析グレード）

グラブループの同期グループバッファを直接読み込みます — **キャリブレーションチェーンなし、 exiftoolもライブビューも不要** — そのため、カメラの最大グラブレートで動作します。RAWフレーム＋フレームごとのマニフェスト＋`<output>/bursts/<base>/`の下で、個別のDLS読み取りごとに1つの`.daq`を書き込みます。 オフラインで再処理（次のコマンド）するか、`--build`を指定して停止時に即座に実行する。

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--array-id ID` | 配陣のみ | ターゲット配陣。 |
| `-o, --output DIR` | `output` | 出力ディレクトリ （バーストデータは `<DIR>/bursts/<base>/` に格納されます）。 |
| `--duration S` | Ctrl+C まで | `S` 秒後に自動停止。 |
| `--max-frames N` | 無制限 | `S` 秒後に自動-停止 |
| `--build` | オフ | 停止後、直ちにバーストを再処理（`array-build-video`と同様）。 |
| `--products …` | `combined:index` | `--build` を使用する場合：どの動画を作成するか （下記参照）。 |
| `--fps F` | `10` | `--build`と併用時：出力動画のfps。 |
| `--save-tiffs` | オフ | `--build` 指定時: フレームごとのキャリブレーション済みTIFFも保存する。 |
| `--gif` | オフ | `--build`を指定した場合：アニメーションGIFも書き出す。 |

### `array-build-video` — 保存済みのバースト画像をオフラインで再処理

各RAWフレームを、保存済みの`.daq`測定値のうち最も近いものに時間的に照合し、**インポートパイプラインと同じ放射度／反射率／屈折率の処理チェーン**を通過させて、1つ以上の 動画をレンダリングします。

`--products`は、`kind:level`項目のカンマ区切りリストであり、ここで`kind` ∈ `per_cam` | `combined` であり、`level` ∈ `radiance` | `reflectance` | `index`。`level`のみ（`kind:`なし）の場合、デフォルトは`per_cam`となります。デフォルトは`combined:index`です。

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--burst-dir DIR` | (必須) | バーストフォルダのパス (`…/bursts/<base>/`)へのパス。 |
| `--products …` | `combined:index` | `kind:level` リスト（上記と同様）。 |
| `--fps F` | `10` | 出力動画のfps。 |
| `--save-tiffs` | off | 動画とともに、フレームごとのキャリブレーション済みTIFFを動画と一緒に保存する。 |
| `--gif` | off | アニメーションGIFも書き出す。 |

> **適切なレコーダーを選択してください。** `array-record`は*モニタリンググレード* — 表示されているライブコンポジットをそのままキャプチャするため、ストリームを開いておく必要があります。`array-burst` → `array-build-video`は*分析グレード* — フルレートで生のセンサーデータを保存し、その後、キャリブレーション済みの放射輝度／反射率／屈折率の動画を再構築します。ライブビューは不要です。

### モノクロ (M3M) シングルバンドカメラ

**M3M**シリーズは、ベイヤー方式の**M3C**のモノクロ版です。各カメラに 1 つの狭帯域干渉フィルターが搭載されています（`M3M-<lens>-F<wavelength>`、例：`M3M-L87-F685`）が搭載されており、センサーはベイヤーモザイクのない**単一のグレースケールバンド**を出力します。デモザイク処理の必要も、チャンネル間のクロストークを分離する必要も、ホワイトバランスの設定も一切ありません。つまり、RGBのカラー処理パイプライン全体が適用されないのです。

これがCLIにおいて意味すること：

- **`lattice white-balance`、`lattice color-profile`、 `lattice color`**は、モノクロカメラを検知すると、無意味な設定を適用する代わりに**1行のメッセージを表示してスキップ**します。同じセッション内でRGB／Bayer M3Cカメラに対しては、通常通り動作します。
- **`lattice calibrate`／`process --reflectance` / `array-capture --processing radiance`** は依然として機能する — 放射輝度および反射率は *バンドごと* の放射測定マップであり、1 バンドに対しては完全に明確に定義されている。モノクロフレームは **恒等** センサー応答行列 （3×3のアンミックスなし）であるため、この平面はキャリブレーション計算を無変更で通過します。
- **単一のモノクロカメラでは植生指数を算出できません。**NDVI / NDRE などには、少なくとも2つのバンドが必要です（例：Red + NIR）。モノクロハードウェアから指数を取得するには、異なる波長の**複数の** M3Mカメラを向け、それらを1つのマルチバンドスタックにアラインし、*その*スタックから指数を算出します：

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` シンボルは、プリセットのチャンネル名と**完全に**一致している必要があります（大文字小文字を区別します。NDVI は小文字の `red`,`nir` — `--list-presets`を参照）、またバンド側の名前は整列されたスタック内のバンドを指定する必要があります（オフラインモードでは0を起点とするバンドインデックスも受け入れられます。例：`--channel red=0 --channel nir=1`）。

スタック全体における識別子は、モデル文字列内の「`M3M`」トークンであり（「`M3C`」文字列には決して出現しない）、GUI／SDK上では「`is_mono`」として表示される。

---

## ホストNICの設定と調整（LATTICEアレイ）

LATTICEカメラはホストのイーサネットアダプタを介してGVSPをストリーミングするため、マルチカメラアレイの場合、アダプタの**ドライバ**および**受信リングサイズ**は、リンク速度と同様に重要です。設定が誤っていると、アレイ設定パネル（および`lattice network-analysis`／SDKの`analyze_array_network()`）に「`FRAMES WILL DROP`」または「`Reduce ROI to enable`」ゲートとして表示されます。これは、カメラ自体が正常に動作している場合でも発生します。

### USB 10GbE アダプタ — Realtek RTL8157（「Realtek USB 10GbE Family Controller」）

| 項目 | 必要な値 | 重要理由 |
| --- | --- | --- |
| **ドライバのバージョン**|**v10.67 以上 (2026年1月)**、INF `rtump64x64sta.inf` | 旧式の**2016**ドライバー（v10.65、`rtump64x64.inf`）は、**`DRIVER_POWER_STATE_FAILURE`（BSOD `0x9F`）**。移行処理がハングし（約5分のタイムアウト）、ユーザーが強制的に電源を切ると、繰り返される不適切なシャットダウンにより**WMIリポジトリが破損**し（PowerShellやツールが`Invalid class`エラーで動作しなくなる）、**次の起動時にUSBスタックが** 固まってしまう（NICが有効化されず、USBドライブの列挙が停止する）。正常な再起動を期待する前に、realtek.com（またはドングルベンダー）から更新を行ってください。 |
| **受信バッファ**— キーワード `ReceiveBufferLen` |**256**（ドライバの最大値） | NICのRXリング。ドライバのデフォルト値である**32**では、使用可能なリング容量が約0.26 MBしか残らない — マルチカメラバーストには明らかに小さすぎる — ため、アレイパネルは`Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB`を報告し、接続をブロックする。**256**ではリングは十分に大きく（**ラボの 10GbE ホストで測定した結果、約 13.5 MB**）、RX パイプラインにマルチカメラ GVSP バーストに対応する実質的な余裕が生まれます。（特定の設定で実際に *接続* できるかどうかは、**ドレイン対応**のアドミタンスチェックと**集計オーバーサブスクリプション**チェック——によって決定され、単純なバースト対リングの比較によるものではない；[アレイのfpsおよびバーストモデル](#array-fps--burst-model)を参照。） |
| **受信URB**— キーワード `PendingReceives` |**64** (最大) | 送信中のUSBリクエストブロック数。バースト吸収のため、受信バッファと併せて増加させる。 |
| **ジャンボフレーム** — キーワード `*JumboPacket` | **9014** | 9000 バイトの GVSP パケットに必要（1500 バイトの場合に比べ、1 フレームあたりのパケット数が 6 分の 1 になる）。 |

> ⚠️ **NIC ドライバーの更新により、これらの高度なプロパティはデフォルト値にリセットされます。**アダプタードライバーの更新または交換後は、`ReceiveBufferLen=256` および `PendingReceives=64` を**再適用**してください。そうしないと、「ハードウェアに変更はない」にもかかわらず、アレイパネルが再びゲート状態になります ハードウェアに変更はない」という状況でも、アレイパネルが再びゲート状態になります。これは、以前は正常に動作していたリグが突然接続を拒否する最大の原因です。**管理者権限**の PowerShell から適用してください（アダプタ名（例: `"Ethernet 5"`）を置き換えてください）：

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` は USB 10GbE アダプタに対応しています。** これにより、アダプタの種類が検出され、PCIe NIC（Intel I219 など）向けに適切な受信-リングのキーワードを調整します：PCIe NIC（Intel I219など）の場合は `*ReceiveBuffers`→2048、または Realtek **USB**10GbE コントローラ（**`*ReceiveBuffers`** を公開しないもの）の場合は、`ReceiveBufferLen`→256 および `PendingReceives`→64 （`*ReceiveBuffers`は公開されていません）。ターゲット値は各ドライバが報告する最大値（`NumericParameterMaxValue`）に制限されるため、範囲外の値が書き込まれることはありません。 **管理者権限**のターミナルから実行してください。レジストリベースの調整と同様に、変更はアダプタの再起動またはシステムの再起動後に有効になります。上記の手動による `Set-NetAdapterAdvancedProperty` コマンドも有効な代替手段です。これらは、再起動なしで （アダプタの再バインド）が可能です。

### ネットワークの基本（すべてのLATTICEリンク）

- **アドレス指定：** リンクローカル `169.254.0.0/16` (GigE Vision LLA)。ホストは静的アドレス `169.254.x.x/16` を割り当て、カメラおよび DAQ-E は同じ範囲内で自動的にアドレスを割り当てます。DHCP やゲートウェイは不要です。
- **パケットサイズ:**ジャンボ（9000）を推奨が、自動プローブに決定を任せること。接続のたびに再測定が行われ、GVSPプローブを通じてカメラの1500バイトのICMP上限をGVSPプローブで既に無視しているため、回線が実際に伝送可能な場所であればどこでもジャンボサイズになります。プローブよりも正確な値が分かっている場合のみ、`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`で固定し、恒久的な設定よりもコマンドごとの設定を優先してください。固定設定はプローブをスキップするため、経路が実際には9000を伝送できない場合、**すべての**キャプチャで`SC_ERR_TIMEOUT -1011`によりタイムアウトが発生します（[環境変数](#environment-variables)を参照）。
- **RXリングは`ReceiveBufferLen`に比例して拡張します：**デフォルトの`32`では、使用可能なリング容量は約0.26 MBです（マルチカメラバーストには小さすぎます）； 最大値の `256` では容量が十分（ラボの 10GbE ホストで測定した結果、約 13.5 MB）であり、実質的な余裕が確保されます。 設定が接続可能かどうかは、ドレインを考慮したアドミタンスチェック**および** 以下の集約オーバーサブスクリプションチェックによって決定され、単純なバースト容量とリング容量の比較によるものではありません。

### アレイのfpsおよびバーストモデル

「アレイ設定」パネル（および`lattice analyze-array`／SDKの`analyze_array_network`）の読み方：

- **バーストは、各カメラの実際のピクセルフォーマットでカメラごとに合計されます。**モノクロ**M3M**カメラは**Mono12 (2 B/px)**をストリーミングします；**M3C**ベイヤー方式のカメラは 8 ビットまたは 12 ビットでストリーミングされます（TRI032S は、BayerRG8 が要求されていても、黙って BayerRG12 を出力します）。したがって、4 台のカメラによるフル解像度のフレームは**約 12.6 MBとなるが、12ビットモノカメラが3台の場合は約25 MBとなる**。この予測は各カメラのフォーマットをそのモデル（IDキャッシュ）から判別するため、バーストは実際に通信線で伝送されるデータと一致する — 画一的なBayerRG8という仮定に基づくものではありません。
- **USBイーサネットアダプタは、銘板の表示にかかわらず、転送速度は200 MB/sが上限です。** リンク速度を持続転送速度に変換する効率表はPCIeに由来するものです。USB NICは*イーサネット*のリンク速度を広告していますが、実際にはUSBバスおよび そのドライバによって制限されます。かつてあるUSB 10GbEドングルは、約1063 MB/sの「持続転送速度」 — この数値は実際に検証されたことはない — が、その結果生じたペーシングにより、フレームの6～18％が破損していたにもかかわらず、正常な目標fpsが報告されていた。USB接続のNICは現在、絶対的な上限として**200 MB/s**に制限されている （制限はバス側にあるため、定格値に応じてスケーリングされることはない。USB 1 GbE アダプタは約 80 MB/s を出力するが、この影響を受けない）。機能レコードの `wire_ceiling_source` にはその旨が明記されており、`nic_is_usb` がこれをフラグ付けしています。いずれの場合も、`--wire-ceiling-mbps`で上書き可能です。
- **アドミタンスはドレインを意識しており、バースト全体対リングの比較ではありません。** 同時バーストは、バースト全体ではなく、*過渡的なバックログ* = `max(0, Σ per-cam arrival − host drain) × emit_window` に収まればよい。高速ホスト／低速CAMのファブリック（**PCIe**10Gホスト＋4× 1 GbE CAMの構成：到着レート ≈ 320 MB/s、 ドレイン ≈ 1063 MB/s）では、ホストのドレイン速度がカメラのフィル速度を上回り、バックログは ≈ 0 となるため、フル解像度のシミュレーション・エミットは**許可される**。25 MBのバーストが13.5 MBのリング容量を超えているにもかかわらず、である。同じ4台のカメラを**USB**10GbEアダプタの背後に配置すると、ドレインは1063ではなく200 MB/sとなる――到着がドレインを上回り、ロスはフレームレートの低下ではなく、破損フレームとして現れる。1 GbEホストでは、カメラの31.25 MB/sというDLThrの下限により、到着データが排出量を上回る→正しく**ブロック**される（*この*種類のブロックについては、ROIを縮小するか、ビニングを2以上にする）。アドミタンスは**2つ**の 接続ゲートの**2つ**のうちの1つであり、もう1つは後述の集計オーバーサブスクリプションチェックである。
- **予測fpsは、シリアル取得時の保守的な上限値である。**ホストの取得ループは現在、各カメラのバッファを**シリアルに**取得しており（各カメラにつき1つのエミットウィンドウ程度）、 そのため、サイクルは `max(readout+emit, N × emit)` によって制限され、カメラごとのエミットはカメラの**アクセスリンク**（1 GbE ≈ 80 MB/s）に制限され、 ホストのアップリンクではありません。4台のカメラによるフル解像度12ビットアレイの場合、これは**約2.8 fps**となり、実測値の約2.7～3.0 fpsと一致していますが、これは意図的に**露出に依存しない**ように設計されているため、暗いシーンでは露出時間が長くなるにつれて実際のフレームレートが上限をわずかに下回る場合がある。シリアル読み出しが実際のfps制限要因であり、これを並列化すれば上限は単一カメラの送信レートに近づく。
- **集計オーバーサブスクリプションは、接続を阻害する重大な要因である。**カメラごとの帯域幅割り当ての下限は**8 MB/s**（`ARRAY_PER_CAM_FLOOR_BPS`）に設定されているため、下限に達すると、集計需要（`per_cam × N`）が**衝突安全帯域の上限**（`sustained × sim_emit_factor`）を超える可能性があります。 1 GbEにおける実用的なフル解像度の上限：**1500 MTUで6台、ジャンボフレーム使用時は9台**。この上限は回線と下限値のみに起因するものであり、**フレームサイズとは無関係**であるため、**ビニングやROIの縮小は効果がない**（これらは*フレーム*あたりのバイト数を低下させるだけで、GevSCPDによって制御される*秒*あたりのバイト数には影響しない）；唯一の解決策は、カメラ台数を減らすこと、エンドツーエンドでのジャンボフレームの使用、またはより高速なNICの導入である。この症状は、スムーズなfps低下ではなくGVSPパケットロスの形で現れるため、`analyze-array`は達成可能な-fpsの数値をゼロにし、`**OVER-SUBSCRIBED**`を出力します。また、解像度が固定されている場合、`array-connect`は**接続を拒否**します（そうでない場合、ウォークダウンがフレームをビニングしますが、この種のブロックも解消されません）。`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`は、ベンチマーク作業においてこの接続拒否を**大きな警告**に格下げする — [環境変数](#environment-variables)を参照。

### 配列の健全性 — どのサブシステムがフレームを損失しているか

接続済みのアレイの `GET /api/camera/array/<array_id>/capability` は、**10 秒**のローリングウィンドウで再評価される
`health` ブロックを保持しています。これは、フレーム損失を
を、両方の原因にそれぞれ異なる修正が必要な2つの原因に分類し、どちらにも言及しない単一の「不完全」
レートを報告するのではなく：

| 項目 | 意味 | 該当サブシステム |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (シリアルごと) | フレームが**到着したが構造的に不正だった**— GVSPパケット損失。 |**ネットワーク**：ワイヤバジェット、ペーシング、NIC RXリング、 MTU |
| `never_arrived_rate_pct` (シリアルごと) | フレームが**まったく到着しなかった**— カメラが起動しなかったか、何も送信されなかった。 |**トリガー / 同期**: M8ケーブル、`--line`、`TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 各カメラの最低転送レート 各カメラごとの最悪のレート。 | — |
| `per_cam_rate_pct` | カメラごとの不完全率の合計（両方の原因を合わせたもの）。 | — |
| `stable_for_seconds` | 各カメラが0.01％未満の状態に留まっていた期間。 | — |

5％を超えると、バックエンドはスプリット名を記載した `[array-health <id>] WARN` 行をログに記録します。これは、
最初の違反時、重大度帯の変更時、状態が継続している間は1分ごとに、そして状態が解消された際に1回
記録されます。破損した半分のカメラは、各カメラおよび
原因ごとの最初のヒット時に `[gvsp-corrupt <SN>]` と出力し、
その後60秒ごとに集計値を出力します。すべての評価結果は依然としてバックエンドのログファイルに記録され、
出力内容にかかわらず、バッファごとにカウンタは更新されます。

同じレコードには、割り当て全体が占有している数値も報告されます：

| フィールド | 意味 |
| --- | --- |
| `wire_ceiling_mbps` | ホストの現在有効な持続ワイヤ予算（MB/s）。 |
| `wire_ceiling_source` | その数値の出典（説明文） — 例：`USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` または `user override 120 MB/s (auto said 200)`。 |
| `wire_ceiling_is_user_set` | `true` が、`--wire-ceiling-mbps`（または GUI の **ワイヤ・バジェット** フィールド）によって設定された場合。 |
| `nic_is_usb` | USBイーサネットアダプタの場合は `true` — 上記の 200 MB/s の上限を参照。 |

**読み方：** `gvsp_corrupt_rate_pct` が 0 以外で、`never_arrived_rate_pct` が 0 の場合
、トリガーと同期ケーブルの状態は完璧であり、損失の 100 % がネットワーク
経路に起因していることを意味します — `--wire-ceiling-mbps`に下げて再接続してください。逆のパターンは、
同期ケーブルまたはトリガーラインに問題があることを示しています。

> **`--target-fps`は、フレーム破損の決定要因ではありません。** GevSCPDのペーシングは
> 接続時に一度だけ設定されるため、トリガーレートを下げてもデューティサイクルは変化しますが、
> 同時送信バーストレートは変化しません。測定上、要求量を5倍削減しても改善は見られませんでした。
> ワイヤの最大転送速度を ワイヤ上限を240 MB/sから200 MB/sに下げたところ、同じリグで破損率が10.4 %
> から0.00 %に低下しました。

> **TRI032Sファームウェアでは、ストリーム途中の自動縮小は利用できません。** 稼働中のアレイは
> これを自ら修正することはできません。接続を切断して再接続することで、接続時のピッカーが
> 新しい上限値に基づいて再計画を行うようにしてください。

### 症状 → 解決策

| 症状 (アレイ設定 / 接続 / `analyze_array_network`) | 原因 | 対処法 |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`、`Reduce ROI to enable` | `ReceiveBufferLen`が32にリセットされる （通常、ドライバーの更新後） | `ReceiveBufferLen`を256に、`PendingReceives`を64に設定し、パネルを再表示する（バックエンドが以前のリングサイズをキャッシュしている場合は、バックエンドを再起動する） |
| 再起動／シャットダウンでハング；その後 `Invalid class` WMI エラー、NIC が有効化されない、USB ドライブが認識されない | 旧 2016 年版 Realtek USB 10GbE ドライバー → BSOD `0x9F` → 強制 電源オフ | アダプタドライバを v10.67 (2026) 以上に更新し、上記の受信リング設定を再適用 |
| 接続は成功するが、ネイティブ解像度未満が返される | Smart-prep が回線に合わせてフレームを自動縮小 | リンクをアップグレード / 縮小を受け入れる / `--force-tier slip-emit-and-capture` |
| アレイは正常な目標fpsを報告しているが、実際にはその一部しか提供されていない； `health.gvsp_corrupt_rate_pct` 非ゼロ、`never_arrived_rate_pct` 0 | ホストが推定したワイヤ・バジェットが、実際に維持できる値を上回っている（USBイーサネットアダプタ、帯域幅の狭いPCIeレーン、 または共有ファブリック） | `--wire-ceiling-mbps`の値を低く設定して再接続し、ヘルスブロックを再確認してください。 **`--target-fps` ではない** — GevSCPD のペーシングは接続時に固定される |
| 公開グループからカメラが欠落している； `health.never_arrived_rate_pct` が 0 以外、`gvsp_corrupt_rate_pct` が 0 | トリガー／ 同期パス — カメラが起動していない（ネットワークの問題ではない） | M8同期ケーブルと`--line`を確認し、すべてのメンバーが武装状態（`TriggerMode=On`）であることを確認してください |
| `**OVER-SUBSCRIBED**` / `Wire budget` が `analyze-array` で上限を超えているか、または固定された解決策による接続拒否 (`array over-subscribes the wire`) | カメラごとの総要求量 (下限 8 MB/s × カメラ数 N) が、衝突防止のための回線上限を超過 — 1 GbE @1500 MTU でのフル解像度カメラ 6 台、ジャンボフレーム使用時は 9 台 | カメラ数を減らす、エンドツーエンドでジャンボフレームを使用する、またはより高速な NIC を使用する。**ROI/ビニングでは解決しない** (上限はフレームサイズに依存しない）。`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`はベンチマーク上でこの制限を無効化する（パケットロスを許容する） |

---

## `chloros-cli daq`

Spectral-sensorコマンド。2つのクラス：
- **`pool-*`**— バックエンドの永続プールを通じてセンサーを駆動する、HTTPのシンクライアント。**これがサポートされているパスであり、出荷時のCLIに含まれる唯一のパスです。** バックエンドがトランスポートを管理するため、GUI、CLIおよびSDKスクリプトはすべて、シリアルポートの奪い合いをすることなく、1つのアクティブなハンドルを共有します。
- **その他すべて**(`test`、`record`、`live`、`stream`、`connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`、`reflectance`、`login`、`logout`、`status`) — ハードウェアへの直接アクセス。完全性を期すため、以下にその詳細を記載する。これらには、`daq`Pythonパッケージが必要だが、これは**出荷されたアーティファクトには一切含まれていない**。コンパイル済みのCLIにはこれが含まれていない（`scripts/Build-CLI.ps1`は`--nofollow-import-to=daq`を設定し、トランスポート `pyserial` / `bleak` / `zeroconf` にはこれが含まれています）、また PyPI の SDK パッケージにも含まれていません。 これらはソースコードをチェックアウトした場合にのみ動作するため、一般に利用可能なものではなく、MAPIR内部の開発パスとして扱うようにしてください。
- **`discover` / `list`** は両方の性質を併せ持っています： これらはソースコードのチェックアウトからはハードウェアへの直接コマンドとして動作しますが、出荷版ビルドでは `pool-discover` にフォールバックし、バックエンドがスキャンを実行します。 したがって、スキャンはあらゆる環境で動作します。これは、DAQ-MのBLE MACを把握する唯一の方法であるため、重要な点です。

> **`chloros-cli daq --help`** （および `-h` / `help`）は、`pool-*` のサブコマンドを一覧表示します。ヘルプは、実際に実行されるコマンドを反映するよう、意図的にプールクライアントにルーティングされています。出荷済みビルドで 出荷版ビルドで direct-hardware サブコマンドを実行すると、欠落しているパッケージ名を明示したエラーで終了し、`pool-*` へ誘導されます。何も黙って失敗することはありません。 (`discover` / `list` は例外です。これらは `pool-discover` にリダイレクトされ、問題なく動作します。)
>
> **顧客が必要とするすべての機能は、`pool-*`を通じて利用可能です** — 接続、ストリーミング、 キャリブレーション済みの`.daq`ファイルを記録し、キャッププロファイルを切り替えることができます。また、DAQはPythonから`chloros_sdk.connect_daq_sensor()`を使用して制御することも可能で、これには同じプールされたパスが使用されます。

### DAQセンサーの初回接続ワークフロー

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` リファレンス

| サブコマンド | 目的 |
| --- | --- |
| `daq pool-connect` (smart-detect) | バックエンド・プール内のセンサーを開く。 |
| `daq pool-connect --port PORT` | 特定のシリアルポートでのDAQ-U。 |
| `daq pool-connect --ble` | BLE経由のDAQ-M、MACアドレスの自動スキャン。 |
| `daq pool-connect --mac MAC` | 既知のMACアドレスでのBLE経由のDAQ-M (`--ble`を暗黙的に含む)。 |
| `daq pool-connect --eth-host HOST` | 既知のホストでのイーサネット経由のDAQ-E。 |
| `daq pool-connect --eth` | イーサネット経由のDAQ-Eをイーサネット経由で実行、ホストは自動検出（mDNS + ARPフォールバック；WindowsおよびLinuxのARPキャッシュが空の状態でも動作）。 |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | 積分ウィンドウ／AE状態の調整。 |
| `daq pool-connect --no-stream` | 接続するが、まだストリーミングを開始しない（`pool-stream --start`で再開）。 |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | キャップ補正プロファイル。バックエンドのデフォルトは `sunshine_cosine` です。 |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 接続せずに、接続可能なセンサーをすべてのトランスポートでスキャンします。**これが、DAQ-MのBLE MACアドレスを見つける方法です。** `daq discover` / `daq list`は、出荷時のビルドでは自動的にここにルーティングされます。 プール内で既にオープン状態のセンサーは一覧に表示されません（接続済みの DAQ-M はアドバタイズを停止するため）。それらの場合は `pool-list` を使用してください。 |
| `daq pool-list` | バックエンドプール内のすべてのセンサーを表示します。 |
| `daq pool-disconnect --sensor-id ID [--all]` | 解放します。 |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 最新のN個のスペクトラムフレームを表示します。 |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | ストリーミングの再開／一時停止。 |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | .daq 記録の開始／停止。 |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 実行時にキャップ補正プロファイルを交換する。 |

### ハードウェア直接操作サブコマンド（ソースコードのチェックアウト時のみ利用可能 — 出荷版ビルドには含まれません）

> 網羅性を期して記載しています。これらを使用するには、`daq` Python パッケージに加え、`pyserial` / `bleak` / `zeroconf` が必要です。これらは、コンパイル済みの CLI や PyPI SDK には含まれておらず、MAPIR ソースチェックアウトからのみ実行可能です。**リリース済みの Chloros ビルドを使用している場合は、 代わりに上記の `pool-*` コマンドを使用してください**。これらは、接続、ストリーム、記録、およびキャプチャの選択に対応しています。

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

開く、接続し、 保存済みのChlorosプロジェクト（`cameras.json` + `sensors.json` + `project.json`を含むフォルダ）を開き、制御します。すべてがバックエンドを経由するため、GUIとCLIは同一のハードウェア状態を生成します。

### サブコマンドリファレンス

| サブコマンド | 目的 |
| --- | --- |
| `project open PATH` | プロジェクトのデバイスマニフェスト（カメラ、アレイ、センサー）を出力します。 |
| `project devices PATH [--reconnect]` | 検出結果の一覧表示または再実行。 |
| `project connect PATH [--cameras-only] [--sensors-only]` | 保存済みのすべてのカメラ／ アレイ／センサーに接続する。 |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | 指定したカメラまたはアレイから単一のキャプチャを行う。 |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | 指定したカメラまたはアレイからNフレームのバースト撮影を行う （`-n/--count` デフォルト 5；`-i/--interval` フレーム間の秒数、デフォルト 0）。アレイのバースト処理では、繰り返される同期グループを重複排除します（陳腐化ウォッチドッグ） 。そのため、部分サイクル中のアレイは1つのフレームのNコピーを返すことができない。反復ごとの結果を出力する。 |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | ストリームをバックエンドジョブ経由でディスクへ書き込み。`--poll-interval` = `/stats` ポーリング間の秒数 （デフォルト 2.0）。 |
| `project sensor read PATH NAME [--json]` | 最新のスペクトルフレーム。 |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | .daq ファイルを記録。 |
| `project run PATH RECIPE.yaml` | YAML/JSONキャプチャレシピを実行します。`--dry-run`は実行せずに検証を行います。 |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | アレイのアライメントを計算します — [以下のフラグ表](#project-align-calibrate-options)を参照。 |
| `project align status PATH NAME [--json]` | 現在のアライメントプロファイルを出力します。 |
| `project align clear PATH NAME` | キャッシュされたプロファイルを削除します。 |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | 1つのスレーブの変換を微調整します。 |
| `project align export PATH NAME --to FILE` | プロファイルをJSONに保存します。 |
| `project align import PATH NAME --from FILE [--no-validate]` | 保存されたプロファイルを読み込みます。 |

#### `project align calibrate` オプション

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | 位置合わせ方法。**これらの表記は `lattice align-calibrate`**とは異なります。*は短縮形である `orb` / `akaze` / `phase` を使用します。このフラグに関しては、2つのコマンドは互換性がありません。 |
| `--model {translation, rigid, affine, homography}` | `affine` | モデルを変形して適合させる。 |
| `--frames N` | `1` | フレームのスナップショットを平均値に合わせて同期する。 |
| `--reference SN` | マスター | 参照カメラのシリアル番号。他のすべてのメンバーはこれにワープされます。 |
| `--max-features N` | `5000` | ORB特徴点数の上限。 |
| `--ratio-threshold F` | `0.75` | ロウの比率検定。 |
| `--ransac-threshold-px F` | `3.0` | RANSACのインライヤー閾値。 |
| `--min-matches N` | `15` | **品質ゲート** — インライアの一致数がこの値を下回る場合、解を拒否する。 |
| `--max-reproj-err-px F` | `4.0` | **品質ゲート** — このRMS再投影誤差を超える場合、解法を却下する。 |
| `--checkerboard RxC` | — | `--method checkerboard`のボード形状（例：`9x6`）。 |
| `--name PROFILE` | 空 | 保存されたJSONに埋め込まれたプロファイル名。**配列名ではありません** — それは位置情報である`NAME`です。 |

これら2つの品質ゲートが存在するため、キャリブレーションが解の算出に成功しても、
保存が拒否される場合があります。いずれか一方のゲートに失敗したプロファイルは、その後の
すべてのキャプチャで黙って位置ずれを引き起こしてしまうため、永続化されるのではなく拒否されます。

### 例

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### レシピ DSL

`project run RECIPE.yaml`は、一連のアクションを記述したYAMLまたはJSONファイルを受け付けます：

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

サポートされているアクション： `apply`、`wait`、`capture`、`stream`、`burst`、 `sensor`。`burst`アクションには、`name`（必須）、`count`（デフォルト値 5）、`interval`（秒単位、デフォルト 0）、`output`、`format`、および `settings`（カメラごとの設定形状は `apply`と同様）；アレイバーストは、`project burst`と同じ「新規同期済みグループ」ウォッチドッグを使用します。

実行例：

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## 環境変数

| 変数 | 効果 |
| --- | --- |
| `CHLOROS_BACKEND_URL` | バックエンド `URL` を上書き（デフォルトは ``http://127.0.0.1:5000``） — **この設定が有効になるのは、``lattice``、 `project`、および `daq pool-*` コマンドファミリーでのみ有効です。** コアコマンド（`process`、`login`、`logout`、`status`、`export-status`、`time-sync`、`selftest`）は、`http://127.0.0.1:<port>` ピンに設定し、この変数を無視します（IPv4 リテラルは、Windows `localhost`→`::1` によるリクエストあたり約 2 秒のペナルティを回避するため）、したがって常にローカルマシンをターゲットとします。 |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1`は、アレイのオーバーサブスクリプションによる接続拒否（CAMごとの総需要 &gt; 衝突-セーフなワイヤ上限値を超える場合、`pin_resolution`）を「警告を発して処理を続行」レベルに引き下げ、GVSPパケット損失を許容する。ベンチマーク用途のみ — [アレイfpsおよびバーストモデル](#array-fps--burst-model)を参照のこと。 |
| `CHLOROS_CLI_MODE` | CLI自体によって設定される。バックエンドに対し、並列処理を有効にするよう指示する。 |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0`は、GVSPフォールバックプローブをスキップします（ICMPの結果のみ）。 **これによりジャンボパケットが無効化されます。単にログの出力を抑制するだけではありません** — カメラは各パスにおいて最大1500までのDF pingにのみ応答するため、ジャンボパケットを検出できるのはこのプローブのみです。接続ごとにカメラ1台あたり約1秒の時間を節約できますが、ネットワークがジャンボパケットを伝送可能だった場合、帯域上限の約1.45倍のコストがかかります。設定時にSDKから警告が表示されます。 |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | GVSPパケットサイズをNバイトに固定します。プローブ処理を完全にスキップします。恒久的な設定（`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`）よりもコマンドごとの設定を推奨します。サイズを固定すると、前段のネットワークへの適応が行われなくなり、ジャンボパケットを伝送できないパスで9000 を固定すると、ジャンボパケットを伝送できないパスでは、**すべての**キャプチャが `SC_ERR_TIMEOUT -1011` でタイムアウトします。 |
| `TMPDIR` (Linux) | Nuitka の onefile 抽出ディレクトリを上書きします。CLIは、存在する場合、自動的に `/mnt/ssd/tmp`が設定されている場合は、これを自動的に使用します。 |

---

## 終了コード

| コード | 意味 |
| --- | --- |
| `0` | 成功。 |
| `1` | 一般的な失敗 （ほとんどのサブコマンドのエラー）。 |
| `2` | 引数のエラー。 |
| `130` | Ctrl+C により中断されました。 |

---

## トラブルシューティングのヒント

- **「ログインが必要です」** → このマシンで `chloros-cli login EMAIL PASSWORD` を 1 回実行してください。
- **「バックエンドに接続できません」** → Chloros デスクトップアプリを起動するか、バックエンドバイナリを直接実行してください (`chloros-backend`)、またはリモート接続の場合は`CHLOROS_BACKEND_URL`を確認してください。
- **`lattice` コマンドが &quot;LATTICEカメラドライバが見つかりません&quot;** → ArenaのSDKランタイムがインストールされていません。CLIにはWindowsに`win32api`が同梱されていますが、CランタイムはGUIインストーラに含まれています。
- **「Array connect」または「Array Settings」に「FRAMES WILL DROP」または「有効にするにはROIを縮小してください&quot;** と表示される → ホストNICの受信リングが小さすぎる（NICドライバの更新後に32にリセットされることがよくある）。[ホストNICの設定と調整](#host-nic-setup--tuning-lattice-arrays) を参照 — `ReceiveBufferLen=256`、 `PendingReceives=64` に設定してください。
- **再起動／シャットダウン時にマシンがフリーズし、その後 WMI `Invalid class` / NIC が有効化されない／USBドライブが認識されない** → 古いUSB 10GbEアダプタドライバが原因で`DRIVER_POWER_STATE_FAILURE`（BSOD `0x9F`）が発生。 アダプタドライバを更新してください — [ホストNICの設定とチューニング](#host-nic-setup--tuning-lattice-arrays)を参照してください。
- **Jetson スワップに関する警告** → ファイルベースのスワップを追加してください。CLIには、正確な`fallocate` / `swapon`コマンドが出力されます。
- **DAQダイレクトコマンドが欠落** → 想定通り：出荷時の`chloros-cli`は、`daq`パッケージを意図的に除外しているため、`pool-*`のみが存在します（PyPIのSDKにも含まれていません）。 バックエンドを介して同じセンサーを駆動する `pool-*`、または Python から入手可能な `chloros_sdk.connect_daq_sensor()` を使用してください。

---

## 関連項目

- [Python SDK リファレンス](sdk-reference.md) — すべての CLI コマンドに相当するプログラム言語での実装。
- [DAQ センサーガイド](../daq/README.md) — センサーごとの配線および校正。
- オンラインドキュメント: `https://mapir.gitbook.io/chloros/cli`
