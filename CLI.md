# CLI : コマンドライン

> **完全なリファレンス：**[CLI リファレンス](reference/cli-reference.md) には**各サブコマンドのすべてのフラグ** が記載されており、AIアシスタント向けに最適化されています。その URL をアシスタントに貼り付け、動作するコマンドを尋ねてみてください：`https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **AIツールのヒント：** このマニュアルのどのページも、そのページURLの末尾に `.md` を追加することで、生のMarkdown形式で利用可能です （例：`https://mapir.gitbook.io/chloros/reference/cli-reference.md`）。また、`https://mapir.gitbook.io/chloros/llms.txt`は、LLMが利用できるようマニュアル全体をインデックス化します。

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->
## CLI とは

`chloros-cli` は、デスクトップアプリ Chloros が使用するのと同じ処理エンジンのコマンドラインフロントエンドです。 これは、Chlorosバックエンド（`127.0.0.1:5000`上のローカルサーバー）上のシンクライアントであり、ほとんどのコマンドはバックエンドを自動的に起動するため、 そのため、スクリプトでは `chloros-cli process …` を 1 回呼び出すだけで済みます。

**Windows 10/11 (x64)**および**Linux (x86_64、および JetPack 6 上の NVIDIA Jetson arm64)** で動作し、 GUIを必要とせず、どのターミナルからでも実行可能です。以下のコマンドでインストールが正常に行われたか確認してください：

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

コマンド群の概要：

* **処理およびアカウント** — `process`、`login`、`logout`、`status`、 `export-status`、`language`（38言語 — [対応言語](supported-languages.md)を参照）、 `set-project-folder` / `get-project-folder` / `reset-project-folder`、`selftest`、`update` (Linux/Jetsonのみ)
* **実機** — `lattice`（LATTICEカメラ制御、45以上のサブコマンド）、`daq pool-*`（DAQ光センサー）、`time-sync` (PTP)
* **自動化** — `project`（YAMLキャプチャレシピを含む、保存済みのChlorosプロジェクトをヘッドレスで実行）

知っておくべきグローバルオプション：`--port N`（バックエンドポート、デフォルトは `5000`）、`-v/--verbose`、 `--restart`（バックエンドの強制再起動）、`--backend-exe PATH`。 完全な一覧については、[CLI リファレンス](reference/cli-reference.md)を参照してください。

***

## インストール

CLI は、すべてのプラットフォームにおいて **Chloros インストーラ内に同梱** されています。CLI を個別にダウンロードする必要はありません。 インストーラーは[ダウンロード](download.md)ページから入手してください。

### Windows

インストーラーは、CLIを以下の場所に配置します：

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

に配置し、そのフォルダをシステムの `PATH` に追加します。インストール後は、更新された `PATH` が認識されるよう、**新しいターミナルを開いてください**。また、インストーラーはインストールルートにランチャースクリプト (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) をインストールルートに配置し、さらに**Chloros CLI** スタートメニューのショートカットも配置されます。これらを実行すると、`chloros-cli`が起動した状態でターミナルが開きます。

### Linux

お使いのアーキテクチャに対応した `.deb` をインストールしてください：

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

これにより、`chloros-cli` が `/usr/bin/chloros-cli` （すでに `PATH` まで更新済み）およびバックエンドを `/usr/lib/chloros/chloros-backend` まで更新するとともに、LATTICE カメラに必要な Arena SDK ランタイムもインストールされます。 詳細については、[Linux のインストール](linux/linux-installation.md)を参照してください。

### 確認

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## ログインとライセンス

CLI（および Python、SDK）へのアクセスには、**有料の Chloros+ プラン**— 有料プランであればどのプランでも利用可能ですが、無料プランでは利用できません。この制限は、CLIバイナリではなく、バックエンドによって**サーバー側**で強制されます： ログアウト状態での呼び出しは `401 AUTH_REQUIRED` で拒否され、無料プランでのログイン状態の呼び出しは `403 PLAN_UPGRADE_REQUIRED` で拒否されます。これは、呼び出し元が `chloros-cli`、 SDK、あるいは手作りのHTTPクライアントからの呼び出しであるかに関わらず、同様の処理が行われます。[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)でアップグレードしてください。**1台のマシンにつき1回**ログインしてください：

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->
{% hint style="warning" %}
**特殊文字を含むパスワード**（`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$`はシェルによって文字化けします（CLIは401エラーでこれを検出し自動再試行しますが、単一引用符を使用すればこの問題を完全に回避できます））。
{% endhint %}

セッションは `~/.chloros/user_session.json` にキャッシュされ、プランの猶予期間中（月額プランの場合は 30 日間、年間プランの場合は有効期限まで）オフラインでも引き続き機能します。 `chloros-cli status`は有料プランがなくても動作するため、拒否の理由は常に確認可能です。

{% hint style="danger" %}
**ヘッドレス作業をスケジュールしますか？ まずログインしてください。**バックエンドを起動するコマンド（`process`、`status`、 `export-status`、…）を**キャッシュされたセッションなし**で実行しても、すぐに失敗とはならず、stdin 上で対話型の `Email:` / `Password:` プロンプトが表示されます。 そのため、無人実行のcronジョブやCIステップでは、**入力を待つ状態でハング**してしまいます。何かをスケジュールする前に、そのマシン上で一度`chloros-cli login EMAIL 'PASSWORD'`を実行してください。
{% endhint %}

***

## 最初の処理実行

`process`をキャプチャファイルのフォルダに指定してください。これにより、Survey3（`.raw` + `.jpg`）、 LATTICE（`.tif`/`.tiff`）、`.dng`、あるいはそれらの組み合わせを自動的に検出します：

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

進行状況はパイプラインの各スレッド（検出、分析、処理、エクスポート）ごとにリアルタイムでストリーミングされ、実行が成功すると、書き込まれた画像プロダクトの数が報告されて終了します（`Image products written: N`）。

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### 出力先の場所

`process`は、入力フォルダではなく**プロジェクトフォルダ**に書き込みを行います：

* `-o` を指定しない場合：プロジェクトはデフォルトのプロジェクトフォルダ（GUI と共有）の下に作成されます（管理には `get-project-folder` / `set-project-folder`、フォールバックは `~/Chloros Projects`）の下に作成され、省略された場合は `-n/--project-name` またはタイムスタンプ（`YYYYMMDD_HHMMSS`）で命名されます。
* `-o PATH` を使用する場合：そのフォルダが **プロジェクトフォルダ** となります。 そのフォルダにすでに `project.json` が存在する場合、上書きするのではなく、`_1`/`_2`… という接尾辞が付いた同階層のフォルダが作成されます。

プロジェクト内では、製品は**カメラ単位、次にファイル形式単位**でグループ化されます：

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

カメラフォルダは、LATTICEの場合は`LATT-<sensor>-<lens>-F<filter>`（キャプチャのEXIF情報`Model`と一致）、`_2`の場合は`<model>_<filter>` （例：`Survey3N_RGN`）となります。 フォーマットフォルダは、`--format`に続き、`tiff16`、`tiff8`、`png8`、 `jpg8`、あるいは `TIFF (32-bit, Percent)` の場合は `tiff32` となります。

{% hint style="info" %}
**エクスポートされたすべての製品は、SOURCEファイルの名前を保持します。**`capture_..._raw.tif`のラディアンスエクスポートも、依然として`capture_..._raw.tif`という名前になります — 単に`tiff32/Radiance_Images/`内に存在するだけです。**ファイル名ではなくフォルダによって成果物が識別されます**。そのため、`*radiance*` という拡張子ではなく、ディレクトリ全体をグロブで指定してください。
{% endhint %}

### 実際に使用するオプション

| フラグ | デフォルト | 機能 |
| --- | --- | --- |
| `-o, --output PATH` | デフォルトのプロジェクトフォルダ | プロジェクトフォルダの場所（上記参照）。 |
| `-n, --project-name NAME` | タイムスタンプ | プロジェクト名。 |
| `--format FMT` | `TIFF (16-bit)` | `TIFF (16-bit)`、 `TIFF (32-bit, Percent)`、`PNG (8-bit)`、`JPG (8-bit)`のいずれか。 |
| `--indices NAME [NAME ...]` | なし | エクスポートする植生指数（[植生指数](#vegetation-indices)を参照）。 |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = ニューラルデベイヤー、処理速度は遅い、最高品質（Chloros+、NVIDIA GPU）。 |
| `--vignette / --no-vignette` | オン | ヴィネット補正。 |
| `--reflectance / --no-reflectance` | オン | 反射率キャリブレーション。LATTICE の場合、これは反射率プロダクトのオン/オフ切り替えでもあります。 |
| `--input-level {auto,raw,debayered,processed}` | `auto` | LATTICE TIFF 用のパイプラインエントリポイントを強制する。 |

その他すべて（ターゲット検出の調整、PPK、露出ピン、アレイ位置合わせフラグなど）については、[CLI リファレンスの `process` セクション](reference/cli-reference.md)の[`process`セクション]を参照してください。

***

## エクスポート対象の選択 (LATTICE製品)

LATTICEの処理は、**1回のパスで対象となるすべての製品**に展開されます。製品ごとの4つのトグルはすべて**デフォルトでON**になっています。1つを無効にするには、`--no-`フォームを使用してください：

| トグル | 製品 |
| --- | --- |
| `--debayered` | リニアデモザイク → `Debayered_Images/` |
| `--preview` | プレビュー表示（ホワイトバランス + ガンマ；マルチスペクトル用の偽色ストレッチ） → `Preview_Images/` |
| `--radiance` | float32 放射輝度、W/m²/sr/nm → `Radiance_Images/`（常に `tiff32/`） |
| `--reflectance` | uint16 反射率、Pix4D対応 → `Reflectance_Calibrated_Images/` |

RGB マスターカメラは常にデベイヤー処理済み + プレビューのみを出力する — 広帯域センサーではバンドごとの放射輝度/反射率は意味をなさないため、これらのトグルはそれらに対してはノーオペとなる。 Survey3 `.raw`はトグル設定を無視し、標準の反射率／ターゲットパスに従います。

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`**（デフォルトは `auto`）は反射率基準を選択します。`auto`は、QAに合格したフレーム内[キャリブレーションターゲット](calibration-targets.md) を絶対基準として選択し、ターゲットが存在しない場合はDAQの光センサーによるダウンウェル分割（ρ = π·L/E）にフォールバックします。`target`は厳格な設定（DAQによる置換なし）です。 `daq`はDAQを優先する。単位ごとの測定ターゲットスキャンは、`--target-reflectance-dir`で指定可能。

{% hint style="info" %}
**反射率ピクセルの読み取り：**DNがρ = 1.0を意味するのは**ソースごと** — LATTICE ファイルは XMP に `Chloros:PixelScale=32768` を付与します。Survey3 ファイルは 65535 を使用し（`Chloros:*` タグは含みません）。 定数であると仮定するのではなく、タグを読み取り、それを基に計算を行ってください。詳細および意図的に設定されたスケールなしの例外ケースについては、[CLI リファレンス](reference/cli-reference.md)を参照してください。
{% endhint %}

**処理は常に `raw` から開始されます。** 派生製品（デベイヤー処理済み／放射輝度／反射率のエクスポートデータ）は、パイプラインに再投入されることはありません。これらを再インポートして処理すると、キャリブレーション計算が二重に適用されてしまうため、Chlorosではこれらをスキップし、その旨を明記しています。 `--input-level`は、本当にエントリポイントを強制する必要がある場合の意図的な「逃げ道」です。***

## 実行が失敗した場合

バージョン 1.2.0 以降、`process` は、何も表示せずに「成功」するのではなく、明確に失敗を通知します:

* **製品を要求したものの、何も書き出さなかった**実行（`project.json`および`calibration_data.json`のみ）は、`Processing finished but wrote no image products.`を出力し、**ゼロ以外の終了コードで終了**するため、 そのため、スクリプトでこれを検出できます。一般的な原因としては、入力フォルダがキャプチャとして認識されなかった場合（レイアウトと `--input-level` を確認してください）、または要求されたすべてのプロダクトがそれらのカメラには適用不可能だった場合（例：RGB 専用のカメラに対して放射輝度／反射率を要求しているなど）。
* **意図的なメタデータのみの実行**（すべてのプロダクトをオフにし、`--indices`を使用しない）は依然として成功とみなされます。この場合、空の画像が出力されるのが正しい結果です。
* `--verbose` を使用して再実行し、バックエンドのログで `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` の行を確認してください。これらはカメラごとのスキップ理由を説明しています。

終了コード：`0`：成功 · `1`：一般的なエラー · `2`：引数エラー · `130`：Ctrl+Cにより中断。

***

## 植生指数

`--indices` に 1 つ以上のプリセット名を指定して実行すると、各指数はそれぞれ独自の `<INDEX>_Index_Images/` フォルダに保存されます：

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

`process --indices` が受け付ける 22 のプリセット名：

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**3つのインデックスリストが存在します。混同しないようにしてください。**GUIの「プロジェクト設定」ドロップダウンには27個の式があります（`FCI1`、`FCI2`、`GARI`、 `GEMI`、`LCI` — これら5つはGUI専用であり、`--indices`には**適用されません**）。 ライブ／オフラインの `lattice index --preset` コマンドは、独自の 22 個のプリセットリストを使用します。計算式およびバンド計算については、[マルチスペクトルインデックス計算式](project-settings/multispectral-index-formulas.md) に記載されています。
{% endhint %}

***

## DAQ 光センサー：概要

`daq pool-*` ファミリーは、バックエンドの永続プール（GUI、CLI、および SDK）を通じて、MAPIR DAQ 分光センサー（USB 経由の DAQ-U、 BLE経由のDAQ-M、イーサネット経由のDAQ-E）を、バックエンドの永続プールを通じて駆動します。GUI、CLI、およびSDKはすべて、1つのライブハンドルを共有しています。 **`pool-*`は、出荷時のCLIでサポートされているDAQパスです**。 参照されることがある他の `daq` サブコマンドは、MAPIR 内部のソース専用サーフェスであり、`pool-*` を指し示す明示的なエラーを伴って終了します。

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` には `--duration` が存在せず、`pool-record --stop` まで実行されます。 デフォルトの出力ディレクトリは **バックエンドのマシン上** の `~/Documents/DAQ Live View/` です。 キャップ補正プロファイルは接続時に選択され（`--cap-id`、バックエンドのデフォルトは `sunshine_cosine`）、実行中に `pool-set-cap` — キャッププロファイルおよびセンサーの校正範囲については、本マニュアルのDAQの章で説明しています。

{% hint style="warning" %}
**マルチNICホスト上のDAQ-E：** 起動後の最初の`pool-connect --eth`による自動検出は、センサーが正常であっても失敗する場合があります。`--eth-host <ip-or-hostname>`は信頼性の高い形式です。検出が失敗した場合は、常にこちらを使用してください。
{% endhint %}

***

## LATTICE カメラ、PTP およびプロジェクトの自動化

`lattice` ファミリー（45 以上のサブコマンド）は、LATTICE カメラの作業をエンドツーエンドでカバーしています。具体的には、検出、単発撮影、GUI のスマートプレップ接続フローによる永続的な同期アレイ、ブラウザでのライブプレビュー、位置合わせ、インデックス演算、およびホスト NIC の診断などです。 一例：

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

これと併せて：`chloros-cli time-sync`は、Chlorosホストが実行するPTPグランドマスターに関するレポートを出力します（LATTICEカメラおよびDAQ-Eセンサーは、デバイス間のタイムスタンプ同期のためにこのグランドマスターにスレーブ接続されます）。 また、`chloros-cli project`は保存済みのChlorosプロジェクトを開き、スクリプト化されたYAMLキャプチャレシピを含め、ヘッドレスでカメラ、アレイ、センサーを制御します。

これら3つのファミリー（`lattice`、`project`、 `daq pool-*`）は、**リモート**バックエンドを制御するための `CHLOROS_BACKEND_URL` をサポートする唯一のファミリーでもあります。コアコマンドは常にローカルマシンを対象とします。

詳細な手順は、本マニュアルの「LATTICE」の章に記載されています。すべてのフラグについては、[CLI リファレンス](reference/cli-reference.md)を参照してください。

***

## トラブルシューティング：トップ5

| 症状 | 対処法 |
| --- | --- |
| `Login required` またはスケジュールされたジョブが `Email:` プロンプトでハングする | このマシンで `chloros-cli login EMAIL 'PASSWORD'` を一度実行してください。キャッシュされたセッションがないコマンドは、即座に失敗するのではなく、対話モードで処理されます。 |
| `backend unreachable` | Chloros デスクトップアプリを起動するか、バックエンドバイナリを直接実行してください（`chloros-backend`）。 `lattice`/`project`/`daq pool-*` をリモートバックエンドに指定する場合は、`CHLOROS_BACKEND_URL` を確認してください。 |
| 配列接続がブロックされました: `FRAMES WILL DROP` / `Reduce ROI to enable` | ホストのNIC受信リングがデフォルト設定にリセットされました。これは、以前は正常に動作していたリグが接続を拒否する主な原因であり、通常はNICドライバの更新後に発生します。 **管理者権限**のあるターミナルから `chloros-cli lattice network --fix` を実行してください（または `ReceiveBufferLen=256`、`PendingReceives=64` を設定してください）。リファレンスの *ホスト NIC の設定と調整* を参照してください。 |
| `daq` サブコマンドが終了します：「完全な DAQ パッケージが必要です…」 | 出荷版ビルドでは想定される動作です。コンパイル済みの CLI には、接続、ストリーム、記録、およびキャプチャ選択をカバーする `daq pool-*` ファミリーのみが含まれています。 `pool-*`（または Python に含まれる `chloros_sdk.connect_daq_sensor()`）を使用してください。 |
| Jetsonは大きなフォルダの前にスワップ警告を表示します | ファイルベースのスワップを追加 — CLIは、実行すべき正確な`fallocate`/`swapon`コマンドを出力します。 |

***

## ヘルプの表示

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **すべてのフラグ、すべてのサブコマンド：** [CLI リファレンス](reference/cli-reference.md)
* **Python 相当：** [Python SDK](api-python-sdk.md) および [SDK リファレンス](reference/sdk-reference.md)
* **サポート:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
