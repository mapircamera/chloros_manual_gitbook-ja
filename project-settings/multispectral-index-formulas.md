---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# 多波長インデックスの計算式

以下のインデックスの計算式では、Survey3フィルターの平均透過率範囲を組み合わせて使用しています：

<table><thead><tr><th align="center">Survey3 フィルターの色</th><th width="196.199951171875" align="center">Survey3 フィルター名</th><th width="159.800048828125" align="center">透過率範囲 (FWHM)</th><th align="center">平均透過率</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468～483nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN - Cyan</td><td align="center">476～512 nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543～558 nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598～640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653～668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712～735nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798～848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835～865nm</td><td align="center">850 nm</td></tr></tbody></table>これらの式を使用する場合、名称の末尾に「\_1」または「\_2」が付くことがありますが、これは、NIRフィルターのうち、NIR1またはNIR2のどちらが使用されたかを示しています。

LATTICE M3C（バイエル・トリプルバンドパス）カメラの場合、同じインデックスエンジンが M3C フィルターのバンドを使用します：

| M3C フィルター | バンド 1（中心/FWHM） | バンド 2 (中心波長/FWHM) | バンド 3 (中心波長/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3Mカメラはシングルバンド（1台のカメラにつき1つの狭帯域フィルター）であるため、M3M画像単体ではマルチバンドインデックスは計算されません。M3Mを使用してインデックスを計算するには、 2台以上のカメラをアラインメント済みのマルチバンド・スタックに結合し、LATTICEインデックス・エンジン（`chloros-cli lattice index`、またはGUIのライブ・インデックス・カルキュレーター）を使用してください。

***

## 各インデックス名の適用範囲

Chlorosには**3つ**のインデックスサーフェスがあり、それぞれのプリセットリストは同一ではありません。このセクションを参照して、使用予定の場所でその名前が機能するかどうかを確認してください。

| 場所 | 適用されるリスト | 数 |
| --- | --- | --- |
| プロジェクト設定 → インデックス → インデックスの追加 (GUI) | サーフェス 1 | 27 |
| 画像ビューア [インデックス/LUTサンドボックス](../image-viewer-gui/index-lut-sandbox.md) (GUI) | サーフェス 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | サーフェス 2 | 22 |
| SDK `process_folder(indices=[...])` | サーフェス 2 | 22 |
| `chloros-cli lattice index --preset` | サーフェス 3 | 22（別の 22） |
| [カメラ]タブのライブインデックス計算機 | サーフェス 3 | 22（別の 22） |

Surface 1 および 2 は、**1 台のカメラから 1 枚ずつ画像**を処理し、そのカメラのフィルターチャンネルに紐付けられたシンボルスロット `x`/`y`/`z`(/`a`) を使用し、そのカメラのフィルターチャンネルにバインドされます。Surface 3は、**アラインメント済みのマルチバンドスタック**（複数のLATTICEカメラを1つのキューブにコレジストレーションしたもの）を処理し、チャンネルを小文字の名前で参照します。

### 1. GUI の「プロジェクト設定」／「画像ビューア」のサンドボックス・ドロップダウン — 27 の数式

ドロップダウンには、以下の順序で一覧が表示されます（これは挿入順であり、アルファベット順ではありません）：

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

GUI では、カメラのフィルターチャンネルを数式のバンドスロットにドラッグして割り当てるため、カメラがサポートする任意のバンド割り当てで、どの数式も使用できます。保存済みのカスタム数式は、このリストの下に追加表示されます。

**GUI限定の5つの**数式（CLI/SDK `--indices`のリストでは使用できないもの）は、次のように実装されています：

| GUI専用プリセット | 数式（実装例） | スロット |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y || FCI2 | `x*y` | x, y |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

各項目に対する想定されるマッピングは、このページの下方にあるそれぞれのセクションに記載されています（例えば、GARI では x=Green、 y=NIR、z=Blue、a=Redとなる）。GARIは、4番目のスロットを使用するChloros において、4番目のスロットを使用する唯一の数式です。

### 2. CLI / SDK `--indices` の名前展開 — 22のプリセット

`chloros-cli process --indices` オプション（および SDK、`indices` パラメータ）では、以下のプリセット名を使用できます：

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**不明なインデックス名は、警告なしにスキップされます。** このリストに含まれていない名前（GUI専用式である`FCI1`、`FCI2`、`GARI`、 `GEMI`、`LCI`、およびGUIで保存した任意のカスタム数式を含む）は、ログに通知が表示されるだけで破棄されます。実行はそのインデックスなしで継続され、実行自体は依然として成功と報告されます。通知は次のように出力されます：

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

名前の照合は、空白を削除した後に大文字小文字を区別せずに実行されるため、`ndvi`、`NDVI`、および` NDVI `は同じプリセットとなります。 また、カメラのフィルターが提供していないバンドを必要とするプリセットもスキップされます。
{% endhint %}

実装された正確な数式（記号 `x`/`y`/`z` はバンドスロットを表し、プリセットごとのデフォルトのマッピングが示されています）：

| プリセット | 数式（実装済み） | デフォルトのフィルター | スロット (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### プリセット名がどのようにバンド位置に変換されるか

`NDVI` のような名前のみを指定した場合、Chloros は、各シンボルがどのファイルのどのチャンネルを読み込むかを決定する必要があります。これには、フィルターコードを各チャンネルの配列位置にマッピングする以下のテーブルが使用されます：

| フィルタコード | チャンネル → 配列インデックス |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red`はOrangeの別名として扱われ、これも0となる) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0、Green 1、Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

プロジェクトにそのフィルターが適用された画像が含まれている場合、プリセットの**デフォルトフィルター**（上記の「デフォルトフィルター」列）が使用されます。含まれていない場合、 Chlorosは、プロジェクト内に実際に存在するフィルターを`RGN, OCN, NGB, RGB, RE, NIR`の順序でスキャンし、プリセットが必要とするすべてのチャンネルを供給できる最初のフィルターを選択します。 どのフィルターも要件を満たせない場合、その実行ではプリセットは破棄されます。これが、OCNのみのデータセットに対して`NDVI`を要求しても、依然として妥当な結果が得られる理由です — OCNのOrangeおよびNIRの位置にバインドされるからです。

LATTICE M3C モデル文字列には、`F` という接頭辞が付いたフィルターが含まれています (`LATT-M3C-L41-FRGN`) が付いたフィルターを含んでいますが、フィルターコードが画像から読み取られる際にこのプレフィックスは削除されるため、FRGN カメラは上の `RGN` 行を通じて解決を行い、特別な処理は必要ありません。

### 3. LATTICE インデックスエンジン (`lattice index --preset`、ライブインデックス計算機) — 22 種類のプリセット

LATTICE エンジンは、位置合わせ済みのマルチバンド・スタック（ライブ・アレイまたはエクスポートされたマルチバンド TIFF）で動作し、小文字のチャンネル名を使用します（`red`、 `green`、`blue`、`red_edge`、`nir`）。そのプリセット一覧は、上記の2つとは異なります：

| プリセット | 式 | チャンネル |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | red, nir |
| GNDVI | `(nir - green) / (nir + green)` | 赤、NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | 青、NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | 赤\_エッジ, 近赤外 |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | 青、緑、近赤外 |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | 赤、近赤外 |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | 赤、近赤外 |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | 赤、近赤外 |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | 青、赤、近赤外 |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | 赤、近赤外 |
| CVI | `(nir / green) - (red / green)` | 赤、緑、近赤外 |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | 赤、近赤外 |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | 赤、近赤外 |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | 赤、緑、近赤外 |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | 赤、緑、青 |
| NGRDI | `(green - red) / (green + red)` | 赤、緑 |
| VARI | `(green - red) / (green + red - blue)` | 赤、緑、青 |
| TGI | `green - 0.39*red - 0.61*blue` | 赤、緑、青 |
| EXG | `2*green - red - blue` | 赤、緑、青 |
| CIRE | `(nir / red_edge) - 1` | 赤\_エッジ, NIR |
| CIGREEN | `(nir / green) - 1` | 緑, NIR |
| NDWI | `(green - nir) / (green + nir)` | 緑、NIR |

インストール済みのビルドからこの表を出力するには `chloros-cli lattice index --list-presets` を実行し、利用可能なカラーグラデーションを確認するには `--list-gradients` を実行してください。 チャンネル記号は大文字と小文字が区別され、プリセットの小文字名（例：`--channel red=Red_660 --channel nir=NIR_850`）と一致させる必要があります。

***

## CVI

GUI および CLI/SDK プリセットリストで実装されているように、CVI は「比率の比率」の式です:

$$
CVI = {(z / y) \over (x / y)}
$$

デフォルトの RGB チャンネルマッピングは、x=Red、y=Green、z=Blue です。 GUI では、カメラの任意のチャンネルを x/y/z スロットにドラッグして配置できます。なお、LATTICE インデックスエンジンの `CVI` プリセットでは、別の計算式が使用されています。 `(NIR / Green) - (Red / Green)` — ご使用のサーフェスについては、上記の表をご確認ください。

***

## ENDVI - 拡張正規化植生指数

この指数は、NIRおよび緑チャンネルに加え、青チャンネルも使用します。赤バンドの代わりに青バンドが使用されるNGBフィルター付きカメラで広く利用されています。

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

実装は記号式 `((x+y)-(2*z))/((x+y)+(2*z))` となります。カメラのNIRおよびGreenチャンネルをx/yスロットに、Blueをzスロットに割り当てます（NGBカメラの場合： x=NIR、y=Green、z=Blue）。

***

## EVI - 拡張植生指数

この指数は、もともとMODISデータで使用するために開発されたもので、葉面積指数が高い領域における植生信号を最適化することで、NDVIを改良したものです （LAI）において植生信号を最適化することで、MODISデータとの併用を想定して開発されたものです。NDVIが飽和しやすい高LAI地域において、最も有用です。 このデータセットは、青色反射率領域を用いて土壌の背景信号を補正し、エアロゾル散乱を含む大気の影響を低減しています。

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVIの値は、植生ピクセルにおいて0から1の範囲にあるべきです。雲 や白い建物などの明るい要素、および水などの暗い要素は、EVI 画像において異常なピクセル値を引き起こす可能性があります。EVI 画像を作成する前に、 、反射率画像から雲や明るい特徴をマスクで除去し、必要に応じてピクセル値を0から1の範囲に閾値処理する必要があります。

_参考文献：Huete, A. 他、「MODIS 植生指数の放射測定および生物物理学的性能の概要」。『Remote Sensing of Environment』83 (2002):195–213._

***

## FCI1 - 森林被覆指数 1

_GUI限定 — CLI/SDK `--indices`プリセットとしては利用できません。_

この指数は、レッドエッジバンドを含むマルチスペクトル反射率画像を用いて、森林の樹冠を他の種類の植生と区別します。

$$
FCI1 = Red * RedEdge
$$

森林地域では、樹木の反射率が低く、樹冠内に影が生じるため、FCI1の値が低くなります。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. 「多スペクトル画像のための堅牢な森林被覆指数」『Photogrammetric Engineering &amp; Remote Sensing』84.8 (2018): 505-512._

***

## FCI2 - 森林被覆指数 2

_GUI限定 — CLI/SDK `--indices` プリセットとしては利用できません。_

この指数は、レッドエッジ帯を含まないマルチスペクトル反射率画像を用いて、森林の樹冠を他の種類の植生から区別します。

$$
FCI2 = Red * NIR
$$

森林地域では、樹木の反射率が低く、樹冠内に影が存在するため、FCI2の値が低くなります。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. 「マルチスペクトル画像のための堅牢な森林被覆指数」『Photogrammetric Engineering &amp; Remote Sensing』84巻8号 (2018): 505-512._

***

## GEMI - 地球環境モニタリング指数

_GUI限定 — CLI/SDK `--indices`プリセットとしては利用できません。_

この非線形植生指数は、衛星画像を用いた地球規模の環境モニタリングに使用され、大気の影響を補正することを目的としています。NDVIと類似していますが、大気の影響に対する感度が低くなっています。 裸地の影響を受けるため、植生がまばらまたは中程度の密度の地域での使用は推奨されません。

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

式：

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_参考文献：Pinty, B.、および M. Verstraete. 「GEMI：衛星による全球植生モニタリングのための非線形指数」。『Vegetation』101 (1992): 15-20._

***

## GARI - Green 大気影響耐性指数

_GUI限定 — CLI/SDK `--indices` プリセットとしては利用不可。_

この指数は、NDVIに比べ、幅広いクロロフィル濃度に対してより感度が高く、大気の影響に対しては感度が低くなっています。

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

ガンマ定数は、大気中のエアロゾル状態に依存する重み付け関数である。ENVIでは、Gitelson、Kaufman、およびMerzylak（1996年、296ページ）が推奨する値である1.7を使用している。

_参考文献：Gitelson, A., Y. Kaufman、および M. Merzylak。「EOS-MODIS による地球規模の植生リモートセンシングにおける Green チャネルの利用」。『Remote Sensing of Environment』58 (1996): 289-298._

***

## GCI - Green クロロフィル指数

この指数は、幅広い植物種にわたる葉のクロロフィル含有量を推定するために用いられる。

$$
GCI = {NIR \over Green} - 1
$$

広帯域のNIRおよび緑色波長域を利用することで、クロロフィル含有量の予測精度が向上するとともに、感度の高さと高い信号対雑音比が得られる。

_参考文献：Gitelson, A., Y. Gritz, and M. Merzlyak. 「葉のクロロフィル含有量と分光反射率との関係、および高等植物の葉における非破壊的クロロフィル評価のためのアルゴリズム」 Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green 葉指数

この指数は、もともとデジタル RGB カメラを使用して小麦の被覆率を測定するために設計されたもので、赤、緑、青のデジタル数値（DN）は 0 から 255 の範囲である。

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLIの値は-1から+1の範囲です。負の値は土壌や非生物的特徴を表し、正の値は緑色の葉や茎を表します。

_参考文献：Louhaichi, M., M. Borman、および D. Johnson。「小麦への放牧の影響を記録するための空間位置特定プラットフォームおよび航空写真」。『Geocarto International』16巻、第1号（2001年）：65-70。_

***

## GNDVI - Green 正規化差分植生指数

この指数はNDVIと類似しているが、赤色スペクトルではなく540～570 nmの緑色スペクトルを測定するという点が異なる。この指数は、NDVIよりもクロロフィル濃度に敏感である。

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_参考文献：Gitelson, A.、M. Merzlyak. 「高等植物の葉におけるクロロフィル濃度のリモートセンシング」『Advances in Space Research』22 (1998): 689-692._

***

## GOSAVI - Green 最適化土壌補正植生指数

この指数は、もともとカラー赤外線写真を用いてトウモロコシの窒素要求量を予測するために設計されたものである。OSAVIと類似しているが、 ただし、緑バンドを赤バンドに置き換えている点が異なる。

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_参考文献：Sripada, R. 他「航空カラー・赤外線写真を用いたトウモロコシの生育期中の窒素必要量の決定」。 博士論文、ノースカロライナ州立大学、2005年。_

***

## GRVI - Green 比植生指数

この指数は、緑および赤の反射率が葉の色素の変化に強く影響を受けるため、森林林冠における光合成速度に敏感である。

$$
GRVI = {NIR \over Green }
$$

_参考文献：Sripada, R. 他「トウモロコシのシーズン初期における窒素要求量の判定のための航空カラー赤外線写真」 Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green 土壌補正植生指数

この指数は、もともとトウモロコシの窒素必要量を予測するために、カラー赤外線写真を用いて設計されたものである。SAVIと類似しているが、緑バンドの代わりに赤バンドが使用されている。

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_参考文献：Sripada, R. 他「航空カラー・赤外線写真を用いたトウモロコシの生育期中の窒素必要量の決定」。博士論文、ノースカロライナ州立大学、2005年。_

***

## LAI - 葉面積指数

この指数は、葉の被覆率を推定し、作物の生育や収量を予測するために使用されます。ENVIは、Boeghらによる以下の経験式を用いて、緑色のLAIを算出します (2002)による以下の経験式を用いて、緑色LAIを算出する：

$$
LAI = 3.618 * EVI - 0.118
$$

ここで、EVIは以下の通りである：

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

LAIの値は通常、およそ0から3.5の範囲にあります。ただし、シーンに雲やその他の明るい要素が含まれており、ピクセルが飽和している場合、LAIの値は3.5を超えることがあります。 LAI画像を作成する前に、シーンから雲や明るい要素をマスクアウトしておくことが理想的です。

_参考文献：Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, および A. Thomsen. 「農業における葉面積指数、窒素濃度、および光合成効率の定量化のための航空多スペクトルデータ」。 Remote Sensing of Environment 81, no. 2-3 (2002): 179-193._

***

## LCI - 葉のクロロフィル指数

_GUI 限定 — CLI/SDK `--indices` プリセットとしては利用不可._

この指数は、クロロフィル吸収による反射率の変動に敏感な、高等植物のクロロフィル含有量を推定するために使用されます。

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_参考文献：Datt, B. 「ユーカリの葉の水分含有量のリモートセンシング」『Journal of Plant Physiology』154巻1号（1999年）：30-36頁。_

***

## MNLI - 修正非線形指数

この指数は、土壌バックグラウンドを考慮するために土壌補正植生指数（SAVI）を組み込んだ、非線形指数（NLI）の改良版である。ENVIでは、樹冠バックグラウンド補正係数（_L_）の値として0.5を使用する。

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_参考文献：Yang, Z., P. Willis, and R. Mueller. 「バンド比強化AWIFS画像が作物分類精度に与える影響」 Pecora 17 リモートセンシングシンポジウム論文集 (2008), コロラド州デンバー。_

***

## MSAVI2 - 修正土壌補正植生指数 2

この指数は、Qiら（1994）が提案したMSAVI指数の簡略版であり、土壌補正植生指数（SAVI）。これにより、土壌ノイズが低減され、植生信号のダイナミックレンジが拡大される。MSAVI2は、健康な植生を強調するために（SAVIのように）定数_L_値を使用しない帰納的手法に基づいている。

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_参考文献：Qi, J., A. Chehbouni, A. Huete, Y. Kerr, および S. Sorooshian. 「改良型土壌補正植生指数（MSAVI）」。『Remote Sensing of Environment』48 (1994): 119-126._

***

## MSR - 修正単純比

この指数は、生物物理学的パラメータとの関係を線形化するように設計された単純なNIR/Red比を改良したもので、生物物理学的パラメータとの関係を線形化するように設計されており、植生密度が高い場合においてNDVIよりも感度が高い。

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_参考文献：Chen, J. 「北方林域における植生指標および修正単純比の評価」『Canadian Journal of Remote Sensing』22 (1996): 229-242._

***

## NDRE - 正規化差分 RedEdge

この指数はNDVIと類似しているが、Redの代わりにNIRとRedEdgeのコントラストを比較する。Redはしばしば 植生のストレスを早期に検出することが多い。

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - 正規化差植生指数

この指数は、健全で緑豊かな植生の指標です。正規化差分方式と、クロロフィルの吸収および反射率が最も高い領域を利用していることから、幅広い条件下で堅牢な性能を発揮します。ただし、植生が密生している状況では、LAIの値が高くなりすぎると、値が飽和する可能性があります。

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

この指数の値は-1から1の範囲です。緑の植生の一般的な範囲は0.2から0.8です。

_参考文献：Rouse, J., R. Haas, J. Schell, and D. Deering. 「ERTSを用いたグレートプレーンズにおける植生システムのモニタリング」。第3回ERTSシンポジウム、NASA (1973): 309-317._

***

## NLI - 非線形指数

この指数は、多くの植生指数と地表の生物物理学的パラメータとの関係が非線形であることを前提としています。非線形になりがちな地表パラメータとの関係を線形化します。

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_参考文献：Goel, N.、および W. Qin。「様々な植生指数と LAI および Fpar との関係に対する樹冠構造の影響：コンピュータシミュレーション」。 Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - 最適化された土壌補正植生指数

この指数は、土壌補正植生指数（SAVI）に基づいている。樹冠背景補正係数として標準値 0.16 を使用している。Rondeaux (1996) は、この値を用いることで、植生被覆率が低い場合、SAVIよりも土壌の変動を効果的に反映できる一方で、植生被覆率が50%を超える場合には感度が高まることが判明した。この指標は、樹冠の間から土壌が見えるような、植生が比較的まばらな地域での使用に最適である。

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_参考文献：Rondeaux, G., M. Steven, and F. Baret. 「土壌補正植生指数の最適化」. Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - 再正規化差分植生指数

この指数は、近赤外線と赤色の波長間の差分と、NDVIを組み合わせて、健全な植生を強調する。土壌や太陽の観測幾何学の影響を受けにくい。

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_参考文献：Roujean, J.、および F. Breon. 「双方向反射率測定による植生によるPAR吸収量の推定」。『Remote Sensing of Environment』51 (1995): 375-384._

***

## SAVI - 土壌補正植生指数

この指数はNDVIと類似しているが、土壌ピクセルの影響を抑制する。これは、植生密度の関数である樹冠背景補正係数_L_を用いるため、多くの場合、植生量の事前知識が必要となる。 Huete (1988) は、一次の土壌背景変動を考慮するために、_L_=0.5 が最適値であると提案している。この指数は、植生が比較的まばらで、林冠越しに土壌が見える地域での使用に最も適している。

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_参考文献：Huete, A. 「土壌補正植生指数（SAVI）」。『Remote Sensing of Environment』 25 (1988): 295-309._

***

## TDVI - 変換差分植生指数

この指数は、都市環境における植生被覆のモニタリングに有用である。NDVIやSAVIのように飽和することはない。

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_参考文献：Bannari, A., H. Asalhi、および P. Teillet。「植生被覆マッピングのための変換差分植生指数（TDVI）」『地球科学・リモートセンシングシンポジウム（IGARSS &#x27;02）論文集』、IEEE International、第5巻（2002年）。_

***

## VARI - 可視光域の大気影響耐性指数

この指数はARVIに基づいており、大気の影響に対する感度が低く、シーン内の植生割合を推定するために使用される。

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_参考文献：Gitelson, A. 他「可視スペクトル空間における植生および土壌ライン：植生率の遠隔推定のための概念と手法」。『International Journal of Remote Sensing』23 (2002): 2537−2562._

***

## WDRVI - 広ダイナミックレンジ植生指数

この指数は NDVI と類似しているが、NDVI に対する近赤外信号と赤色信号の寄与度の不均衡を軽減するために、重み係数 (_a_) を使用している。 WDRVIは、NDVIが0.6を超える場合、植生密度が中程度から高いシーンで特に有効です。NDVIは、植生率および葉面積指数 （LAI）が増加すると、NDVIは横ばいになる傾向があるのに対し、WDRVIは、より広い範囲の植生率およびLAIの変化に対してより敏感である。

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

重み係数（_a_）の範囲は 0.1 から 0.2 である。Henebry、Viña、および Gitelson (2004)は、0.2の値を推奨している。

_参考文献_

_Gitelson, A. 「植生の生物物理学的特性の遠隔定量化のための広ダイナミックレンジ植生指数」『Journal of Plant Physiology』161巻、第2号 (2004): 165-173._

_Henebry, G., A. Viña, and A. Gitelson. &quot;広ダイナミックレンジ植生指数とそのギャップ分析への応用可能性.&quot; 『Gap Analysis Bulletin』 12: 50-56._
