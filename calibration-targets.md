---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# キャリブレーションターゲット

MAPIRは、幅広い用途に対応するさまざまなキャリブレーションターゲットを提供しています。下図のコンパクトなT4-R50には、250～2,500 nmの波長範囲における光反射率が測定済みの4枚のパネルが含まれています。

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4拡散反射基準ターゲットの反射率曲線は以下の通りです。[データはこちらからダウンロード](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 400-1000nm</p></figcaption></figure>T4P拡散反射基準ターゲットの反射率曲線は以下の通りです。[データはこちらからダウンロード](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P 反射率 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P 反射率 :: 400-1000nm</p></figcaption></figure>反射率グラフを見ると、値は波長（x軸）対反射率（％）（y軸）で表されていることがわかります。キャリブレーションターゲットの画像を撮影すると、カメラの各センサーバンドが感度を持つスペクトル範囲内で、ピクセル値と反射率（％）の相関関係が作成されます。

つまり、当社のカメラで撮影したすべての画像について、[T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50)や[T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125)などの反射率ターゲットの写真を使用して、画像の反射率をキャリブレーションすることができます。 キャリブレーションが完了すると、画像内の各ピクセルは反射率（％）に相当するようになります。

Chlorosでキャリブレーション済みの画像を一般的なJPG形式やTIFF形式で出力する場合、反射率（％）は、ピクセル値を画像形式のビット深度で割ることで算出されます。つまり、JPGの場合は255で割り、TIFFの場合は65,535で割ります。 また、ChlorosでPERCENT形式の出力を選択することも可能です。この場合、各ピクセルの値は0.0から1.0の範囲（反射率0%から100%）になります。ただし、一部の画像アプリケーションではパーセント（浮動小数点）形式の画像に対応していない場合があること、また保存容量が大きくなる点にご注意ください。

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
