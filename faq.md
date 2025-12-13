---
description: よくあるご質問
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---
# よくある質問

<details>

<summary>Chlorosで、MAPIRブランド以外のカメラの画像を処理できますか？</summary>

いいえ、ChlorosはMAPIRカメラの画像処理のみをサポートしています。 詳細は[対応カメラモデル一覧](supported-cameras.md)をご確認ください。MAPIR Cloudでは他カメラの処理も提供しており、全リストは[こちら](https://mapir.gitbook.io/mapir-cloud/supported-cameras)をご覧ください。

</details>

<details>

<summary>キャリブレーションターゲットなしで反射率のキャリブレーションは可能ですか？</summary>

いいえ。ターゲット以外の画像を撮影する際に、キャリブレーションターゲットの画像を同時に撮影していない場合、画像のピクセル値を既知の反射率パーセントに関連付けることができません。また、MAPIR光センサーのログを含めない場合、周囲光のスペクトルが測定されず、反射率の結果は正確ではありません。

</details>

<details>

<summary>Chlorosでの処理前に画像を編集できますか？</summary>

いいえ。Chlorosは入力データが変更されていないことを前提としています。ファイル名を変更しないでください。

</details>

<details>

<summary>MAPIR Survey3カメラを自動露出に設定し、Chlorosで画像を処理できますか？</summary>

いいえ。Survey3 イメージデータセットは、露出が固定/ロックされている必要があります。したがって、自動シャッタースピードや自動 ISO は使用できません。同じカメラモデルの画像はすべて、シャッタースピードと ISO (露出) が同一である必要があります。

</details>

<details>

<summary>Chlorosはオルソモザイク画像の処理や解析が可能ですか？</summary>

いいえ。MAPIRカメラの個別画像のみをサポートしており、オルソモザイクマップのような結合済み画像はサポートしていません。

</details>

<details>

<summary>Chlorosのターゲット検出ステップを高速化する方法は？</summary>

ファイルブラウザテーブルの右列でターゲット画像を事前に選択すると、Chloros はそれらの画像のみからキャリブレーションターゲットを検索するようになり、処理が大幅に高速化されます。

</details>

<details>

<summary><a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR </a>Cloudに画像をアップロードする場合、事前にChlorosで処理すべきですか？</summary>

当社のオンライン処理プラットフォーム[MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)へのアップロードを予定している場合、アップロード前に画像を編集しないでください。Cloudが同じ処理をすべて行い、さらに追加処理も実施します。

</details>

<details>

<summary>MAPIRはX機能をサポートする予定はありますか？MAPIRにX機能が搭載されればと強く願っています。</summary>

当社製品に関するフィードバックは常に歓迎しております。製品の問題点や改善提案がございましたら、[お問い合わせ](https://www.mapir.camera/community/contact)よりご意見をお寄せください。当社の研究開発の大半は、お客様の最重要ニーズを反映して進められています。

</details>
