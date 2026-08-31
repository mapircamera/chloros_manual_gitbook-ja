---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# 校正ターゲット

MAPIRは、幅広い用途に対応するさまざまな校正ターゲットを提供しています。下図に示すコンパクトなT4-R50には、250～2,500 nmの波長範囲における光反射率が測定済みの4枚のパネルが含まれています。

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4拡散型基準ターゲットの反射率曲線は以下の通りです。[データはこちらからダウンロード可能](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157)：

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 400～1,000 nm</p></figcaption></figure>T4P拡散リファレンスターゲットの反射率曲線は以下の通りです。[データはこちらからダウンロード](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157)：

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P 反射率 :: 250～2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P 反射率 :: 400～1000nm</p></figcaption></figure>反射率グラフを見ると、値は波長（x軸）対反射率（％）（y軸）で表されていることがわかります。キャリブレーションターゲットの画像を撮影すると、カメラの各センサーバンドが感度を持つスペクトル範囲内で、ピクセル値と反射率（％）の相関関係が確立されます。

つまり、当社のカメラで撮影するすべての画像について、[T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50)や[T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125)などの当社の反射率ターゲットの写真を活用して、画像の反射率をキャリブレーションできます。キャリブレーションが完了すると、画像内の各ピクセルは反射率（％）に等しくなります。

**Survey3** の出力については、Chloros形式でキャリブレーション済みの画像を一般的なJPG形式やTIFF形式で出力する場合、反射率（％）は、ピクセル値を画像形式のビット深度で割ることで算出されます。 したがって、JPGの場合は255で割り、TIFFの場合は65,535で割ります。 また、ChlorosでPERCENT形式の出力を選択することも可能で、その場合、各ピクセルの値は0.0～1.0のパーセント値（0%～100%の反射率）の範囲になります。 ただし、一部の画像アプリケーションではパーセント（浮動小数点）形式の画像に対応していない場合があること、また保存容量が大きくなることに留意してください。

{% hint style="info" %}
**LATTICEの反射率は、異なるピクセルスケールを使用しています。** LATTICEの反射率は、DN 32768 = 100%の反射率（65535ではありません）として保存されており、すべてのファイルにはそのスケールを示すXMPタグ（`Chloros:PixelScale`）が含まれています。 定数であると仮定するのではなく、タグを読み取り、その値で割ってください — [出力画像フォーマット](output-image-formats.md)を参照してください。
{% endhint %}

## LATTICEカメラでのキャリブレーションターゲット

LATTICEカメラでは、反射率のキャリブレーションターゲットは**任意**です。代わりに、Chlorosでは、DAQ光センサーによって測定された下向き放射照度（ρ = π·L/E）を基準として反射率を算出することも可能です。この基準は、反射率ソース設定で選択します （GUIの「プロジェクト設定」；`--reflectance-source`はCLI内；`reflectance_source`はSDK内）で選択します：

| 値 | 動作 |
| --- | --- |
| `auto` *(デフォルト)* | QAに合格したフレーム内のターゲットが**絶対基準**となる。ターゲットが存在しない場合、またはQAに失敗した場合は、ChlorosはDAQのダウンウェル除算値にフォールバックする。 |
| `target` | ターゲットのみに限定 — DAQによる置換なし。 |
| `daq` | DAQを優先 — ダウンウェル測定値が常に基準となる。 |

LATTICE 向けの追加ターゲット動作：

* **ターゲットの形状** — ArUco マーキング付きパネル、固定 ROI パネル、およびストリップターゲットがすべてサポートされています。形状はプロジェクトのターゲット設定から取得されます。
* **ターゲット単位ごとの測定データ** — `--target-reflectance-dir DIR`は、ターゲット単位ごとの反射率スキャンデータが格納されたディレクトリを指します（`<serial>.csv`、ターゲット単位のシリアル番号/QRコードで検索されます）。 検出に失敗した場合、Chlorosは公称のT3/T4Pスペクトルにフォールバックします。
* **時間的アンカー** — 検出されたターゲットは、その周囲のフレームを校正し、ターゲットの再検出までの間、その位置が保持されます。

フラグの完全な意味論と例については、[CLI リファレンス](reference/cli-reference.md)（「製品ごとのエクスポート切り替え」を参照）に記載されています。

### F988

「F988の反射率は、シーン内の反射率パネルを使用して校正されます。このバンドはDAQ光センサーの校正範囲を超えているため、Chlorosは最新のパネルキャプチャを適用し、パネルの観測間ではその値を保持します。」

F988をDAQのみのキャリブレーションで実行した場合、ChlorosはそのバンドのDAQベースの反射率を拒否し、その理由を通知します（スキップ理由 `dls-uncalibrated-band-988`）。パネルを使用したワークフローが推奨される方法です。

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
