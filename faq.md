---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# よくある質問

<details>

<summary>Chlorosを使用して、MAPIR以外のブランドのカメラの画像を処理することはできますか？</summary>

いいえ、ChlorosはMAPIR製カメラの画像処理のみに対応しています。 詳細については、[対応カメラモデルの一覧](supported-cameras.md)をご覧ください。MAPIR Cloud では、他のカメラの画像処理も提供しています。完全なリストは[こちら](https://mapir.gitbook.io/mapir-cloud/supported-cameras)をご覧ください。

</details>

<details>

<summary>キャリブレーションターゲットなしで、画像の反射率をキャリブレーションすることはできますか？</summary>

いいえ。非ターゲット画像の撮影時にキャリブレーションターゲットの画像も併せて撮影していない場合、画像のピクセル値を既知の反射率（％）に関連付けることはできません。また、MAPIR光センサーのログを含めない場合、周囲光のスペクトルが測定されず、反射率の結果は正確になりません。

</details>

<details>

<summary>Chlorosでの処理前に画像を編集できますか？</summary>

いいえ。Chlorosは、入力データが変更されていないことを前提としています。ファイル名を変更しないでください。

</details>

<details>

<summary>MAPIR および Survey3 カメラを自動露出に設定し、Chloros で画像を処理することはできますか？</summary>

いいえ。Survey3の画像データセットは、露出が固定（ロック）されている必要があるため、自動シャッタースピードや自動ISOは使用できません。同じカメラモデルのすべての画像は、同一のシャッタースピードとISO（露出）でなければなりません。

</details>

<details>

<summary>Chlorosはオルソモザイク画像の処理や分析が可能ですか？</summary>

いいえ。MAPIRでは、オルソモザイク地図のような合成画像ではなく、個々のカメラ画像のみがサポートされています。

</details>

<details>

<summary>Chlorosのターゲット検出ステップを高速化するにはどうすればよいですか？</summary>

ファイルブラウザのテーブルで、右側の列にあるターゲット画像を事前に選択しておくと、Chloros はその画像内のみをキャリブレーションターゲットとして検索するため、処理が大幅に高速化されます。

</details>

<details>

<summary>画像を <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> にアップロードする場合、アップロード前に Chloros で処理すべきですか？</summary>

当社のオンライン処理プラットフォーム [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) へのアップロードを予定している場合は、アップロード前に画像を編集しないでください。Cloud では、同じ処理に加え、さらに高度な処理が行われます。

</details>

<details>

<summary>MAPIRは将来的にX機能をサポートする予定はありますか？MAPIRにX機能が搭載されることを強く希望しています。</summary>

弊社では、製品に関するフィードバックを常時歓迎しております。製品に問題が見つかった場合や、改善に関するご提案がございましたら、[お問い合わせ](https://www.mapir.camera/community/contact)よりご意見をお寄せください。弊社の研究開発（R&amp;D）の多くは、お客様からの最も重要なニーズを反映して進められています。

</details>

<details>

<summary>ChlorosはLinuxで利用可能ですか？</summary>

はい！Chloros 1.1.0は、`.deb`パッケージを介して、Linuxのamd64（x86_64）およびarm64（NVIDIA Jetson JetPack 6）をサポートしています。 CLIおよびPythonは、Linux上で完全にサポートされています。 LinuxにはGUIがありません。すべての操作は[CLI](CLI.md)または[Python SDK](api-python-sdk.md) を通じて行われます。詳細については、[Linux 概要](linux/linux-overview.md) をご覧ください。

</details>

<details>

<summary>NVIDIA Jetson で Chloros を実行できますか？</summary>

はい！Chloros 1.1.0は、JetPack 6を実行するJetson Nano、Orin Nano、Orin NX、AGX Orinを含むNVIDIA Jetsonプラットフォームに対応しています。Chlorosは、お使いのJetsonモデルを自動的に検出し、処理戦略を最適化します。 セットアップおよびデプロイの手順については、[NVIDIA Jetsonガイド](linux/nvidia-jetson-guide.md)をご覧ください。

</details>

<details>

<summary>Chlorosは、私のハードウェアに合わせて自動的に最適化されますか？</summary>

はい！Chloros 1.1.0 には、CPU、GPU、RAM、および（Jetson では）温度センサーを自動検出する [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) が含まれています。 その後、高メモリシステムでは `GPU_PARALLEL`、リソースが限られたデバイスでは `GPU_SINGLE`、NVIDIA GPU を搭載していないシステムでは `CPU_PARALLEL` など、最適な処理戦略が選択されます。手動での設定は不要です。

</details>

<details>

<summary>4スレッド処理パイプラインとは何ですか？</summary>

Chloros 1.1.0は、Chloros以上のユーザー向けに4スレッドのパイプラインアーキテクチャを採用しています： スレッド1（検出）は画像を読み込み、キャリブレーションターゲットを検出します。スレッド2（キャリブレーション）は反射率キャリブレーションを計算します。スレッド3（処理）はGPUアクセラレーションによるデベイヤー処理とインデックス計算を実行します。スレッド4（エクスポート）は出力ファイルを書き込みます。 スループットを最大化するため、複数の画像を異なるスレッドで同時に処理できます。詳細については、[処理パイプライン](processing-architecture/processing-pipeline.md)を参照してください。

</details>

<details>

<summary>Chlorosのインストール環境で診断を実行するにはどうすればよいですか？</summary>

`selftest` コマンドを使用して、バージョン確認、ポートの可用性、バックエンドの起動、API 接続性、システム情報、ノイズ除去モデル、CUDA の可用性を含む 7 つのシステム診断を実行します：

```bash
chloros-cli selftest
```

これは、Linux/Jetsonシステムにおいて、GPUおよびCUDAの設定を確認する際に特に役立ちます。

</details>
