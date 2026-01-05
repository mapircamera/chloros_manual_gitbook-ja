# API : Python SDK

**Chloros Python SDK** は、Chloros画像処理エンジンへのプログラムによるアクセスを提供し、自動化、カスタムワークフロー、およびPythonアプリケーションや研究パイプラインとのシームレスな統合を可能にします。

### 主な機能

* 🐍 **ネイティブPython** - 画像処理のためのクリーンでPythonicなAPI
* 🔧 **完全なAPIアクセス** - Chloros処理の完全な制御
* 🚀 **自動化** - カスタムバッチ処理ワークフローの構築
* 🔗 **統合** - 既存アプリケーションへのChloros組み込み
* 📊 **研究対応** - 科学分析パイプラインに最適
* ⚡ **並列処理** - CPUコア数に応じたスケーリング (Chloros+)

### 要件

| 要件                         | 詳細                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | ローカルにインストール済みであること                                           |
| **ライセンス**          | Chloros+ ([有料プランが必要](https://cloud.mapir.camera/pricing)) |
| **オペレーティングシステム** | Windows 10/11 (64ビット)                                              |
| **XPROTX**           | XPROTX 3.7 以上                                                                        |
| **メモリ**           | 8GB RAM以上必須 (16GB推奨)                                  |
| **インターネット接続**         | ライセンス有効化に必須                                     |

{% hint style=&quot;warning&quot; %}
**ライセンス要件**: Python SDK の利用には、有料の Chloros+ サブスクリプションが必要です。 標準（無料）プランでは API/SDK へのアクセス権がありません。アップグレードには [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) をご覧ください。
{% endhint %}

## クイックスタート

### インストール

pip経由でインストール:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**初回設定**: SDK を使用する前に、Chloros+ ライセンスを有効化してください。Chloros、Chloros (ブラウザ) または Chloros CLI を起動し、認証情報でログインして Chloros+ のライセンスを有効化してください。この操作は一度だけ必要です。
{% endhint %}

### 基本操作

数行のみのフォルダ処理:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### 詳細操作

高度なワークフローの場合:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

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

### 事前準備

SDK をインストールする前に、以下を確認してください:

1. **Chloros Desktop** がインストール済み ([ダウンロード](download.md))
2. **Python 3.7以上** がインストール済み ([python.org](https://www.python.org))
3. **有効な Chloros+ ライセンス** ([アップグレード](https://cloud.mapir.camera/pricing))

### pip 経由でのインストール

**標準インストール:**

```bash
pip install chloros-sdk
```

**進行状況モニタリングサポート付き:**

```bash
pip install chloros-sdk[progress]
```

**開発用インストール:**

```bash
pip install chloros-sdk[dev]
```

### インストール確認

SDK が正しくインストールされているかテスト:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## 初回セットアップ

### ライセンスの有効化

SDK は、Chloros、Chloros (ブラウザ)、および Chloros CLI と同じライセンスを使用します。 GUIまたはCLIで一度アクティベートしてください：

1. **ChlorosまたはChloros（ブラウザ）**を開き、ユーザー <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> タブでログインします。または、**CLI**を開きます。
2. Chloros+の認証情報を入力し、ログインします
3. ライセンスはローカルにキャッシュされます（再起動後も保持されます）

{% hint style=&quot;success&quot; %}
**初回設定**: GUIまたはCLI経由でログイン後、SDKは自動的にキャッシュされたライセンスを使用します。追加認証は不要です！
{% endhint %}

{% hint style=&quot;info&quot; %}
**ログアウト**: SDK ユーザーは、`logout()` メソッドを使用して、プログラムでキャッシュされた認証情報をクリアすることができます。 詳細は API リファレンスの [logout() メソッド](#logout) を参照してください。
{% endhint %}

### 接続テスト

SDK が Chloros に接続できることを確認します：

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
| `api_url`                 | str  | `"http://localhost:5000"` | URL ローカルChloros バックエンドの`"http://localhost:5000"`          |
| `auto_start_backend`      | bool | `True`                    | 必要に応じてバックエンドを自動起動 |
| `backend_exe`             | str  | `None` (自動検出)      | バックエンド実行ファイルのパス            |
| `timeout`                 | int  | `30`                      | リクエストタイムアウト（秒）            |
| `backend_startup_timeout` | int  | `60`                      | バックエンド起動タイムアウト（秒） |

**例:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### メソッド

#### `create_project(project_name, camera=None)`

新しいChlorosプロジェクトを作成します。

**パラメータ:**

| パラメータ      | 型        | 必須        | 説明                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Yes      | プロジェクト名                                     |
| `camera`       | str  | No       | カメラテンプレート (例: &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**返り値:** `dict` - プロジェクト作成応答**例:**

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
| `folder_path` | 文字列/パス | 必須     | 画像を含むフォルダのパス         |
| `recursive`   | ブール値 | 任意     | サブフォルダを検索 (デフォルト: False) |

**戻り値:** `dict` - ファイル数付きインポート結果**例:**

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

| パラメータ                 | 型         | デフォルト                 | 説明                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;高品質（高速）&quot; | デベイヤー法                  |
| `vignette_correction`     | bool | `True`                  | ヴィネット補正を有効化      |
| `reflectance_calibration` | bool | `True`                  | 反射率キャリブレーション有効化  |
| `indices`                 | list | `None`                  | 計算対象植生指数 |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | 出力フォーマット                   |
| `ppk`                     | bool | `False`                 | PPK補正を有効化          |
| `custom_settings`         | 辞書 | `None`                  | 高度なカスタム設定        |

**エクスポート形式:**

* `"TIFF (16-bit)"` - GIS/写真測量に推奨
* `"TIFF (32-bit, Percent)"` - 科学分析用
* `"PNG (8-bit)"` - 視覚検査用
* `"JPG (8-bit)"` - 圧縮出力

**利用可能なインデックス:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI、SAVI、MSAVI、MTVI2、その他。**例:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
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

| パラメータ         | タイプ     | デフォルト  | 説明                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | 処理モード: &quot;parallel&quot; または &quot;serial&quot;   |
| `wait`              | bool     | `True`       | 完了待ち                       |
| `progress_callback` | callable | `None`       | 進行状況コールバック関数(progress, msg) |
| `poll_interval`     | float    | `2.0`        | 進行状況ポーリング間隔 (秒)   |

**戻り値:** `dict` - 処理結果

{% hint style=&quot;warning&quot; %}
**並列モード**: Chloros+ ライセンスが必要。CPU コア数に応じて自動スケーリング（最大 16 ワーカー）。
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

バックエンドのステータス情報を取得します。

**戻り値:** `dict` - バックエンドのステータス**例:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

バックエンドをシャットダウンします（SDKによって起動された場合）。

**例:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

ローカルシステムからキャッシュされた認証情報をクリアします。

**説明:**

キャッシュされた認証資格情報を削除することで、プログラム的にログアウトします。以下の場合に有用です:
* 異なるChloros+アカウント間の切り替え
* 自動化された環境での資格情報のクリア
* セキュリティ目的（例: アンインストール前の資格情報削除）

**戻り値:** `dict` - ログアウト操作の結果**使用例:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style=&quot;info&quot; %}
**再認証が必要**: `logout()`呼び出し後、Chloros、Chloros (ブラウザ)、または Chloros CLI を介して再度ログインする必要があります。
{% endhint %}

***

### 便利関数

#### `process_folder(folder_path, **options)`

フォルダを処理するワンラインの便利関数。

**パラメータ:**

| パラメータ                 | 型     | デフォルト         | 説明                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | 必須        | 画像を含むフォルダのパス     |
| `project_name`            | str      | 自動生成  | プロジェクト名                   |
| `camera`                  | 文字列     | `None`          | カメラテンプレート                |
| `indices`                 | リスト     | `["NDVI"]`      | 計算対象のインデックス           |
| `vignette_correction`     | bool     | `True`          | ビネット補正を有効化     |
| `reflectance_calibration` | bool     | `True`          | 反射率キャリブレーション有効化 |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | 出力フォーマット                  |
| `mode`                    | str      | `"parallel"`    | 処理モード                |
| `progress_callback`       | callable | `None`          | 進行状況コールバック              |

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

## コンテキストマネージャーのサポート

SDK は自動クリーンアップのためのコンテキストマネージャーをサポートします:

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

### 例 1: 基本処理

デフォルト設定でフォルダを処理:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### 例 2: カスタムワークフロー

処理パイプラインの完全制御:

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
    debayer="High Quality (Faster)",
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

### 例 3: 複数フォルダのバッチ処理

複数のフライトデータセットを処理:

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

### 例 4: 研究パイプライン統合

Chloros をデータ分析と統合:

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

### 例 5: カスタム進捗監視

ログ記録による高度な進捗追跡:

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
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
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

### 例7: アカウント管理とログアウト

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

### 例8: コマンドラインツール

SDK を使用してカスタム CLI ツールを構築:

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

SDK は、異なるエラータイプに対応する特定の例外クラスを提供します:

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
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## 高度なトピック

### カスタムバックエンド設定

カスタムバックエンドの場所または構成を使用します:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### 非ブロッキング処理

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

大規模なデータセットの場合、バッチ処理を行います:

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

1. Chloros Desktop がインストールされていることを確認:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Windows ファイアウォールがブロックしていないか確認
3. 手動バックエンドパスを試す:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### ライセンスが検出されない**問題:** SDK がライセンス不足を警告**解決策:**

1. Chloros、Chloros（ブラウザ）、またはChloros CLIを開き、ログインしてください。
2. ライセンスがキャッシュされていることを確認してください：

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. 認証情報に問題がある場合、キャッシュされた認証情報をクリアして再ログイン:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. サポートに連絡: info@mapir.camera

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

1. タイムアウト時間を延長:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. 処理バッチサイズを縮小
3. 空きディスク容量を確認
4. システムリソースを監視

***

### ポートが既に使用中**問題:** バックエンドポート5000が占有されている**解決策:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

または競合するプロセスを見つけて終了させる:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## パフォーマンスのヒント

### 処理速度の最適化

1. **並列モードの使用** (Chloros+ が必要)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **出力解像度の低下** (許容可能な場合)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **不要なインデックスを無効化**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **SSD上で処理** (HDDではなく)***

### メモリ最適化

大規模データセットの場合：

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### バックグラウンド処理

他のタスク用にPythonを解放:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## 統合例

### Django統合

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

## FAQ

### Q: SDKはインターネット接続が必要ですか？

**A:** 初期ライセンス認証時のみ必要です。Chloros、Chloros（ブラウザ）、またはChloros CLI経由でログイン後、ライセンスはローカルにキャッシュされ、30日間オフラインで動作します。***

### Q: GUIのないサーバーでSDKを使用できますか？**A:** はい！要件：

* Windows Server 2016以降
* Chloros のインストール（1回限り）
* いずれかのマシンでライセンスをアクティベート（キャッシュされたライセンスがサーバーにコピーされます）

***

### Q: Desktop、CLI、SDK の違いは何ですか？

| 機能         | デスクトップGUI | CLI コマンドライン | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **インターフェース**   | ポイント＆クリック | コマンドライン          | Python API  |
| **最適な用途**    | ビジュアル作業 | スクリプティング        | 統合 |
| **自動化**  | 限定的     | 良好             | 優れている   |
| **柔軟性** | 基本       | 良好             | 最大     |
| **ライセンス**     | Chloros+    | Chloros+         | Chloros+    |***

### Q: SDKで構築したアプリを配布できますか？**A:** SDKコードはアプリケーションに組み込めますが、以下の条件を満たす必要があります：

* エンドユーザーはChlorosをインストールする必要があります
* エンドユーザーは有効なChloros+ライセンスが必要です
* 商用配布にはOEMライセンスが必要です

OEMに関するお問い合わせはinfo@mapir.cameraまでご連絡ください。

***

### Q: SDK の更新方法は？

```bash
pip install --upgrade chloros-sdk
```

***

### Q: 処理済み画像はどこに保存されますか？

デフォルトではプロジェクトパス内に保存されます：

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Q: スケジュールで実行されるスクリプトから画像を処理できますか？**A:** はい！Windows タスクスケジューラとPython スクリプトを使用してください：

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

タスク スケジューラで毎日実行するようスケジュール設定してください。

***

### Q: SDK は async/await をサポートしていますか？**A:** 現在のバージョンは同期処理です。非同期動作が必要な場合は、`wait=False` を使用するか、別スレッドで実行してください：

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### Q: 異なるChloros+アカウント間で切り替えるには？**A:** `logout()`メソッドでキャッシュされた認証情報をクリアし、新しいアカウントで再ログインしてください：

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

ログアウト後、GUI、ブラウザ、またはCLI経由で新しいアカウントで認証してから、再度SDKを使用してください。

***

## ヘルプの入手方法

### ドキュメント

* **APIリファレンス**: このページ

### サポートチャネル

* **メール**: info@mapir.camera
* **ウェブサイト**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **価格**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### サンプルコード

ここに記載されているすべての例はテスト済みで、本番環境での使用が可能です。ご自身のユースケースに合わせてコピーし、適宜変更してください。

***

## ライセンス**プロプライエタリソフトウェア** - Copyright (c) 2025 MAPIR Inc.

SDK は有効な Chloros+ サブスクリプションが必要です。無断使用、配布、または改変は禁止されています。
