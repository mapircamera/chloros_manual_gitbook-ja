# NVIDIA Jetson ガイド NVIDIA Jetson 向けの

Chloros

は、現場、UAV、遠隔地などのエッジ環境におけるマルチスペクトル画像処理を実現します。Chloros

1.2.0は、起動時にJetsonモデルを自動的に検出し、検出されたハードウェアに合わせて処理戦略を最適化します。 **手動での調整は不要です。**

***

## 対応する Jetson モデル

| モデル                | RAM            | 処理戦略                                     | 推奨用途                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32～64GB 共有 | `GPU_PARALLEL` (2 ワーカー)                              | 最高性能、大規模データセット                      |
| **Jetson Orin NX**   | 8～16GB 共有  | `GPU_PARALLEL` (2 ワーカー、16GB) / `GPU_SINGLE` (8GB)   | 航空機搭載および野外展開における主な推奨構成 |
| **Jetson Orin Nano** | 8GB 共有     | `GPU_SINGLE` (1 ワーカー、順次処理)                     | エントリーレベルのエッジコンピューティング                                 |

{% hint style="info" %}
Linux

のarm64パッケージには**JetPack 6**が必要ですが、これはJetson Orinファミリーで利用可能です。旧モデル（Nano、TX2、Xavier NX）はJetPack 6を実行できず、現在のパッケージではサポートされていません。
{% endhint %}

***

## 要件

* **JetPack 6.x**（最新バージョンの使用を推奨）
* **NVIDIA CUDA**（JetPackに同梱）
* **有料のChloros

+プラン** — Copperティア以上（CLI

/SDK

へのアクセスにはすべて必須。サーバー側で強制適用されます）

## インストール

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Linux

の一般的なインストール手順、ファイルの保存場所、およびトラブルシューティングについては、[Linux

インストール](linux-installation.md)を参照してください。

{% hint style="info" %}
**解凍先ディレクトリは高速ストレージに設定してください。** コンパイル済みのバイナリは、起動のたびに一時ディレクトリに展開されますが、SDカードからの処理は極めて遅くなります。Chloros

は、`/mnt/ssd/tmp`が存在する場合、自動的にそれを使用します。存在しない場合は、`TMPDIR`をNVMe上のパスに設定してください （`export TMPDIR=/mnt/nvme/tmp`）のパスを設定してください。
{% endhint %}

***

## Jetson における動的演算適応

### 仕組み

起動時、Chloros

はシステムのプロファイリングを行います：

1. **`/proc/device-tree/model` を使用して Jetson モデルを検出**

2.**利用可能な共有 GPU/CPU メモリを読み取る**（Jetson はユニファイドメモリを使用）
3. **処理戦略を選択**します（`GPU_PARALLEL`、`GPU_SINGLE`、または `CPU_PARALLEL`）
4. **ワーカー数、パイプラインタイプ、およびメモリ割り当て**を自動的に設定します

この決定は、モデル名ではなく**共有RAMの合計容量**に基づいて行われます：

* **RAM合計が12GB未満の場合**（8GBのJetson全機種）：`GPU_SINGLE`で**ワーカー1つ — 意図的な順次処理**。 メモリが不足しているため、ワーカーを並行して実行できず、画像は1つずつ処理されます。**8GB以下**のJetsonでは、スレッド3はワーカープールを完全にスキップし、画像ごとの処理をプロセス内で実行します。
* **12GB以上**（Orin NX 16GB、AGX Orin）：ユニファイドメモリは`GPU_PARALLEL`の要件を満たしますが、**Jetsonではワーカー数が2に制限**されます。GPU、ワーカープロセスのRAM、 およびワーカーごとのCUDAコンテキストはすべて同じ共有プールから割り当てられるため、ワーカー数を増やすとメモリ不足による障害が発生するリスクが高まります。

環境変数 `CHLOROS_STRATEGY` を使用すると、この自動設定を上書きできます。詳細は [動的コンピュート適応](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override) を参照してください。

### モデルごとの動作

| Jetsonモデル                | 戦略       | ワーカー数 | 実行                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | シーケンシャルなインプロセスループ（メモリ不足時は `tiled_gpu`） |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | プロセス内順次ループ                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | 並行ワーカープロセス、`fused_gpu`パス  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | 並行ワーカープロセス、`fused_gpu` パス  |

プラットフォーム間の主な違いは**メモリ**です。8GBのJetsonは負荷が高い場合、メモリ効率に優れたタイル方式を用いて画像を1枚ずつ処理する必要がありますが、16GB以上のOrinは、高スループットの融合パイプラインを使用して、GPU上で2枚の画像を同時に処理できます。

### モデルごとのGPUバジェット

各Jetsonモデルには、共有プール処理が割り当てられる上限を規定し、バッチサイズをスケーリングするハードウェアプロファイルも設定されています：

| モデル | GPUバジェットの上限 | バッチサイズ乗数 | システム/ディスプレイ用に確保 |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0.8 | 2.0 GB |
| **Jetson Orin NX** | 75% | ×1.0 | 3.0 GB |
| **Jetson AGX Orin** | 80% | ×1.5 | 4.0 GB |

検出されたRAM容量に応じてプロファイルが調整されます。**16GB以上**を報告するJetsonの場合、バッチ乗数が×1.2に引き上げられます。乗数を適用する前の基本バッチサイズは8画像です。

演算適応に関する完全なリファレンスについては、[動的演算適応](../processing-architecture/dynamic-compute-adaptation.md)を参照してください。

***

## NanoおよびOrin NanoにおけるTexture AwareのGPU周波数制限

Texture Awareのデベイヤー処理ではGPUニューラルネットワーク推論が実行されますが、これにより、低消費電力Jetsonモデル（10～15Wクラス）において、GPUがフルクロック速度で動作している場合に**過電流警告**が発生する可能性があります。**Jetson Nano または Orin Nano** で Texture Aware 処理を実行する前に、Chloros

は GPU の最大周波数をチェックし、現在の周波数がそれより高い場合は **510 MHz** (510000000) に制限します：

*CLI

がGPU周波数のsysfsノードに書き込み可能な場合、制限は**自動的に適用**され、確認メッセージが表示されます。
* 書き込みができない場合（root権限が必要）、CLI

は上限を手動で適用するための正確な`sudo`コマンドを表示し、ユーザーがそれを確認できるようしばらく待機してから処理を続行します。処理自体は実行されますが、過電流警告が表示される場合があります。

処理の前に自分で制限を適用するには：

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

高電力モデル（Orin NX 25W、AGX Orin 60W）はフルGPU速度で動作し、制限は適用されません。Standardデベイヤーは、どのモデルでも制限をトリガーすることはありません。

{% hint style="info" %}
**Jetson における「Texture Aware」は、常に 1 画像ずつ処理されます。** 各ワーカーには、独自の CUDA コンテキスト（約 1GB）に加え、独自のノイズ除去モデルのコピーが必要になりますが、ユニファイドメモリではこれを賄うことができません。そのため、Jetson では Texture Aware パスは単一のワーカーに固定され、GPU アクセスはシリアル化されます。どの Jetson でも、Texture Aware は Standard よりも著しく遅くなると予想されます。
{% endhint %}

***

## 熱管理

Jetson デバイスは、特に密閉環境や航空機搭載環境において、熱的余裕が限られています。Chloros

は SoC の温度を監視し、バッチサイズを自動的に調整します：

| 温度         | 対応                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | 通常動作 — フル処理速度          |
| **70°C** (警告)  | バッチサイズが段階的に縮小 (70°C～80°Cの間で 100% → 50%) |
| **80°C** (危険) | 大幅なスロットリング (80°C～90°Cの間で50% → 0%) |
| **90°C** (シャットダウン) | GPU処理を完全に停止 — 冷却が必要 |

{% hint style="warning" %}
特に密閉された屋外設置環境や航空機搭載システムにおいて、継続的な処理を行うためには、**十分な換気と放熱を確保**してください。ハードウェアを保護するため、サーマルスロットリングにより処理スループットが低下します。
{% endhint %}

***

## メモリ管理

Jetsonデバイスは**ユニファイドメモリ**を採用しており、GPUとCPUが同じ物理RAMを共有します。表示されるVRAM（例：Orin NX 16GBで約15.3GB）はGPU専用のメモリではなく、オペレーティングシステムやその他のすべてのプロセスが使用しているのと同じRAMです。

### スワップに関する警告と推奨事項

Jetson上で処理を行う前に、CLI

は入力フォルダ内のRAW画像の数をカウントし（`.tif`、`.tiff`、`.raw`、 `.dng` — JPG プレビューはカウントされません）をカウントし、実行に必要なピークメモリ量を推定し、RAM とスワップの合計が不足する可能性がある場合は **開始前に警告** します。 警告メッセージの件名は「`LOW MEMORY WARNING - Jetson Detected`」となり、画像数、RAM、現在のスワップ容量、および推定ピーク値が表示された後、プロジェクトに合わせてサイズ設定された正確な「`fallocate`」／「`chmod`」／ `mkswap` / `swapon` コマンドを表示します（8GB未満になることはありません）。メッセージがスクロールバックに埋もれないよう数秒間一時停止した後、処理を続行します。**この警告で使用されるメモリ推定値：**

| デベイヤーモード | ベース | 画像あたり |
| --- | --- | --- |
| 標準 | ~1.5 GB | ~10 MB |
| テクスチャ対応 | ~2.5 GB (モデル +Python

実行時) | ~15 MB |

この警告は、推定ピーク値が「RAM ＋ スワップ領域」から 1 GB の安全マージンを差し引いた値を超えた場合に発動します。なお、計算には**ファイルベースの**スワップ領域のみがカウントされるため、zram のみの設定でも警告が表示されます。

スワップを手動で追加する場合（例：8GB）：

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

### OOM（メモリ不足）の処理

処理中、Chloros

はGPUメモリを監視し、クラッシュする代わりに段階的にパフォーマンスを低下させます：

1. GPUメモリ使用率が**85%**を超えると、予防的にバッチサイズが縮小されます
2. それでもメモリ不足が発生した場合、バッチサイズは**半減**され、その後もメモリ不足が発生するたびにさらに半減されます。その後のバッチ処理が成功するたびに、このペナルティは1段階ずつ緩和されます
3. 負荷が持続する場合、パイプラインは `fused_gpu` からメモリ効率の高い `tiled_gpu` パスへ切り替わり、最終手段として CPU 処理に移行します

***

## 現場での導入

### 消費電力の考慮事項

| Jetson モデル     | 標準消費電力 | 備考                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7～15W              | DCバレルジャック          |
| Jetson Orin NX   | 10～25W             | DCバレルジャック          |
| Jetson AGX Orin  | 15～60W             | USB-C PD またはバレルジャック |

持続的な処理のための電力予算を計画してください。ピーク時の消費電力は、GPUを多用するスレッド3（処理）の間に発生します。

### ストレージに関する推奨事項

* **NVMe SSD** は、arm64環境での導入に強く推奨されます
* SDカードは処理には速度が不十分です。ブートメディアとしてのみ使用してください
* 処理済み出力データには、生画像データの2～3倍の容量を確保してください

###SSH

によるヘッドレス運用

Chloros

CLI

は、ヘッドレスでの Jetson 導入に最適です:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### LATTICE / DAQ-E タイムシンク用の常時稼働バックエンド

JetsonがLATTICEカメラやDAQ-E光センサーをヘッドレスで制御する場合は、PTPグランドマスターが継続的に動作するように、バックエンドのsystemdサービスを有効にしてください（このユニットはインストールされていますが、デフォルトでは有効になっていません）：

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

詳細については、[Linux

インストール手順](linux-installation.md#always-on-ptp-for-headless-hosts) を参照してください。これには、root権限なしでPTPポート319/320をバインド可能にする方法も含まれています。

### systemd による自動処理

自動処理用の systemd サービスを作成します：

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

`chloros-cli process` は、プロダクトを要求した実行で画像が書き込まれなかった場合、0 以外の値で終了するため、systemd の失敗ステータスは監視において有意義です。

スケジュールされた処理を行うには、systemd タイマーと組み合わせて使用します：

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

### 基本的な Jetson 処理

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

### Jetson での「Python

」SDK



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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## フィールド使用におすすめの Jetson システム

フィールドおよび航空機搭載での展開には、以下の Jetson Orin NX 16GB キャリアボードのオプションをご検討ください：

* **航空機搭載／ドローン**：耐振動規格（MIL-STD）に準拠し、軽量（300g未満）、パッシブ冷却を採用したシステム
* **過酷な屋外環境**：IP67/IP69K防水エンクロージャーを備え、PoE GigEカメラ接続が可能なもの
* **最小構成／低コスト**：アドオンエンクロージャー付きのデベロッパーキット

導入シナリオに応じた具体的なハードウェアの推奨事項については、[MAPIR

サポート](https://www.mapir.camera/community/contact) までお問い合わせください。

***

## 次の手順

* [Linux

インストール](linux-installation.md) —Linux

の一般的なインストール詳細
* [動的コンピューティング適応](../processing-architecture/dynamic-compute-adaptation.md) — コンピューティング戦略の完全なリファレンス
* [処理パイプライン](../processing-architecture/processing-pipeline.md) — 4スレッドパイプラインの概要
* [CLI

: コマンドライン](../CLI.md) — 『CLI

』ガイド
* [API

:Python

SDK

](../api-python-sdk.md) — 『SDK

』ガイド
* [CLI

リファレンス](../reference/cli-reference.md) および [SDK

リファレンス](../reference/sdk-reference.md) — バージョン 1.2.0 のコマンドおよびAPI

の網羅的な一覧
