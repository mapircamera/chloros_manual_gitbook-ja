---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# ダウンロード

マルチスペクトル画像処理を始めるには、Chlorosの最新版をダウンロードしてください。

### システム要件

#### Windows

| 要件          | 最小                                              | 推奨                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **オペレーティングシステム** | Windows 10 (64ビット)                                  | Windows 11 (64ビット)                                  |
| **プロセッサ**        | Intel Core i5 または同等のもの                          | Intel Core i7 以上                              |
| **メモリ (RAM)**     | 8GB                                                  | 16GB 以上                                         |
| **グラフィックカード**    | DirectX 11 対応                                | 4GB 以上の VRAM を搭載した NVIDIA GPU                            |
| **ストレージ**          | 6GB の空き容量                                       | 10GB 以上の空き容量がある SSD                            |
| **ディスプレイ**          | 1920x1080                                            | 2560x1440 以上                                  |
| **インターネット**         | [オプション] Chloros+ ライセンスのアクティベーションに必要 | [オプション] Chloros+ ライセンスのアクティベーションに必要 |

#### Linux amd64 (x86_64)

| 要件             | 最小要件                    | 推奨要件               |
| ----------------- | -------------------------- | ------------------------- |
| **ディストリビューション**  | Ubuntu 20.04 以降 / Debian 11 以降 | Ubuntu 22.04 以降             |
| **プロセッサ**     | x86_64 (Intel/AMD)        | Intel Core i7 以上   |
| **メモリ (RAM)**  | 8GB                        | 16GB以上              |
| **グラフィックカード** | 不要 (CPU処理)      | 4GB以上のVRAMを搭載したNVIDIA GPU |
| **ストレージ**       | 2GBの空き容量             | 10GB以上の空き容量があるSSD       |
| **Python**        | Python 3.7以上 (SDK用)      | Python 3.10以上              |

#### Linux arm64 (NVIDIA Jetson)

| 要件      | 最小要件                      | 推奨要件                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **プラットフォーム**     | JetPack 6 搭載 NVIDIA Jetson | Jetson Orin NX 16GB または AGX Orin |
| **メモリ (RAM)** | 8GB (GPU/CPU 共有)         | 16GB 以上 (共有)                    |
| **ストレージ**      | 2GB の空き容量               | 10GB 以上の空き容量がある NVMe SSD        |
| **Python**       | Python 3.7+ (SDK用)        | Python 3.10+                    |

{% hint style="info" %}
**GPU アクセラレーション**: NVIDIA GPU 搭載の Chloros+ ユーザーは、CUDA アクセラレーションを利用して処理速度を大幅に向上させることができます。これは、Windows（デスクトップ GPU）および Linux（デスクトップ GPU および NVIDIA Jetson）の両方で動作します。 Chloros+ ユーザーは、マルチスレッド処理も利用でき、処理速度を最大化できます。
{% endhint %}

***

## Chloros のダウンロード

### 最新の安定版 (2026年3月23日): バージョン 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Windows用 Chloros のダウンロード (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Linux amd64用 Chloros のダウンロード (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Linux arm64 / Jetson用 Chloros (.deb)をダウンロード####</a>

Windows インストーラー (GUI + CLI + バックエンド)

* **ファイル形式**: .exe (Windows インストーラー)**インストール手順:**

1. 上記の .exe ファイルをダウンロードします
2. インストーラーをダブルクリックしてインストールを開始します
3. インストールウィザードの指示に従います
4. インストール先フォルダを選択します（デフォルト：`C:\Program Files\[USER]\Chloros\`）
5. インストールを完了し、Chloros または Chloros CLI を起動します
6. [MAPIR Cloud Chloros+ アカウント](https://cloud.mapir.camera/pricing) でサインインします（または無料版で続行します）

{% hint style="success" %}
インストーラーは、コマンドラインからアクセスできるよう、システム PATH に `chloros-cli` を自動的に追加します。
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

詳細なセットアップ手順については [Linux インストール](linux/linux-installation.md) を、Jetson に関する具体的なガイダンスについては [NVIDIA Jetson ガイド](linux/nvidia-jetson-guide.md) を参照してください。

#### Python SDK (全プラットフォーム)

```bash
pip install chloros-sdk
```

ドキュメントについては、[API : Python SDK](api-python-sdk.md)を参照してください。

{% hint style="info" %}
**Linux ユーザー**: `.deb` パッケージは、CLI およびバックエンドをインストールします。Python SDK は、pip 経由で別途インストールされます。 LinuxにはGUIがありません。すべての操作はCLIまたはSDKを介して行われます。
{% endhint %}

***

## 追加リソース

### Python SDK

開発者や自動化ワークフローの場合は、Chloros Python SDK をインストールしてください:

```bash
pip install chloros-sdk
```

**ドキュメント**: [API: Python SDK](api-python-sdk.md)**要件**: Chlorosがインストールされている必要があります（WindowsインストーラーまたはLinux `.deb`パッケージ）、Chloros+ライセンスへのログインが必要です***

## 内容

### Windows インストーラー

* ✅ **Chloros GUI** - フル機能のグラフィカルインターフェース
* ✅ **Chloros CLI** - コマンドラインインターフェース（Chloros+ ライセンスが必要）
* ✅ **Chloros バックエンド** - 処理エンジン
* ✅ **カメラプロファイル** - 事前設定済みの MAPIR カメラテンプレート

### Linux .deb パッケージ

* ✅ **Chloros CLI** - コマンドラインインターフェース (Chloros以上のライセンスが必要)
* ✅ **Chloros バックエンド** - 処理エンジン
* ✅ **カメラプロファイル** - 事前設定済みの MAPIR カメラテンプレート
* ❌ GUIなし — Linuxはヘッドレス版のみ（CLI/SDK）

### Python SDK (PIP、全プラットフォーム)

* ✅ **Chloros SDK** - Python API (Chloros+ ライセンスが必要)***

## Chloros+ へのアップグレード

Chloros+ サブスクリプションで高度な機能を利用可能：

* 🚀 **マルチスレッド処理** - 画像を並列処理
* ⚡ **GPU (CUDA) アクセラレーション** - NVIDIA GPUのパワーを活用
* 💻 **CLI アクセス** - コマンドラインツールによる自動化
* 🐍 **Python SDK** - プログラムによる API アクセス
* 📱 **複数デバイス** - 2～10台以上のデバイスで使用可能（プランにより異なります）
* **🐻 高度なテクスチャ対応デベイヤー法** - 高品質なエッジ認識デベイヤーとAI/MLノイズ除去モデルを組み合わせ、デベイヤー処理によるノイズをほぼ完全に除去します。
* 🧮 **カスタム数式** - カスタムマルチスペクトルインデックスを作成

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ のプランと価格を見る</a></p>***

## インストールに関するヘルプ

### トラブルシューティング

**エラーメッセージが表示され、インストールに失敗する場合：**

* 管理者権限を持っていることを確認してください
* ウイルス対策ソフトを一時的に無効にしてください
* システム要件を満たしているか確認してください

**アプリケーションが起動しない場合 (Windows)：**

* Windows 10/11 (64ビット) がインストールされているか確認してください
* グラフィックドライバを更新してください
* Windows イベントビューアでエラーの詳細を確認してください
* エラーログを添えてサポートにお問い合わせください

**CLI が起動しない (Linux):**

* `.deb` パッケージが正しくインストールされているか確認してください: `dpkg -l | grep chloros`
* アクセス権限を確認してください: `sudo chmod +x /usr/bin/chloros-cli`
* 診断を実行してください：`chloros-cli selftest`
* 不足しているライブラリがないか確認してください：`ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**ライセンスのアクティベーションに関する問題：**

* インターネット接続が有効であることを確認してください
* [https://cloud.mapir.camera](https://cloud.mapir.camera) で認証情報を確認してください
* ファイアウォールが Chloros をブロックしていないか確認してください
* 詳細な手順については、[Chloros+ ログイン](chloros+-login.md) をご覧ください

### サポートについて

インストールや設定でお困りですか？

* 📧 **メール**: info@mapir.camera
* 🌐 **ウェブサイト**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **ドキュメント**: [はじめに](./)
* ❓ **FAQ**: [よくある質問](faq.md)***

## 変更履歴

<details>

<summary>バージョン 1.1.0 (最新)</summary>

**リリース日: 2026年3月**

**新機能*** **Linux サポート** — Linux amd64 (x86_64) および arm64 (NVIDIA Jetson JetPack 6) 向けのネイティブ CLI および SDK。 `.deb`パッケージ経由でインストールしてください。
* **NVIDIA Jetsonのサポート** — Jetson Nano、Orin Nano、Orin NX、およびAGX Orinエッジデバイス向けに最適化された処理。
* **動的演算適応** — ハードウェアの自動検出と処理戦略の最適化。Chlorosは、Jetson NanoからマルチGPUワークステーションまで、お使いのハードウェアに適応します。
* **4スレッド処理パイプライン** — 動的なGPUメモリ割り当てを備えた、検出、キャリブレーション、処理、エクスポートのスレッドを並行実行します。
* **新しい CLI コマンド** — `selftest`（システム診断）および `update`（Linux 更新管理）。
* **新しい CLI プロセスフラグ** — `--debayer`（標準/テクスチャ対応）、`--indices`（インデックス指定）、`--target`（検出速度向上のため、まずターゲットサブフォルダを検索）。
* **新しいGUIメニュー項目** — 「ファイルの追加」、「フォルダの追加」、および「処理の開始/停止」が、メインメニューのドロップダウンから利用可能になりました。**改善点**

* クロスプラットフォームのバックエンド自動検出（WindowsおよびLinuxパス）
* スレッドごとの進行状況追跡機能を備えた SDK および `get_status()` の機能強化
* 新しい SDK 例外：`ChlorosConfigurationError`、`ChlorosAuthenticationError`
* NVIDIA Jetson 向けの熱管理および適応型スロットリング
* OOM 発生時のタイル化 GPU 処理へのフォールバックを備えた自動メモリ管理

</details>

<details>

<summary>バージョン 1.0.5</summary>

**リリース日: 2026年2月10日**

**新機能*** **テクスチャ対応デベイヤー法 \[Chloros+ のみ] -** テクスチャ対応は、高品質なエッジ認識デベイヤーとAI/MLノイズ除去モデルを組み合わせ、デベイヤー処理によるノイズをほぼ完全に除去します。
* **T4Pキャリブレーションターゲットのサポート*** **Chloros+のGPU処理高速化、メモリ管理の改善**

**バグ修正*** 完全に新しいフロントエンド（GUI）を採用し、すべてのWindowsコンピュータで動作するようになりました。

</details>

<details>

<summary>バージョン 1.0.4</summary>

**リリース日：2026年1月5日**

**新機能*** **画像/メタデータ切り替え**：ファイルブラウザに、選択した画像のメタデータを画像グリッドではなくテーブル形式で表示する切り替え機能を追加
* **画像グリッドズームスライダー**：サムネイルサイズを調整する新しいUIスライダー （CTRL + マウスホイールも対応）
* **画像グリッドエクスポートボタン**：上段に配置されたボタンで、サムネイルをJPGから処理済みエクスポート（ターゲット、反射率、インデックス、LUT）に切り替え可能
* **マップタブ**: 画像のGPS位置マーカーを表示する新しいインタラクティブな2Dマップ
  * Google MapsおよびESRIマップタイルに対応（ズームレベルに応じて最適なタイルサービスを自動選択）
  * マップ上のマーカーにマウスを合わせるとサムネイルプレビューが表示されます

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

## ライセンス契約**専有ソフトウェア** - Copyright (c) 2026 MAPIR Inc.

無断での使用、配布、または改変は禁止されています。

**無料版**：機能に制限がありますが、個人および商用利用が可能です**Chloros+**：高度な機能および商用展開向けのサブスクリプション型ライセンス
