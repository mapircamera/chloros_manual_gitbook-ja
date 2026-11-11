---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# 出力画像形式

Chlorosは、処理済みの成果物を4つのファイル形式でエクスポートします。形式は、プロジェクト設定（GUI）で、または`--format`（CLI）、`export_format`（SDK）を使用して選択してください。CLIおよびSDKでは、以下の文字列をそのまま指定できます。

| フォーマット文字列 | 拡張子 | ピクセルタイプ | ピクセル範囲 | 備考 |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(デフォルト)* | `.tif` | uint16 数値 | 0 – 65535 | フォトグラメトリ／GIS に推奨。 |
| `TIFF (8-bit)` | `.tif` | uint8 数値 | 0 – 255 | 8ビット形式のみを提供する全8ビットキャプチャ用：Chlorosはビット深度をアップスケールしません。 |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0.0 – 1.0 | 1.0 = 100%の反射率。一部のアプリケーションでは浮動小数点形式のTIFFを読み込めない場合があります。ファイルサイズが大きくなります。 |
| `PNG (8-bit)` | `.png` | uint8 数値 | 0 – 255 | ロスレス圧縮。Webでの閲覧や可視化に適しています。 |
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

カメラごとのフォルダは、LATTICEの場合は`LATT-<sensor>-<lens>-F<filter>`、Survey3の場合は`<model>_<filter>`（例：`Survey3N_RGN`）となります。 **エクスポートされたすべてのプロダクトは、ソースファイルの名前を保持します。プロダクトを識別するのはフォルダであり、ファイル名の拡張子ではありません。** 詳細なルールについては、CLIリファレンスの [出力ファイルの保存先](reference/cli-reference.md) を参照してください。

## LATTICE プロダクト（キャプチャおよびエクスポートレベル）

1つのLATTICE生フレームは、1回のパスで要求されたすべてのプロダクトに分岐されます。各プロダクトタイプには独自のトグルがあります（GUIのチェックボックス、またはCLIの`--debayered` / `--preview` / `--radiance` / `--reflectance`、デフォルトではすべてON）があります：

| レベル | 内容 | データ型 |
| --- | --- | --- |
| `raw` | センサーから直接取得したベイヤーデータ（モノクロカメラ：単一バンド）。処理は常にRAWデータから開始されます。 | 撮影時のまま |
| `debayered` | リニアデモザイク — M3Cの場合は3チャンネル、M3Mの場合は1チャンネルのグレースケール：キャリブレーションパックのダークオフセットを除去した後、エッジを意識したデモザイクを適用。線形性補正、フラットフィールド補正、アンミックスは行わない。 `process` 形式でのエクスポートは、どの処理モードにおいても、このレベルでキャプチャが保存するファイルと同じファイルを書き出します。 | リニアDN |
| `radiance` | 完全な放射測定チェーンから得られた絶対スペクトル放射輝度（単位：**W/m²/sr/nm**）。 選択したエクスポート形式にかかわらず、常に32ビットTIFF（`tiff32/Radiance_Images/`）として書き出されます。 | float32 |
| `reflectance` | 反射率 ρ。**DN 32768 = ρ 1.0 (100%)** であり、ρ 2.0 までのヘッドルームがあります。Pix4D 対応。 | uint16 |
| `preview` | 表示用レンダリング：RGB = ホワイトバランス + ガンマ；マルチスペクトル = 偽色伸張。 | 8ビット表示 |

### 向き

`raw` および `debayered` ファイルは常にセンサーの**ネイティブ**方位を維持します。カメラの反転／回転設定は、他の製品（プレビュー、放射輝度、反射率、インデックス）にのみ適用されます。 アレイキャプチャでは、向きを指定しない限り（CLI では `--orient`、SDK では `orient=True`）、各メンバーのファイルはネイティブのまま保持されます。 各ファイルは、そのピクセルに属する向きを XMP に記録します — `ChlorosRotation`、`ChlorosFlipHorizontal`、 `ChlorosFlipVertical` — また、`ChlorosOrientationApplied`はそのピクセルが方向情報を保持しているかどうかを示します（RAWファイルおよびデベイヤリング済みファイルでは`false`）。 Chloros Image Viewerは、表示のみを目的として、そのレコードに基づいてRAWファイルおよびデベイヤー処理済みファイルを反転表示しますが、ファイル自体は決して上書きされません。レコードがないファイル（バージョン1.3.0以前に作成されたもの、または他のツールで作成されたもの）は、保存されたままの状態で表示されます。

## クリップマスク

マルチスペクトル M3C カメラの放射輝度および反射率データは、3×3 バンドのアンミックス処理が行われます。アンミックス処理の結果、バンド内で負の値が生成された場合、その値はゼロにクリップされます（ライブビューでもまったく同じ方法でクリップされます）。 バージョン1.3.0の新機能として、少なくとも1ピクセルがクリップされたプロダクトには、どのピクセルとバンドがクリップされたかを正確に示す**クリップマスク**が付属するようになりました：

* **場所：**`clamp_masks/` フォルダ内の、データ出力フォルダの**隣**にあり、決してその内部にはありません： `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`、および `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`。
* **内容：** 製品独自のピクセルグリッド上に配置された、単一バンドの `uint8` 形式のTIFFファイル（向き、ROI、ビニング、および配列の整列を含む）。 製品の R チャンネルでバンドがクリップされた場合はビット 0 (`mask & 1`) が設定され、ビット 1 (`& 2`) は G チャンネル、ビット 2 (`& 4`) は B チャンネルでクリップされた場合にセットされます。 クリップされたピクセルのバンドごとの割合も、`Chloros:BandRUnmixClampFraction`、`Chloros:BandGUnmixClampFraction`、および `Chloros:BandBUnmixClampFraction` として製品に記録されます。
* **製品での表記方法：** XMP タグ `Chloros:UnmixClampMask` には、製品のフォルダを基準としたマスクのパスが格納されています（`../clamp_masks/<product folder>/<name>_clampmask.tif`、 常に`/`で区切られています）。ピクセルがクリップされなかった場合は`none`（マスクは書き込まれません）、マスクを書き込めなかった場合は`unavailable`となります。 タグのないプロダクトは、アンミックスが実行されていないことを意味します（RGBおよびモノクロカメラ、RAWおよびデベイヤー処理済みレベル）。

どのプロダクトフォルダにもマスクは含まれていないため、`Reflectance_Calibrated_Images/` のようなプロダクトフォルダを、サブフォルダも含めてそのまま Pix4D、Metashape、またはその他の写真測量ソフトウェアに渡すことができます。その際、その上のフォーマットフォルダではなく、プロダクトフォルダそのものを渡してください。 Chloros 独自のインポート機能では、クランプマスクはスキップされます。CLI では、これらは個別にカウントされます：`Image products written: N (plus M clamp masks)`。

## インデックスプロダクト

各インデックスは、独自の `<INDEX>_Index_Images/` フォルダ（例：`NDVI_Index_Images/`）に格納されます。

* **整数フォーマット**（`TIFF (16-bit)`、`TIFF (8-bit)`、`PNG (8-bit)`、 `JPG (8-bit)`）は、−1…1へのマッピングではなく、**画像ごとの線形伸縮**を格納しています。DN 0はそのフレームの最小インデックス値であり、フルスケールが最大値となります。 このファイルには `Chloros:IndexScaling=linear_minmax`、`Chloros:IndexMin`、`Chloros:IndexMax` が含まれているため、`value = IndexMin + DN / full_scale × (IndexMax − IndexMin)` も含まれていることになります。 各フレームには独自の端点があるため、フレーム間の比較は DN ではなく値で行ってください。
* **`TIFF (32-bit, Percent)`** は、リスケールを行わずにインデックス値そのものを float32 として格納します。これは、フライト全体にわたる定量解析に最適な選択肢です。

バージョン 1.3.0 からの新機能として、インデックスプロダクトはソースの生データのタグを繰り返すのではなく、**それ自体**を記述するようになりました。その GPS およびカメラタグは、計算元となったプロダクト（存在する場合は反射率プロダクト）からコピーされ、その後：

| タグ | 値 |
| --- | --- |
| `Chloros:ProcessingLevel` | `index`（したがって、ファイルがキャプチャとして再インポートされることはありません） |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / 使用された計算式 |
| `Chloros:IndexLevel` | `reflectance` または `radiance` — インデックスの算出元（どちらでもない場合は記載なし） |
| `Chloros:IlluminationInvariant` | 反射率から算出されたインデックスの場合は `True`、放射度またはセンサー値から算出された場合は `False` |
| `Chloros:ChlorosPipelineVersion` | インデックスを作成したChlorosのバージョン |

`Chloros:PixelScale`はインデックス製品から削除されました — 上記の`IndexMin` / `IndexMax`を使用してください。 完全な読み込み手順は、[CLIリファレンス](reference/cli-reference.md)の「整数インデックスラスタは画像ごとのコントラストストレッチである」の項に記載されています。

## 反射率ピクセル値の読み取り

反射率は整数のデジタル数値（DN）として格納されており、**ρ = 1.0（反射率 100%）に相当する DN は、ソースカメラによって異なります**：

| ソースカメラ | ρ = 1.0 に対応する DN | 判別方法 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (許容範囲は ρ 2.0 まで) | ファイルに XMP タグ `Chloros:PixelScale=32768` が付与されている。 |
| Survey3 | `65535`（ρ 1.0でクリップ） | `Chloros:PixelScale`タグなし — その欠如が識別点となります。 （Survey3の製品には、バージョン1.3.0以降、すべての処理モードにおいて、`Chloros:*`などの他の`Chloros:CalibrationApplied`処理タグが含まれています。） |

**`Chloros:PixelScale` XMPタグを読み取り、それを除算する**ようにし、定数であると仮定しないこと。 このタグは uint16 ドメインで定義されているため、再スケーリングを行う出力形式間でも `32768` のまま維持されます。保存されたデータ型をまず uint16 に正規化してください（8 ビットの場合は ×257、float32 の場合は ×65535）。

{% hint style="warning" %}
**8ビットキャプチャ。** 8ビットキャプチャのエクスポート値は8ビットです。その反射率は再スケーリングされ、`Chloros:PixelScale=32768`というタグが付けられるため、スケールは維持されます。 8ビットのDNデータおよびバージョン1.3.0以前に書き込まれた反射率ファイルは、0～255にクリップされており、スケールタグは含まれていません。LATTICEの反射率ファイルにタグがない場合は、スケールが設定されていると仮定せず、代わりに16ビットまたは32ビットで再エクスポートしてください。
{% endhint %}

完全なルール（MicaSense互換タグを含む）については、[CLIリファレンス](reference/cli-reference.md)の**「反射率ピクセルの読み取り」**を参照してください。
