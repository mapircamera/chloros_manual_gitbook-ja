# GUI : ナビゲーション

Chloros および Chloros (ブラウザ) を初めて起動すると、バックエンドが起動します。準備が整うと、左上のメインメニューアイコンが表示されます <img src=".gitbook/assets/image (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

左から右へ、上部ヘッダーには以下の項目が含まれます：

### <img src=".gitbook/assets/image (1) (1).png" alt="" data-size="line"> メインメニュー

メインメニューから新規プロジェクトの作成、既存プロジェクトの開く、プロジェクトフォルダの開くが可能です。

### <img src=".gitbook/assets/image (2).png" alt="" data-size="line"> 再生/開始ボタン

有効化されている場合、この処理開始ボタンで画像処理パイプラインが開始されます。

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> 進行状況バー <img src=".gitbook/assets/image (5).png" alt="" data-size="line">無料版Chlorosモード（全ファイルを順次処理）では、進行状況バーは2段階（ターゲット検出と処理）を表示します。

有料版Chloros+ライセンスモード（全ファイルを同時処理）では、進行状況バーは4段階（検出中、分析中、キャリブレーション中、エクスポート中）を表示します。 マウスカーソルをChloros+の進捗バー上に置くと、拡張された4段階の進捗バーパネルがドロップダウン表示され、処理状況を追跡できます。上部の進捗バーをクリックするとドロップダウンパネルが固定され、再度クリックすると解除されます。

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## サイドメニュー

左サイドバーメニューには、操作可能な各種アイコンが含まれます：

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [プロジェクト設定](project-settings/project-settings.md)

プロジェクト設定タブでは、プロジェクト全体の設定および処理設定を調整できます。ファイル処理を開始する前にこれらの設定を行ってください。

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ファイルブラウザ

プロジェクトへのファイル/フォルダの追加や削除を行います。重複ファイルは無視されます。ターゲット画像の列ボックスをチェックすると、処理はチェックされた画像のみを対象とするため、処理時間が大幅に短縮されます。

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [画像ビューア](image-viewer-gui/opening-an-image-full-screen.md)

メイン画像ビューアで画像をクリックすると、画像ビューアタブで全画面表示されます。

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> デバッグログ

問題発生時にデバッグ出力のログを確認してください。ログをコピー/ダウンロードし、[MAPIRサポート](https://www.mapir.camera/community/contact)に送信して支援を受けてください。

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [ユーザーログイン](chloros+-login.md)

ユーザーログインサイドバーでは、Chloros+アカウントにログインして高度な機能を有効化できます。また、現在のアプリケーションバージョンを確認したり、Chloros GUIおよびCLIで表示されるテキストの言語を変更したりできます。
