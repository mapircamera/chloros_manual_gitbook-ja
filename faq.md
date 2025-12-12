---
description: よくある質問
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# よくある質問

<details>

<summary>MAPIR ブランドではないカメラからの画像を Chloros で処理できますか?</summary>

いいえ、Chloros は MAPIR カメラ画像の処理のみをサポートしています。詳細については、[サポートされているカメラモデル](supported-cameras.md) のリストをご覧ください。 MAPIR Cloud では他のカメラの処理も提供しています。完全なリストは [ここ](https://mapir.gitbook.io/mapir-cloud/supported-cameras) をご覧ください。

</details>

<details>

<summary>キャリブレーション ターゲットを使用せずに画像の反射率をキャリブレーションできますか?</summary>

いいえ。ターゲット以外の画像がキャプチャされる頃にキャプチャされたキャリブレーション ターゲットの画像がなければ、画像のピクセル値を既知の反射率パーセントに関連付けることはできません。 MAPIR 光センサーからのログも含めないと、周囲光スペクトルは測定されず、反射率の結果は正確ではなくなります。

</details>

<details>

<summary>Chloros で処理する前に画像を編集できますか?</summary>

いいえ。Chloros は入力データが変更されていないことを前提としています。ファイル名は変更しないでください。

</details>

<details>

<summary>MAPIR Survey3 カメラを自動露出に設定し、Chloros で画像を処理できますか?</summary>

いいえ。Survey3 の画像データセットは露出が固定/ロックされている必要があるため、自動シャッター スピードや自動 ISO は使用できません。同じカメラ モデルのすべての画像は、同じシャッター スピードと ISO (露出) を持つ必要があります。

</details>

<details>

<summary>Chloros はオルソモザイク画像を処理または分析できますか?</summary>

いいえ。個別の MAPIR カメラ画像のみがサポートされており、オルソモザイク マップのような結合画像はサポートされていません。

</details>

<details>

<summary>クロロスのターゲット検出ステップをスピードアップするにはどうすればよいですか?</summary>

ファイル ブラウザのテーブルで、右側の列でターゲット画像を事前に選択すると、Chloros はそれらの画像でのみキャリブレーション ターゲットを調べるように指示され、処理が大幅に高速化されます。

</details>

<details>

<summary>画像を <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> にアップロードする場合、アップロードする前に Chloros で処理する必要がありますか?</summary>

オンライン処理プラットフォーム [MAPIRクラウド](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) にアップロードする予定がある場合は、アップロード前に画像を編集しないでください。クラウドはすべて同じ処理などを実行します。

</details>

<details>

<summary>MAPIR は X 機能をサポートする予定ですか? MAPIR が X を提供してくれることを本当に願っています。</summary>

当社では、製品に関するフィードバックを常にお待ちしております。当社の製品に問題を見つけた場合、または製品を改善する方法についてご提案がある場合は、[お問い合わせ](https://www.mapir.camera/community/contact) してご意見を共有してください。当社の研究開発のほとんどは、お客様の最大のニーズに耳を傾けることによって導かれます。

</details>
