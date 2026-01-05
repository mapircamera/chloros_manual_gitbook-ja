# GUI : ナビゲーション

Chloros および Chloros (ブラウザ) を初回起動すると、バックエンドが起動します。準備が整うと、左上のメインメニューアイコンが表示されます <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

左上から順に、上部ヘッダーには以下の項目が含まれます：

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> メインメニュー

メインメニューから新規プロジェクトの作成、既存プロジェクトの開く、プロジェクトフォルダの開くが可能です。

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> 再生/開始ボタン

有効化されている場合、この処理開始ボタンで画像処理パイプラインが開始されます。

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> 進行状況バー <img src=".gitbook/assets/image (5).png" alt="" data-size="line">無料版Chlorosモード（全ファイルを順次処理）では、進行状況バーに2段階（ターゲット検出と処理）が表示されます。

有料版Chloros+ライセンスモード（全ファイルを同時処理）では、進行状況バーに4段階（検出中、分析中、キャリブレーション中、エクスポート中）が表示されます。 マウスカーソルをChloros+の進捗バー上に置くと、拡張された4段階の進捗バーパネルがドロップダウン表示され、処理状況を追跡できます。上部進捗バーをクリックするとドロップダウンパネルが固定され、再度クリックすると解除されます。

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## サイドメニュー

左サイドバーメニューには、操作可能な各種アイコンが含まれています：

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [プロジェクト設定](project-settings/project-settings.md)

プロジェクト設定タブでは、プロジェクト全体の設定および処理設定を調整できます。ファイル処理を開始する前にこれらの設定を行ってください。

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ファイルブラウザ

プロジェクトへのファイル/フォルダの追加や削除を行います。重複ファイルは無視されます。ターゲット画像の列ボックスをチェックすると、処理対象はチェックされた画像のみに限定され、処理時間が大幅に短縮されます。画像/メタデータトグルを使用すると、選択した画像のサムネイルグリッド表示と詳細なメタデータテーブル表示を切り替えられます。

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [画像ビューア](image-viewer-gui/opening-an-image-full-screen.md)

メイン画像ビューアで画像をクリックすると、画像ビューアタブで全画面表示されます。

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [マップ](image-viewer-gui/map-markers.md)

GPS座標に基づき、インタラクティブな2Dマップ上で画像を閲覧できます。Google MapsとESRIタイルプロバイダーに対応し、位置情報に応じて最適なサービスを自動選択します。マーカーにカーソルを合わせると画像サムネイルのプレビューが表示されます。

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> デバッグログ

問題発生時にデバッグ出力のログを確認できます。ログをコピー/ダウンロードし、[MAPIRサポート](https://www.mapir.camera/community/contact)へ送信して支援を受けてください。

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [ユーザーログイン](chloros+-login.md)

ユーザーログインサイドバーからChloros+アカウントにログインし、高度な機能を有効化できます。現在のアプリケーションバージョンを確認したり、Chloros GUIおよびCLIで表示されるテキストの言語を変更したりすることも可能です。
