---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# よくある質問

<details>

<summary>Chloros以外のカメラで撮影した画像をMAPIRで処理することはできますか？</summary>

いいえ、ChlorosではMAPIR製カメラ（Survey3およびLATTICEシリーズ）の画像処理のみをサポートしています。 詳細については、[対応カメラモデル一覧](supported-cameras.md)をご確認ください。MAPIR Cloudでは、他のカメラの画像処理も提供しています。全リストは[こちら](https://mapir.gitbook.io/mapir-cloud/supported-cameras)をご覧ください。

</details>

<details>

<summary>ChlorosはLATTICEカメラに対応していますか？</summary>

はい。Chloros 1.3.0 では、LATTICE M3C および M3M カメラモジュールをエンドツーエンドでサポートしています： **ライブ制御**— GUIの「カメラ」タブ（`chloros-cli lattice`）またはPython SDK を使用して、カメラの検出、接続、プレビュー、キャプチャが可能で、PTPタイムシンクによる同期化されたマルチカメラアレイにも対応しています — および、キャプチャデータの**完全な放射測定処理** （RAW → デベイヤー → 放射輝度 → 反射率 → インデックス）まで対応しています。[対応カメラ一覧](supported-cameras.md)および[LATTICEガイド](lattice/README.md)をご参照ください。

</details>

<details>

<summary>キャリブレーションターゲットなしで、画像の反射率をキャリブレーションすることはできますか？</summary>

**Survey3：** いいえ。ターゲット以外の画像を撮影する際に、その周辺でキャリブレーションターゲットの画像も同時に撮影しておかないと、画像のピクセル値を既知の反射率（％）に関連付けることはできません。 また、MAPIR光センサーのログを含めない場合、周囲の光スペクトルが測定されないため、反射率の結果は正確になりません。

**LATTICE:** はい。反射率は、パネルの代わりにDAQ光センサーで測定された下向き放射照度を基準とすることができます（ρ = π·L/E）。 QAに合格したフレーム内のターゲットが*存在する場合*、デフォルトではそれが絶対基準となります（`--reflectance-source auto`）。ただし、フラグ付きで合格した場合は、ターゲットとDAQチェーンが測定スコアで競合します。 1つの例外：「F988の反射率は、シーン内の反射率パネルを使用して校正されます。このバンドはDAQ光センサーの校正範囲を超えているため、Chlorosは最新のパネルキャプチャを適用し、パネル観測の合間にその値を保持します。」 [キャリブレーションターゲット](calibration-targets.md)を参照してください。

</details>

<details>

<summary>DAQ光センサーは必要ですか？</summary>

放射輝度の場合は不要です：LATTICEの放射輝度エクスポートデータは、各カメラの工場出荷時の放射測定校正に基づいており、DAQセンサーもターゲットも必要ありません。**反射率**の場合は、周囲光の基準値が必要です。これは、DAQ光センサーによる下向き測定値、またはフレーム内の校正ターゲットのいずれかです。 DAQセンサーを使用すれば、**シーン内にパネルを配置することなく**、校正済みの反射率データを生成できます。記録された`.daq`ファイルは、タイムスタンプに基づいて自動的に画像と照合されます。 [キャリブレーションターゲット](calibration-targets.md) および [CLIリファレンス](reference/cli-reference.md) を参照してください。

</details>

<details>

<summary>ChlorosをAIアシスタント（Claude、ChatGPTなど）と一緒に使用できますか？</summary>

はい。このマニュアルおよびCLI/SDKは、そのために設計されています：

* マニュアルの完全な目次は `https://mapir.gitbook.io/chloros/llms.txt` で提供されており、AIアシスタントがすべてのページを検索できるようになっています。
* 各ページの生のマークダウンは、そのページURLの小文字版に `.md` を付加した形式で利用可能です（例：`https://mapir.gitbook.io/chloros/reference/cli-reference.md`）。
* [CLIリファレンス](reference/cli-reference.md)および[SDKリファレンス](reference/sdk-reference.md)は、LLMが利用できるよう作成されています。正確なフラグ、デフォルト値、終了時の動作、およびコピー＆ペースト可能なコマンドが記載されています。

アシスタントをChlorosに設定する方法については、[AIアシスタント](ai-assistants.md)を参照してください。

</details>

<details>

<summary>処理後の出力ファイルはどこに保存されますか？</summary>

出力ファイルはプロジェクトフォルダ内に保存され、カメラごとにグループ化された後、ファイル形式ごとに分類されます：

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **カメラフォルダ** — LATTICE用は`LATT-<sensor>-<lens>-F<filter>`、Survey3用は`<model>_<filter>`（例：`Survey3N_RGN`）
* **フォーマットフォルダ** — `tiff16`、`tiff8`、`png8`、 `jpg8`、または `tiff32`
* **製品フォルダ** — `Reflectance_Calibrated_Images/`、`Debayered_Images/`、 `Preview_Images/`、`Radiance_Images/`（常に `tiff32` の下にある）、`<INDEX>_Index_Images/`
* **`clamp_masks/`** — LATTICEマルチスペクトルのみ：プロダクトフォルダとは別にピクセル単位のマスクが存在します。詳細は以下を参照してください**エクスポートされたファイルはソースファイルの名前を保持します。プロダクトを識別するのはフォルダ名であり、ファイル名の拡張子ではありません。**CLI を使用する場合、`-o` を指定しない限り、プロジェクトフォルダはデフォルトのプロジェクトフォルダ（変更していない限り `Documents/MAPIR/Chloros Projects`）の下に作成されます。 相対パス「`-o`」を指定した場合は、コマンドを実行したフォルダを基準とします。 製品を要求したが何も書き出しなかった `chloros-cli process` の実行は、`Processing finished but wrote no image products.` を出力し、**ゼロ以外の値で終了**するため、スクリプトでこれを検出できます。 [出力画像形式](output-image-formats.md) および [CLI リファレンス](reference/cli-reference.md) を参照してください。

</details>

<details>

<summary>clamp_masks フォルダおよび *_clampmask.tif ファイルとは何ですか？</summary>

バージョン 1.3.0 の新機能です。マルチスペクトル LATTICE M3C カメラの放射輝度および反射率プロダクトでは、3×3 バンドのアンミックス処理が行われます。バンド内で負の値が生成された場合、Chloros はその値をゼロにクリップします。 クリップされたピクセルが 1 つ以上あるプロダクトには、どのピクセルとバンドがクリップされたかを示すシングルバンドの `uint8` マスクが付与されます（ビット 0 = プロダクトの R チャンネル、ビット 1 = G、ビット 2 = B）。 マスクは、出力フォルダ**の隣**にある `clamp_masks/` フォルダに保存されます。例：`tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — に保存され、製品の `Chloros:UnmixClampMask` XMP タグにはそのマスクの名前が記載されます（または `none` と表示されます）。

これらはシーンの画像ではありません。Chloros 独自のインポート機能ではこれらはスキップされ、製品フォルダ内に格納されることは決してないため、製品フォルダをそのまま Pix4D や Metashape に渡すことができます。[クランプマスク](output-image-formats.md#clamp-masks) を参照してください。

</details>

<details>

<summary>なぜChlorosは「このコンピュータではプロジェクトフォルダの階層が深すぎます」と表示するのですか？</summary>

これはWindowsのみの現象です。Windowsでは、長パス機能がデフォルトで無効になっており、ファイルパスは260文字を超えてはなりません。 Chlorosの製品はプロジェクトフォルダから最大約130文字下位に配置されるため、Chlorosは処理前にそれより深いプロジェクトフォルダを拒否します。これは、バージョン1.3.0以前のように実行はされるものの何も書き込まれないという事態を防ぐためです。このメッセージには、条件を満たす最長のプロジェクトフォルダ名が示されます。

以下のいずれかの方法で修正してください：

* より短いプロジェクトフォルダを使用してください（プロジェクト設定の**作業ディレクトリ**、またはCLIの場合は `-o` / `chloros-cli set-project-folder`）。
* Windowsの長いパスを有効にして、Chlorosを再起動してください。Chlorosインストーラの**「Windowsの長いパスを有効にする」**オプション（デフォルトでチェックされています）がこれを行います。チェックを外していた場合は、インストーラを再実行してください。

[処理の開始](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)を参照してください。

</details>

<details>

<summary>Chlorosでの処理前に画像を編集することはできますか？</summary>

いいえ。Chlorosは、入力データが変更されていないことを前提としています。ファイル名を変更しないでください。

</details>

<details>

<summary>MAPIR Survey3カメラを自動露出に設定し、Chlorosで画像を処理することはできますか？</summary>

いいえ。Survey3の画像データセットは、露出が固定（ロック）されている必要があります。したがって、自動シャッタースピードや自動ISOは使用できません。同じカメラモデルのすべての画像は、同一のシャッタースピードとISO（露出）で撮影されている必要があります。

LATTICEカメラにはこの制限はありません。Chlorosがリアルタイムで露出を制御（Smart AE）し、各撮影時に実際に使用された露出とゲインが記録されるため、放射測定パイプラインでこれを考慮します。

</details>

<details>

<summary>Chlorosでオルソモザイク画像を処理または解析することはできますか？</summary>

いいえ。MAPIRカメラの個別画像のみがサポートされており、オルソモザイクマップのような合成画像はサポートされていません。

</details>

<details>

<summary>Chlorosのターゲット検出ステップを高速化するにはどうすればよいですか？</summary>

ファイルブラウザのテーブルで、右側の列にあるターゲット画像を事前に選択しておくと、Chloros はその画像のみからキャリブレーションターゲットを検索するようになり、処理が大幅に高速化されます。

</details>

<details>

<summary><a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>に画像をアップロードする場合、アップロード前にChlorosで処理すべきですか？</summary>

当社のオンライン処理プラットフォーム [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) へのアップロードを予定している場合は、アップロード前に画像を編集しないでください。Cloud 側で同様の処理に加え、さらに高度な処理もすべて行われます。

</details>

<details>

<summary>MAPIRは将来的にX機能をサポートする予定はありますか？MAPIRにX機能が搭載されることを強く望んでいます。</summary>

弊社では、製品に関するフィードバックを常に歓迎しております。製品に問題が見つかった場合や、改善に関するご提案がございましたら、[お問い合わせ](https://www.mapir.camera/community/contact)よりご意見をお寄せください。 当社の研究開発の多くは、お客様の最も大きなニーズに耳を傾けることを指針としています。

</details>

<details>

<summary>ChlorosはLinuxで利用できますか？</summary>

はい！Chloros 1.3.0は、`.deb`パッケージを通じて、Linuxのamd64（x86_64）およびarm64（NVIDIA Jetson JetPack 6）に対応しています。 CLIおよびPython SDKは、Linux上で完全にサポートされており、LATTICEカメラやDAQセンサーのリアルタイム制御も含まれます。Linux用のGUIは用意されていません。すべての操作は[CLI](CLI.md)または[Python SDK](api-python-sdk.md)を介して行われます。 詳細については、[Linuxの概要](linux/linux-overview.md)を参照してください。

</details>

<details>

<summary>ChlorosをNVIDIA Jetsonで実行できますか？</summary>

はい！Chlorosは、JetPack 6を実行するNVIDIA Jetson **Orinファミリー**（Orin Nano、Orin NX、AGX Orin）に対応しています。 旧モデル（Jetson Nano、TX2、Xavier NX）は JetPack 6 を実行できず、現在のパッケージではサポートされていません。Chloros は Jetson モデルを自動的に検出し、処理戦略を最適化します。 Jetson パッケージは JetPack 6 の CUDA ビルド版 PyTorch を使用しているため、処理は Jetson の GPU 上で実行されます。セットアップおよびデプロイの手順については、[NVIDIA Jetson ガイド](linux/nvidia-jetson-guide.md) をご覧ください。

</details>

<details>

<summary>Chlorosは、私のハードウェアに合わせて自動的に最適化してくれますか？</summary>

はい！Chlorosには、CPU、GPU、RAM、および（Jetsonの場合）温度センサーを自動検出する[Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md)機能が搭載されています。これにより、最適な処理戦略が選択されます — 大容量メモリ搭載GPU向けの`GPU_PARALLEL`から、メモリ容量の小さいGPU向けの`GPU_SINGLE`、さらにはPyTorchが使用できるCUDA GPUを搭載していないシステム向けの`CPU_PARALLEL`まで。 手動での設定は不要です。

</details>

<details>

<summary>4スレッド処理パイプラインとは何ですか？</summary>

Chlorosは、Chloros+ユーザー向けに4スレッドのパイプラインアーキテクチャを採用しています： スレッド1（検出）は画像を読み込み、キャリブレーションターゲットを検出します。スレッド2（キャリブレーション）は反射率キャリブレーションを計算します。スレッド3（処理）はGPU加速によるデベイヤー処理とインデックス計算を実行します。スレッド4（エクスポート）は出力ファイルを書き込みます。 スループットを最大化するため、複数の画像を異なるスレッドで同時に処理できます。詳細については、[処理パイプライン](processing-architecture/processing-pipeline.md)を参照してください。

</details>

<details>

<summary>Chlorosのインストール環境で診断を実行するにはどうすればよいですか？</summary>

`selftest` コマンドを使用して、7 段階のスモークテストを実行します。テスト項目は、バージョン、ポートの可用性、バックエンドの起動、API 接続性 (`/api/test`)、システム情報 (`/api/system-info` — GPU/CUDA/PyTorch)、ノイズ除去モデルの有無、および CUDA ＋ ノイズ除去モデルの準備状況：

```bash
chloros-cli selftest
```

これは、LinuxシステムでGPUおよびCUDAの設定を確認する際に特に役立ちます。Jetsonでは、`CUDA: False`が表示される場合、通常はJetPack 6のCUDAおよびcuDNNライブラリが欠落していることを意味します。`sudo apt install nvidia-jetpack`を実行してこれらをインストールしてください。

</details>
