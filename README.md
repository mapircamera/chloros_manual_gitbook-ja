---
metaLinks: {}
---

# はじめに

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chlorosは、[MAPIR](https://www.mapir.camera)が提供する、画像やその他のセンサーデータを処理するためのソフトウェアアプリケーションです。

***{% hint style="success" %}**Chloros 1.1.0の新機能**: Linuxのネイティブサポート（amd64およびarm64）、NVIDIA Jetsonエッジコンピューティング、ダイナミック・コンピュート・アダプテーション、4スレッド処理パイプライン、新しいCLIコマンドおよびオプション。 完全な変更履歴については、[ダウンロード](download.md)をご覧ください。
{% endhint %}

Chlorosには、以下の3つのアプリケーションモードがあります：

## Chloros: デスクトップGUIアプリケーション

すべての機能を備えた独立したスタンドアロンウィンドウ。_Windows のみ_。

## [Chloros CLI: コマンドラインインターフェース](CLI.md)

コマンドラインによるバッチ処理。自動化、スクリプト作成、ヘッドレス操作に最適です。 **Windows、Linux amd64、および Linux arm64 (NVIDIA Jetson)** で利用可能です。_CLI を利用するには、Chloros 以上のライセンスが必要です。_

## [Chloros API: Python SDK](api-python-sdk.md)

自動化およびカスタムワークフローのためのプログラム用インターフェース。研究パイプライン、既存のアプリケーションとの統合、カスタムツールの構築に最適です。`pip install chloros-sdk`を介して**すべてのプラットフォーム**で利用可能です。 _APIへのアクセスには、Chloros以上のライセンスが必要です。_***

## 対応プラットフォーム

| プラットフォーム | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | はい | はい | はい |
| **Linux amd64 (x86_64)** | いいえ | はい | はい |
| **Linux arm64 (NVIDIA Jetson)** | いいえ | はい | はい |

Linuxのインストール手順については、[Linux &amp; エッジコンピューティング](linux/linux-overview.md)のセクションを参照してください。

***

## Chloros+

Chlorosはほとんどのタスクで無料で利用できますが、さらに多くの機能が必要になる場合もあります。そのような場合、Chloros+の有料ライセンスが役立ちます。 Chloros+ライセンスを取得すると、次のような新機能を利用できるようになります：

* **マルチスレッド処理**：パイプラインを通じて画像を同時に処理することで、大規模なプロジェクトにおける画像処理を大幅に高速化します。
* **GPU (CUDA) アクセラレーション**：最新のGPUメモリを活用し、画像処理パイプラインをさらに高速化します。最適な結果を得るには、4GB以上のVRAMを推奨します。
* **Chloros+**[**CLI**](CLI.md)**アクセス**：コマンドラインから Chloros+ を実行し、自動化やご自身のソフトウェアへの統合が可能です。
* **Chloros+**[**API**](api-python-sdk.md)**アクセス:** Python から Chloros+ を実行してプログラム制御を行い、研究パイプライン、データ分析ワークフロー、およびカスタムアプリケーションとのシームレスな統合を実現します。
* **複数デバイスでの利用**: 各 Chloros+ ライセンスで 2 台以上のデバイスを登録できます。登録されたデバイスは、MAPIR Cloud アカウントで管理してください。Chloros+ ライセンスをアップグレードすることで、より多くのデバイスをサポートできます。
* **高度なテクスチャ認識デベイヤー法：** 高品質なエッジ認識デベイヤーとAI/MLノイズ除去モデルを組み合わせ、デベイヤー処理によるノイズをほぼ完全に除去します。
* **カスタムマルチスペクトル指標式：** Chlorosラスター計算ツールにカスタムマルチスペクトル指標を入力できます。これは、画像処理および画像閲覧サンドボックスの両方で利用可能です。
* **Linux およびエッジコンピューティング:** フィールドおよびエッジ処理向けに、NVIDIA Jetsonを含むLinux x86_64およびARM64プラットフォーム上でChlorosを実行できます。 [Linuxの概要](linux/linux-overview.md)を参照してください。

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ の価格と登録</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
