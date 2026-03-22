# NVIDIA Jetson ガイド

NVIDIA Jetson 上の Chloros は、現場、UAV、遠隔地での設置環境など、エッジ環境におけるマルチスペクトル画像処理を実現します。Chloros は、お使いの Jetson モデルを自動的に検出し、ハードウェアに合わせて処理戦略を最適化します。

***

## 対応する Jetson モデル

| モデル                | RAM            | 処理戦略                                   | 推奨用途                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB 共有 | `GPU_PARALLEL` (4 ワーカー)                            | 最高性能、大規模データセット                      |
| **Jetson Orin NX**   | 8-16GB 共有  | `GPU_PARALLEL` (3 ワーカー、16GB) / `GPU_SINGLE` (8GB) | 航空機搭載およびフィールド展開における主な推奨構成 |
| **Jetson Orin Nano** | 8GB 共有     | `GPU_SINGLE` (1ワーカー)                               | エントリーレベルのエッジコンピューティング                                 |
| **Jetson Nano**      | 4-8GB 共有   | `GPU_SINGLE` (1ワーカー)                               | エントリーレベル、メモリ制約あり                          |

{% hint style="info" %}
**旧型Jetsonモデル** (TX2、TX1、Xavier NX) はサポートされていない場合があります。パフォーマンスは、利用可能なGPUメモリとCUDA機能によって異なります。
{% endhint %}

***

## 要件

* **JetPack 6.x** (最新版を推奨)
* **NVIDIA CUDA** (JetPackに同梱)
* **Chloros+ ライセンス** (CLI/SDKへのアクセスに必要)

## インストール

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Linuxの一般的なインストール手順については、[Linuxのインストール](linux-installation.md)を参照してください。

***

## Jetsonにおける動的演算適応

Chlorosは、お使いのJetsonモデルを自動的に検出し、最適な処理戦略を選択します。**手動での調整は不要です。**

### 仕組み

起動時、Chlorosはシステムのプロファイリングを行います：

1. **`/proc/device-tree/model`**を使用して**Jetsonモデルを検出**

2.**利用可能なGPU/共有メモリを読み取ります**

3.**処理戦略を選択します**（`GPU_PARALLEL`、`GPU_SINGLE`、または`CPU_PARALLEL`）
4. **ワーカー数、パイプラインタイプ、およびメモリ割り当てを**自動的に設定します

### モデルごとの動作

| Jetson モデル                | 戦略       | ワーカー数 | パイプライン                       | 並行処理数 |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (メモリ効率化) | シリアル化  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | シリアル化  |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | シリアライズ済み  |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (フルGPUパス)    | 並列  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | 並行  |

{% hint style="success" %}
**Jetson Orin NX 16GB**はエッジ展開に最適なモデルです。3つの並列ワーカーを備えた`GPU_PARALLEL`戦略を採用し、コンパクトなフォームファクタで真の並列GPU処理を実現します。
{% endhint %}

プラットフォーム間の主な違いは**メモリ**です。 8GBの共有メモリを搭載したJetson Nanoは、メモリ効率に優れたタイル処理方式を用いて画像を1枚ずつ処理する必要がありますが、16GBを搭載したOrin NXは、高スループットの融合パイプラインを使用して、GPU上で3枚の画像を同時に処理できます。

コンピュート適応に関する詳細については、[動的コンピュート適応](../processing-architecture/dynamic-compute-adaptation.md)を参照してください。

***

## 熱管理

Jetsonデバイスは、特に密閉環境や航空機搭載環境において、熱的余裕が限られています。Chlorosには、自動的な温度監視およびスロットリング機能が含まれています：

| 温度         | アクション                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | 通常動作 — フル処理速度                        |
| **70°C** (警告)  | バッチサイズを自動的に縮小                        |
| **80°C** (危険) | 強力なスロットリング — 並行処理数を低減         |
| **90°C** (シャットダウン) | GPU処理を完全に停止 — 冷却が必要 |

{% hint style="warning" %}
特に密閉されたフィールドエンクロージャーや航空機搭載システムでは、持続的な処理を行うために**十分な換気と放熱を確保**してください。サーマルスロットリングは、ハードウェアを保護するために処理スループットを低下させます。
{% endhint %}

***

## メモリ管理

Jetsonデバイスは**ユニファイドメモリ**を採用しており、GPUとCPUが同じ物理RAMを共有します。つまり、報告されるVRAM（例：Orin NX 16GBでは15.3GB）はGPU専用メモリではなく、オペレーティングシステムや他のプロセスと共有されています。

### スワップ領域の推奨

大規模なデータセットやテクスチャ対応デベイヤー処理の場合、Chlorosはスワップ領域の作成を推奨することがあります：

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**画像ごとのメモリ推定値：**

* 標準デベイヤー：画像あたり約10MB
* テクスチャ対応デベイヤー：画像あたり約15MB

Chlorosは、データセットのサイズに基づいて必要なメモリを自動的に計算し、スワップが推奨される場合は警告を表示します。

### OOM（メモリ不足）時のフォールバック

処理中にメモリ不足の状態が検出された場合：

1. Chlorosは自動的にGPUワーカー数を削減します
2. `fused_gpu`から`tiled_gpu`パイプラインへフォールバックします（メモリ効率が向上）
3. クラッシュすることなく、スループットを低下させた状態で処理を継続します

***

## 現場での導入

### 電力に関する考慮事項

| Jetsonモデル     | 標準消費電力 | 備考                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10W              | USB-C またはバレルジャック    |
| Jetson Orin Nano | 7-15W              | DCバレルジャック          |
| Jetson Orin NX   | 10-25W             | DCバレルジャック          |
| Jetson AGX Orin  | 15-60W             | USB-C PD またはバレルジャック |

持続的な処理のための電力予算を計画してください。ピーク時の消費電力は、GPUを多用するスレッド3（処理）中に発生します。

### ストレージの推奨事項

* **NVMe SSD**：arm64環境での導入には強く推奨されます
* SDカードは処理には速度が不足しています — ブートメディアとしてのみ使用してください
* 処理後の出力データには、生画像データの2～3倍の容量を確保してください

### SSH を使用したヘッドレス運用

Chloros CLIは、ヘッドレスJetsonデプロイメントに最適です：

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### systemdを使用した自動処理

自動処理用のsystemdサービスを作成します：

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

スケジュール処理を行うには、systemdタイマーと組み合わせて使用します:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## ワークフローの例

### 基本的なJetson処理

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Jetson での Python SDK

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### 複数のフライトのバッチ処理

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## フィールド用途におすすめの Jetson システム

フィールドおよび航空機搭載用途には、以下の Jetson Orin NX 16GB キャリアボードのオプションをご検討ください：

* **航空機/ドローン**：耐振動規格（MIL-STD）対応、軽量（300g未満）、パッシブ冷却のシステム
* **過酷なフィールド環境**：PoE GigEカメラ接続機能を備えたIP67/IP69K防水エンクロージャー
* **最小構成/低予算**：アドオンエンクロージャー付き開発者キット

導入シナリオに応じた具体的なハードウェアの推奨事項については、[MAPIR サポート](https://www.mapir.camera/community/contact)までお問い合わせください。

***

## 次のステップ

* [Linux インストール](linux-installation.md) — Linux インストールの概要
* [動的コンピューティング適応](../processing-architecture/dynamic-compute-adaptation.md) — コンピューティング戦略の完全なリファレンス
* [処理パイプライン](../processing-architecture/processing-pipeline.md) — 4スレッドパイプラインの概要
* [CLI : コマンドライン](../CLI.md) — CLIの完全なリファレンス
* [API : Python SDK](../api-python-sdk.md) — SDKの完全リファレンス
