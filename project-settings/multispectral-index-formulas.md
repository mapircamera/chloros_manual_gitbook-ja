---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# 多波長インデックスの計算式

以下のインデックスの計算式は、Survey3フィルターの平均透過率範囲を組み合わせて使用しています：

<table><thead><tr><th align="center">Survey3 フィルターの色</th><th width="196.199951171875" align="center">Survey3 フィルター名</th><th width="159.800048828125" align="center">透過率範囲 (FWHM)</th><th align="center">平均透過率</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476～512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653～668 nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table>これらの式を使用する場合、名称の末尾に「\_1」または「\_2」が付くことがありますが、これはどのNIRフィルター（NIR1またはNIR2）が使用されたかを示しています。

***

## EVI - 拡張植生指数

この指数は、葉面積指数（LAI）が高い地域における植生信号を最適化することで、NDVIを改良したものであり、もともとMODISデータで使用するために開発されました。 NDVIが飽和しやすい高LAI地域において、最も有用である。青色反射率領域を用いて土壌の背景信号を補正し、エアロゾル散乱を含む大気の影響を低減する。

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVIの値は、植生ピクセルにおいて0から1の範囲にあるべきです。雲や白い建物などの明るい特徴や、水などの暗い特徴は、EVI画像において異常なピクセル値を引き起こす可能性があります。 EVI画像を作成する前に、反射率画像から雲や明るい特徴をマスクで除去し、必要に応じてピクセル値を0から1の範囲に閾値処理する必要があります。

_参考文献: Huete, A. 他「MODIS植生指数の放射測定および生物物理学的性能の概要」 Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - 森林被覆指数 1

この指数は、レッドエッジバンドを含むマルチスペクトル反射率画像を用いて、森林樹冠を他の種類の植生から識別します。

$$
FCI1 = Red * RedEdge
$$

森林地域では、樹木の反射率が低く、樹冠内に影が存在するため、FCI1の値が低くなります。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. 「多スペクトル画像のための堅牢な森林被覆指数」 Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - 森林被覆指数 2

この指数は、レッドエッジバンドを含まないマルチスペクトル反射率画像を用いて、森林樹冠を他の種類の植生から区別します。

$$
FCI2 = Red * NIR
$$

森林地域では、樹木の反射率が低く、樹冠内に影が存在するため、FCI2の値が低くなります。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - 地球環境モニタリング指数

この非線形植生指数は、衛星画像を用いた地球環境モニタリングに使用され、大気の影響を補正することを目的としています。NDVIと類似していますが、大気の影響に対する感度が低くなっています。裸地の影響を受けるため、植生がまばらまたは中程度の密度の地域での使用は推奨されません。

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

参照：

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_参考文献：Pinty, B.、および M. Verstraete. GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites. Vegetation 101 (1992): 15-20._

***

## GARI - Green 大気影響耐性指数

この指数は、NDVIと比較して、幅広いクロロフィル濃度に対してより感度が高く、大気の影響に対しては感度が低い。

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

ガンマ定数は、大気中のエアロゾル条件に依存する重み付け関数です。ENVIでは、Gitelson、Kaufman、およびMerzylak（1996年、296ページ）が推奨する値である1.7を使用しています。

_参考文献：Gitelson, A., Y. Kaufman, and M. Merzylak. &quot;Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green クロロフィル指数

この指数は、幅広い植物種にわたる葉のクロロフィル含有量を推定するために使用される。

$$
GCI = {NIR \over Green} - 1
$$

広帯域のNIRおよび緑色波長域を利用することで、感度と信号対雑音比を高めつつ、クロロフィル含有量の予測精度を向上させることができる。

_参考文献：Gitelson, A., Y. Gritz, and M. Merzlyak. &quot;Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green 葉指数

この指数は、もともとデジタルカメラ（赤、緑、青のデジタル数値（DN）が0から255の範囲）を用いて小麦の被覆率を測定するために設計されたものである。

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLIの値は-1から+1の範囲です。負の値は土壌や非生物的要素を表し、正の値は緑色の葉や茎を表します。

_参考文献：Louhaichi, M., M. Borman, and D. Johnson. 「小麦への放牧の影響を記録するための空間位置特定プラットフォームおよび航空写真」『Geocarto International』16巻1号（2001年）：65-70頁。_

***

## GNDVI - Green 正規化植生指数

この指数はNDVIと類似しているが、赤色スペクトルではなく540～570 nmの緑色スペクトルを測定する点が異なる。この指数は、NDVIよりもクロロフィル濃度に敏感である。

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_参考文献: Gitelson, A., and M. Merzlyak. &quot;Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green 最適化土壌補正植生指数

この指数は、もともとトウモロコシの窒素要求量を予測するために、カラー赤外線写真を用いて設計されたものである。OSAVIと類似しているが、緑バンドを赤バンドに置き換えている。

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_参考文献: Sripada, R., et al. 「航空カラー赤外線写真を用いたトウモロコシの生育期における窒素要求量の決定」. 博士論文、ノースカロライナ州立大学、2005年。_

***

## GRVI - Green 比植生指数

この指数は、緑および赤の反射率が葉の色素の変化に強く影響を受けるため、森林樹冠の光合成速度に敏感です。

$$
GRVI = {NIR \over Green }
$$

_参考文献：Sripada, R. 他「トウモロコシの生育初期における窒素要求量を判定するための航空カラー赤外線写真」『Agronomy Journal』98 (2006): 968-977._

***

## GSAVI - Green 土壌補正植生指数

この指数は、もともとトウモロコシの窒素要求量を予測するために、カラー赤外線写真を用いて設計されたものである。SAVIと類似しているが、赤バンドの代わりに緑バンドを使用している。

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_参考文献: Sripada, R. 他. 「航空カラー赤外線写真を用いたトウモロコシの生育期窒素要求量の決定」. 博士論文, ノースカロライナ州立大学, 2005._

***

## LAI - 葉面積指数

この指数は、葉の被覆率を推定し、作物の生育および収量を予測するために使用されます。 ENVIは、Boegh et al (2002) の以下の経験式を用いて、緑のLAIを算出します：

$$
LAI = 3.618 * EVI - 0.118
$$

ここで、EVIは：

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

LAIの値は通常、約0から3.5の範囲にあります。ただし、シーンに雲やその他の明るい要素が含まれており、それらが飽和ピクセルを生成する場合、LAIの値は3.5を超えることがあります。理想的には、LAI画像を作成する前に、シーンから雲や明るい要素をマスクアウトする必要があります。

_参考文献：Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, and A. Thomsen. &quot;Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.&quot; Remote Sensing of Environment 81, no. 2-3 (2002): 179-193._

***

## LCI - 葉のクロロフィル指数

この指数は、クロロフィル吸収による反射率の変化に敏感であり、高等植物のクロロフィル含有量を推定するために使用されます。

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_参考文献: Datt, B. 「ユーカリの葉における水分含有量のリモートセンシング」. Journal of Plant Physiology 154, no. 1 (1999): 30-36._

***

## MNLI - 修正非線形指数

この指数は、土壌背景の影響を考慮するために土壌補正植生指数（SAVI）を組み込んだ、非線形指数（NLI）の改良版である。ENVIでは、樹冠背景補正係数（_L_）の値として0.5を使用する。

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_参考文献：Yang, Z., P. Willis, and R. Mueller. &quot;Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.&quot; Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 - 修正土壌補正植生指数 2

この指数は、Qiら（1994）によって提案されたMSAVI指数の簡略版であり、土壌補正植生指数（SAVI）を改良したものです。これにより、土壌ノイズが低減され、植生信号のダイナミックレンジが拡大されます。 MSAVI2は、健全な植生を強調するために（SAVIのように）定数_L_値を使用しない帰納的手法に基づいている。

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_参考文献: Qi, J., A. Chehbouni, A. Huete, Y. Kerr, and S. Sorooshian. 「改良型土壌補正植生指数（A Modified Soil Adjusted Vegetation Index）」。Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE - 正規化差分 RedEdge

この指数はNDVIと類似しているが、Redの代わりにNIRとRedEdgeのコントラストを比較するものであり、前者は植生のストレスをより早期に検出することが多い。

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - 正規化植生指数（NDVI）

この指数は、健全な緑の植生の指標です。正規化差分方式と、クロロフィルが最も強く吸収・反射する波長域を利用していることから、幅広い条件下で高い信頼性を発揮します。ただし、植生が密生している条件下では、LAIの値が高くなり、飽和状態になることがあります。

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

この指数の値は-1から1の範囲です。緑の植生の一般的な範囲は0.2から0.8です。

_参考文献：Rouse, J., R. Haas, J. Schell, and D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309-317._

***

## NLI - 非線形指数

この指数は、多くの植生指数と地表の生物物理学的パラメータとの関係が非線形であることを前提としています。非線形になりがちな地表パラメータとの関係を線形化します。

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_参考文献：Goel, N.、および W. Qin。「様々な植生指数と LAI および Fpar との関係に対する樹冠構造の影響：コンピュータシミュレーション」。Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - 最適化された土壌補正植生指数

この指数は、土壌補正植生指数（SAVI）に基づいている。樹冠背景補正係数として標準値0.16を使用する。 Rondeaux (1996) は、この値を用いることで、植生被覆率が低い場合において SAVI よりも土壌の変動性をより高く表現できると同時に、植生被覆率が 50% を超える場合において感度が高まることを明らかにした。この指数は、樹冠を通して土壌が見えるような、比較的植生がまばらな地域での使用に最適である。

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_参考文献: Rondeaux, G., M. Steven, and F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - 再正規化植生差指数

この指数は、近赤外線と赤色波長の差分とNDVIを組み合わせて、健全な植生を強調表示する。土壌の影響や太陽の観測角度の影響を受けにくい。

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_参考文献: Roujean, J., and F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - 土壌補正植生指数

この指数はNDVIと類似しているが、土壌ピクセルの影響を抑制する。これは、植生密度の関数である樹冠背景補正係数_L_を使用しており、多くの場合、植生量の事前知識を必要とする。 Huete (1988) は、一次的な土壌背景の変動を考慮するために、_L_=0.5 が最適値であると提案している。この指数は、植生が比較的まばらで、樹冠を通して土壌が見える地域での使用に最も適している。

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_参考文献：Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - 変換差分植生指数

この指数は、都市環境における植生被覆のモニタリングに有用である。NDVIやSAVIのように飽和することはない。

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_参考文献: Bannari, A., H. Asalhi, and P. Teillet. &quot;Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping&quot; In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - 可視光域大気抵抗指数

この指数は ARVI に基づいており、大気の影響に対する感度が低く、シーン内の植生の割合を推定するために使用されます。

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_参考文献: Gitelson, A. 他. 「可視スペクトル空間における植生と土壌の境界線：植生率の遠隔推定のための概念と手法」. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - 広ダイナミックレンジ植生指数

この指数はNDVIと類似しているが、NDVIに対する近赤外線信号と赤色信号の寄与度の差を低減するために、重み係数（_a_）を使用している。 WDRVIは、NDVIが0.6を超える場合、植生密度が中程度から高いシーンにおいて特に有効です。 NDVI は、植生率および葉面積指数 (LAI) が増加すると横ばいになる傾向がありますが、WDRVI は、より広い範囲の植生率や LAI の変化に対してより敏感です。

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

重み係数（_a_）は 0.1 から 0.2 の範囲で設定可能である。Henebry、Viña、および Gitelson (2004) は 0.2 の値を推奨している。

_参考文献_

_Gitelson, A. 「植生の生物物理的特性の遠隔定量化のための広ダイナミックレンジ植生指数」『Journal of Plant Physiology』161巻2号（2004年）：165-173頁。_

_Henebry, G., A. Viña, and A. Gitelson. &quot;広ダイナミックレンジ植生指数とそのギャップ分析への応用可能性.&quot; Gap Analysis Bulletin 12: 50-56._
