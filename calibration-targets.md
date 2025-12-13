---
description: Lab-measured panels used to calibrate captured data in post-processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# 校正ターゲット

MAPIRは、様々な用途に対応する多様な校正ターゲットを提供しています。下記のコンパクトなT4-R50には、250～2,500 nmの光反射率を測定済みの4枚のパネルが含まれています。

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4拡散基準ターゲットの反射率曲線は以下の通りです。[データダウンロードはこちら](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 400-1000nm</p></figcaption></figure>反射率グラフを見ると、値が波長（x軸）と反射率パーセント（y軸）の関係で表示されていることがわかります。キャリブレーションターゲットの画像を撮影すると、カメラの各センサーバンドが感度を持つスペクトル範囲内で、ピクセル値と反射率パーセントの関係が作成されます。

つまり、当社のカメラで撮影するすべての画像について、[T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) や [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) などの当社の反射率ターゲットの写真を、反射率のキャリブレーションに使用することができます。 キャリブレーション後、画像の各ピクセルは反射率パーセントに等しくなります。

校正済み画像をChlorosで一般的なJPG形式、またはTIFFで出力する場合、反射率パーセントはピクセル値を画像フォーマットのビット深度で割ることで算出されます。つまりJPGでは255で、TIFFでは65,535で割ります。 Chlorosでパーセント形式出力を選択することも可能です。この場合、各ピクセルは0.0～1.0のパーセント値（0%～100%の反射率）の範囲になります。ただし、一部の画像アプリケーションはパーセント（浮動小数点）形式の画像に対応しておらず、またストレージ容量が非常に大きくなる点に留意してください。

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
