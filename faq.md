---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# よくある質問

<details>

<summary>Chloros を使用して、MAPIR 以外のブランドのカメラから取得した画像を処理することはできますか？</summary>

いいえ、Chlorosは、MAPIRカメラの画像（Survey3およびLATTICEシリーズ）の処理のみをサポートしています。 詳細については、[対応カメラモデル一覧](supported-cameras.md)をご確認ください。 MAPIR Cloud では、他のカメラの処理も提供しています。完全なリストは [こちら](https://mapir.gitbook.io/mapir-cloud/supported-cameras) をご覧ください。

</details>

<details>

<summary>ChlorosはLATTICEカメラに対応していますか？</summary>

はい。Chloros 1.2.0 では、LATTICE M3C および M3M カメラモジュールをエンドツーエンドでサポートしています：**ライブ制御**— GUI の [カメラ] タブから検出、接続、プレビュー、およびキャプチャが可能で、 `chloros-cli lattice`、またはPythonおよびSDKから、PTPタイムシンクによる同期マルチカメラアレイを含め、キャプチャ画像の**完全な放射計処理** （RAW → デベイヤー → 放射輝度 → 反射率 → インデックス）まで。詳細は[対応カメラ一覧](supported-cameras.md)および[LATTICEガイド](lattice/README.md)をご覧ください。

</details>

<details>

<summary>キャリブレーションターゲットなしで、画像の反射率をキャリブレーションすることはできますか？</summary>

**Survey3:** いいえ。ターゲット以外の画像を撮影する際に、その周辺でキャリブレーションターゲットの画像も同時に撮影しておかないと、画像のピクセル値を既知の反射率（％）に関連付けることはできません。 また、MAPIRの光センサーによるログも含まれていない場合、周囲光のスペクトルは測定されず、反射率の結果は正確になりません。**LATTICE:** はい。反射率は、パネルの代わりに DAQ 光センサーによって測定された下向き放射照度を基準とすることができます（ρ = π·L/E）。 QAに合格したフレーム内ターゲットが存在する場合、デフォルトでそれが絶対基準となります（`--reflectance-source auto`）。 1つの例外：「F988の反射率は、シーン内の反射率パネルを使用して校正されます。このバンドはDAQ光センサーの校正範囲を超えているため、Chlorosが適用され、直近のパネル測定値が使用され、パネル観測の間はその値が保持されます。」 [キャリブレーションターゲット](calibration-targets.md)を参照してください。

</details>

<details>

<summary>DAQ光センサーは必要ですか？</summary>

放射輝度の場合は必要ありません。LATTICEの放射輝度データは、各カメラの工場出荷時の放射測定校正に基づいており、DAQセンサーもターゲットも不要です。**反射率**の場合は、周囲光の基準値が必要となります。これは、DAQ光センサーによる下向き測定値、またはフレーム内の校正ターゲットのいずれかです。 DAQセンサーを使用すれば、**シーン内にパネルを配置することなく**、較正済みの反射率データを生成できます。記録された`.daq`ファイルは、タイムスタンプによって画像と自動的に紐付けられます。 [キャリブレーションターゲット](calibration-targets.md) および [CLI リファレンス](reference/cli-reference.md) を参照してください。

</details>

<details>

<summary>ChlorosをAIアシスタント（Claude、ChatGPTなど）で使用できますか？</summary>

はい。このマニュアルおよび CLI/SDK は、AI アシスタントでの利用を想定して作成されています。

* マニュアルの完全な目次は `https://mapir.gitbook.io/chloros/llms.txt` で提供されており、AI アシスタントがすべてのページを検索できるようになっています。
* 各ページの生のマークダウンは、そのページの小文字表記（URL）に `.md` を付加した形式（例：`https://mapir.gitbook.io/chloros/reference/cli-reference.md`）で利用可能です。
* [CLI リファレンス](reference/cli-reference.md) および [SDK リファレンス](reference/sdk-reference.md)は、LLMが利用できるよう作成されています。正確なフラグ、デフォルト値、終了のセマンティクス、およびコピー＆ペースト可能なコマンドが記載されています。

アシスタントを Chloros に設定する方法については、[AI アシスタント](ai-assistants.md) を参照してください。

</details>

<details>

<summary>処理後の出力ファイルはどこに保存されますか？</summary>

出力ファイルはプロジェクトフォルダ内に保存され、カメラごとにグループ化され、さらにファイル形式ごとに分類されます：

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **カメラフォルダ** — LATTICE用は`LATT-<sensor>-<lens>-F<filter>`、Survey3用は`<model>_<filter>`（例：`Survey3N_RGN`）
* **フォーマットフォルダ** — `tiff16`、`tiff8`、`png8`、 `jpg8`、または `tiff32`
* **製品フォルダ** — `Reflectance_Calibrated_Images/`、`Debayered_Images/`、 `Preview_Images/`、`Radiance_Images/`（常に`tiff32`の下に配置）、`<INDEX>_Index_Images/`**エクスポートされたファイルはソースファイルの名前を保持します。製品を識別するのはフォルダ名であり、ファイル名の拡張子ではありません。**CLI を使用する場合、`-o` を指定しない限り、プロジェクトフォルダは入力フォルダの隣に作成されます。 なお、製品を要求したものの何も書き込まなかった `chloros-cli process` の実行では、`Processing finished but wrote no image products.` が出力され、**非ゼロで終了**するため、スクリプトでこれを検出できます。 [出力画像形式](output-image-formats.md) および [CLI リファレンス](reference/cli-reference.md) を参照してください。

</details>

<details>

<summary>Chlorosでの処理前に画像を編集することはできますか？</summary>

いいえ。Chlorosは、入力データが変更されていないことを前提としています。ファイル名を変更しないでください。

</details>

<details>

<summary>MAPIR Survey3カメラを自動露出に設定し、Chlorosで画像を処理することはできますか？</summary>

いいえ。Survey3の画像データセットは、露出が固定／ロックされている必要があります。したがって、自動シャッタースピードや自動ISOは使用できません。同じカメラモデルのすべての画像は、同一のシャッタースピードとISO（露出）で撮影されている必要があります。

LATTICEカメラにはこの制限はありません。Chlorosは露出をリアルタイムで制御（Smart AE）し、各撮影で実際に使用された露出とゲインが記録されるため、放射測定パイプラインでこれを考慮します。

</details>

<details>

<summary>Chloros はオルソモザイク画像の処理や分析を行うことができますか？</summary>

いいえ。MAPIRでは、個々のカメラ画像のみがサポートされており、オルソモザイクマップのような合成画像はサポートされていません。

</details>

<details>

<summary>Chloros のターゲット検出ステップを高速化するにはどうすればよいですか？</summary>

ファイルブラウザのテーブルで、右側の列にあるターゲット画像を事前に選択しておくと、Chloros はその画像内のみからキャリブレーションターゲットを検索するようになり、処理が大幅に高速化されます。

</details>

<details>

<summary>画像を <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> にアップロードする場合、アップロード前に Chloros で処理を行うべきですか？</summary>

当社のオンライン処理プラットフォーム [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) へのアップロードを予定している場合は、アップロード前に画像を編集しないでください。Cloud では、Chloros で行われる処理すべてに加え、さらに高度な処理も実行されます。

</details>

<details>

<summary>MAPIRは、将来的にX機能をサポートする予定はありますか？MAPIRにX機能が搭載されることを強く希望しています。</summary>

弊社では、製品に関するフィードバックを常に歓迎しております。製品に問題が見つかった場合や、改善に関するご提案がございましたら、[お問い合わせ](https://www.mapir.camera/community/contact)よりご意見をお寄せください。 当社の研究開発（R&amp;D）の多くは、お客様の最も切実なニーズに耳を傾けることを指針としています。

</details>

<details>

<summary>ChlorosはLinuxで利用可能ですか？</summary>

はい！Chloros 1.2.0は、Linuxのamd64（x86_64）およびarm64 （NVIDIA Jetson JetPack 6）に対応しています。 CLI および Python SDK は、Linux で完全にサポートされており、LATTICE カメラおよび DAQ センサーのライブ制御も含まれます。 Linux用のGUIはありません — すべての操作は、[CLI](CLI.md) または [Python SDK](api-python-sdk.md) を通じて行われます。詳細については、[Linux 概要](linux/linux-overview.md) を参照してください。

</details>

<details>

<summary>NVIDIA Jetson で Chloros を実行できますか？</summary>

はい！Chlorosは、JetPack 6を実行しているJetson Nano、Orin Nano、Orin NX、AGX Orinを含むNVIDIA Jetsonプラットフォームに対応しています。Chlorosは、お使いのJetsonモデルを自動的に検出し、処理戦略を最適化します。 セットアップおよびデプロイの手順については、[NVIDIA Jetson ガイド](linux/nvidia-jetson-guide.md) をご覧ください。

</details>

<details>

<summary>Chloros は、お使いのハードウェアに合わせて自動的に最適化されますか？</summary>

はい！Chlorosには、CPU、GPU、RAM、および（Jetson では）温度センサーを自動検出する [ダイナミック・コンピュート・アダプテーション](processing-architecture/dynamic-compute-adaptation.md) が搭載されています。 その後、大容量メモリシステムでは `GPU_PARALLEL`、リソースに制約のあるデバイスでは `GPU_SINGLE`、NVIDIA GPU を搭載していないシステムでは `CPU_PARALLEL` など、最適な処理戦略が選択されます。 手動での設定は不要です。

</details>

<details>

<summary>4スレッド処理パイプラインとは何ですか？</summary>

Chlorosは、Chloros+ユーザー向けに4スレッドのパイプラインアーキテクチャを採用しています。 スレッド 1（検出）は画像を読み込み、キャリブレーションターゲットを検出します。スレッド 2（キャリブレーション）は反射率キャリブレーションを計算し、スレッド 3（処理）は GPU アクセラレーションによるデベイヤー処理とインデックス計算を実行し、スレッド 4（エクスポート）は出力ファイルを書き込みます。 スループットを最大化するため、複数の画像を異なるスレッドで同時に処理することができます。詳細については、[処理パイプライン](processing-architecture/processing-pipeline.md)を参照してください。

</details>

<details>

<summary>Chlorosのインストール環境で診断を実行するにはどうすればよいですか？</summary>

`selftest` コマンドを使用して、7 段階のスモークテストを実行します。テスト項目は、バージョン、ポートの可用性、バックエンドの起動、API 接続性（`/api/test`）、システム情報 （`/api/system-info` — GPU/CUDA/PyTorch）、ノイズ除去モデルの有無、および CUDA + ノイズ除去モデルの準備状況：

```bash
chloros-cli selftest
```

これは特に、Linux/Jetson システムにおいて、GPU および CUDA の設定を確認する際に役立ちます。

</details>
