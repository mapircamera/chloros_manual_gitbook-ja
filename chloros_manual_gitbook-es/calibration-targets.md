---
description: 後処理でキャプチャされたデータを校正するために使用されるラボ測定パネル
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# キャリブレーションターゲット

MAPIR は、さまざまなアプリケーションをカバーするさまざまなキャリブレーション ターゲットを提供します。以下に示すコンパクトな T4-R50 には、250 ～ 2,500 nm の光反射率が測定された 4 つのパネルが含まれています。

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4 拡散参照ターゲットには、次の反射率曲線 [データダウンロードはこちらから](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157) があります。

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 400-1000nm</p></figcaption></figure>

反射率グラフを見ると、値が波長 (x 軸) 対反射率パーセント (y 軸) であることがわかります。キャリブレーション ターゲットの画像をキャプチャすると、カメラの各センサー バンドが感知できるスペクトル内で、ピクセル値と反射率パーセントの間の関係が作成されます。

これは、カメラでキャプチャしたすべての画像で、[T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) や [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) などの反射率ターゲットの写真を使用して、画像の反射率を調整できることを意味します。キャリブレーションが完了すると、画像内の各ピクセルは反射率のパーセントに等しくなります。

Chloros でキャリブレーションされた画像を一般的な JPG または TIFF として出力する場合、反射率パーセントは、ピクセル値を画像形式のビット深度で割ることによって計算されます。したがって、JPG の場合は 255 で除算し、TIFF の場合は 65,535 で除算します。 Chloros で PERCENT 形式の出力を選択することもできます。その場合、各ピクセルのパーセント値は 0.0 ～ 1.0 (反射率 0% ～ 100%) の範囲になります。一部の画像アプリケーションではパーセント (浮動小数点) 画像を受け入れることができず、ストレージのサイズが大きいことに注意してください。

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
