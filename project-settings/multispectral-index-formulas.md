---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# 多波長インデックスの算出式

以下のインデックスの算出式では、Survey3フィルターの平均透過率範囲を組み合わせて使用しています：

<table><thead><tr><th align="center">Survey3フィルターの色</th><th width="196.199951171875" align="center">Survey3フィルター名</th><th width="159.800048828125" align="center">透過率範囲（FWHM）</th><th align="center">平均透過率</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476～512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543～558nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598～640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653～668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712～735nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798～848 nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835～865nm</td><td align="center">850 nm</td></tr></tbody></table>

これらの式を使用する場合、名称の末尾に「\_1」または「\_2」が付くことがありますが、これはNIRフィルターのうち、NIR1またはNIR2のどちらが使用されたかを示しています。

LATTICE M3C（バイエル式トリプルバンドパス）カメラの場合、同じインデックスエンジンが M3C フィルターバンドを使用します：

| M3C フィルター | バンド 1（中心波長／FWHM） | バンド 2（中心波長／FWHM） | バンド 3（中心波長／FWHM） |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3Mカメラはシングルバンド（1台のカメラにつき1つの狭帯域フィルター）であるため、M3M画像1枚だけではマルチバンドインデックスは算出されません。 M3Mでインデックスを計算するには、2台以上のカメラ画像を位置合わせしたマルチバンド・スタックに結合し、LATTICEインデックスエンジン（`chloros-cli lattice index`、またはGUIのライブインデックス計算機）を使用してください。

***

## 各インデックス名の適用場所

Chlorosには**3つ**のインデックスサーフェスがあり、それぞれのプリセットリストは同一ではありません。このセクションを使用して、使用予定の場所でその名前が機能するかどうかを確認してください。

| 場所 | 適用されるリスト | 数 |
| --- | --- | --- |
| プロジェクト設定 → インデックス → インデックスの追加 (GUI) | サーフェス 1 | 27 |
| 画像ビューア [インデックス/LUT サンドボックス](../image-viewer-gui/index-lut-sandbox.md) (GUI) | サーフェス 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Surface 2 | 22 |
| SDK `process_folder(indices=[...])` | Surface 2 | 22 |
| `chloros-cli lattice index --preset` | サーフェス 3 | 22 (別の 22) |
| [カメラ]タブのライブインデックス計算機 | サーフェス 3 | 22 (別の 22) |

Surface 1 および 2 は、**1 台のカメラから 1 枚ずつ画像**を処理し、 そのカメラのフィルターチャンネルにバインドされたシンボルスロット `x`/`y`/`z`(/`a`) を使用します。 サーフェス3は、**位置合わせされたマルチバンドスタック**（複数のLATTICEカメラを1つのキューブに位置合わせしたもの）を処理し、チャンネルを小文字の名前で参照します。

### 1. GUIのプロジェクト設定 / 画像ビューアのサンドボックスドロップダウン — 27個の式

ドロップダウンには、以下の順序でリストされます （アルファベット順ではなく、挿入順です）：

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

GUI では、カメラのフィルターチャンネルを数式のバンドスロットにドラッグして割り当てるため、カメラがサポートする任意のバンド割り当てで、どの数式も使用できます。保存済みのカスタム数式は、このリストの下に追加されます。

**GUI専用**の5つの数式（CLI/SDKの`--indices`リストでは受け付けられないもの）は、次のように実装されています：

| GUI専用プリセット | 数式（実装内容） | スロット |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a（4つのスロット） |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

各オプションの想定されるマッピングについては、このページの下部にある各セクションで説明しています（例えば、GARIでは x=Green、y=NIR、z=Blue、a=Red となります）。GARIは、Chlorosにおいて4つ目のスロットを使用する唯一の式です。

### 2. CLI / SDK `--indices` 名の展開 — 22 のプリセット

`chloros-cli process --indices` オプション（および SDK `indices` パラメータ）は、以下のプリセット名を受け付けます：

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**リストにないインデックス名は、警告なしでスキップされます。** このリストに含まれていない名前（GUI専用式である `FCI1`、`FCI2`、`GARI`、`GEMI`、 `LCI`、およびGUIで保存した任意のカスタム数式を含む）は、ログに通知が表示されるだけで除外されます。実行はそのインデックスなしで継続され、実行自体は依然として成功と報告されます。通知は次のように出力されます：

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

名前の照合は、空白を削除した後、大文字小文字を区別せずに実行されるため、`ndvi`、`NDVI`、および ` NDVI ` は同じプリセットとなります。 また、カメラのフィルターで提供されていないバンドを必要とするプリセットもスキップされます。
{% endhint %}

実装された正確な数式（記号 `x`/`y`/`z` はバンドスロットを表し、プリセットごとのデフォルトのマッピングが示されています）：

| プリセット | 数式（実装済み） | デフォルトのフィルター | スロット (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red、NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green、NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE、NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red、NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red、NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red、NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red、NIR、Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red、NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red、NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red、NIR、Blue |
| GCI | `(y/x)-1` | RGN | Green、NIR |
| GRVI | `y/x` | RGN | Green、NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green、NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green、NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red、NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red、NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red、NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red、NIR |
| CVI | `(z/y)*(x/y)` | RGN | Red、Green、NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR、Green、Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red、Green、Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red、Green、Blue |

#### プリセット名がどのようにバンド位置に変換されるか

`NDVI` のような単純な名前を指定した場合、Chloros は各シンボルがどのファイルのどのチャンネルを読み取るかを決定する必要があります。そのため、フィルターコードを各チャンネルの配列位置にマッピングする以下のテーブルを使用します：

| フィルターコード | チャンネル → 配列インデックス |
| --- | --- |
| OCN | Orange 0、Cyan 1、NIR 2（`Red`はOrangeの別名として受け入れられ、これも0となる） |
| RGN | Red 0、Green 1、NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

プロジェクトにそのフィルターの画像が含まれている場合は、プリセットの**デフォルトのフィルター**（上の表の「デフォルトのフィルター」列）が使用されます。 含まれていない場合、Chlorosはプロジェクト内に実際に存在するフィルターを`RGN, OCN, NGB, RGB, RE, NIR`の順にスキャンし、プリセットが必要とするすべてのチャンネルを提供できる最初のフィルターを選択します。該当するフィルターがない場合、その実行ではプリセットは破棄されます。 これが、OCNのみのデータセットで`NDVI`を指定しても、妥当な結果が得られる理由です。これは、OCNのOrangeおよびNIRの位置に紐づくためです。

LATTICE M3Cモデル文字列には、`F`という接頭辞が付いたフィルターが指定されています (`LATT-M3C-L41-FRGN`) を含んでいますが、画像からフィルターコードが読み取られる際にこのプレフィックスは削除されるため、FRGNカメラは上記の`RGN`行を通じて解決され、特別な処理は必要ありません。

### 3. LATTICE インデックスエンジン (`lattice index --preset`、ライブインデックス計算機) — 22 種類のプリセット

LATTICE エンジンは、アライメント済みのマルチバンド・スタック（ライブ・アレイまたはエクスポートされたマルチバンド TIFF）で動作し、小文字のチャンネル名を使用します（`red`、`green`、 `blue`、`red_edge`、`nir`）を使用します。そのプリセット一覧は、上記の2つとは異なります：

| プリセット | 計算式 | チャンネル |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | 赤、近赤外 |
| GNDVI | `(nir - green) / (nir + green)` | 赤、近赤外 |
| BNDVI | `(nir - blue) / (nir + blue)` | 青、近赤外 |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | 赤エッジ、近赤外 |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | 青、緑、近赤外 |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | 赤、近赤外 |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | 赤, 近赤外 |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | 赤, 近赤外 |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | 青, 赤, 近赤外 |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | 赤、近赤外 |
| CVI | `nir * red / green**2` | 赤、緑、近赤外 |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | 赤、近赤外 |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | 赤、近赤外 |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | 青、赤、近赤外 |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | 赤、緑、青 |
| NGRDI | `(green - red) / (green + red)` | 赤、緑 |
| VARI | `(green - red) / (green + red - blue)` | 赤、緑、青 |
| TGI | `green - 0.39*red - 0.61*blue` | 赤、緑、青 |
| EXG | `2*green - red - blue` | 赤、緑、青 |
| CIRE | `(nir / red_edge) - 1` | 赤\_エッジ, NIR |
| CIGREEN | `(nir / green) - 1` | 緑, NIR |
| NDWI | `(green - nir) / (green + nir)` | 緑、近赤外 |

インストール済みのビルドからこの表を出力するには `chloros-cli lattice index --list-presets` を実行し、 利用可能なカラーグラデーションを表示するには `--list-gradients` を実行してください。チャンネル記号は大文字と小文字が区別され、プリセットの小文字名（例：`--channel red=Red_660 --channel nir=NIR_850`）と一致する必要があります。

***

## CVI - クロロフィル植生指数

この指数は、幅広い樹冠密度にわたって葉のクロロフィル含有量に敏感です。NIR、赤、緑のバンドを使用します。

$$
CVI = NIR * {Red \over Green^{2}}
$$

GUIおよびCLI/SDKのプリセットリストでは、シンボル式は `(z/y)*(x/y)` となり、x=Red、y=Green、z=NIR （RGNカメラのバンド）であり、LATTICEインデックスエンジンのプリセットは `nir * red / green**2` です。すべての表面で同じインデックスが算出されます。

_参考文献：Vincini, M., E. Frazzi, and P. D'Alessio. "A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale." Precision Agriculture 9 (2008): 303-319._

***

## ENDVI - 拡張正規化差分植生指数

この指数は、NIRおよび緑チャンネルに加え、青チャンネルも使用します。赤バンドの代わりに青バンドを使用するNGBフィルター搭載カメラで広く利用されています。

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

実装にはシンボル式 `((x+y)-(2*z))/((x+y)+(2*z))` を使用します。カメラのNIRおよびGreenチャンネルをx/yスロットに、Blueをzに割り当ててください（NGBカメラの場合：x=NIR、y=Green、z=Blue）。

***

## EVI - 拡張植生指数

この指数は、もともとMODISデータで使用するために開発されたもので、葉面積指数（LAI）の高い地域における植生信号を最適化することで、NDVIを改良したものです。NDVIが飽和しやすい高LAI地域で最も有用です。青色反射率領域を利用して、土壌の背景信号を補正し、エアロゾルの散乱を含む大気の影響を低減します。

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVIの値は、植生ピクセルにおいて0から1の範囲にあるべきです。雲や白い建物などの明るい特徴や、水域などの暗い特徴は、EVI画像において異常なピクセル値を引き起こす可能性があります。EVI画像を作成する前に、 反射率画像から雲や明るい特徴をマスクで除去し、必要に応じてピクセル値の閾値を0から1に設定する必要があります。

_参考文献：Huete, A. 他「MODIS植生指数の放射測定および生物物理学的性能の概要」『Remote Sensing of Environment』83 (2002):195–213._

***

## FCI1 - 森林被覆指数 1

_GUI専用 — CLI/SDKの`--indices`プリセットとしては利用できません。_

この指数は、レッドエッジバンドを含むマルチスペクトル反射率画像を用いて、森林の樹冠を他の種類の植生から区別します。

$$
FCI1 = Red * RedEdge
$$

森林地域では、樹木の反射率が低く、樹冠内に影が存在するため、FCI1の値は低くなります。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. 「多スペクトル画像のための堅牢な森林被覆指数」。『Photogrammetric Engineering & Remote Sensing』84.8 (2018): 505-512._

***

## FCI2 - 森林被覆指数 2

_GUI専用 — CLI/SDKの`--indices`プリセットとしては利用できません。_

この指標は、レッドエッジ帯を含まないマルチスペクトル反射率画像を用いて、森林樹冠を他の種類の植生から区別します。

$$
FCI2 = Red * NIR
$$

森林地域では、樹木の反射率が低く、樹冠内に影が存在するため、FCI2の値は低くなります。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. 「多スペクトル画像のための堅牢な森林被覆指数」。『Photogrammetric Engineering & Remote Sensing』84.8 (2018): 505-512。_

***

## GEMI - 地球環境モニタリング指数

_GUI専用 — CLI/SDKの`--indices`プリセットとしては利用できません。_

この非線形植生指数は、衛星画像を用いた地球規模の環境モニタリングに使用され、大気の影響を補正することを目的としています。 NDVIと類似しているが、大気の影響に対する感度は低い。裸地の影響を受けるため、植生がまばらまたは中程度の密度の地域での使用は推奨されない。

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

ここで：

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_参考文献：Pinty, B.、および M. Verstraete。「GEMI：衛星による全球植生モニタリングのための非線形指数」。『Vegetation』101 (1992): 15-20._

***

## GARI - Green Atmospherically Resistant Index

_GUI専用 — CLI/SDKの`--indices`プリセットとしては利用できません。_

この指数は、NDVIに比べて、幅広いクロロフィル濃度に対してより感度が高く、大気の影響に対しては感度が低くなっています。

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

ガンマ定数は、大気中のエアロゾル状態に依存する重み付け関数である。ENVIでは、Gitelson、Kaufman、およびMerzylakが推奨する値である1.7を採用している (1996, 296ページ)による推奨値です。

_参考文献：Gitelson, A., Y. Kaufman, and M. Merzylak. "Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS." Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - グリーンクロロフィル指数（Green Chlorophyll Index）

この指数は、幅広い植物種にわたる葉のクロロフィル含有量を推定するために使用されます。

$$
GCI = {NIR \over Green} - 1
$$

広範囲のNIRおよび緑色波長を利用することで、クロロフィル含有量の予測精度が向上するとともに、感度の高さと高い信号対雑音比が得られる。

_参考文献：Gitelson, A., Y. Gritz, and M. Merzlyak. 「葉のクロロフィル含有量と分光反射率との関係、および高等植物の葉における非破壊的クロロフィル評価のためのアルゴリズム」。『Journal of Plant Physiology』160 (2003): 271-282._

***

## GLI - グリーンリーフインデックス（Green Leaf Index）

この指標は、もともとデジタルRGBカメラを用いて小麦の被覆率を測定するために設計されたもので、赤、緑、青のデジタル数値（DN）は0から255の範囲をとる。

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLIの値は-1から+1の範囲である。負の値は土壌や非生物的要素を表し、正の値は緑色の葉や茎を表す。

_参考文献：Louhaichi, M., M. Borman, and D. Johnson. 「小麦への放牧の影響を記録するための空間位置特定プラットフォームおよび航空写真」。『Geocarto International』16巻1号（2001年）：65-70頁。_

***

## GNDVI - 緑色正規化植生指数（Green Normalized Difference Vegetation Index）

この指数はNDVIと類似しているが、赤色スペクトルではなく540～570 nmの緑色スペクトルを測定するという点が異なる。この指数はNDVIよりもクロロフィル濃度に対して感度が高い。

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_参考文献：Gitelson, A.、M. Merzlyak. 「高等植物の葉におけるクロロフィル濃度のリモートセンシング」『Advances in Space Research』22 (1998): 689-692._

***

## GOSAVI - 緑帯最適化土壌補正植生指数（Green Optimized Soil Adjusted Vegetation Index）

この指数は、もともとカラー赤外線写真を用いてトウモロコシの窒素要求量を予測するために設計された。OSAVIと類似しているが、赤帯の代わりに緑帯を使用している。

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_参考文献：Sripada, R. 他「航空カラー・赤外線写真を用いたトウモロコシの生育期における窒素必要量の決定」。博士論文、ノースカロライナ州立大学、2005年。_

***

## GRVI - 緑色比率植生指数（Green Ratio Vegetation Index）

この指数は、緑色および赤色の反射率が葉の色素の変化に強く影響を受けるため、森林樹冠の光合成速度に敏感である。

$$
GRVI = {NIR \over Green }
$$

_参考文献：Sripada, R. 他「トウモロコシの生育初期における窒素要求量を推定するための航空カラー赤外線写真」『Agronomy Journal』98 (2006): 968-977._

***

## GSAVI - 緑色土壌補正植生指数（Green Soil Adjusted Vegetation Index）

この指数は、もともとトウモロコシの窒素要求量を予測するために、カラー赤外線写真を用いて設計されたものである。SAVIと類似しているが、赤バンドの代わりに緑バンドを使用している。

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_参考文献：Sripada, R. 他「航空カラー赤外線写真を用いたトウモロコシの生育期中の窒素要求量の決定」。博士論文、ノースカロライナ州立大学、2005年。_

***

## LAI - 葉面積指数

この指数は、葉の被覆率を推定し、作物の生育や収量を予測するために使用されます。ENVIは、Boeghら（2002）による以下の経験式を用いて、緑のLAIを算出します：

$$
LAI = 3.618 * EVI - 0.118
$$

ここで、EVIは以下の通りである：

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

LAIの値は通常、およそ0から3.5の範囲にあります。ただし、シーンに雲やその他の明るい要素が含まれており、それらが飽和ピクセルを生成する場合、LAIの値は3.5を超えることがあります。LAI画像を作成する前に、シーンから雲や明るい要素をマスクで除去することが理想的です。

_参考文献：Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, および A. Thomsen. 「農業における葉面積指数、窒素濃度、および光合成効率の定量化のための航空多スペクトルデータ。」『Remote Sensing of Environment』81巻、第2-3号（2002年）：179-193頁。_

***

## LCI - 葉クロロフィル指数

_GUIのみ — CLI/SDKの`--indices`プリセットとしては利用不可。_

この指数は、高等植物のクロロフィル含有量を推定するために使用され、クロロフィルによる吸収に起因する反射率の変動に敏感である。

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_参考文献：Datt, B. 「ユーカリの葉の水分含有量のリモートセンシング」『Journal of Plant Physiology』154巻、第1号（1999年）：30-36._

***

## MNLI - 修正非線形指数

この指数は、土壌バックグラウンドを考慮するために土壌補正植生指数（SAVI）を取り入れた、非線形指数（NLI）の改良版である。ENVIでは、樹冠バックグラウンド補正係数（_L_）の値として0.5を使用する。

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_参考文献：Yang, Z., P. Willis, and R. Mueller. 「バンド比強化AWIFS画像が作物分類精度に与える影響」. Pecora 17 リモートセンシングシンポジウム論文集 (2008), コロラド州デンバー._

***

## MSAVI2 - 改良型土壌補正植生指数 2

この指数は、Qiらによって提案されたMSAVI指数の簡略版である （1994）によって提案されたMSAVI指数の簡略版であり、土壌調整植生指数（SAVI）を改良したものである。これにより、土壌ノイズが低減され、植生信号のダイナミックレンジが拡大される。MSAVI2は、健全な植生を強調するために（SAVIのように）定数の_L_値を使用しない帰納的手法に基づいている。

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_参考文献：Qi, J., A. Chehbouni, A. Huete, Y. Kerr, and S. Sorooshian. "A Modified Soil Adjusted Vegetation Index." Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR - 修正単純比

この指数は、単純なNIR/Red比を修正したもので、生物物理学的パラメータとの関係を線形化するように設計されており、植生密度が高い場合、NDVIよりも感度が高い。

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_参考文献：Chen, J. 「北方林地における植生指数および修正単純比の評価」『Canadian Journal of Remote Sensing』22 (1996): 229-242._

***

## NDRE - 正規化差分RedEdge

この指数はNDVIと類似しているが、Redの代わりにNIRとRedEdgeのコントラストを比較するため、植生のストレスをより早期に検出できることが多い。

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - 正規化差分植生指数

この指標は、健全で緑豊かな植生の度合いを測るものである。正規化差分方式と、クロロフィルの吸収および反射率が最も高い領域を利用していることから、幅広い条件下で頑健な性能を発揮する。ただし、 LAIが高くなるような植生が密な条件下では、値が飽和することがあります。

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

この指数の値の範囲は-1から1です。緑の植生における一般的な範囲は0.2から0.8です。

_参考文献：Rouse, J., R. Haas, J. Schell, and D. Deering. 「ERTSを用いたグレートプレーンズにおける植生システムのモニタリング」。第3回ERTSシンポジウム、NASA (1973): 309-317._

***

## NLI - 非線形指数

この指数は、多くの植生指数と地表の生物物理学的パラメータとの関係が非線形であることを前提としています。非線形になりがちな地表パラメータとの関係を線形化します。

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_参考文献：Goel, N.、および W. Qin。「様々な植生指数とLAIおよびFparとの関係に対する樹冠構造の影響：コンピュータシミュレーション」。『Remote Sensing Reviews』10 (1994): 309-347._

***

## OSAVI - 最適化土壌補正植生指数

この指数は、土壌補正植生指数（SAVI）に基づいている。樹冠背景補正係数として、標準値0.16を用いる。 Rondeaux (1996) は、この値を用いることで、植生被覆率が低い場合にはSAVIよりも土壌の変動をより大きく反映できる一方で、植生被覆率が50%を超える場合には感度が高まることを明らかにした。この指数は、樹冠の間から土壌が見えるような、比較的植生がまばらな地域での使用に最も適している。

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_参考文献：Rondeaux, G., M. Steven, and F. Baret. "Optimization of Soil-Adjusted Vegetation Indices." Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - 再正規化差分植生指数

この指数は、近赤外波長と赤色波長の差分とNDVIを組み合わせて、健全な植生を強調する。土壌や太陽の観測幾何学の影響を受けにくい。

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_参考文献：Roujean, J.、および F. Breon。「双方向反射率測定による植生が吸収するPARの推定」。『Remote Sensing of Environment』51 (1995): 375-384._

***

## SAVI - 土壌補正植生指数

この指数はNDVIと類似しているが、土壌ピクセルの影響を抑制する。樹冠背景補正係数_L_を使用しており、これは植生密度の関数であり、多くの場合、植生量の事前知識を必要とする。 Huete (1988) は、一次の土壌背景変動を考慮するために、_L_=0.5 が最適値であると提案している。この指数は、植生が比較的まばらで、樹冠越しに土壌が見える地域での使用に最も適している。

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_参考文献：Huete, A. 「A Soil-Adjusted Vegetation Index (SAVI)」。Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - 変換差分植生指数

この指数は、都市環境における植生被覆のモニタリングに有用である。NDVIやSAVIのように飽和することはない。

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_参考文献：Bannari, A., H. Asalhi、および P. Teillet。「植生被覆マッピングのための変換差分植生指数（TDVI）」『地球科学・リモートセンシングシンポジウム（IGARSS '02）論文集』、IEEE International、第5巻（2002年）。_

***

## VARI - 可視光域大気影響耐性指数

この指数はARVIに基づいており、大気の影響に対する感度が低く、シーン内の植生割合を推定するために用いられる。

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_参考文献：Gitelson, A., et al. 「可視スペクトル空間における植生および土壌の境界線：植生率のリモート推定のための概念と手法」。『International Journal of Remote Sensing』23 (2002): 2537−2562._

***

## WDRVI - 広ダイナミックレンジ植生指数

この指数はNDVIと類似しているが、NDVIに対する近赤外および赤色信号の寄与度の格差を低減するために重み係数（_a_）を用いている。WDRVIは、NDVIが0.6を超える中～高密度の植生が存在するシーンにおいて特に有効である。 NDVIは植生率や葉面積指数（LAI）が増加すると横ばいになる傾向があるのに対し、WDRVIはより広い範囲の植生率やLAIの変化に対してより敏感である。

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

重み係数（_a_）の範囲は0.1から0.2である。Henebry、Viña、およびGitelson（2004）は、0.2の値を推奨している。

_参考文献_

_Gitelson, A. 「植生の生物物理的特性のリモート定量化のための広ダイナミックレンジ植生指数（WDRVI）」。『Journal of Plant Physiology』161巻、第2号（2004年）：165-173。_

_Henebry, G., A. Viña、および A. Gitelson。「広ダイナミックレンジ植生指数（WDRI）とギャップ分析におけるその潜在的有用性。」『Gap Analysis Bulletin』12: 50-56._
