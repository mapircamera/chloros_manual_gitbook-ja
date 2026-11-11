# Linuxの概要

Chloros 1.3.0 は、Linux ワークステーション、サーバー、および NVIDIA Jetson エッジデバイス上で、**CLI**および**Python SDK** に対するネイティブな Linux サポートを提供します。これには、ヘッドレスでのマルチスペクトル画像処理に加え、LATTICE カメラおよび DAQ 光センサーのリアルタイム制御が含まれます。

{% hint style="info" %}
**Linux ではデスクトップ GUI は利用できません。**Chloros デスクトップ GUI は Windows 専用です。Linux ユーザーは、[CLI](../CLI.md) および [Python SDK](../api-python-sdk.md) を通じて Chloros を操作します。 `.deb`は、アプリケーションメニューに**Chloros CLI**のエントリを追加しますが、これは単に`chloros-cli`を実行するターミナルエミュレータを開くだけです。
{% endhint %}

***

## プラットフォーム対応マトリックス

| 機能 | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/ --- | --- | --- | --- |
| **デスクトップGUI** | あり | 該当なし | なし | なし |
| **CLI** (`chloros-cli`) | あり | あり | あり | あり |
| **Python SDK** (`chloros-sdk`) | あり | あり | あり | あり |
| **画像処理パイプライン** | あり | あり | あり | あり |
| **LATTICE カメラ制御（ライブ）** | あり（[Cameras] タブ） | はい (`chloros-cli lattice`、SDK) | はい | はい |
| **DAQ 光センサー（ライブ）** | はい（「光センサー」タブ） | はい (`chloros-cli daq pool-*`、SDK) | はい | はい |
| **PTP タイムシンク（ホストがグランドマスター）** | はい | はい (`chloros-cli time-sync`) | はい | はい |
| **GPU アクセラレーション (CUDA)** | はい | はい | はい | はい (JetPack 6) |
| **テクスチャ対応デベイヤー**（CUDA対応GPUが必要） | あり（Chloros+） | あり（Chloros+） | あり（Chloros+） | あり（Chloros+） |
| **動的コンピュート適応** | あり | あり | あり | あり |
| **システムサービスとしてのバックエンド** (`chloros-backend@<user>.service`) | いいえ | いいえ | はい（オプトイン） | はい（オプトイン） |
| **インプレース・アップデータ** (`chloros-cli update`) | いいえ（インストーラーを実行） | いいえ（インストーラーを実行） | はい | はい |***

## サポートされているアーキテクチャ

| アーキテクチャ | 説明 | パッケージ |
| --- | --- | --- |
| **amd64 (x86_64)** | 標準的なデスクトップ/サーバー用プロセッサ (Intel、AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM プロセッサ — NVIDIA Jetson Orin ファミリー (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## サポートされているLinuxディストリビューション

* **Ubuntu 22.04 LTS 以降** (amd64)
* **Debian 12 以降** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin プラットフォーム)***

## Linux ユーザーが利用できる機能

* **Chloros CLI** — バッチ処理、自動化、スクリプト作成のための完全なコマンドラインインターフェース
* **Chloros Python SDK** — 研究パイプラインやカスタムツール向けのプログラム用 Python インターフェース。システム `python3` 向けに `.deb` によってインストールされます （virtualenv用のバージョン対応wheelは`/usr/lib/chloros/sdk/`に含まれています。PyPIはChlorosパッケージがないマシン向けです）
* **LATTICE カメラ制御** — `chloros-cli lattice` および SDK を通じて、LATTICE カメラや同期されたマルチカメラアレイの検出、接続、設定、および画像取得を行います。`.deb` には、カメラに必要な Arena SDK ランタイムがバンドルされています
* **DAQ光センサー制御** — `chloros-cli daq pool-*`およびSDKを介して、DAQ-U/M/Eセンサーに接続し、校正済みスペクトルをストリーミングし、`.daq`ファイルを記録します
* **PTP 時刻同期** — Chloros バックエンドは、LATTICE カメラおよび DAQ-E センサーがスレーブとして同期する PTP グランドマスターを実行します。 `chloros-cli time-sync`でこれを確認し、`chloros-backend@<user>.service` systemdユニットを使用してヘッドレスで稼働させ続けます（[Linuxインストール](linux-installation.md#always-on-ptp-for-headless-hosts)を参照）
* **プロジェクトの自動化** — `chloros-cli project`およびSDKの`open_project`を使用して、保存されたプロジェクトをヘッドレスで実行します
* **GPU アクセラレーション** — NVIDIA GPU での CUDA アクセラレーション処理（amd64 上のデスクトップ GPU、および JetPack 6 を搭載した Jetson の GPU）
* **動的演算適応** — ハードウェアの自動検出と処理戦略の選択。専門家向けの緊急回避手段として、`CHLOROS_STRATEGY`による上書きが可能
* **すべての処理機能** — Windows と同じパイプライン：キャリブレーション、ヴィネット補正、植生指数、およびすべてのエクスポート形式
* **Chloros+ の機能** — マルチスレッド（パイプライン）処理、テクスチャ対応デベイヤー（CUDA GPU上）、およびカスタム指数。有料のChloros+ プランが必要

## Linuxユーザーには提供されない機能

* **デスクトップGUI** — グラフィカルインターフェースなし。すべての操作はCLIまたはPythonSDKを介して行われます
* **画像ビューア** — インタラクティブな画像ビューア、グリッドビュー、マップマーカーはありません
* **視覚的なプロジェクト管理** — プロジェクトの作成および管理は、CLIコマンドおよびSDK呼び出しを通じて行われます（カメラ、センサー、キャプチャといったハードウェア自体は、ターミナルから完全に制御可能です）***

## ライセンス要件

CLIおよびSDKへのアクセスには、**有料のChloros+プラン — Copper以上**（Copper、Bronze、Silver、Gold）が必要です。無料の**Iron**プランでは、CLI/SDKへのアクセスはできません。この制限はCLIだけでなく、バックエンド側でも強制されます：

| 状況 | バックエンドの応答 |
| --- | --- |
| ログインしていない | `401` と `error_code: AUTH_REQUIRED` |
| 無料のIronティアでログイン中 | `403` と `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status`はどのティアでも機能します（ゲートから除外される唯一のルートです）。そのため、拒否の理由は常に確認可能です。

***

## Linuxでの Linuxでの

1. **Chlorosのインストール** — `.deb`のインストールについては、[Linuxインストール](linux-installation.md)を参照してください
2. **動作確認** — `chloros-cli --version` を実行すると `Chloros CLI 1.3.0` が出力され、`chloros-cli selftest` を実行すると 7 ステップの診断が実行されます
3. **Python SDKの確認** — `.deb`がこれをインストールします：`python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"`（virtualenv内では、`pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`）
4. **サインイン** — `chloros-cli login your@email.com 'your-password'`（1台のマシンにつき1回。パッケージのアップグレード時にはログイン情報が保持されますが、パッケージを削除するとログイン情報は消去されます）
5. **最初のデータセットを処理** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson については、プラットフォーム固有の設定、熱特性、および現場での導入については、専用の [NVIDIA Jetson ガイド](nvidia-jetson-guide.md) を参照してください。

***

## 次の手順

* [Linux インストール](linux-installation.md) — amd64 および arm64 向けの詳細なインストール手順、ファイルの場所、トラブルシューティング
* [NVIDIA Jetson ガイド](nvidia-jetson-guide.md) — Jetson固有の設定、メモリおよび熱挙動、実環境での導入
* [CLI : コマンドライン](../CLI.md) — CLIガイド
* [API : Python SDK](../api-python-sdk.md) — SDKガイド
* [CLIリファレンス](../reference/cli-reference.md) および [SDKリファレンス](../reference/sdk-reference.md) — バージョン1.3.0のコマンドおよびAPIの網羅的な一覧
* [動的コンピューティング適応](../processing-architecture/dynamic-compute-adaptation.md) — Chlorosがハードウェアに適応する方法

{% hint style="info" %}
**このマニュアルをプログラムで読み込む。** 各ページは、そのページ固有のURLに`.md`を付加した形式（例：`https://mapir.gitbook.io/chloros/linux/linux-installation.md`）で生のMarkdownとして提供されており、マニュアル全体の索引は[`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt)で公開されています。
{% endhint %}
