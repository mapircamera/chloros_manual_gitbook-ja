# Linux のインストール

Chloros は、Linux 向けに、CLI およびバックエンドをインストールする `.deb` パッケージとして配布されています。 Python SDK は、pip を使用して別途インストールされます。

***

## Linux amd64 (x86_64)

### システム要件

| 要件 | 最小 | 推奨 |
| --- | --- | --- |
| **ディストリビューション** | Ubuntu 20.04 以降 / Debian 11 以降 | Ubuntu 22.04 以降 |
| **プロセッサ** | x86_64 (Intel/AMD) | Intel Core i7 以上 |
| **メモリ (RAM)** | 8GB | 16GB以上 |
| **グラフィックカード** | 不要 (CPU処理) | 4GB以上のVRAMを搭載したNVIDIA GPU |
| **ストレージ** | 2GBの空き容量 | 10GB以上の空き容量があるSSD |
| **Python** | Python 3.7 以上 (SDK 用) | Python 3.10 以上 |

### インストール

`.deb` パッケージをダウンロードしてインストールします:

```bash
sudo dpkg -i chloros-amd64.deb
```

インストールを確認します:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### システム要件

| 要件 | 最小 | 推奨 |
| --- | --- | --- |
| **プラットフォーム** | JetPack 6 搭載の NVIDIA Jetson | Jetson Orin NX 16GB または AGX Orin |
| **JetPack** | JetPack 6.x | 最新の JetPack 6 |
| **メモリ (RAM)** | 8GB (GPU/CPU 共有) | 16GB 以上 (共有) |
| **ストレージ** | 2GB の空き容量 | 10GB 以上の空き容量がある NVMe SSD |
| **Python** | Python 3.7 以上 (SDK用) | Python 3.10以上 |

### インストール

JetPack 6 `.deb` パッケージをダウンロードしてインストールします:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

インストールを確認してください：

```bash
chloros-cli --version
```

熱管理や現場展開を含む詳細な Jetson のセットアップについては、[NVIDIA Jetson ガイド](nvidia-jetson-guide.md)を参照してください。

***

## Python SDK のインストール (すべて Linux)

Python SDK は pip 経由で個別にインストールされ、amd64 および arm64 の両方で動作します:

```bash
pip install chloros-sdk
```

オプションの進行状況ストリーミング機能を有効にするには：

```bash
pip install chloros-sdk[progress]
```

SDK を確認してください：

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` パッケージは、Chloros、CLI、およびバックエンドをインストールします。 Python SDK は、ローカルの HTTP API を介してバックエンドと通信する、独立した pip パッケージです。
{% endhint %}

***

## 設定ディレクトリ

Chloros 上の Linux は、[XDG ベースディレクトリ仕様](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) に準拠しています：

| 目的 | Linux パス | Windows 相当 |
| --- | --- | --- |
| **設定** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **データ / プロジェクト** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **キャッシュ / 認証情報** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## バックエンド実行ファイルの場所

`.deb` パッケージは、バックエンドを標準の場所にインストールします。 CLI および SDK は、バックエンドのパスを自動検出します：

| インストール方法 | バックエンドのパス |
| --- | --- |
| `.deb` パッケージ | `/usr/lib/chloros/chloros-backend` |
| 手動 / カスタム | `/opt/mapir/chloros/backend/chloros-backend` |

`--backend-exe` および CLI フラグ、または `backend_exe` および SDK コンストラクタパラメータを使用して、バックエンドのパスを上書きできます。

***

## 初回セットアップ

### 1. ライセンスの有効化

Chloros+ ライセンスは、CLI および SDK へのアクセスに必要です：

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. ライセンスの状態を確認する

```bash
chloros-cli status
```

### 3. 最初のデータセットを処理する

```bash
chloros-cli process ~/datasets/flight001
```

### 4. システム診断を実行する

システムが正しく設定されていることを確認してください：

```bash
chloros-cli selftest
```

これにより、バージョン、バックエンドの起動、API接続、およびCUDA/GPUの利用可能性を含む7つの診断チェックが実行されます。

***

## Bashスクリプトの例

### 複数のデータセットを処理する

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### カスタム設定で処理する

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Cron を使用した自動処理

新しいデータセットを自動的に処理するために、crontab に以下を追加してください (`crontab -e`):

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK の例

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## トラブルシューティング

### CLI インストール後に見つからない

`chloros-cli` パッケージをインストールした後、`.deb` が見つからない場合：

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### アクセス拒否

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### バックエンドの起動に失敗

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA が検出されません

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### 共有ライブラリが見つかりません

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Linux 上の Chloros の更新

組み込みの update コマンドを使用して、更新を確認しインストールしてください:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## 次の手順

* [NVIDIA Jetson ガイド](nvidia-jetson-guide.md) — Jetson 向けの最適化とデプロイ
* [CLI : コマンドライン](../CLI.md) — CLI コマンドの完全なリファレンス
* [API : Python SDK](../api-python-sdk.md) — SDK の完全なリファレンス
* [動的演算適応](../processing-architecture/dynamic-compute-adaptation.md) — Chloros がハードウェアにどのように適応するか
