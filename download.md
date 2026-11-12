---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# ダウンロード

マルチスペクトル画像処理を始めるには、Chlorosの最新バージョンをダウンロードしてください。

### システム要件

#### Windows


| 要件          | 最小要件                                              | 推奨要件                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **オペレーティングシステム** | Windows 10 (64ビット)                                  | Windows 11 (64ビット)                                  |
| **プロセッサ**        | Intel Core i5 または同等のもの                          | Intel Core i7 以上                              |
| **メモリ (RAM)**     | 8GB                                                  | 16GB 以上                                         |
| **グラフィックカード**    | DirectX 11 対応                                | 4GB以上のVRAMを搭載したNVIDIA製GPU                            |
| **ストレージ**          | 6GBの空き容量                                       | 10GB以上の空き容量があるSSD                            |
| **ディスプレイ**          | 1920x1080                                            | 2560x1440以上                                  |
| **インターネット**         | \[オプション\]のChloros+ライセンス認証に必要          | \[オプション\]のChloros+ライセンス認証に必要          |

#### Linux amd64 (x86_64)

| 要件             | 最小要件                    | 推奨要件               |
| ----------------- | -------------------------- | ------------------------- |
| **ディストリビューション**  | Ubuntu 22.04 LTS 以降 / Debian 12 以降 | Ubuntu 24.04 LTS      |
| **プロセッサ**     | x86_64 (Intel/AMD)        | Intel Core i7 以上   |
| **メモリ (RAM)**  | 8GB                        | 16GB以上              |
| **グラフィックカード** | 不要 (CPU処理)      | 4GB以上のVRAMを搭載したNVIDIA GPU |
| **ストレージ**       | 空き容量 2GB             | 空き容量 10GB以上のSSD       |
| **Python**        | Python 3.7以上 (SDK用)      | Python 3.10以上              |

#### Linux arm64 (NVIDIA Jetson)

| 要件      | 最小要件                      | 推奨要件                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **プラットフォーム**     | JetPack 6 搭載 NVIDIA Jetson | Jetson Orin NX 16GB または AGX Orin |
| **メモリ (RAM)** | 8GB (GPU/CPU 共有)         | 16GB 以上の共有メモリ                    |
| **ストレージ**      | 空き容量 2GB               | 空き容量 10GB以上の NVMe SSD        |
| **Python**       | Python 3.7以上 (SDK用)        | Python 3.10以上                    |

{% hint style="info" %}
**GPU アクセラレーション**：NVIDIA GPU をお持ちの Chloros+ ユーザーは、CUDA アクセラレーションを利用して処理速度を大幅に向上させることができます。これは、Windowsおよび Linux amd64 上の NVIDIA デスクトップ GPU、ならびに JetPack 6 を搭載した NVIDIA Jetson で動作します。また、Chloros+ ユーザーは、最大速度を実現するためのマルチスレッド処理も利用できます。
{% endhint %}

***

## Chloros
のダウンロード

### 最新の安定版：バージョン 1.3.0 (2026年11月11日)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Windows用Chlorosのダウンロード (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Linux amd64用Chlorosのダウンロード (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Linux arm64 / Jetson用Chlorosのダウンロード (.deb)</a>


#### Windowsインストーラー (GUI + CLI + バックエンド)

* **ファイル形式**: .exe (Windowsインストーラー)

**インストール手順：**

1. 上記の .exe ファイルをダウンロードします
2. インストーラーをダブルクリックしてインストールを開始します
3. インストールウィザードの指示に従います
4. インストール先ディレクトリを選択します（デフォルト：`C:\Program Files\Chloros\`）
5. **[追加タスクの選択]**ページで、**[システム設定]** の両方のオプションにチェックを入れたままにします（推奨 — 下記参照）
6. インストールを完了し、Chlorosまたは Chloros CLIを起動します7. [MAPIR Cloud Chloros+ アカウント](https://cloud.mapir.camera/pricing) でサインインします （または無料版のまま続行）

**システム設定オプション**（デフォルトで両方チェック済み）：

| インストーラー上のオプション名 | 機能 | サイレントインストール除外 |
| --- | --- | --- |
| **Windowsの長いパスを有効にする**（プロジェクトフォルダのパスを260文字より深く設定可能；推奨） | Windowsのシステム全体に適用される`LongPathsEnabled`設定を「1」に設定します。Windowsではデフォルトでこの設定がオフになっており、ファイルパスは260文字を超えてはなりません。Chloros製品はプロジェクトフォルダの下に最大約130文字まで配置されるため、長いパスがオフになっていると、それより深い場所にあるプロジェクトフォルダは拒否されます。 インストール後に起動するプログラムに対して有効になります。他のプログラムがこの設定に依存している可能性があるため、アンインストール後も有効なままになります。 | `/MERGETASKS="!longpaths"` |
| **Microsoft DefenderのスキャンからChlorosバックエンドを除外**（カメラのキャプチャ速度が大幅に向上します。推奨） | インストールフォルダ内の`chloros-backend.exe`に対してMicrosoft Defenderのプロセス除外を設定します。これにより、バックエンドが書き込むファイルがどこに書き込まれていても、リアルタイム保護によってスキャンされることがなくなります。 高いキャプチャレートでは、スキャンにかかるコストはキャプチャレートの約30%でした（9台のカメラによる連続キャプチャにおいて、1秒あたり77.7回対52.7回）。 他のプログラムは除外されず、アンインストール時にこの除外設定は解除されます。Defender が存在しない場合、またはポリシーによってロックされている場合、この手順は静かに失敗し、インストールは続行されます。 | `/MERGETASKS="!defenderexclusion"` |

サイレントインストール（`/SILENT` または `/VERYSILENT`）では、表示されている `/MERGETASKS` 値でオプトアウトしない限り、両方のオプションが適用されます。

{% hint style="success" %}
インストーラーは、CLIフォルダー（`C:\Program Files\Chloros\cli\`）をシステムに一度だけ追加します（`PATH`）。再インストールやアップグレードを行っても、重複するエントリが追加されることはなく、以前のインストーラーによって追加された重複エントリは削除されます。 インストール後に新しいターミナルを開き、`chloros-cli`を使用してください。また、インストーラーは、すべてのネットワークプロファイルにおいて、バックエンド用に 2 つの Windowsファイアウォール ルールを追加します。 **Chloros Backend (UDP-In)**は、カメラストリーム、PTP、mDNS、および DAQ-E データストリーム用です。また、**Chloros Backend (TCP-In)** はローカルサブネットに限定され、DAQ-E ファームウェアの更新時に、センサーがこのコンピュータに接続して画像を取得するために使用されます。 どちらもアンインストール時に削除されます。
{% endhint %}

#### Linux amd64 (.deb パッケージ — CLI + バックエンド)

* **ファイル形式**: .deb (Debian/Ubuntu パッケージ)
* **アーキテクチャ**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb パッケージ — CLI + バックエンド)

* **ファイル形式**: .deb (JetPack 6)
* **アーキテクチャ**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

実際にダウンロードしたファイル名を使用し、`./`の部分はそのまま残してください。そうすることで、`apt`がパッケージ名を指定して検索するのではなく、そのファイルを直接インストールするようになります。`apt install`を使用してインストールしてください。 `dpkg -i`ではインストールしないでください。aptは、パッケージが必須ではなく推奨しているPython SDKの依存関係もインストールしますが、`dpkg -i`ではそれらをスキップします。 バージョン 1.3.0 以降、バックエンドとCLIは `/usr/lib/chloros/` の下に解凍済みでインストールされるため、最初に自身を解凍することなく起動します。

Jetson パッケージは JetPack 6 の CUDA ビルド版 PyTorch を使用しているため、含まれるテクスチャ対応デベイヤーである processing は、Jetson の GPU 上で実行されます。 JetPack 6 がインストールする CUDA および cuDNN ライブラリを使用します（これらが含まれていないイメージ上では `sudo apt install nvidia-jetpack`）。

詳細なセットアップ手順については[Linuxインストール](linux/linux-installation.md)を、Jetson固有のガイダンスについては[NVIDIA Jetsonガイド](linux/nvidia-jetson-guide.md)を参照してください。

#### Python SDK (全プラットフォーム)

各インストーラーには対応する `chloros_sdk` wheel が同梱されているため、SDKのバージョンは常にインストールされた GUI/CLI/バックエンドと一致します。Windowsでは、インストーラーがシステム Pythonに自動的にインストールします。 Linuxでは、`.deb`がシステムSDK自体（`python3`内）にSDKをインストールするため、 そのため、`import chloros_sdk`はインストール直後から、pipやネットワーク接続がなくても正常に動作します。システムのパッケージを認識しないvirtualenvのみ、同梱のwheelをその中にインストールする必要があります：

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

pipのみのホスト（Chlorosパッケージがインストールされていない場合。この場合、SDKは別のマシン上のバックエンドを駆動します）については、SDKもPyPIで公開されています：

```bash
pip install chloros-sdk
```

ドキュメントについては、[API: Python SDK](api-python-sdk.md) および [SDKリファレンス](reference/sdk-reference.md) を参照してください。

{% hint style="info" %}
**Linuxユーザー**：`.deb`パッケージには、CLI、バックエンド、およびPython SDKが含まれています。Linux用のGUIは存在せず、すべての操作はCLIまたはSDKを介して行われます。
{% endhint %}

***

## 追加リソース

### Python SDK


開発者や自動化ワークフロー向けに、Chloros Python SDKはすべてのインストーラに同梱されています。WindowsインストーラはPython環境にインストールし、Linuxの`.deb`はシステム用の`python3`をインストールします。 Chlorosパッケージがインストールされていないマシン（別のマシン上のバックエンドを駆動するマシン）では、PyPIからインストールしてください：

```bash
pip install chloros-sdk
```

**ドキュメント**：[API：Python SDK](api-python-sdk.md)

**必要条件**: 制御対象のChlorosバックエンド（WindowsインストーラまたはLinux用`.deb`パッケージがこれをインストールします）、および有料プランのChloros+ログイン

***

## 含まれるもの

### Windowsインストーラ

* ✅ **Chloros GUI** - フル機能のグラフィカルインターフェース
* ✅ **Chloros CLI** - コマンドラインインターフェース（Chloros+ ライセンスが必要）
* ✅ **Chlorosバックエンド** - 処理エンジン
* ✅ **Python SDK** - Pythonがインストールされている場合、そこにインストールされます（Chloros+ ライセンスが必要です）
* ✅ **カメラプロファイル** - 事前設定済みのMAPIRカメラテンプレート

### Linux.deb パッケージ

* ✅ **Chloros CLI** - コマンドラインインターフェース（Chloros+ ライセンスが必要）
* ✅ **Chlorosバックエンド** - 処理エンジン
* ✅ **Python SDK** - システム `python3` 向けにインストールされます（Chloros+ ライセンスが必要）
* ✅ **カメラプロファイル** - 事前設定済みのMAPIRカメラテンプレート
* ❌ GUIなし — Linuxはヘッドレスで、CLI/SDKのみ対応

### Python SDK (PyPI、Chlorosパッケージがないマシン用)

* ✅ **Chloros SDK** - Python API（Chloros+ ライセンスが必要）

***

## Chloros+ へのアップグレード

Chloros+ サブスクリプションで高度な機能を利用可能：

* 🚀 **マルチスレッド処理** - 画像を並列処理
* ⚡ **GPU (CUDA) アクセラレーション** - NVIDIA GPUの性能を活用
* 💻 **CLIアクセス** - コマンドラインツールによる自動化
* 🐍 **Python SDK** - プログラムによるAPIアクセス
* 📱 **複数デバイス** - 2～10台以上のデバイスで使用可能（プランにより異なります）
* **🐻 高度なテクスチャ対応デベイヤー法** - 高品質なエッジ認識デベイヤーと、デベイヤー処理によるノイズをほぼ完全に除去するAI/MLノイズ除去モデルを組み合わせたもの。
* 🧮 **カスタム計算式** - カスタムマルチスペクトル指標を作成

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ のプランと価格を見る</a></p>

***

## インストールに関するヘルプ

### トラブルシューティング

**エラーメッセージが表示され、インストールに失敗する場合：**

* 管理者権限を持っていることを確認してください
* ウイルス対策ソフトを一時的に無効にしてください
* システムの最低要件を満たしているか確認してください

**アプリケーションが起動しない場合（Windows）：**

* Windows 10/11（64ビット）がインストールされていることを確認してください
* グラフィックドライバを更新してください
* Windowsのイベントビューアでエラーの詳細を確認してください
* エラーログを添えてサポートにお問い合わせください

**CLIが起動しない（Linux）：**

* `.deb` パッケージが正しくインストールされているか確認してください：`dpkg -l | grep chloros`
* アクセス権限を確認してください：`sudo chmod +x /usr/bin/chloros-cli`
* 診断を実行してください：`chloros-cli selftest`
* 不足しているライブラリがないか確認してください：`ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"`（バージョン 1.3.0 以降、`/usr/lib/chloros/chloros-backend` は小さなランチャースクリプトであり、プログラム本体は `/usr/lib/chloros/backend/` にあります）

**ライセンス認証に関する問題：**

* インターネット接続が有効であることを確認してください
* [https://cloud.mapir.camera](https://cloud.mapir.camera) で認証情報を確認してください
* ファイアウォールが Chlorosをブロックしていないか確認してください* 詳細な手順については、[Chloros+ ログイン](chloros+-login.md) をご覧ください

### サポートについて

インストールや設定についてサポートが必要ですか？

* 📧 **メール**: info@mapir.camera
* 🌐 **ウェブサイト**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **ドキュメント**: [はじめに](./)
* ❓ **FAQ**: [よくある質問](faq.md)

***

## ソフトウェアの更新

Chlorosは更新を確認し、新しいバージョンが利用可能になると通知し、このダウンロードページへのリンクを表示します。新しい署名付きインストーラーを実行することで更新できます。設定、プロジェクト、ログイン情報は更新後も保持されます。 LinuxおよびJetsonでは、`chloros-cli update`が新しいバージョンの有無を確認し、対応する`.deb`のダウンロードとインストールを提案します（このコマンドはLinux専用です）。

***

## 変更履歴**バージョン 1.3.0 (最新版、2026年11月11日)**— 変更点の完全なリストについては、[はじめに](./) ページの**「Chloros

1.3.0 の新機能」** をご覧ください。**バージョン 1.2.0**— 詳細は、[はじめに](./) ページの**「Chloros 1.2.0 の新機能」** をご覧ください。

<details>

<summary>バージョン 1.0.5</summary>

**リリース日：2026年2月10日**

**新機能**

* **テクスチャ対応デベイヤー方式 \[Chloros+ 限定] -** テクスチャ対応方式では、高品質なエッジ認識デベイヤーとAI/MLノイズ除去モデルを組み合わせており、デベイヤー処理によるノイズをほぼ完全に除去します。
* **T4Pキャリブレーションターゲットのサポート**
* **Chloros+のGPU処理速度の向上、メモリ管理の改善**

**バグ修正**

* 完全に刷新されたフロントエンド（GUI）。これで、すべてのWindowsコンピュータで動作するはずです。

</details>

<details>

<summary>バージョン 1.0.4</summary>

**リリース日：2026年1月5日**

**新機能**

* **画像/メタデータ切り替え**：ファイルブラウザに、選択した画像のメタデータを画像グリッドではなく表形式で表示するための切り替え機能を追加しました
* **画像グリッドズームスライダー**：サムネイルのサイズを調整するための新しいUIスライダー （CTRL キー＋マウスホイールも対応）
* **画像グリッドのエクスポートボタン**：上段に、サムネイルをJPGから処理済みエクスポート形式（ターゲット、反射率、インデックス、LUT）に切り替えるボタンを追加
* **マップタブ**：画像のGPS位置マーカーを表示する新しいインタラクティブな2Dマップ
  * Google MapsおよびESRIのマップタイルに対応（ズームレベルに応じて最適なタイルサービスを自動選択）
  * マップ上のマーカーにマウスをホバーするとサムネイルプレビューが表示されます

**バグ修正**

* 英語以外の言語設定のコンピュータでのChlorosのインストール対応を改善

</details>

<details>

<summary>バージョン 1.0.3</summary>

**リリース日：2025年12月20日**

**新機能**

* 初回リリース

**改善点**

* 初回リリース

**バグ修正**

* 初回リリース

**既知の問題**

* 初回リリース

</details>

***

## ライセンス契約**プロプライエタリソフトウェア** - Copyright (c) 2026 MAPIR Inc.

無断での使用、配布、または改変は禁止されています。

**無料版**：機能に制限がありますが、個人および商用利用が可能です**Chloros+**：高度な機能および商用展開向けのサブスクリプション型ライセンス
