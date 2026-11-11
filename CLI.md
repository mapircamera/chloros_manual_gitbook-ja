# CLI：コマンドライン

> **完全なリファレンス：**[CLIリファレンス](reference/cli-reference.md) には、**すべてのサブコマンドのすべてのフラグ**が記載されており、AIアシスタント向けに最適化されています。URLをアシスタントに貼り付けて、動作するコマンドを尋ねてみてください：`https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **AIツール向けのヒント：** このマニュアルのどのページも、URLの末尾に`.md`を追加することで（例：`https://mapir.gitbook.io/chloros/reference/cli-reference.md`）、生のMarkdown形式で利用可能です。 また、`https://mapir.gitbook.io/chloros/llms.txt`は、LLMが利用できるようマニュアル全体をインデックス化します。

> **アシスタントにChloros自体を実行させたいですか？** `chloros-mcp`は、Python SDKがインストールされたMCPサーバーであり、Claude Desktop、Claude Code、Cursor、その他のMCPクライアントと連携します：[AIアシスタントでのChlorosの使用方法](ai-assistants.md)を参照してください。

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## CLI
とは

`chloros-cli`は、Chloros
デスクトップアプリが使用するのと同じ処理エンジンのコマンドラインフロントエンドです。 これは、Chloros
バックエンド（`127.0.0.1:5000`上のローカルサーバー）に対するシンHTTP
クライアントです。ほとんどのコマンドはバックエンドを自動的に起動するため、スクリプトでは`chloros-cli process …`を1回呼び出すだけで済みます。 その後、バックエンドは次のコマンドが実行されるまで稼働し続けます： `chloros-cli logout`で停止します（アプリが開かれている場合、または実行やビルドが進行中の場合は除く）、`chloros-cli backend stop`で停止するか、クライアント、実行、または接続されたデバイスが30分間ない場合は自動的に停止します。 `chloros-cli backend status` コマンドでこれを確認できます。

**Windows
10/11 (x64)**および**Linux
(x86_64、JetPack 6 上の NVIDIA Jetson arm64)** で、GUI を必要とせず、任意のターミナルから実行可能です。 以下のコマンドでインストールが正常に行われたか確認してください：

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

コマンド群の概要：

* **処理とアカウント** — `process`、`login`、`logout`、`status`、 `export-status`、`language`（38言語 — [対応言語](supported-languages.md)を参照）、 `set-project-folder` / `get-project-folder` / `reset-project-folder`、`selftest`、`update` （Linux
／Jetsonのみ）
* **実機** — `lattice` (LATTICE カメラ制御、45 以上のサブコマンド)、`daq pool-*` (DAQ 光センサー)、`time-sync` (PTP)
* **自動化** — `project` (YAMLキャプチャレシピを含む、保存済みのChloros
プロジェクトをヘッドレスで実行)

知っておくべきグローバルオプション：`--port N`（バックエンドポート、デフォルトは `5000`）、`-v/--verbose`、 `--restart`（バックエンドの強制再起動）、`--backend-exe PATH`。完全な一覧については、[CLI
リファレンス](reference/cli-reference.md)を参照してください。

***

## インストール

CLI
は、すべてのプラットフォームにおいて**Chloros
インストーラ内に同梱**されています。CLI
を個別にダウンロードする必要はありません。インストーラは[ダウンロード](download.md)ページから入手してください。

### Windows


インストーラーは、CLI
を以下の場所に配置します：

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

に配置し、そのフォルダをシステムパス `PATH` に一度追加します（アップグレード時には、以前のインストーラーが残した重複エントリも削除されます）。インストール後は、更新された `PATH` が認識されるよう、**新しいターミナルを開いてください**。 また、インストーラーはインストールルートにランチャースクリプト（`Chloros_CLI.bat` / `Chloros_CLI.ps1`）を配置し、さらに**Chloros
CLI
** スタートメニューのショートカットも配置されます。これらを実行すると、`chloros-cli`が使用可能な状態でターミナルが開きます。

### Linux


お使いのアーキテクチャに対応した `.deb` を、`apt` を使用してインストールしてください（Python
SDK
の推奨要件をスキップする `dpkg -i` ではありません）：

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

これにより、`chloros-cli`から`/usr/bin/chloros-cli` （`PATH`には既に含まれています）、`/usr/lib/chloros/backend/`および`/usr/lib/chloros/cli/`に展開されたバックエンドおよびCLI
プログラム （`/usr/lib/chloros/chloros-backend`はバックエンドを起動するランチャーです）、LATTICEカメラ用にバックエンドが必要とするArena SDK
ランタイム、およびシステム用Python
SDK
である`python3`がインストールされます。 詳細については、[Linux
へのインストール](linux/linux-installation.md)を参照してください。

### 確認

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## ログインとライセンス

CLI
（および Python
SDK
）へのアクセスには、**有料の Chloros
+ プラン** が必要です。有料プランであればどのティアでも利用可能ですが、無料ティアでは利用できません。この制限は、CLI
バイナリではなく、バックエンドによって **サーバー側** で強制されます。ログアウト状態での呼び出しは `401 AUTH_REQUIRED` で拒否され、 また、無料プランでのログイン済み状態での呼び出しは、`chloros-cli`、SDK
、あるいは手作りのHTTP
クライアントのいずれから行われた場合でも、`403 PLAN_UPGRADE_REQUIRED`というエラーで拒否されます。 [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) でアップグレードしてください。

**1 台のマシンにつき 1 回** ログインしてください：

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**特殊文字を含むパスワード**（`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$`はシェルによって文字化けします（CLI
は401エラーでこれを検出し自動再試行しますが、一重引用符を使用すればこの問題は完全に回避できます）。
{% endhint %}

セッションは `~/.chloros/user_session.json` にキャッシュされ、プランの猶予期間中（月額プランの場合は30日間、年間プランの場合は有効期限まで）オフラインでも引き続き利用可能です。 `chloros-cli status`は有料プランがなくても動作するため、拒否の理由は常に確認可能です。

{% hint style="danger" %}
**ヘッドレス作業をスケジュールしますか？ まずログインしてください。**バックエンドを起動するコマンド（`process`、`status`、 `export-status`、…）を**キャッシュされたセッションなし**で実行すると、即座に失敗せず、stdin 上で対話型の `Email:` / `Password:` プロンプトが表示されます。 そのため、無人実行のcronジョブやCIステップでは、**入力を待ち続けてハング**してしまいます。何かをスケジュールする前に、そのマシン上で一度`chloros-cli login EMAIL 'PASSWORD'`を実行してください。
{% endhint %}

***

## 最初の処理実行

`process` をキャプチャデータのフォルダに向ける — Survey3
（`.raw` + `.jpg`）、 LATTICE（`.tif`/`.tiff`）、`.dng`、またはそれらの組み合わせを自動的に検出します：

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

パイプラインの各スレッド（検出、分析、処理、エクスポート）ごとに進行状況がリアルタイムでストリーミングされ、実行が成功すると、書き込まれた画像プロダクトの数が報告されて終了します： `Image products written: N`。さらに、実行時に [クランプマスク](output-image-formats.md#clamp-masks) も書き出した場合は、その後に `(plus M clamp masks)` が続きます（これらは成果物としてカウントされません）。



Chloros<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->
バックエンドがすでに実行中の場合（例：デスクトップアプリ用）、`process`はそれを利用し、そのプロジェクトを開きます。アプリがプロジェクトを開いている間は処理を拒否します。アプリから処理を実行するか、そのプロジェクトを閉じてください。 以前の実行がまだ完了していない場合、最大 15 秒間待機してから拒否します。`--restart` は新しいバックエンドを強制的に起動します。 Linux
では、systemd サービスとして実行されているバックエンドは決して再起動されません。`process` はそのバックエンドをそのまま使用し、通常であれば再起動されるはずの場面（`--restart`、または何も見つからなかったインポートなど）では、 `sudo systemctl restart <unit>` を出力して終了します（`1`）。

### 出力の保存先

`process` は、入力フォルダではなく **プロジェクトフォルダ** に書き込みます：

* `-o` を指定しない場合：プロジェクトはデフォルトのプロジェクトフォルダ（GUI と共有。`get-project-folder` / `set-project-folder` で管理。 設定されていない場合は、ユーザーフォルダ内の`Documents/MAPIR/Chloros Projects`）に作成されます。ファイル名は、`-n/--project-name`で指定するか、省略した場合はタイムスタンプ（`YYYYMMDD_HHMMSS`）が使用されます。
* `-o PATH` を使用する場合：そのフォルダが **プロジェクトフォルダ** となります。そのフォルダにすでに `project.json` が存在する場合、上書きされる代わりに、`_1`/`_2`…という接尾辞が付いた同階層のファイルが作成され、上書きは行われません。相対パス `PATH` は、`chloros-cli` を実行したフォルダを基準として解釈されます。

{% hint style="info" %}
**相対パス（1.3.0で変更）。** バックエンドが開くすべてのパス — 任意のコマンド（キャプチャコマンドのデフォルトである `output` を含む）における `-o`/`--output`、 `--input`、`--burst-dir`、`--profile`、`--save`、および`--mask` — は、`chloros-cli` を実行したフォルダを基準に解決されるため、`-o captures/` は `<current folder>/captures/` に書き込みを行います。 バージョン 1.2.0 では、バックエンドは自身のインストールフォルダ内でパスを解決していましたが、これにより「アクセスが拒否されました」というエラーが発生していました。リモートバックエンド（`CHLOROS_BACKEND_URL`）に対しては、パスは入力されたまま送信され、そのマシン上で解決されます。
{% endhint %}

{% hint style="warning" %}
**Windows
: 長いパスが有効になっていない限り、プロジェクトフォルダの名前は短くしてください。** Windows
で長いパスが無効になっている場合（Windows
のデフォルト設定）、ファイルパスは 260 文字を超えてはならず、製品はプロジェクトフォルダの下に最大約 130 文字まで配置されます。 `process` は開始前にこれを確認します。プロジェクトフォルダの階層が深すぎる場合は、**「このコンピュータではプロジェクトフォルダの階層が深すぎます」**というメッセージが表示され、収まる最長のプロジェクトフォルダ名と、長いパスの有効化方法が提示されます。この段階では何も処理されません。 より短い `-o`（または `set-project-folder`）を使用するか、Windows
の長いパスを有効にし（インストーラの**「Windows
の長いパスを有効にする」**オプション、デフォルトでチェック済み）、Chloros
を再起動してください。
{% endhint %}

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

LATTICE のカメラフォルダは `LATT-<sensor>-<lens>-F<filter>`（キャプチャの EXIF `Model` と一致）であり、Survey3 の場合は `<model>_<filter>` （例：`Survey3N_RGN`）がSurvey3
用となります。フォーマットフォルダは、`--format`に続き、`tiff16`、`tiff8`、 `png8`、`jpg8`、または`tiff32`（`TIFF (32-bit, Percent)`の場合）。

マルチスペクトル M3C カメラの場合、フォーマットフォルダ内には、プロダクトフォルダ（例：`tiff16/clamp_masks/Reflectance_Calibrated_Images/` や `tiff32/clamp_masks/Radiance_Images/`）のほかに、`clamp_masks/` フォルダも表示されることがあります。 このフォルダには、バンドのアンミックス処理によって負の値がゼロにクリップされた放射輝度および反射率プロダクトのピクセルごとのマスクが格納されています — [クリップマスク](output-image-formats.md#clamp-masks) を参照してください。プロダクトフォルダにマスクが含まれることはありません。

{% hint style="info" %}
**エクスポートされたすべてのプロダクトは、SOURCEファイルの名前を保持します。**`capture_..._raw.tif`の放射輝度エクスポートは、依然として`capture_..._raw.tif`という名前ですが、`tiff32/Radiance_Images/`内に存在します。**フォルダがプロダクトを識別するものであり、ファイル名ではありません**。したがって、`*radiance*`というサフィックスではなく、ディレクトリ全体を対象にグロブ検索を行ってください。
{% endhint %}

### 実際に使用するオプション

| フラグ | デフォルト | 機能 |
| --- | --- | --- |
| `-o, --output PATH` | デフォルトのプロジェクトフォルダ | プロジェクトフォルダの場所（上記参照）。 |
| `-n, --project-name NAME` | タイムスタンプ | プロジェクト名。 |
| `--format FMT` | `TIFF (16-bit)` | `TIFF (16-bit)`、 `TIFF (8-bit)`、`TIFF (32-bit, Percent)`、`PNG (8-bit)`、`JPG (8-bit)`のいずれか。 Chloros
はアップスケーリングを行いません：すべての8ビット画像において、より深い形式のデータは`TIFF (8-bit)`として書き込まれ、その旨が一度だけ記載されています。 |
| `--indices NAME [NAME ...]` | なし | エクスポートする植生指数（[植生指数](#vegetation-indices)を参照）。 |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = ニューラルデベイヤー、処理速度は遅いが、 最高画質（Chloros
+、CUDA GPUが必要：GPUがない場合は`standard`が使用される；LATTICEによるキャプチャでは常に`standard`が適用されるため、この設定は適用されない）。 |
| `--vignette / --no-vignette` | オン | ヴィネット補正。 |
| `--reflectance / --no-reflectance` | オン | 反射率キャリブレーション；LATTICE の場合、これは反射率積のオン/オフ切り替えも兼ねる。 |
| `--input-level {auto,raw,debayered,processed}` | `auto` | LATTICE TIFF ファイルのパイプラインエントリポイントを強制する。 |

その他すべて（ターゲット検出の調整、PPK、露出ピン、アレイ位置合わせフラグなど）については、[CLI
リファレンスの`process`セクション](reference/cli-reference.md)を参照してください。

***

## エクスポート対象の選択（LATTICE製品）

LATTICE処理は、**1回のパスで該当するすべての製品**に展開されます。製品ごとの4つのトグルはすべて**デフォルトでON**になっています。`--no-`形式を使用して、1つを無効にしてください：

| トグル | 製品 |
| --- | --- |
| `--debayered` | リニアデモザイク → `Debayered_Images/`: キャリブレーションパックのダークオフセットを除去した後、エッジ認識デモザイクを適用。線形補正、フラットフィールド補正、アンミックスは行わない — キャプチャが`debayered`レベルで保存するファイルと同じ。センサーのネイティブ向き（XMPに記録）を維持。 |
| `--preview` | プレビュー表示（ホワイトバランス＋ガンマ；マルチスペクトル用の偽色ストレッチ） → `Preview_Images/` |
| `--radiance` | float32 放射輝度、W/m²/sr/nm → `Radiance_Images/`（常に `tiff32/`） |
| `--reflectance` | uint16 反射率、Pix4D対応 → `Reflectance_Calibrated_Images/` |

RGB
マスターカメラは、常にデベイヤー処理済み＋プレビューデータのみを出力します。広帯域センサーではバンドごとの放射輝度／反射率は意味をなさないため、これらのトグルはこれらのカメラに対してはノーオペレーションとなります。Survey3
の`.raw`は、これらのトグルを無視し、標準の反射率／ターゲットパスに従います。

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`**（デフォルトは `auto`）は、反射率の基準を選択します： `auto`は、QAに合格したフレーム内の[キャリブレーションターゲット](calibration-targets.md)を絶対基準とし、ターゲットが存在しない場合はDAQの光センサーによる下向き放射の分割値 (ρ = π·L/E) にフォールバックします（フラグ付きでQAに合格したターゲットは、測定スコアにおいてDAQと競合します）。`target`は厳格な設定（DAQによる置換なし）です。 `daq`はDAQを優先する。単位ごとの測定ターゲットスキャンは、`--target-reflectance-dir`で指定可能。

{% hint style="info" %}
**反射率ピクセルの読み取り：**ρ = 1.0 を意味する DN は**ソースごと** です — LATTICE ファイルは XMP に `Chloros:PixelScale=32768` を付与します； Survey3
ファイルは65535を使用し、`Chloros:PixelScale`タグは含まれません（ただし、他の`Chloros:*`処理タグは含まれています）。 定数であると仮定するのではなく、タグを読み取り、それに基づいて除算を行ってください。詳細および意図的にスケールが設定されていない唯一の例外ケースについては、[CLI
リファレンス](reference/cli-reference.md)を参照してください。
{% endhint %}

**処理は常に `raw` から開始されます。** 派生製品（デベイヤー処理済み／放射輝度／反射率のエクスポートデータ）は、パイプラインに再投入されることはありません。これらを再インポートして処理すると、キャリブレーション計算が二重に適用されてしまうため、Chloros
はそれらをスキップし、その旨を通知します。 `--input-level`は、本当にエントリポイントを強制する必要がある場合の、意図的な「逃げ道」です。

***

## 実行が失敗した場合

バージョン 1.2.0 以降、`process` は、何も表示せずに「成功」するのではなく、明確に失敗を通知します：

* **製品を要求したにもかかわらず、何も書き出さなかった**実行（`project.json`および`calibration_data.json`のみ）は、`Processing finished but wrote no image products.`を出力し、**ゼロ以外の値で終了**します。 そのため、スクリプトでこれを検出できます。一般的な原因としては、入力フォルダがキャプチャとして認識されなかった場合（レイアウトと`--input-level`を確認してください）、または要求されたすべてのプロダクトがそれらのカメラには適用不可能だった場合（例：RGB
専用カメラから放射輝度/反射率を要求した場合）などが挙げられます。
* **意図的にメタデータのみを出力する実行**（すべてのプロダクトをオフにし、`--indices`を指定しない）は、依然として成功とみなされます。この場合、空の画像が出力されるのが正しい結果です。
* `--verbose`を指定して再実行し、バックエンドログで`[LATTICE-EXPORT]` / `[EXPORT-CHECK]`の行を確認してください。これらはカメラごとのスキップ理由を説明しています。
* 長いパスがオフになっているWindows
では、260文字のパス制限を超えるほど深い階層にあるプロジェクトフォルダは、処理が開始される**前に**拒否され、制限内に収まる最も長いプロジェクトフォルダが使用されます（[出力の保存先](#where-the-outputs-land)を参照）。
* キャプチャコマンド（`lattice capture`、`lattice array-capture`、およびその他のキャプチャファミリー）は、書き込みに失敗したすべてのファイル名を列挙し、`1`で終了します。繰り返しキャプチャは、ドライブが満杯になると停止します。

終了コード：`0` 成功 · `1` 一般的なエラー · `2` 引数エラー · `130` Ctrl+C により中断。

***

## 植生指数

`--indices` に 1 つ以上のプリセット名を指定して実行します。各指数は、それぞれ独自の `<INDEX>_Index_Images/` フォルダに保存されます：

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

`process --indices` が受け付ける 22 のプリセット名：

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

プロジェクトにまだリストされていない名前が追加されます。プリセットではない名前、またはカメラのバンドでは作成できない名前は、その名前を明記した警告とともにスキップされます。 CLI
がプロジェクトの現在のインデックスリストを読み取れない場合、リストを置き換えるのではなく、リストをそのままにして警告を表示します。

{% hint style="warning" %}
**3つのインデックスリストが存在します。混同しないようにしてください。**GUIの「プロジェクト設定」ドロップダウンには27個の式があります（`FCI1`、`FCI2`、`GARI`、 `GEMI`、`LCI` — これら5つはGUI専用であり、`--indices`には**適用されません**）。 ライブ／オフラインの `lattice index --preset` コマンドは、独自の 22 個のプリセットリストを使用します。計算式およびバンド演算については、[マルチスペクトルインデックス計算式](project-settings/multispectral-index-formulas.md) に記載されています。
{% endhint %}

***

## DAQ 光センサー：概要

`daq pool-*` ファミリーは、バックエンドの永続プールを通じて MAPIR
DAQ スペクトルセンサー（USB 接続の DAQ-U、BLE 接続の DAQ-M、イーサネット接続の DAQ-E）を駆動します。GUI、CLI
、SDK
はすべて、1 つのライブハンドルを共有します。 **`pool-*`は、出荷時のCLI
でサポートされているDAQパスです**。他に言及されることがある他の`daq`サブコマンドは、MAPIR
内部のソース専用インターフェースであり、`pool-*`を指し示す明示的なエラーを伴って終了します。

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

`pool-record`（`--duration`を指定しない場合）は、`pool-record --stop`まで実行されます。 デフォルトの出力ディレクトリは、**バックエンドのマシン上**の `~/Documents/DAQ Live View/` です。 キャップ補正プロファイルは接続時に選択され（`--cap-id`、バックエンドのデフォルトは `sunshine_cosine`）、実行中に `pool-set-cap` — キャッププロファイルおよびセンサーの校正範囲については、本マニュアルのDAQの章で解説しています。

{% hint style="warning" %}
**マルチNICホスト上のDAQ-E：** 起動後の最初の`pool-connect --eth`自動検出は、センサーが正常であっても失敗する場合があります。`--eth-host <ip-or-hostname>`は信頼性の高い形式です。検出が失敗した場合は、常にこちらを使用してください。
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

これと併せて：`chloros-cli time-sync`は、Chloros
ホストが実行するPTPグランドマスターに関する情報を報告します（LATTICEカメラおよびDAQ-Eセンサーは、デバイス間のタイムスタンプ同期のためにこのグランドマスターにスレーブとして接続されます）。 また、`chloros-cli project`は保存済みのChloros
プロジェクトを開き、スクリプト化されたYAMLキャプチャレシピを含め、ヘッドレスでカメラ、アレイ、センサーを制御します。

これら3つのファミリー（`lattice`、 `project`、`daq pool-*`）は、**リモート**バックエンドを制御するための `CHLOROS_BACKEND_URL` をサポートする唯一のファミリーでもあります。コアコマンドは常にローカルマシンを対象とします。

詳細な手順は、本マニュアルの「LATTICE」の章に記載されています。すべてのフラグについては、[CLI
リファレンス](reference/cli-reference.md)を参照してください。

***

## トラブルシューティング：トップ5

| 症状 | 対処法 |
| --- | --- |
| `Login required`、またはスケジュールされたジョブが `Email:` プロンプトでハングする | このマシンで `chloros-cli login EMAIL 'PASSWORD'` を一度実行してください。キャッシュされたセッションがないコマンドは、即座に失敗するのではなく、対話モードで実行されます。 |
| `backend unreachable` | Chloros
デスクトップアプリを起動するか、バックエンドを手動で起動してください：Windows
では`"C:\Program Files\Chloros\backend\chloros-backend.exe"`（インストールフォルダの`backend`フォルダ内； `PATH`にはありません）、Linux
上では`chloros-backend`。`chloros-cli backend status`は実行中の有無を示します； `lattice`/`project`/`daq pool-*`は決してバックエンドを起動しません。これらをリモートバックエンドに指定する場合は、`CHLOROS_BACKEND_URL`を確認してください。 |
| 配列接続がブロックされました: `FRAMES WILL DROP` / `Reduce ROI to enable` | ホストのNIC受信リングがデフォルトにリセットされました — 以前は正常に動作していたリグが接続を拒否する最大の原因であり、通常はNICドライバの更新後に発生します。 **管理者権限**のターミナルから `chloros-cli lattice network --fix` を実行してください（または `ReceiveBufferLen=256`、`PendingReceives=64` を設定してください）。リファレンスの *ホスト NIC の設定と調整* を参照してください。 |
| `daq` サブコマンドが終了します：「完全な DAQ パッケージが必要です…」 | `daq pool-*` コマンドを使用してください。これらはバックエンドを介して同じセンサーを制御します：接続、ストリーム、記録、キャプチャ選択、および DAQ-E の場合はネットワーク設定、再起動、ファームウェアの更新（または Python
からの `chloros_sdk.connect_daq_sensor()`）。 |
| Jetsonは大きなフォルダを処理する前にスワップ警告を出力します | ファイルベースのスワップを追加してください — CLI
には、実行すべき正確な`fallocate`/`swapon`コマンドが表示されます。 |

***

## ヘルプの表示

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **すべてのフラグ、すべてのサブコマンド：** [CLI
リファレンス](reference/cli-reference.md)
* **Python
での対応：** [Python
SDK
](api-python-sdk.md) および [SDK Reference](reference/sdk-reference.md)
* **サポート:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
