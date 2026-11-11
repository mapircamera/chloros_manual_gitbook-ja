# AIアシスタントとChlorosの併用

このマニュアルは、人間と、人間がますます活用するようになっているAIアシスタントという、2つの対象読者を想定して作成されています。各ページには正確な値、デフォルト設定、コピー＆ペースト可能なコマンドが記載されており、アシスタント（Claude、ChatGPT、Copilot、コーディングエージェントなど）が、初回から正常に動作するChlorosの自動化スクリプトを作成できるようになっています。

Chlorosのバージョン：**

1.3.0**。CLI/SDKプラットフォーム：Windows 10/11 x64 および Linux (x86_64 / Jetson aarch64)。

## アシスタントに何を渡すべきか

| リソース | URL | 用途 |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | このマニュアルの全ページを網羅した、機械可読なインデックス。 |
| **CLIリファレンス** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | `chloros-cli`コマンドの完全な仕様：すべてのコマンド、フラグ、デフォルト値、終了コード、および出力フォルダのルール。LLMが処理できるよう記述されています。 |
| **SDKリファレンス** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | `chloros_sdk`のPython APIの完全な一覧：クラス、シグネチャ、例外、および実用例。LLMが利用できるよう記述されています。 |
| **任意のページを生のMarkdown形式で取得** | ページのURLに「`.md`」を追加 | 例：`https://mapir.gitbook.io/chloros/reference/sdk-reference.md` を実行すると、ページが生のMarkdown形式で返されます。コンテキストウィンドウへの貼り付けや、エージェントからの取得に最適です。 |

マニュアル内のリンク：[CLIリファレンス](reference/cli-reference.md) · [SDKリファレンス](reference/sdk-reference.md)。

{% hint style="info" %}
これら2つのリファレンスページは独立した内容となっています。いずれか一方を読んだアシスタントであれば、正しいスクリプトを作成するためにマニュアルの他の部分を参照する必要はありません。
{% endhint %}

## アシスタントにChlorosを直接実行させる：`chloros-mcp`

上記のすべては、アシスタントにスクリプトやコマンドを*記述*させるものです。 `chloros-mcp` を使用すると、Claude Desktop、Claude Code、Cursor など、[Model Context Protocol](https://modelcontextprotocol.io) (MCP) をサポートするアシスタントが、Chloros 自体を操作します。 平易な言葉で指示すると、アシスタントがサーバー上のツールを呼び出します：

* **画像処理：** プロジェクトを作成し、キャプチャ画像のフォルダをインポートし、設定を選択し、処理を行い、実行結果のエクスポート内容を報告します。
* **ハードウェアの操作：** LATTICEカメラ、アレイ、DAQ光センサーを接続し、プレビュー、露出や色の設定、撮影、記録、ファームウェアの更新を行います。

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

サーバー側のツールは、[Python SDK](api-python-sdk.md)の機能と重複しています。 これらはSDKとともにインストールされるため、通常は他にインストールする必要はありません。Windowsでは、Chlorosインストーラーが`chloros-mcp.exe`をPythonの`Scripts`フォルダに配置し、 Linux版では`.deb`が`chloros-mcp`を`/usr/bin`に配置します。

**Claudeコード：**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**これを `claude_desktop_config.json` に追加し（**[設定] > [開発者] > [設定の編集]**から開けます）、その後 Claude Desktop を再起動してください。**カーソル** については、`~/.cursor/mcp.json` に同じエントリを追加します。

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

クライアントが「サーバーの起動に失敗しました」と表示する場合は、`chloros-mcp`が、クライアントが認識している`PATH`に含まれていない可能性があります： その場合は、代わりにフルパスを指定してください（Windowsの場合は`where chloros-mcp`、Linuxの場合は`which chloros-mcp`）。

同じコンピュータにChlorosがインストールされており、アプリ内または`chloros-cli login`を使用して、有料のChloros+プラン（Copper以上）のいずれかでサインインしている必要があります。サーバーが代わりにサインインすることはできません。

{% hint style="info" %}
各アプリ、オプション、すべてのツールのクライアント設定、および実行時の動作については、SDKリファレンスの [MCPサーバー（AIアシスタント）](reference/sdk-reference.md#mcp-server-ai-assistants) を参照してください。
{% endhint %}

## プロンプトレシピ

`<placeholders>`をコピーして必要事項を記入し、アシスタントに貼り付けてください。

### 1. フライトフォルダをNDVIに変換する

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
コマンドラインからのDAQスクリプト実行は、常に`daq pool-*`ファミリー（`pool-connect`、`pool-list`、`pool-latest`、 `pool-stream`、`pool-record`、`pool-set-cap`、`pool-disconnect`）を経由します。 アシスタントが考案するその他の `daq` サブコマンドは、出荷版では利用できず、エラーで終了します。
{% endhint %}

## AIが作成したスクリプトがChlorosでうまく機能する理由

これらはすべて、Chloros 1.3.0における実証済みの動作です。これらは、機械によって作成された自動化にありがちな従来の失敗パターンを解消します：

* **面倒なセットアップ作業が不要。**SDKのスマートコネクトヘルパー（`connect_camera`、`connect_array`、`connect_daq_sensor`）および処理エントリポイント （`ChlorosLocal`、`process_folder`）は、**ローカルバックエンドを自動起動**します。生成されたスクリプトは、GUIを開いておく必要も、手動でサーバーを起動する必要もありません。必要なのは、デスクトップ版またはCLIパッケージがインストールされていることだけです。
* **パイプライン全体が 1 回の呼び出しで実行されます。** `chloros_sdk.process_folder("path", indices=["NDVI"])` は、インポート → キャリブレーション → 反射率測定 → 屈折率エクスポートをエンドツーエンドで実行します。処理範囲が狭いため、生成されたスクリプトでエラーが発生する可能性が低くなります。
* **出力がゼロの場合、実行は自己診断を行います。** `process()` の実行後、実行の概要が結果に添付され、すべての処理に関するヒント（例： *なぜ* 実行で出力が得られなかったか）も、Pythonスクリプト`UserWarning`として再出力されます。そのため、結果の辞書（dict）を一切確認しないスクリプトであっても、診断情報が表示されるのです。
* **CLIは明確にエラーを通知します。**出力ファイルを要求したにもかかわらず何も書き出さなかった実行では、`chloros-cli process` が表示され、**非ゼロで終了**するため、シェルスクリプトやCIでは単純な終了コードチェックでこれを検出できます。 正常に実行された場合は `Image products written: N` が報告され、マスクが書き込まれた場合は `(plus M clamp masks)` が追加で報告されます。キャプチャコマンドも同様に、書き込みに失敗したすべてのファイル名を列挙し、`1` で終了します。
* **相対パスは、スクリプトが想定する通りです。** ローカルバックエンドの場合、相対出力パス（CLI では `-o out/`、SDK では `output_dir="out/"`）は、コマンドまたはスクリプトが実行されたフォルダを基準に解決されます。

アシスタントが知っておくべき非対称性として、SDKの`process()`は、成果物がゼロのランでは意図的に例外を**発生させない**ようになっています。その代わりに、サマリーやヒントを通じて報告されます。 Pythonパイプラインが空の実行時に停止する必要がある場合は、サマリーを確認してください（レシピ2ではそうしています）。

## 注意事項

* **Chloros+へのログインが必要です。**CLIおよびSDKを利用するには、**有料**のChloros+プランが必要です。これはサーバー側で強制されており、ログインしていない場合は`401 AUTH_REQUIRED`エラーでリクエストが失敗し、無料プランの場合は`403 PLAN_UPGRADE_REQUIRED`エラーが発生します。 生成されたスクリプトを実行する前に、マシンごとに1回、`chloros-cli login`を実行してください。[Chloros+ ログイン](chloros+-login.md)を参照してください。
* **キャプチャコマンドは実際のハードウェアを制御します。** `lattice` / `daq` / `project` コマンド、SDK セッションオブジェクト、および `chloros-mcp` のツールは、物理的なカメラやセンサーへの接続、ストリーミング、およびトリガーを行います。生成されたスクリプトは、初めて実行する前に内容を確認し、ハードウェアのそばで監視しながら実行してください。
* **出力をスポットチェックしてください。** 結果を公開する前に、製品フォルダと数個のピクセル値を確認してください。 特に、反射率 TIFF ファイルは光源ごとにスケーリングされます。`Chloros:PixelScale` の XMP タグを確認してください（LATTICE ファイルにはこの情報が含まれています：32768 = 1.0 の反射率； Survey3ファイルには含まれておらず、65535 = 1.0を使用します）を参照し、除数を仮定しないでください。両方のリファレンスページで、「反射率ピクセルの読み取り」の項にこれが記載されています。
* **生成されたコードを混乱させる小さな落とし穴：**`pool-record`は**バックエンドホストの**ファイルシステムに書き込みを行います（デフォルトは`~/Documents/DAQ Live View/`）； 複数のネットワークインターフェースを持つマシンでは、自動検出よりも`daq pool-connect --eth-host <ip-or-hostname>`を優先してください；バックエンドのURLが登場する箇所では、必ず`http://127.0.0.1:5000`を使用し（`localhost`は絶対に使用しないでください）； プロジェクトの出力ツリーを走査するスクリプトは、`clamp_masks/`フォルダをスキップする必要があります（これらのフォルダ内の`*_clampmask.tif`ファイルはマスクであり、画像ではありません — 参照 [出力画像形式](output-image-formats.md#clamp-masks)を参照)；また、長いパスが無効になっているWindowsでは、プロジェクトフォルダ名を短くしてください。約130文字を超える階層にあるプロジェクトフォルダは、処理開始前に拒否されます。
