---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# 多波長指数計算式

以下の指数計算式は、Survey3フィルターの平均透過率範囲を組み合わせて使用します：

<table><thead><tr><th align="center">Survey3 フィルター色</th><th width="196.199951171875" align="center">Survey3 フィルター名称</th><th width="159.800048828125" align="center">透過率範囲 (FWHM)</th><th align="center">平均透過率</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table>これらの式を使用する場合、名称は「\_1」または「\_2」で終わる場合があります。これは、NIRフィルターとしてNIR1またはNIR2のいずれが使用されたかを示します。

***

## EVI - 強化植生指数

この指数は、LAI（葉面積指数が高い領域）における植生信号を最適化することでNDVIを改良し、MODISデータとの併用を目的に開発されました。 NDVIが飽和する可能性のある高LAI地域で最も有用である。青色反射領域を用いて土壌背景信号を補正し、エアロゾル散乱を含む大気の影響を低減する。

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI値は植生ピクセルにおいて0から1の範囲であるべきです。雲や白い建物などの明るい特徴、および水などの暗い特徴は、EVI画像において異常なピクセル値を引き起こす可能性があります。 EVI画像を作成する前に、反射率画像から雲や明るい特徴をマスク処理し、必要に応じてピクセル値を0から1の範囲に閾値処理すべきである。

_参考文献: Huete, A. 他「MODIS植生指標の放射測定学的・生物物理学的性能概観」 Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - 森林被覆指数1

この指数は、レッドエッジバンドを含むマルチスペクトル反射率画像を用いて、森林樹冠を他の植生タイプから識別します。

$$
FCI1 = Red * RedEdge
$$

森林地域は、樹木の反射率が低く、樹冠内に影が存在するため、FCI1 値が低くなります。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. 「マルチスペクトル画像のための堅牢な森林被覆指数」 Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - 森林被覆指数 2

この指数は、レッドエッジバンドを含まないマルチスペクトル反射率画像を使用して、森林キャノピーを他の種類の植生と区別します。

$$
FCI2 = Red * NIR
$$

森林地域は、樹木の反射率が低く、キャノピー内に影が存在するため、FCI2 値が低くなります。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. 「マルチスペクトル画像のための堅牢な森林被覆指数」 Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - 地球環境モニタリング指数

この非線形植生指数は、衛星画像による地球環境モニタリングに使用され、大気の影響を補正しようと試みます。NDVIと似ていますが、大気の影響を受けにくいという特徴があります。裸地の影響を受けるため、植生がまばらまたは中程度の密度の地域での使用は推奨されません。

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

出典:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_参考文献: Pinty, B., and M. Verstraete. GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites. Vegetation 101 (1992): 15-20._

***

## GARI - Green 大気抵抗指数

この指数は、NDVIよりも幅広いクロロフィル濃度に対して感度が高く、大気の影響に対する感度が低い。

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

ガンマ定数は大気中のエアロゾル状態に依存する重み付け関数である。ENVIは1.7の値を使用しており、これはGitelson, Kaufman, Merzylak (1996, p.296)が推奨する値である。

_参考文献: Gitelson, A., Y. Kaufman, and M. Merzylak. &quot;Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green クロロフィル指数

この指数は、幅広い植物種における葉のクロロフィル含有量を推定するために用いられる。

$$
GCI = {NIR \over Green} - 1
$$

広いNIRおよび緑色波長域をカバーすることで、クロロフィル含有量の予測精度が向上し、感度とS/N比も高まる。

_参考文献: Gitelson, A., Y. Gritz, and M. Merzlyak. &quot;Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green 葉指数

この指数は、小麦被覆率を測定するためのデジタルRGBカメラ用に設計されたもので、赤・緑・青のデジタル数値（DN）は0から255の範囲で表される。

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI値の範囲は-1から+1です。負の値は土壌や非生物的特徴を表し、正の値は緑の葉や茎を表します。

_参考文献: Louhaichi, M., M. Borman, and D. Johnson. &quot;Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.&quot; Geocarto International 16, No. 1 (2001): 65-70._

***

## GNDVI - Green 正規化植生指数

この指数はNDVIと類似しているが、赤色スペクトルではなく540～570 nmの緑色スペクトルを測定する点が異なる。この指数はNDVIよりもクロロフィル濃度に対して感度が高い。

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_参考文献: Gitelson, A., and M. Merzlyak. &quot;Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green 最適化土壌補正植生指数

この指標は、トウモロコシの窒素要求量を予測するためにカラー赤外線写真を用いて設計された。OSAVIと類似しているが、緑バンドを赤バンドに置換している。

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_参考文献: Sripada, R., et al. 「航空カラー赤外線写真を用いたトウモロコシの生育期窒素要求量の決定」 博士論文、ノースカロライナ州立大学、2005年。_

***

## GRVI - Green 比類植生指数

本指数は森林樹冠の光合成速度に敏感である。緑色と赤色の反射率は葉の色素変化に強く影響されるためである。

$$
GRVI = {NIR \over Green }
$$

_参考文献: Sripada, R. 他「トウモロコシの生育初期における窒素要求量を決定するための航空カラー赤外線写真」『Agronomy Journal』98 (2006): 968-977._

***

## GSAVI - Green 土壌補正植生指数

この指数は、トウモロコシの窒素要求量を予測するためにカラー赤外線写真を用いて設計された。SAVIと類似しているが、緑バンドを赤バンドに置換している。

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_参考文献: Sripada, R. 他「航空カラー赤外線写真を用いたトウモロコシの生育期窒素要求量の決定」博士論文、ノースカロライナ州立大学、2005年._

***

## LAI - 葉面積指数

本指数は葉被覆率の推定、作物の生育および収量の予測に使用される。 ENVIはBoegh et al (2002)の以下の経験式を用いて緑色LAIを計算します：

$$
LAI = 3.618 * EVI - 0.118
$$

ここでEVIは：

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

高い LAI 値は通常、約 0 から 3.5 の範囲です。ただし、雲やその他の明るい特徴が含まれ飽和ピクセルを生じる場合、LAI 値は 3.5 を超えることがあります。LAI 画像を作成する前に、雲や明るい特徴をシーンからマスク処理することが理想的です。

_参考文献: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, and A. Thomsen. &quot;Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.&quot; Remote Sensing of Environment 81, no. 2-3 (2002): 179-193._

***

## LCI - 葉緑素指数

本指数は、葉緑素吸収による反射率の変化に敏感な高等植物の葉緑素含有量を推定するために用いられます。

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_参考文献: Datt, B. 「ユーカリ葉の水分含有量のリモートセンシング」『Journal of Plant Physiology』154巻1号 (1999): 30-36頁._

***

## MNLI - 修正非線形指数

本指標は、土壌背景を考慮するため土壌補正植生指数（SAVI）を組み込んだ非線形指数（NLI）の改良版である。ENVIでは樹冠背景補正係数（_L_）値として0.5を使用する。

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_参考文献: Yang, Z., P. Willis, and R. Mueller. &quot;Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.&quot; Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 - 修正土壌補正植生指数 2

本指数は、Qiら（1994）が提案したMSAVI指数（土壌補正植生指数SAVIを改良）の簡略版である。土壌ノイズを低減し、植生信号のダイナミックレンジを拡大する。 MSAVI2は誘導的手法に基づき、健全な植生を強調するために定数_L_値（SAVIのように）を使用しない。

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_参考文献: Qi, J., A. Chehbouni, A. Huete, Y. Kerr, and S. Sorooshian. 「改良型土壌補正植生指数」『リモートセンシング・オブ・エンバイロメント』48巻（1994年）：119-126頁。

***

## NDRE- 正規化差分RedEdge

この指標はNDVIと類似しているが、RedではなくNIRとRedEdgeのコントラストを比較するため、植生のストレスをより早期に検出できることが多い。

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - 正規化差分植生指数

この指数は健全な緑色植生の指標です。正規化差分方式とクロロフィル吸収・反射率の最高域を組み合わせることで、幅広い条件下で頑健性を発揮します。ただし、LAIが高値となる密植状態では飽和する可能性があります。

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

本指数の値は-1から1の範囲で変動する。緑色植生の一般的な範囲は0.2から0.8である。

_参考文献: Rouse, J., R. Haas, J. Schell, and D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. 第三回ERTSシンポジウム、NASA (1973): 309-317.

***

## NLI - 非線形指数

この指数は、多くの植生指数と地表の生物物理学的パラメータとの関係が非線形であることを前提としています。非線形になりがちな地表パラメータとの関係を線形化します。

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_参考文献: Goel, N., and W. Qin. 「樹冠構造が各種植生指数とLAIおよびFparとの関係に及ぼす影響：コンピュータシミュレーション」『リモートセンシングレビュー』10号（1994年）：309-347頁。_

***

## OSAVI - 最適化土壌補正植生指数

本指数は土壌補正植生指数（SAVI）を基に構築される。樹冠背景補正係数には標準値0.16を用いる。 Rondeaux (1996) は、この値が低植生被覆域において SAVI よりも大きな土壌変動を提供すると同時に、50%を超える植生被覆に対して感度が増すことを実証した。本指数は、植生が比較的疎で樹冠を通して土壌が視認可能な地域での使用に最適である。

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_参考文献: Rondeaux, G., M. Steven, and F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - 再正規化差分植生指数

この指標は、近赤外波長と赤色波長の差分とNDVIを用いて健全な植生を強調する。土壌の影響や太陽観測幾何学の影響を受けにくい。

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_参考文献: Roujean, J., and F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - 土壌補正植生指数

この指数はNDVIと類似しているが、土壌ピクセルの影響を抑制する。樹冠背景補正係数_L_を用いる。これは植生密度に依存する関数であり、多くの場合、植生量の事前知識を必要とする。 Huete (1988) は、一次的な土壌背景変動を考慮するため、最適値として_L_=0.5を提案している。この指数は、植生が比較的疎で、樹冠を通して土壌が見える地域での使用が最適である。

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_参考文献: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - 変換差分植生指数

この指数は都市環境における植生被覆のモニタリングに有用である。NDVIやSAVIのように飽和しない。

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_参考文献: Bannari, A., H. Asalhi, and P. Teillet. &quot;Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping&quot; In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - 可視大気抵抗指数

この指数はARVIを基にしており、大気の影響を受けにくいシーンにおける植生の割合を推定するために用いられる。

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_参考文献: Gitelson, A. 他. 「可視スペクトル空間における植生と土壌の境界線：植生割合の遠隔推定に関する概念と技術」 International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - 広ダイナミックレンジ植生指数

この指数はNDVIと類似しているが、 近赤外信号と赤信号の寄与度の差を軽減するため重み係数（_a_）を用いる点が異なる。WDRVIは、NDVIが0.6を超える中～高植生密度シーンにおいて特に有効である。 NDVIは植生率と葉面積指数（LAI）が増加すると平坦化する傾向がある一方、WDRVIはより広い範囲の植生率とLAIの変化に対して感度が高い。

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

重み係数（_a_）は0.1から0.2の範囲で設定可能である。0.2の値はHenebry, Viña, and Gitelson (2004)によって推奨されている。

_参考文献_

_Gitelson, A. &quot;Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.&quot; Journal of Plant Physiology 161, No. 2 (2004): 165-173._

_Henebry, G., A. Viña, and A. Gitelson. 「ワイドダイナミックレンジ植生指数とそのギャップ分析への応用可能性」『ギャップ分析速報』12: 50-56._
