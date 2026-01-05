---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# ダウンロード

マルチスペクトル画像処理を始めるには、最新バージョンのChlorosをダウンロードしてください。

### システム要件

| 要件                         | 最低要件                         | 推奨要件                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **オペレーティングシステム** | Windows 10 (64ビット)             | Windows 11 (64ビット)             |
| **プロセッサ**        | Intel Core i5 または同等品     | Intel Core i7 またはそれ以上         |
| **メモリ (RAM)**     | 8GB                             | 16GB 以上                    |
| **グラフィックカード**    | DirectX 11 互換           | NVIDIA GPU (4GB 以上の VRAM)       |
| **ストレージ**          | 6GB の空き容量                  | SSD (10GB 以上の空き容量)       |
| **ディスプレイ**          | 1920x1080                       | 2560x1440以上             |
| **インターネット**         | ライセンス認証に必須 | ライセンス認証に必須 |

{% hint style=&quot;info&quot; %}
**GPUアクセラレーション**: NVIDIA GPU（4GB以上のVRAM）を搭載したChloros+ユーザーは、CUDAアクセラレーションを利用して処理速度を大幅に向上させることができます。Chloros+ユーザーは、マルチスレッド処理も利用可能で、最高速度を実現します。
{% endhint %}

***

## Chloros をダウンロード

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Chloros はこちらからダウンロード</a>

### 最新安定版リリース

**Windows 用 Chloros インストーラー*** **バージョン**: 1.0.4
* **リリース日**: 2026年1月5日
* **ファイルサイズ（ダウンロード時）**: 1.8GB
* **ファイルサイズ（インストール後）**: 5.7GB
* **ファイルタイプ**: .exe (Windows インストーラー)

#### **インストール手順:**

1. `CHLOROS INSTALLER - CURRENT VERSION.exe` ファイルをダウンロード
2. インストーラーをダブルクリックしてインストールを開始
3. インストールウィザードの指示に従う
4. インストール先ディレクトリを選択 (デフォルト: `C:\Program Files\[USER]\Chloros\`)
5. インストールを完了し、Chloros、Chloros（ブラウザ）、またはChloros CLIを起動
6. [MAPIR Cloud Chloros+ アカウント](https://cloud.mapir.camera/pricing) でサインイン（または無料版を継続）

{% hint style=&quot;success&quot; %}
インストーラーはコマンドラインアクセス用に`chloros-cli`をシステムPATHに自動追加します。
{% endhint %}

***

## 追加リソース

### Python SDK

開発者および自動化ワークフロー向けには、Chloros Python SDK をインストールしてください:

```bash
pip install chloros-sdk
```

**ドキュメント**: [API: Python SDK](api-python-sdk.md)**要件**: Chloros Desktop のインストール必須、Chloros+ ライセンスログインが必要***

## 含まれる内容

Chloros のインストールには以下が含まれます:

* ✅ **Chloros** - フル機能のグラフィカルインターフェース
* ✅ **Chloros (Browser)** - 低スペックシステム向けWebベースインターフェース
* ✅ **Chloros CLI** - コマンドラインインターフェース（Chloros+ライセンスが必要）
* ✅ **Chloros SDK** - Python API (Chloros+ ライセンスが必要)
* ✅ **カメラプロファイル** - 事前設定済み MAPIR カメラテンプレート***

## Chloros+ へのアップグレード

Chloros+ サブスクリプションで高度な機能を解放:

* 🚀 **マルチスレッド処理** - 画像を並列処理
* ⚡ **GPU (CUDA) アクセラレーション** - NVIDIA GPU のパワーを活用
* 💻 **CLI アクセス** - コマンドラインツールによる自動化
* 🐍 **Python SDK** - プログラムによるAPIアクセス
* 📱 **複数デバイス** - 2～10台以上で利用可能（プランによる）
* 🧮 **カスタム式** - カスタムマルチスペクトル指標を作成

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ プランと価格を見る</a></p>***

## インストールヘルプ

### トラブルシューティング

**インストール時にエラーメッセージが表示される場合：**

* 管理者権限があることを確認してください
* ウイルス対策ソフトを一時的に無効にしてください
* 最低システム要件を満たしていることを確認してください

**アプリケーションが起動しない場合：**

* Chloros (ブラウザ版) を試す
* Windows 10/11 (64ビット) がインストールされていることを確認
* グラフィックドライバを更新
* Windows イベント ビューアーでエラー詳細を確認
* エラーログを添えてサポートに連絡

**ライセンス有効化の問題:**

* インターネット接続が有効であることを確認
* [https://cloud.mapir.camera](https://cloud.mapir.camera) で認証情報を確認
* ファイアウォールがChlorosをブロックしていないか確認
* 詳細な手順は[Chloros+ ログイン](chloros+-login.md)を参照

### サポートの受け方

インストールや設定でお困りですか？

* 📧 **メール**: info@mapir.camera
* 🌐 **ウェブサイト**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **ドキュメント**: [はじめに](./)
* ❓ **FAQ**: [よくある質問](faq.md)***

## 変更履歴

<details>

<summary>バージョン 1.0.4</summary>

#### **リリース日**: 2026年1月5日**新機能*** **画像/メタデータ切り替え**: ファイルブラウザに切り替え機能を追加。選択した画像のメタデータを画像グリッドではなくテーブル形式で表示可能
* **画像グリッドズームスライダー**: サムネイルサイズを調整する新しいUIスライダー （CTRL + マウスホイール操作も対応）
* **画像グリッドエクスポートボタン**: 上段に配置されたボタンで、サムネイル表示をJPGから処理済みエクスポート（ターゲット、反射率、インデックス、LUT）に切り替え可能
* **マップタブ**: 画像のGPS位置マーカーを表示する新しいインタラクティブ2Dマップ
  * Google MapsとESRIマップタイルに対応（ズームレベルに応じて最適なタイルサービスを自動選択）
  * マップマーカー上でマウスオーバーするとサムネイルプレビューを表示

**バグ修正*** 非英語環境でのChlorosインストールサポートを改善

</details>

<details>

<summary>バージョン 1.0.3</summary>

#### **リリース日**: 2025年12月20日**新機能*** 初回リリース

**改善点*** 初回リリース

**バグ修正*** 初回リリース

**既知の問題*** 初回リリース

</details>***

## ライセンス契約**専有ソフトウェア** - Copyright (c) 2025 MAPIR Inc.

無断使用、配布、または改変を禁じます。

**無料版**: 機能制限付きで個人および商用利用が可能**Chloros+**: 高度な機能と商用展開のためのサブスクリプション型ライセンス
