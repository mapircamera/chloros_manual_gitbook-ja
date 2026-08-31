# Linux の概要

Chloros 1.2.0 では、**CLI**および**Python SDK** — ヘッドレス多波長画像処理に加え、LATTICEカメラおよびDAQ光センサーのリアルタイム制御 — に対するネイティブサポートを、Linuxワークステーション、サーバー、およびNVIDIA Jetsonエッジデバイス上で提供します。

{% hint style="info" %}
**Linux ではデスクトップ GUI は利用できません。**Chloros デスクトップ GUI は Windows 専用です。 Linuxのユーザーは、[CLI](../CLI.md) および [Python SDK](../api-python-sdk.md) を通じて Chloros とやり取りします。 `.deb`は、アプリケーションメニューに**Chloros CLI** エントリを追加するのではなく、単に `chloros-cli` を実行するターミナルエミュレータを開くだけです。
{% endhint %}

***

## プラットフォーム対応マトリックス

| 機能 | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **デスクトップGUI** | あり | 該当なし | なし | なし |
| **CLI** (`chloros-cli`) | あり | あり | あり | あり |
| **Python SDK** (`chloros-sdk`) | あり | あり | あり | あり |
| **画像処理パイプライン** | はい | はい | はい | はい |
| **LATTICE カメラ制御（ライブ）** | はい（[Cameras] タブ） | はい（`chloros-cli lattice`、SDK） | はい | はい |
| **DAQ光センサー（ライブ）** | はい（「光センサー」タブ） | はい（`chloros-cli daq pool-*`、SDK） | はい | はい |
| **PTP タイムシンク (ホストがグランドマスター)** | はい | はい (`chloros-cli time-sync`) | はい | はい |
| **GPU アクセラレーション (CUDA)** | はい | はい | はい | はい (JetPack 6) |
| **テクスチャ対応デベイヤー** | あり (Chloros+) | あり (Chloros+) | あり (Chloros+) | はい (Chloros+) |
| **動的コンピュート適応** | はい | はい | はい | はい |
| **システムサービスとしてのバックエンド** (`chloros-backend.service`) | いいえ | いいえ | はい (オプトイン) | はい (オプトイン) |
| **インプレース・アップデーター** (`chloros-cli update`) | いいえ (インストーラーを実行) | いいえ (インストーラーを実行) | はい | はい |***

## サポートされるアーキテクチャ

| アーキテクチャ | 説明 | パッケージ |
| --- | --- | --- |
| **amd64 (x86_64)** | 標準的なデスクトップ/サーバー用プロセッサ（Intel、AMD） | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARMプロセッサ — NVIDIA Jetson Orinファミリー | `chloros_<version>_arm64_jp6.deb` (JetPack 6 ビルド) |

## サポートされている Linux ディストリビューション

* **Ubuntu 22.04 LTS 以降** (amd64)
* **Debian 12 以降** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin プラットフォーム)***

## Linux ユーザーが得られる機能

* **Chloros CLI** — バッチ処理、自動化、およびスクリプト作成のための完全なコマンドラインインターフェース
* **Chloros Python SDK** — 研究用パイプラインやカスタムツール向けのプログラムによる Python インターフェース（PyPI からインストール可能。また、バージョンが一致した wheel として `.deb` に同梱されています）
* **LATTICE カメラ制御** — `chloros-cli lattice` および SDK を通じて、LATTICE カメラや同期されたマルチカメラアレイの検出、接続、設定、および画像取得を行うことができます。 `.deb`には、カメラに必要なArena SDKランタイムが同梱されています
* **DAQ光センサー制御** — `chloros-cli daq pool-*`およびSDKを介して、DAQ-U/M/Eセンサーを接続し、校正済みスペクトルをストリーミングし、`.daq`ファイルを記録します
* **PTP 時刻同期** — Chloros バックエンドは、LATTICE カメラおよび DAQ-E センサーがスレーブとして同期する PTP グランドマスターを実行します。`chloros-cli time-sync` を使用してこれを確認し、 `chloros-backend.service` systemd ユニットを使用してヘッドレスで稼働させ続けてください（[Linux インストール](linux-installation.md#always-on-ptp-for-headless-hosts)を参照）
* **プロジェクトの自動化** — `chloros-cli project` および SDK の `open_project` を使用して、保存されたプロジェクトをヘッドレスで実行します
* **GPU アクセラレーション** — NVIDIA GPU（デスクトップおよび Jetson）での CUDA アクセラレーション処理
* **動的演算適応** — ハードウェアの自動検出と処理戦略の自動選択。専門家が手動で介入できる手段として、`CHLOROS_STRATEGY`による上書き機能も備えています
* **すべての処理機能** — Windows と同じパイプライン：キャリブレーション、ヴィネット補正、植生指数、およびすべてのエクスポート形式
* **Chloros+の機能** — マルチスレッド（パイプライン）処理、テクスチャ対応デベイヤー、カスタム指数。有料のChloros+プランが必要

## Linux ユーザーには利用できない機能

* **デスクトップGUI** — グラフィカルインターフェースなし。すべての操作はCLIまたはPython SDK経由で行われます
* **画像ビューア** — 対話型の画像ビューア、グリッド表示、マップマーカーはありません
* **視覚的なプロジェクト管理** — プロジェクトの作成および運用は、CLIコマンドおよびSDK呼び出しを通じて行われます（カメラ、センサー、キャプチャなどのハードウェア自体は、ターミナルから完全に制御可能です）***

## ライセンス要件

CLI および SDK へのアクセスには、**有料の Chloros+ ティア — Copper 以上**（Copper、Bronze、Silver、Gold）が必要です。 無料の**Iron**プランでは、CLI/SDKへのアクセスはできません。この制限は、CLIだけでなく、バックエンドによって強制されます：

| 状況 | バックエンドの応答 |
| --- | --- |
| ログインしていない | `401`（`error_code: AUTH_REQUIRED`付き） |
| 無料の Iron ティアにログイン済み | `403` と `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status`はどのティアでも動作します（ゲートから除外される唯一のルートです）。そのため、拒否の理由は常に確認可能です。

***

## Linuxの開始手順

1. **Chlorosをインストール** — `.deb`のインストールについては、[Linuxのインストール](linux-installation.md)を参照してください
2. **確認** — `chloros-cli --version`は`Chloros CLI 1.2.0`を出力します。`chloros-cli selftest`は7段階の診断を実行します
3. **Python SDK** をインストールする（オプション） — `pip install chloros-sdk`
4. **サインイン** — `chloros-cli login your@email.com 'your-password'`（1台のマシンにつき1回、およびパッケージのアップグレードのたびに再度実行）
5. **最初のデータセットを処理** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson については、プラットフォーム固有の設定、熱特性、および現場での導入については、専用の [NVIDIA Jetson ガイド](nvidia-jetson-guide.md) を参照してください。

***

## 次の手順

* [Linux インストール](linux-installation.md) — amd64 および arm64 向けの詳細なインストール手順、ファイルの場所、トラブルシューティング
* [NVIDIA Jetson ガイド](nvidia-jetson-guide.md) — Jetson 固有の設定、メモリおよび熱挙動、実環境での導入
* [CLI : コマンドライン](../CLI.md) — CLIガイド
* [API : Python SDK](../api-python-sdk.md) — SDKガイド
* [CLI リファレンス](../reference/cli-reference.md) および [SDK リファレンス](../reference/sdk-reference.md) — バージョン 1.2.0 におけるコマンドおよび API の網羅的な一覧
* [動的演算適応](../processing-architecture/dynamic-compute-adaptation.md) — Chlorosがハードウェアにどのように適応するか

{% hint style="info" %}
**このマニュアルをプログラムで読み込む場合。** 各ページは、個別の URL および `.md` （例：`https://mapir.gitbook.io/chloros/linux/linux-installation.md`）で提供されており、マニュアル全体の索引は [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt) に公開されています。
{% endhint %}
