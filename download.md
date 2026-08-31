---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# ダウンロード

マルチスペクトル画像処理を始めるには、Chlorosの最新バージョンをダウンロードしてください。

### システム要件

#### Windows

| 要件          | 最小                                              | 推奨                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **オペレーティングシステム** | Windows 10 (64ビット)                                  | Windows 11 (64ビット)                                  |
| **プロセッサ**        | Intel Core i5 または同等のもの                          | Intel Core i7 またはそれ以上のもの                              |
| **メモリ (RAM)**     | 8GB                                                  | 16GB以上                                         |
| **グラフィックカード**    | DirectX 11 対応                                | 4GB以上のVRAMを搭載したNVIDIA GPU                            |
| **ストレージ**          | 6GBの空き容量                                       | 10GB以上の空き容量があるSSD                            |
| **ディスプレイ**          | 1920x1080                                            | 2560x1440以上                                  |
| **インターネット**         | [オプション] Chloros+ ライセンスのアクティベーションに必要 | [オプション] Chloros+ ライセンスのアクティベーションに必要 |

#### Linux amd64 (x86_64)

| 要件             | 最小要件                    | 推奨要件               |
| ----------------- | -------------------------- | ------------------------- |
| **ディストリビューション**  | Ubuntu 22.04 LTS 以降 / Debian 12 以降 | Ubuntu 24.04 LTS      |
| **プロセッサ**     | x86_64 (Intel/AMD)        | Intel Core i7 以上   |
| **メモリ (RAM)**  | 8GB                        | 16GB以上              |
| **グラフィックカード** | 不要 (CPU処理)      | 4GB以上のVRAMを搭載したNVIDIA GPU |
| **ストレージ**       | 2GBの空き容量             | 10GB以上の空き容量があるSSD       |
| **Python**        | Python 3.7以上 (SDK用)      | Python 3.10以上              |

#### Linux arm64 (NVIDIA Jetson)

| 要件            | 最小要件                      | 推奨要件                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **プラットフォーム**     | JetPack 6 搭載の NVIDIA Jetson | Jetson Orin NX 16GB または AGX Orin |
| **メモリ (RAM)** | 8GB (GPU/CPU 共有)         | 16GB 以上（共有）                    |
| **ストレージ**      | 2GBの空き容量               | 10GB以上の空き容量を持つNVMe SSD        |
| **Python**       | Python 3.7+ (SDK用)        | Python 3.10以上                    |

{% hint style="info" %}
**GPU アクセラレーション**: NVIDIA GPU を搭載した Chloros+ ユーザーは、CUDA アクセラレーションを利用して処理速度を大幅に向上させることができます。 これは、Windows（デスクトップ用GPU）およびLinux（デスクトップ用GPUおよびNVIDIA Jetson）の両方で動作します。 Chloros+ ユーザーは、マルチスレッド処理も利用でき、処理速度を最大化できます。
{% endhint %}

***

## Chloros のダウンロード

### 最新の安定版リリース：バージョン 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Windows（.exe）用の Chloros をダウンロード</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Linux用 Chloros (amd64) (.deb) のダウンロード</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Linux arm64 / Jetson用 Chloros のダウンロード (.deb)</a>

#### Windows インストーラ (GUI + CLI + バックエンド)

* **ファイル形式**: .exe (Windows インストーラ)**インストール手順:**

1. 上記の .exe ファイルをダウンロードします
2. インストーラをダブルクリックしてインストールを開始します
3. インストールウィザードの指示に従います
4. インストール先ディレクトリを選択します（デフォルト: `C:\Program Files\MAPIR\Chloros\`）
5. インストールを完了し、Chloros または Chloros CLI を起動します
6. [MAPIR Cloud Chloros+ アカウント](https://cloud.mapir.camera/pricing) でサインインします（または無料版のまま続行します）

{% hint style="success" %}
インストーラは、コマンドラインからアクセスできるよう、システムの PATH に `chloros-cli` を自動的に追加します。
{% endhint %}

#### Linux amd64 (.deb パッケージ — CLI + バックエンド)

* **ファイル形式**: .deb (Debian/Ubuntu パッケージ)
* **アーキテクチャ**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb パッケージ — CLI + バックエンド)

* **ファイル形式**: .deb (JetPack 6)
* **アーキテクチャ**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

詳細なセットアップ手順については [Linux のインストール](linux/linux-installation.md) を、Jetson に関する具体的なガイダンスについては [NVIDIA Jetson ガイド](linux/nvidia-jetson-guide.md) を参照してください。

#### Python SDK (全プラットフォーム)

各インストーラには対応する `chloros_sdk` wheel が同梱されているため、SDK のバージョンは常にインストールされた GUI/CLI/バックエンドと一致します。 Windows では、インストーラが自動的にシステム Python にこれをインストールします。 Linuxでは、`.deb`がwheelを`/usr/lib/chloros/sdk/`に配置し、インストールコマンドを出力します：

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

pip専用ホスト（Chlorosパッケージがインストールされていない場合）については、SDKもPyPIで入手可能です：

```bash
pip install chloros-sdk
```

[API : Python SDK](api-python-sdk.md)および [SDK リファレンス](reference/sdk-reference.md) を参照してください。

{% hint style="info" %}
**Linux ユーザー**: `.deb` パッケージは、CLI およびバックエンドをインストールします。 Linux には GUI がありません。すべての操作は、CLI または SDK を通じて行われます。
{% endhint %}

***

## 追加リソース

### Python SDK

開発者および自動化ワークフローの場合は、Chloros、Python、SDKをインストールしてください：

```bash
pip install chloros-sdk
```

**ドキュメント**: [API: Python SDK](api-python-sdk.md)**要件**: Chloros がインストールされている必要があります（Windows インストーラーまたは Linux `.deb` パッケージ）、 Chloros+ ライセンスへのログインが必要です***

## 同梱内容

### Windows インストーラー

* ✅ **Chloros GUI** - フル機能のグラフィカルインターフェース
* ✅ **Chloros CLI** - コマンドラインインターフェース（Chloros+ ライセンスが必要です）
* ✅ **Chloros バックエンド** - 処理エンジン
* ✅ **カメラプロファイル** - あらかじめ設定済みの MAPIR カメラテンプレート

### Linux .deb パッケージ

* ✅ **Chloros CLI** - コマンドラインインターフェース（Chloros以上のライセンスが必要）
* ✅ **Chloros バックエンド** - 処理エンジン
* ✅ **カメラプロファイル** - 事前設定済みの MAPIR カメラテンプレート
* ❌ GUI なし — Linux はヘッドレス版のみ（CLI/SDK）

### Python SDK (PIP、全プラットフォーム)

* ✅ **Chloros SDK** - Python API （Chloros+ ライセンスが必要）***

## Chloros+ へのアップグレード

Chloros+ サブスクリプションで高度な機能を利用可能に：

* 🚀 **マルチスレッド処理** - 画像を並列処理
* ⚡ **GPU (CUDA) アクセラレーション** - NVIDIA GPUの性能を活用
* 💻 **CLI アクセス** - コマンドラインツールによる自動化
* 🐍 **Python SDK** - プログラムによるAPIへのアクセス
* 📱 **複数デバイス** - 2～10台以上のデバイスで使用可能（プランにより異なります）
* **🐻 高度なテクスチャ対応デベイヤー方式** - 高品質なエッジ認識デベイヤーとAI/MLノイズ除去モデルを組み合わせ、デベイヤー処理によるノイズをほぼ完全に除去します。
* 🧮 **カスタム数式** - カスタムマルチスペクトル指標を作成

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+のプランと価格を確認する</a></p>***

## インストールに関するヘルプ

### トラブルシューティング

**以下のエラーメッセージが表示され、インストールに失敗する場合：**

* 管理者権限を持っていることを確認してください
* ウイルス対策ソフトを一時的に無効にしてください
* システムの最低要件を満たしているか確認してください

**アプリケーションが起動しない (Windows):**

* Windows 10/11 (64ビット) がインストールされていることを確認してください
* グラフィックドライバを更新してください
* Windowsのイベントビューアーでエラーの詳細を確認してください
* エラーログを添えてサポートにお問い合わせください

**CLIが起動しない (Linux):**

* `.deb` パッケージが正しくインストールされているか確認してください：`dpkg -l | grep chloros`
* アクセス権限を確認してください：`sudo chmod +x /usr/bin/chloros-cli`
* 診断を実行してください：`chloros-cli selftest`
* 不足しているライブラリがないか確認してください: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**ライセンスの有効化に関する問題:**

* インターネット接続が有効であることを確認してください
* [https://cloud.mapir.camera](https://cloud.mapir.camera) で認証情報を確認してください
* ファイアウォールが Chloros をブロックしていないか確認してください
* 詳細な手順については、[Chloros+ ログイン](chloros+-login.md) をご覧ください

### サポートを受ける

インストールや設定についてサポートが必要ですか？

* 📧 **メール**: info@mapir.camera
* 🌐 **ウェブサイト**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **ドキュメント**: [はじめに](./)
* ❓ **FAQ**: [よくある質問](faq.md)***

## ソフトウェアの更新

Chlorosは更新を確認し、新しいバージョンが利用可能になったことを通知するとともに、このダウンロードページへのリンクを表示します。新しい署名付きインストーラーを実行することで更新を行います。設定やプロジェクトは更新後も保持されます。 Linux および Jetson では、`chloros-cli update` が新しいバージョンの有無を確認し、対応する `.deb` のダウンロードとインストールを提案します （このコマンドは Linux でのみ利用可能です）。

***

## 変更履歴**バージョン 1.2.0 (最新)**— 機能の完全なリストについては、[はじめに](./) ページの**Chloros 1.2.0 の新機能** をご覧ください。

<details>

<summary>バージョン 1.0.5</summary>

**リリース日：2026年2月10日**

**新機能*** **テクスチャ対応デベイヤー手法 \[Chloros+ 限定] -** テクスチャ対応では、高品質なエッジ認識デベイヤーとAI/MLノイズ除去モデルを組み合わせており、デベイヤー処理によるノイズをほぼ完全に除去します。
* **T4Pキャリブレーションターゲットのサポート*** **Chloros+のGPU処理速度の向上、メモリ管理の改善**

**バグ修正*** 完全に新しいフロントエンド（GUI）を採用し、すべての Windows コンピュータで動作するようになりました。

</details>

<details>

<summary>バージョン 1.0.4</summary>

**リリース日：2026年1月5日**

**新機能*** **画像/メタデータの切り替え**：ファイルブラウザに、選択した画像のメタデータを画像グリッドではなく表形式で表示するための切り替え機能を追加しました。
* **画像グリッドのズームスライダー**：サムネイルのサイズを調整するための新しいUIスライダー （CTRL キーとマウスホイールによる操作も対応）
* **画像グリッドのエクスポートボタン**：上段に、サムネイルを表示形式を JPG から処理済みエクスポート（ターゲット、反射率、インデックス、LUT）に切り替えるボタンを追加
* **マップタブ**：画像の GPS 位置マーカーを表示する新しいインタラクティブな 2D マップ
  * Google MapsおよびESRIのマップタイルに対応（ズームレベルに応じて最適なタイルサービスを自動選択）
  * マップマーカー上でマウスをホバーするとサムネイルプレビューが表示されます

**バグ修正*** 英語以外の言語設定のコンピュータでのChlorosのインストール対応を改善

</details>

<details>

<summary>バージョン 1.0.3</summary>

**リリース日：2025年12月20日**

**新機能*** 初回リリース

**改善点*** 初回リリース

**バグ修正*** 初回リリース

**既知の問題*** 初回リリース

</details>***

## ライセンス契約**プロプライエタリソフトウェア** - Copyright (c) 2026 MAPIR Inc.

無断での使用、配布、または改変は禁止されています。

**無料版**：機能に制限がありますが、個人および商用利用が可能です**Chloros+**：高度な機能および商用展開向けのサブスクリプション型ライセンス
