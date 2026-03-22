# Linux の概要

Chloros 1.1.0 では、**CLI**および**Python SDK**へのネイティブサポートを追加し、Linuxワークステーション、サーバー、およびNVIDIA Jetsonエッジデバイス上でヘッドレスなマルチスペクトル画像処理を可能にします。

{% hint style="info" %}
**LinuxにはGUIがありません。** ChlorosデスクトップGUIは、Windowsでのみ利用可能です。 Linuxユーザーは、[CLI](../CLI.md)および[Python SDK](../api-python-sdk.md) を通じて Chloros と連携します。
{% endhint %}

***

## プラットフォーム対応マトリックス

| 機能 | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **デスクトップGUI** | あり | 該当なし | なし | なし |
| **CLI** | あり | あり | あり | あり |
| **Python SDK** | あり | あり | あり | あり |
| **GPU アクセラレーション (CUDA)** | あり | あり | あり | あり (JetPack 6) |
| **テクスチャ対応デベイヤー** | あり (Chloros+) | あり (Chloros+) | あり (Chloros+) | はい (Chloros+) |
| **動的演算適応** | はい | はい | はい | はい |***

## 対応アーキテクチャ

| アーキテクチャ | 説明 | インストール方法 |
| --- | --- | --- |
| **amd64 (x86_64)** | 標準的なデスクトップ/サーバー用プロセッサ (Intel、AMD) | `.deb` パッケージ |
| **arm64 (aarch64)** | ARMベースのプロセッサ（主にNVIDIA Jetson） | `.deb` パッケージ (JetPack 6) |

## 対応する Linux ディストリビューション

* **Ubuntu 20.04 以降** (amd64)
* **Debian 11 以降** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetsonプラットフォーム)***

## Linuxユーザーが得られるもの

* **Chloros CLI** — バッチ処理、自動化、スクリプト作成のための完全なコマンドラインインターフェース
* **Chloros Python SDK** — 研究パイプラインやカスタムツールへの統合のためのプログラムインターフェース (Python)
* **GPU アクセラレーション** — NVIDIA GPU（デスクトップおよび Jetson）での CUDA アクセラレーション処理
* **動的演算適応** — ハードウェアの自動検出および処理戦略の最適化
* **すべての処理機能** — Windows と同じマルチスペクトル処理パイプライン（キャリブレーション、ヴィネット補正、植生指数、すべてのエクスポート形式）
* **Chloros+の機能** — マルチスレッド処理、テクスチャ対応デベイヤー、カスタム指数（Chloros+ライセンスが必要）

## Linux ユーザーには提供されない機能

* **デスクトップ GUI** — グラフィカルインターフェースなし。すべての操作は CLI または Python SDK 経由で行います
* **画像ビューア** — 対話型画像ビューア、グリッド表示、マップマーカーは利用できません
* **ビジュアルプロジェクト管理** — プロジェクトは CLI コマンドおよび SDK 呼び出しを通じて管理されます***

## Linux の開始方法

1. **Chlorosのインストール** — `.deb`パッケージのインストールについては、[Linux インストール](linux-installation.md)を参照してください
2. **Python SDK** をインストールする（オプション） — `pip install chloros-sdk`
3. **ライセンスを有効化する** — `chloros-cli login your@email.com 'password'`
4. **最初のデータセットを処理する** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson ユーザーの方は、プラットフォーム固有の設定および最適化について、専用の [NVIDIA Jetson ガイド](nvidia-jetson-guide.md) をご覧ください。

***

## 次の手順

* [Linux インストール](linux-installation.md) — amd64 および arm64 向けの詳細なインストール手順
* [NVIDIA Jetsonガイド](nvidia-jetson-guide.md) — Jetson固有の設定、熱管理、および実環境での導入
* [CLI : コマンドライン](../CLI.md) — CLIの完全なリファレンス
* [API : Python SDK](../api-python-sdk.md) — SDKの完全なリファレンス
* [動的演算適応](../processing-architecture/dynamic-compute-adaptation.md) — Chloros がハードウェアにどのように適応するか
