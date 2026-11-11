# GUI：ナビゲーション

Chlorosを初めて起動すると、処理バックエンドが起動します。バックエンドの準備が整うと、左上のメインメニューアイコン <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> が表示され、左サイドバーの「カメラ」および「光センサー」タブのロックが解除されます（それまではこれらのタブはグレーアウトしています）。

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

上部ヘッダーには、左から順に以下の項目が含まれます：

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> メインメニュー

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>メインメニューからは、以下の操作が可能です：

* **新規プロジェクト**— 新しいプロジェクトを作成します。プロジェクトテンプレートを保存している場合は、**テンプレートの選択**ドロップダウンが表示され、テンプレートの設定に基づいて新しいプロジェクトを開始できます。
* **プロジェクトを開く**— 既存のプロジェクトを開きます。 このリストには、ファイルエクスプローラーでプロジェクトフォルダを開く**プロジェクトフォルダを開く** ボタンが含まれています。
* **プロジェクトの複製** — 現在開いているプロジェクトを新しい名前（「MyProject (2)」のような自由な名前が提案されます）でコピーし、そのコピーを開きます。 _(プロジェクトを開いた後に表示されます)_
* **ファイルの追加** — 個々の画像ファイルを現在のプロジェクトに追加します _(プロジェクトを開いた後に表示されます)_
* **フォルダの追加** — 1つ以上の画像フォルダを現在のプロジェクトに追加します _(プロジェクトを開いた後に表示されます)_
* **処理の開始 / 処理の停止** — 画像処理パイプラインを開始または停止します _(ファイルが追加された後に有効になります)_
* **カメラへの接続** — [「カメラ」タブ](lattice/) に移動し、LATTICE カメラまたはアレイを接続します。プロジェクトを開いていなくても動作します。
* **光センサーに接続** — [光センサータブ](daq/) に移動し、DAQ光センサーを接続します。プロジェクトを開いていなくても動作します。

{% hint style="info" %}
**Windowsのみ**：ChlorosデスクトップGUIはWindowsで利用可能です。Linuxユーザーは、ヘッドレス処理について[CLI](CLI.md)および[Python SDK](api-python-sdk.md)のドキュメントを参照してください。
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

再生/開始ボタン

有効にすると、処理開始ボタンで画像処理パイプラインが開始されます。

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

進行状況バー <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



すべてのファイルを順次処理する無料のChlorosモードでは、進行状況バーには「ターゲット検出」と「処理」の2つの段階が表示されます。

すべてのファイルを同時に処理する有料のChloros+ライセンスモードでは、進行状況バーには「検出」、「分析」、「キャリブレーション」、「エクスポート」の4つの段階が表示されます。 Chloros+の進行状況バーにマウスカーソルを合わせると、4つの進行状況バーからなる拡張パネルがドロップダウン表示され、処理の進捗を確認できます。上部の進行状況バーをクリックするとドロップダウンパネルが固定され、もう一度クリックすると固定が解除されます。

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## サイドメニュー

左側のサイドバーメニューには、操作可能なさまざまなアイコンが、上から順に次の通り配置されています：

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [プロジェクト設定](project-settings/project-settings.md)

「プロジェクト設定」タブでは、プロジェクト全体の設定および処理設定を調整できます。 ファイルの処理を開始する前に、これらの設定を調整してください。

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ファイルブラウザ

プロジェクトへのファイルやフォルダの追加、およびプロジェクトからの削除を行います。重複するファイルは無視されます。対象画像の列にあるチェックボックスにチェックを入れると、処理ではチェックされた画像のみを対象として処理されるため、処理時間が大幅に短縮されます。 「画像／メタデータ」トグルを使用すると、選択した画像のサムネイルグリッド表示と詳細なメタデータ表の表示を切り替えることができます。

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [画像ビューア](image-viewer-gui/opening-an-image-full-screen.md)

メインの画像ビューアで画像をクリックすると、「画像ビューア」タブで全画面表示されます。

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [マップビューア](image-viewer-gui/map-markers.md)

GPS座標に基づいて、インタラクティブな2Dマップ上で画像を表示します。 Google MapsおよびESRIのタイルプロバイダーに対応しており、現在地に応じて最適なサービスが自動的に選択されます。マーカーにカーソルを合わせると、画像のサムネイルプレビューが表示されます。

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [カメラ](lattice/)

LATTICEカメラを接続し、ライブで制御できます。1台ずつ、または同期されたマルチカメラアレイとして操作可能です。 このタブには、オーバーレイやヒストグラム付きのライブプレビュータイル、カメラごとおよびアレイごとの設定、さらに「すべてをキャプチャ」でどのカメラとエクスポート形式を選択するかを指定するキャプチャ設定が表示されます。バックエンドの準備が整い次第利用可能になります。詳細な手順については、[LATTICEセクション](lattice/)をご覧ください。

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [光センサー](daq/)

DAQ光センサー（DAQ-U（USB）、DAQ-M（Bluetooth）、DAQ-E（イーサネット））を接続し、W/m²/nm単位で校正済みのスペクトルチャートをリアルタイムで表示できます。 ここから、開いているプロジェクトに `.daq` ファイルを記録したり、センサーの名前を変更したり、キャップ補正プロファイルを選択したり、DAQ-E のファームウェアを更新したりできます。バックエンドの準備が整い次第利用可能になります。詳細な手順については、[DAQ セクション](daq/)を参照してください。

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> デバッグログ

問題が発生した際は、ログを確認してデバッグ出力がないか確認してください。ログをコピーまたはダウンロードし、[MAPIRサポート](https://www.mapir.camera/community/contact)に送信してサポートを受けてください。

1.3.0の新機能：ログタブの上部にあるバグアイコン（**バグを報告**）をクリックすると、説明文、最大10枚の画像、Chlorosログ、およびシステムの詳細を含むレポートがMAPIRに送信されます。レポートを送信するには、ログインしている必要があります。 詳細は [ログタブからのバグ報告](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab) をご覧ください。

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [ユーザーログイン](chloros+-login.md)

ユーザーログインサイドバーを使用すると、Chloros+ アカウントにログインして、高度な機能を利用できるようになります。また、現在のアプリケーションバージョンを確認したり、Chloros GUI および CLI で表示されるテキストの言語を設定したりすることもできます。
