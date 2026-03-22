# GUI : ナビゲーション

Chloros および Chloros (Browser) を初めて起動すると、バックエンドが起動します。準備が整うと、左上のメインメニューアイコンが表示されます <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

上部ヘッダーには、左から右の順に以下の項目が含まれています：

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> メインメニュー

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

メインメニューからは以下の操作が可能です：

* **新規プロジェクト** — 新しいプロジェクトを作成します
* **プロジェクトを開く** — 既存のプロジェクトを開きます
* **プロジェクトフォルダを開く** — ファイルエクスプローラーでプロジェクトフォルダを開きます
* **ファイルを追加** — 現在のプロジェクトに個別の画像ファイルを追加します _(プロジェクトを開いた後に表示されます)_
* **フォルダの追加** — 画像のフォルダを現在のプロジェクトに追加します _(プロジェクトを開いた後に表示されます)_
* **処理の開始 / 処理の停止** — 画像処理パイプラインを開始または停止します _(ファイルが追加された後に有効になります)_

{% hint style="info" %}
**Windowsのみ**: ChlorosデスクトップGUIはWindowsで利用可能です。Linuxユーザーは[CLI](CLI.md) および [Python SDK](api-python-sdk.md) のドキュメントを参照してください。
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> 再生/開始ボタン

有効にすると、処理開始ボタンで画像処理パイプラインが開始されます。

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> 進行状況バー <img src=".gitbook/assets/image (5).png" alt="" data-size="line">すべてのファイルを順次処理する無料の Chloros モードでは、進行状況バーに「ターゲット検出」と「処理中」の 2 つの段階が表示されます。

すべてのファイルを同時に処理する有料の Chloros+ ライセンスモードでは、進行状況バーに「検出中」、「分析中」、「キャリブレーション中」、「エクスポート中」の 4 つの段階が表示されます。 Chloros+の進行状況バーにマウスカーソルを合わせると、詳細な4段階の進行状況パネルがドロップダウン表示され、処理の進捗を確認できます。上部の進行状況バーをクリックするとドロップダウンパネルが固定され、再度クリックすると固定が解除されます。

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## サイドメニュー

左側のサイドバーメニューには、操作を行うためのさまざまなアイコンが含まれています：

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [プロジェクト設定](project-settings/project-settings.md)

「プロジェクト設定」タブでは、プロジェクト全体の設定や処理設定を調整できます。ファイルの処理を開始する前に、これらの設定を調整してください。

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ファイルブラウザ

プロジェクトへのファイルやフォルダの追加、および削除を行います。重複ファイルは無視されます。ターゲット画像の列にあるチェックボックスをオンにすると、処理はチェックされた画像のみを対象とするため、処理時間が大幅に短縮されます。「画像/メタデータ」トグルを使用すると、選択した画像のサムネイルグリッド表示と詳細なメタデータ表の表示を切り替えることができます。

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [画像ビューア](image-viewer-gui/opening-an-image-full-screen.md)

メインの画像ビューアで画像をクリックすると、[画像ビューア]タブで全画面表示されます。

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [マップ](image-viewer-gui/map-markers.md)

GPS座標に基づいて、インタラクティブな2Dマップ上で画像を表示します。Google MapsおよびESRIタイルプロバイダーに対応しており、現在地に応じて最適なサービスが自動的に選択されます。マーカーにカーソルを合わせると、画像のサムネイルプレビューが表示されます。

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> デバッグログ

問題が発生した際は、ログを確認してデバッグ出力内容を確認してください。ログをコピーまたはダウンロードし、[MAPIR サポート](https://www.mapir.camera/community/contact)まで送信してサポートを受けてください。

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [ユーザーログイン](chloros+-login.md)

ユーザーログインサイドバーから、Chloros+アカウントにログインして、高度な機能を利用できるようになります。また、現在のアプリケーションバージョンを確認したり、Chloros GUIおよびCLIで表示されるテキストの言語を変更したりすることもできます。
