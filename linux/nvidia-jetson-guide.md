# NVIDIA Jetson ガイドNVIDIA Jetson 上の

Chlorosにより、現場、UAV、遠隔地での設置環境など、エッジ環境でのマルチスペクトル画像処理が可能になります。Chloros 1.3.0 は起動時に Jetson モデルを検出し、検出したハードウェアに合わせて処理戦略を最適化します。**手動での調整は不要です。**
{% hint style="info" %}
**

1.3.0 版の Jetson パッケージは、Jetson の GPU 上で処理を行います。** `chloros_1.3.0_arm64_jp6.deb`はJetPack 6向けに構築されており、JetPack 6のCUDA版PyTorchを使用するため、処理（テクスチャ対応デベイヤーを含む）はOrinのGPU上で実行されます。 このパッケージ自体に CUDA は含まれていません。JetPack 6 がインストールする CUDA および cuDNN ライブラリを使用します（これらがないイメージ上では `sudo apt install nvidia-jetpack` が使用されます）。CUDA GPU が利用できない場合、処理は CPU 上で実行され、Texture Aware は Standard デベイヤーを使用します。
{% endhint %}

***

## 対応するJetsonモデル

| モデル                | RAM            | 処理戦略                                     | 推奨用途                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32～64GB 共有 | `GPU_PARALLEL` (2 ワーカー)                              | 最高のパフォーマンス、大規模なデータセット                      |
| **Jetson Orin NX**   | 8～16GB 共有  | `GPU_PARALLEL` (2 ワーカー、16GB) / `GPU_SINGLE` (8GB)   | フィールド展開における主な推奨構成              |
| **Jetson Orin Nano** | 8GB 共有     | `GPU_SINGLE` (1 ワーカー、順次実行)                     | エントリーレベルのエッジコンピューティング                                 |

{% hint style="info" %}
Linux arm64 パッケージには **JetPack 6** が必要ですが、これは Jetson Orin ファミリーで利用可能です。旧モデル（Nano、TX2、Xavier NX）では JetPack 6 を実行できず、現在のパッケージではサポートされていません。
{% endhint %}

***

## 要件

* **JetPack 6.x**（最新バージョンを推奨）
* **JetPack 6に含まれるNVIDIA CUDAおよびcuDNN**（イメージにこれらが含まれていない場合は`sudo apt install nvidia-jetpack`）
* **有料のChloros+プラン** — Copperティア以上（すべてのCLI/SDKアクセスに必須；サーバー側で強制適用）

## インストール

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` は、このビルドに対応するシステム `python3` 用の Python SDKをインストールします。 バンドルされたwheelをインストールする必要があるのは、virtualenvのみです：`pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`（完全オフライン）。一般的なLinuxのインストール詳細、ファイルの場所、およびトラブルシューティングについては、[Linuxのインストール](linux-installation.md)を参照してください。

{% hint style="info" %}
**起動時の解凍は不要になりました。** バージョン 1.3.0 以降、バックエンドとCLIは `/usr/lib/chloros/` 配下に解凍済みでインストールされるため、起動のたびに一時ディレクトリへ自動的に解凍されることはなくなりました。 一時ファイルは、`/mnt/ssd/tmp` フォルダが存在する場合、引き続き自動的にそこに保存されます。存在しない場合は、`TMPDIR` を NVMe 上のパス（`export TMPDIR=/mnt/nvme/tmp`）に設定できます。
{% endhint %}

***

## Jetson における動的演算適応

### 仕組み

起動時、Chlorosはシステムのプロファイルを作成します：

1. **`/proc/device-tree/model` を通じて Jetson モデルを検出**

2.**利用可能な共有 GPU/CPU メモリを読み取ります**（Jetson はユニファイドメモリを使用しています）
3. **PyTorch に CUDA を使用できるかどうかを確認します** — JetPack 6 の CUDA および cuDNN がインストールされていれば、使用可能です
4. **処理戦略**（`GPU_PARALLEL`、`GPU_SINGLE`、または`CPU_PARALLEL`）とワーカー数を自動的に選択します

この決定は、モデル名ではなく**合計共有RAM**に基づいて行われます：

* **合計RAMが12GB未満の場合**（8GBのJetson全機種）：**ワーカー1つ — 意図的な順次処理**。メモリが不足しているため、複数のワーカーによる並列処理は行えず、画像は1枚ずつ処理されます。**8GB以下**のJetsonでは、スレッド3はワーカープールを完全にスキップし、画像ごとの処理をインプロセスで実行します。バックエンドログでは、この戦略を`GPU_SINGLE`と呼んでいます。
* **12GB以上**（Orin NX 16GB、AGX Orin）：統合メモリは`GPU_PARALLEL`の条件を満たしますが、**Jetsonではワーカー数が2に制限されます** — GPU、ワーカープロセスのRAM、およびワーカーごとのCUDAコンテキストはすべて同じ共有プールから割り当てられるため、ワーカー数を増やすとメモリ不足による障害が発生するリスクが高まります。
* **CUDAが使用不可**（JetPackのCUDAライブラリが欠落）：12GB以上の場合、`CPU_PARALLEL`が適用され、それ未満の場合は1つのイメージごとに適用されます。

`CHLOROS_STRATEGY` 環境変数を使用して、自動選択を上書きできます — [動的演算適応](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override) を参照してください。

### モデルごとの動作

| Jetson モデル                | 戦略       | ワーカー数 | 実行                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | 順次インプロセスループ（メモリ負荷が高い場合は `tiled_gpu`） |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | プロセス内順次ループ                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | 並行ワーカープロセス、`fused_gpu` パス  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | 並行ワーカープロセス、`fused_gpu` パス  |

プラットフォーム間の主な違いは**メモリ**です。8GBのJetsonは負荷が高い場合、メモリ効率に優れたタイル処理方式を用いて画像を1枚ずつ処理する必要がありますが、16GB以上のOrinは、高スループットの融合パイプラインを使用して、GPU上で2枚の画像を同時に処理できます。

### モデルごとのGPU割り当て

各Jetsonモデルには、共有プール処理が消費できる上限を規定し、バッチサイズをスケーリングするハードウェアプロファイルも設定されています：

| モデル | GPU割り当て上限 | バッチサイズ乗数 | システム/ディスプレイ用に予約 |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0.8 | 2.0 GB |
| **Jetson Orin NX** | 75% | ×1.0 | 3.0 GB |
| **Jetson AGX Orin** | 80% | ×1.5 | 4.0 GB |

検出されたRAM容量に応じてプロファイルが調整されます。**16GB以上**を報告するJetsonの場合、バッチ乗数が×1.2に引き上げられます。乗数を適用する前の基本バッチサイズは8画像です。

コンピュート適応に関する完全なリファレンスについては、[動的コンピュート適応](../processing-architecture/dynamic-compute-adaptation.md)を参照してください。

***

## NanoおよびOrin NanoにおけるTexture AwareのGPU周波数制限

Texture AwareのデベイヤーはGPUニューラルネットワーク推論を実行しますが、これにより、低消費電力Jetsonモデル（10～15Wクラス）において、GPUクロック速度が最大値に達すると**過電流警告**が発生する可能性があります。**Jetson NanoまたはOrin Nano**で`--debayer texture-aware`を指定すると、CLIはGPUの最大周波数をチェックし、現在の周波数がそれより高い場合は**510 MHz**（510000000）に制限します：

* CLIがGPU周波数のsysfsノードに書き込み可能な場合、制限は**自動的に適用**され、確認メッセージが表示されます。
* 書き込みができない場合（root権限が必要）、CLIは上限を手動で適用するための正確な`sudo`コマンドを表示し、ユーザーがそれを確認できるよう少し待機してから処理を続行します。処理自体は実行されますが、過電流警告が表示される場合があります。

処理の前に自分で上限を適用するには：

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

高電力モデル（Orin NX 25W、AGX Orin 60W）は GPU フルスピードで動作し、上限は適用されません。Standard デベイヤーは、どのモデルでも上限をトリガーすることはありません。

{% hint style="info" %}
**Jetson での Texture Aware は、常に 1 枚の画像ずつ処理されます。** 各ワーカーには、独自の CUDA コンテキスト（約 1GB）に加え、独自のノイズ除去モデルのコピーが必要となりますが、ユニファイドメモリではこれを賄うことができません。そのため、Jetson では Texture Aware パスは単一のワーカーに固定され、GPU アクセスはシリアル化されます。どの Jetson でも、Texture Aware は Standard よりも著しく遅くなると予想されます。 Texture AwareはSurvey3画像に適用されます。LATTICEでキャプチャされた画像には、常にStandardデベイヤーが適用されます。使用可能なCUDA GPUがない場合、処理はStandardモードで行われ、バックエンドにはその理由を説明する`[DEBAYER] … needs a CUDA GPU …`行が1行ログに記録されます（`chloros-cli --verbose`で表示）。
{% endhint %}

***

## 熱管理

Jetson デバイスは、特に密閉環境や航空機搭載環境において、熱的余裕が限られています。Chlorosは SoC の温度を監視し、バッチサイズを自動的に調整します：

| 温度         | アクション                                            |
| ------------------- | ------------------------------------------------- |
| **< 70°C**          | 通常動作 — フル処理速度          |
| **70°C** (警告)  | バッチサイズが段階的に縮小（70°C～80°Cの間で100% → 50%） |
| **80°C** (危険) | 大幅なスロットリング (80°C～90°Cの間で50% → 0%) |
| **90°C** (シャットダウン) | GPU処理を完全に停止 — 冷却が必要 |

{% hint style="warning" %}
特に密閉されたフィールドエンクロージャーや航空機搭載システムでは、持続的な処理を行うために**十分な換気と放熱を確保**してください。ハードウェアを保護するため、サーマルスロットリングにより処理スループットが低下します。
{% endhint %}

***

## メモリ管理

Jetsonデバイスは**ユニファイドメモリ**を採用しており、GPUとCPUが同じ物理RAMを共有します。表示されるVRAM（例：Orin NX 16GBで約15.3GB）はGPU専用のメモリではなく、オペレーティングシステムやその他のすべてのプロセスが使用しているのと同じRAMです。

### スワップに関する警告と推奨事項

Jetsonで処理を行う前に、CLIは入力フォルダ内のRAW画像の数をカウントします （`.tif`、`.tiff`、`.raw`、`.dng` — JPGプレビューはカウントされません）をカウントし、実行に必要なピークメモリを推定し、RAMとスワップの合計が不足する可能性がある場合は**開始前に警告**を表示します。警告の件名は「`LOW MEMORY WARNING - Jetson Detected`」となり、画像数、 RAM、現在のスワップ容量、および推定ピーク値を表示した後、**プロジェクトに合わせてサイズ調整された**正確な `fallocate` / `chmod` / `mkswap` / `swapon` コマンドを出力します （8GBを下回ることはありません）。メッセージがスクロールバックに埋もれないよう数秒間一時停止した後、処理が再開されます。**警告で使用されるメモリ推定値：**

| デベイヤーモード | ベース | 画像あたり |
| --- | --- | --- |
| 標準 | ~1.5 GB | ~10 MB |
| テクスチャ対応 | ~2.5 GB (モデル + Python ランタイム) | ~15 MB |

この警告は、推定ピーク値が「RAM + スワップ」から1GBの安全マージンを差し引いた値を超えた場合に発動します。なお、計算には**ファイルベースの**スワップのみがカウントされるため、zramのみの設定でも警告が表示されます。

スワップを手動で追加するには（例：8GB）：

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

処理中、ChlorosはGPUメモリを監視し、クラッシュする代わりに段階的に処理を縮小します：

1. GPUメモリ使用率が**85%**を超えると、予防的にバッチサイズが縮小されます
2. それでもメモリ不足が発生した場合、バッチサイズは**半減**され、その後もメモリ不足が発生するたびにさらに半減されます。その後、バッチ処理が正常に完了するたびに、このペナルティは1段階ずつ緩和されます
3. 負荷が持続する場合、パイプラインは `fused_gpu` からメモリ効率の高い `tiled_gpu` パスへ切り替わり、最終手段として CPU 処理に移行します

GPUはJetsonのRAMを共有しているため、実行にはRAMとスワップ領域の合計容量が上限となります。上記のスワップ警告は、注意すべきチェック項目です。

***

## 現場での導入

### 電力に関する考慮事項

| Jetsonモデル     | 標準的な消費電力 | 備考                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7～15W              | DCバレルジャック          |
| Jetson Orin NX   | 10～25W             | DCバレルジャック          |
| Jetson AGX Orin  | 15～60W             | USB-C PD またはバレルジャック |

持続的な処理のための電力予算を計画してください。ピーク時の消費電力は、GPUを多用するスレッド 3（処理）の実行中に発生します。

### ストレージに関する推奨事項

* **NVMe SSD** は、arm64 環境での導入に強く推奨されます
* SD カードは処理には速度が不十分です。ブートメディアとしてのみ使用してください
* 処理済み出力データには、生画像データの2～3倍の容量を見込んでください

### SSH
によるヘッドレス運用

Chloros CLIは、ヘッドレスでの Jetson 導入に最適です：

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### LATTICE / DAQ-E タイムシンク用の常時稼働バックエンド

JetsonがLATTICEカメラやDAQ-E光センサーをヘッドレスで制御する場合、**自身のユーザーとして**バックエンドのsystemdサービスを有効にしてください。これにより、PTPグランドマスターが継続的に動作し、SDKスクリプトが実行中のバックエンドを検出できるようになります（このユニットはインストールされていますが、デフォルトでは有効になっていません）：

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

これにより、バックエンドはあなたとして実行されます。つまり、バックエンドが書き込むキャプチャデータはあなたのものとなり、あなたの `chloros-cli login` が使用されます。 詳細については、[Linuxインストール](linux-installation.md#always-on-ptp-for-headless-hosts)を参照してください。これには、root `chloros-backend.service` や、パッケージが root 権限なしで PTP ポート 319/320 をバインド可能にする方法などが含まれています。

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

`chloros-cli process` は、製品の書き込みを要求した実行で画像が書き込まれなかった場合、0 以外の終了コードを返します。そのため、systemd の失敗ステータスは監視において有意義な情報となります。

スケジュールされた処理を行うには、systemdタイマーと組み合わせて使用します：

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
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### Jetson 上の Python SDK


```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## 現場での使用に推奨されるJetsonシステム

現場や航空機への搭載には、以下のJetson Orin NX 16GBキャリアボードのオプションをご検討ください：

* **航空機/ドローン**：耐振動規格（MIL-STD）に準拠し、軽量（300g未満）、パッシブ冷却を採用したシステム
* **過酷な野外環境**：IP67/IP69K 防水エンクロージャーを備え、PoE GigE カメラ接続が可能なもの
* **最小構成／低コスト**：アドオンエンクロージャー付きのデベロッパーキット

導入シナリオに応じた具体的なハードウェアの推奨事項については、[MAPIRサポート](https://www.mapir.camera/community/contact)までお問い合わせください。

***

## 次の手順

* [Linuxのインストール](linux-installation.md) — Linuxのインストールに関する一般的な詳細
* [動的演算適応](../processing-architecture/dynamic-compute-adaptation.md) — 演算戦略の完全なリファレンス
* [処理パイプライン](../processing-architecture/processing-pipeline.md) — 4スレッドパイプラインの理解
* [CLI : コマンドライン](../CLI.md) — CLIガイド
* [API: Python SDK](../api-python-sdk.md) — SDKガイド
* [CLIリファレンス](../reference/cli-reference.md) および [SDKリファレンス](../reference/sdk-reference.md) — バージョン1.3.0のコマンドおよびAPIの完全な一覧
