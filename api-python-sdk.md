# API : Python SDK

**Chloros Python SDK** は、Chloros画像処理エンジンへのプログラムによるアクセスを提供し、自動化、カスタムワークフロー、およびPythonアプリケーションや研究パイプラインとのシームレスな統合を可能にします。

### 主な機能

* 🐍 **ネイティブ Python** - 画像処理のためのクリーンでPythonらしい API
* 🔧 **完全な API アクセス** - Chloros 処理を完全に制御
* 🚀 **自動化** - カスタムバッチ処理ワークフローの構築
* 🔗 **統合** - 既存のアプリケーションへの組み込み
* 📊 **研究用途に対応** - 科学的な分析パイプラインに最適
* ⚡ **並列処理** - CPUコア数に応じたスケーリング (Chloros+)

### 要件

| 要件          | 詳細                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros がインストール済み** | Windows: デスクトップインストーラー; Linux: `.deb` パッケージ                  |
| **ライセンス**          | Chloros+ ([有料プランが必要](https://cloud.mapir.camera/pricing)) |
| **オペレーティングシステム** | Windows 10/11 (64ビット)、Linux x86_64 (amd64)、Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 以降                                                |
| **メモリ**           | 8GB RAM 以上 (16GB 推奨)                                  |
| **インターネット**         | ライセンスのアクティベーションに必要                                     |

{% hint style="warning" %}
**ライセンス要件**: Python SDK を利用するには、API へのアクセス権を含む有料の Chloros+ サブスクリプションが必要です。 スタンダード（無料）プランでは、API/SDKにアクセスできません。アップグレードするには、[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)をご覧ください。
{% endhint %}

## クイックスタート

### インストール

pip 経由でインストール:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**初回セットアップ**: SDKを使用する前に、Chloros+ライセンスを有効化してください。これを行うには、Chloros、Chloros （ブラウザ）または Chloros CLI を開き、認証情報でログインして、Chloros+ のライセンスを有効化してください。これは一度だけ行う必要があります。Linux（GUIなし）では、以下を使用してください：`chloros-cli login user@example.com 'password'`
{% endhint %}

### 基本的な使い方

数行のコードでフォルダを処理します:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**クロスプラットフォームのパス**：このページのコード例では、Windows形式のパス（例：`C:\\DroneImages\\Flight001`）を使用しています。 Linuxでは、代わりにLinux形式のパスを使用してください（例：`/home/user/drone_images/flight001`または`~/drone_images/flight001`）。SDKは、両プラットフォームで同じように動作します。
{% endhint %}

### 完全制御

高度なワークフローの場合：

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## インストールガイド

### 前提条件

SDKをインストールする前に、以下がインストールされていることを確認してください：

1. **Chloros がインストール済み** — Windows: デスクトップインストーラー ([ダウンロード](download.md)); Linux: `.deb` パッケージ ([Linux インストール](linux/linux-installation.md))
2. **Python 3.7+** がインストール済み ([python.org](https://www.python.org))
3. **有効な Chloros+ ライセンス** ([アップグレード](https://cloud.mapir.camera/pricing))

### pip によるインストール

**標準インストール:**

```bash
pip install chloros-sdk
```

**進行状況の監視機能付き:**

```bash
pip install chloros-sdk[progress]
```

**開発用インストール:**

```bash
pip install chloros-sdk[dev]
```

### インストールの確認

SDKが正しくインストールされているかテストします:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## 初回セットアップ

### ライセンスのアクティベーション

SDKは、Chloros、Chloros（ブラウザ）、およびChloros CLIと同じライセンスを使用します。 GUI または CLI を使用して一度アクティベーションを行ってください：

**Windows:** **Chloros または Chloros (Browser)** を開き、[ユーザー] <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> タブでログインするか、CLIを使用してください。**Linux:** CLIを使用してください（GUIは利用不可）：

```bash
chloros-cli login user@example.com 'your_password'
```

ライセンスはローカルにキャッシュされ、再起動後も保持されます。

{% hint style="success" %}
**初回設定**: GUIまたはCLI経由でログインした後、SDKは自動的にキャッシュされたライセンスを使用します。追加の認証は不要です！
{% endhint %}

{% hint style="info" %}
**ログアウト**: SDK ユーザーは、`logout()` メソッドを使用して、キャッシュされた認証情報をプログラムでクリアできます。 APIリファレンスの[logout()メソッド](#logout)を参照してください。
{% endhint %}

### 接続テスト

SDKがChlorosに接続できることを確認します:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API リファレンス

### ChlorosLocal クラス

ローカル Chloros 画像処理のメインクラス。

#### コンストラクタ

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**パラメータ:**

| パラメータ                 | 型 | デフォルト                   | 説明                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | ローカル Chloros バックエンドの URL          |
| `auto_start_backend`      | bool | `True`                    | 必要に応じてバックエンドを自動的に起動する |
| `backend_exe`             | str  | `None` (自動検出)      | バックエンド実行ファイルへのパス            |
| `timeout`                 | int  | `30`                      | リクエストタイムアウト（秒）            |
| `backend_startup_timeout` | int  | `60`                      | バックエンド起動のタイムアウト（秒） |

**例:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**クロスプラットフォーム自動検出**: SDKは、お使いのプラットフォームに適したバックエンドパスを自動的に試行します:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (手動)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### メソッド

#### `create_project(project_name, camera=None)`

新しい Chloros プロジェクトを作成します。

**パラメータ:**

| パラメータ      | 型 | 必須 | 説明                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | はい      | プロジェクト名                                     |
| `camera`       | str  | いいえ       | カメラテンプレート（例：「Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**戻り値:** `dict` - プロジェクト作成レスポンス**例:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

フォルダから画像をインポートします。

**パラメータ:**

| パラメータ     | 型     | 必須 | 説明                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | はい      | 画像が含まれるフォルダのパス         |
| `recursive`   | bool     | いいえ       | サブフォルダを検索する (デフォルト: False) |

**戻り値:** `dict` - ファイル数を伴うインポート結果**例:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

処理設定を構成します。

**パラメータ:**

| パラメータ                 | 型 | デフォルト                 | 説明                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Standard (Fast, Medium Quality)&quot; | デベイヤー処理方法            |
| `vignette_correction`     | bool | `True`                  | ヴィネット補正を有効にする      |
| `reflectance_calibration` | bool | `True`                  | 反射率キャリブレーションを有効にする  |
| `indices`                 | リスト | `None`                  | 算出する植生指数 |
| `export_format`           | 文字列 | &quot;TIFF (16ビット)&quot;         | 出力形式                   |
| `ppk`                     | bool | `False`                 | PPK補正を有効にする          |
| `custom_settings`         | dict | `None`                  | 詳細なカスタム設定        |

**エクスポート形式:**

* `"TIFF (16-bit)"` - GIS/写真測量に推奨
* `"TIFF (32-bit, Percent)"` - 科学分析
* `"PNG (8-bit)"` - 目視検査
* `"JPG (8-bit)"` - 圧縮出力

**利用可能なインデックス:**NDVI、NDRE、GNDVI、OSAVI、CIG、EVI、SAVI、 MSAVI、MTVI2、その他。**例:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

プロジェクト画像を処理します。

**パラメータ:**

| パラメータ           | タイプ     | デフォルト      | 説明                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | 処理モード: &quot;parallel&quot; または &quot;serial&quot;   |
| `wait`              | bool     | `True`       | 完了を待機する                       |
| `progress_callback` | callable | `None`       | 進行状況コールバック関数(progress, msg) |
| `poll_interval`     | float    | `2.0`        | 進行状況のポーリング間隔 (秒)   |

**戻り値:** `dict` - 処理結果

{% hint style="warning" %}
**並列モード**: Chloros+ ライセンスが必要です。CPU コア数に応じて自動的にスケールします（最大 16 ワーカー）。
{% endhint %}

**例:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

現在のプロジェクト設定を取得します。

**戻り値:** `dict` - 現在のプロジェクト設定**例:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

スレッドごとの処理進捗を含むバックエンドのステータス情報を取得します。

**戻り値:** `dict` - 以下の構造を持つバックエンドのステータス:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**例:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

バックエンドをシャットダウンします（SDKによって起動されている場合）。

**例:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

ローカルシステムからキャッシュされた認証情報をクリアします。

**説明:**

キャッシュされた認証情報を削除することで、プログラム的にログアウトします。これは以下の場合に役立ちます:
* 異なる Chloros+ アカウント間の切り替え
* 自動化された環境での認証情報のクリア
* セキュリティ上の目的（例：アンインストール前の認証情報の削除）

**戻り値:** `dict` - ログアウト操作の結果**例:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**再認証が必要**: `logout()`を呼び出した後、Chloros、Chloros （ブラウザ）、または Chloros CLI 経由で再度ログインする必要があります。
{% endhint %}

***

### 便利機能

#### `process_folder(folder_path, **options)`

フォルダを処理するためのワンラインの便利機能。

**パラメータ:**

| パラメータ                 | 型     | デフォルト         | 説明                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | 必須        | 画像が含まれるフォルダのパス     |
| `project_name`            | str      | 自動生成  | プロジェクト名                   |
| `camera`                  | str      | `None`          | カメラテンプレート                |
| `indices`                 | リスト     | `["NDVI"]`      | 計算するインデックス           |
| `vignette_correction`     | bool     | `True`          | ヴィネット補正を有効にする     |
| `reflectance_calibration` | bool     | `True`          | 反射率キャリブレーションを有効にする |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | 出力形式                  |
| `mode`                    | str      | `"parallel"`    | 処理モード                |
| `progress_callback`       | 呼び出し可能 | `None`          | 進行状況コールバック              |

**戻り値:** `dict` - 処理結果**例:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## コンテキストマネージャのサポート

SDKは、自動クリーンアップのためのコンテキストマネージャをサポートしています:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## 完全な例

{% hint style="info" %}
**Linux ユーザー**: 以下のすべての例では、Windowsのパスを使用しています。`C:\\...`のパスを、ご自身のLinuxのパス（例：`/home/user/...`または`~/...`）に置き換えてください。 SDKの機能は、すべてのプラットフォームで同一です。
{% endhint %}

### 例 1: 基本処理

デフォルト設定でフォルダを処理します:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### 例 2: カスタムワークフロー

処理パイプラインを完全に制御します:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### 例 3: 複数のフォルダのバッチ処理

複数のフライトデータセットを処理します:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### 例 4: 研究パイプラインへの統合

Chlorosをデータ分析に統合します:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### 例 5: カスタム進捗モニタリング

ロギングによる高度な進捗追跡:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### 例 6: エラー処理

本番環境向けの堅牢なエラー処理:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### 例 7: アカウント管理とログアウト

プログラムによる認証情報の管理:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### 例 8: コマンドラインツール

SDK を使用したカスタム CLI ツールの構築:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**使用方法:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## 例外処理

SDKは、さまざまなエラータイプに対応した特定の例外クラスを提供します:

### 例外階層

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### 例外の例

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## 応用トピック

### カスタムバックエンドの設定

カスタムバックエンドの場所または設定を使用します:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### ノンブロッキング処理

処理を開始し、他のタスクを継続します:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### メモリ管理

大規模なデータセットの場合は、バッチ処理を行います:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## トラブルシューティング

### バックエンドが起動しない

**問題:** SDK がバックエンドの起動に失敗する**解決策:**

1. Chloros がインストールされているか確認する:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. ファイアウォール (Windows) またはポートの可用性を確認する (Linux: `lsof -i :5000`)
3. バックエンドのパスを手動で指定して試す:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### ライセンスが検出されません**問題:** SDK がライセンスの欠落について警告を表示します**解決策:**

1. Chloros、Chloros (ブラウザ) または Chloros CLI を開き、ログインしてください。
2. ライセンスがキャッシュされていることを確認します：

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. 認証情報に関する問題が発生している場合は、キャッシュされた認証情報を消去して再ログインします：

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. サポートにお問い合わせください：info@mapir.camera

***

### インポートエラー**問題:** `ModuleNotFoundError: No module named 'chloros_sdk'`**解決策:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### 処理タイムアウト**問題:** 処理がタイムアウトする**解決策:**

1. タイムアウト時間を延長する:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. 処理バッチを小さくする
3. 空きディスク容量を確認する
4. システムリソースを監視する

***

### ポートが既に使用中**問題:** バックエンドポート 5000 が使用中**解決策:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

または、競合しているプロセスを見つけて終了させる:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## パフォーマンスのヒント

### 処理速度の最適化

1. **並列モードを使用する** (Chloros以上が必要)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **出力解像度を下げる**（許容できる場合）

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **不要なインデックスを無効にする**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **SSD上で処理する**（HDDではなく）***

### メモリの最適化

大規模なデータセットの場合：

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### バックグラウンド処理

他のタスクのためにリソースを解放する:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## 統合例

### Django 統合

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## よくある質問

### Q: SDKはインターネット接続が必要ですか？

**A:** 初期のライセンス有効化時のみ必要です。Chloros、Chloros（ブラウザ）、またはChloros CLI経由でログインすると、ライセンスはローカルにキャッシュされ、30日間オフラインで動作します。***

### Q: GUIのないサーバーでSDKを使用できますか？**A:** はい！ SDKは、WindowsおよびLinuxサーバーの両方でヘッドレス動作します。**Linux（ヘッドレス環境に推奨）：**
* `.deb`パッケージ経由でインストール
* ライセンスの有効化：`chloros-cli login user@example.com 'password'`

**Windows サーバー：**
* Windows Server 2016 以降
* Chloros をインストール（1回のみ）
* CLI または任意のマシンでライセンスを有効化

***

### Q: Desktop、CLI、および SDK の違いは何ですか？

| 機能         | デスクトップ GUI | CLI コマンドライン | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **インターフェース**   | ポイント＆クリック | コマンドライン | Python API  |
| **最適用途**    | ビジュアル作業 | スクリプト作成 | 統合 |
| **自動化**   | 限定的     | 良好             | 優秀   |
| **柔軟性** | 基本       | 良好             | 最高     |
| **ライセンス**     | Chloros+    | Chloros+         | Chloros+    |***

### Q: SDKで構築したアプリを配布することはできますか？**A:** SDKのコードをアプリケーションに組み込むことは可能ですが、以下の条件があります：

* エンドユーザーは Chloros をインストールしている必要があります
* エンドユーザーは有効な Chloros+ ライセンスを保有している必要があります
* 商用配布には OEM ライセンスが必要です

OEM に関するお問い合わせは、info@mapir.camera までご連絡ください。

***

### Q: SDK を更新するにはどうすればよいですか？

```bash
pip install --upgrade chloros-sdk
```

***

### Q: 処理済みの画像はどこに保存されますか？

デフォルトでは、プロジェクトパス内に保存されます：

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Q: スケジュールに従って実行されるPythonスクリプトから画像を処理することはできますか？**A:** はい！ OSのスケジューラを使用して、Pythonスクリプトを実行してください：

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** タスクスケジューラを使用して毎日実行するようにスケジュールします。**Linux:** cronを使用してスケジュールします：

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### Q: SDKはasync/awaitをサポートしていますか？**A:** 現在のバージョンは同期処理です。非同期動作が必要な場合は、`wait=False`を使用するか、別スレッドで実行してください：

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### Q: 異なる Chloros+ アカウントを切り替えるにはどうすればよいですか？**A:** `logout()` メソッドを使用してキャッシュされた認証情報をクリアし、新しいアカウントで再ログインしてください:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

ログアウト後、CLIを再度使用する前に、GUI、ブラウザ、またはSDK経由で新しいアカウントで認証を行ってください。

***

## ヘルプ

### ドキュメント

* **API リファレンス**: このページ

### サポートチャネル

* **メール**: info@mapir.camera
* **ウェブサイト**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **価格**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### サンプルコード

ここに掲載されているすべてのサンプルはテスト済みで、本番環境での使用が可能です。ご自身のユースケースに合わせてコピーして修正してください。

***

## ライセンス**プロプライエタリソフトウェア** - Copyright (c) 2025 MAPIR Inc.

SDK をご利用いただくには、有効な Chloros+ サブスクリプションが必要です。無断での使用、配布、または改変は禁止されています。
