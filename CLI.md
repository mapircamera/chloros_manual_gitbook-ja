# CLI : コマンドライン

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI**は、Chloros画像処理エンジンへの強力なコマンドラインアクセスを提供し、画像処理ワークフローの自動化、スクリプト化、およびヘッドレス操作を可能にします。

### 主な機能

* 🚀 **自動化** - 複数のデータセットのバッチ処理をスクリプト化
* 🔗 **統合** - 既存のワークフローやパイプラインへの組み込み
* 💻 **ヘッドレス操作** - GUIなしで実行
* 🌍 **多言語対応** - 38言語をサポート
* ⚡ **並列処理** - [動的演算適応](processing-architecture/dynamic-compute-adaptation.md)により、お使いのハードウェアに合わせて自動的に最適化

### 要件

| 要件          | 詳細                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **オペレーティングシステム** | Windows 10/11 (64ビット)、Linux x86_64 (amd64)、Linux arm64 (NVIDIA Jetson JetPack 6) |
| **ライセンス**          | Chloros+ ([有料プランが必要](https://cloud.mapir.camera/pricing)) |
| **メモリ**           | 8GB RAM 以上 (16GB 推奨)                                  |
| **インターネット**         | ライセンスの有効化に必要                                     |
| **ディスク容量**       | プロジェクトのサイズにより異なる                                              |

{% hint style="warning" %}
**ライセンス要件**: CLI には、有料の Chloros+ サブスクリプションが必要です。 スタンダード（無料）プランでは CLI を利用できません。アップグレードするには [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) をご覧ください。
{% endhint %}

## クイックスタート

### インストール

#### Windows

CLIは、Chlorosインストーラーに自動的に含まれています:

1. **Chloros Installer.exe** をダウンロードして実行します
2. インストールウィザードの手順に従います
3. CLI は以下の場所にインストールされます: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
インストーラーは自動的に `chloros-cli` をシステムの PATH に追加します。インストール後、ターミナルを再起動してください。
{% endhint %}

#### Linux

お使いのアーキテクチャに対応した `.deb` パッケージをインストールしてください:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Linuxの詳細な設定については、[Linuxのインストール](linux/linux-installation.md)を参照してください。

### 初回設定

CLIを使用する前に、Chloros+ライセンスを有効化してください:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### 基本操作

デフォルト設定でフォルダを処理します:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## コマンドリファレンス

### 一般的な構文

```
chloros-cli [global-options] <command> [command-options]
```

***

## コマンド

### `process` - 画像の処理

フォルダ内の画像をキャリブレーションして処理します。

**構文:**

```bash
chloros-cli process <input-folder> [options]
```

**例:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### 処理コマンドのオプション

| オプション                | タイプ    | デフォルト        | 説明                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | パス    | _必須_     | RAW/JPGマルチスペクトル画像を含むフォルダ                                         |
| `-o, --output`        | パス    | 入力と同じ  | 処理済み画像の出力フォルダ                                                     |
| `-n, --project-name`  | 文字列  | 自動生成  | カスタムプロジェクト名                                                                    |
| `--vignette`          | フラグ    | 有効        | ヴィネット補正を有効にする                                                             |
| `--no-vignette`       | フラグ    | -              | ヴィネット補正を無効にする                                                            |
| `--reflectance`       | フラグ    | 有効        | 反射率キャリブレーションを有効にする                                                         |
| `--no-reflectance`    | フラグ    | -              | 反射率キャリブレーションを無効にする                                                        |
| `--ppk`               | フラグ    | 無効       | .daq 光センサーデータからの PPK 補正を適用                                      |
| `--format`            | 選択  | TIFF (16 ビット)  | 出力形式: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | 整数 | 自動           | キャリブレーションパネル検出のための最小ターゲットサイズ（ピクセル単位）                          |
| `--target-clustering` | 整数 | 自動           | ターゲットクラスタリングの閾値（0～100）                                                    |
| `--debayer`           | 選択  | `standard`     | デベイヤー法: `standard` または `texture-aware` (Chloros+ のみ)                          |
| `--target`, `--targets` | フラグ  | 無効       | キャリブレーションターゲットを「target」または「targets」サブフォルダ内でのみ検索する（処理を高速化） |
| `--indices`           | リスト    | なし           | 計算する植生指数（例：`--indices NDVI NDRE GNDVI`）                    |
| `--exposure-pin-1`    | 文字列  | なし           | カメラモデル（ピン1）の露出を固定                                                 |
| `--exposure-pin-2`    | 文字列  | なし           | カメラモデル用の露光ロック（ピン2）                                                 |
| `--recal-interval`    | 整数 | 自動           | 再校正間隔（秒）                                                      |
| `--timezone-offset`   | 整数 | 0              | タイムゾーンオフセット（時間）                                                               |

***

### `login` - アカウントの認証

Chloros+ の認証情報を使用してログインし、CLI 処理を有効にしてください。

**構文:**

```bash
chloros-cli login <email> <password>
```

**例:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**特殊文字**: `$`、`!`、またはスペースなどの文字を含むパスワードは、一重引用符で囲んでください。
{% endhint %}

**出力:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - 認証情報のクリア

保存されている認証情報をクリアし、アカウントからログアウトします。

**構文:**

```bash
chloros-cli logout
```

**例:**

```bash
chloros-cli logout
```

**出力:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK ユーザー**: Python SDK では、Python スクリプト内で認証情報をクリアするためのプログラム的な `logout()` メソッドも提供しています。 詳細については、[Python SDK ドキュメント](api-python-sdk.md#logout)を参照してください。
{% endhint %}

***

### `status` - ライセンス状態の確認

現在のライセンスおよび認証ステータスを表示します。

**構文:**

```bash
chloros-cli status
```

**例:**

```bash
chloros-cli status
```

**出力:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - エクスポートの進行状況の確認

処理中または処理後に、スレッド 4 のエクスポートの進行状況を監視します。

**構文:**

```bash
chloros-cli export-status
```

**例:**

```bash
chloros-cli export-status
```

**使用例:** 処理の実行中にこのコマンドを呼び出し、エクスポートの進行状況を確認します。***

### `language` - インターフェース言語の管理

CLI インターフェースの言語を表示または変更します。

**構文:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**例:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### サポートされている言語 (計38言語)

| コード    | 言語              | 現地名      |
| ------- | --------------------- | ---------------- |
| `en`    | 英語               | English          |
| `es`    | スペイン語               | Español          |
| `pt`    | ポルトガル語            | Português        |
| `fr`    | フランス語                | Français         |
| `de`    | ドイツ語                | Deutsch          |
| `it`    | イタリア語               | Italiano         |
| `ja`    | 日本語              | 日本語              |
| `ko`    | 韓国語                | 한국어              |
| `zh`    | 中国語（簡体字）  | 简体中文             |
| `zh-TW` | 中国語（繁体字） | 繁體中文             |
| `ru`    | ロシア語               | Русский          |
| `nl`    | オランダ語                | Nederlands       |
| `ar`    | アラビア語                | العربية          |
| `pl`    | ポーランド語                | Polski           |
| `tr`    | トルコ語               | Türkçe           |
| `hi`    | ヒンディー語                 | हिंदी            |
| `id`    | インドネシア語            | Bahasa Indonesia |
| `vi`    | ベトナム語            | Tiếng Việt       |
| `th`    | タイ語                  | ไทย              |
| `sv`    | スウェーデン語               | Svenska          |
| `da`    | デンマーク語                | Dansk            |
| `no`    | ノルウェー語             | Norsk            |
| `fi`    | フィンランド語               | Suomi            |
| `el`    | ギリシャ語                 | Ελληνικά         |
| `cs`    | チェコ語                 | Čeština          |
| `hu`    | ハンガリー語             | Magyar           |
| `ro`    | ルーマニア語              | Română           |
| `uk`    | ウクライナ語             | Українська       |
| `pt-BR` | ブラジルポルトガル語  | Português Brasileiro |
| `zh-HK` | 広東語             | 粵語             |
| `ms`    | マレー語                | Bahasa Melayu    |
| `sk`    | スロバキア語                | Slovenčina       |
| `bg`    | ブルガリア語             | Български        |
| `hr`    | クロアチア語              | Hrvatski         |
| `lt`    | リトアニア語            | Lietuvių         |
| `lv`    | ラトビア語               | Latviešu         |
| `et`    | エストニア語              | Eesti            |
| `sl`    | スロベニア語             | Slovenščina      |

{% hint style="success" %}
**自動保存**: 言語設定は `~/.chloros/cli_language.json` に保存され、すべてのセッションで維持されます。
{% endhint %}

***

### `set-project-folder` - デフォルトのプロジェクトフォルダの設定

デフォルトのプロジェクトフォルダの場所を変更します（WindowsのGUIと共有）。

**構文:**

```bash
chloros-cli set-project-folder <folder-path>
```

**例:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - プロジェクトフォルダの表示

現在のデフォルトのプロジェクトフォルダの場所を表示します。

**構文:**

```bash
chloros-cli get-project-folder
```

**例:**

```bash
chloros-cli get-project-folder
```

**出力:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - デフォルトにリセット

プロジェクトフォルダーをデフォルトの場所にリセットします。

**構文:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - システム診断の実行

システム構成を確認するために、7つの診断チェックを実行します。

**構文:**

```bash
chloros-cli selftest
```

**実行される診断項目:**

1. バージョン確認
2. ポートの可用性 (5000)
3. バックエンドの起動
4. API 接続テスト
5. システム情報および GPU 検出
6. ノイズ除去モデルの検証
7. CUDA の可用性チェック

{% hint style="info" %}
**トラブルシューティングに有用**：インストール後に`selftest`を実行し、システムが正しく設定されているかを確認してください。特に、GPUおよびCUDAの設定確認が必要なLinux/Jetsonでは重要です。
{% endhint %}

***

### `update` - 更新プログラムの確認 (Linux のみ)

Linux システム上で CLI の更新プログラムを確認し、インストールします。

**構文:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| オプション    | 説明                        |
| --------- | ---------------------------------- |
| `--check` | 更新プログラムの確認のみ（インストールは行わない） |

{% hint style="info" %}
このコマンドは Linux でのみ利用可能です。Windows では、更新プログラムはインストーラーを通じて提供されます。
{% endhint %}

***

## グローバルオプション

これらのオプションはすべてのコマンドに適用されます:

| オプション            | タイプ    | デフォルト       | 説明                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | パス    | 自動検出 | バックエンド実行ファイルへのパス                       |
| `--port`          | 整数 | 5000          | バックエンド API のポート番号                          |
| `--restart`       | フラグ    | -             | バックエンドの再起動を強制（既存のプロセスを終了） |
| `--version`       | フラグ    | -             | バージョン情報を表示して終了                |
| `--help`          | フラグ    | -             | ヘルプ情報を表示して終了                   |

{% hint style="info" %}
**バックエンドの自動検出**: `--backend-exe` パスはプラットフォームごとに自動検出されます:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (手動)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**グローバルオプションを使用した例:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## 処理設定ガイド

### 並列処理と動的演算適応

Chloros 1.1.0 には [動的演算適応](processing-architecture/dynamic-compute-adaptation.md) が含まれています。これは、処理エンジンが **ハードウェアを自動的に検出し**、最適な戦略を選択する機能です:

| プラットフォーム | 戦略 | ワーカー数 | パイプライン | 備考 |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | メモリ効率重視、シリアル化 |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | 並列GPU処理 |
| **8GB GPU搭載デスクトップ** | `GPU_SINGLE` | 3 | `tiled_gpu` | 優れたデスクトップ性能 |
| **12GB以上のGPUを搭載したデスクトップ** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | 最適なデスクトップ性能 |
| **CPUのみのシステム** | `CPU_PARALLEL` | コア数 - 1 | `cpu_fallback` | GPU不要 |

{% hint style="success" %}
**手動設定は不要！** Chlorosは、CPU、GPU、RAM、および（Jetsonの場合）温度センサーを自動検出し、最適な処理パイプラインを自動的に設定します。
{% endhint %}

### デベイヤー処理方法

| 方法 | CLI フラグ | 画質 | 速度 | ライセンス |
| --- | --- | --- | --- | --- |
| **標準 (高速、中程度の品質)** | `--debayer standard` | 良好 | 高速 | 無料 / Chloros+ |
| **テクスチャ対応 (低速、最高品質)** | `--debayer texture-aware` | 最高 | 低速 | Chloros+ のみ |

デフォルトのデベイヤー方式は **標準**です。**テクスチャ対応** 方式は、最高品質の出力を得るために AI/ML ノイズ除去モデルを使用しますが、Chloros+ ライセンスと NVIDIA GPU が必要です。

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### ヴィネット補正

**機能：** 画像の端での光の減衰（カメラ画像によく見られる四隅の暗さ）を補正します。

* **デフォルトで有効** - ほとんどのユーザーはこの設定を有効にしたままにしておくことを推奨します
* `--no-vignette` を使用して無効化

{% hint style="success" %}
**推奨事項**：フレーム全体で均一な明るさを確保するため、常にヴィネット補正を有効にしてください。
{% endhint %}

### 反射率キャリブレーション

キャリブレーションパネルを使用して、センサーの生データを標準化された反射率（パーセント）に変換します。

* **デフォルトで有効** - 植生分析に不可欠です
* 画像内にキャリブレーションターゲットパネルが必要です
* 無効にするには `--no-reflectance` を使用してください

{% hint style="info" %}
**要件**: 正確な反射率変換を行うため、画像内でキャリブレーションパネルが適切に露出され、確認できる状態であることを確認してください。
{% endhint %}

### PPK補正

**機能:** DAQ-A-SDのログデータを使用してポストプロセッシング・キネマティック（PPK）補正を適用し、GPS精度を向上させます。

* **デフォルトでは無効**
* `--ppk`を使用して有効化
* MAPIR DAQ-A-SD光センサーから取得した.daqファイルがプロジェクトフォルダに必要です。

### 出力形式

<table><thead><tr><th width="197">形式</th><th width="130.20001220703125">ビット深度</th><th width="116.5999755859375">ファイルサイズ</th><th>最適用途</th></tr></thead><tbody><tr><td><strong>TIFF (16ビット)</strong> ⭐</td><td>16ビット整数</td><td>大</td><td>GIS解析、写真測量（推奨）</td></tr><tr><td><strong>TIFF (32 ビット、パーセント)</strong></td><td>32 ビット浮動小数点</td><td>超大</td><td>科学分析、研究</td></tr><tr><td><strong>PNG (8 ビット)</strong></td><td>8ビット整数</td><td>中</td><td>視覚的検査、Web共有</td></tr><tr><td><strong>JPG (8ビット)</strong></td><td>8ビット整数</td><td>小</td><td>クイックプレビュー、圧縮出力</td></tr></tbody></table>***

## 自動化とスクリプト

### PowerShell バッチ処理 (Windows)

Windows 上で複数のデータセットフォルダを自動的に処理します:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows バッチスクリプト (Windows)

Windows でのバッチ処理のためのシンプルなループ:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash バッチ処理 (Linux)

Linux での複数のデータセットフォルダの処理:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python 自動化スクリプト (クロスプラットフォーム)

エラー処理機能付き高度な自動化（WindowsおよびLinuxで動作）：

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## 処理ワークフロー

### 標準ワークフロー

1. **入力**: RAW/JPG画像ペアを含むフォルダ
2. **検出**: CLIがサポートされている画像ファイルを自動スキャン
3. **処理**: 並列モードでCPUコア数に応じてスケール（Chloros+）
4. **出力**: 処理済みの画像を含むカメラモデルごとのサブフォルダを作成

### 出力構造の例

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### 処理時間の目安

画像100枚（各12MP）の一般的な処理時間：

| プラットフォーム | モード | 推定時間 | 備考 |
| --- | --- | --- | --- |
| **デスクトップ（12GB以上のGPU）** | `GPU_PARALLEL` | 5～10分 | 最速のオプション |
| **デスクトップ（8GB GPU）** | `GPU_SINGLE` | 10～15分 | 良好なパフォーマンス |
| **Jetson Orin NX（16GB）** | `GPU_PARALLEL` | 15～25分 | エッジコンピューティング |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 30～60分 | メモリ制約あり |
| **CPUのみ** | `CPU_PARALLEL` | 20～40分 | GPU不要 |

{% hint style="info" %}
**パフォーマンスのヒント**: 処理時間は、画像数、解像度、デベイヤー方式、およびハードウェアによって異なります。テクスチャ対応デベイヤーは、標準方式よりも大幅に時間がかかります。詳細については、[動的コンピューティング適応](processing-architecture/dynamic-compute-adaptation.md)を参照してください。
{% endhint %}

***

## トラブルシューティング

### CLI が見つかりません

**Windows エラー:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows 解決策:**

1. インストール場所を確認する:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. PATHに設定されていない場合はフルパスを使用する:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. PATHに手動で追加する:
   * [システムのプロパティ] → [環境変数] を開く
   * PATH 変数を編集する
   * 追加: `C:\Program Files\Chloros\resources\cli`
   * ターミナルを再起動

**Linux エラー:**

```
chloros-cli: command not found
```

**Linux 解決策:**

1. インストールを確認:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. シェルを再読み込みする:

```bash
source ~/.bashrc
```

3. 権限を確認する:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### バックエンドの起動に失敗しました**エラー:**

```

Backend failed to start within 30 seconds
```

**解決策:**

1. バックエンドがすでに実行されていないか確認する（実行中の場合はまず終了させる）
2. ファイアウォールによるブロックがないか確認する（Windows）またはポートの空き状況を確認する（Linux: `lsof -i :5000`）
3. 別のポートを試す:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. バックエンドを強制再起動する:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Linux の場合、バックエンドの実行ファイルが存在するか確認する:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### ライセンス / 認証に関する問題**エラー:**

```

Chloros+ license required for CLI access
```

**解決策:**

1. Chloros+の有効なサブスクリプションがあることを確認してください
2. 認証情報を使用してログインしてください:

```bash
chloros-cli login user@example.com 'password'
```

3. ライセンスの状態を確認してください:

```bash
chloros-cli status
```

4. サポートにお問い合わせください：info@mapir.camera

***

### 画像が見つかりません**エラー:**

```

No images found in the specified folder
```

**解決策:**

1. フォルダにサポートされている形式（.RAW、.TIF、.JPG）が含まれているか確認してください
2. フォルダパスが正しいか確認してください（パスにスペースが含まれる場合は引用符を使用してください）
3. フォルダへの読み取り権限があることを確認してください
4. ファイル拡張子が正しいか確認してください

***

### 処理が停止またはフリーズする**解決策:**

1. 空きディスク容量を確認してください（出力用に十分な容量があることを確認してください）
2. 他のアプリケーションを閉じてメモリを解放してください
3. 画像の数を減らしてください（バッチ処理を行ってください）

***

### ポートが既に使用中**エラー:**

```

Port 5000 is already in use
```

**解決策:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## よくある質問

### Q: CLI を使用するにはライセンスが必要ですか？

**A:**はい！CLI を使用するには、有料の**Chloros+ ライセンス**が必要です。

* ❌ スタンダード（無料）プラン：CLIは利用不可
* ✅ Chloros+（有料）プラン：CLIは完全に有効

購読はこちら：[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q: GUIのないサーバーでCLIを使用できますか？**A:** はい！CLIは完全にヘッドレスで動作します。 これは Linux における主な使用例です。**Windows サーバー:**
* Windows Server 2016 以降
* Visual C++ 再配布可能パッケージがインストールされていること

**Linux サーバー:**
* Ubuntu 20.04 以降 / Debian 11 以降 (amd64) または JetPack 6 (arm64)
* `.deb` パッケージ経由でインストール

**両プラットフォーム共通:**
* メモリ 8GB 以上 (16GB 推奨)
* ライセンスの初回アクティベーション: `chloros-cli login user@example.com 'password'`

***

### Q: 処理済みの画像はどこに保存されますか？**A:**デフォルトでは、処理済みの画像は**入力ファイルと同じフォルダ**内のカメラモデルごとのサブフォルダ（例：`Survey3N_RGN/`）に保存されます。

別の出力フォルダを指定するには、`-o` オプションを使用してください：

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Q: 複数のフォルダを一度に処理できますか？**A:** 1つのコマンドで直接行うことはできませんが、スクリプトを使用してフォルダを順次処理することは可能です。[自動化とスクリプト](CLI.md#automation--scripting)のセクションを参照してください。***

### Q: CLIの出力をログファイルに保存するにはどうすればよいですか？**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**バッチ:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Q: 処理中に Ctrl+C を押すとどうなりますか？**A:** CLIは以下の動作を行います：

1. 正常に処理を停止します
2. バックエンドをシャットダウンします
3. コード130で終了します

一部処理済みの画像が出力フォルダに残る場合があります。

***

### Q: CLIの処理を自動化できますか？**A:** もちろんです！CLIは自動化を想定して設計されています。PowerShell (Windows)、Batch (Windows)、 Bash (Linux)、および Python（クロスプラットフォーム）のサンプルについては、[Automation &amp; Scripting](CLI.md#automation--scripting) をご覧ください。***

### Q: CLI のバージョンを確認するにはどうすればよいですか？**A:**

```bash
chloros-cli --version
```

**出力:**

```

Chloros CLI 1.1.0
```

***

## ヘルプの表示

### コマンドラインのヘルプ

CLI内で直接ヘルプ情報を表示するには:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### サポートチャネル

* **メール**: info@mapir.camera
* **ウェブサイト**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **価格**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## 完全な例

### 例 1: 基本処理

デフォルト設定での処理（ヴィネット、反射率）：

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### 例 2: 高品質な科学出力

32 ビット浮動小数点 TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### 例 3: 高速プレビュー処理

迅速な確認のためのキャリブレーションなしの 8 ビット PNG:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### 例 4: PPK補正処理

反射率を用いたPPK補正の適用：

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### 例 5: カスタム出力先

特定の形式で別の場所に処理する:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### 例 6: 認証ワークフロー

完全な認証フロー（すべてのプラットフォームで共通）：

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### 例 7: 多言語対応

インターフェース言語の変更（すべてのプラットフォームで共通）：

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
