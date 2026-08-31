# AIアシスタントでのChlorosの使用方法

このマニュアルは、人間と、人間がますます活用するようになっているAIアシスタントという2つの対象読者を想定して作成されています。 各ページには、正確な値、デフォルト設定、およびコピー＆ペースト可能なコマンドが記載されているため、アシスタント（Claude、ChatGPT、Copilot、コーディングエージェントなど）が、初回から正常に動作するChlorosの自動化スクリプトを作成できるようになっています。

Chloros バージョン: **

1.2.0**。CLI/SDK 対応プラットフォーム: Windows 10/11 x64 および Linux (x86_64 / Jetson aarch64)。

## アシスタントに渡すべきもの

| リソース | URL | 用途 |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | 本マニュアルの全ページを網羅した、機械可読な索引。 |
| **CLI リファレンス** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | `chloros-cli` コマンドの完全な概要：すべてのコマンド、フラグ、デフォルト設定、終了コード、および出力フォルダに関するルール。 LLM向けに作成されています。 |
| **SDK リファレンス** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | `chloros_sdk`、Python、APIの完全な概要：クラス、シグネチャ、例外、および実例。 LLM向けに作成。 |
| **任意のページを生のMarkdownとして** | ページ URL の末尾に `.md` を追加 | 例： `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` はページを生のMarkdown形式で返します。コンテキストウィンドウへの貼り付けやエージェントからの取得に最適です。 |

マニュアル内のリンク： [CLI リファレンス](reference/cli-reference.md) · [SDK リファレンス](reference/sdk-reference.md)。

{% hint style="info" %}
これら2つのリファレンスページは独立した内容となっています。いずれか1つを読んだアシスタントであれば、正しいスクリプトを作成するためにマニュアルの他の部分を参照する必要はありません。
{% endhint %}

## プロンプトレシピ

`<placeholders>`をコピーして必要箇所を記入し、アシスタントに貼り付けてください。

### 1. フライトフォルダを NDVI 形式で処理する

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. キャプチャディレクトリのバッチ監視

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. LATTICEアレイの接続とキャプチャ

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. DAQ光センサーのスペクトルを記録する

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
コマンドラインからのDAQスクリプト実行は、常に`daq pool-*`ファミリー（`pool-connect`、`pool-list`、`pool-latest`、 `pool-stream`、`pool-record`、`pool-set-cap`、`pool-disconnect`）を経由します。 アシスタントが考案するその他の `daq` サブコマンドは、出荷版ビルドでは利用できず、エラーで終了します。
{% endhint %}

## AI が作成したスクリプトが Chloros でうまく機能する理由

これらはすべて、Chloros 1.2.0 の実際に検証済みの動作であり、機械によって生成された自動化スクリプトにありがちな失敗パターンを排除しています：

* **面倒な設定作業が不要。**SDKのスマートコネクトヘルパー（`connect_camera`、`connect_array`、`connect_daq_sensor`）および処理エントリポイント （`ChlorosLocal`、`process_folder`）は、**ローカルバックエンドを自動起動**します。 生成されたスクリプトは、GUI を開いたり、手動でサーバーを起動したりする必要はありません。必要なのは、desktop/CLI パッケージがインストールされていることだけです。
* **パイプライン全体が 1 回の呼び出しで実行されます。** `chloros_sdk.process_folder("path", indices=["NDVI"])` は、インポート → キャリブレーション → 反射率測定 → 屈折率のエクスポートをエンドツーエンドで実行します。処理範囲が狭いため、生成されたスクリプトでエラーが発生する可能性が低くなります。
* **出力がゼロの場合、実行は自己診断を行います。** `process()` の実行後、実行の概要が結果に添付され、すべての処理に関するヒント（例： *なぜ* 実行で出力が得られなかったか）も、Python `UserWarning` として再出力されます。そのため、結果の辞書（dict）を一切確認しないスクリプトであっても、診断情報が表示されるのです。
* **CLI は明らかに失敗します。**プロダクトを要求したにもかかわらず何も書き出さなかった `chloros-cli process` の実行は、`Processing finished but wrote no image products.` を出力し、**ゼロ以外の終了コードで終了**するため、シェルスクリプトや CI では単純な終了コードチェックでこれを検出できます。 正常に実行された場合は、`Image products written: N`が報告されます。

アシスタントが知っておくべき不対称な点が1つあります。SDKの`process()`は、生成物がゼロの実行時には意図的に**例外を発生させません**。その代わりに、summary/hintsを通じて報告します。 Python パイプラインが空の実行時に停止する必要がある場合は、サマリーを確認してください（レシピ 2 ではそうしています）。

## 注意事項

* **Chloros+ へのログインが必要です。**CLI および SDK には、**有料** Chloros+ ティアが必要であり、これはサーバー側で強制されます。ログインしていない場合、リクエストは `401 AUTH_REQUIRED` で失敗し、無料ティアでは `403 PLAN_UPGRADE_REQUIRED` となります。 生成されたスクリプトを実行する前に、マシンごとに1回`chloros-cli login`を実行してください。[Chloros+ ログイン](chloros+-login.md)を参照してください。
* **キャプチャコマンドは実ハードウェアを制御します。** `lattice` / `daq` / `project` コマンドおよび SDK セッションオブジェクトは、物理的なカメラやセンサーへの接続、ストリーミング、およびトリガーを行います。 生成されたスクリプトは、最初の実行前に確認し、ハードウェアのそばで監視しながら実行してください。
* **出力を抜き打ちで確認してください。** 結果を公開する前に、出力フォルダといくつかのピクセル値を確認してください。特に、反射率 TIFF ファイルは光源ごとにスケーリングされています。除数を仮定するのではなく、`Chloros:PixelScale` XMP タグ （LATTICE: 32768 = 1.0 反射率; Survey3: 65535）を参照してください。両方のリファレンスページでは、「反射率ピクセルの読み取り」の項でこれが記載されています。
* **生成されたコードを混乱させる小さな落とし穴：**`pool-record`は**バックエンドホストの**ファイルシステムに書き込みを行います（デフォルトは`~/Documents/DAQ Live View/`）。 複数のネットワークインターフェースを持つマシンでは、自動検出よりも `daq pool-connect --eth-host <ip-or-hostname>` を優先してください； また、バックエンドの URL が登場する箇所では、`http://127.0.0.1:5000` を使用してください（`localhost` は絶対に使用しないでください）。
