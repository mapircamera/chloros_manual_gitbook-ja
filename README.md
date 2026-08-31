---
metaLinks: {}
---

# はじめに

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chlorosは、 [MAPIR](https://www.mapir.camera)が提供するソフトウェアアプリケーションであり、マルチスペクトル画像の処理、MAPIRハードウェアのリアルタイム制御、およびセンサーデータの記録を行います。 Chloros 1.2.0 は、MAPIR 製品ファミリー全体に対応しています：

* **Survey3 カメラ** — RAW+JPG 形式で取得した画像を処理し、校正済みの反射率マップおよび植生指数マップに変換します。[対応カメラ](supported-cameras.md) をご覧ください。
* **LATTICEカメラ** — GigEマルチスペクトルカメラモジュールを、単体または同期されたマルチカメラアレイとしてリアルタイムで接続し、プレビュー、撮影、および校正済みの放射輝度および反射率データへの処理を行います。[LATTICEのセクション](lattice/README.md)を参照してください。
* **DAQ 光センサー** — DAQ-U（USB）、DAQ-M（Bluetooth）、DAQ-E（イーサネット）のスペクトルセンサー：リアルタイムの校正済みスペクトル、`.daq` による記録、および反射率処理のための下向き照度。 詳細は[DAQセクション](daq/README.md)を参照してください。

{% hint style="success" %}
**Chloros 1.2.0の新機能**： LATTICEカメラおよびアレイのリアルタイム制御、DAQと光センサーの統合、キャプチャモードとレコーダー、完全なLATTICE放射測定処理パイプライン、CLI/SDKからのプロジェクト自動化など、多数の新機能が追加されました。 以下の「新機能一覧」をご確認いただき、変更履歴については[ダウンロード](download.md)をご覧ください。
{% endhint %}

{% hint style="info" %}
**AIアシスタントでChlorosをご利用ですか？** このマニュアルは、そのために作成されています。アシスタントに以下のURLを指定してください：

* `https://mapir.gitbook.io/chloros/llms.txt` — すべてのページの機械可読インデックス。
* 生のMarkdown形式の任意のページ — そのURLの末尾に`.md`を追加してください（例：`https://mapir.gitbook.io/chloros/reference/cli-reference.md`）。
* [CLI リファレンス](reference/cli-reference.md) および [SDK リファレンス](reference/sdk-reference.md) — LLMが利用できるよう作成された、完全かつ正確な値が記載されたリファレンスページ。

プロンプト例：*「https://mapir.gitbook.io/chloros/reference/cli-reference.md,を読み込み、~/flights/flight_001 フォルダにログインして、その内容を reflectance + NDVI GeoTIFF 形式で処理するスクリプトを作成してください。」*

完全ガイド：[AIアシスタントでのChlorosの使用方法](ai-assistants.md)。
{% endhint %}

***

## Chloros 1.2.0 の新機能

* **ライブカメラ制御 — 新しい「カメラ」タブ。** LATTICEカメラを1台ずつ、または同期されたマルチカメラアレイ（PTPタイムシンク、ハードウェアトリガーによるキャプチャ）として接続できます。ライブプレビューのオーバーレイ、バンドごとのヒストグラム、スマート自動露出、ライブインデックス計算機能、アプリ内でのカメラファームウェア更新機能を備えています。
* **光センサー — 新しい「光センサー」タブ。** DAQ-U（USB）、DAQ-M（Bluetooth）、DAQ-E（イーサネット）センサーを接続可能。 校正済みのスペクトル（W/m²/nm）をリアルタイムで表示し、`.daq`ファイルをプロジェクトに記録し、キャップ補正プロファイルを選択し、ネットワーク経由でDAQ-Eのファームウェアを更新できます。
* **キャプチャモードとレコーダー。** シングル／連続／インターバルキャプチャに加え、RAW専用「最速キャプチャ」モード；「すべてキャプチャ」が生成するカメラおよびエクスポート形式をプロジェクトごとに選択可能；モニタリンググレードのインデックス動画および分析グレードのRAWバーストに対応したアレイレコーダー（オフライン動画ビルド機能付き）。
* **LATTICE 処理パイプライン。** LATTICE キャプチャフォルダをインポートし、各生データをデベイヤー処理済みの画像、プレビュー、float32 ラディアンス (W/m²/sr/nm)、および反射率の各出力形式に展開します。各出力形式ごとに表示/非表示を切り替えることができます。 反射率は、フレーム内のキャリブレーションターゲットまたはDAQによるダウンウェリングから取得可能です。エクスポートにはアレイアライメントが適用され、工場出荷時のキャリブレーションデータが欠落している場合は、カメラのシリアル番号に基づいて自動的にダウンロードされます。
* **プロジェクトはハードウェア情報を記憶します。** 接続されたカメラや光センサーはプロジェクト（`cameras.json` / `sensors.json`）とともに保存され、プロジェクトを再開する際に保存された設定で再接続されます。 [GUI : プロジェクト](projects.md)を参照してください。
* **画像ビューアの機能強化。** ファイルごとの正しい反射率スケーリングを適用したカーソル位置のピクセル/インデックス読み出し、レイヤーヒストグラム、GSDビニングスライダー、トリガーごと/カメラごとのグリッド表示モード、LATTICE製品ビュー、およびインデックス/LUTのサンドボックスデータをディスクへエクスポートする機能が追加されました。
* **CLI および SDK の機能が大幅に拡張されました。** 新しいコマンドファミリー `lattice`、`daq pool-*`、`project`、および `time-sync`； 新しい `process` オプション（`--input-level`、製品ごとのトグル、`--reflectance-source`、配列アラインメントフラグ）； バックエンドを自動起動する SDK スマートコネクトハンドル（`connect_camera` / `connect_array` / `connect_daq_sensor`）； `open_project()`の自動化；SDKホイールはインストーラに同梱されており、PyPIには`chloros-sdk`として公開されています。
* **正確な失敗セマンティクス。** プロダクトを要求したものの、何も書き出しなかった `chloros-cli process` の実行は、明確に失敗し、非ゼロで終了するようになりました。成功した実行では、書き出されたイメージプロダクトの数が報告されます。
* **新しい出力レイアウト。** 生成されたプロダクトは `<project>/<camera>/<format>/<Product>_Images/` フォルダに保存され、ソースファイル名がそのまま保持されます。プロダクトを識別するのはファイル名のサフィックスではなく、フォルダ名です。[出力画像フォーマット](output-image-formats.md) を参照してください。
* **入力、プラン、言語の拡充。** `.dng` 入力のサポート；38 言語すべてのインターフェースが完全に実装されました；プランごとのハードウェア制限として、無料（ログイン不要）利用で最大 4 台のカメラと 2 つの光センサーが利用可能です。
* **信頼性の向上。** 「処理の停止」操作で正確な実行概要が表示され、処理が正常に終了します。マルチカメラプロジェクトではすべてのカメラデータがエクスポートされ、インストーラーによるアップグレード時にログアウトされることがなくなりました。***

Chlorosは、以下の3つのアプリケーション環境で利用可能です：

## Chloros：デスクトップGUIアプリケーション

「ライブカメラ」および「光センサー」タブを含むすべての機能を備えた、独立したスタンドアロンウィンドウです。_Windows専用_。

## [Chloros CLI: コマンドラインインターフェース](CLI.md)

コマンドラインによるバッチ処理に加え、ライブの `lattice`、`daq pool-*`、`project`、および `time-sync` コマンドを利用できます。 自動化、スクリプト作成、ヘッドレス操作に最適です。 **Windows、Linux amd64、および Linux arm64 (NVIDIA Jetson)** で利用可能です。 _CLI を利用するには、有料の Chloros+ ティアが必要です。_

## [Chloros API: Python SDK](api-python-sdk.md)

自動化およびカスタムワークフローのためのプログラムによる Python インターフェース：フルパイプライン処理、ライブカメラ/アレイセッション、DAQ センサーセッション、および保存済みプロジェクトの自動化。 デスクトップ/CLIパッケージとともにインストールされ、`pip install chloros-sdk`としても公開されています。_APIにアクセスするには、有料のChloros+ティアが必要です。_

***

## 対応プラットフォーム

| プラットフォーム | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | はい | はい | はい |
| **Linux amd64 (x86_64)** | いいえ | はい | はい |
| **Linux arm64 (NVIDIA Jetson)** | いいえ | はい | はい |

Linux のインストール手順については、[Linux およびエッジコンピューティング](linux/linux-overview.md) のセクションを参照してください。

***

## 3つのステップで始める

1. **インストール** — お使いのプラットフォーム用のインストーラーをダウンロードして実行してください。[ダウンロード](download.md)を参照してください。
2. **ログイン（GUIの場合は任意）** — GUIでは、アカウントがなくても無料で画像処理を行うことができます。[Chloros+ ログイン](chloros+-login.md)を行うと、並列処理、 GPU アクセラレーション、デバイス制限の緩和、および CLI/SDK へのアクセスが可能になります。
3. **最初のプロジェクトを作成する** — Chloros を開き、[新規プロジェクト](projects.md)を作成し、[画像を追加](processing-images-gui/adding-files-to-a-project.md)し、[処理を開始](processing-images-gui/starting-the-processing.md)します。 代わりにライブハードウェアを制御するには、「Cameras」または「Light Sensors」タブを開いてください — [GUI : ナビゲーション](navigation.md)を参照してください。***

## Chloros+

Chlorosはほとんどのタスクで無料で利用できますが、さらに機能が必要になる場合もあります。そのような場合、Chloros+の有料ライセンスが役立ちます。 Chloros+ ライセンスを取得すると、次のような新機能を利用できるようになります：

* **マルチスレッド処理**：パイプラインを通じて画像を同時に処理することで、大規模なプロジェクトにおける画像処理を大幅に高速化します。
* **GPU（CUDA）アクセラレーション**：現在の高容量GPUメモリを活用し、画像処理パイプラインをさらに高速化します。最適な結果を得るには、4GB以上のVRAMを推奨します。
* **Chloros+**[**CLI**](CLI.md)**アクセス**：コマンドラインから Chloros+ を実行し、自動化や独自のソフトウェアへの統合を行うことができます。 すべての有料プランで利用可能。サーバー側で強制適用されます。
* **Chloros+**[**API**](api-python-sdk.md)**利用方法：** Python から Chloros+ を実行してプログラムによる制御を行い、研究パイプライン、データ分析ワークフロー、およびカスタムアプリケーションとのシームレスな統合を実現します。すべての有料プランで利用可能。サーバー側で適用されます。
* **ハードウェア制限の緩和**：より多くのカメラと光センサーを同時に接続可能です。ログインなしのGUIでは、最大4台のカメラと2つのDAQ光センサーを接続できますが、有料プランでは両方の上限が引き上げられます：

| プラン | カメラ | DAQ光センサー |
| --- | --- | --- |
| Iron（無料、ログイン不要） | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **複数デバイスの利用**：1つのChloros+ライセンスにつき、2台以上のデバイスを登録できます。登録されたデバイスの管理には、MAPIR Cloudアカウントをご利用ください。 Chloros+ ライセンスをアップグレードすることで、より多くのデバイスをサポートできます。
* **高度なテクスチャ対応デベイヤー手法：** 高品質なエッジ認識デベイヤーと AI/ML ノイズ除去モデルを組み合わせ、デベイヤー処理によるノイズをほぼ完全に除去します。
* **カスタムマルチスペクトル指標式：** Chloros ラスター計算機にカスタムマルチスペクトル指標を入力できます。これは、処理および画像表示サンドボックスの両方で利用可能です。
* **Linux およびエッジコンピューティング：** フィールドおよびエッジ処理向けに、NVIDIA Jetson を含む Linux x86_64 および ARM64 プラットフォーム上で Chloros を実行できます。 [Linuxの概要](linux/linux-overview.md)を参照してください。

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ の価格と登録</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
