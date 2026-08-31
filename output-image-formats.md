---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# 出力画像形式

Chloros は、処理済みの成果物を 4 つのファイル形式でエクスポートします。 形式は、プロジェクト設定（GUI）で、`--format`（CLI）または`export_format` （SDK）を使用して選択します。CLI および SDK では、以下の文字列をそのまま指定できます。

| フォーマット文字列 | 拡張子 | ピクセルタイプ | ピクセル範囲 | 備考 |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(デフォルト)* | `.tif` | uint16 数値 | 0 – 65535 | 写真測量／GIS に推奨。 |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0.0 – 1.0 | 1.0 = 100%の反射率。浮動小数点形式のTIFFを読み込めないアプリケーションもあります。ファイルサイズが大きくなります。 |
| `PNG (8-bit)` | `.png` | uint8 数値 | 0 – 255 | ロスレス圧縮。Web での閲覧や可視化に適しています。 |
| `JPG (8-bit)` | `.jpg` | uint8 数値 | 0 – 255 | 非可逆圧縮。ファイルサイズが最小。 |

## 出力ファイルの保存先

出力ファイルはプロジェクトフォルダ内に、カメラごとにグループ分けされ、さらにファイル形式ごとに分類されて保存されます：

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

カメラごとのフォルダは、LATTICEの場合は`LATT-<sensor>-<lens>-F<filter>`、Survey3の場合は`<model>_<filter>`（例：`Survey3N_RGN`）となります。 **エクスポートされたすべての製品は、ソースファイルの名前を保持します。製品を識別するのはフォルダであり、ファイル名の拡張子ではありません。** 完全なルールについては、『CLI リファレンス』の [出力ファイルの保存先](reference/cli-reference.md) を参照してください。

## LATTICE 製品（キャプチャおよびエクスポートレベル）

1つのLATTICE生フレームは、1回のパスで要求されたすべてのプロダクトに分岐されます。各プロダクトタイプには独自の切り替え設定（GUIのチェックボックス、またはCLI、`--debayered` / `--preview` / `--radiance` / `--reflectance`、デフォルトではすべてON）があります：

| レベル | 内容 | データ型 |
| --- | --- | --- |
| `raw` | センサーから直接取得したベイヤーデータ（モノクロカメラ：単一バンド）。処理は常にRAWデータから開始されます。 | 撮影時のまま |
| `debayered` | リニアデモザイク — M3Cの場合は3チャンネル、M3Mの場合は1チャンネルのグレースケール。 | リニアDN |
| `radiance` | 完全な放射測定チェーンから得られる絶対スペクトル放射輝度（単位：**W/m²/sr/nm**）。 選択したエクスポート形式にかかわらず、常に32ビットのTIFF（`tiff32/Radiance_Images/`）として記述される。 | float32 |
| `reflectance` | 反射率 ρ。**DN 32768 = ρ 1.0 (100%)** であり、ρ 2.0 までのヘッドルームがあります。Pix4D 対応。 | uint16 |
| `preview` | 表示用レンダリング：RGB = ホワイトバランス + ガンマ補正；マルチスペクトル = 偽色伸張。 | 8ビット表示 |

## 反射率ピクセル値の読み取り

反射率は整数のデジタル数値（DN）として保存され、**ρ = 1.0（反射率100%）を表すDNは、ソースカメラによって異なります**：

| ソースカメラ | ρ = 1.0 に対応するDN | 判別方法 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768`（ヘッドルームは ρ 2.0 まで） | ファイルに XMP タグ `Chloros:PixelScale=32768` が付与されている。 |
| Survey3 | `65535`（ρ 1.0でクリップ） | `Chloros:*` XMPタグがない — その欠如が判断材料となる。 |

**定数であると仮定するのではなく、`Chloros:PixelScale` XMPタグを読み取り、それを除算する**。 このタグは uint16 ドメインで定義されているため、再スケーリングを行う出力形式間でも `32768` のまま維持されます。保存されたデータ型をまず uint16 に正規化してください（8 ビットからは ×257、float32 からは ×65535）。

{% hint style="warning" %}
**設計上、スケールを持たないケースが1つあります。** 8ビットソースのキャプチャ（BayerRG8）が8ビットのTIFFとして書き込まれる場合、 パイプラインは再スケーリングを行わず0～255の範囲にクリップするため、このファイルにはスケール情報が含まれません。— Chlorosでは、意図的に`Chloros:PixelScale`が省略されています。 LATTICEの反射率ファイルにこのタグがない場合、スケールがあるとは仮定しないでください。代わりに、16ビットまたは32ビットで再エクスポートしてください。
{% endhint %}

完全なルール（MicaSense 互換タグを含む）については、[CLI リファレンス](reference/cli-reference.md) の **「反射率ピクセルの読み取り」** を参照してください。
