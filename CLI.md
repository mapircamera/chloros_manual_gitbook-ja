# CLI : コマンドライン

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** は、Chloros 画像処理エンジンへの強力なコマンドラインアクセスを提供し、イメージングワークフローの自動化、スクリプティング、ヘッドレス操作を可能にします。

### 主な機能

* 🚀 **自動化** - 複数データセットのスクリプトによるバッチ処理
* 🔗 **統合** - 既存ワークフローやパイプラインへの組み込み
* 💻 **ヘッドレス操作** - GUIなしで実行
* 🌍 **多言語対応** - 38言語をサポート
* ⚡ **並列処理** - CPUに動的にスケーリング（最大16の並列ワーカー）

### 要件

| 要件                                                                               | 詳細                                                                               |
| -------------------- | ------------------------------------------------------------------- |
| **オペレーティングシステム** | Windows 10/11 (64ビット)                                              |
| **ライセンス**          | Chloros+ ([有料プランが必要](https://cloud.mapir.camera/pricing)) |
| **メモリ**           | 8GB RAM以上 (16GB推奨)                                                    |
| **インターネット接続** | ライセンス有効化に必須                                                    |
| **ディスク容量**       | プロジェクトサイズにより異なる                                              |

{% hint style=&quot;warning&quot; %}
**ライセンス要件**: CLI には有料の Chloros+ サブスクリプションが必要です。 標準（無料）プランではCLIにアクセスできません。アップグレードは[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)をご覧ください。
{% endhint %}

## クイックスタート

### インストール

CLI は Chloros インストーラーに自動的に含まれます：

1. **Chloros インストーラー.exe** をダウンロードして実行
2. インストールウィザードを完了
3. CLI は以下にインストールされます: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
インストーラーは自動的に `chloros-cli` をシステム PATH に追加します。インストール後、ターミナルを再起動してください。
{% endhint %}

### 初回セットアップ

CLI を使用する前に、Chloros+ ライセンスをアクティベートしてください:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### 基本操作

デフォルト設定でフォルダを処理:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## コマンドリファレンス

### 基本構文

```
chloros-cli [global-options] <command> [command-options]
```

***

## コマンド一覧

### `process` - 画像処理

キャリブレーション付きでフォルダ内の画像を処理します。

**構文:**

```bash
chloros-cli process <input-folder> [options]
```

**例:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### 処理コマンドオプション

| オプション                | タイプ    | デフォルト        | 説明                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | パス    | _必須_     | RAW/JPGマルチスペクトル画像を含むフォルダ                                         |
| `-o, --output`        | パス    | 入力と同じ  | 処理済み画像の出力フォルダ                                                     |
| `-n, --project-name`  | 文字列  | 自動生成 | カスタムプロジェクト名                                                                    |
| `--vignette`          | フラグ    | 有効        | ビネット補正を有効化                                                             |
| `--no-vignette`       | フラグ    | -              | ビネット補正を無効化                                                            |
| `--reflectance`       | フラグ    | 有効        | 反射率キャリブレーションを有効化                                                         |
| `--no-reflectance`    | フラグ    | -              | 反射率キャリブレーションを無効化                                                        |
| `--ppk`               | フラグ    | 無効         | .daq光センサーデータからのPPK補正を適用                                      |
| `--format`            | 選択    | TIFF (16ビット)  | 出力形式: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | 整数 | 自動         | キャリブレーションパネル検出の最小ターゲットサイズ（ピクセル単位）                          |
| `--target-clustering` | 整数 | 自動         | ターゲットクラスタリング閾値（0-100）                                                    |
| `--exposure-pin-1`    | 文字列  | なし           | カメラモデル用露光ロック (ピン1)                                                 |
| `--exposure-pin-2`    | 文字列  | なし           | カメラモデル用露光ロック (ピン2)                                                 |
| `--recal-interval`    | 整数 | 自動         | 再校正間隔（秒単位）                                                      |
| `--timezone-offset`   | 整数 | 0              | タイムゾーンオフセット（時間単位）                                                               |

***

### `login` - アカウント認証

Chloros+の認証情報でログインし、CLI処理を有効化してください。

**構文:**

```bash
chloros-cli login <email> <password>
```

**例:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**特殊文字**: `$`、`!`、スペースなどの文字を含むパスワードはシングルクォートで囲んでください。
{% endhint %}

**出力:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - 認証情報のクリア

保存された認証情報をクリアし、アカウントからログアウトします。

**構文:**

```bash
chloros-cli logout
```

**例:**

```powershell
chloros-cli logout
```

**出力:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - ライセンス状態の確認

現在のライセンスと認証状態を表示します。

**構文:**

```bash
chloros-cli status
```

**例:**

```powershell
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

### `export-status` - エクスポート進捗の確認

処理中または処理後にスレッド4のエクスポート進捗を監視します。

**構文:**

```bash
chloros-cli export-status
```

**例:**

```powershell
chloros-cli export-status
```

**使用例:** 処理実行中にこのコマンドを呼び出し、エクスポート進捗を確認します。

***

### `language` - インターフェース言語の管理

CLI インターフェース言語を表示または変更します。

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

```powershell
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### サポート言語 (合計38言語)

| コード    | 言語               | ネイティブ名      |
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
| `nl`    | オランダ語         | Nederlands       |
| `ar`    | アラビア語         | العربية          |
| `pl`    | ポーランド語        | Polski           |
| `tr`    | トルコ語            | Türkçe           |
| `hi`    | ヒンディー語         | हिंदी            |
| `id`    | インドネシア語       | Bahasa Indonesia |
| `vi`    | ベトナム語         | Tiếng Việt       |
| `th`    | タイ語                  | ไทย              |
| `sv`    | スウェーデン語               | Svenska          |
| `da`    | デンマーク語                | Dansk            |
| `no`    | ノルウェー語             | Norsk            |
| `fi`    | フィンランド語     | Suomi            |
| `el`    | ギリシャ語         | Ελληνικά         |
| `cs`    | チェコ語         | Čeština          |
| `hu`    | ハンガリー語         | Magyar           |
| `ro`    | ルーマニア語         | Română           |
| `uk`    | ウクライナ語         | Українська       |
| `pt-BR` | ブラジルポルトガル語 | Português Brasileiro |
| `zh-HK` | 広東語             | 広東語             |
| `ms`    | マレー語                 | マレー語    |
| `sk`    | スロバキア語                | スロバキア語       |
| `bg`    | ブルガリア語             | Български        |
| `hr`    | クロアチア語              | Hrvatski         |
| `lt`    | リトアニア語            | Lietuvių         |
| `lv`    | ラトビア語            | Latviešu         |
| `et`    | エストニア語            | Eesti            |
| `sl`    | スロベニア語            | Slovenščina      |

{% hint style=&quot;success&quot; %}
**自動保存機能**: 言語設定は `~/.chloros/cli_language.json` に保存され、すべてのセッションで維持されます。
{% endhint %}

***

### `set-project-folder` - デフォルトプロジェクトフォルダの設定

デフォルトプロジェクトフォルダの場所を変更します（GUIと共有）。

**構文:**

```bash
chloros-cli set-project-folder <folder-path>
```

**例:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - プロジェクトフォルダを表示

現在のデフォルトプロジェクトフォルダの場所を表示します。

**構文:**

```bash
chloros-cli get-project-folder
```

**例:**

```powershell
chloros-cli get-project-folder
```

**出力:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - デフォルトにリセット

プロジェクトフォルダをデフォルトの場所にリセットします。

**構文:**

```bash
chloros-cli reset-project-folder
```

***

## 全体オプション

これらのオプションは全てのコマンドに適用されます:

| オプション          | タイプ    | デフォルト       | 説明                                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | パス    | 自動検出 | バックエンド実行ファイルへのパス                       |
| `--port`        | 整数 | 5000          | バックエンド API ポート番号                          |
| `--restart`     | フラグ    | -             | バックエンドの再起動を強制（既存プロセスを終了） |
| `--version`     | フラグ    | -             | バージョン情報を表示して終了                |
| `--help`        | フラグ    | -             | ヘルプ情報を表示して終了                   |

**グローバルオプション使用例:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## 処理設定ガイド

### 並列処理

Chloros+ CLI は、コンピューターの性能に合わせて並列処理を**自動スケーリング**します:

**動作原理:**

* CPUコア数とRAMを検出
* ワーカーを割り当て: **CPUコア数×2** (ハイパースレッディング利用)
* **最大: 16並列ワーカー** (安定性確保のため)

**システム階層:**

| システムタイプ   | CPU        | RAM      | ワーカー  | パフォーマンス     |
| ---------| **ハイエンド**  | 16+ コア  | 32+ GB   | 最大16   | 最高速度   |
| **ミドルレンジ** | 8-15 コア | 16-31 GB | 8-16     | 優れた速度 |
| **ローエンド**   | 4-7 コア  | 8-15 GB  | 4-8      | 良好な速度      |

{% hint style=&quot;success&quot; %}
**自動最適化**: CLIはシステム仕様を自動検知し、最適な並列処理を設定します。手動設定不要！
{% endhint %}

### デベイヤー方式

CLI はデフォルトで推奨されるデベイヤーアルゴリズムとして **高品質 (高速)** を使用します:

| 方式                      | 品質 | 速度 | 説明                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **高品質（高速）** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | エッジ認識アルゴリズム（デフォルト、推奨） |

### ヴィネット補正

**機能：** 画像端部の光量減衰（カメラ画像でよく見られる暗い四隅）を補正します。

* **デフォルトで有効** - ほとんどのユーザーはこの機能を有効にしたままにしておくべきです
* 無効化するには `--no-vignette` を使用

{% hint style=&quot;success&quot; %}
**推奨設定**: フレーム全体の輝度を均一にするため、常にビネット補正を有効にしてください。
{% endhint %}

### 反射率キャリブレーション

キャリブレーションパネルを用いて、センサーの生の値を標準化された反射率パーセンテージに変換します。

* **デフォルトで有効** - 植生解析に必須
* 画像内に校正ターゲットパネルが必要
* 無効化には `--no-reflectance` を使用

{% hint style=&quot;info&quot; %}
**要件**: 正確な反射率変換のため、画像内で校正パネルが適切に露出され可視化されていることを確認してください。
{% endhint %}

### PPK補正

**機能:** GPS精度向上のため、DAQ-A-SDログデータを用いた後処理動的補正（PPK補正）を適用します。

* **デフォルトで無効化**
* 有効化には`--ppk`を使用
* MAPIR DAQ-A-SD光センサーからの.daqファイルをプロジェクトフォルダに必要。

### 出力フォーマット

<table><thead><tr><th width="197">フォーマット</th><th width="130.20001220703125">ビット深度</th><th width="116.5999755859375">ファイルサイズ</th><th>最適用途</th></tr></thead><tbody><tr><td><strong>TIFF (16ビット)</strong> ⭐</td><td>16ビット整数</td><td>大</td><td>GIS分析、写真測量（推奨）</td></tr><tr><td><strong>TIFF (32ビット、パーセント)</strong></td><td>32ビット浮動小数点</td><td>非常に大きい</td><td>科学分析、研究</td></tr><tr><td><strong>PNG (8 ビット)</strong></td><td>8 ビット整数</td><td>中</td><td>目視検査、ウェブ共有</td></tr><tr><td><strong>JPG (8ビット)</strong></td><td>8ビット整数</td><td>小</td><td>クイックプレビュー、圧縮出力</td></tr></tbody></table>***

## 自動化とスクリプト

### PowerShell バッチ処理

複数のデータセットフォルダを自動処理:

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

### Windows バッチスクリプト

バッチ処理用簡易ループ:

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

### Python 自動化スクリプト

エラー処理付き高度な自動化:

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
2. **検出**: CLI が対応画像ファイルを自動スキャン
3. **処理**: 並列モードでCPUコア数に応じてスケーリング (Chloros+)
4. **出力**: 処理済み画像をカメラモデル別サブフォルダに生成

### 出力構造例

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

### 処理時間の見積もり

100枚の画像（各12MP）の典型的な処理時間：

| モード              | 時間      | ハードウェア                                     |
| ----------------- | --------- | -------------------------------------------- |
| **並列モード** | 5-10 分  | i7/Ryzen 7、16GB RAM、SSD (最大16ワーカー) |
| **並列モード** | 10-15 分 | i5/Ryzen 5、8GB RAM、HDD (最大8ワーカー)   |

{% hint style=&quot;info&quot; %}
**パフォーマンスのヒント**: 処理時間は画像数、解像度、コンピュータのスペックによって異なります。
{% endhint %}

***

## トラブルシューティング

### CLIが見つかりません

**エラー:**

```
'chloros-cli' is not recognized as an internal or external command
```

**解決策:**

1. インストール場所を確認:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. PATHにない場合はフルパスを使用:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. PATHに手動で追加:
   * システムのプロパティ → 環境変数を起動
   * PATH変数を編集
   * 追加: `C:\Program Files\Chloros\resources\cli`
   * ターミナルを再起動

***

### バックエンドの起動に失敗しました

**エラー:**

```
Backend failed to start within 30 seconds
```

**解決策:**

1. バックエンドが既に実行中か確認（実行中の場合は終了）
2. ファイアウォールがブロックしていないか確認
3. 別のポートを試す:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. バックエンドを強制再起動:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### ライセンス/認証の問題

**エラー:**

```
Chloros+ license required for CLI access
```

**解決策:**

1. 有効なChloros+サブスクリプションを確認
2. 認証情報でログイン:

```powershell
chloros-cli login user@example.com 'password'
```

3. ライセンス状態を確認:

```powershell
chloros-cli status
```

4. サポートに連絡: info@mapir.camera

***

### 画像が見つかりません

**エラー:**

```
No images found in the specified folder
```

**解決策:**

1. フォルダにサポートされている形式（.RAW、.TIF、.JPG）が含まれていることを確認してください
2. フォルダパスが正しいことを確認してください（スペースを含むパスには引用符を使用してください）
3. フォルダへの読み取り権限があることを確認してください
4. ファイル拡張子が正しいか確認してください

***

### 処理が停止またはハングする

**解決策:**

1. 空きディスク容量を確認してください（出力用に十分な容量を確保してください）
2. 他のアプリケーションを閉じてメモリを解放してください
3. 画像数を減らしてください（バッチ処理で処理してください）

***

### ポートが既に使用中

**エラー:**

```
Port 5000 is already in use
```

**解決策:**

別のポートを指定してください:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## よくある質問

### Q: CLIの使用にはライセンスが必要ですか？

**A:** はい！CLIには有料の**Chloros+ライセンス**が必要です。

* ❌ スタンダード（無料）プラン: CLIが無効化
* ✅ Chloros+（有料）プラン：CLIが完全に有効化

購読はこちら：[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q: GUIのないサーバーでCLIは使用できますか？

**A:** はい！CLIは完全にヘッドレスで動作します。要件：

* Windows Server 2016以降
* Visual C++ 再配布可能パッケージのインストール
* 十分なRAM（最低8GB、推奨16GB）
* 任意のマシンでのGUIライセンスの一時的なアクティベーション

***

### Q: 処理済み画像はどこに保存されますか？

**A:** デフォルトでは、処理済み画像は入力画像と同じフォルダ内のカメラモデル別サブフォルダ（例：`Survey3N_RGN/`）に保存されます。

別の出力フォルダを指定するには `-o` オプションを使用してください：

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Q: 複数のフォルダを同時に処理できますか？

**A:** 単一のコマンドでは直接できませんが、スクリプトを使用してフォルダを順次処理できます。[自動化とスクリプト](CLI.md#automation--scripting)セクションを参照してください。

***

### Q: 出力をログファイルに保存するにはどうすればよいですか？

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**バッチ:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Q: 処理中にCtrl+Cを押すとどうなりますか？

**A:** CLIは以下を実行します:

1. 処理を正常に停止
2. バックエンドをシャットダウン
3. 終了コード130で終了

部分的に処理された画像が出力フォルダに残る場合があります。

***

### Q: CLI処理を自動化できますか？

**A:** もちろん可能です！CLIは自動化を前提に設計されています。 PowerShell、バッチ、Pythonの例については[Automation &amp; Scripting](CLI.md#automation--scripting)を参照してください。

***

### Q: CLIのバージョンを確認するには？

**A:**

```powershell
chloros-cli --version
```

**出力:**

```
Chloros CLI 1.0.2
```

***

## ヘルプの取得

### コマンドラインヘルプ

CLI内で直接ヘルプ情報を表示:

```powershell
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
* **価格**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## 完全な例

### 例 1: 基本処理

デフォルト設定での処理 (ビネット、反射率):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### 例 2: 高品質な科学出力

32ビット浮動小数点 TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### 例3: 高速プレビュー処理

迅速な確認用、キャリブレーションなしの8ビット PNG:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### 例4: PPK補正処理

反射率を用いたPPK補正を適用:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### 例5: カスタム出力先

特定のフォーマットで別ドライブへ処理:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### 例6: 認証ワークフロー

認証フローを完了する:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### 例7: 多言語対応

インターフェース言語を変更する:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
