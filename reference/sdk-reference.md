# Chloros Python SDK リファレンス

**バージョン:**

1.3.0**生成日:**2026-07-29 19:19 ·**改訂日:** 2026-11-06**パッケージ:** `chloros-sdk` (PyPI)**対象読者:** LLMによる利用に最適化されており、人間が読みやすい形式です。**範囲:** `import chloros_sdk`によって公開されているすべてのパブリッククラス、関数、ヘルパー。画像処理、単一カメラ制御、同期化された配列、 DAQセンサー、プロジェクトの自動化を網羅した、コピー＆ペースト可能なサンプルに加え、AIアシスタント向けの`chloros-mcp`サーバーも含まれます。

要点のみを確認したい場合は、以下へ進んでください：
- [インストールとクイックスタート](#installation)
- [LATTICEアレイ用Smart-Connect](#smart-connect-for-lattice-cameras)
- [DAQセンサーセッション](#daq-sensor-sessions)
- [カメラユーティリティ](#camera-utilities)
- [トランジェントアレイ、アライメントプロファイル、インデックス](#transient-arrays-alignment-profiles--indices)
- [プロジェクトフォルダとレシピ](#project-folder--recipes)
- [プロジェクトの自動化](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [独自のパイプラインでのライブフレーム](#live-frames-in-your-own-pipeline)
- [MCPサーバー（AIアシスタント）](#mcp-server-ai-assistants)

> **

1.3.0での変更点**（既存の1.2.0スクリプトが検知する内容）。
>
> **

1.2.0スクリプトで対応が必要となる可能性のある変更点：**
>
> - **バックエンドが確認できないリリースが、現在エラーとして発生するようになりました。** セッションが開いた配列に対して `CameraSession.disconnect()`、`ArraySession.disconnect()` / `force_disconnect()` が発生し、`DAQSensorSession.disconnect()` / `force_disconnect()`が、セッションが開いたセンサーで発生し、`ChlorosProject.disconnect_all()`は、デバイスがまだ保持されている可能性があるため、バックエンドが解放を拒否したり、時間内に確認しなかったりした場合に、`ChlorosConnectError`を発生させます。 バージョン 1.2.0 では、これらは静かに戻されるようになりました。`with` ブロックの終了は、依然として例外を発生させません。
> - **借用されたデバイスは決して破棄されません。** `with attach_array(...)` およびそれに対する明示的な `disconnect()` は、配列を稼働させたままにします（1.2.0 では、GUI が所有している場合でも、ブロックを終了するとデバイスは破棄されていました）; 終了させる場合は `force_disconnect()` を呼び出してください。同様に、センサーがすでにプール内にあることを検出した `connect_daq_sensor()`（ゼロフラグのスマート検出を含む）は、`disconnect()`実行中および`with`ブロックを離れる際も、接続は開いたままになります。
> - **PNGおよびJPEGは、`display`およびインデックス付き製品のみを受け付けます。** `capture()`と`format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) および `radiance` または `reflectance` レベルの場合、 または `processing="all"` の場合、いかなるリクエストを行う前にも `ValueError` を設定する必要があります。 12ビットの`raw` / `debayered`はバックエンドによって拒否されます。[PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only)を参照してください。
> - **`connect_daq_sensor(sensor_id=...)` はそのセンサーを開くか、** 検出した内容を列挙する `ChlorosConnectError` を発生させます。他のセンサーにフォールバックすることはありません。[固定トランスポート](#pinned-transport)を参照してください。
> - **`frame_stream()`は沈黙する代わりにエラーを発生させます：** `FrameFetchError`は拒否された場合、次のティックで （DAQを一切含まない反射率の読み取り）、60秒間フレームがない場合、および60秒間新しいトリガーがない場合。詳細は[1回の読み取りにつき1つのトリガー](#one-trigger-per-read--group_id)を参照してください。
> - **読み取りタイムアウトまたは接続切断が発生すると、すべての smart-connect 呼び出しから**`ChlorosConnectError`** および `analyze_array_network()` が発生します。バージョン 1.2.0 では、基盤となるタイムアウトまたは接続エラーが発生していました。
> - **`connect_array()`はデフォルトで600秒待機し（`timeout`、バージョン1.2.0では180秒）、接続に失敗した場合はその操作が取り消されます。** この呼び出しは `timeout` をバックエンドに送信します。バックエンドは、すべてのカメラを検出時の状態に戻し、接続が時間内に完了しなかった場合は何も接続しません。この呼び出しは `ChlorosConnectError` を発生させます （HTTP 504 `CLIENT_DEADLINE_PASSED`）を発生させます。バージョン 1.2.0 では、カメラの電源投入直後の接続が、この呼び出しの処理時間を超えて続き、例外が発生した後にアレイが接続されてしまう可能性がありました。`connect_all()` は、保存された各アレイに対して同様の方法で待機します （1.2.0では120秒）待機します。[`connect_array()` 署名](#connect_array-signature)を参照してください。
> - **`connect_array()`はセンサー全体を保持します。** `width` / `height` / `binning` がない場合、すべてのカメラはフルセンサーに設定され、フレームレートはホストとネットワークが提供するレートまで低下します。1.2.0は、フレームレートを維持するためにフレームを縮小またはビンニングしました。[同期アレイ](#synchronized-array--arraysession-smart-prep)を参照してください。
> - **M3M（モノクロ）カメラにおけるベイヤー画素フォーマットとは、その深度を指します。** M3Mはモノクロフォーマットのみを提供しており、プリセット名はベイヤー形式のものを指しています： バージョン1.2.0では、カメラはこのフォーマットを拒否して独自のフォーマットを維持していましたが、`settings.pixel_format`はベイヤー形式の名称を指定していました。`LatticeCamera(preset=...)`、`apply_preset()`、`set_format()`、`CameraSettings(pixel_format=...)`、`connect_camera(preset=...)`、および配列のの共有フォーマットは、M3M上で同じ深度のMonoフォーマットを設定するようになった。`"BayerRG8"`と`high_speed`は`Mono8`を生成し、`"BayerRG12"`、`high_quality`、および `triggered` は `Mono12` となり、`cam.settings.pixel_format` がその名前を指定します。 その逆も同様です。M3CはMono形式に保存されることは決してないため、`"Mono12"`――これらいずれか、 `connect_array(pixel_format=...)`、`set_settings(pixel_format=...)`、あるいは配列の保存形式のいずれからであっても――M3C上では、その配列のマスターとなっているカメラがどれであっても、`BayerRG12`となります。 渡された `CameraSettings` はその名前を保持するため、同じオブジェクトは引き続き各カメラに対して、自身のファミリー内のその深度を要求します。`connect_camera(preset=...)` はプリセットのフォーマットを保持します （バージョン1.2.0では、プリセットの適用後にカメラが12ビットに設定される）、また、プリセットで開かれたカメラは、取得したデータを`PRESETS`に書き込まなくなりました。カメラが拒否するその他の形式については （そのカメラの `PixelFormat` リストにないもの、例えば M3M 上の `RGB8` や、対応していないファームウェア上の `BayerRG16` など）も、独自の名前は付けられません。`LatticeCamera`接続後、`apply_settings()`または`set_format()`、 `cam.settings.pixel_format`および`packed_transport`は、カメラがストリーミングするフォーマットを示し、1行の印刷出力で拒否されたフォーマットが示されます。
> - **`analyze_array_network()` は、connect が適用される内容を示します：** `pixel_format=None`（旧 `"BayerRG8"`）は、connect が設定するフォーマット （Mono12 / BayerRG12）を意味し、`packed="auto"`はパケット転送の決定を示すため、そこに記載されているfpsは1.2.0'とは異なる場合があります。`pixel_format`を指定すると、その深度でM3Mが生成されます。[ネットワーク分析ヘルパー](#network-analysis-helper)を参照してください。
> - **`CameraSession.capture()` は `CaptureResult` を返します** （依然としてフレーム辞書のリスト）を返し、その中には `.skipped`、`.failed`、`.daq_files`、および `.daq_matches` が含まれます。`ArraySession.capture()`の結果には、`.alignment`（アライメントワープが適用されたかどうか、および適用されなかった理由）と`.oriented`も含まれます。
> - **配列のキャプチャは、アプリと同様に、デフォルトでアラインメントされます。** `ArraySession.capture()`、`continuous()`、 `capture_repeated()`、および `ArrayHandle.capture()` / `capture_smart()` は、各メンバーの `debayered`、`display`、`radiance`、`reflectance`、およびインデックスファイルをマスターカメラのフレームにワープし、メンバーのオーバーラップ （カメラのオフセット分だけセンサーより小さい、`XMP-mapir:AlignmentJSON` `"applied": true`）にトリミングされるため、これらはChlorosアプリが同じシーンをキャプチャしたファイルと同じものとなります。 `raw`はネイティブのままです。 以前は、`aligned=True`を使用しないキャプチャでは、すべてのファイルがネイティブサイズで、ワープ処理なしで保存されていました。`aligned=False`はこの動作を維持します。`chloros-mcp`の`capture_array` および `start_capture_run` は同じデフォルト設定を採用します。[`ArraySession` メソッド](#arraysession-methods)を参照してください。
> - **`connect_all()` は、SDK を拒否するバックエンドを指定します。** ログアウトしている、または Copper 以下のプランの稼働中のバックエンドは、独自の理由 （HTTP 401 / 403）として報告され、「Chlorosバックエンドが実行されていない」とは報告されません。また、各`connect_all()`呼び出しはバックエンドを再度確認するため、ログイン後に呼び出しても同じプロジェクトオブジェクトに対して機能します。
> - **相対パスの `output_dir` は、ローカルバックエンドに対して機能します。** 相対パスは送信される前に、スクリプトの作業ディレクトリを基準として絶対パスに変換されます。バージョン 1.2.0 では、バックエンドが自身のインストールフォルダ内でパスを解決しようとしたため、「アクセスが拒否されました」というエラーが発生していました。
> - **`numpy`は、SDKの必須要件となりました**（live-frameの呼び出しはnumpy配列を返します）。
> - **1以外のエクスポートレベル名は拒否されます。** `processing=` および `levels=` は、`raw`、`debayered`、`radiance`、 `reflectance`、または `display` （または `preview` などのエイリアス。`processing=` および `"all"` も同様）を受け入れます。 カメラにアクセスする前の段階で、それ以外の名前は拒否される（HTTP 400 `UNKNOWN_LEVEL`: `capture()` は `ChlorosConnectError`、`grab()` / `frame_stream()` `FrameFetchError`）；バージョン1.2.0では、これに対して`raw`が保存または返されました。
> - **LATTICEキャプチャでは、Texture Awareは拒否されます。** LATTICEキャプチャを含むプロジェクトでは、`configure(debayer="Texture Aware (Slow, Highest Quality)")`が`ChlorosConfigurationError`を発生させ、また、インポート前に 以前にこの設定を要求していた`configure()`の後に発生します。プロジェクトが以前に保存していたTexture Awareの値は「Standard」に設定されており、`UserWarning`が発生します。LATTICEキャプチャには常に標準のデモザイクが適用されるため、バージョン1.2.0ではこの設定を受け入れつつ無視していました。Chlorosアプリでも、LATTICEキャプチャに対しては同様の設定オプションを提供していません。 [サポートされている値](#supported-values)を参照してください。
> - **カメラは、`keep_settings=True`が指定されていない限り、検出された状態に戻ります。** `ArraySession.disconnect()` / `force_disconnect()`、すべての一時関数の終了時（`capture_cameras_transient()`、 `start_multi_camera_stream()`、`test_array_sync()`、`calibrate_alignment_profile()`、`capture_aligned_multiband()`、`start_aligned_stream()`、 `capture_live_index()`）の終了および `start_camera_continuous()` の停止により、各カメラは元の状態（トリガーおよび同期ラインの配線、PTP、およびセッション中に変更されたすべてのカメラ設定（露光とゲイン、 AEターゲット、リミットおよびダンピング、ホワイトバランス、ブラックレベル、ピクセルフォーマット、ビニングおよびROI、反転、フレームレート）。 一方、`keep_settings=True`は各カメラの設定をカメラ自身のメモリに保存するため、電源投入時にその設定で起動します。`clear_settings=True`は、そのメモリを工場出荷時の設定にリセットします。 いずれの場合も、カメラはフリーラン状態のままになります。プロジェクトは、カメラを接続する際に、保存済みの設定に基づいてカメラを設定します。[カメラメモリ](#camera-memory--keep_settings--clear_settings)を参照してください。
> - **どのモードでも、プロジェクトの保存方法は同じです。** プロジェクトに接続されたカメラ、アレイ、または光センサーに対する変更（SDK、アプリ、CLI、またはMCPサーバーからの変更を含む）は、すべてそのプロジェクトの`cameras.json` / `sensors.json`に保存され、どのモードでプロジェクトを開いても、そこからデバイスが設定されます。また、`ChlorosProject`は、バックエンドに対して独自のフォルダ名を指定します。以前は、アプリのみがプロジェクトを保存していました。詳しくは [1つのプロジェクトレコード、すべてのモード](#one-project-record-every-mode)を参照してください。
> - **複数のカメラから作成されたファイルには、各カメラの名前が付けられます。** マルチバンドスタック、複数のカメラのバンドから作成されたライブインデックス、およびアレイキャプチャの `composite/` 画像は、各ソースカメラのシリアル番号、モデル、およびキャリブレーションパックをリストとして保持します（`XMP-mapir:SourceCameraSerials`、 `SourceCameraModels`、`SourceCalBundleSha256s`、…）として保持され、EXIF `SerialNumber`にはシリアル番号がカンマ区切りでリストされています。以前は、カメラ情報が含まれていなかったか、最初のカメラのシリアル番号のみが含まれていました。 [複数のカメラから作成されたファイル](#files-made-from-several-cameras)を参照してください。
> - **カメラのキャリブレーションタグが含まれているのは、`raw`ファイルのみです。** MicaSense 互換セット（`BlackLevel`、`RadiometricCalibration`、各チャンネルごとの `Band{R,G,B}RadiometricCalibration`、`MicaSenseVignetting*` セット）および `CalibrationJSON` ペイロードは、センサーの生データを記述しているため、`raw`キャプチャファイルにのみ含まれます。それ以外のすべてのファイル（キャプチャ`debayered`、`display`、`radiance`、`reflectance`、およびインデックスファイル、連続記録およびマルチカメラストリームのフレーム、およびすべての`process()`製品 には、カメラの識別情報（`XMP-mapir:CameraSerial`、`CameraModel`、`FilterType`、`LensModel`、 `CalBundleSha256`、…）を保持しており、撮影時にはそのフレーム固有の記録（`XMP-mapir:FrameJSON`：露出、ゲイン、温度、ピクセル形式、撮影時刻）も含まれます。バージョン1.2.0では、RAWセットも含まれており、これを適用するツール（Pix4D、Metashape）は、RAW値ではないピクセルにこれを適用していました。Processingは、シリアル番号と`CalBundleSha256`に基づいて、そのようなファイルのキャリブレーションを特定します。
>
> **その他の変更点：**
>
> - **アレイの自動フレームレートは、アレイが提供するレートに安定します。** トリガー数の1％以上を失ったレートは、1回ではなく数回の長時間の監視に基づいて判断され、段階的に引き下げられます。また、引き下げられた元のレートは、時折再試行されます。手動で設定したレートは変更されません。
> - **自動露出がより安定しました。** 一定の光量では露出を維持し、変化後数秒以内に安定し、わずかなハイライトを無視し、明るい日差しの下で応答しなくなったカメラを復旧させます。[自動露出の動作について](#how-auto-exposure-behaves)を参照してください。
> - **`import_images(recursive=True)`は、画像と録画のみをカウントします。** プロジェクトの`camera_cal/`を含むフォルダでは、それらのキャリブレーションファイルが画像や録画としてカウントされなくなり、すでにそれらを取り込んでいたプロジェクトでは、それらが除外されます。
> - **`configure(radiometric_output=...)`が適用されます。** これにより、プロジェクトの`Export radiance` / `Export reflectance`出力形式が設定されるようになりました。以前は、各プロジェクト独自の出力設定が優先されていました。 そのため、`radiance`、`sensor-response`、および`all`はすべて、放射輝度と反射率をエクスポートしていました。
> - **`test_array_sync()`は、カメラの電源投入状態から処理を開始します。** 実行中にカメラのPTPを有効にし、最初のトリガー（`ptp_lock`）が発行される前に、各カメラのクロックがロックされ安定するのを待ちます。各カメラは電源投入から約2分間（120秒）かけてロックされます （`ptp_settle_s=`が設定）。これは、ロックされたばかりのクロックが約2分間、最大で約200マイクロ秒の誤差を持つためです — その後、各カメラのPTPを元の状態に戻します。以前のテストでロックされたままのカメラについては、再度待機することはありません。以前は、DAQ-Eが応答しないシングルスイッチネットワークではPTPをオフにしたままにし、各カメラのフリーランニングクロックを、ロックされないPTPクロックとして読み取っていました （「PTPはロックされていません」、すべての実行で）。
> - **`.daq`の記録には、キャップが選択されたか、あるいは想定されたかが記載されます。** DAQ-U、 DAQ-M、およびDAQ-Eにはキャップが付属しておらず、Chlorosはキャップが装着されているかどうかを判別できないため、キャップ名が指定されずに接続されたセンサーには、デフォルトで*仮定*として`sunshine_cosine`が割り当てられます。すべてのChloros記録（スキーマ1.30）では、`cap_id`の横に`cap_id_source`が保存されるようになりました。キャップに名前を付けた場合は`operator`（`none`を含む）、 デフォルトだった場合は`auto_default`となります。キャップが仮定された記録を処理する際、プロジェクトのDAQキャップを「Auto」のままにすると、ファイル名を指定した`[daq-dls] WARNING`が出力されます： サンシャイン補正を適用していないDAQ-Uのみの場合、測定値は約30倍高くなります。接続時に**装着したキャップに名前を付けてください**（`connect_daq_sensor(cap_id=...)`、`DAQSensorSession.set_cap()`、`chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`、またはGUIのコンデンサドロップダウン）で**取り付けたコンデンサの名前**を指定するか、プロジェクトのDAQコンデンサを設定すると、警告は消えます。
> - **Windows での pip コンソールウィンドウの表示がなくなりました。** Python がインストールされた PC では、バックエンドとその各イメージワーカーが起動時に `pip` を一度実行し、それぞれがコンソールウィンドウを開いていましたが、これらはもう行われません。
> - **`.daq`によるライブ記録は、停電時でも維持されます。**記録は、センサーの背後で、コミットごとにジャーナルとディスクの完全同期を行って書き込まれるため、ディスクの速度が遅くてもファイルの書き込みが遅れるだけで、測定値の記録が遅れることはありません。 コミット途中で停止した記録（停電、強制終了）の場合、その横に `<name>.daq-journal` が残ることがあります：**これら2つを一緒にコピーしてください。**Chlorosは、ファイルを最初に読み込む際にジャーナルをロールバックするため、記録ファイルを開くと停止前のすべてのデータがコミットされた状態になります（最後の約3秒分が失われます）。バージョン1.2.0では、不適切なタイミングで停電が発生すると、ファイル全体が読み取れなくなる可能性がありました（「データベースのディスクイメージが不正な形式です」というエラー）。
> - **`DAQSensorSession.record_start()`がビジー状態のディスク上にあっても、センサーが停止しなくなった**： 測定値はキューに入れられ、専用のスレッドで書き込まれます。書き込みに失敗したものは、`status()`の`record_errors`（および`record_last_error`）にカウントされます。
> - **ダイレクト `LatticeCamera`：**チャンク露出／ゲインのないフレームは、カメラの**実際の**露出とゲインを報告するようになりました（フォールバックでは、自動露出実行全体を通じて固定されたままになる可能性のあるキャッシュされた値が読み込まれていました）。また、`apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')`では、Chlorosのケーブルスレーブが受けるのと同じ5 µsの入力デグリッチ処理が適用されます（フィルタリングなし。トリガーが送信されていない状態でスレーブがフレームを撮影した場合）。
> - **ソフトウェアトリガー方式のカメラは、トリガーを1回おきにドロップしなくなりました。** ソフトウェアトリガーモードのカメラは、読み出し中に到着したトリガーをキューに格納するようになりました（あるいは、Chlorosがカメラの準備が整うのを待ちます）。これは、ケーブル同期方式のカメラがすでに実施していた動作と同じです。
> - **`discover_daq_sensors()` は、DAQ-E-S `DAQ-E-S` を指定します。** すべてのイーサネットセンサーに対して、その `model` は `DAQ-E` として扱われていました。約 2 秒以内に識別チェックに応答しないセンサーも、`DAQ-E` として引き続きリストに表示されます。
> - **新機能：`chloros-mcp`、AIアシスタント用のMCPサーバー。** Claude Desktop、Claude Code、Cursor などの MCP クライアントは、画像処理（プロジェクトの作成、キャプチャのインポート、設定の選択、処理、実行結果のレポート）や、LATTICE カメラ、アレイ、DAQ 光センサーの操作（接続、プレビュー、露出や色の設定、キャプチャ、記録）を行うことができます。 そのツールは、このSDKの機能を一通り網羅しています。SDKのインストール時に一緒にインストールされ、Linux用の.debパッケージでは`/usr/bin`に配置されます。[MCPサーバー（AIアシスタント）](#mcp-server-ai-assistants)を参照してくださいを参照してください。
> - **新機能：カメラユーティリティ**。SDKが呼び出す`chloros-cli lattice`コマンド：カラープロファイル、彩度／コントラスト、ハイライト、ホワイトバランス（ワンショット、 自動、固定比率）、プリセット、フィルターモデル、ファームウェアの状態と更新、電源（プローブ、スリープ、ウェイク）、ターゲットキャリブレーション、ネットワークアダプター（チェック、推定、修正）、ファイルのヴィネット補正、DLS測定、およびホストプローブ。詳細は[カメラユーティリティ](#camera-utilities)を参照してください。
> - **新機能：残りのCLIがSDK呼び出しとして実装**されたため、CLI、SDK、および`chloros-mcp`は同一の機能を提供します（セキュリティ上の理由から、CLIではサインイン、DAQ-Eのプロビジョニング、および直接DAQサーバーへのアクセス機能のみが維持されています）: 1つのコマンドでカメラを開く機能（`start_camera_continuous()`、`start_multi_camera_stream()`、`capture_cameras_transient()`、`test_array_sync()`、`detect_array_master()`、`analyze_array_models()`）、アライメントプロファイルファイル（`calibrate_alignment_profile()`、`capture_aligned_multiband()`、`start_aligned_stream()`、 `alignment_profile_info()`、`reorder_alignment_profile()`）、インデックス（`index_catalog()`、 `compute_index()`、`capture_live_index()`）、PTPグランドマスター（`time_sync_status()`および関連ファイル）、プロジェクトフォルダ（`project_folder()`、 `set_project_folder()`、`reset_project_folder()`）、およびレシピ（`run_project_recipe()`、CLI独自のランナー）。光センサー用：`connect_daq_sensor(cap_id=...)`、`DAQSensorSession.set_cap()`、`daq_bindings()` / `set_daq_bindings()`、`daq_band_presets()`、および `light_sensor_reflectance()`（2つのセンサーによる周囲光＋物体の反射率および屈折率）；カメラ用： `CameraSession.status()` および `capture_repeated()`；アレイの場合：`ArraySession.capability()` および `set_member_settings()`（構成要素の露光量、ゲインなど）を参照してください。[過渡配列、アライメントプロファイル、および指標](#transient-arrays-alignment-profiles--indices)、[プロジェクトフォルダおよびレシピ](#project-folder--recipes)、および[DAQセンサーセッション](#daq-sensor-sessions)も参照してください。
> - **LATTICE インデックスプロダクトの改善**（NDVI およびその他のインデックス）が、`process()` から、すべての処理モードで適用されました。
> - **LATTICE OCNカメラ：日照下における放射輝度、反射率、および指標プロダクトが改善されました。** バンドアンミックス後にフリート補正が適用されます（ライブ、キャプチャ、および`process()`）。RGNカメラについては変更ありません。 詳細は [放射測定出力](#radiometric-output-lattice-multispectral-pipeline) を参照してください。
> - **GUIと同様に、すべてのキャプチャで、使用したDAQの読み取り値が保存されるようになりました。** `CameraSession.capture()` および `ArraySession.capture()` / `continuous()` は、`.daq` およびその `.match.json` を書き込みます。X に書き込みます。1.2 では、RAW のみのキャプチャを含め、DAQ がバインドまたはプールされるたびに `.daq` およびその `.match.json` が書き込まれていました。0 では、生データまたはデベイヤー処理済みのキャプチャは、`force_daq=True` でのみ保存されていました。`daq=False` はこれを無効にし、`daq_tolerance_ms=` は一致許容誤差を設定します。両方の結果には、`.daq_files` および `.daq_matches` に何が保存されたかを示しています。キャプチャは現在、DAQ の次の読み取りを待機し、露光前後の読み取り値をブレンドできるようにしています。[キャプチャが保存する DAQ の読み取り値](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches)を参照してください。
> - **`ArraySession` にはライブフレームと、サーバー側の連続キャプチャがあります。** `grab()` および `frame_stream()` は、1 つのトリガーからの numpy フレームを返します （プロジェクト API の `ArrayHandle.grab()` / `frame_stream()` と同じ読み取り）から numpy フレームを返し、`continuous()` はバックエンド内のすべての同期済みグループを保存し、 [`ContinuousHandle`](#continuoushandle)を返します。[独自のパイプラインでのライブフレーム](#live-frames-in-your-own-pipeline)を参照してください。
> - **配列 `grab()` は 1 つのトリガーに固定されます**（`ArrayHandle.grab()` および新しい `ArraySession.grab()`）。バージョン 1.2.0 では、20 個の 4-カメラの `ArrayHandle.grab()` 読み取りのうち 20 件中 12 件が、2 つのトリガーにまたがっていました。現在はすべてのフレームに `group_id` が含まれ、配列 `frame_stream()` によって各トリガーが 1 回ずつ生成されます。 [1回の読み取りにつき1つのトリガー](#one-trigger-per-read--group_id)を参照してください。
> - **バンド別およびアライン済みフレーム：** `grab()` / `frame_stream()` は `band=` を取得し、 `align=`、`bit_depth=`、および `refl_max=` をカメラおよびアレイハンドルに対して受け取ります。[単一バンドのエクスポート](#exporting-a-single-band--band)を参照してください。
> - **新しい `ArraySession` メソッドおよび引数：** `set_target_fps()`、`force_disconnect()`、`capture(after_seq=..., orient=...)` （`CaptureResult.group_seq` および `.failed` を含む）、`continuous()`、`connect_array(packed=...)`、`capture_repeated()`は連続するグループを連結し、ドライブが満杯になると停止します。
> - **`ArraySession.record()`は、GUIが開いていない状態で記録を行います。** GUIの統合ビューが閉じられている場合、バックエンドはビュー自体をレンダリングします：配列のNDVI、または`expr=`として渡されたインデックス。`RecorderHandle.stop()`は、失敗した記録（フレームが書き込まれなかったもの）に対して`ChlorosConnectError`を発生させ、停止結果を`.result`に保持されます（`with`ブロックを残しても例外は発生しません：`rec.result`を確認してください）。[`RecorderHandle`](#recorderhandle)を参照してください。
> - **新しい `DAQSensorSession` メソッド：** `enable_ptp()`、`force_disconnect()`、および DAQ-E 自身の設定用 `device_info()`、`set_network()`、`set_auto_stream()`、`reboot()`、および`update_firmware()`。 `record_stop()` は、`warning` および `gaps` を返します。記録を終了させた再起動は、その応答内で、あるいは再起動が失敗した場合は `ChlorosConnectError`（`recording_closed`；[DAQ-E デバイス設定](#daq-e-device-settings)を参照）に表示されます。 [`DAQSensorSession` メソッド](#daqsensorsession-methods)を参照してください。
> - **データ送信を停止したプールされた DAQ-E は、バックエンドによって**、以前の設定のまま再接続され、その DAQ-E での記録は、ファイル内にギャップがマークされた状態で同じ `.daq` 内で継続されます。 `record_stop()`は、その欠落部分を`gaps`として返します。バージョン1.2.0では、センサーは切断されて再接続されるまで無応答の状態が続き、これにより記録が終了していました。CLIリファレンスの [データ送信を停止するDAQ-E](cli-reference.md#a-daq-e-that-stops-delivering)を参照してください。
> - **`open_project()`は、インストール済みのSDKから動作します。** ホイール（インストーラーのやPyPI）には、必要なプロジェクトローダーが含まれるようになりました。以前は、`open_project()`が`ImportError`を発生させ、`project_cameras` / `project_sensors`という名前で`ImportError`という例外を発生させていました。`connect_all()`は、実行中のバックエンド （光センサーはDAQプール経由）を介して保存されたカメラ、アレイ、および光センサーを接続し、検索できない保存済みデバイスは、何の説明もなくスキップされるのではなく、理由を明記して `errors` の下に一覧表示される。バックエンド（GUI、 または`chloros-cli project connect`の）で既に起動している保存済みのアレイは、以前はエラーとして報告されていた（`Camera <sn> is already in array <id>`）のに対し、接続されたまま実行され続けます。[`ChlorosProject`のメソッド](#chlorosproject-methods)を参照してください。
> - **`ChlorosLocal()`、`process_folder()`、および `process_lattice_capture()` は、Windows 上で Chloros を検出します。** これらは、インストーラーがバックエンドを配置した場所（デフォルトでは `<install folder>\backend\`、`C:\Program Files\Chloros\backend\`、 インストーラ自身の記録に基づくカスタムフォルダ）を検索し、バックエンドを一切探すことなく、すでに実行中のバックエンドに接続します。バージョン1.2.0では、`backend_exe=`を指定しない限り、インストール済みのマシン上で「Chlorosバックエンドが見つかりません」というエラーが発生していました。
> - **起動時に終了するバックエンドは直ちに報告されます。** `ChlorosLocal()` またはスマートコネクト呼び出しによってバックエンドが起動され、そのバックエンドが応答する前に終了した場合、 その呼び出しは数秒以内に例外を発生させ、`ChlorosBackendError`は`ChlorosLocal`から、`ChlorosConnectError`はスマート-コールの `ChlorosLocal` および `ChlorosConnectError` から、終了コード、実行されたコマンド、およびバックエンドが出力した最後の行を伴って例外が発生します。1.2.0 では 60 秒の起動タイムアウトを待機するのみで、その理由は報告されませんでした。[バックエンドの自動起動](#backend-auto-start)を参照。
> - **スクリプト間でバックエンドが実行され続ける。** `with ChlorosLocal(...)`ブロックを終了すると、 `process_folder()` / `process_lattice_capture()` が戻り、`chloros-mcp` が終了しても、SDKが起動したバックエンドは停止しなくなりました。 次のスクリプトがそれを再利用します。バックエンドは、`logout()`（Chlorosアプリが開いているか、実行が進行中の場合を除く）、`shutdown_backend()`、またはバックエンドの環境で何も使用されていない状態が30分続いた後に自動的に停止します （バックエンドの環境変数`CHLOROS_BACKEND_IDLE_MIN`を設定するとこの動作が変更されます；`0`：決して終了しません）。
> - **SDKによって起動されたバックエンドは、強制終了されるのではなく、シャットダウンを求められます。** `shutdown_backend()`では、バックエンドがカメラや光センサーを解放し、記録を完了できるようにします。10秒経過しても実行中の場合のみ、バックエンドは終了されます。1.2.0では即座に終了させていました （Windowsでは強制終了）、これにより、そのバックエンドに接続されたDAQ光センサーが応答しなくなる可能性がありました。SDKが実行中と検出したバックエンドについては、引き続きそのままにされます。
> - **Linuxでは、自動起動機能は現在のフォルダから`backend_server.py`を実行することはありません。** バックエンドがインストールされていない場合、バックエンドを含むフォルダでスクリプトを実行すると、そのスクリプトがバックエンドを実行していました。現在は、インストール済みのバックエンドのみが起動されます（または、`ChlorosLocal` の場合、`backend_exe=` で指定されたバックエンドが起動されます）。
> - **`configure(indices=...)`**は、各プリセットをそのカメラのバンドに対応するプロジェクトに追加し、新しいプロジェクトを含め、追加できない名前については警告（`UserWarning`）を表示します。**`import_images()`**は、`.daq`の録画を`scan_count`として報告し、LATTICEのキャプチャフォルダに関する警告は表示されなくなりました。
> - **`read_image_audit_tags()`はpipインストールから動作します。** 画像のメタデータを独自に読み取ります。バージョン1.2.0では、Pythonパス上にデスクトップ版インストールの内部ファイルがない場合、`{}`を返していました。
> - **`capture()`（`connect_array()`の直後）**は、HTTP 503「Array still warming up」エラーで失敗する代わりに、配列の最初のフレームグループが到着するまで最大20秒待機します。
> - **`status()` はストールを報告します。** 新しい `stalled` フィールドは、グラブループが実行されている間は True になりますが、何も公開されず、その後 `actual_fps` は 0 になります。1.2.0 では、`actual_fps` はストール中も最後の値を維持していたため 、ストール状態の配列は `actual_fps > 0` を示し、キャプチャは 503「最新の同期済みフレームグループなし」エラーで失敗していました。
> - **不完全なフレームセットやキャプチャ時の503エラーの減少：** 各スレーブカメラのトリガー入力のグリッチが除去されました。
> - **パックド12ビット転送**（`connect_array(packed="auto")`）により、フルセンサーアレイのフレームレートが約3分の1向上しました。[パックド12ビット転送](#packed-12-bit-transport-packed)を参照してください。
> - **ROI 内のアレイ露光：** 小さな ROI であっても、メンバーの露光時間がセンサーのフリーラン周期（768 × 304 の場合 4.9 ms）に制限されることはなくなりました。[アレイメンバーの露光](#exposure-on-array-members)を参照してください。(#exposure-on-array-members)を参照してください。
> - **自動露光によってアレイの自動フレームレートが低下することはなくなりました。** ケーブル同期されたアレイは、ネットワークおよびカメラの読み出しが許容するレートで動作します。自動露光は、そのレートによって与えられる期間を埋め、その後ゲインを上げます。その期間よりも長い手動露光の場合、拒否されるのではなく、それに合わせてトリガー速度が低下します。[アレイメンバーの露光](#exposure-on-array-members)を参照してください。
> - **`read_nodes()`**は、自動露出の実行中に `frame_values` および `notes` を追加します（[自動露出下での `read_nodes()`](#read_nodes-under-auto-exposure--frame_values-and-notes)を参照）。カメラが露光やゲインを拒否した場合、**`set_settings()`**は受け入れ可能な範囲を示し、その値に対してオフにされていた自動モードを元に戻します。 `set_settings(target_brightness_auto=True)`は新規です。
> - **`orient=`** `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` および `ArrayHandle.capture()` において / `capture_smart()`は、カメラの向きに応じて画像を回転させます。[`ArraySession` メソッド](#arraysession-methods)を参照してください。
> - **`DLS()` には、センサー固有の `mac_address` が必要です。** デフォルトの MAC アドレスはもはや存在しません（以前は 1 ユニットのアドレスでした）。MAC アドレスを指定しない `DLS()` は、`DLSError` を発生させます。 [DAQ-M バンドの振幅](#daq-m-band-magnitudes--dlsband_magnitudes)を参照してください。
> - **Windowsインストーラー**には、「Windowsの長いパスを有効にする」と「Microsoft DefenderのスキャンからChlorosバックエンドを除外する」というオプションがあり、どちらもデフォルトでチェックされています。[ステップ1](#step-1--install-the-chloros-platform-package)を参照してください。
> - **Linux用.debパッケージ**は、システム用のPython SDKをインストールします（`python3` （以前はディスク上に wheel を配置するだけでした）をインストールし、カメラストリームに必要なカーネルのソケット受信バッファを拡大し、ユーザーとして実行されるバックエンドサービスを追加します。Jetson パッケージは `chloros_1.3.0_arm64_jp6.deb` で、JetPack 6 向けであり、 JetPackのCUDA版PyTorchに対応しています。[ステップ1](#step-1--install-the-chloros-platform-package)および[ステップ2](#linux-deb)を参照してください。

---

## 60秒でわかるアーキテクチャ

このSDKは、Chlorosバックエンド（デスクトップGUIやCLIが使用するのと同じFlaskサーバー）の上に構築された、軽量なPythonレイヤーです。自動化を行うには、`chloros_sdk`をインポートして高レベルなメソッドを呼び出します。内部的には、 すべての呼び出しが、ポート5000上のローカルバックエンドへのHTTPリクエストに変換されます — `http://127.0.0.1:5000/api/...` （Windowsでは最初に`::1`に解決され、IPv4のみのバックエンドに対してはリクエストごとに約2秒かかるため、意図的に`localhost`は使用していません）。 バックエンドはハードウェアプール（カメラ、DAQセンサー、アライメントプロファイル、フレームバッファ）を管理しているため、SDKスクリプトはGUIと共存でき、シリアルポートやNICの帯域幅を奪い合うことはありません。

使用するインターフェースは3つあります：

1. **`ChlorosLocal` + フリー関数** (`process_folder`、`process_lattice_capture`) — 画像処理パイプライン。1回のPython呼び出しで、フォルダ全体の画像に対してキャリブレーション／デベイヤー／インデックスエクスポートを実行します。
2. **Smart-connectハンドル** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — ライブハードウェア用の永続的なバックエンドセッションを開きます。GUI と同じ「smart-prep」フロー：ネットワークプローブ、ティアの自動選択、PTP、AE シード、GPIO トリガー設定。
3. **`ChlorosProject` / `open_project`** — 保存済みのプロジェクト（`cameras.json` + `sensors.json` + `project.json` を含むフォルダ）を読み込み、そのカメラ、アレイ、光センサーを一度に接続し、名前付きハンドルを使用してキャプチャを実行します。

サーフェス 1 および 2 は、まだリスニング中のバックエンドが存在しない場合、**ローカルバックエンドを自動起動**します（GUI や CLI が起動するのと全く同じバンドルされたバイナリです）。そのため、バックエンドを事前に起動しなくても、新しいシェルからスクリプトをそのまま実行できます：`ChlorosLocal` （および `process_folder()` / `process_lattice_capture()`）、`connect_camera()`、 `connect_array()`、`connect_daq_sensor()`、 `discover_lattice_cameras()`、および `discover_daq_sensors()`。`list_cameras()`、`list_arrays()`、`list_daq_sensors()`、 `attach_array()` および `analyze_array_network()` は決して起動しません。これらは実行中のバックエンドを必要とします。`auto_start_backend=False` を渡してオプトアウトしてください （例：決して起動されないリモートバックエンドを指定する場合など）。[バックエンドの自動起動](#backend-auto-start)を参照してくださいを参照してください。Surface 3 の動作は異なります。`open_project()` は `auto_start_backend` パラメータを受け付けず、`connect_all()` はバックエンドを起動しません — `http://127.0.0.1:5000`を一度プローブし、応答がない場合は、直接（バックエンドなし）の`lattice_sdk`デバイス制御にフォールバックします。これには、Arena SDKランタイムと、Pythonでインポート可能なOpenCV （`cv2`）をPythonでインポート可能にする必要がありますが、SDKのwheelパッケージではこれらはインストールされません。 これらがなければ、保存された各カメラおよび配列は、その理由とともにエラーとして報告されます。プロジェクトの光センサーには、バックエンドを使用しないパスはありません。バックエンドが実行されていない場合、保存された各センサーは、その理由とともにエラーとして報告されます（[`SensorHandle`](#sensorhandle)を参照)。`proj.process()`と`stream(..., overlays=True)`のみが、`ChlorosLocal()`（自動起動機能あり）を遅延構築します。

これら3つすべては認証ゲート方式を採用しています。マシン上で `chloros-cli login` を一度実行するか、デスクトップ GUI 経由でサインインしてください。 有効なセッションがない状態で呼び出しが行われた場合に発生する例外は、インターフェースによって異なります（[例外](#exceptions)を参照）：`ChlorosAuthenticationError`は`ChlorosLocal`（および`process_folder()` / `process_lattice_capture()`)、スマート-connect呼び出しおよびそのセッションからは`ChlorosConnectError`、`status: "error"`には`error_code`が、`analyze_array_network()`からは`errors`が報告され、`connect_all()`レポート内のエントリ。

要件：
- Python 3.7 以上（パッケージの仕様による。3.10 で開発・テスト済み）
- ローカルにChloros Desktopがインストールされていること（バックエンドバイナリはインストーラー内に同梱されています）
- **有料のChloros+プラン（Copper以上）**のいずれかで有効なChloros+ログインがあること。無料の**Iron**プランではSDK/CLIへのアクセスはできません。これは**サーバー側**で強制されます：SDK/CLIフラグが設定されたすべてのリクエストには、有効なセッションと有料プランの両方が含まれている必要があります。Copper未満のプランでは、スマートコネクト呼び出しから`403`と`error_code: PLAN_UPGRADE_REQUIRED`（`ChlorosLicenseError`は`ChlorosLocal`、スマートコネクト呼び出しからは`ChlorosConnectError`）。ログアウトした呼び出し元には、代わりに`401` / `AUTH_REQUIRED`が割り当てられます （`ChlorosAuthenticationError` は `ChlorosLocal` から、`ChlorosConnectError` はスマートコネクトの通話から）— これら 2 つは区別される。なぜなら、`chloros-cli login` を再実行すると を実行すると、`401`は修正されますが、プランのアップグレードが必要な`403`は修正できないため、これら2つは別物となります。
- プランの猶予期間中はオフラインでの使用がサポートされます。この期間中、ティア情報はサーバー検証キャッシュ（5分間）または署名済みでマシンに紐付けられたライセンスキャッシュ（月額プランの場合は30日間、年間プランの場合はサブスクリプションの有効期限まで）から読み込まれます。この猶予期間が終了すると、プランは無料プランに切り替わり、マシンがサーバーに一度接続できるまでSDK/CLIへのアクセスが停止します。`chloros-cli status`（`GET /api/license-status`）は無料プランでもアクセス可能な状態を維持するため、原因を確認できます。
- Windows 10/11 64ビット、**Ubuntu 22.04 LTS 以降**、または Jetson (JetPack 6)。Ubuntu 20.04 は**サポートされていません**： `.deb`の依存関係は、`libc6 (>= 2.35)`を含め、バックエンドがリンクしているものから派生しており、focalにはglibc 2.31が同梱されているためです。

---

## インストール

Python SDKは、Chlorosバックエンドの上に構築された薄いPythonレイヤーです。別のマシン上でバックエンドを駆動する場合（[リモートバックエンドモード](#remote-backend-mode-pip-only-host-via-tunnel)を参照）を除き、**Chlorosデスクトップパッケージをローカルにインストール** （WindowsインストーラまたはLinux `.deb`）をローカルにインストールする必要があります。このパッケージには、DAQライトセンサーが含まれています。このパッケージには、バックエンドバイナリ、LATTICEカメラ用のArena SDKランタイム、およびキャリブレーションバンドルが含まれています。

最新のダウンロード：[`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### ステップ 1 — Chloros プラットフォームパッケージのインストール

#### Windows (.exe)

1. ダウンロードページから `Chloros Installer 1.3.0.exe` をダウンロードします。
2. インストーラーを実行し、ウィザードの指示に従ってください。デフォルトのインストールパスは `C:\Program Files\Chloros\` です。
3. Chloros を少なくとも 1 回起動し、Chloros+ アカウントでサインインしてください。

インストーラーは、すべてのネットワークプロファイルにおいて、バックエンド（`chloros-backend.exe`、 それ以外はありません）が、すべてのネットワークプロファイルにおいてWindowsファイアウォールを通過できるようにします。具体的には、カメラストリーム、PTP、mDNS、およびDAQ-Eストリーム用のUDP受信、ならびにファームウェア更新中にDAQ-Eがコンピュータへ戻る接続用の、ローカルサブネットからのみ許可されるTCP受信です ([`update_firmware()`](#daq-e-device-settings))。アンインストーラーはこれらのルールを両方とも削除します。バックエンドのHTTPAPIは、いずれの場合も`127.0.0.1`に残ります。

ウィザードの**「システム設定」**グループには 2 つのオプションがあり、どちらもデフォルトでチェックされています：

| オプション | 機能 | サイレントインストールオプトアウト |
| --- | --- | --- |
| **Windowsの長いパスを有効にする** | プロジェクトフォルダーを、Windowsの260文字というパス制限を超える深さに配置できるようにします。長いパスが無効になっている場合（Windowsの既定設定）、製品に対して深すぎるプロジェクトフォルダーは、処理が開始される前に拒否されます — [トラブルシューティング](#troubleshooting-pointers)を参照）。 | `/MERGETASKS="!longpaths"` |
| **Microsoft DefenderのスキャンからChlorosバックエンドを除外** | Defenderによるスキャンを停止します。キャプチャレートが高い場合、スキャン処理はレートの約30％を消費します。9台のカメラアレイでROI 512を測定した結果、連続RAWキャプチャはスキャンありで52.7キャプチャ/秒、バックエンドを除外した場合は77.7キャプチャ/秒でした。 | `/MERGETASKS="!defenderexclusion"` |

連続キャプチャでは、Defenderの負荷が`defender_cores`として報告されます（[`ContinuousHandle`](#continuoushandle)を参照）。

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

`dpkg -i`ではなく、`apt install ./…`を使用してインストールしてください。aptはPython SDKの依存関係もインストールしますが、これはパッケージが必須としているのではなく推奨しているものです（[ステップ2](#linux-deb)を参照）。

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

JetPack 6 (L4T R36) 上の Jetson Orin / Orin NX / Orin Nano 用です。 その PyTorch は JetPack 6 の CUDA ビルドであるため、GPU 処理は Orin の GPU 上で実行されます。JetPack 6 がインストールする CUDA および cuDNN ライブラリを使用します（これらが欠落しているイメージでは `sudo apt install nvidia-jetpack`）。

#### Linux：受信バッファとバックエンドサービス

どちらの .deb パッケージも、LATTICE カメラのストリームに必要なカーネルのソケット受信バッファを 32 MB に増やします（`/etc/sysctl.d/60-chloros-gige.conf`）。カーネルのデフォルト設定では、 カメラからのパケットはカーネル内で破棄され、配列はフレームやフレームレートを失います。`chloros-cli lattice network`は、バッファが不足した際にその状態を報告します。CLIリファレンスの [Linuxホスト — ソケット受信バッファ](cli-reference.md#linux-hosts--socket-receive-buffers) を参照してください。

ヘッドレスマシンでは、スクリプトが実行中のバックエンドを検出できるように、**自分のユーザーとして**バックエンドをサービスとして実行してください（`connect_all()`はバックエンドを起動しません）：

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

これにより、バックエンドはあなたとして実行されます。つまり、バックエンドが書き込むキャプチャデータはあなたのものとなり、あなたの `chloros-cli login` が使用されます。root `chloros-backend.service` は、それを有効にしているマシンには引き続きインストールされていますが、root バックエンドが書き込むすべてのファイルは root に属します。 CLIリファレンスの [バックエンドをサービスとして実行する](cli-reference.md#running-the-backend-as-a-service-headless) を参照してください。

サービスによって実行されるバックエンドは共有されており、クライアントによって停止されることはありません。SDKスクリプトや`chloros-cli`は、そのバックエンドをそのまま利用します （`ChlorosLocal.shutdown_backend()` は、`ChlorosLocal` が自ら起動したバックエンドのみを停止します）、また `GET /api/status` はこのユニットを `service` と名付けます。 `sudo systemctl restart chloros-backend@$USER.service`で再起動してください。パッケージのアップグレード中は、インストール中にバックエンドサービスが停止します。 そのため、実行中のスクリプトはカメラやセンサーとの接続を失い、その後、有効になっているものをすべて再起動します。パッケージを削除すると、それらが停止し、有効になっているものは無効になります。一度に実行できるバックエンドは1つだけですので、1つのユニットのみを有効にしてください。2つ目を有効にすると、起動を拒否して `failed` （終了ステータス 98、再起動されません）の状態のまま停止し、その `systemctl status` に理由が記載されています。ユニットを切り替えるには、使用していない方のユニットを無効にしてください。

### ステップ 2 — Python SDK のインストール

**Chlorosのインストーラーには、対応するSDKホイールが同梱されています。** すべてのWindowsインストーラーおよびLinuxの.debパッケージは、GUI／CLI／バックエンドのバージョンと完全に一致する「`chloros_sdk-X.Y.Z-py3-none-any.whl`」をディスクに配置します。バージョンを同期させるためにPyPIをわざわざ探す必要はありません。

#### Windows

インストーラーは、システムのPythonを使用して、バンドルされたwheelに対して`pip install`を自動実行します（`py.exe`ランチャーが推奨されますが、`python -m pip`にフォールバックします）。特別な操作は不要です。インストールが正常に完了すれば、`import chloros_sdk`はPython環境下で動作します。システムにPythonがインストールされていない場合、インストーラーはこの手順を自動的にスキップし、GUIおよびCLIは引き続き動作します。

#### Linux (.deb)

両方の .deb パッケージ（x86_64 および Jetson）は、システム用の SDK（`python3`）をインストールするため、インストール後は pip やネットワーク接続がなくても `import chloros_sdk` が動作します。パッケージ （`chloros_sdk`、 `lattice_sdk`、`lattice_cam`、`arena_api`、およびプロジェクトローダー）は`/usr/lib/python3/dist-packages`に配置され、 また、SDKの`chloros-mcp`コマンド（[MCPサーバー](#mcp-server-ai-assistants)）は`/usr/bin`に配置されます。これらはすべて当該パッケージが所有しており、アップグレード時には置き換えられ、パッケージを削除するとこれらも削除されます。

SDK独自の要件である`python3-numpy`および`python3-requests`は、**必須ではなく推奨**です（GUI、CLI、バックエンドは独立して動作します）： `sudo apt install ./chloros_<version>_amd64.deb`（Jetson では `…_arm64_jp6.deb`）はこれらをインストールしますが、`dpkg -i` はインストールしません。 インストール出力には、`python3`がインポートするSDKのバージョン、不足している要件、 および、`python3`のパス上の別の場所に存在し、パッケージのコピーを上書きしている`chloros_sdk`（例：以前に`pip install --user`でインストールされたもの）を報告します。

システムのパッケージを認識しない virtualenv 内では、同梱の wheel をそこにインストールしてください：

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

エアギャップ環境の Jetson 展開の場合、これは完全にオフラインで行われます。SDK と wheel はすでにディスク上に存在しています。

#### 公開 PyPI

pip のみのホスト（Chloros デスクトップパッケージがインストールされていない場合。この場合、SDKは別のマシン上のバックエンドを駆動します。[リモートバックエンドモード](#remote-backend-mode-pip-only-host-via-tunnel)を参照）では、次のようにします：

```bash
pip install chloros-sdk
```

PyPIには最新の安定版が掲載されています。SDKは、それが駆動するChlorosバックエンドと同じバージョンに保ってください（[バージョン管理と互換性](#versioning--compatibility)を参照）。 DAQ-M ヘルパー `DLS` には、`bleak`: `pip install "chloros-sdk[camera]"`（または `pip install bleak`）も必要です。

#### 確認

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ のサブスクリプションが必要です。** すべての SDK 呼び出しには、有効な Chloros+ ログインが必要です。各マシンで `chloros-cli login user@example.com 'YourPassword'` を 1 回実行してください。認証情報は `~/.chloros/` にキャッシュされます。

### デスクトップパッケージは必要ですか？

ほとんどのワークフローでは、pipパッケージだけでは**不十分**です。各SDKインターフェースに必要なものは以下の通りです：

| SDKインターフェース | デスクトップパッケージが必要？ | 理由 |
| --- | --- | --- |
| `ChlorosLocal`、`process_folder`、`process_lattice_capture` | **はい**（起動するため）**／いいえ** （すでにアクセス可能なバックエンドがある場合） | 実行中のバックエンド（デスクトップアプリのもの、または別のスクリプトで起動されたもの）を使用する。そうでない場合は、インストール済みのものを自動起動する：`/usr/lib/chloros/chloros-backend`（Linux）または `<install folder>\backend\chloros-backend.exe` （Windows；デフォルトでは `C:\Program Files\Chloros\backend\`、カスタムインストールフォルダはインストーラの記録から読み込まれます）。`backend_exe=` を指定して、自分で名前を付けることも可能です。 |
| `chloros-mcp` ([MCPサーバー](#mcp-server-ai-assistants)) | **はい**（起動する場合）**/ いいえ**（バックエンドがすでに利用可能な場合） | `ChlorosLocal` を基盤としているため、上の行と同じです。 |
| `read_image_audit_tags` | **いいえ** | 処理済み画像自身のメタデータを読み取ります（Python標準ライブラリのみ）。 |
| `connect_camera`、 `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **はい**（ローカル）**/ いいえ**（リモート） | バックエンド経由の純粋なHTTPクライアント。ローカルバックエンド → デスクトップパッケージが必要。リモートバックエンド →**トンネル経由**の`backend_url=`（「リモートバックエンドモード」を参照 — 同梱のバックエンドはループバックのみにバインドされます）。 |
| `ChlorosProject` / `open_project` | **はい** | 保存されたプロジェクトをバックエンド経由で操作します。 |
| 直接 LATTICE クラス（`LatticeCamera`、`CameraPool`、`Calibration`、`DLS`、…） | **はい** | デスクトップパッケージに含まれるArena SDKランタイムと、Python環境にOpenCV (`cv2`) が必要です。これらはデスクトップパッケージでもSDKでもインストールされません (`pip install opencv-python`)。`CAMERA_AVAILABLE`は、インポート時には`False`として扱われます。 また、`DLS` には `bleak` も必要です。 |

### リモートバックエンドモード（pip 専用ホスト、トンネル経由）

> **インストールされたバックエンドは LAN 経由では到達できません。** インストールされたバックエンドは
> ループバックポートでのみリスニングしているため（`127.0.0.1` / `::1`）、`backend_url="http://<lan-ip>:5000"`は
> **インストール済みのChlorosに到達できません**。 別の
> マシン上のバックエンドを操作するには、そのループバックポートを手動で転送し、SDKを
> トンネルに向けてください：

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

ヘッドレス／CI／ロボティクスホストでは、フルデスクトップ環境がインストールされた1台のマシンを「Chlorosサーバー」として保持し、その他の場所には`pip install chloros-sdk`を配置できます。ただし、それら間の通信には、上記のユーザーが手配したトンネルを使用し、直接のLAN URLは使用しません。

> **pip専用ホスト上の`ChlorosLocal`。** `ChlorosLocal(backend_url=BACKEND)`は、デスクトップ環境をインストールしなくても到達可能なバックエンドに接続します。ローカルのバックエンドバイナリを検索するのは、バックエンドを起動する必要がある場合（応答がない場合や`auto_start_backend=True`が発生した場合）に限られます。バージョン1.2.0までは、まずバイナリを検索し、バックエンドに到達可能であっても、pip専用ホスト上で`ChlorosBackendError`（「Chlorosバックエンドが見つかりません…」）を発生させていました。指定したフォルダ （`process_folder`、`import_images`）は、このマシン上で確認され、バックエンドによって読み込まれるため、リモートバックエンドを使用する場合は、共有ドライブなど、両方で同じフォルダ名を指定する必要があります。

### DAQ専用ワークフロー（pip専用ホスト）

`pip install chloros-sdk`だけではDAQセンサーを駆動できません。DAQセンサーは常に、`connect_daq_sensor()`を備えたChlorosバックエンドを介して駆動されます：

- **このマシン上のセンサー：** ここにChlorosデスクトップパッケージをインストールします。`connect_daq_sensor()`は、バックエンドが実行されていない場合、バックエンドを起動します。
- **別のマシン上のセンサー：** そのマシンにはデスクトップパッケージが必要です。[リモートバックエンドモード](#remote-backend-mode-pip-only-host-via-tunnel) のトンネルを介してそのバックエンドにアクセスします。バックエンドは自身のマシン上でセンサーを開きます。 そのため、DAQ-Uはそのマシンに接続され、DAQ-MはそのマシンのBluetooth通信範囲内にあり、DAQ-Eはそのマシンが到達可能なネットワーク上に存在する必要があります。

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

いずれの場合も、バックエンドはChloros+プランを適用します。つまり、バックエンドを実行しているマシン（`chloros-cli login`、またはデスクトップアプリ）に、有料プラン（Copper以上）でログインする必要があります。バックエンドからログアウトしている場合や、Copper未満のプランの場合は、いずれもバックエンドの理由（`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`）を含む`ChlorosConnectError`が発生します。

---

## クイックスタート

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## トップレベル API インデックス

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## 画像処理 — `ChlorosLocal`

主要なパイプラインクラスです。コンストラクタが呼び出されるとバックエンドを起動し（応答するバックエンドが存在しない場合、および `auto_start_backend=True`）、プロジェクトを作成・設定し、進行状況を監視し、実行後の要約を返します。

### コンストラクタ

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### メソッド

| メソッド | 説明 |
| --- | --- |
| `create_project(project_name, camera=None)` | 新しいプロジェクトを作成します（オプションで、`"Survey3N_RGN"`のようなカメラテンプレートを使用することも可能です）。 |
| `open_project(project_path, relink=None)` | 保存済みのプロジェクト（そのフォルダ、またはプロジェクトフォルダ内のファイル名）を開き、開いているプロジェクトとして設定します。 移動されたプロジェクトのパスは、現在の所在フォルダを指すように再設定されます。`relink`は、その他の欠落ファイルが現在格納されているフォルダ名を指定します。バックエンドからの応答を`source_files`で返します。 見つからない場合は `ChlorosProjectFilesMissingError` を発生させます。以下の **移動されたプロジェクト** を参照してください。 |
| `import_images(folder_path, recursive=False)` | RAW/TIF/JPG/DNG 画像 **および `.daq` 光センサーの記録** をインポートします。`count` （画像）および `scan_count` （記録データ）を返します。フォルダにどちらも含まれていない場合にのみ警告を表示します。 |
| `export_light_sensor(daq=True, csv=True)` | プロジェクト内のすべての光センサー記録について、校正済みの `.daq` および `.csv` を `<project>/Light Sensor/`に書き込みます。[光センサーの記録](#light-sensor-recordings--calibrated-daq--csv)を参照してください。 |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | 処理ノブを設定します。 `indices` はプリセット名（`NDVI`、`GNDVI`、 `OSAVI`、…）であり、それぞれそのカメラのバンドに合わせてプロジェクトに追加されます。プリセットではない名前、またはそのバンドでは作成できない名前（`NDRE`にはREとNIRが必要ですが、単一のカメラではこれらを提供できません）、`UserWarning`という名前が付けられ、インデックス画像は生成されません。`daq_log_path`は、すべてのフレームが照合される光センサー記録 （`.daq` または `.csv` ファイル、あるいはそれらのフォルダ）を指し、画像と共にインポートされた記録の代わりに、LATTICE および Survey3 のすべてのフレームがこれと照合されます。 パスが存在しないか、記録が含まれていない場合は、インポートされたものが使用されます。 |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | パイプラインを実行します。`{"status": "complete", "async": False}`を返し、 さらに、バックエンドが提供する場合、`summary`キーを返します — [実行後の要約とヒント](#post-run-summary--hints)を参照してください — 実行のエクスポートが検証できなかった場合は `export_check_failure`、イメージが失われた場合は `image_failure` を返します。`wait=False` は、一度に `{"status": "started", "async": True}`を一度に返します。 |
| `get_config()` / `get_status()` / `status()` | バックエンドの状態を確認します。 |
| `logout()` | ログアウト：キャッシュされた認証情報をクリアし、このマシンのデバイススロットを解放します（結果として `device_deregistration` となります）。 進行中の処理実行は停止されません。開始時のモードのまま完了し（結果内の `run_in_progress: true` および `run_mode`）、次の実行では標準処理が使用されます。 |
| `shutdown_backend()` | このインスタンスがバックエンドを起動していた場合は停止します。実行中であることが確認されたバックエンドについては、そのまま放置されます。`with` ブロックが残っている場合や、インスタンスがガベージコレクションの対象となっている場合は、バックエンドは停止しません： バックエンドは次のスクリプトのために実行を継続し、30分間何も使用されていない状態で自動的に停止します。バックエンドにはシャットダウンが要求されるため、まずカメラや光センサーを解放し、録画を完了させます； 10秒経過してもまだ実行中の場合にのみ、バックエンドは終了されます。 |
| `discover_cameras()` | **このインスタンスのバックエンドを介して** LATTICEカメラを検出します (`/api/camera/discover`)。辞書のリスト（`serial`、`model`、 `ip`, …) のリストを返します。これは、GUI や CLI で表示されるものと同じ形状です。見つからない場合やバックエンドに到達できない場合は空のリストになります。 |
| `camera_capture(output_dir, format="tiff", **settings)` |**バックエンド経由で**（このハンドルによって自動起動され）、GUIやCLIと同じ準備状態（デフォルトで12ビット、プールの再利用、埋め込みキャリブレーションメタデータ）で単一フレームをキャプチャします。ターゲットの解決には `serial=` または `device_index=` を使用してください； `exposure`/`gain`/`pixel_format`/`preset`を`**settings`として渡すとして渡す。レガシーメタデータ辞書（`filepath`、`width`、 `height`、`pixel_format`、`exposure_time`、`gain`、`timestamp`）. |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | プールされたカメラからオーバーレイ合成されたプレビューフレームを生成 — バックエンドの `/api/camera/<serial>/stream-annotated` ルートを経由する軽量 MJPEG クライアント （サーバー側で描画されるゼブラ／グリッド／クロスヘア／ヒストグラム／ピーキング／スポット）を経由します。`decode=True`はBGR配列を出力し、OpenCV（`cv2`）が必要ですが、SDKにはインストールされていません。`False`は生のJPEGバイトを返します。また、プロジェクト単位で`ChlorosProject.stream(overlays=True)`としても利用可能です。 |

**実行中の処理はプロジェクトを維持します。** 処理の実行中（このスクリプト、アプリ、`chloros-cli process`、または`chloros-mcp`によるもの）は、バックエンドはプロジェクトの作成、開く、インポート、または処理設定の変更を拒否します （HTTP 409 `PROCESSING_IN_PROGRESS`）：`create_project()` および `import_images()` は `ChlorosProcessingError` を発生させ、 また、`configure()`は`ChlorosConfigurationError`を発生させ、実行中のモードとは異なるモードの`process()`は`ChlorosProcessingError`を発生させます （実行は開始時のモードを維持する）、それぞれバックエンドからのメッセージを伴う。プロジェクトに保存されたキャプチャは、実行終了時にそれに合流する（その応答の `project_add` がそう示している）。実行が終了するのを待ち（`get_status()`）、その後続行してください。**プロジェクトの移動。** プロジェクトの`project.json`は、各ソースファイル（すべての画像および光センサーの記録）をフルパスで指定しているため、プロジェクトフォルダ、あるいは「projects」フォルダ全体を別のドライブに移動しても、それらのファイル名は以前の場所のままになります。 ファイルを開くたびに、可能な範囲で自動的に修正が行われます。つまり、プロジェクトフォルダ内にあったファイルは、プロジェクトが現在置かれているフォルダ内で検索され、そこで見つかった各パスは再指定されて保存されます （`project.json`は、まず`project.json.before-relink-<date>-<time>`としてバックアップされます）。パスの再設定は、その場所にファイルが存在する場合にのみ行われます。 プロジェクトフォルダの外にあったファイル（ハブセッションや「フォルダの追加」によるインポートなど）は、`relink=<the folder they are in now>`（またはその上位のフォルダ）でのみ検出されます。`open_project()`は`source_files`を返します： `total`、`found`、`missing`、`missing_roots`（欠落していたファイルが入っていたフォルダ）、 `missing_outside_project` および `relink`（再参照されたファイル、元の場所から新しい場所へのパス）。一部のファイルが欠落しているプロジェクトは残りのファイルと共に開かれますが、欠落ファイルが1つもないプロジェクトは開かれず、 また、`open_project()`は`ChlorosProjectFilesMissingError`（`.missing_roots`、`.audit`； HTTP 409 `PROJECT_SOURCES_MISSING`）。CLIの`project open --relink`および`chloros-mcp`’`open_project` (`relink_folder`) も同様の動作をします。

確実なクリーンアップを行うためのコンテキストマネージャーとして使用してください：

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### 光センサーの記録 — 校正済み `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E は、そのキャリブレーションバンドルが**なくても**記録可能です。これは、
公開されている [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
レコーダー（`record_daq.py`）がデフォルトで行っていることとは異なります。それらは生のセンサーカウントを書き出し、
ファイルにタイムスタンプを付与することで、Chlorosがそのセンサーの工場校正データを**シリアル番号**順に取得できるようにします（まずローカルキャッシュ、
から、次にMAPIR Cloudから——そのセンサーの工場出荷時キャリブレーションを取得し、インポート時に適用します。

Chlorosは、その結果を1回の記録につき2つの製品として、
`<project>/Light Sensor/`の下に書き出します：

| 製品 | 内容 |
| --- | --- |
| `<name>_calibrated.daq` | 再処理可能なアーカイブ — ライブ記録と同じスキーマですが、これを生成したバンドルが宣言されています。これを再インポートしても、**再度**校正されることはありません。 |
| `<name>_calibrated.csv` | センサー独自の波長グリッドにおける分光放射照度（W/m²/nm）。測定値ごとに1行、さらに測光パラメータの列（総電力、明所視／暗所視ルクス、PPFDおよびその青・緑・赤の分割値、ピーク波長）が含まれます。 |
| `<name>_raw.daq` / `<name>_raw.csv` | **バンドルのないセンサーのみ（DAQ-A）.** センサーの生スペクトルカウント値 — 放射照度ではありません。以下を参照してください。 |

`<name>` は記録そのもののファイル名です。異なるフォルダにある 2 つの記録が
1つ — 2台のカメラによって
同じミリ秒に記録されたキャプチャごとの測定値（`<capture>/daq/daq_data_<sensor id>_<ts>.daq`） — を共有できますが、それぞれが独自の成果物を取得します。最初の記録は
`<name>`を保持し、次の記録は名前**の後に**区別するためのフォルダ名を付加します
（`<name>__<capture>_daq_calibrated.daq`）。したがって、記録の日付と時刻が依然として先頭に表示されます。

`process()`は、その処理段階の一つとしてこのエクスポートを実行します。画像データは**不要**です：
単独で飛行する光センサーだけでも十分なワークフローとなり、そのようなプロジェクトには
設計上、画像データは一切含まれません。

**DAQ-Aの記録データは生カウントとしてエクスポートされます。** DAQ-Aシリーズは、シリアル番号ごとの
バンドルシステムが導入される以前に開発されたものであり、取得すべきバンドルが存在しません。代わりに、現場で
反射率ターゲットを用いて校正されるため、バンドルを必要としたことがないのです。 これらの記録データは、
`_calibrated` ではなく `_raw` というステム名でエクスポートされます。これはファイル内のフラグではなく、
ファイル名そのものが異なるためです。これは、ファイル名がそのままの状態でメールで転送されても情報が失われないようにするためです。
`.csv`ヘッダーには`raw spectral sensor counts (NOT irradiance)`と記載されており、
値はファイル**内**でのみ比較可能であるという警告が表示されます。これはまさに、ターゲットベースのキャリブレーションが
目的であり、センサー間で比較されるものではない。電力依存の測光カラム（総電力、
明所視／暗所視ルクス、PPFD）は、カウント値から積分されるのではなく、**NULL**として返される。

バンドルをまったく取得できなかった DAQ-U / DAQ-M / DAQ-E は、**スキップ**され、
生データとして書き込まれることはありません。この場合、バンドルは存在しており、「再接続して再処理する」ことが有効な対処法となります。

レガシーな**v1.01 / v1.02**の記録（DAQ-A-SDがこれらを書き込みます）には、読み取りごとのエポック情報がなく、
ファイルの書き込み時刻のみが含まれています。image↔downwellingマッチャーは依然としてこれらを受け付けません — 書き込み時刻に対して
書き込み時刻との照合は、目に見えない形で誤りを生じかねないため）——しかし、エクスポーターはこれらを読み込み、
CSVには `clock=daq_created_on` と出力されるため、生成されたデータにはどのクロックが使用されているかが明記されます。

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

キャリブレーション・バンドルを取得できない記録（オフライン、またはファイルにキャリブレーション情報が
ないセンサー）は、**理由を明記して** `skipped` として報告されます。 生カウントを含む「校正済み」ファイルとして
書き出されることは決してありません。インターネットに接続して
再実行すると、エクスポートが完了します。

### 進行状況のコールバック

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### 実行後の要約とヒント

完了時、`process()`は`GET /api/processing-summary`を取得し、その本文を`result["summary"]`として添付します。この取得はベストエフォート方式であり、正常な返却をブロックすることは決してありません — 概要が利用できない場合、`process()`は通常の`{"status": "complete", "async": False}`形式にフォールバックします。`summary["hints"]`の各エントリ — 推奨される是正措置を含む完全な文、 例えば、実行結果がゼロ出力となった理由など — も、Python形式の`UserWarning`として再出力されるため、辞書を確認しなくても、ゼロ出力の実行は自己診断が可能です：

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` は機械可読な部分です：

| キー | 集計対象 |
| --- | --- |
| `models` | 実行内のカメラグループ数。 |
| `images_in_groups` | それらのグループにまたがるソース画像。 |
| `targets_found` | 検出された反射率ターゲット。 |
| `images_calibrated` | 実行で反射率プロダクトが生成された画像。 |
| `images_failed` | 処理に失敗し、エクスポートされなかった画像。 |
| `exported_files` | **実行で生成された画像プロダクトファイル**：** すべてのプロダクト（デベイヤー処理済み、プレビュー、ラディアンス、反射率、インデックス、ターゲットレンダリング）を、すべてのフォーマットで。 |
| `reflectance_files` | その中の反射率プロダクト (`Reflectance_Calibrated_Images`)は、ターゲットによるものであれ、光センサー（DAQ）の放射照度によるキャリブレーションによるものであれ、すべて含まれます。ターゲットがないにもかかわらず反射率データが書き込まれた実行については、ヒントとしてその旨が表示されます。反射率出力が欠落していると通知されるのは、反射率データを一切書き込まなかった実行のみです。 |
| `clamp_masks_written` | [アンミックス・クランプ・マスク](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) — これらは付随情報であり、出力データではないため、`exported_files`には含まれません（CLIでも同様に別個にカウントされます）。 |
| `captures_left_out` | この実行で取得されなかった、プロジェクトの `captures` フォルダ内の生フレーム。 実行は開始時に、それ以前にキャプチャされたデータを待機するため、実行中にキャプチャが保存されたか、待機時間が切れた場合を除き、この値は0になります。`summary["captures_left_out_note"]`には、これらに対する処理方法が記載されています。 |
| `daq_recordings_exported` / `daq_recordings_skipped` | 光センサーの記録。意図的に別々にカウントされています — これらは別の段階から得られたものであり、画像が全くない実行でも存在するため、これらを統合してしまうと、DAQのみの実行でも画像がエクスポートされたかのように見えてしまう。 |

これらに加えて：`summary["output_dirs"]`（書き込みが行われたすべてのディレクトリ）、
`summary["light_sensor_export"]`、`summary["stopped"]`（ユーザーが実行を中断した場合に
真となる。 そのため、部分的なカウント結果が、生産量が不足した完了済みの実行として認識されないようにするため）、および
`summary["groups"]`（グループごとの内訳）。

`exported_files`は、パイプラインが**書き込みを行う際**に記録されるものであり、
実行後にプロジェクトのイメージオブジェクトからスキャンされるものではない。並列およびGPU戦略は独自のイメージ
オブジェクトを構築するため（GPUパスではワーカーサブプロセス内で）、従来のスキャンでは
`0 file(s) written`を報告し、その後「ゼロエクスポート」のヒントを出力していました — すべてが正常に動作していた実行においてです。
この数値に基づいてスクリプトを作成している場合、正常な並列実行では現在、
ゼロ以外のカウントが報告されるようになりました。

スキップ報告では、リーダーが各ファイルに対して実際に特定した理由（
読み取り不可能なスキーマ、バンドルの欠落、書き込みエラーなど）が**重複排除**されて表示されるため、
1つの原因でスキップされた20個のファイルは、その原因が20回繰り返されるのではなく、1つの原因として扱われます。

> **`process()` は、実行で画像が生成されなかった場合には発生しません。** これは、SDKと
> CLIが意図的に異なる2つの点のうちの1つです。`chloros-cli process`は「製品が要求されたが、書き込まれたものは
> なかった」という状況を失敗とみなして非ゼロで終了するのに対し、SDKは通常通り終了し、
> 状況を報告します。パイプラインを空の実行時に停止させる必要がある場合は、
> ご自身で確認してください — `summary` （またはプロジェクトフォルダ内のファイル数を数える）など、
> 例外が発生していないことだけに頼らないでください。一般的な原因としては、入力フォルダが
> キャプチャとして認識されなかった場合や、設置されているカメラには適用できないとして製品がスキップされた場合（例：RGB専用
> カメラからのラディアンス）などが挙げられます。
>
> 実行のエクスポートを検証できなかった場合でも、`process()`は発生しません。代わりに警告が表示され、
> `result["export_check_failure"]`が設定されます。プロダクトは書き込まれています。この時点で停止させるには、そのキーを確認してください。 画像が
> 欠落した実行についても同様です。警告が表示され、`result["image_failure"]`が設定されます。`chloros-cli process`は、
> このフラグが設定されていると、非ゼロで終了します。

### 便利関数

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

各実行は独自の `with ChlorosLocal(...)` 内で実行されます。実行が開始したバックエンドは、実行が戻った後も実行を継続し、 次の呼び出しまで実行され続け、30分間何も使用されていない場合は自動的に停止します。いずれも `auto_start_backend`、`radiometric_output`、または `daq_log_path` を受け付けません。それらの場合は `ChlorosLocal`および`configure()`を使用してください。 `process_lattice_capture()`は、`camera`、`debayer`、または`ppk`を受け付けません。このオプションはパネル検出をスキップし、標準のデベイヤーを使用します。

### サポートされる値

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### 放射測定出力 (LATTICE マルチスペクトルパイプライン)

`process`パイプラインのLATTICEマルチスペクトル（M3C/M3M）エクスポートレベル — `reflectance`（デフォルト）、 `radiance`、`sensor-response`、または `all`（画像ごとに適用可能な各モード）— は、プロジェクトの **「放射輝度をエクスポート」** および **「反射率をエクスポート」** プロダクトを設定します：`radiance`は放射度のみをエクスポートし、`reflectance`は反射率のみをエクスポートし、`sensor-response`はどちらもエクスポートせず（デベイヤー処理済みおよびプレビュープロダクトのみ）、`all`は両方をエクスポートします。デベイヤー処理済みおよびプレビュープロダクトは、このレベルでは変更できません。`configure()`には、それ専用のキーワードがあります：

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

高度な回避策 — `custom_settings` を通じてプロダクトを記述する — は依然として機能しますが、これにより設定ブロック全体が上書きされることに注意してください （以下の警告を参照）。`"Export <product>"`キーを記述します：プロジェクト固有の`"Export radiance"` / `"Export reflectance"`設定が、古い`"Radiometric output"`キーよりも優先されるため、そのキーだけでは何も変更されません。

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` （デフォルト）は、カメラの放射輝度を、**タイムスタンプが一致するDAQのダウンウェルリング**で除算します。これは、記録された `.daq` (DAQ-U/M/E)**または、画像と共に検出された DAQ-M ネイティブの `.csv`**に基づいて、カメラごとの、あるいは DAQ ごとのキャリブレーションバンドルがローカルに存在しない場合は、初回使用時に**AWS から自動的に取得** されます。CLI では、これを- タイプ別のプロダクト切り替えオプションとして提供されます：`chloros-cli process`: `--radiance`/`--no-radiance`、`--reflectance`/`--no-reflectance`、`--debayered`、`--preview`。**どのカメラの測定値か。**キャプチャが保存する `.daq` は、それが記述するカメラの測定値であり — 隣接する `<file>.daq.cameras.json` サイドカーに名前が記載されている （シングル、同期、および配列キャプチャ；RAWバーストの場合は、マニフェスト内で各メンバーの読み取り値が指定される）— であり、**それらのカメラからの画像にのみ**対応付けられます。したがって、独自の読み取り値を持たないスタンドアロンカメラは、他のカメラのDAQを借用できなくなります。DAQをそのカメラにバインドするか、DAQ自身の録画 （`chloros-cli daq pool-record`、またはGUIで開始された記録）、あるいはそのサイドカーを持たない任意の`.daq`（古いキャプチャを含む）は、引き続きすべてのカメラに対応します。**日光下のOCNカメラ。** すべてのOCNキャリブレーションパックは、狭帯域光（帯域ごとに1つのLED）の下で構築されていますが、太陽光では各帯域が混ざり合います。Chloros 1.3.0では、帯域の分離後に一斉に太陽光ベースの補正を適用し、これにより太陽光下でのOCNの放射輝度、反射率、および屈折率の算出結果が改善されます。 この補正は、バンド分離が行われる場所（`process()`、キャプチャ、`grab()` / `frame_stream()`、およびライブビュー）であればどこでも実行され、対応するOCNパックで有効になります。RGNカメラについては変更ありません。 以前のバージョンで撮影されたOCN画像も、撮影時に使用されたキャリブレーションが保存されているコンピュータで処理される場合（Chlorosはカメラが一度接続されるとそのキャリブレーションを保持します）、この補正が適用されます。それ以外の場合は、従来通り処理されます。したがって、OCNの放射輝度、反射率、および屈折率の値は、バージョン1.2.0のものとは異なります。

> `custom_settings` は、計算された設定ブロック全体を**置き換えます** （設計上、`configure()`の他のキーワードや検証をバイパスします）。これを使用する際は、上記の例のように、必要なすべての`Project Settings`キーを含めてください。

#### インデックスピクセルの読み取り （NDVIおよびその他の植生指数）

**整数インデックスラスタは、画像ごとのコントラスト伸張であり、−1…1のマップではありません。** `TIFF (16-bit)`、`TIFF (8-bit)`、 `PNG (8-bit)`、および `JPG (8-bit)` といったインデックス製品（8ビットキャプチャとして書き出される8ビットTIFFを含む）は、

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

ここで、`IndexMin` / `IndexMax` は **そのフレーム固有の最小および最大の有限インデックス値** です。 DN 0 は −1 ではなく、フルスケールは +1 ではありません。

Chlorosは、マッピングが可逆性を保つように両端点にスタンプを付けます：

| タグ | 意味 |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — 以下の規約 |
| `Chloros:IndexMin` | DN 0 として格納されたインデックス値 |
| `Chloros:IndexMax` | フルスケールとして格納されたインデックス値 |

これらを読み取り、逆変換を行い、保存された DN を、保存されたデータ型の最大値で 0…1 の範囲に正規化します。これにより、1 つの式ですべての整数形式を網羅できます：

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

読み取りには `-config pix4d.config` は不要です。exiftool はこれらの XMP プロパティを単独で読み取ります。この設定は、それらを *書き込む* 場合にのみ必要です。

> **`IndexMin == IndexMax`** は、そのフレームのすべてのピクセルが同じインデックス値を持っていたことを意味します。 ラスタは中間スケールで埋められており、除算する対象がないため、端点値を採用します。

> **`Chloros:Index*`タグがない場合は、明示的なマッピングがないことを意味します。** LUTエクスポートとは、ピクセルがLUTのしきい値ウィンドウに基づいてグラデーション化されたカラーレンダリングであり、有限のインデックス値を持つピクセルがないフレームには、指定すべき範囲が存在しません。いずれの場合もマッピングがあると仮定しないでください。これは、`PixelScale`が欠落している場合と同じルールです。**ストレッチ処理は画像ごとに適用されるため、1回のフライトの2つのフレームをDNとして比較することはできません**— 各フレームは独自の端点を持つため、同じNDVI値でも画像によって異なるDNに割り当てられます。フライト全体にわたる定量的な作業（ゾーン統計、固定閾値、時系列分析）を行う場合は、**`TIFF (32-bit, Percent)`**をエクスポートしてください。インデックスラスタは、リスケールやタグを必要とせず、**インデックス値そのもの**がfloat32としてそこに書き込まれます。フォーマット名とは異なり、インデックス製品は×100ではありません。そのスケーリングは整数データ型の反射率にのみ適用されます。

#### アンミックス・クランプ・マスク — どのピクセルでバンドがクランプされたか

3×3スペクトルアンミックスにより、バンドが負の値になる場合があります。これは、光にそのバンドがほとんど含まれていない場合や、シーンがカメラのアンミックスマトリックスで記述される範囲外にある場合です。これは実際の値ではないため、エクスポートでは0に固定されます — ライブビューにも同じ 0 が表示されます。バンドは数ピクセルでクランプされることもあれば、ほぼすべてのピクセルでクランプされることもあり、マスクにはそのいずれかが記録されます。`Chloros:Band{R,G,B}UnmixClampFraction` は、フレームのどの程度の範囲でそれが発生したかを示し、クランプマスクは**どのピクセル**で発生したかを示します：

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— 製品のフォルダ**の隣**にある `clamp_masks/` フォルダ内にあり、 決してその内部には置かない（`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`） — 少なくとも1つのピクセルでクリップが発生したすべての放射輝度、反射率、および屈折率プロダクトについて。 シングルバンドの `uint8` については、そのプロダクト独自のピクセルグリッド（オリエンテーション、 ROI、ビニング、および配列位置合わせのワープを含む）。各マスクには、そのプロダクトのカメラ（`Make` / `Model` / 連番）、撮影時刻、レンズタグが記載されており、そのビットが表すバンドがリストされています （`XMP-Camera:BandName`、ビット0が先頭）。`ImageDescription`および`Software`は、これをクランプマスクとして識別し、キャリブレーション情報は含まれていません。
- **ビット**：ビット 0 = 製品の R チャンネルのバンド (`Chloros:BandRName`)、ビット 1 = G (`BandGName`)、ビット 2 = B （`BandBName`）。ビットがセットされている場合、そのバンドにおける当該ピクセルの値が 0 にクランプされたことを意味します。このピクセルおよびそこから構築されたインデックスは、信頼度が低いものとして扱ってください。
- **`Chloros:UnmixClampMask`** は、アンミックス処理が実行されたすべての製品に付与されます。これは、製品自身のフォルダを基準としたマスクのパス（`../clamp_masks/<product folder>/<product name>_clampmask.tif`、常に `/`で区切られています — 画像のフォルダにこのパスを追加するとマスクが開きます）、またはピクセルがクリップされていない場合は `none` となります。タグがない場合は、その製品でアンミックス処理が実行されていないことを意味します（RGB またはモノクロカメラ、RAW/デベイヤー処理済みレベル）。
- キャプチャ結果では、フレームの **`clamp_mask`** として扱われます（マスクのパス、またはクリップされていない場合は `None`） — `CameraSession.capture()`の結果では、`ArraySession.capture()`'`CaptureResult`の各フレームおよびHTTPキャプチャ応答に含まれます。オフラインビルドでは、マスクは`clamp_masks`として一覧表示されます。
- ライブインデックスのマルチバンドスタック（`/api/camera/cli-array-index` と `save_multiband`）は、ソースカメラごとに 1 つのマスクを取得し、出力フォルダ内の `clamp_masks/` に保存されます。詳細は、CLIリファレンスのの `lattice index` を参照）。動画にはマスクが存在せず、代わりにオフラインビルドの結果では各フレームのクランプ比率が報告されます。

どの階層においてもマスクを格納するプロダクトフォルダは存在しないため、プロダクトフォルダはサブフォルダを含めたままそのままフォトグラメトリーソフトウェアに渡すことができ、ソフトウェアは画像のみを取り込みます。プロダクトフォルダそのものを渡してください。 その上のフォルダではなく、プロダクトフォルダそのものを渡してください。Chloros独自のインポート機能では、マスクはスキップされます。

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## LATTICEカメラ用Smart-Connect

ライブハードウェア用の永続的なバックエンドセッション。GUIが使用するエンドポイントと同じであるため、SDK / CLI / GUIを問わず動作は同一です。

### 単一カメラ — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

ここで設定された露出、ゲイン、ターゲットはユーザーの設定として保持されます。Chlorosの自動露出制御（DAQによる露出測定を含む）は、これらを変更しません。`cam.set_settings(exposure_auto="Continuous")`（または`gain_auto=`）は露出（ゲイン）を返却し、 `cam.set_settings(target_brightness_auto=True)`はターゲットをChlorosの自動ターゲット設定に戻します。

#### `connect_camera()` 署名

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **ストリームの開始とともに戻ります。** バックエンドは、制限時間内に待機します（ストリームの開始まで約15秒、その後最初のバッファが完全に埋まるまで10秒 — 約1.5秒後に続く最初のプレビューフレームまでの時間は含まれません）。 そのため、その直後に `status()` が表示され、続いて `streaming` が表示されます。このセッションの `connect_stream` は、応答の `stream` ブロックです： `state` `streaming`、`no_frames`（ストリームは開始されたが何も送信されなかった — `note`がそれを示しており、ジャンボパケットを使用しているカメラについては、1500バイトのパケットサイズで再接続する）、`starting`または`start_failed`（`waited_s`。すでに接続されていたカメラの場合は、`None`。プレビュー （`get_camera_preview`、`/api/camera/jpeg/<serial>`）は、応答する代わりに、カメラの最初のプレビューフレームが送信されるまで（4秒間）待機します。 「まだ利用可能なフレームがありません」と応答する代わりに、カメラの最初のプレビューフレームが到着するまで（4秒間）待機する。一方、ストリームから何も送信されていないカメラ（`no_frames`、またはハードウェアトリガー用に武装されているもの）は、依然として即座に応答する。
- **パケットサイズは任意に設定可能です。** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}`は1500バイトのパケットでカメラを開きますが、プローブは行われません（`chloros-cli lattice cam-connect --packet-size standard`も同様に送信します）。
- **一度に1つの接続のみがカメラプールを占有する**：接続の確立、パケットサイズのプローブ、最初のキャリブレーションデータのダウンロード（このコンピュータで初めて使用するカメラの場合、約70秒）、ストリームの開始。その間にカメラを必要とするリクエスト（`read_nodes`、 `set_settings`、キャプチャ、2回目の接続）は、最大5秒間待機した後、HTTP 503エラー（`CAMERA_POOL_BUSY`）で失敗し、接続中のカメラ名を通知します（`ChlorosConnectError`）。 しばらくしてから再度リクエストしてください。一覧表示は決して待機しません：`list_cameras()`（`GET /api/camera/pool`）、`list_arrays()`、および`CameraSession.status()`は即座に応答します。 接続中のカメラは、プール応答の `connecting` の下にリストされます （`serial`、`phase` — `opening the camera`、`rebalancing the pool bandwidth`、 `loading its calibration`、`starting its stream`、`waiting for its stream` — および `elapsed_s`)、その`status()`は`connecting: True`を、その`phase`は`connecting: True`を返します； その他のカメラは、プールが空き状態のときはリアルタイムで読み取られ、そうでない場合は前回の読み取り値（`live: False`）が返されます。

#### `CameraSession` メソッド

| メソッド | 説明 |
| --- | --- |
| `status(timeout=15.0)` | `chloros-cli lattice status` が読み取るカメラのライブ状態： `streaming`、`stream_state`（`starting`、 `no_frames`、`streaming`、`start_failed` または `stopped`）、`device`（モデル、シリアル番号、ファームウェア、温度、`is_mono`）およびその現在の設定を読み取ります。 |
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICamノードを読み取ります。自動露出実行中に、`{nodes, errors, enums, device, was_in_pool, in_array_id}`、 さらに `frame_values` および `notes` を返します（自動露出実行中、詳細は後述）。 |
| `set_settings(**kwargs)` | フレンドリー名でノードを書き込みます（`exposure_time`、`gain`、 `pixel_format`、`width`、`height`、 `target_brightness`、`target_brightness_auto`、`ae_damping`、`ae_upper_limit`、`trigger_mode`、 `trigger_source`, …）。`exposure_time`（`gain`）は手動露出（ゲイン）であり、`exposure_auto` (`gain_auto`)を省略して記述すると、まずオートモードがオフに切り替えられるため、動作中のオート機能によって値が拒否されることはありません。その結果、`applied`にはカメラが取得した値が保持されます。 カメラが拒否した書き込みは `errors` に格納され、保持されません。エラーメッセージには、カメラが現在受け入れる範囲が示されます（`ExposureTime` の最大値はカメラのフレームレートに準じます）。 また、拒否された値に対してのみオフにされていた自動モードは元に戻されます； `restored` にはその一覧が記載されています（例：`{"ExposureAuto": "Continuous"}`）。 |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | **1フレーム**のみをキャプチャします フレームをキャプチャする。`CaptureResult`を返す：フレームメタデータ辞書のリスト（保存されたレベルごとに1つずつ）で、`.skipped`、`.failed`、 `.daq_files`、および `.daq_matches` が含まれます。（バースト／マルチフレームキャプチャ機能は削除されました。一連の画像が必要な場合は、ループ内で `capture()` を呼び出してください。） |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | このカメラからの連続（`interval_s=0`）または間隔指定キャプチャ。 `count`によるキャプチャおよび／または`duration_s`（少なくとも1回実行されれば終了）に対して、`ArraySession.capture_repeated()`が配列に対して行うのと同様に実行されます。 `CaptureResult`のリストを返します。ドライブが満杯であると報告するキャプチャの後で停止します。`on_capture(i, result)`は各キャプチャの後に実行され、その他のキーワードは`capture()`に送られます。 |
| `disconnect(keep_settings=False, clear_settings=False)` | プールから解放し、カメラを接続時に検出された状態（トリガー配線、PTP、設定）に戻す; `keep_settings=True`は代わりに設定を自身のメモリに保存し、`clear_settings=True`はそのメモリをリセットする（ [カメラメモリ](#camera-memory--keep_settings--clear_settings))を参照); いずれかが要求された際に `camera_memory` を返す。配列の要素であるカメラは、配列全体を解放する。すでに開かれているセッションに接続した場合は何もしない。 バックエンドが解放を確認した時点、または解放すべき対象がない場合（カメラがすでに存在しない、あるいはバックエンドが実行されていない場合）に返却される。タイムアウトした、またはバックエンドによって拒否された解放操作は、カメラがまだ保持されている可能性があり、次の接続で競合が発生する恐れがあるため、`ChlorosConnectError`を発生させます。`with`ブロックの終了時には、例外は発生しません。 |

属性：`serial`、`device`（カメラの検出レコード）、および`backend_url`。

`capture()` エクスポート制御（配列＋GUIと同じモデル）：

- `processing` / `levels` — `processing="all"` は適用可能なすべてのエクスポートタイプを保存します。`levels=["raw","radiance"]` はそれらのみ保存します（`processing` を上書きします）。バックエンドのデフォルト設定にする場合は、両方を省略してください。
- `daq` — 使用されたキャプチャを読み取るDAQは、デフォルトで`.daq`サイドカーとして保存されます。これには生データのみのグラブが含まれており、後でフレームを反射率/屈折率データに再処理することが可能です。 `daq=False` は、どのレベルでも使用されていない場合、これをスキップします。`force_daq=True` は、デフォルト設定の旧表記です。DAQ なし、 `.daq`も使用されません。
- `daq_tolerance_ms` — 露光からどの程度離れていても許容範囲内とみなされるか（デフォルトは500）。判定のみが変更されます。

#### 自動露出時の `read_nodes()` — `frame_values` および `notes`

`ExposureAuto` (`GainAuto`) の実行中、`ExposureTime` (`Gain`) ノードは、カメラの現在の動作状況ではなく、**最後に書き込まれた値**を読み戻します。`nodes` はそのノードの値を保持したままになります。 自動モードが `Off` ではない接続済みカメラに対して `ExposureTime` または `Gain` を要求すると、応答には以下が追加されます：

- `frame_values` — カメラの最新のフレームが保持していた値を、ノード名の下に表示（`ExposureTime`はµs単位、`Gain`はdB単位）；
- `notes` — 該当するノードごとに、両方の値を記載した1つの文。

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

自動モードが `Off` の場合、両方のキーは存在しません（この場合、ノードはカメラの実値）となるほか、バックエンドに接続されていないカメラの場合も同様です。配列の場合、`ArraySession.status()["member_ae"]`はすべての要素のフレーム値を一度に返します。

#### DAQによる露出上限（HTTP）

DAQ光センサーが接続されている場合、Chlorosは測定された日照量に基づいてカメラの露出×ゲインに上限を設定し、白い基準パネルがクリップしないようにします。 この上限は、キャリブレーションターゲットがそのカメラの視野内にある間（過去10秒以内にそのフレーム内に認識された場合）にのみ適用されます。視野内にターゲットがない場合は、カメラ独自のハイライト保護機能を備えた通常の自動露出が代わりに動作するため、シーンの他の部分が必要以上に暗く抑えられることはありません。 SDKラッパーは存在しません。`GET` / `POST /api/camera/<serial>/daq-exposure-cap` を使用してカメラごとに設定を読み取ったり切り替えたりしてください（POSTボディの `{"enabled": false}` を指定すると、そのカメラでこの機能がオフになります）。 応答には、`enabled`、`cap_us`（ライブ上限値。`None`は非アクティブ時、およびキャリブレーションターゲットが視野内に入っていない場合を含む）、 `k_exp_g`（上限値の算出に使用される定数）、および：

| フィールド | 意味 |
| --- | --- |
| `k_exp_g_source` | 使用中の定数：`camera` — このカメラ自身のもの（そのキャリブレーションパックのラボポイントから計算されたもの （RGBカメラの場合）；`fleet` — パックにラボポイントがない場合に使用されるフリート定数；`override` — バックエンドで設定されたフィールドオーバーライド。 |
| `k_exp_g_fleet` | 比較用のフリート定数。 |

### 同期アレイ — `ArraySession` (Smart-Prep)

`connect_array`は、マルチカメラ構成における**推奨のエントリーポイント**です。この製品は、内部でGUIベースのSmart-Prepフローを完全に実行します：

1. **ネットワーク分析**(`/api/camera/array/recommend`) —**センサー全体** （ビニングなし、ROI クロップなし）を設定し、Connect が使用するピクセルフォーマットおよびパッキング形式に基づいて、このホストとネットワークが提供できる範囲内でフレームレートを調整します。フレームレート維持のためにフレームが縮小されることは決してありません。解像度を犠牲にしてフレームレートを向上させるには、より小さい `width` / `height` または `binning=2` / `4` を指定して、解像度を犠牲にしてフレームレートを優先してください（エンドポイントの`alternatives` では、これらのオプションと予測されるfpsがリストされています）。
2. **ティア自動選択** — `sim-capture-sim-emit`（すべてのカメラが同じトリガーで露光し、ワイヤを共有するようにペース調整され、共有幅が狭まるにつれてフレームレートが低下する）は、カメラのペース調整によってワイヤを共有できるあらゆるネットワークで適用可能； `slip-emit-and-capture`（カメラは通信回線を順番に使用し、同時に露出を行わない）は、それが不可能な場合にのみ適用されます。
3. **PTP有効**（デフォルト） — カメラ間のタイムスタンプは、**約1 ms**の精度で1つの共有クロックに同期し、バックエンドの[PTPグランドマスター](#ptp-grandmaster--time_sync_status)。同時露光は、PTPではなくM8ハードウェアトリガー（**< 100 µs**のモジュール間）によって実現されます。PTPは*タイムスタンプ*を同期させるものであり、露光を同期させるものではありません。
4. **カメラごとのピクセルフォーマットの自動選択** — モノクロ → `Mono12`、カラー → `BayerRG12`。
5. **AEの初期化** — 各カメラの現在のAE状態をスナップショットとして保存し、接続中に露出設定がリセットされないようにします。
6. **GPIOトリガー設定** — `connect_array`で各カメラ（`TriggerMode=On`、`TriggerSource=Line2`）をアーム状態にすることで、マスターのパルスがM8ケーブルを介してスレーブを駆動するようにします。これはアレイ専用手順です。`LatticeCamera`で単一のカメラを開いた場合は、代わりにフリーラン動作になります。アレイを解放すると（`disconnect()` / `force_disconnect()`、`chloros-cli lattice array-disconnect`）を実行すると、すべてのカメラがフリーラン状態（`TriggerMode=Off`）に戻り、それ以外の設定は接続時に検出された状態（トリガーおよび同期ラインの設定 （`TriggerSource`、`TriggerActivation`、`TriggerOverlap`、`TriggerDelay`、 `LineMode` / `LineSource`；マスターのシンクロラインは入力に戻す）、PTP、およびセッション中に変更されたカメラ設定 — ただし、`keep_settings=True` / `clear_settings=True` の場合を除き（[カメラのメモリ](#camera-memory--keep_settings--clear_settings)を参照）。

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

属性：`array_id`、`master_serial`、`slave_serials`、`ptp_enabled`、 `sync_mode`、`packed_transport`、および`packed_reason` （最後の4つは、`attach_array`からのハンドル上の`None`）、`connect_stream`、および`backend_url`。 `connect_stream` は、接続応答の `stream` ブロックであり、単一の接続の形状に `members` (`{serial: {state, why?}}`)：配列の最初の同期済みグループが到着すると（バックエンドはこれを、上限約10秒の間待機する）、`streaming`、 あるいは、各メンバーの最後のフェッチ失敗を `why` として持つ `no_frames`、およびジャンボパケットの場合は、 `note`内の1500バイトのフォールバック。`None`は`attach_array`ハンドル上で発生します。

#### `connect_array()` シグネチャ

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` は、呼び出しが接続を待機する時間であり、これは
バックエンドに対して期限として送信されます。 その時点で接続が完了していない場合、その操作は取り消されます。つまり、
開かれていたすべてのカメラは検出時の状態に戻され、閉じられます。配列は接続されず、
呼び出しは `ChlorosConnectError` （HTTP 504 `CLIENT_DEADLINE_PASSED`、これは
接続が到達していた段階を示す）を発生させます。したがって、例外を発生させた呼び出しは、その背後に
接続された配列を残すことはありません。9台のカメラからなる配列の接続には約1分かかりますが、
カメラの電源投入直後は数分かかる場合があります。起動はしているものの、まだ
同期されたフレームグループを持たない配列は、`connect_stream["state"]`
`no_frames`が返されます。バックエンドのログには、接続ステップごとに1行のタイミング情報
（`[array-connect <id>] <step>: <s> s …`）と、最も時間がかかったステップの名前を示す最後の行が記録されます。

`force_tier`の値：
- `"sim-capture-sim-emit"` — 真の同時性（すべてのカメラが同じクロックエッジで発光する）。
- `"sim-capture-ftd-stagger"` — 柔軟な時間領域での時差（カムがわずかにずれたタイミングで信号を出力するため、ワイヤ上でパケットが直列化される）。
- `"slip-emit-and-capture"` — カムごとの順次キャプチャ （時間的な同期なし）。カメラの動作速度を十分に遅くして回線を同時に共有できない場合にのみ、自動的に選択されます。

`wire_ceiling_mbps` は、**ホストの持続的な回線帯域幅**（MB/s）を上書きします。これは、アレイ全体の割り当てが依存する唯一の
数値です。 `None` のままにしておくと、自動検出された
値が使用されます。アレイが GVSP 破損フレームを報告した場合は、この値を下げてください： 自動設定値は
NICが通知するリンクレートに基づいて算出されますが、このレートはUSBアダプタ、帯域の狭いPCIeレーン、
および負荷の高い共有ファブリックにおいて過大評価されがちです。この過大評価は、
目に見えるリンク速度の低下ではなく、破損フレームとして現れます。 この値はプロジェクトのアレイキャプチャブロックに永続化されるため、
プロジェクトを再開したり、後で `connect_array` を実行したりすると、他のアレイ設定と同様に復元されます。
[アレイの健全性](#array-health--which-subsystem-is-losing-frames) を参照してください。

#### パックされた12ビット転送 (`packed=`)

センサーがフル稼働している場合、アレイのフレームレートは、各カメラが
フレームを送信できる速度によって決まります。12ビット形式（`BayerRG12`、 `Mono12`）は通常、16ビット
ワードとして転送されますが、パックトランスポートでは、同じ12ビットを1ピクセルあたり1.5バイトで送信し、
ホスト側で展開するため、各フレームの送信時間が25%短縮されます — フルセンサー稼働時で約
**センサーフル稼働時で毎秒**33％多くのフレーム**（測定値：9台のアレイで10.7 → 14.0 fps）**。**保存されるデータは同一**です：値も、16ビットの
ファイルも、メタデータもすべて同じです。

- `packed="auto"`（デフォルト）は、圧縮が有益であり、かつホストが
  展開処理に十分な処理能力を持っている場合にのみ圧縮を行います。接続時に、バックエンドは自身の展開速度、
  CPUコア数、現在の負荷、およびネットワーク帯域幅を測定します。 余剰コアを持つデスクトップ
  では、センサーのフルレートで圧縮処理を行います。一方、負荷が高いホストや低消費電力のホスト（シングルボードコンピュータ、
  組み込みコントローラ）では、圧縮解除済みのデータをストリーミングします。小さなROIやビン処理されたフレームでは、圧縮によるメリットはほとんどありません。
- `packed="off"`（または `False`）は決して圧縮を行いません。これは、診断中に圧縮を無効にする場合や、
  他の作業のために CPU リソースを確保しなければならないホストで使用します。
- `packed="on"`（または `True`）は、12 ビット形式を常にパックします。
- それ以外の値の場合、`ValueError`が発生します。8ビットおよび16ビット形式は決して
  パックされません。

`analyze_array_network(..., packed="auto")`も同様の判断を行うため、
そのfpsは配列が実行されるレートとなります（応答には
`packed_transport`および`packed_reason`が含まれます）。 その `pixel_format` は、8 ビットではなく、
connect が適用する値（Mono12 / BayerRG12）がデフォルトとなります。

決定とその理由はセッション上にあります：

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)`は、プロジェクト内のすべての
配列に対して同じ上書きを適用します（デフォルトは`None`であり、これは`"auto"`を意味します：プロジェクトは配列ごとの
パック設定を保存しません）。
マシン全体のデフォルト設定にするには、バックエンドの環境で `CHLOROS_ARRAY_PACKED12=off`（または `on`）を設定してください。
ただし、 明示的に指定された `packed="on"` / `"off"` が優先されます。
`/api/camera/array/status` は、`packed_transport` および `packed_decision`を報告します。

#### 予想フレームレート

測定されたアレイのフレームレート（ハードウェア同期、12ビット；センサー全解像度 = 2048 × 1536）。 これらを使用して `status()["actual_fps"]` の妥当性を確認してください。`analyze_array_network()` では、お使いのホストおよびネットワークにおけるフレームレートを予測しています。

| 設定 | アレイのフレームレート |
| --- | --- |
| フルセンサー、PCIe 10 GbE ホスト NIC、パックトランスポート（デスクトップ上の `packed="auto"`） | 約 14 fps |
| フルセンサー、PCIe 10 GbE ホスト NIC、アンパック (`packed="off"`) | 約 10.5 fps |
| フルセンサー、USB 10 GbE アダプタに接続された 8 台のカメラ | 約 3.8 fps |
| フルセンサー、1 GbEホストリンク | 1.2～1.7 fps |
| 2×2ビニング（`binning=2`）、10 GbEホストNIC | 約30～33 fps |

- PCIe 10 GbE リンクでは、フルセンサー時における制限要因はネットワークではなく**カメラ**側です。4台および8台のカメラは同じレートで動作します。
- **`target_fps`はトリガーレートであり、アレイの動作速度がこれを超えることはありません。** `target_fps=1`ではアレイは1 fpsで動作し、`None`（デフォルト）ではホストとネットワークが処理可能な最高レートで動作します。そのレートを超えるピンは、そのレートまで引き下げられます （[`set_target_fps`](#interval-capture-for-drone-surveys)を参照）。
- **`link_speed_mbps` から `4294` までの値は、リンクレートではありません。** これは、Windows がアダプタの速度を読み取れなかった（ドライバが最大の 32 ビット値を報告した）ことを意味し、USB イーサネットアダプタではよくある現象です。Windows における USB アダプタの使用可能な転送速度は、いずれの場合も 200 MB/s に制限されています（[Array Health](#array-health--which-subsystem-is-losing-frames])を参照してください。調整方法については、CLIリファレンスの[ホストNICの設定と調整](cli-reference.md#host-nic-setup--tuning-lattice-arrays)にある「USB 10GbEアダプタ」のセクションを参照してください。
- **Linux でも USB の上限が適用されます：x86_64 では 400 MB/s、NVIDIA Jetson では 200 MB/s です。** アダプタは sysfs から識別される（識別できない場合は `ethtool -i` にフォールバックするため）、USB 5 GbE または 10 GbE アダプタは、その定格値である 500 / ~1060 MB/s ではなく、この上限値に基づいて処理される。 x86_64の数値はLinux上で測定されたものです。Windowsでは200 MB/sを超えるとフレームが破損する同じアダプタが、Linuxでは480 MB/sまでのあらゆる上限速度において、4台のカメラアレイを問題なく動作させました。 NVIDIA Jetson では、同じアダプタは 200 MB/s でのみ正常に動作し、それを超えると断続的にフレームが失われました。`gvsp_corrupt` フレームなしでホストがそれ以上のデータを処理できる場合は、 `wire_ceiling_mbps`で上限を引き上げてください。Linuxでは、USB 2.0ポートに接続されたアダプターの帯域幅は30 MB/sに制限されています。USB 3ポートに接続してください。

#### オーバーサブスクリプション（カメラごとの下限）

アレイはどのネットワークでも動作しますが、フレームレートはネットワークの性能に左右されます。 Sim-emit ペーシングでは、各カメラに衝突防止のための帯域幅を均等に割り当て、すべてのカメラが同じトリガーで出力を生成するため、低速な回線では割り当て量（およびフレームレート）が小さくなります。8台のフル解像度の12ビットカメラ8台を、ジャンボフレームを使用しない1 GbEで接続した場合、各カメラは6.5 MB/s、つまり約**1 fps**（パック処理時は約1.3 fps）となります。割り当て量は、カメラのペーシングが可能となる最低レートで下限が設定されます （`per_cam_floor_bps`：接続時に各カメラのGevSCPD範囲から読み取り、1 MB/sを下回ることはありません）。 回線側の要件によりそれより低速な転送が必要となる場合、接続は**シリアル**送信（`slip-emit-and-capture`）にフォールバックします。カメラはフルリンクレートで順番に送信を行うため、すべてのフレームは到着しますが、メンバーは**同時に公開されません**— レスポンスのティアおよびブロッカー (`SERIAL: ...`) がその旨を示します。フォールバックを拒否するには、`force_tier="sim-capture-sim-emit"` を渡してください。フレームサイズは共有の計算には含まれないため、**ビニングやROIはフレームレートを向上させますが、アレイが収まるかどうかに影響しません**。いかなるペース（同時または順番）でも配列を伝送できない回線のみが**オーバーサブスクライブ**状態となる — 障害モードはGVSPパケット損失であり、フレームレートの低下ではない。対策：カメラ数の削減、エンドツーエンドでのジャンボフレームの使用、またはより高速なNICの導入。

- `analyze_array_network()` および `/api/camera/array/connect` の応答には、`oversubscribed`、`aggregate_demand_bps`、`collision_safe_ceiling_bps`、`max_cams_collision_safe`、および `per_cam_floor_bps` を含みます。`oversubscribed` が true の場合、プロジェクションは **fps フィールドをゼロに設定**します（`achievable_fps_max` / `fps_bright` / `fps_dark`）**fpsフィールドをゼロに設定**し、誤解を招くような「遅いものの動作している」レートを報告することはありません。
- `POST /api/camera/array/connect` は `pin_resolution` ボディパラメータを受け付けます（**HTTP のみ — SDK の kwarg ではありません**; `connect_array` はこのパラメータを公開していません）。これは厳格モードです：`pin_resolution` が設定された状態でオーバーサブスクライブされた接続は、すべての是正策を明記したエラーと共に**厳格に拒否** されます。 これを設定しない場合、接続は配列でパケットが損失する可能性があるという警告を表示して続行されます。いずれの場合も、接続は要求されたフレームサイズを維持します — 独自にビニングやクロッピングを行うことはありません。
- テスト目的のみ： バックエンドの環境で `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` を設定すると、拒否処理が「明確な警告」に緩和されます。この場合、接続は実行されますが、パケット損失を受け入れることになります。

#### アレイの健全性 — どのサブシステムがフレームを損失しているか

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) は、接続済みのアレイ上のアクティブな `health` ブロックを
保持しており、**10 秒**ごとのローリングウィンドウで再評価されます ウィンドウで再評価されます。これは、フレーム損失を
単一の「不完全」レートとして扱うのではなく、
それぞれ異なる対処を必要とする2つの原因に分類します：

| 項目 | 意味 | 対象サブシステム |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (シリアルごと) | フレームが**到着したが構造的に不正だった**— GVSPパケット損失。 |**ネットワーク**：ワイヤ・バジェット、ペーシング、NIC RXリング、MTU |
| `never_arrived_rate_pct`（シリアル番号ごと） | フレームが**まったく到着しなかった**— カメラが起動しなかったか、何も送信されなかった。 |**トリガー／同期**: M8ケーブル、`line=`、`TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 各カメラの最低転送レート 各カメラごとの最悪のレート。 | — |
| `per_cam_rate_pct` | カメラごとの不完全率の合計（両方の原因を合わせたもの）。 | — |
| `stable_for_seconds` | 各カメラが0.01％未満の状態にあった期間。 | — |

`health` と併せて、同じレコードには、割り当て全体が保留されている数も報告されています：

| フィールド | 意味 |
| --- | --- |
| `wire_ceiling_mbps` | ホストの有効な持続ワイヤ帯域幅（MB/s）。 |
| `wire_ceiling_source` | その数値の出典（文言） — 例：`USB-capped 200 MB/s (was theoretical 1062; …)` または `user override 120 MB/s (auto said 200)`。 |
| `wire_ceiling_is_user_set` | `true` が `wire_ceiling_mbps=` を設定した際の値。 |
| `nic_is_usb` | USBイーサネットアダプタの場合は `true`。 |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**読み方：** `gvsp_corrupt_rate_pct` が 0 以外で、`never_arrived_rate_pct` が 0 の場合、
トリガーと同期ケーブルの状態は完璧であり、損失の 100% がネットワーク経路に起因していることを意味します。この値を下げ
`wire_ceiling_mbps`を下げて再接続してください。逆のパターンは、同期ケーブルまたは
トリガーラインに問題があることを示しています。

> **`target_fps`は、破損フレームの要因ではありません。** GevSCPDのペーシングは
> 接続時に一度だけ設定されるため、トリガーレートを下げてもデューティサイクルは変化しますが、
> 同時送信バーストレートは変化しません。測定上、要求量を5倍削減しても改善は見られませんでしたが、
> ワイヤのシーリングを 240 MB/s から 200 MB/s に引き下げたところ、同じリグで破損率が 10.4 % から
> 0.00 % へと低下しました。

> **稼働中のアレイは、LATTICE (M3C / M3M) ファームウェア上では自身で再計画を行うことはできません。** これについては
> 自身で修正することはできません。接続を一度切断して再接続し、接続時間ピッカーが
> 新しい上限値に基づいて再プランニングを行うようにしてください。

**USBイーサネットアダプタは、その
銘板の記載にかかわらず、**USBイーサネットアダプタは200 MB/sに制限されます**。リンク速度を持続転送速度に変換する効率表は
PCIeに基づいており、USB NICはイーサネットのリンク速度を広告しますが、実際には
USBバスとそのドライバによって制限されています。 この上限は相対的なものではなく絶対的なものです。つまり、USB 1 GbE アダプタは
約 80 MB/s を実現しますが、この制限の影響を受けません。この制限は Windows および Linux（Jetson を含む）に適用されます。
測定結果については、[予想フレームレート](#expected-frame-rates) を参照してください。

#### `ArraySession` メソッド

| メソッド | 説明 |
| --- | --- |
| `status(timeout=10.0)` | 実行中のアレイのリアルタイム状態：`target_fps` / `target_fps_auto`（トリガーレートおよびレートコントローラがこれを所有しているかどうか）、`target_fps_clamped_reason`（固定レートが低く抑えられている理由、それ以外の場合は`None`）、 `actual_fps`（測定値；グラブループが放棄またはストールした時点で0）、`recommended_fps`、`streaming` （グラブループが放棄すると False になる — `last_error` だけでなく、これを確認すること； 設定の再起動後もTrueのままとなり、これを`restarting`がフラグとして示す）、`stalled`（グラブループが実行中であるが、約10フレーム期間（2～30秒）にわたってフレームグループをパブリッシュしていない間、Trueとなる； `last_error`は60秒経過後にのみこれを示す）、`last_frame_age_s`、`last_error`、`serials`、 `ptp_enabled`（`True`は、配列がPTPで接続されていた場合のみ **かつ** すべてのメンバーが最後に `PtpEnable` を読み取った状態；これは `ptp_epoch` が測定する「ロック済み」状態とは異なる）、`ptp_configured`（接続要求で指定された状態）、`ptp_off_members` / `ptp_unread_members`（最後に `PtpEnable` をオフと読み取った／応答しなかったメンバー）、`member_recovery` （`None`、または電源またはリンクを失ったメンバー：`state`、応答していない間の`waiting`、`reopening`、 続いて`recovered`、`members`および`why` — アレイは、そのノードが再び応答すると接続を再開し、接続時のPTP、 トリガーおよび同期ラインの配線、ペーシング、カメラ設定を行い、その後、新たな `connect_array` なしで再びストリーミングを開始し、`ptp_epoch`（測定値）、`sync_mode` / `sync_tier`、`packed_transport` / `packed_decision`、`per_cam_emit_mbps`、`link_speed_mbps`（`4294` は、Windows がアダプタの速度を読み取れなかったことを意味します — [予想フレームレート](#expected-frame-rates) を参照してください）、`trigger_pipeline` / `pipeline_tune`、`radiometric_chain`（バンドごとの放射輝度チェーンが使用可能かどうか、およびどのメンバーがそれを破損させたか）、および `band_clamp` — ライブビューで配列がレンダリングされる間：アンミックス処理により0にクリップされた点灯ピクセルの割合（各メンバーごとに （`members`：カメラのR、G、Bバンド順における`{serial: [band0, band1, band2]}`）およびバンドごとに、合成画像が示す （`composite`: `{band name: share}`）、さらに`warn`は、GUIが「信頼性の低いバンドデータ」としてフラグを立てた割合 （約1秒に1回測定される；ライブビューが実行されていない場合は`None`）、および`member_ae` — 各メンバーごとに、露出（`exposure_us`）とゲイン （`gain_db`）を、そのメンバーごとに、フレーム自体から取得します（`read_nodes` または `ExposureTime` / `Gain`は、自動露出の実行中に最後に書き込まれた値を返すものであり、現在行われている処理の内容ではない）。 |
| `set_member_settings(serial, **kwargs)` | 1つのメンバーカメラの設定を書き込む。`CameraSession.set_settings()`はそれらを （露光時間、ゲイン、目標輝度など）を1つのメンバーカメラの設定として書き込みます。メンバーは配列に残ります。メンバーではないシリアルに対しては`ChlorosConnectError`を発生させます。[配列メンバーの露光](#exposure-on-array-members)を参照してください。 |
| `capability(timeout=10.0)` | スマートプレップが決定した内容（キャプチャ層、フレームサイズ、`pixel_format`）、ライブ `health` ブロック、およびホストのワイヤ予算。 [アレイの健全性](#array-health--which-subsystem-is-losing-frames)を参照してください。 |
| `set_target_fps(fps, timeout=10.0)` | 再接続せずに、アレイの稼働中にトリガーレートを変更します。数値を設定すると固定されます（レートコントローラは変更しません）； `None`は自動設定に戻します。`{requested, target_fps, target_fps_auto, clamped}`を返します。レートがホストおよびネットワークが処理可能なレベルまで低下した場合、`clamped`はTrueになります。 [ドローン測量におけるインターバルキャプチャ](#interval-capture-for-drone-surveys)を参照。 |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | すべてのカメラから1フレームずつ、**すべて1つのトリガーから**、numpy配列として取得（ファイルは書き込まれません）。`GrabResult`（または`dict`）を返します: `{serial: array}`、または `{serial: (array, metadata)}` と `with_metadata=True` を返します。 `.group_id`は各フレームが共有するトリガーです。`.errors`は、除外された各カメラをその理由にマッピングします（そのトリガーで処理できないカメラは省略され、他のカメラから補完されることはありません）。 `processing` は、`raw` / `debayered` / `radiance` / `reflectance` / `display`です。これは、プロジェクトAPIの[`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — 1つの共有実装 — と同様に動作するため、そこにあるすべてのキーワード（`band`、`align`、 `bit_depth`、`refl_max`）はここで動作します。`numpy`が必要です。 |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | 1ティックごとに1つの `grab()` 結果を生成するイテレータ（キーワード引数のみ）。`fps` にペース制御され、 `duration`（秒）および／または `count`（`None` = 無制限）によって制限されます。**返される各セットは新しいトリガーとなります：** 最新のトリガーがすでに生成済みのセットであるティックは、それを繰り返す代わりに次のトリガーを待機するため、ストリームは `fps` と配列自身のフレームレートのうち遅い方で実行されます。沈黙する代わりに `FrameFetchError` を発生させます： すべてのカメラが、次のティックで繰り返しが発生する理由により拒否した場合、フレームのないティックが 60 秒続いた場合、および新しいトリガーがない状態が 60 秒続いた場合（「60 秒間新しいトリガーなし」）。 `ArrayHandle.frame_stream()` と同じ実装です。[独自のパイプラインでのライブフレーム](#live-frames-in-your-own-pipeline)を参照してください。 |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | バックエンド独自の連続キャプチャ（GUIの「連続モード」）を開始 → [`ContinuousHandle`](#continuoushandle)。`capture()`が保存するのと同様に（エクスポート制御も同様）、停止されるか、`max_captures` / `duration_s` （`None` = 停止されるまで）が実行されるまで続行します。この配列に対して既に実行中のプロセスがある場合は、`ChlorosConnectError` を発生させます。 |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | 1つの同期されたキャプチャグループ。`CaptureResult`を返す （フレーム辞書のリスト + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`）。前の結果`group_seq`を`after_seq`として渡すことで、新しいグループを待つのではなく、その直後の同期済みグループを取得します。 以下のエクスポート制御。 |
| `capture(..., smart=True)` | **スマートキャプチャ** — すべてのカメラで AE が安定するのを待ってからトリガーします。 |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | 最速キャプチャ：RAW データのみ + 割り当てられた DAQ 読み取り値 （＋フリーの結合インデックス）。GUIの「最速キャプチャ」ボタンと同様の動作。 |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | 単発／連続／間隔を1つの境界付きループ内で実行。 `list[CaptureResult]`を返します。**終了させるには`count`および／または`duration_s`が必要です**（SDKにはCtrl+C機能がありません）。また、パス終了後、 パスがドライブの満杯を報告した場合（`.failed`エントリに`disk_full`が含まれる場合）、そのパスが最終結果となります。 |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | ライブの複合インデックスビューの録画を video/GIF へ開始 → `RecorderHandle`。動画は Motion-JPEG 形式の `.avi` （ffmpegは不要）。GUIビューが開かれていない場合、バックエンドは以下をレンダリングします：`expr`（配列のバンド名によるインデックス式、例：`"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`）、またはデフォルトでは配列のNDVI。 配列ごとに1つの合成レコーダー。 |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | 高fpsの生ベイヤーバーストを開始 → `RecorderHandle`。`build_video()` を使用してオフラインで再処理します。`orient=True`：`build_video()`が生成する出力は、カメラの向きに応じて回転されます（各カメラごとの出力は当該カメラの向きで、結合インデックスはマスターカメラの向きで）。RAWフレームはネイティブのまま保持され、バーストマニフェストには各メンバーの向きが記録されます。 |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | 保存済みのRAWバーストを、キャリブレーション済みの動画にオフラインで再処理します。 完了するまでブロックし（`wait=True`）、ビルド結果を返します：`{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — [ビルド結果](#the-build_video-result) を参照。 |
| `build_video_status(job_id, timeout=15.0)` | オフラインビルドジョブ（`{running, result, error, burst_dir}`）をポーリングします。 |
| `disconnect(keep_settings=False, clear_settings=False)` | アレイ全体を解放します — **このセッションがそれを開いた場合** （`connect_array`）：その場合は `force_disconnect()` となり、解放が確認されない場合に `ChlorosConnectError` を発生させる （`with`ブロックの終了時には例外は発生しない）。`attach_array`からのハンドルに対しては、アレイを実行したままにし、その旨をstderrに1行出力する。このアレイはGUIや別のスクリプトに属している可能性がある。 各カメラは、接続時に検出された状態のまま、自由に動作し続けます。`keep_settings=True` はその設定をカメラ自身のメモリに保存し、`clear_settings=True` はそのメモリを工場出荷時の設定にリセットします（[カメラのメモリ](#camera-memory--keep_settings--clear_settings)を参照）。 |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | 誰がアレイを開いたかに関係なく、アレイ全体を解放します。アレイを終了させたい場合（例：異なる設定で再接続する場合など）に、接続済みのハンドルに対して使用してください。 バックエンドが解放を確認した時点、または解放すべき対象がない場合（配列がすでに存在しない、またはバックエンドが実行されていない）に返ります。タイムアウトした、またはバックエンドによって拒否されたテアダウンでは、`ChlorosConnectError` が発生します。いずれのフラグの場合でも、各カメラのメモリが （`camera_memory`）、それ以外の場合は `None` を返します。 |

`capture()` エクスポート制御（GUI/CLI が使用するのと同じエンドポイント）：

- `processing` / `levels` — `processing="all"`（または `levels=["raw","radiance",…]`）は、カメラごとに適用可能なすべてのエクスポートタイプを保存します。 単一の `processing` 値は、そのレベルのみを保存します。
- `aligned` — **デフォルトで有効（`None`）です。これは、Chlorosアプリによる同一アレイのキャプチャが**（その配列ごとの**アラインメント** キャプチャ設定）に準拠しているためです。各メンバーの非RAWエクスポートは、配列の [アラインメントプロファイル]（#array-alignment）によってマスターカメラのフレームにワープされ、メンバー間の共通オーバーラップ領域にトリミングされるため、ファイルはピクセル単位で位置合わせされ、アプリと同じサイズ、トリミング、および `Alignment*` タグで出力される（ROI 内では、トランスフォームはウィンドウに基づいて再計算される）；`XMP-mapir:AlignmentJSON`は、その`"applied": true`について`crop_roi`と記述しています。 RAWファイルはネイティブサイズのまま歪み補正されませんが、メタデータに変換情報が含まれます。`aligned=False`はすべてのファイルをネイティブサイズで、歪み補正なしに保存します。`aligned=True`は、このデフォルト設定を明示したものです。`continuous()`、`capture(smart=True)`、`capture_repeated()`、およびプロジェクトレベルの`ArrayHandle.capture()` / `capture_smart()`は、同じデフォルト設定を採用しています。 ワープが適用されているかどうかに関わらず、配列にアライメントプロファイルが設定されると、各メンバーファイルはそのカメラの `Alignment*` 変換を記録します。ただし、プロファイルがアライメントできないカメラ （除外されているか、解法の失敗により単位行列に強制されている）カメラ、および他のカメラがアライメントされていない場合の参照カメラを除きます。これらには `Alignment*` タグは付加されません。 配列にプロファイルがない場合、キャプチャはアライメントされていない状態で保存され、結果にもその旨が示されます。`result.alignment` は `{"requested": True, "applied": False, "warnings": [...]}` となり、ワープが適用できなかった理由（プロファイルなし、メンバーの解像度が異なる、配列が同期できない、プロファイルをマッピングできないフレームなど）。`applied` が `True` となるのは、ワープを要求したすべてのエクスポートがそれを取得した場合のみです。 アレイの自動位置合わせにより、露光が安定するとプロファイルが作成されます。`ArrayHandle.calibrate_alignment()` / CLI の `align-calibrate` を使用すると、オンデマンドでプロファイルを作成できます。 `result.oriented`は、`orient=True`がプロダクトを反転させたかどうかを示します。
- `render_index=False` — カメラごとの植生インデックスのオーバーレイをスキップします。デフォルトでは、設定された場所にレンダリングされます。
- `daq` / `daq_tolerance_ms` — DAQの測定値は、選択されたレベルで必要とされない場合でも、デフォルトで`.daq`サイドカーとして保存されます。`daq=False`を指定するとこれをスキップし、 `daq_tolerance_ms`はマッチング許容誤差を設定します（デフォルトは500）。`force_daq=True`はデフォルト設定の旧表記です。[キャプチャが保存するDAQ測定値]（#the-daq-reading-a-capture-saves--daq_files-and-daq_matches）を参照。
- `orient=True` — 各メンバーの**積**（`display`、`radiance`、`reflectance`、カメラごとのインデックスおよび合成画像）を、アライメントワープ処理後にカメラの向きで回転させます：**マスター**の座標系にワープされた際の**マスター**の向き（`aligned`のデフォルト — メンバーは位置合わせされたまま）で乗算するか、そうでない場合は各カメラ自身の向きで乗算する。**デフォルト：すべてのメンバーファイルはネイティブ**（配列キャプチャでは常にこの設定）。`raw`および`debayered`はいずれの場合もネイティブのままです。各ファイルには、そのピクセルが含まれるフレームの向きが記録されます （`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — RAWデータ、自身のカメラのフレーム）、また`Alignment*`のXMPには、向きが適用される前に適用されたネイティブフレーム変換が保持されます； その変換を含むRAWファイルには、マスターの座標系に変換されたプロダクトの向きである`ChlorosAligned*`（マスターの向き）も記録されます。処理では、各プロダクトを、そのピクセルに属する向き（RAWを変換する際はアライメントが有効な場合のアライメントされた向き、 アライメントを行わない場合はカメラ自身の回転角で回転させます。 `capture_repeated()` および `capture_fastest()` はこの変換をそのまま通過させます。プロジェクトレベルの `ArrayHandle.capture()` / `capture_smart()` も同様にこの変換を引き継ぎます。

**TIFF圧縮 （HTTP専用設定）：** `ArraySession.capture()` は `compression` キーを送信しないため、 そのためバックエンドのデフォルト設定が適用されます。`POST /api/camera/array/capture`は`compression`ボディパラメータを読み取り、`"none"`のデフォルト設定では： 非圧縮で、センサーのピクセルデータを読み取ったままの状態で保存されます（フル解像度のフレームあたり約6.3 MB）。 `"deflate"`（ロスレス zlib L1 ＋ 水平予測、約 4.1 MB/フレーム）は、ファイルサイズは小さくなりますが書き込み速度は遅くなります。どちらもロスレスであり、インポート時の読み取り結果は同一です。 SDKにはこれに対応するkwargが用意されていません。回避策として、`chloros-cli lattice array-capture --compression deflate`または生のHTTPを使用します。DEFLATEはPythonのGILを保持するため、圧縮書き込みはカメラごとのライタースレッド間で並列化されません。センサーレートでのフル解像度キャプチャを継続するには、非圧縮のデフォルト設定が必要です。詳細： [CLIリファレンス → array-capture](cli-reference.md)。**連続するグループ（HTTP）：** エンドポイントは `group_seq` を返します。これを次のリクエストで `after_seq` として再送信するループを実行すると、次の同期済みグループを取得できます。これには、前のキャプチャの保存中に公開されたグループも含まれます — リクエスト後に公開されたグループを待つのではなく、`skipped_groups`はループが見逃したグループを報告します。`capture_repeated`、 CLIの`--continuous`やGUIの「連続モード」はすべてこの動作を行い、間隔ごとのパスはスケジュールされた各時点で常に最新の状態が維持されます。**メンバーごとのエクスポート上書き（HTTPのみ）：**同じエンドポイントは、`exclude_serials`（リスト — 保存されたセットからメンバーを削除。配列は引き続き 1 つの同期グループとしてトリガーされ、除外されたメンバーは `excluded` で返されます）、`serial_levels` （`{serial: [level tokens]}`：カメラ単位のオーバーライド）、および `serial_index`（`{serial: bool}`：カメラ単位のインデックス・オーバーレイのオーバーライド）を受け付けます。 これらはGUIと互換性のある本体パラメータであり、**現時点ではSDKのkwargsではありません**。マップに存在しないメンバーは、配列全体の`levels` / `render_index`にフォールバックします。

##### スキップされたカムの検査 — `CaptureResult.skipped`

`ArraySession.capture()`は`CaptureResult`を返します。これは`list`のサブクラスです。これを反復処理し、インデックスを指定し、 `len()` 処理 — 既存のパターンはすべて引き続き機能します。新しいコードでは、`.skipped` 属性を検査することで、どのカムが除外されたか、およびその理由を確認できます。 最も一般的なケースは、混合フィルターアレイ内のRGBカメラで、`processing="radiance"`または`"reflectance"`を要求した場合です — ブロードバンドセンサーではベイヤー単位の放射輝度は意味をなさないため、バックエンドは意味のない出力を生成するよりも、それらのカメラをスキップします。

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

理由トークンは `<level>-not-applicable-to-rgb-cam` というパターンに従います（スキップされたレベルごとに 1 つのエントリがあり、それぞれが `level` を含みます）。 キャリブレーションパックが読み込まれていないカメラは、`radiance` / `reflectance` を `<level>-needs-calibration (connect the camera so its cal pack loads)` でスキップし、 また、使用可能なDAQの反射率測定値がない場合は、`reflectance-skipped-no-dls`でスキップされます。その他の反射率に関連するスキップには、`reflectance-skipped-no-fresh-dls`（新しい下向き測定値が利用できない場合）、`reflectance-skipped-bound-daq-unavailable (…)` （バインドされたDAQに到達できなかった）、および`dls-uncalibrated-band-<nm>` — バンドの大部分がDAQ光センサーの放射測定校正範囲外にある（~374–974 nm）の範囲外にあるため、DAQに基づく絶対反射率の分割処理は拒否され、フレームはセンサー応答モードへと明示的に降格されます。出荷中のSKUの中ではF988のみがこの現象を引き起こします。このカメラでサポートされているパスは、反射率パネルワークフローです。

##### 書き込みできなかったファイル — `CaptureResult.failed`

一部のファイルしか書き込みできなかったキャプチャでも、以下のリストが返されます： リストには**書き込まれた**すべてのファイルが含まれており、`.failed`には残りのファイルがリストされています。各ファイルごとに1つの辞書が記載されており、`serial` （合成画像と配列のサイドカーを組み合わせたものについては `None`）、`level` （`raw`、`radiance`、…、`index`、 `composite`、`daq`、 または `match` — `.daq` の `.match.json` 由来（`.daq` が書き込まれたが、そのサイドカーは書き込まれなかった場合）および `error`。ドライブが満杯の場合、このエントリには`disk_full: True`も含まれます。そのドライブへのそれ以降のキャプチャはすべて同様に失敗するため、`capture_repeated()`はそこで停止します。 部分的なキャプチャに対しては例外は発生せず、`.failed`内のファイルが書き込み途中でディスクに残されることもありませんでした。クリーンなキャプチャでは、`.failed`は空です。

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### キャプチャの読み取りを行う DAQ は、`.daq_files` および `.daq_matches` を保存します

DAQ がカメラやアレイにバインドされている場合、またはプールされている場合、すべてのキャプチャは、使用したダウンウェル測定値を保存します： キャプチャフォルダの `daq/` サブフォルダ内に `.daq` が保存され、そのファイルには `.match.json` および `.cameras.json` （その測定値に対応するファイル）も保存されます。これはRAWキャプチャやデベイヤー処理済みのキャプチャにも当てはまり、GUI、CLI、SDKのいずれからキャプチャされた場合でも同じファイルになります。`daq=False`は、DAQのレベルを使用していないキャプチャについてはこれをスキップします （`radiance`、`reflectance`、または`display`レベルのキャプチャでは依然として書き込まれます）。`daq_tolerance_ms`は、その キャプチャの許容範囲ラインを移動させ、使用された読み取り値は決して移動させません。どのDAQか：カメラまたはアレイにバインドされているもの（GUIの光センサー選択、プロジェクトのバインド、または`chloros-cli daq pool-bind`）、あるいはバックエンドが持つ最新のものです。 CLIでは、これらの制御項目を`--no-daq`および`--daq-tolerance-ms`と表記しています（[CLIリファレンス](cli-reference.md#the-daq-reading-a-capture-saves)）。

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

キャプチャは、使用した光センサーの測定値を `.daq` として保存し、その横に `<file>.daq.match.json` を付加して、その測定値が露出とどのように関連付けられたかを示します：`method`、`gap_ms`（露光から最も近いDAQ測定値まで）および`in_tolerance`（500ミリ秒以内、またはキャプチャの`daq_tolerance_ms`）。反射率ファイルのXMPには、`Chloros:DlsMatchMethod`、`Chloros:DlsMatchGapS`、および`Chloros:DlsInTolerance`と同じ判定が記録されています。

キャプチャは、露光後の DAQ による最初の読み取り値を待つため、その前後の読み取り値をブレンドすることができます（`method` `interpolated`）。 DAQ-Eは、日中は0.4秒ごと、屋内では1～2秒ごとに測定値を送信するため、`capture()`は、それらの間隔の最大約1.5倍後になって返されますが、 露光自体に遅延は生じません。送信を停止したDAQは待機されません。DAQ-Eの測定値は、前回の測定値からの間隔における光の平均値であるため、その間隔の中間に配置されます： `gap_ms`はその中間点で測定され、`.match.json`'の`bracket_stamps_ns`は、測定値自身のタイムスタンプを示します。`ArraySession.continuous()`は、キャプチャの速度を落とすことなく待機します。`capture_repeated()`'の- バックパスは待機しないため、レートを維持し、露光前の最も近い測定値と照合されます。

測定値には有効期限がないため、遅延した測定値によってフレームが失われることはありません。代わりに、次のようにラベル付けされます：

- **DAQのストリームが停止** — 最新の測定値が使用され、`gap_ms`はその経過時間を示します（`in_tolerance: false`は500 ms以上経過）。
- **読み取るDAQなし**— 接続が切断されている（`disconnect()`、`chloros-cli daq pool-disconnect`）、またはバインドされたDAQに到達できない場合 — カメラの最後の測定値が再利用され、**この** 露出に対して記述されます：この露出からの `method`、`retained`、`gap_ms`、 `in_tolerance`は再計算され、`retained_from_epoch_ns`は最初に照合された露光の名称が付けられます。一度も測定値を取得したことのないカメラには依然として反射率が割り当てられず、結果にはその理由が明記されます （`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`、あるいはキャプチャの拒否）。

Chlorosで開いているプロジェクトのフォルダ内にキャプチャが保存されたファイルは、アプリ内でのキャプチャと同様に、そのプロジェクトのファイルリストに追加されます（他の場所に保存されたファイルはそのまま残されます）： `CameraSession.capture()`、`ArraySession.capture()`（`smart=True`も同様） および `capture_cameras_transient()`。その結果は `.project_add` に記載されています（`capture_cameras_transient()`：辞書の `project_add`） -- `held`、`files`、およびバックエンドの `message`；`held` は `True`であり、処理の実行中はファイルが結合されず、終了後に結合されます。プロジェクトにファイルが取り込まれなかった場合は、`None`となります。

`CameraSession.capture()` および `ArraySession.capture()` は、結果として `.daq_files` および `.daq_matches` を返します。 HTTPキャプチャ応答（`/api/camera/<serial>/cli-capture`、`/api/camera/capture`、`/api/camera/capture-all-synced`、`/api/camera/array/capture`） には、`daq_files`および`daq_matches`が含まれており、それぞれ1つずつ、`.daq`：`daq_file`、 `cameras`、`method`、`gap_ms`、 `in_tolerance`、`tolerance_ms`、および `stream_fault` — DAQ固有のストリーム障害であり、許容範囲外の測定値がある場合にのみ参照されます。キャプチャからの反射率を信頼する前に、その `.match.json` または XMP 内の `in_tolerance`、あるいは DAQの [`status()`](#daqsensorsession-methods)を確認してください。

`processing`のレベル：

| レベル | 出力 |
| --- | --- |
| `"raw"` | センサーから直接取得したシングルチャンネル・ベイヤー（モノクロカメラ：単一バンド） |
| `"debayered"` *(SDKのデフォルト)* | 3チャンネルカラー：RAWデータからキャリブレーションパックのダークオフセット（DSNU）を差し引いた後、OpenCVのエッジ認識デモザイク処理を適用 — 直線性LUT、フラットフィールド、またはアンミックスは適用されません （モノクロカメラ：1チャンネルグレースケール）。 保存されたTIFFファイルは**RGB**チャンネル順（測光用RGBのため、どの画像ビューアでも真の色で表示されます）ですが、`grab()` / `frame_stream()`が返すメモリ内配列はOpenCVの**BGR** 順序で返されます — 保存されたファイルと比較するには、チャンネルを逆順（`frame[..., ::-1]`）にしてください。`chloros-cli process`は、`Debayered_Images`と同じ結果を書き込みます。 |
| `"radiance"` | 完全な放射測定チェーンを経由した float32 W/m²/sr/nm。マルチスペクトルのみ — RGBカメラはスキップされます。 |
| `"reflectance"` | uint16 0..32768 (Pix4D対応); 絶対基準値を得るには、ライブDAQとのペアリングが必要です。マルチスペクトルのみ。 |
| `"display"` | GUIプレビューと一致する完全な処理チェーン（カメラのプロファイルに基づくCCM + WB + ガンマ）。 |
| `"all"` | **各カメラにつき、該当するレベル**ごとに1ファイル（GUIの「Capture All」およびCLIのデフォルト設定に準拠）。返される`CaptureResult`には、`(cam, level)`ごとに1つのフレーム辞書が含まれ、各辞書にはレベルが格納されます。 適用対象外のレベルは `.skipped` に格納されます。キャプチャの読み取りに使用された DAQ は、`.daq` サイドカーとして保存されます。 |

> **注 — デフォルト設定はCLIとは異なります。** `ArraySession.capture()`のデフォルトは`processing="debayered"`、`chloros-cli lattice array-capture`コマンドのデフォルトは`processing="all"`です。 SDK から `processing="all"` を明示的に指定することで、CLI/GUI のマルチレベル保存を再現できます。

##### 配列キャプチャファイル名

各レベルは、`output_dir` の個別のサブフォルダ（`raw/`、`debayered/`、 `radiance/`、`reflectance/`、 `preview/`は`display`、`index/`、`composite/`に対応）。各ファイル名は、カメラのシリアル番号を「`SN<serial>-<filter>`という形式で、シリアル番号の後にカメラモデルのフィルター部分が続き、例えば`SN214000533-FOCN`は`M3C-L87-FOCN`用、 `SN261902142-F450`は`M3M-L87-F450`用、`-`は したがって、ファイル名は以前と同様に「`_`」で区切られたフィールドで構成されます。 モデル名が読み取れないカメラは「`SN<serial>`」のみであり、以前のリリースからのキャプチャには名前にフィルターが含まれていません。ファイル名は、メンバーに要求されたレベル数によって異なります：

| メンバーに求められたレベル | ファイル名 | 例 (`reflectance`、デフォルトは `format="tiff"`) |
| --- | --- | --- |
| **1レベル** (`processing="reflectance"`、または1項目の`levels`) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — 以前のリリースと同様にレベル接尾辞がないため、既存のスクリプトは引き続き動作します | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **複数レベル** (`processing="all"`、または項目が複数ある `levels`) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[クランプマスク](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on)は製品名に続いて指定され、`clamp_masks/<product folder>/<product name>_clampmask.tif`となります。単一のレベルの反射率キャプチャでは `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif` が書き込まれ、`processing="all"` では `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif` が書き込まれます。 接尾辞のルールは、各メンバーに対して要求されたレベルに基づいて決定されるため、スキップされたレベル（例：RGBカメラの反射率）によって変更されることはありません。パスを構築するのではなく、返されたフレームからパスを読み取ってください。単一カメラによるキャプチャ （`CameraSession.capture()`）では、常にレベル `capture_<ts>_SN<serial>-<filter>_<level>.tiff` が追加されます。

##### PNG／JPEGは、表示用およびインデックス用ファイルのみを受け付けます

PNGまたはJPEGは、チャンネルあたり8ビットを保持します。保持できないレベルを指定して要求した場合、キャプチャが行われる前に拒否され、そのレベル名と、それを保持できるフォーマットとしてTIFFが指定されたメッセージが表示されます：

- **`radiance`**(float32) および**`reflectance`**(uint16、`32768` = 1.0) — したがって、これらを含む `processing="all"` も同様です。 SDKは引数に対してこれをチェックします。`capture()`は、リクエストなしに**`ValueError`** をリクエストなしに呼び出すと（`CameraSession`、`ArraySession`、`CameraHandle`、 `ArrayHandle`、`capture_smart`）。
- **8ビットを超えるストリームを送信するカメラからの`raw` / `debayered`**（12ビット M3C、Mono12 M3M）— バックエンドは各カメラのビット深度を認識しているため、これらを拒否します： `error_code` および `FORMAT_CANNOT_HOLD_LEVEL` に対しては HTTP 400 が返され、`ChlorosConnectError` として発生します (`CameraSession` / `ArraySession`) または `RuntimeError` (`CameraHandle` / `ArrayHandle`)として発生します。1台のカメラによるシングルレベルの`debayered`キャプチャは、PNG/JPEG用に8ビットのフレームを取得しますが、これは拒否されません。

放射測定データは`format="tiff"`（`ext=".tiff"`）として保存してください。`display`およびインデックスレンダリングにはPNG/JPEGを使用してください。

#### 自動露出の動作

- **露出を優先し、ゲインは後回し。** ゲインが「自動」設定の場合、露出が許容される最大値に達するまで、ゲインは 0 dB のまま維持されます。
- **手動露出設定時でも、ゲインを変更するか手動に設定するまでは、ゲインは「自動」のままです。** 再び「自動」に戻すと、ゲインの上限は以前の状態に戻ります。
- **小さなハイライトはシーンを暗くしません。** フレームの 1 % までのスペキュラーは無視され、既知のキャリブレーションターゲットのパネル内では、パネルの 0.5 % までが対象となります。
- **DAQ の露出上限は、キャリブレーションターゲットが視野内にある間のみ適用されます。** これにより、白いパネルがクリッピングされるのを防ぎます。パネルが視野に入っていない場合は、カメラはシーンに合わせて露出を調整します。
- **光量が一定の場合は露出が安定し、光量が変化した場合は数秒で安定します。** 光量が一定の場合、露出とその上限は変わりません。光量に大きな変化があった後、表示は数秒以内に安定します。 クリッピングが発生しても、露出が恒久的に上限に達することはなくなり、ROIが変更されても適切な露出が再設定されます。
- **輝度ターゲットは制御可能な範囲内に収まります。** カメラが露出上限に達し、ゲインを上昇させられない状態では、輝度ターゲットが上昇することはないため、光が戻ってもフレームがフレア状態になることはありません。
- **自動露出が反応しなくなったカメラは、その時点の光量に応じたレベルに維持されます。** 強い日差しの下では、カメラが最短露出まで下がり、暗いフレームのままその状態を維持し、すべての設定値が正常を示すことがあります。Chlorosはフレームを監視し、その光量に対してカメラが維持していたレベルに固定し、光量が変化した時点でその状態を解除します。 光センサーが暗くなっていたり覆われていたりしても、この動作は妨げられません。

#### アレイ構成要素の露出

LATTICEカメラは、`ExposureTime`をフレームレートの1周期に制限します。アレイがストリーミングしている間、Chlorosはすべてのメンバーに対して`AcquisitionFrameRateEnable`を有効にして実行し、 トリガーレートよりわずかに高いレートで実行するため、メンバーの露光上限は、**どのROIにおいても**、トリガー周期よりわずかに短い値となります：

- メンバーのフレーム周期は、トリガー周期から周期の15％に相当するスラック（最小3 ms、最大で周期の半分）を差し引いたものであり、その露光上限は、そのフレーム周期から約0.1 msを差し引いた値となります。 例：10 fpsのトリガー（100 ms）の場合、メンバーレートは約11.8 fps、露出上限は約85 msとなります。
- **その上限を超える手動設定の`ExposureTime`の場合、それに合わせるためにトリガーの速度が低下します。**Chlorosはまず、そのメンバーの上限をその値に合わせて開放し、低速なレートで数トリガー周期実行させます（書き込みには数秒長くかかる場合があります）。その後、書き込みを行い、 トリガーは、露出が許容する範囲を超えて速く発火することはありません。自動レート配列の場合、手動露出によって遅くなっていた各メンバーが自動露出に戻るたびに、以前のレートが復活します。 カメラ自体が取得できない値は依然として拒否され、その値はエラー範囲として扱われます。また、その書き込みのためだけにオフにされていたメンバーの自動露出は、元の状態に戻されます。バージョン1.3.0以前では、小さなROIが各メンバーのレートにセンサー自身のフリーラン周期の上限を設けていました。 トリガーレートに関係なく、768 × 304 では 4.9 ms でした。そのため、どのメンバーも明るくすることはできませんでした。
- Chlorosは、より高速なトリガーに合わせて手動露出時間を短縮することはありません。自動レートは手動露出が許容する範囲内に収まります。露出が許容する速度よりも速い固定レートが設定されている場合、そのメンバーにはそのレートに対応できる十分な長さの上限が設定され、露出が許容する速度以上のトリガーは受け付けません。すべてのメンバーがすべてのトリガーに対応する必要がある場合は、露出時間または固定レートを下げてください。
- **読み出し：** メンバーのノードを読み取ると、`AcquisitionFrameRateEnable`がTrue、`AcquisitionFrameRate`がメンバーのレートで表示されます。アレイが停止すると、イネーブルはオフに戻ります。
- **これら2つのノードはアレイが所有しています。アレイのメンバーに対して `AcquisitionFrameRateEnable` または `AcquisitionFrameRate` を書き込まないでください** — 代わりに、アレイのレートを `target_fps` / `set_target_fps()` で設定してください。

メンバーのエクスポージャーは、`arr.set_member_settings(serial, exposure_time=20000)` で書き込みます（`CameraSession.set_settings()` のキーワード；`chloros-cli lattice exposure --serial` も同様の動作をします）。 メンバーは配列に残ります。`connect_camera()` を使用して配列メンバーを開かないでください。 connectは、すでに配列に含まれているカメラを拒否し、`CameraSession`が返す値は、切断時にそのカメラを配列から削除します。`ArraySession.status()["member_ae"]`は、各メンバーの最新のフレームからの露出とゲインを報告します。

### 1つのプロジェクトレコード、すべてのモード

プロジェクトフォルダには、すべてのデバイスの設定が格納されます：`cameras.json` — 各カメラの設定（`cameras[].settings`：単一カメラおよび各アレイ構成要素用）と、各アレイ全体に適用される設定（`arrays[].capture`： フォーマット、ROI、ビニング、トリガーレート、ワイヤ予算）— および `sensors.json`（各光センサーの設定：積分時間、フレーム平均、上限値）が含まれます。アプリ、CLI、SDK、MCPサーバーはすべて、同じファイルを同じ方法で読み書きします：

- **プロジェクトに接続されたデバイスは、そのプロジェクト内に保存され、すべての変更がそこに記録されます。** 「プロジェクトの下」とは、バックエンド（アプリ側）でプロジェクトが開かれている状態、あるいは — `ChlorosProject`（`open_project()`）を介して、バックエンドに独自のフォルダ名を指定している状態を指します (`project_path`)。プロジェクトにまだ登録されていないカメラ、アレイ、または光センサーは、接続時にその設定とともにファイルに追加されます（`connect_camera()`、`connect_array()`、光センサーの接続、CLI、アプリ、 `chloros-mcp`）に、設定情報と共にファイルとして追加されます。いずれかのモードからの変更（`set_settings()`、`ArraySession.set_member_settings()`、 `set_target_fps()`、アレイ設定、`set_cap()`、CLI、 アプリのコントロール）からの変更は、その直後にそのプロジェクトのファイルに保存されます。カメラで設定された露出、ゲイン、またはAEターゲットは「保持」状態として保存されるため、プロジェクトを再開すると再びその設定が適用されます。
- **プロジェクトを開くと、その設定に基づいてデバイスが設定されます**。どのモードでも同様です。独自の設定がない状態で接続されたカメラ、アレイ、または光センサーには、プロジェクトの保存された設定を引き継ぎます（プロジェクトを開いている状態での `lattice cam-connect` / `array-connect`、`chloros-cli project connect`、`open_project().connect_all()`、アプリの再起動時）。 新しいIDで接続されたアレイは、プロジェクトが保存したのと同じカメラ構成でアレイを更新します。異なるカメラ構成のアレイは、その横に追加されます。
- **アプリ自身の保存処理は、デバイスのライブ設定を**古いコピーで上書きすることは決してなく、CLIやSDKからの呼び出しによって保存されたメンバーごとの上書き設定を維持します。
- **設定したピクセル形式が、プロジェクトに保持される形式となります。**「配列設定」で設定された形式、`--pixel-format`または`connect_array(pixel_format=)`によって設定された形式（8ビットを含む）は、そのまま保存され、再接続時にも正確に反映されます。 何も設定されていない場合、各カメラは12ビットのデフォルト設定（モノクロセンサーでは`BayerRG12`、`Mono12`）で接続されます。フォーマット名はビット深度を意味します。 ファミリーはセンサーのものであるため、カラーカメラがモノクロフォーマットに設定されたり、モノクロカメラがベイヤーフォーマットに設定されたりすることは決してありません。 また、フォーマットがマッピングされた場合は、行 `[PixelFormat <serial>] ... written` にその旨が表示されます。8ビットキャプチャのエクスポートデータは、反射率データを含め8ビットであり、再スケーリングされ、`PixelScale` がスタンプとして付加されます。
- プロジェクトに紐付けられていない状態で接続されたデバイスは 何も記録されません。
- **録画は、それより後に作成されたプロジェクトに従います。開くと、そのプロジェクトのまま残ります。**プロジェクトが作成されると — `create_project()`、`chloros-mcp`の`create_project`、またはアプリの**[新規プロジェクト]**— または複製（アプリの**複製**機能。これによりコピーが開かれます）が行われた際、録画がまだ以前に開かれていたプロジェクトに書き込みを行っている最中であっても、録画は継続され、書き込まれた内容は新しいプロジェクト内の同じ場所 （下図参照）。既存のプロジェクトを開く場合 — `ChlorosProject.process()` （バックエンドでそのプロジェクトが開かれます）、`chloros-cli process`、アプリの**プロジェクトを開く** — これにより、録画中のプロジェクトへの書き込みは継続されます：録画はそこで完了し、そのファイルは当該プロジェクトのファイル一覧に追加されます（アプリはプロジェクトを開く前に、すべての録画を停止します）。
  - `ArraySession.record()` および `ArraySession.burst()`：録画が停止すると、動画、またはすべてのフレームを含むバーストフォルダが丸ごとそこに移動します。停止の結果として新しいパスが報告されます (`out_dir`、`video_path`)が報告され、そこからバーストのビルドが行われます。 ドライブが移動を拒否した場合、録画データは書き込まれた場所に残り、結果にもその旨が示されます（`relocation_error`）。停止されたバーストの生フレームと `.daq` は、そのバーストが最終的に属することになるプロジェクトのファイルリストに追加されますが、動画は プロジェクトのファイルリストには追加されません。
  - `ArraySession.continuous()`：変更後のすべてのキャプチャは新しいプロジェクトに保存されますが、変更前に保存されたものは元の場所に残ります。
  - `DAQSensorSession.record_start()`（およびアプリ独自の光センサー録画）: `.daq`は、録画が停止すると新しいプロジェクトの`light_sensor`フォルダに移動します。`record_stop()`は新しいパスを返し、そのファイルは当該プロジェクトのファイル一覧に追加されます。

  開いているプロジェクト外のフォルダへの録画は元の場所に残り、`start_camera_continuous()` / マルチストリームジョブは指定されたフォルダを維持します。
- **アプリ内では、新しいプロジェクト作成後に実行されたキャプチャはそのプロジェクトに保存されます。** プロジェクトの作成または複製を行っても、実行中のすべてのキャプチャと録画は継続され、それ以降に実行された各キャプチャ（「すべてキャプチャ」、「キャプチャ開始」）は新しいプロジェクトに保存され、そのファイルリストに追加されます。 アプリで既存のプロジェクトを開くと、まずそれらすべてが停止します。
- **プロジェクトは録画中は移動されません。**開いているプロジェクトを別の作業ディレクトリに移動しても（アプリの**設定 → 作業ディレクトリ**；SDK呼び出しではプロジェクトは移動されません、 HTTPクライアントが`/api/move-project`に投稿する場合は移動可能）は、配列録画やバースト、配列連続キャプチャ、カメラの連続／マルチストリームジョブ、または光センサー録画がそのプロジェクトに書き込みを行っている間は、HTTP 409 `PROJECT_RECORDING`で拒否されます： `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

これは、切断時にカメラがどのような状態になるかとは別問題です（[カメラメモリ](#camera-memory--keep_settings--clear_settings)を参照）：プロジェクトは、開かれる際に自らカメラの設定を行います。

### カメラのメモリ — `keep_settings` / `clear_settings`

アレイが接続されると、そのカメラはアレイの設定を引き継ぎますが、その後、設定を変更することができます。 Chlorosがカメラを解放するとき — `ArraySession.disconnect()` / `force_disconnect()`、一時的な関数の終了時、または`start_camera_continuous()`の停止時 — 各カメラは、接続前に検出された状態（トリガーおよび同期ラインの配線、PTP、カメラ設定）に戻されます — `ExposureAuto` / `GainAuto` （および手動設定の `ExposureTime` / `Gain`）、`TargetBrightness`、およびAEリミットとダンピング、ホワイトバランス、ブラックレベル、`PixelFormat`、 ビニングおよびROI、カラーパイプライン（`GammaEnable` / `Gamma`、`LUTEnable`、`ColorTransformationEnable`）、`ReverseX` / `ReverseY`、`UserSetSelector`、PTPの`PtpSlaveOnly`／`PtpDomainNumber`、 パケットサイズ（`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`）、ペーシング （`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`、`GevSCPD`、`GevSCFTD`）、チャンクデータ （`ChunkModeActive`、各 `ChunkEnable` およびその `ChunkSelector`）、トリガー入力のデグリッチ、および配列がスレーブの同期ラインに与える影響 （`LineFilterSelector` / `LineFilterWidth`）および取得フレームレート （`AcquisitionFrameRate`：イネーブルがオンの場合のみ。オフの場合は、設定値ではなく、ペーシングが許容する最大レートが読み取られます）。記載されるのは相違点のみです。カメラが最初に停止した時点、およびフレーム期間を必要とする露光前のペーシングとフレームレートです。

プロジェクト外でカメラを使用する場合、2つの引数がカメラの**内蔵メモリ**（電源投入時に読み込まれるデータ）を書き込みます。これは、どのソフトウェア環境でも適用されます（`disconnect()` / `force_disconnect()`、すべての一時関数、`start_camera_continuous()`、および`ChlorosProject.disconnect_all()`；MCP：`disconnect_device`および一時ツールと同じ引数； CLI：`--keep-settings` / `--clear-settings`）：

- **`keep_settings=True`** は、各カメラの設定をセッション終了時の状態のままメモリ（UserSet2、起動セットとして設定）に保存します。 カメラは引き続きフリーランニング状態（`TriggerMode` `Off`）にあり、同期ラインの配線は現状のままです。 UserSet2がないカメラはUserSet1に保存されます。このコンピュータはそれを記憶しているため、Chlorosのアレイの自己修復機能（Chlorosのベースラインから逸脱したUserSet1をリセットする機能）は、そのカメラには手を加えません。
- **`clear_settings=True`** は、各カメラのメモリを工場出荷時の設定（Chlorosのベースライン、つまりそのアレイの自己修復機能が反映した状態）にリセットします — `TriggerMode`、`Off`、`TriggerSource`、`Line2`による工場出荷時のデフォルト設定、自動露出、RGBカメラのキャリブレーション済みホワイトバランス は維持 — UserSet1に格納され、ブートセットとして設定された。
- 両方は不可 （`ValueError`；バックエンドは400を返す）。応答には`camera_memory`が含まれ、カメラごとに`{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}`または`{"cleared": bool, "errors": [...]}` — `force_disconnect()`がこれを返す；ジョブの場合は`stop()`が伴う。

アプリはこれらを提供しない。

**どの出口を経由しても、カメラは元の状態に戻されます。**各終了パスで、検出時の状態への復元処理が実行されます：`CameraSession.disconnect()`（アプリのカメラ**X**）、アプリの**X**、または任意の配列要素（配列全体は解放される）に対して実行され、アプリを終了します（バックエンドには、すべてのカメラ、配列、および実行中の連続録画を元に戻すために最大20秒が与えられます。実行中の一時関数やワンショット呼び出しは、次のセーフポイントで停止し、まずカメラを元に戻し、 最大8秒間かかり、HTTP 503 `SHUTTING_DOWN`を返す；実行中の`connect_array()`は、次のカメラ処理の前に停止し、開いていたカメラを最大10秒かけて元に戻し、`ChlorosConnectError`（HTTP 503 `SHUTTING_DOWN`）を発生させる；また、シャットダウン中に到着したものは何も開かない：HTTP 503 `SHUTTING_DOWN`）、および強制終了されたりターミナルが閉じられたりした `chloros-cli` コマンド（そのジョブは停止され、カメラは最初に元に戻される）。クラッシュしたり強制終了されたりしたバックエンドは、何も元に戻すことができない。電源の再投入により、各カメラはメモリの状態に戻される。**プロジェクトはカメラの設定を自身で行う。** アレイがプロジェクト（アプリで開かれているプロジェクト、または `ChlorosProject`）を介して接続されると、 バックエンド（`project_path`）に対して自身を登録する）を介して接続される場合、各メンバーはプロジェクトによって保存された設定を受け取ります。具体的には、そのカメラに設定された露出、ゲイン、またはAEターゲット（「ユーザー設定」として保持されるため、自動露出はこれを変更しません）およびホワイトバランスモード（Chlorosの補正； カメラ自身のホワイトバランスは無効のまま）、カメラのメモリに保存されている内容にかかわらず適用されます。

### 撮影モードとレコーダー

配列の表面は、GUIの撮影パネルを反映しています：シングル／連続／インターバル／最速シャッターモードに加え、2つのレコーダー（ライブコンポジットビデオとRAWバースト→オフライン再処理）があります。

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**は、SDKの連続／インターバルループです。スクリプトからこれを中断するための `Ctrl+C` が存在しないため、 `count` および／または `duration_s` を渡す**必要があります**（いずれかに達すると停止します）。`interval_s` は各パスの開始時点から計測されます （GUIと同様）。連続パス（`interval_s=0`）は `group_seq` → `after_seq` と連鎖するため、これらは配列のレートで動作する連続した同期グループとなります。残りのkwargsは`capture()`にそのまま渡されます。
- **`continuous`**は、バックエンド独自の連続キャプチャ機能であり、GUI の「連続モード」に相当します。**高フレームレートでディスク上にすべてのトリガーを記録したい場合は、`capture_repeated()` ではなくこちらを使用してください：** `capture_repeated()`はスクリプトからグループごとに1回のHTTPリクエストを行うのに対し、`continuous()`はグループ間の往復通信を伴わずに、バックエンド内で2つのキャプチャを並行して処理します。 9台のカメラアレイでROI 512を測定した結果：73～77キャプチャ/秒（アレイの公開レートは約78）。`capture_repeated()`はインターバルキャプチャ（`interval_s`）用に保持し および、1つずつ確認するパス用に保持してください。 Windows では、高レートを実現するために、Microsoft Defender からバックエンドを除外してください（[ステップ 1](#step-1--install-the-chloros-platform-package) を参照）。
- **`record`** は *モニタリンググレード* です： レンダリングされたライブの複合インデックス合成画像を記録します。 GUIの複合ビューが開かれている場合は、表示されているそのビューを記録します。開かれていない場合は、バックエンドがビュー自体をレンダリングします：ビューを指定した場合は`expr`、指定しない場合は配列のNDVI（660 nmおよび850 nmに最も近い赤色およびNIRバンド、 ライブビューでそれらが選択されるように）、配列のアライメントが完了した時点で。`expr`内のバンド名は、配列が指定する通りに命名してください： M3Cカメラの場合は`<band>_<filter>`（`NIR_850_RGN`、`Red_660_RGN`、`Orange_615_OCN`）、 M3M用には`Mono_<filter>`（`Mono_850`）；短いカラー名（`NIR`、`Red`） は、配列の1つのバンドにのみ設定されている場合にのみ機能します。録画中に開かれたGUIビューは、開いている間、録画されます。フレームを書き込まない録画は、`error`および`"no-frames: …"`で失敗します（配列がまだアライメント中、または露光の安定化中である可能性があります）。 また、`stop()` が発生します（[`RecorderHandle`](#recorderhandle) を参照）。アレイごとに 1 つの複合レコーダー（すでに実行中の場合は例外）。
- **`burst` → `build_video`** は *分析グレード* です： `burst`は、RAWフレーム＋フレームごとのマニフェスト＋`<output>/bursts/<base>/`の下で個別のDLS読み取りごとに1つの`.daq`を、グラブループのフルレートで書き込みます（チェーンなし、 exiftoolなし、ライブビューなし）。`build_video`は各フレームを最も近い`.daq`と時刻を照合し、インポートパイプラインの放射輝度／反射率／インデックス・チェーンを再実行します。`products`は、`{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`（デフォルト：統合インデックス）のリストです。`burst().stop()`は、ベストエフォート型の統合-indexビルドを自動で開始し、停止結果として`build_job`として返されます。

##### `build_video`の結果

| キー | 意味 |
| --- | --- |
| `success` | `True` は、要求されたすべての製品が要求どおりに構築された場合のみ表示されます：少なくとも 1 つの出力があり、`skipped` には何も含まれず、 かつ `degraded` に何も含まれていない場合のみ。 |
| `outputs` | 書き出された動画（および GIF）のパス。 |
| `errors` | 問題点のテキスト。 |
| `skipped` | 動画を**生成しなかった**各リクエスト対象製品につき1つの `{"serial", "level", "reason"}` — カメラの （そのカメラ用のキャリブレーションバンドルの欠如、反射率のDLS測定値の欠如、処理チェーンの失敗）か、または複合インデックス（`"kind": "combined"`、`serial`、`None`を含む）によるものである。 （カメラ群に赤/NIRバンドのペアが存在しない、両バンドの放射度を持つグループがない）。いずれかの項目が該当すると、`success` `False` となり、その他の動画は引き続き書き込まれ、一覧に表示されます。 |
| `substitutions` | 要求されたレベルとは異なるレベルでレンダリングされたカメラとその理由 — 放射度／反射度を要求したRGBカメラは、カメラのキャリブレーションバンドルが利用可能な場合に、ホワイトバランス調整済みの表示用動画が生成されます。これはエラーではありません。 |
| `notes` | 問題ではなく情報的なテキスト — 例：カメラごとのインデックス式がないカメラにはインデックス動画が存在しない、またはRGBカメラの表示動画にホワイトバランスが適用されていない（キャリブレーションバンドルがない、あるいは工場出荷時のホワイトバランス設定がない場合など）。この場合、カメラ名が明記される。 エラーではありません。 |
| `degraded` | `True`：製品が記述されているが、要求されたものとは異なる場合：オフラインの統合インデックスは、組み込みの赤/NIR正規化差分であるため、 そのため、設定されたインデックス式は評価されません。これにより、`success` `False` となります。 |
| `combined` | 結合インデックスの概要： 使用されたバンド、LUT範囲、構築されたフレーム。 |
| `clamp_masks` | フレームごとのフレームごとのキャリブレーション済みTIFF（`save_tiffs=True`）の[アンミックス・クランプ・マスク](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on)のパス。これらは、TIFFの`<level>/`フォルダ内の`clamp_masks/<level>/`に格納されており、ピクセルがクランプされたTIFFごとに1つずつ存在する。動画には含まれない。 |
| `clamp_fractions` | マスクの代わりに動画が示す情報：動画製品ごとに1つの`{"kind", "serial", "level", "frames"}` （`kind` `"combined"` は、統合インデックスを構成する各カメラ用）、`frames` は `[frame, R, G, B]` のリストであり、そのフレームにおいて、アンミックス処理が R / G / B 各チャンネルが保持する帯域内でクリップしたピクセルの割合を表す (`Chloros:Band{R,G,B}UnmixClampFraction`)。 |

#### `RecorderHandle`

`ArraySession.record()` および `ArraySession.burst()` によって返されます。スコープの終了時に自動的に停止させるコンテキストマネージャーとして使用するか、手動で操作してください。

| メンバー | 説明 |
| --- | --- |
| `job_id` | バックエンドジョブ ID (文字列)。 |
| `kind` | `"composite"` （`record` 由来）または `"raw"` （`burst` 由来）。 |
| `start_stats` | `start` の呼び出しによって返される辞書。 |
| `result` | 実行中の `None`；停止後の最終的な停止結果ディクショナリ。 |
| `stats(timeout=10.0)` | ジョブのリアルタイム統計（書き込まれたフレーム数、実測フレームレート、経過）。 |
| `stop(timeout=60.0)` | レコーダーを停止します。最終結果を返してキャッシュします（停止要求自体が失敗した場合は、`.result`は`None`のままとなります）。 `record()` の記録が失敗した場合（フレームが書き込まれなかった場合（`error` `"no-frames: …"`） — 結果は `.result` にキャッシュされ、その後 `ChlorosConnectError` を発生させます。冪等性があります（2回目の呼び出しでは、例外を発生させることなくキャッシュされた結果が返されます）。 `with` ブロックをそのままにすると、例外を発生させずにレコーダーが停止するため、その後に `rec.result.get("error")` を確認してください。 |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

`ArraySession.continuous()` によって返されます。コンテキストマネージャーとして使用してください — `with` ブロックを離れると、キャプチャがまだ実行中の場合は停止します — または、`status()` / `wait()` / `stop()` を使用して制御してくださいで制御してください。一度に実行される連続キャプチャは、配列ごとに1つです。

| メンバー | 説明 |
| --- | --- |
| `start_status` | 開始呼び出しによって返されるステータス辞書。 |
| `result` | 実行中の `None`；キャプチャ終了時の最終ステータス辞書（`stop()` によって設定される、 または終了を確認した`status()` / `wait()`によって設定される）。 |
| `status(timeout=10.0)` | ライブステータス辞書（以下のフィールド）。 |
| `wait(timeout=None, poll_s=0.5)` | キャプチャが自然に終了するまで（`max_captures` または `duration_s` に到達、配列が消失、ドライブの容量がいっぱい、エラーなど）ブロックし、最終ステータスを返す。 `timeout` を使用する場合、実行中であっても指定された秒数経過後の最新のステータスを返す：`running` を参照。 |
| `stop(timeout=30.0)` | キャプチャを停止し、その最終ステータスを返します。冪等です（2回目の呼び出しではキャッシュされた結果が返されます）。 |

ステータスフィールド：

| フィールド | 意味 |
| --- | --- |
| `running` | キャプチャの実行中は True。 |
| `captures` | 同期されたグループが保存されました。 |
| `files` | ファイルが書き込まれました。 |
| `failed_files` | 書き込みできなかったファイル（[`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed) と同様）. |
| `failed_captures` | キャプチャ全体として失敗した試行（キャプチャが再試行したケースを含む。アレイに新しいグループが間に合わなかった際に発生したHTTP 503など）。 |
| `skipped_groups` | アレイが公開したが保存されなかったグループ（キャプチャが処理に遅れをとった）。 |
| `captures_per_s` | 開始以降の平均保存率。 |
| `elapsed_s` / `started_at` | これまでの実行時間（秒）；開始時刻（Unix エポック秒）。 |
| `last_group_seq` | 保存された最新の同期済みグループ。 |
| `last_error` | 最新のエラー、または `None`。 |
| `stop_reason` | 終了した時点：`count`（`max_captures`に到達）、`duration` （`duration_s`に到達）、`stopped`（`stop()`が呼び出された）、`array_gone`（配列が切断された）、`error`（20回連続の失敗、またはバックエンドが拒否したリクエスト）、`disk_full`（ドライブに空き容量がないファイル）。`None` 実行中。 |
| `max_captures` / `duration_s` | 起動時の境界（`0` = なし）。 |
| `defender_cores` | 過去 10 秒間のウィンドウで Microsoft Defender が使用した CPU コア数。`None` は測定値が得られるまで、Defender がインストールされていないホストでは常に `None` となります。 |
| `defender_warned` | キャプチャが、Defender によって処理が遅延していることを記録した時点で（1 コア以上で）True になります。バックエンドを Defender のスキャン対象から除外してください — [ステップ 1](#step-1--install-the-chloros-platform-package) を参照してくださいを参照）。 |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### 独自のパイプラインでのライブフレーム

同期されたフレームを独自のコード（機械学習、カスタムビューア、独自のレコーダーなど）に供給するには、`ArraySession.frame_stream()`を使用します。各ティックごとに、`GrabResult`が生成されます： 各カメラごとに `{serial: numpy array}` が生成され、**すべて1つのトリガーから**取得され、そのトリガーのIDは `.group_id` に格納されます。これはプロジェクト API の `ArrayHandle.frame_stream()` と同じ実装であり、 したがって、[読み取りごとに1つのトリガー](#one-trigger-per-read--group_id)、[`band`](#exporting-a-single-band--band)、[`align`](#aligning-the-streams--align)、および [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) の内容は、両方に適用されます。

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **生成される各セットは新しいトリガーとなります。** `frames.group_id`はストリーム内で決して繰り返されません。配列の実行速度よりも多くの`fps`を要求した場合、ストリームは次のトリガーを待機するため、2つのうち遅い方の速度で実行されます。
- **欠落したカメラは名前で指定され、バックフィルは行われません。** そのトリガーで配信できなかったカメラは辞書から除外され、`frames.errors`にリストされます。
- **終了時は目立つ処理が行われます。** 拒否された場合、次のティックで `FrameFetchError` が繰り返し実行されます。また、60 秒間フレームがない場合、または 60 秒間新しいトリガーがない場合にも繰り返し実行されます。**提供される機能。**`frame_stream()`は、numpyフレーム全体をローカルHTTP接続経由で転送し、最大で**170 MB/s**近くの転送速度を達成します。 これは、生フレームやビンニング済みフレーム、ROI配列のリアルタイム処理には十分ですが、フル解像度での**すべての**トリガーを処理するには不十分です。**すべての**トリガーを保持するには、 [`continuous()`](#continuoushandle) を使用してください。パケット転送を用いた4台のカメラアレイ（FOCN、FRGN、FRGB、F450）で測定。各ティックは1トリガーセットの完全なデータでした：

| | フルセンサー（アレイ 14.2–14.4 fps） | 2 × 2 ビニング（アレイ 約33 fps） |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6.7セット/秒 | 22.2セット/秒 |
| `frame_stream(processing="debayered")` | 2.3 セット/秒 | 9.3 セット/秒 |
| `frame_stream(processing="display")` | 0.8 セット/秒 | 3.0 セット/秒 |
| カメラごとの MJPEG、 720pで4ストリーム | 各13.5 fps；すべてのフレームがトリガーと一致 | 各約25 fps；4ストリームすべてでトリガーが存在する割合は約45％ |

スループットとデモザイク処理を側で行う場合は `processing="raw"` を使用してください。`debayered` レベルでは、キャリブレーションパックのダークオフセットも除去されますが、 これに対し、`raw`のデモザイク処理では除去されません。

#### カメラごとのMJPEGストリーム（RTSP非対応）

**RTSPはサポートされていません。** ストリーミングインターフェースはHTTP経由のMJPEG（`multipart/x-mixed-replace`、パートごとに1つのJPEG、各パートに`Content-Length`）で、カメラごとに1つのURLが割り当てられます：

| URL | 配信内容 |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8ビットのプレビュー、最大約30 fps。アレイのメンバーの場合、`max_h` （デフォルト 720、64–2160）および `jpeg_q`（デフォルト 80）でサイズと画質を設定します。 |
| `GET /api/camera/<serial>/stream-annotated` | 同じプレビューにGUIオーバーレイ（ゼブラ、グリッド、クロスヘア、ヒストグラム、ピーキング、スポットメーター）が描画されたもの。`ChlorosLocal.camera_stream()`がこれを読み取ります。 |
| `GET /api/camera/<serial>/stream-reflectance` | バンドごとの8ビット反射率 — [ライブ反射率ストリーム](#live-reflectance-stream-mjpeg)を参照。 |

- **トリガーヘッダー。** 配列の要素については、3つのストリームすべての一部が `X-Chloros-Group-Id`、 `X-Chloros-Frame-Id`、および `X-Chloros-Timestamp-Ns` が含まれます。異なるカメラからのパーツであっても、同じ `X-Chloros-Group-Id` を持つものは同一のトリガーとなります： パーツを収集し、そのヘッダーに基づいてグループ化してセットを再構築します。スタンドアロンカメラにはグループがありません。そのプレビューパーツにはこれらのヘッダーは含まれておらず （その反射率パーツには空の `X-Chloros-Group-Id` が付与されています）。
- **プレビューストリームは負荷がかかるとフレームをドロップします。** 最新のフレームを配信し、時間内に送信できなかったフレームはスキップするため、アレイレートが高い場合、すべてのトリガーがすべてのカメラのストリームに到達するわけではありません（上の表を参照）。すべてのトリガーには、`frame_stream()` または `continuous()` を使用してください。
- **`/api/camera/stream/<serial>`は、アレイの統合ライブストリームが開いている間、一時停止します**（`/api/camera/array/stream/<array_id>` — GUIのアレイビューが開くと起動します）。その後、各メンバーのストリームは、`X-Chloros-*`ヘッダーを含まない「Paused (array stream active)」というプレースホルダーフレームを1つ送信し、 統合ストリームが閉じられるまではそれ以上の送信は行われません。その後、自動的に再開されます。`stream-annotated`および`stream-reflectance`は一時停止しません。スクリプトがカメラごとのプレビューストリームを読み取る際は、GUIのアレイビューを閉じてください。
- **画像ではなく数値を取得する場合は、`frame_stream()`を使用してください。** JPEGは非可逆圧縮で8ビットです。パスにコーデックが含まれていない場合、`frame_stream()`は同じ結果を返します。

### ドローン測量におけるインターバルキャプチャ

測量飛行では、アレイが生成できるすべてのフレームではなく、数メートルごとに1つの同期された画像グループが必要です。これを制御する設定は2つあります：**`target_fps`**はカメラがフレームを撮影する頻度を設定し、**`capture_repeated(interval_s=…)`** はフレームが保存される頻度を設定します。**カメラは保存の合間もストリーミングを継続します。** 各インターバルで最新の同期済みグループが保存されますが、その間のフレームは書き込まれません。自動露出はフレームごとに調整されるため、保存が行われる時点では露出はすでに安定しています。撮影の合間にカメラが停止し、撮影のたびに起動するモードはありません。そのようなモードでは、すべての撮影でストリームの開始と露出の安定を待つ必要が生じてしまうからです。**フレームレートは、間隔に合わせて固定してください。**自動のままにすると、アレイはホストとネットワークが許容する最高レートで動作します。レートを低く固定すると、保存内容を変えることなく、カメラの発熱、ネットワーク負荷、ホストのCPU負荷を低減できます。**1間隔あたり2～4フレーム**となるレートを選択してください:

- 保存処理はリクエスト後、次のフレームが到着するまで待機するため、保存されたグループは予定時刻から最大1フレーム分遅れて到着します。レートが高いほど、この遅れは小さくなります。
- 自動露出は1フレームごとに調整されるため、レートが極端に低いと、通り過ぎる雲に対する露出の追従が遅くなります。

**フライトプランから間隔を算出してください：**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

例えば、前方オーバーラップ率80％で60 mの撮影範囲の場合、間隔は12 mとなります。速度10 m/sでは、`interval_s=1.2`となり、 固定された`target_fps=3`の場合、1インターバルあたり3～4フレームとなります。撮影は時間ベースで行われます。SDKには距離やGPSによるトリガーモードがないため、測量ライン上では一定の対地速度を維持してください。

**生データを機体上で保存し、後でキャリブレーションを行ってください。** `interval_s`は各保存の開始時点から計測されるため、保存に間隔時間より長くかかると、次の保存の開始が遅れ、間隔がずれてしまいます。 生データに加え、光センサーの測定値（`processing="raw"`；この測定値はデフォルトで保存されます）を保存することで、各保存処理を短く抑え、後で各フレームを反射率データに変換できるようにします。

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**は開始時からレートを固定します。**`arr.set_target_fps(fps)`**は再接続せずに飛行中にレートを変更し、**`arr.set_target_fps(None)`**はレートを自動設定に戻します。ホストおよびネットワークが処理可能な上限を超えるレートは、その上限値まで引き下げられます（結果の`clamped`を確認してください）。最低レートは0.1 fpsです。
- **フレームを間引くために `burst()` を使用しないでください。** この設定では、配列が生成するすべてのフレームが出力されます。固定された `target_fps` に続くため、すべてのフレームが必要な場合にのみ適切なツールとなります。
- **`target_fps`は、破損したフレームを修復しません。** この値を下げると、生成されるフレーム数が減少しますが、各フレームの送信速度は変わりません。GVSP によるフレーム破損の場合は、代わりに `wire_ceiling_mbps` を下げてください（[アレイの健全性](#array-health--which-subsystem-is-losing-frames) を参照）。

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### すでに接続済みのアレイへの接続 — `attach_array`

アレイがすでに起動している場合 （GUIで開かれたか、以前のSDKセッションで`connect_array`が呼び出された場合）、再接続する代わりに`attach_array`を使用してそのアレイへのハンドルを取得してください。その状況では、`connect_array`は常に`Camera <sn> is already in array <id>`というエラーを常に返します。これは、プール内のメンバーに対して`/array/connect`をPOSTしても冪等ではないためです。 `attach_array`は`/api/camera/array/list`を読み取り、array_idまたはserialsのいずれかで照合を行います。

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

対処法：デスクトップGUIと共用するSDKスクリプトは、まず`attach_array`を試み、プールにまだ配列がない場合は`connect_array`にフォールバックするようにしてください。

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **アタッチされたアレイは所有されているのではなく、借用されているものです。** `attach_array`からのハンドルは、アレイを解放することはありません。`with`ブロックの終了時でも、明示的な`disconnect()`による解放時でも、アレイは解放されません（明示的な呼び出しでは、stderrに1行の出力が表示され、 配列が実行されたままになっている）と表示する1行をstderrに出力する）。これは`CameraSession` / `DAQSensorSession`と同じルールである。したがって、`with`の有無にかかわらず、GUIとの共用は安全である：
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> 本当に終了させたい場合（たとえば、別の `wire_ceiling_mbps` に再接続する場合など）は、`arr.force_disconnect()` を呼び出してください。接続済みのハンドルに対して単純な `disconnect()` を実行した後に再接続しようとすると、何も解放されていないため、 `Camera <sn> is already in array <id>`というエラーで失敗します。これは、何も解放されていないためです。

### ネットワーク分析ヘルパー

配列を開く前に役立つ機能です。提案された設定が適合するかどうかを予測します：

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

バックエンドは決して起動しません。応答がない場合は `ChlorosConnectError` が発生し、リクエストが拒否された場合はその結果が返されます。不正な `packed` が発生すると、`ValueError` が発生します。

`status` は、`ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` のいずれかです（それ以外の場合は `error` — 拒否されたリクエスト（ログインしていない場合、`error_code: AUTH_REQUIRED`、またはSDKの最低基準を下回るプラン、`PLAN_UPGRADE_REQUIRED`）を含みます）。`auto_capped_fps`は、要求された解像度が上限トリガーレートでのみRXリングに適合することを意味する — 解像度を維持し、`target_fps=result["recommended"]["recommended_target_fps"]`から`connect_array`へ移行する（[例6](#6-capability-probe-before-connecting-a-4-cam-array）を参照）。`auto_shrunk`は、要求されたフレームがいかなるフレームレートでも配信できない場合にのみ表示されます。connectは独自にフレームを縮小することはないため、`recommended`は自身で適用してください。

**投影の読み方**（GUIの「アレイ設定」パネルと同じモデル）：

- **バースト（`frame_bytes_total`）は、各カメラの実際のピクセル形式でカメラごとに合計されます。**モノ**M3M**カメラは Mono12（2 B/px）をストリーミングするか、`pixel_format` を適用して、その深度のモノフォーマットをストリーミングします （`"BayerRG8"`は`Mono8`、 1 B/px）のモノフォーマットでストリーミングされます。したがって、`pixel_format`がない場合、4台のカメラによるフル解像度のフレームは、3台のモノカメラで**約25 MB**となり、すべて8ビットという仮定から導かれる約12.6 MBとは異なります。バックエンドは、各カメラのモデルからそのフォーマットを判別します。
- **アドミタンス (`burst_fits_nic_ring`) はドレインを意識している**、 バースト全体対リングという区別ではなく、ホストがRXリングをカメラがリングを埋める速度よりも速く排出する場合に、sim-emitが適用されます。10Gホスト＋1 GbEカメラの場合、バーストがリング容量を超過していてもフル解像度を**許容**します； 1 GbEホストの場合はブロックされます（`needs_force_slip` / `auto_shrunk`）。
- **`achievable_fps_max`は、シリアル取得における保守的な上限値**である — `max(readout+emit, N×emit)`では、カメラごとの送信量が1 GbEカメラリンクに制限され、露光時間には依存しない。 例：4カメラのフル解像度12ビットアレイの場合、約2.8 fps（ランタイムでの実測値である約2.7～3.0と一致）。 完全なモデル：[CLIリファレンス → アレイのfpsおよびバーストモデル](cli-reference.md#array-fps--burst-model)。
- **連続キャプチャ終了後、プロジェクションにはその間の処理状況が表示されます。** 自動レートで30秒以上キャプチャした場合、このコンピュータにおけるそのアレイサイズで安定したレートが記録されます。レートコントローラによる高速レートへの短時間の試行は除外されます。それ以降については、`array-connect`の行以降、`connect_array()`の分析および「配列設定」パネルでは、そのレートがモデルのレートを下回る場合、そのレートを引用し、`cycle_limiter`で制限要因を明記しています：`"clock"` （アレイは最終的に決定されたレートで動作し、グラブ・ループはトリガクロックを待機していた）、`"loop"`（グラブ・ループに残された時間がなかった）、 `"loop"`（グラブ・ループの残り時間がなかった）、`"capture"`（キャプチャが遅れ、グループをスキップした）、または`"measured"`（どちらであるかは明記されていない）といった状況です。モデルの数値は`wire_fps_max`に記載されています。 配列は依然として、回線が伝送できる値まで上昇する。
- **オーバーサブスクリプション（`oversubscribed: true`）とは、カメラが到達するいかなるペースでも、回線が配列を伝送できないことを意味する** — fpsフィールド（`achievable_fps_max` / `fps_bright` / `fps_dark`）の値は0となり、ビニング処理ではこれを修正できない （ビニングは1フレームあたりのバイト数を減らすだけで、1秒あたりのペーシングバイト数を減らすわけではない）。回線が単に遅いだけではこの現象は起こらない。その場合はフレームレートが低下するのみであり、カメラのペーシングを十分に遅くできない場合、ティアは`slip-emit-and-capture` （シリアル）となります。対策としては、カメラ台数の削減、ジャンボフレームの使用、またはより高速なNICの導入があります。`max_cams_collision_safe`は、回線とカメラの性能上限を示しています。 この応答には、`aggregate_demand_bps`、`collision_safe_ceiling_bps`、および `per_cam_floor_bps` も含まれます。[オーバーサブスクリプション](#over-subscription-the-per-cam-floor) を参照してください。

### 検出と一覧表示

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

それぞれがキーワードのみの引数 `backend_url=` および `timeout=30.0` を受け取り、辞書のリストを返します。 `discover_lattice_cameras()` (`serial`, `ip`, `mac`、`model`、`lattice_model`、`vendor`、`connected`、 `array_id`, …) も `auto_start_backend=True` を受け取りますが、`list_*` の呼び出しはバックエンドを起動することはありません。

---

## Smart-AE / Smart-Capture

LATTICEアレイは接続されるとすぐにバックグラウンドで連続的なAEを実行しますが、新たにターゲットを指定したシーンは収束するまで少し時間がかかります。**Smart-capture**は、この処理をパッケージ化した便利な機能です。各カメラの露出をポーリングし、ウィンドウ全体でアレイの状態が安定するまで待機してから、キャプチャをトリガーします。これはGUIと同等の機能であり、デスクトップアプリの「smart」キャプチャボタンも、同じバックエンドエンドポイントを呼び出します。

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

`ChlorosProject`（次のセクション）経由で制御する場合、さらに多くの設定項目が利用可能になります：

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

スマートAEポリシーはデフォルトで保守的です。厳密な放射測定を行う場合は`exposure_tolerance_pct`を厳しく設定し、変化の激しいシーンで「おおよそ正確」な結果が求められている場合は緩く設定してください。

---

## DAQセンサーセッション

スペクトルセンサー（USB経由のDAQ-U、BLE経由のDAQ-M、イーサネット経由のDAQ-E）用の永続的なバックエンドプール。カメラの表面を反映しています：スマート検出、プールの再利用、冪等なアタッチ。

### スマート検出（ゼロコンフィグ）

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

優先順位：イーサネット → BLE → USB。明示的なヒントを1つ指定することで、転送方式を固定できます。

### 固定転送方式

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=`は1つのセンサーを指定し、そのセンサーのみが開かれます。これは既に開かれているセンサーと照合され、その後、DAQ-Eのホスト名またはIP（ネットワークスキャンでIPのみで検出されるDAQ-Eに対しては、読み取り専用で名前、MAC、シリアル番号が問い合わせられ、 そのため、これらいずれかでも検出可能です）、DAQ-MのBLE MAC、およびDAQ-UまたはDAQ-Mがオープン時に報告するID（確認のためにオープンされた他のデバイスはすべて再度閉じられます）と照合されます。見つからない場合、 `connect_daq_sensor`は、検出された内容とアドレスを記載した`ChlorosConnectError`を発生させます（`port=` / `mac=` / `eth_host=`）が別のセンサーを保持している場合も同様に例外が発生します。表示されているセンサーのいずれかにフォールバックすることは決してありません。それこそが、zero-flag呼び出しの目的です。

### `connect_daq_sensor()` 署名

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

すでにプール内にあるセンサーはそのまま返されます（借用）：`integration_time`、`frame_avg`、および`enable_ae`は、このセンサーには適用されません。 設定を適用して使用するには、まずセンサーを取り外してください。`cap_id` は例外です。借用中のセンサーでは、`set_cap()` で設定されます。指定しない場合は、バックエンドのデフォルト値が適用されます（メソッド表の下にある注記を参照）。

### `DAQSensorSession` メソッド

| メソッド | 説明 |
| --- | --- |
| `status(timeout=10.0)` | プールエントリの概要（ストリーミング／記録状態、波長範囲、キャリブレーションSHA、積分時間、frame_avg、AE状態）。ストリームが停止したDAQ-Eの場合：`reconnect` （バックエンドの自動再接続が有効な場合：`state`、`attempts`、`next_attempt_in_s`、`last_error`；`None`はストリーム送信中）、`stream_lost_s`、`reconnects`、`last_reconnect`、および進行中の記録については 進行中の録画用；`rx_stream_fault`は、最初のスペクトルが到着するまでの再接続状態を示す。 |
| `latest(*, n=1, timeout=10.0)` | 直近のスペクトルフレームを最大 N 個まで返す（それぞれ `ts_ns`、`spectrum`、 `is_saturated`、`integration_time`、`x`、`y`、`z`）. |
| `stream_start()` / `stream_stop()` | ストリーミングの再開 / 一時停止 (ハンドルは開いたまま)。 |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | .daq ファイルの記録を開始します。ファイルパスを返します。AWS キャリブレーションバンドルがない DAQ-U/M では拒否されます（DAQ-E は例外）。 |
| `record_stop()` | 記録を停止します。`{path, rows, warning, gaps}`を返します。クリーンな記録の場合は`warning`が`None`となり、 それ以外の場合はバックエンドの診断情報（例：空のファイルやストリームの欠落など）が返され、stderrにも出力されます。 `gaps`は、ストリームから何も配信されなかった各区間を一覧表示します：`start_utc`、`end_utc`（`None`：停止時に依然として失われている）、 `duration_s`、 `reconnected`、`marked_in_file`。連続記録の場合は空欄となります。 |
| `set_cap(cap_id, timeout=15.0)` | 次の測定値以降に適用されるキャップ補正プロファイルを設定します： `"sunshine_cosine"`、`"fov_15"` … `"fov_90"`、または裸センサーの場合は `"none"`（あるいは `None`）。センサーの概要を返します。これを読み戻すには `cap_id` を使用します。DAQ-E-S は、要求内容にかかわらず `as_recorded` を保持します（ディフューザーが内蔵されているため）。プロファイルが 読み込めないプロファイルに対しては、`ChlorosConnectError`を発生させます。CLI：`chloros-cli daq pool-set-cap`。 |
| `enable_ptp(timeout=15.0)` | DAQ-Eのみ：PTPがオフになっているユニットに対して、PTPを再びオンにします （`status()["rx_ptp_enabled"]`は`False`に相当します。設定はユニットに保存されるため、再有効化されるまで無効のままとなります）。 更新されたステータスを返します。DAQ-U / DAQ-M の場合、またはユニットが確認に応答しない場合は、`ChlorosConnectError` を発生させます。Chlorosは、オフ状態が意図的なものである可能性があるため、決して独自にユニットの電源を再投入することはありません。 CLI：`chloros-cli daq pool-enable-ptp`。 |
| `device_info(timeout=15.0)` | DAQ-Eのみ：ユニット自身の設定。`firmware` (`running`、`bundled`、`update_available`)、 `ip` および `ip_mode` (`dhcp` / `linklocal` / `static`: 現在実行中のもの）、`network`（保存された設定：`mode`、および `ip` / `mask` / 静止時は`gateway`）、`network_change_pending` （ユニットが保存された設定（`set_network(..., reboot=False)`を含むもの）を実行するまでは true）、`auto_stream`、`ptp`。 `firmware["update_available"]`は、ユニットの。本機のストリームがこのコンピュータに到達していない間、新規接続時に読み取られ、辞書には `stream_silent` (`True`)、`stream_fault`、および`reconnect`が辞書に追加されます。応答しないユニットについては、CLI：`pool-info`が記録されます。 |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E のみ：DHCP (`"auto"`) または静的アドレス。設定を適用するためにユニットを再起動し、再接続します。このセッションは引き続き動作し、`address`はユニットに追従します。`reboot=False`は設定を保存するのみです。 [DAQ-E デバイス設定](#daq-e-device-settings)を参照してください。CLI：`pool-set-network`。|
| `set_auto_stream(enabled, timeout=15.0)` | DAQ-E のみ：ソフトウェアが接続されていない状態で電源投入時にストリーミングを開始します（ユニットに保存されます）。確認済みの設定を返します。CLI：`pool-set-auto-stream`。 |
| `reboot(*, comeback_timeout=45.0)` | DAQ-E のみ：ユニットを再起動し、以前の設定のまま再接続します。CLI：`pool-reboot`。 |
| `update_firmware(*, comeback_timeout=45.0)` | DAQ-Eのみ：古いバージョンを実行しているユニットに、このChlorosリリースに含まれるファームウェアを書き込み、その後再接続します（約1分間。電源を切らないでください）。 `updated`、`fw_before`、`fw_after`、`bundled`、および `warning`が返されます。また、ユニットの応答が遅かった場合や、別のバージョンで応答した場合も同様のエラーが返されます。すでに最新バージョンのユニットの場合、`updated`は`False`となります。 バージョンを読み取れないユニットに対しては、`ChlorosConnectError`を発生させます。CLI：`pool-update-firmware`。 |
| `disconnect()` | プールからの解放。-op。このセッションが開いたセンサーでは `force_disconnect()` となり、解放が確認されない場合は `ChlorosConnectError` を発生させます（`with` ブロックの終了時には は決して例外を発生させない）。 |
| `force_disconnect(timeout=30.0)` | 誰がオープンしたかに関係なくセンサーを解放する — 例えば、クラッシュしたスクリプトがプールに残したセンサーで、新しいスクリプトが `disconnect()` が何もしないアタッチされたハンドルとしてのみアクセスできる場合など。他のクライアントのセンサーを終了すると、そのセンサーによる記録はすべて終了します。バックエンドが解放を確認したとき、または解放すべき対象がない場合 （センサーがプールにない、またはバックエンドが実行されていない場合）; タイムアウトした、またはバックエンドによって拒否された解放では、`ChlorosConnectError`が発生します。CLI: `chloros-cli daq pool-disconnect --sensor-id ID`。 |

属性: `sensor_id`、`model`、`transport`、 `address`（`set_network()`によって再アドレス指定されたDAQ-Eに続くもの）、`info`、および`backend_url`。

> **キャップの名称を記入してください。** DAQ-U、DAQ-M、およびDAQ-Eにはコンデンサが付属しておらず、Chlorosはコンデンサが取り付けられているかどうかを判別できません。`cap_id`なしで接続されたセンサーは`sunshine_cosine`を取得しますが、その記録データによると、これは （`cap_id_source=auto_default`）。この補正を適用した状態のキャップなしの DAQ-U は約 30 倍の高値を示すため、キャップなしのセンサーには `cap_id="none"` を指定してください （または `set_cap("none")`）を指定し、それ以外の場合は装着したキャップのプロファイルを指定してください。

### DAQ-E-S — 固定式ディフューザーおよび傾斜センサー

**DAQ-E-S**は、ソフトウェアの観点から重要な2つの違いがあるDAQ-Eです。それ以外のすべて（検出、接続、PTP、記録、以下のデバイス設定）は、DAQ-Eとまったく同じように動作します。**ディフューザーは固定式であるため、キャップ補正は必要ありません。** サンシャイン式ディフューザーは内蔵されており、工場出荷時の校正測定時にすでにユニットに取り付けられていたため、その校正値はすでに真のダウンウェル放射照度を示しています。 Chlorosは、要求内容にかかわらず、キャップを`as_recorded`（プロファイルなし）に設定します。これと同じ名前を持つキャップ（`sunshine_cosine`、 `fov_*`、`none`）はメッセージを表示して無視され、記録データには `cap_id_source=model` が保存されるため、想定キャップに関する警告は発動しません。 手動でデータにキャップを適用することも避けてください。最上部に`sunshine_cosine`を配置すると、測定値が約11倍高くなってしまいます。

**このセンサーには3軸加速度計が搭載されています。** サンシャインディフューザーのコサイン応答は、水平な状態でのみ有効です。そのため、センサーが傾いていると、他の症状は現れず、誤った放射照度が報告され、その誤差が反射率にも引き継がれます。したがって、各 DAQ-E-S の測定値にはその姿勢情報が含まれています：

| フィールド | 意味 |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | 角度（度単位）。ファームウェアが角度を未定義（自由落下、軸の故障）と判定した場合は `None` となります。 |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | ミリg単位の加速度。 `mag_mg`が約1000から大きく外れている場合、ユニットが移動しており、そのサンプルにおける角度には意味がないことを示します。 |
| `cal_applied` | 設定済み：軸と角度はディフューザーの座標系に基づいています（ユニットの取り付け校正が適用されています）。クリア：ボードの生の座標系で、最大約2.3°の傾きが表示される場合がありますが、これは実際の傾きではありません。 |
| `sample_age_ms` | スペクトルが送信された時点での加速度計サンプルの経過時間。 |
| `temp_raw` | 校正されていないダイ温度の**傾向** — 差分のみであり、温度計の測定値そのものではありません。 |

`DAQSensorSession.latest()`は、各フレームの`imu`キー（DAQ-U、DAQ-M、または標準のDAQ-Eでは`None`）の下にこれらの値を返します。 DAQ-E-Sから記録された`.daq`は、その`imu_*`列（`imu_tilt_deg`、 `imu_roll_deg`、`imu_pitch_deg`、 `imu_mag_mg`、`imu_cal_applied`、…）に同じ値を格納するため、読者は雲と三脚の倒れを区別できます。他のすべてのモデルでは、これらの列は空欄です。

姿勢ストリームは、`status()` （`rx_imu_stream_fault`、`rx_imu_stream_faults`、`rx_last_imu_stream_fault`）の独自のフィールドで報告され、スペクトルデータとは区別されているため、加速度計の故障があっても記録の欠落として読み取られることはありません。 `discover_daq_sensors()` およびセッションの `model` には、`DAQ-E-S` と記載されています。

### DAQ-E デバイス設定

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **このコンピュータが属するネットワーク上に静的アドレスが存在する必要があります。** `set_network`は、サブネットにこのコンピュータの再起動後、センサーにアクセスして設定を元に戻すことができません。このコンピュータが新しいサブネットに再アドレス指定されるか、ルーティングされる場合にのみ、`force=True`を通過させてください。
- **`set_network`、`reboot`、および `update_firmware` はセンサーを再起動**し、記録中に `ChlorosConnectError` を発生させ、 かつ、`comeback_timeout`秒以内に復帰しない場合。センサーIDおよび`daq-e-<id>.local`ホスト名は決して変更されません。
- **再起動の直前に開始された記録は、再起動によって終了され、ファイル名が付けられます。** センサーが復帰すると、返される辞書には `recording_closed`（ファイルのパス） と、その旨を示す `warning` が含まれます。センサーが復帰しない場合、またはファームウェアの転送に失敗した場合は、代わりに `ChlorosConnectError` がその旨を示します。そのメッセージは警告で終わり、`e.recording_closed` / `e.warning`が設定されます（録画が終了していなかった場合は、両方の`None`が設定されます）。センサーが復帰したら、録画を再開してください。
- **センサー本体が再接続されるのみです。** 再起動後、ユニットは元の位置から応答する必要があります -- また、`set_network("auto")`以降は、その`.local`名でも -- 以前使用していたネットワークアダプタ（MAC）から、名前、ホスト名、モデルを回答する必要があります； `address`は応答したIPアドレスとなります。名前での応答は、そのMACアドレスでのみ有効となるため、ルーターを経由してセンサーに到達するコンピュータや、WSL2のデフォルトNATネットワーク環境からは、DHCPによって移動されたセンサーは再接続されません （`ChlorosConnectError`；新しいアドレスで再接続してください）。
- **ファームウェアの更新には、センサーが再接続する必要があります。** `update_firmware()`はユニットに接続を要求し、ユニットはOSが選択したポート上で、イメージを取得するためにこのコンピュータへTCP接続を確立します。バックエンドは、ユニットに到達するアダプタ（どのアダプタか判別できない場合はすべてのアダプタ）でリスニングを行い、そのユニットのみに対応します： 待機中は、それ以外の接続はすべて拒否されます（バックエンドの出力に `[DAQ-FW] … refused a firmware connection from <ip>` が表示されます）。接続をブロックするファイアウォールがある場合、`device never connected back on TCP <port>` が表示され、更新は失敗します。 Windowsインストーラーはこれに対応するルールを追加します（[ステップ1](#windows-exe)を参照）。別のファイアウォールを使用する場合は、センサーのサブネットからバックエンドへのTCPインバウンド通信を許可してください。
- **WSL2 環境では、ミラーリングネットワークを使用してください**（`networkingMode=mirrored` および `.wslconfig`）：デフォルトの NAT モードでは、すべての静的アドレスが拒否されます（WSL 自身のアドレスはセンサーのネットワーク上にありません）ため、ストリームが WSL に到達せず、ファームウェアの更新が完了しません。
- GUI はセンサーとの独自の接続を維持します。スクリプトによる再起動を行うと、GUI のセッションは終了します。GUI には同じ設定（[Light Sensors] タブ、センサーの設定、[Network]）が適用されます。

### 検出 — 接続先のアドレスを見つける

`discover_daq_sensors()`は、USB / BLE / ETHをスキャンして、*開くことのできる*センサーを探します。これは`discover_lattice_cameras()`のDAQ版であり、**DAQ-MのBLE MAC**を取得する唯一の方法です — DAQ-Eにはホスト名があり、DAQ-UにはCOMポートがありますが、MACアドレスはデバイスに印字されておらず、OSにも表示されません。

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| フィールド | 説明 |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COMポート / BLE MAC / ホスト名 — `connect_daq_sensor` へ `port=` として渡す / `mac=` / `eth_host=`。 |
| `display` | 人間が読み取れるラベル。 |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`、 または `None`。イーサネットセンサーは、スキャン中に問い合わせがあった際に自身を報告する（読み取り専用）ため、`DAQ-E-S` または `DAQ-E` となります （時間内に応答しない場合は `DAQ-E`）。USBシリアルポートは、そのブリッジチップがDAQ-Uで使用されるもの（FTDI、CH340、CP210x、Prolific）である場合、`DAQ-U`と読み取られます。これはポートの説明から得られるヒントであり、 プローブではない）——その他のUSBデバイスでは`None`と読み取られますが、これらは依然としてリストに載っているため、`port=`として渡すことができます。 USBベンダーIDがなく、そのようなブリッジチップも搭載していないシリアルポート（Linuxの`/dev/ttyS*`のようなマザーボードのUART、Bluetoothシリアルリンク、Intel AMTなど）はDAQ-Uには該当せず、リストには含まれません。 |
| `extra` | トランスポートごとの詳細 (BLE アドバタイズ名；USB `vid`、`pid`、`manufacturer`、`serial_number`；DAQ-E ip/fw/…）。空の値は省略されています。 |

| パラメータ | デフォルト | 説明 |
| --- | --- | --- |
| `transports` | 3つすべて | スキャンを制限するシーケンス（またはCSV文字列）。目的が明確な場合に指定すると有効です — BLEは処理速度が遅いため。 |
| `scan_timeout` | 5 | トランスポートごとのスキャンウィンドウ（秒単位）。バックエンドにより1～20に制限されます。 |
| `timeout` | 60.0 | 呼び出し全体に対するHTTPの上限（SDKの他の箇所と同様）。 |
| `auto_start_backend` | `True` | 実行中のローカルバックエンドがない場合、ローカルバックエンドを起動します。リモートの `backend_url` に対しては決して起動しません。 |

> **プール内で既にオープンされているセンサーは表示されません。** 接続済みの BLE 周辺機器はアドバタイズを停止し、オープン状態の COM ポートはプローブできないため、ディスカバリーでは *接続可能な* デバイスのみが一覧表示されます。 何かを接続した直後に結果が空になるのは想定内です。すでに保持しているデバイスについては、`list_daq_sensors()` を使用してください。スキャンを実行できないトランスポート（bleak / zeroconf がインストールされていない場合）は、例外を発生させるのではなくスキップされるため、Bluetooth がないマシンでも USB および ETH の応答が取得できます。

### リスト

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### どのDAQがどのカメラに接続されているか — `daq_bindings()`

キャプチャの反射率および保存される `.daq` は、そのカメラまたはアレイにバインドされた DAQ を使用します。それ以外の場合は、バックエンドが持つ最新の DAQ が使用されます。GUI での光センサーの選択および `chloros-cli daq pool-bind` によって設定されます。 また、以下も同様です：

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()`は`{"cameras": {serial: b}, "arrays": {array_id: b}}`を返します。各`b`には、`binding`（設定された値）、`source`（`project`、 プロジェクトの `sensors.json` または `runtime` からのもの）、`known` / `known_as` （カメラまたは配列のキー名が、Chlorosが知っているものかどうか、およびその方法：`connected`、`project` — 開いているプロジェクトの `cameras.json` — あるいは、カメラの場合は `seen`：このコンピュータで以前に開かれたもの；何も指定していないキーの場合は `known: False`）および `resolved` （現在マッピングされているプールセンサー：`in_pool`、`sensor_id`、 `address`、`model`、`streaming`）。ランタイムバインディングは、セッションのプロジェクトを上書きします。プロジェクトファイルは変更されません。カメラ、アレイ、またはDAQの接続を解除すると、誰が設定したかに関わらず（アレイの場合はそのメンバーによる設定も含む）、そのハードウェアへのすべてのバインディングが解除されます。

`set_daq_bindings()`は、接続されているカメラ、開いているプロジェクト内にあるカメラ、または以前にこのコンピュータで開かれたことのあるカメラ、および接続されているアレイ、開いているプロジェクト内にあるアレイ（あるいはすでにバインドされているキー）のみをバインドします。それ以外のキーは、そのキー名を指定して`ChlorosConnectError`を発生させます （`400`、`code`、`UNKNOWN_BINDING_TARGET`、および`unknown`キー）という名前のエラーを発生させますが、その呼び出しでは**何も**バインドされません。 `None`は、すべてのキーをクリアします。

### 周囲光 + 対象物の反射率 — `light_sensor_reflectance()`

2つのプールされたDAQ（1つは空（アンビエント）を、もう1つはターゲット（オブジェクト）を下向きに観測）により、最新の測定値からターゲットの反射率と植生指数が得られます。GUIの「アンビエント + オブジェクト」ペアおよび`chloros-cli daq reflectance`を使用すれば、センサーを再度開くことなくこれらの値を取得できます。 両方が接続され、データストリームが送信されている必要があります。

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **バンド。** 各バンドの反射率は、対象物のバンド- 積分した放射照度です。`daq_band_presets(filter_code=None)`にはバンドセットが一覧表示されています：LATTICE M3Cフィルターごとに1つ（`RGN`、`OCN`、 `NGB`、カメラ固有のバンドモデル。したがって、DAQ NDVIとカメラ NDVIは同一の量となる）および`GENERIC`の広帯域です。 その`suggested`は、`filter_code`と一致するプリセットであり、そうでない場合は最初に接続されたカメラのフィルター （`suggested_from`がそれを示す）。
- **戻り値** `bands` （帯域ごとに `rho`、`valid`、`reason`、`e_ambient`、 `e_object`）、`indices`（組み込み-イン）、`custom`（それぞれに`value`、または`valid: False`および`error`を伴う）、`invalid_bands`、`reflectance_type`、`preset`、およびこれら2つの測定値である`timestamps` （`ts_ns`）の2つの測定値。環境センサーが測定できなかった範囲（測定範囲外、ノイズフロア）のデータは無効となり、その理由が明記される。また、そのデータが必要なすべての指標は除外され、ゼロを基準に計算されることは決してない。
- **見かけ値または相対値。** スケール依存の指標（SAVI、OSAVI、EVI、MSAVI2、RDVI、TDVI、LAI、NLI、MNLI、FCI、GEMI）は絶対比を必要とするため、両方のセンサーが電力校正されている場合にのみ算出される （各センサーに `calibration_bundle_sha` が存在する場合）：`"Apparent Reflectance"`。そうでない場合、結果は `"Relative Reflectance"` となり、スケール不変の指標のみが含まれます。
- **2つの異なる波長グリッド**の重なり部分について比較が行われます。つまり、対象物の測定範囲内にある周囲センサーの測定点と、それらに補間された対象物のスペクトルです。
- **2つのセンサー、決して1つではない。** 1つのセンサーを両方として指定すると（プールが受け入れるあらゆる表記での同一ID）、`ChlorosConnectError`が発生する：センサー自身の測定値はすべてのバンドで1.0となり、何も測定されない。 バックエンドもこれを拒否します（`POST /api/daq/indices`は`400`に応答し、 `code` `SAME_SENSOR` に対し、SDKが送信する `ambient_sensor_id` および `object_sensor_id` は、1つのプールされたセンサー（例：その IDとそのポート）です。

### CLIおよびその他のスクリプトとの共用

バックエンドのDAQセンサープールは、そのマシン上のすべてのCLIおよびSDKクライアントで共有されます。センサーがすでにプール内にある場合（`chloros-cli daq pool-connect`や他のスクリプトによって開かれている場合）、Pythonから`connect_daq_sensor(port="COM3")`を呼び出すと、借用されたハンドルが返されますが返されます。その `disconnect()` は何もしないため、スクリプトがスコープを終了する際に、そのセンサーを開いていたクライアントからセンサーを奪い取ることはありません。 同じことが、`sensor_id=`（`:` または `-` と指定された場合、どちらでも一致）についても当てはまります。これは、ホスト名で開かれた DAQ-E については、ホスト名で開かれた際に IP アドレスで指定された場合（またはその逆の場合）、およびバックエンドがすでに保持しているセンサーで応答した際のゼロフラグのスマート検出についても同様です。`with` ブロックは、自身で開いたセンサーのみを解放します。

ChlorosのGUIは、このプールの一部**ではありません**。そのDAQパネルはセンサーへの独自の接続を実行するため、`connect_daq_sensor`がGUIが開いているセンサーを借用することは決してありません。 GUIとスクリプトで同時に同じセンサーを使用しないでください。GUIが保持しているDAQ-UまたはDAQ-Mは通常、開くことができませんが、 一方、DAQ-Eは2つ目のセッションを受け入れます。スクリプトのconnectがGUI下でユニットの取得設定を変更し、その`disconnect()`がユニットのストリームを停止させるため、GUIのライブビューおよび行われている録画がすべて終了します。まずGUIでセンサーを閉じてください。

### PTPグランドマスター — `time_sync_status()`

バックエンドは、LATTICEカメラおよびDAQ-Eユニットがタイムスタンプの取得元とするPTPグランドマスターを実行します。これはオンデマンドで起動します：`connect_camera()`、 `connect_array()`、`connect_daq_sensor()`、および `connect_all()` は、PTPグランドマスターが実行されていない場合にこれを起動します — たとえば、`connect_all()`が存在しないDAQ-Eへの接続を試みて失敗する場合など — 同様に、`test_array_sync()`や、1つのコマンドのためにカメラを開くすべての呼び出しも同様です （`capture_cameras_transient()`、`start_multi_camera_stream()`、アライメント呼び出し、`capture_live_index()`）。

- **通知先。** カメラに接続されている有線ネットワークアダプタ上（`169.254.x.x`のリンクローカルアドレスを含む）、それ以外の場合はデフォルトルートのアダプタ上（そのアダプタが有線接続の場合）。Wi-Fiや仮想アダプタ（Tailscale、WSL）上では決して行われない：それらの背後にはカメラやDAQ-Eが存在しないため。 そのようなアダプタが起動していない場合、何もアドレスのない状態でアナウンスを実行し（`health` `NO_SOCKETS`）、次にカメラやDAQが接続されて起動したアダプタにバインドします。
- **リンク切断後。** カメラアダプターがリンクを失い、再びリンクを回復した場合（ケーブルの抜き差し、スイッチの交換、USBアダプターの再認識など）、グランドマスターは再起動することなく、独自にソケットを再構築し、 接続されているすべてのDAQ-Eを再チェックし、接続が途絶えていたものを再起動します。また、サービス対象のすべてのカメラおよびDAQ-Eが15秒間応答しなくなった場合にも再構築を行います。
- **通信は正常に行われていますか？** `time_sync_status()`（`GET /api/time-sync/status`）を参照してください。`state`はグランドマスタースレッドが実行されていることのみを示し、`health`はそのパケットが送信されているかどうかを示します： `OK`（異常なし）、`STOPPED`、`NO_SOCKETS`、`SENDS_FAILING`、 `DEGRADED`（一部のアダプタが故障している）または `PEERS_SILENT`（サービス対象のカメラおよび DAQ-E が応答を停止した）、そして `health_detail` に説明文が表示されます。 `announce_sent` / `sync_sent` / `delay_resps_sent`は、送信されたメッセージのみをカウントし、 `announce_send_errors`、`sync_send_errors`、`follow_up_send_errors`、`delay_resp_send_errors`、`last_send_error`、 `failing_interfaces` および `last_delay_req_age_s` は送信に失敗したものを、`rebinds` / `last_rebind` は再送信をそれぞれ表します。 `sync_sends_slow`は、送信に200マイクロ秒以上かかった同期をカウントし、`sync_send_max_us`は最も時間がかかったものです。各Follow_Upの発生時刻は、その送信に要した時間のうち最大200マイクロ秒までカウントされます。 グランドマスターは独自のプロセス（`own_process`、`process_pid`）で実行され、Windows上では高優先度で動作します（`process_priority`）。 そのため、負荷の高いバックエンドがタイムスタンプの付与を遅らせたり、すべてのカメラのクロックを一度に同期させたりすることはできません。そのプロセスを開始できない場合は、以前と同様にバックエンド内で実行されます（`own_process` false）。

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| 関数 | 機能 |
| --- | --- |
| `time_sync_status()` | グランドマスターの状態、健全性、クロック識別子、優先度、およびメッセージカウンタ（上記）。`chloros-cli time-sync status`。 |
| `time_sync_peers()` | 受信しているPTPスレーブ （カメラ、DAQ-E）：`peers`、各`port_identity`、`last_seen_ip`、`last_delay_req_age_ms`、 `delay_req_count`。`time-sync peers`。 |
| `time_sync_cameras()` | 接続された各アレイカメラの `PtpStatus`、 `PtpOffsetFromMaster`、および `PtpMeanPathDelay`（ファームウェアに PTP ノードがないカメラはシリアル番号のみ）。 `time-sync cameras`。 |
| `restart_time_sync()` | グランドマスターを停止して再起動します。その後、プールされたすべての DAQ-E の PTP が再確認されます（`daq_ptp_rechecked`）。 復旧しない場合に`ChlorosConnectError`を発生させる。`time-sync restart`。 |
| `set_time_sync_priority(priority1=None, priority2=None)` | 別の PTP マスターが存在するネットワークにおける、その PTP 優先度（0～255、値が低いほどベストマスター選出で有利）。PTP マスターが稼働している必要がある。`time-sync set-priority`。 |

それぞれが `backend_url=` および `timeout=` を受け取り、実行されていないローカルバックエンドを起動します。

---

## カメラユーティリティ

`chloros-cli lattice`が、接続やキャプチャ以外にも行う処理は、`chloros_sdk`（モジュール `chloros_sdk.camera_utils`）内の単純な関数として実装されています。それぞれが、CLIコマンドが使用するバックエンドルートを、CLIのルールに従って呼び出すため、 したがって、両者の動作は同様です。これらはセッションではなくカメラの**シリアル番号**を受け取り、スマートコネクトの呼び出しと同じ `backend_url=` / `timeout=` キーワードを使用します。 `connect_camera()`と同様に、実行されていないバックエンドが起動されます。拒否された場合は、バックエンドの理由を伴う`ChlorosConnectError`が発生します。

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| 関数 | 動作内容 |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | ディスプレイの色プロファイル： `raw`、`linear`、`natural`、 `enhanced` または `custom_temp`、およびその色温度（2000～10000 Kに制限される。プロファイルが `custom_temp` の場合に読み取られる）。 |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | 表示の彩度とコントラスト、それぞれ0～200％（100＝変更なし）; `reset=True` は両方を 100 に戻します。 |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | ディスプレイのレンダリングにおいて、最も明るく、最も白いピクセルを以下のように引き下げます：-100..0（ゼロ側に切り捨てられ、その後クリップされます； 0 = オフ）。 |
| `camera_white_balance_once(serial, settle_s=None)` | ワンショットホワイトバランス：カメラが短時間ストリーミングを行い、収束した後、`red_ratio` / `blue_ratio`（報告できない場合は`ratios_read` False）。算出したホワイトバランスを維持し、openが書き込んだその他の設定はすべて、検出されたままの状態で返される。 |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` （連続）または `"off"`；あるいは固定の `red` / `blue` バランス比（これらはまず自動機能をオフにする）。 カメラが拒否する比率は、`errors`の下に名前が付けられています。|
| `camera_presets()` / `apply_camera_preset(serial, preset)` | カメラのプリセットであり、いずれかを適用すると、カメラはプリセット名が示す設定を維持します （応答は`settings`となり、これはカメラがその後読み取る値、例えば動作フレームレートなどです。プリセットで指定されていないフィールドの場合は`None`となります）。また、open関数で書き込まれたその他の設定はすべて、元の状態に戻されます。 `"default"`は、カメラを工場出荷時のデフォルト設定（フリーランニングの自動露出、リンクスループット制限および欠陥補正がオフ、RGBカメラのキャリブレーション済みホワイトバランスは保持）にリセットしますが、メモリには保存されません： 応答には `factory_defaults: True` および `settings` のすべてのフィールドが含まれます。 `camera_presets()`において、`default`は`{factory_defaults: True, description}`となります。 |
| `camera_filter_models()` / `camera_filter_model_info(name)` | フィルターモデル、および1つのモデルの詳細（バンド、分光感度条件数）。カメラのフィルターモデルはモデル文字列から読み込まれ、手動で設定されることはありません。 |
| `camera_firmware_status(serial=None)` | 各カメラのファームウェア：`current_version`、`available_version`、`update_offered`、`needs_minimum_update`、および `minimum_firmware`。 |
| `update_camera_firmware(serial, force=False)` | このChlorosビルドのファームウェアを1台のカメラに書き込みます：工場出荷時設定へのリセット、書き込み、再起動を行い、Chlorosのベースライン設定が書き換えられます。 カメラのキャリブレーションは保持されます。数分間かかります。この間、カメラの電源は入れたままにしておく必要があります。`force=True`は、現在のカメラに再書き込みを行います。 |
| `camera_power_probe(serial)` | カメラの電源および熱ノード、ならびにその温度。 |
| `sleep_camera(serial)` / `wake_camera(serial)` | カメラの低電力アイドル状態（ウェイクアップされるまでスリープ状態を維持： リンクスループット制限が有効化されている場合（`DeviceLinkThroughputLimitMode` `On`）は下限値に設定され、カメラに省電力モードがある場合はそのモードが有効化されている）、およびその前後の温度； ウェイクアップ時には、スリープ状態での設定（制限値、モード、およびパケット遅延 `GevSCPD`。制限値が有効な間、カメラはこの値を0に保持する）がリンクに復元される。この記録はこのコンピュータに保存されるため、後続のバックエンドからのウェイクアップでもその状態が維持される）、 あるいは、このコンピュータがスリープ状態にしなかったものの、スリープ状態に見えるカメラ（リミットがフロアで有効）については、リミットを無効にし、`GevSCPD`を80（工場出荷時の設定）で0のままにします； 下限値より上に設定されたリミットはそのまま残されます。保存されるのは電源状態のみであり、それ以外の設定については、open関数が書き込んだ値が検出されたままの状態に戻されます。 |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | MAPIRキャリブレーションターゲットが視野に入るまでフレームをキャプチャし、そのパネルからチャンネルごとの係数を適合させる：`target_id`、`panel_dns`、 `coefficients`。 |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | このコンピュータの GigE Vision 用イーサネットアダプタ（MTU、リンク速度、ジャンボフレーム）および `issues`； `estimate=True`は、`cameras`カメラが取得するフレームレートと帯域幅を追加します。 |
| `fix_network_settings()` | このコンピュータのカメラアダプタに適用します。これによりアダプタの設定が変更されますが、Chlorosが管理者として実行されている場合にのみ機能します。そうでない場合は、検出結果のログを表示して終了します。 |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | フォルダ内の画像に対して、各画像ごとに独自のレンズのフラットフィールドマップ（`processed`、 `lens_models`（各ファイルの補正に使用されたレンズ）、`skipped`（補正理由）を指定します。`lens_model`を指定しない場合、各ファイルのレンズはEXIFの「Model」フィールドから取得されます； `lens_model`は、モデル名にカメラ名が指定されていないファイルに適用され、モデル名がこれと矛盾するファイルはスキップされます。ファイルのみ対象です。 |
| `read_dls(mac, filter=None, camera_serial=None)` | DLS（Bluetoothダウンウェル光センサー）からの1回の測定値： スペクトル、バンドごとの放射照度、およびカメラ固有の応答曲線に基づくバンドごとの輝度（`camera_serial`が指定された場合）。`discover_daq_sensors(transports=["ble"])`はMACアドレスを検出します。 |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | ホストのカメラ用ハードウェアと推奨設定（`report_text`）、およびシステムのCPU／GPU／PyTorchランタイム。 |

- **カメラを独占する必要があるものもあります。** `camera_white_balance_once()`、 `apply_camera_preset()`、`sleep_camera()` / `wake_camera()`、`calibrate_camera_with_target()`、および `update_camera_firmware()` は カメラを開きます。一方、Chlorosがカメラを保持している間（アプリで接続中、`connect_camera()`として配列のメンバーとして、または現在別のコマンドによって使用中、 （配列テストや、カメラを開くキャプチャなど）バックエンドはHTTP 409 `CAMERA_HELD`で拒否し（`code`および`error_code`の両方で）、その解放方法を指定します。色設定、ハイライト、`camera_power_probe()`、および読み取りは、接続されたカメラで機能します。
- **色、ハイライト、ホワイトバランスはRGBカメラ用です。** シングルバンドモノクロ（M3M）カメラでは、CLIと同様に、これらの設定関数は何も変更せず、`{"skipped": True, "reason": ...}`を返します。これらは表示設定であり、ライブビューや表示レベルのキャプチャに適用されるもので、放射輝度や反射率には一切影響しません。

---

## 一時配列、アライメントプロファイル、およびインデックス

`chloros-cli lattice`が永続的な接続がない場合に実行する処理は、`chloros_sdk`（モジュール `chloros_sdk.array_utils`）内の関数：`continuous`、`multi-capture`、`multi-stream`、`multi-test`、`multi-detect`, `analyze-array`、`align-calibrate`、`align-apply`、`align-stream`、`align-info`、 `align-reorder` および `index`。それぞれが、CLI コマンドが送信するリクエストを、CLI のデフォルト設定で送信します。

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **これらはカメラ自体を開きます。** 各コマンドはカメラを開き、処理を実行して閉じます。そのため、Chlorosが保持していないカメラ（アプリに接続されているもの、 `connect_camera()`として、または配列の要素として）カメラを保持していない必要があります。バックエンドはHTTP 409 `CAMERA_HELD`で拒否し、カメラを解放する方法を示します。一時的なオープンには時間がかかります（数台のカメラで約40秒）。繰り返しキャプチャを行う場合、 `connect_array()`を1回実行するのが最速のパスであり、一時的なキャプチャでデベイヤー処理済みの画像が保存されるすべてのエクスポート形式で時間を節約できます。`calibrate_alignment_profile()`は例外です。これらのカメラで構成された接続済みアレイの場合、アレイ自身のフレームから処理を行います （`used_live_array`）から解決します。
- **`serials`** は開くカメラのリストであり、最初のカメラがトリガーを起動します。`None` はネットワーク上のすべてのカメラを開きます。
- **`settings`** は、カメラ設定の上書きを行う辞書であり、CLIのフラグによって設定されるフィールドです：`exposure_time` (µs)、 `gain` (dB)、`pixel_format`、 `width`、`height`、`target_brightness`、`ae_damping`、`ae_upper_limit`、 `frame_rate`、`gev_packet_size`、および `preset`（名称によるカメラプリセット）。指定された露出値やゲイン値は、フラグと同様に自動制御を無効にします。
- **`gpu`**: `True`はGPUワープを強制し、`False`はCPU （`--gpu` / `--no-gpu`）を強制します。`None`はバックエンドに選択を任せます。
- **`keep_settings`**/**`clear_settings`**（カメラを開くすべての関数、デフォルトは `False`）: 処理の最後に、各カメラは検出時の状態に戻されます。`keep_settings=True`は、実行時の設定をカメラ自身のメモリに保存します（フリーランニング）。 `clear_settings=True` はそのメモリを工場出荷時の設定にリセットします — [カメラのメモリ](#camera-memory--keep_settings--clear_settings)を参照してください。 応答（ジョブに対するもの、`stop()`の）には、`camera_memory`が含まれます。
- **パスはバックエンドのものです**：バックエンドがこのマシン上にある場合、相対パスは作業ディレクトリを基準に絶対パスに変換されます。アライメントプロファイルファイル関数（`alignment_profile_info()`、`reorder_alignment_profile()`）は、CLIと同様に、このコンピュータ上のファイルを読み込みます。

| 関数 | 機能 |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | 停止されるまで、1台のカメラを連続して録画します。 `CliJob`を返します。`tiff`の実行で生成されるフレームは、そのカメラのキャプチャ配列が書き込む`raw`ファイルです（センサーの独自の値で構成され、非圧縮）の配列として書き出されます； `png` および `jpg` は 8 ビットを保持するため、これらの実行で生成されるフレームは、表示用にデモザイク処理されたものです。バインドまたはプールされた DAQ は、実行の`daq/`フォルダ（統計情報上は`daq_files`）に書き込みます。各フレームは、完全なメタデータ（モデル、撮影時刻、レンズ、バンド、RAWフレーム上のキャリブレーション）とともに1回書き込まれます； `stop()`は、取得済みのすべてのフレームの書き込みが完了するたびに（最大25秒）一度返され、その時点で初めてカメラが閉じられます。ハードウェアトリガー用にアームされたままのカメラ （`TriggerMode` `On`）は、その実行中はフリーランモードに設定されます（開始応答の `trigger` に検出結果が示されます）。 停止時には、カメラは検出時の状態（トリガー設定および実行中に変更されたすべての設定）に戻されます。ただし、`keep_settings=True`（実行中の設定をカメラ自身のメモリに保存する（フリーラン）） または、そのメモリをリセットする`clear_settings=True`が指定された場合を除きます。フレームを記録しなかった実行は、10秒後に`stop_reason`および`no_frames`によって終了し、その理由は`last_error`にその理由が記録され、`stop()`はその理由を`error`として返します。ライターが追いつけなかったためにドロップされたフレームはカウントされ、その理由は`drop_reason`で示されます。 Chlorosが保持するカメラについては、`CameraSession.capture_repeated()`を使用します。`lattice continuous`。 |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | `fps`、`count`のグループ分、または停止されるまで。`CliJob`を返します （その `start_reply` には `n_cams` とワイヤ `pacing` が含まれる）； その最終統計には、取得されたグループ、書き込まれたグループ、破棄されたグループ、ライターが書き込みできなかったファイル、およびカメラから何も送信されなかったグループの数が含まれます。 `lattice multi-stream`。 |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | 1回の同期キャプチャ、カメラごとに1枚のデベイヤー処理済み画像：`results` (`filepath`、`serial`、`frame_id`) および `pacing`。 カメラが1台欠けているグループでは、最初に保存されたファイル数が表示されます。アレイが接続されていない `lattice multi-capture`。 |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` は固定露出で同期キャプチャ： `sync_groups`、`master`、トリガーごとに1行の`group_timestamps`（1回撮影を逃したカメラはその行に存在しない）および`sync`：マスターからの各カメラのオフセット（トリガーごとに異なる）（`offsets_us`：カメラごとに `mean_us`、`min_us`、 `max_us`、`jitter_us`、 `groups`）、1クロック単位（`clock_domain`：`ptp`、または`device` — PTPオフ、各カメラ独自の稼働時間クロックであるため、そのオフセットは起動時点であり、同期を測定するのはジッタのみ）。デバイスクロックとPTPが混在するタイムスタンプを持つトリガー（PTPオフまたはロック中）は除外され、カウントされる（`groups_mixed_domain`）。また、PTPクロックがマスターから依然として`agree_within_us`以上離れている場合も同様である （`groups_clocks_disagree`）も除外される；`notes`がこれを示している。`clock_offset_mean_us` / `clock_offset_jitter_us`は、カメラごとの最大平均値およびジッタである。 `success`はfalseとなり、`error`は、トリガーが欠落した場合、1つのクロックで全カメラのトリガーが2回未満だった場合、またはクロックが一致しなかった場合にfalseとなる (`sync.problems`) の場合、`success` は false となり、`error` も同様です。いずれの場合でも、行は返されます。 PTPグランドマスターが実行されていない場合は、それを起動します（`time_sync`は、`grandmaster_running`および`grandmaster_started_by_this_test`と指定します）、すべてのカメラの PTP を有効にし（`connect_array()` と同様に）、最初のトリガーが発生する前に、すべてのカメラの PTP クロックがグランドマスターに同期して安定するまで待機します： `PtpStatus`：1つのグランドマスターに対してスレーブ状態となり、グランドマスターから100 us以内の誤差で、3秒間安定していること。各カメラがこのようにロックされた状態で、 中断することなく、120秒間――電源投入から約2分間――その状態を維持します。ロックされたばかりのクロックは、「Slave」と読み取られてから約2分間、最大で約200マイクロ秒の誤差が生じますが、カメラ自身の`PtpOffsetFromMaster`にはそれが表示されません； 120秒経過すると、カメラのクロックは安定状態から約10マイクロ秒以内の誤差に収まります。このバックエンドから見て、以前のテストでPTPをオンにしたままロックされた状態を維持しているカメラは、そのロック時点からカウントを開始するため、 したがって、連続してテストを行う場合でも再度待機することはありません。テストで電源が投入されたカメラ、またはPTPが有効な状態で検出されたものの、このバックエンドからはロック状態が確認されていないカメラ（別の場所で電源が投入された、あるいはバックエンドの再起動前に電源が投入されたもの）は、120秒待機します。また、ロック直後のカメラと、まだ待機中のカメラが混在する配列においても、ロック直後のカメラは引き続き待機を続けます。 スレーブ状態を離れ、100 usを超過するか、待機中に3回連続で別のグランドマスターの読み取り値に従ったカメラ （1.5秒）間、別のグランドマスターに従った場合、カウントを再開します。そのような読み取りが1回または2回記録され（`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`）、カウントは継続されます。また、読み取れないグランドマスターIDは、変更ではなく「不明」として扱われます。 すべてのカメラのカウントが同時に（1秒以内に）再起動するのは、カメラ側の時間ではなく、グランドマスター側の時間が動いているためです。バックエンドのログには `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …` と記録され、`ptp_lock` は `grandmaster_steps` にリストされ、その後待機時間が切れた場合、エラーは `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …` から始まるで始まります。この処理に対して制限時間は延長されません。`ptp_settle_s`（0～600；セトル待ちがない場合は`0`）が実行のセトル時間を設定します。これ以外の値は拒否されます（HTTP 400）。 待機時間は、セトルタイムに90秒を加えた値（デフォルトは210秒、最低90秒）に制限され、呼び出しはバックエンドが`timeout`秒間待機します。デフォルトでは、240秒にセトルタイムを加えた値（360秒）となります。 `chloros-mcp`の`test_array_sync`と同様に動作します。`ptp_lock`には`converged`、 `waited_s`、`timeout_s`、`settle_s`、`locked_s` （各カメラがロックされて安定して固定されていた期間）、`found_settled`（検出時にすでに安定時間分ロックされていたカメラ）、およびカメラごとに、`status`、`offsets_ns`、 `slave_s` （ロックされてからの経過時間；「None」：ロックされていない）および `remaining_s`（どのカメラの PTP もオンになっていない場合は「None」）。バックエンドのログには、5 秒ごとに各カメラの状態、ロックされてからの経過時間、および残り時間が記録されます。 すべてのカメラはPTPをオフにして電源が入り、そこから約10秒でクロックが同期します。その後、各カメラはテストで検出された状態（PTP、自動露出、およびテスト実行中にオフに設定されたゲイン）に戻されます （`ExposureAuto`／`GainAuto`、および手動設定の`ExposureTime`／`Gain`）、 残りのカメラについては、`keep_settings=True`の場合を除き — つまり、PTPがオンのままのカメラ（`PtpEnable=True`、または`keep_settings=True`が設定された実行） はロックされた状態を維持し、このバックエンドがセトル時間の間ロック状態を確認すると、その後のテストではそれを待たなくなる。クロックがセトルしない場合、`success`はどのトリガーが発生する前にもfalseとなり、`error`が対処方法を示す： ロックされたことがないカメラ（`not_slave`）は、グランドマスターがそのカメラに到達していないことを意味します（`time_sync_status()`）。ロック状態が頻繁に解除されていたカメラは、ロックされていた時間によって名前が付けられます （`remaining_s`）；クロックがまだ収束中（`far`）の場合：`time_sync_status()`を確認し、テスト前にロック状態にするには、各カメラでPTPをオンにしてそのままにしておく （各カメラで `PtpEnable=True` または `keep_settings=True`；テストは検出結果を元に戻すため、有効なままになります）、その後テストを実行してください。`lattice multi-test`。 |
| `detect_array_master(line=None)` | 同期ライン上のカメラとデフォルトのマスター：`master_serial`、 `slave_serials`、`line`が検出されました。`lattice multi-detect`。 |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | これらの一連のカメラモデル（製品名、 `"LATT-M3C-L87-FOCN"`）が、このコンピュータのネットワークにどれほど多数現れることか：`tier`、`achievable_fps_max`、配線予算（`max_cams_collision_safe`、 `oversubscribed`）、`cam_caps`、および `fix_suggestions`。デフォルトでは、検出されたカメラのモデルは、 開かれているカメラはありません。お持ちのカメラについて、接続時に使用される設定については、[`analyze_array_network()`](#network-analysis-helper)を参照してください。`lattice analyze-array`。 |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | ライブフレームからアライメントプロファイルを算出し、保存します。 `method`: `orb`、`akaze`、`phase`、`checkerboard` （`checkerboard="9x6"`を含む、内側の角）または`manual`（`control_points={serial: [[x, y], ...]}`を含む、基準カメラのものも）。`model`: `translation`、`rigid`、`affine`、または `homography`。 より詳細なオプションはCLIのフラグです：`max_features`、`ratio_threshold`、`matcher`（`bf`／ `flann`）、`knn_k`、`prefilter`（`none` / `gradient` / `clahe` / `blur` / `hist_match`)、`ransac_threshold_px`、 `ransac_max_iters`、 `ransac_confidence`、`averaging`、`lock_rotation`、`lock_scale`、`lock_axis`; `per_cam_overrides` は `{serial: {option: value}}` です。`rms_threshold_px` は、より悪い解を保存することを拒否します。 `save_path`、`output_width` / `output_height`、`crop_roi`、 `reference_serial`、`transforms`（カメラごとに `reprojection_error`、 `num_matches`）および `preview_path`。`lattice align-calibrate`。 |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | プロファイル全体にわたる1枚の整列済みマルチバンド画像：`files`（その`bands`およびサイズを持つマルチバンドTIFF、 さらに、各バンドごとに1つずつ（`per_band`）、各カメラごとに1つずつ（`per_camera`）を含む）および`vignette_corrected`。 `bands`はバンドを選択し、`order`はバンドを名前順（リストまたはカンマ区切りの文字列）に並べ替えます。`lattice align-apply`。 |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | ストリームアラインされたマルチバンド画像（`capture_aligned_multiband()`と同じワープオプション）を、`count`まで、または停止されるまで処理します。 `CliJob`を返します。その最終的な統計値には、`frames_written`および任意の`frame_groups_incomplete`が含まれます。`lattice align-stream`。 |
| `alignment_profile_info(path)` | `align-info`として要約されたプロファイルファイルは、`name`、`created`、`method`、`reference_serial`、 `sensor_resolution`、`output_width` / `output_height`、`crop_roi`、`camera_serials`、 `transforms` （カメラごとに `model`、`reprojection_error`、`num_matches`、`reference`）, `camera_models`, `layer_order` (カメラごとに `band_names`, `wavelengths_nm`, `enabled`); `profile`はファイル全体です。 |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | プロファイルファイルをその場で変更：`order`はバンド名（それぞれ、その順序で自身のカメラのレイヤーを上に移動させる）、 `enable` / `disable`はシリアル番号です。新しい`layer_order`を返します。`lattice align-reorder`。 |
| `index_catalog()` | インデックスプリセット（`presets`：それぞれ `name`、`channels`、 `formula`）およびカラー `gradients`。 `lattice index --list-presets` / `--list-gradients`。 |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | 位置合わせされたマルチバンドTIFFに対するインデックス：`preset` (`NDVI`、`NDRE`、 …）または `formula` 上の `channel_map` は、バンド（`{"NIR": "NIR_850", "Red": 0}`、名前またはインデックスによる）にマッピングされます。「プリセット」シンボル `channel_map` は、そのシンボルで始まる名前のファイルに含まれる 1 つのバンドを除外します（`red` → `Red_660`、 `nir` → `NIR_850_RGN`）、ファイル自体（そのバンドリスト、またはChlorosキャプチャ／プロダクトの`Chloros:Band{R,G,B}Name`）によって命名されたもの、 したがって、Chlorosの反射率キャプチャ上のNDVIには`channel_map`は必要ありません； 1つのシンボルに対応する2つのバンドは拒否され、推測されることはありません。`capture_level`はバンドの内容を示し、`reflectance`または`radiance`は値を定量的なものにします； 未校正のスタック（`align-apply`など）は定性的な値を返します； `"unknown"`はファイルから読み込み、Chlorosファイルに含まれる`Chloros:ProcessingLevel`も対象となる。`stats`を返す （形状、1～99パーセンタイル範囲、LUT範囲、使用された`capture_level`）および入力の横に記述された`files` （`<input stem>_<preset>.tif`、`..._LUT.png`）の横、または`output`に記述されたパス：画像拡張子を含むファイルは、そのステムに基づいて命名されます （`output="D:/out/ndvi_FRGN.png"`は`ndvi_FRGN.tif`および`ndvi_FRGN_LUT.png`を書き込みます）、拡張子がない場合は接頭辞となります （`D:/out/ndvi` → `ndvi.tif`、 `ndvi_LUT.png`）；`files`からパスを読み取ります。これらは入力のカメラ記録（メーカー／モデル／シリアル番号、GPS、レンズ、UTCでの撮影時刻、キャリブレーションID（ただしraw-DNキャリブレーションは含まない））、`Chloros:ProcessingLevel=index`、およびインデックスを単一のバンドとして保持している（`XMP-Camera:BandName`、TIFFおよびLUT PNGも同様）。ファイルのみ。カメラ情報は含まれない。`lattice index --input`。 |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | 撮影、キャリブレーション（`reflectance` または `radiance`； `raw`はこれをスキップする）、位置合わせ、およびインデックスの計算を1つのステップで行う。`save_multiband=True`はキャリブレーション済みのスタックを保持し、`compute_index()`は後でそれを取得できる。 `stats`（使用したバンドを含む `serials` とともに）と `files` を返します。 `serials` を開きます（最初のものがトリガーを起動します）。それ以外の場合は、`layer_order` エントリが有効になっているプロファイルのカメラ（参照カメラを最初に （バンドが無効であっても開かれる）；バンドが無効なカメラは決してキャリブレーションされない。`channel_map`は、スタックが応答できないシンボルに対してのみ必要となる。 `compute_index()`のように、スタックが応答できないシンボルにのみ必要となる。プリセットのシンボルは、その名前で始まる有効なバンドを1つ取得し、これはどのカメラも開く前にチェックされる（応答するカメラが1台もない場合や複数ある場合は拒否され、それらの名前が指定される）。カスタム`formula`には、すべてのシンボルが必要となる。 `index.tif`とLUTは、スタックのカメラ、撮影時刻（基準カメラの露光時間、UTC）、およびレンズ情報を保持し、両者ともその1つのバンドとしてインデックス名を指定します（`XMP-Camera:BandName`、例： `NDVI`：カラーPNGファイルは、その色チャンネルではなくインデックスを指定する）。`lattice index --live`。 |

### `CliJob`

3つのストリームが返すハンドル。ジョブはバックエンド上で実行され、停止されるまでそのカメラを所有し続けます。

| メンバー | 説明 |
| --- | --- |
| `stats(timeout=10.0)` | `running`、`state`、`stats`（保存されたフレーム／ 書き込み済み、破棄済み、`fps`、`elapsed`、書き込み失敗、`stop_reason`；継続ジョブの場合はさらに`frames_grabbed`および`frames_queued`――キャプチャされたすべてのフレームは、書き込み、キューへの追加、破棄、または失敗のいずれかとなる――および `drop_reason`、 `writers`および`write_ms`（1フレームの書き込みにかかる平均時間）および`output_dir`。`state`は`running`、`ended`（自身で停止し、まだ停止処理は完了していない）、`stopping`（停止中のプロセスがキューを書き出している： ライブ統計）、および `stopped_by`）または `finished`（その最終統計、停止処理の `error` および `stopped_by`）。停止したジョブの記録は、各クライアントごとに10分間（最大で直近32件）保持されます。別のスクリプトや`chloros-mcp`が、同じ状態と最終統計情報を読み取ります。ジョブ`chloros-cli`が開始された （`lattice continuous`、`multi-stream`、`align-stream`） は、CLIが30秒間チェックインを停止するとバックエンドによって停止される（強制終了、またはマシンがスリープ状態）— バックエンドには`stop_reason`、`client_gone`、および`stopped_by`が記録されるが、 ここからその統計情報を読み取っても、ジョブは存続しません。SDKから開始されたジョブは、停止されるまで実行されます。各起動時には、その`timeout`がバックエンドの期限（`client_deadline_s`）として送信されます：その後に開かれたカメラ（まだ起動中のもの）は、元に戻されて閉じられ、ジョブは開始されず、その呼び出しにより `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`) が発生します。ジョブが保持しているカメラは、それ以外のすべてのオープン要求に対して拒否されます （`connect_camera()`、 再起動、一時的なキャプチャ）に対して、HTTP 409 `CAMERA_HELD`を返して拒否され、その際、ジョブ名と開始者が明示される。|
| `stop(timeout=30.0)` | 停止し、そのカメラを閉じる： `final_stats`、`output_dir`、および停止が正常に行われなかった場合の`error`。2回目の呼び出しでも同じ応答が返され、 別のクライアントからの停止要求も同様です。最初のクライアントがまだキューの書き出しを行っている間に送信された要求は、それが完了するまで待機します（最大25秒。それを過ぎると例外が発生し、最終的な統計情報は`stats()`に続きます）。 |
| `wait(timeout=None, poll_s=0.5)` | 自動的に終了するまで待機し（`count`に到達、ドライブ満杯、エラーなど）、その後`stop()`を実行して最終カウントを収集する。 `timeout`は、実行中の状態で`TimeoutError`を発生させます。`count`を持たないジョブは、停止されるまで終了しません。 |
| `job_id`、`kind`、`output_dir`、`start_reply`、 `stopped` | そのIDと種類（`continuous`、 `multi-stream`, `align-stream`）、書き込み先、開始ルートの応答、および `stop()` が実行されたかどうか。 |

コンテキストマネージャーとして使用し、スコープの終了時に処理を停止します。

---

## プロジェクトフォルダとレシピ

**プロジェクトフォルダ**とは、新しいプロジェクトが作成されるフォルダであり、Chlorosアプリ、CLI、およびSDK（モジュール `chloros_sdk.system_utils`; `chloros-cli project-folder`）で共有されます：

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

設定済みのフォルダがもはや存在しない場合、アプリと同様にデフォルト（`Documents/MAPIR/Chloros Projects`）にフォールバックします。 `set_project_folder()` に指定された相対パスは、バックエンドがこのマシン上にある場合、作業ディレクトリを基準として解釈されます。

**レシピ**とは、`chloros-cli project run PATH RECIPE`が保存されたプロジェクトに対して実行するYAML／JSON形式のアクションリストのことです。`run_project_recipe()`は、CLIの独自のランナー（SDKと共にインストールされる）を使用して実行するため、1つのレシピは以下のいずれから実行しても同じ動作をします：

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** は、`.yaml` / `.yml` / `.json` ファイル、あるいはレシピそのもの（`actions`（`apply`、 `capture`、`burst`、`stream`、 `sensor`、`wait`）を含む辞書であり、オプションで `overrides` および `stop_on_error`（デフォルトは true： 最初のアクションが失敗した時点で実行が停止する）。アクションは、CLIリファレンスの[Recipe DSL](cli-reference.md#recipe-dsl)に準拠しており、相対パス指定の`output`はスクリプトの作業ディレクトリを基準に解決される。
- **戻り値** `ok`（すべてのアクションが成功）、`actions`（成功したアクションの数）、`log` （ランナーの出力；`echo=True`も実行中にこれを出力します）および `validation`。レシピファイルが存在しないか、解析できない場合は、`ChlorosConnectError`が発生します。
- **実行前にチェックされます。**`validation` は `{"ok", "errors", "warnings", "steps"}` です：すべてのアクションは、キーとして正確に 1 つの既知の動詞（`{"capture": {"name": ...}}`、 `{"action": "capture"}`や`{"type": "capture"}`ではない）で、プロジェクトが持つ適切な種類のカメラ、配列、またはセンサーを指定するものでなければなりません。エラーのあるレシピは、**いかなる**アクションも実行しません（`ok` false、 `log`のエラー参照）、`stop_on_error`の記述にかかわらず。未使用のキー、 およびレシピが制御するものの現在接続されていないすべてのアレイや光センサーは、`warnings`となります。レシピはデバイスを接続しません。プロジェクトの最初のデバイスを接続するには（`ChlorosProject.connect_all()`、`chloros-cli project connect`、 またはMCPツール`connect_project_devices`）で接続してください。`dry_run=True`は何も実行せず、`{"recipe": ..., "validation": ...}`を返します。
- プロセス内では、一度に 1 つのレシピのみが実行されます。YAML レシピには PyYAML が必要です。

---

## プロジェクトの自動化 — `ChlorosProject`

保存されたChlorosプロジェクトは、`cameras.json` + `sensors.json` + `project.json`を含むフォルダです。`open_project(path, relink=None)`はマニフェスト（`relink`：移動されたプロジェクトの欠落しているソースファイルが現在格納されているフォルダ。バックエンドがそれを読み込む際に使用される――`process()`――**「移動されたプロジェクト」** （`ChlorosLocal`）を参照）、そして `connect_all` は、保存された設定とともに、すべての保存済みカメラ、アレイ、および光センサーをオンラインにします。これは、GUI が生成するのと同じハードウェア状態です。光センサーはバックエンドの [DAQセンサープール](#daq-sensor-sessions)を介して接続されるため、バックエンドが実行されている必要があります（[`SensorHandle`](#sensorhandle)を参照）。

### 最小限の例

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

または、コンテキストマネージャーとして：

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject` メソッド

| メソッド | 説明 |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | 保存されているすべてのカメラ、アレイ、および光センサーを検出・接続します。クラスごとの接続レポートを返します （`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`、それぞれに`connected`、`missing`、`errors`が含まれます。また、`arrays`は`alignment`もサポートしています）。`127.0.0.1:5000`でリスニングしているバックエンドが存在する場合、保存されたカメラおよびアレイはそれを通じて検出・接続されます。バックエンドが存在しない場合、このプロセスは`lattice_sdk`を介して自動的に検出・制御します。この処理には、Arena SDKランタイムおよびOpenCV (`cv2`) が必要です。SDKホイールはこれらをインストールしません。インストールできない場合、保存されたすべてのカメラとアレイは、その理由と共に `errors` に一覧表示され、`missing` は空のままになります。 実行中であるがSDKを拒否しているバックエンド（ログアウト：HTTP 401、Copper以下のプラン：403）は、「バックエンドなし」とはみなされません。保存されたすべてのカメラ、配列、および光センサーは、バックエンドの理由とともに`errors`の下に一覧表示され、 このプロセスでは何も開かれていません。各呼び出しでバックエンドが再確認されるため、ログイン後やChloros起動後の`connect_all()`は、同じプロジェクトオブジェクトに接続します。 同じカメラ（GUI または `chloros-cli project connect` のもの）を含む、バックエンドですでに起動している保存済みアレイは、保存された設定を再適用することなく、そのままアタッチされ、 `disconnect_all()`はそれを実行したままにします。光センサーは常にバックエンドを経由します（[`SensorHandle`](#sensorhandle)を参照）。バックエンドを生成することは決してありません。 |
| `disconnect_all()` | すべてを解放します。まずすべてのデバイスを解放し、その後、バックエンドがカメラ、アレイ、または光センサーの解放を確認しなかった場合（解放処理がタイムアウトしたか、拒否された場合）、 それぞれに `ChlorosConnectError` という名前が付けられます。すでに存在しないデバイス、 またはバックエンドが実行されていない場合は、何も出力されません。他のクライアントが開いていたアレイや光センサーは、そのまま実行されたままになります。プロジェクトの `with` 終了処理は決して発生しません。 |
| `capture_all(output_dir=".")` | 接続されているすべてのカメラおよびアレイから1フレームずつ、 接続されたすべてのライトセンサーから1つのスペクトル（`SensorHandle.read()`）を、一斉ではなく順番に（カメラ、次にアレイ、次にセンサー）取得する。 |
| `stream(camera, overlays=False, fps=10.0)` | 指定されたカメラ（またはアレイ）から BGR `numpy` フレームを生成するジェネレータ。`overlays=False` は、`grab()` レベルのハンドル`grab()` を使用して、`debayered` レベルでフレームを読み取ります： プロジェクトがバックエンドを介して接続されている場合はバックエンド経由で、そうでない場合は直接 `lattice_sdk` 取得ループ（配列は `{serial: frame}` 辞書を生成）を行います。`overlays=True` は `ChlorosLocal.camera_stream()` → バックエンドの `/api/camera/<serial>/stream-annotated` MJPEG フィードへルーティングされ、 カメラに保存された `ui.overlay` ブロックがクエリパラメータとして渡されます。これには、バックエンドモード、OpenCV (`cv2`)、および **スタンドアロンカメラ** が必要です： ダイレクトモードのカメラは `RuntimeError` を発生させ（バックエンドはこのプロセスが所有するカメラを取得できない）、配列は `NotImplementedError` を発生させます（カメラごとに合成オーバーレイ — メンバー名を指定してストリーム）。ワンショットでの同等処理： `CameraHandle.capture(annotated=True)`。 |
| `align_arrays(align=True, verbose=False)` | 現在接続されているすべての配列に対してアライメントを実行します。 |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | プロジェクトの画像に対してキャリブレーション／インデックスパイプラインを実行する（`ChlorosLocal.process`をラップする。これら4つが**唯一**受け入れられるキーワード引数である — `indices=`など は例外を発生させる；インデックスは `ChlorosLocal.configure()` 経由で設定する）。`ChlorosLocal()` を遅延生成し、これによりバックエンドが自動起動する。 |

属性:
- `proj.cameras` — `Dict[str, CameraHandle]` は、名前およびシリアル番号をキーとする。
- `proj.arrays` — `Dict[str, ArrayHandle]`：名前および array_id をキーとする。
- `proj.sensors` — `Dict[str, SensorHandle]`：名前および slot_id をキーとする。
- `proj.config` — `project.json["config"]` 辞書。

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**処理レベル。** `capture()`、`grab()`、および `frame_stream()` はすべて同じ `processing`
トークンを採用しており、チェーンは累積的です。各レベルはそれより上のすべての処理を実行します：

| レベル | 出力 | 備考 |
| --- | --- | --- |
| `raw` | 1チャンネル・ベイヤー、センサーネイティブ | デモザイク処理なし。このレベルではオーバーレイは利用できません。 |
| `debayered` | 3チャンネルカラー（`CameraHandle`およびすべての`grab()` / `frame_stream()`；`ArrayHandle.capture()`、`capture_smart()`、および`stream()`はデフォルトで`raw`となる）：メモリ内はBGR（`grab()` / `frame_stream()`）、`capture()`によってRGBとして保存 | バックエンドモードでは、キャプチャとして： RAWデータからキャリブレーションパックのダークオフセット（DSNU）を差し引いた後、OpenCVのエッジ認識デモザイク処理を行う — 線形LUT、フラットフィールド補正、またはアンミックスは行わない。バックエンドモードなしで動作する唯一のレベルは、カメラライブラリの単純なバイリニアデモザイク （ダークオフセットなし）。保存されたTIFFは測光RGBであるため、メモリ内のフレームのチャンネルを反転させ （`frame[..., ::-1]`）してファイルと比較する。 |
| `radiance` | float32, W/m²/sr/nm, **バンドごとに1つのプレーン** | 完全な放射測定チェーン：デモザイク + 3×3 アンミックス（マルチスペクトル） + DSNU + フラットフィールド + NIST スケール。露出 × ゲインが除算されており、値は絶対値となっている。 |
| `reflectance` | uint16、32768 = 1.0（または uint8、255 = 1.0 — `bit_depth`を参照）、**バンドごとに1プレーン** | 放射度を下向き放射照度で除した値（ρ = π·L/E）。DLS/DAQの測定値が必要 — 以下の注を参照。 |
| `display` | 8ビット sRGB風 | GUIと同等のレンダリング：カメラのアクティブなカラープロファイルによるCCM + ホワイトバランス + ガンマ補正。 |

ダイレクト（バックエンドなし）モードでは、`debayered`のみが利用可能です。別のレベルにある`grab()`は
`NotImplementedError`、`capture()`は`processing`を無視し、`frame_stream()`は
`RuntimeError`を発生させます。 `reflectance`には、使用可能なダウンウェル測定値が必要です。フレームの終了点は、
プールされたDAQを自動的にカメラのDLSスロットに引き込みますが、DAQがバインドされていない場合、チェーンは
反射率の出口を拒否し、黙って
劣った結果を黙って返すのではなく、返されたメタデータに降格を明示的に記録する。

> **反射率DNスケール — ハードコーディングしないでください。** LATTICEの反射率は`32768` = ρ 1.0を使用し、
> XMPに`Chloros:PixelScale=32768`と記録する。Survey3の反射率は`65535` = ρ 1.0 を使用し、
> `Chloros:*` タグを含みません。タグを読み取り、それを除算してください。これは uint16 ドメインで定義されているため、
> `32768` として、再スケーリングを行うすべてのフォーマット（16ビットTIFF、8ビットTIFF、 8ビットPNG/JPG、32ビットパーセント）
> — まず、保存されたデータ型を uint16 に正規化します（8 ビットからは ×257、float からは ×65535）。唯一の
> 例外は、1.3.0 以前のビルドからのファイルです。当時の 8 ビット TIFF はリスケールされず *クリップ* されていたため、
> スケールでは説明できず、`PixelScale`やMicaSenseのタプルも含まれていません。
> LATTICE反射率ファイルでタグが欠落している場合は、デフォルト値としてではなく「有効なスケールなし」として扱ってください。

**キャリブレーションタグ：`raw` のみ。** キャプチャの `raw` ファイルには、カメラのキャリブレーションパケットが含まれています： `CalibrationJSON`ペイロードおよびMicaSense互換タグ（`BlackLevel`、`RadiometricCalibration`、`Band{R,G,B}RadiometricCalibration`、 `MicaSenseVignetting*`）が含まれており、これらはセンサーの生データを記述しています。それ以外の各レベルのファイル（`debayered`、`display`、`radiance`、`reflectance`、`index/`のレンダリング）およびすべての連続記録／マルチカメラストリームのフレームには、カメラの識別子が含まれています （`XMP-mapir:CameraSerial`、`CameraModel`、`FilterType`、`LensModel`、 `CalBundleSha256`、…）およびフレーム固有の記録データである`XMP-mapir:FrameJSON`（露光時間、ゲイン、 温度、ピクセル形式、撮影時刻、DAQマッチ）で構成されており、すべての `process()` 製品にはこの識別子が含まれています。処理では、Chloros のキャリブレーションキャッシュ内で、シリアル番号と `CalBundleSha256` に基づいて当該ファイルのキャリブレーションを検索し、このコンピュータにそれが存在しない場合はパックを一度ダウンロードします。

#### 複数のカメラから作成されたファイル

複数のカメラのバンドから作成されたファイル — マルチバンド・スタック （`capture_aligned_multiband()`、`start_aligned_stream()`、`capture_live_index(save_multiband=True)`、`Registration.save_multiband()`）、複数のカメラから作成されたライブインデックス（`index.tif`、`index_LUT.png`）、アレイキャプチャの `composite/` 画像 — これらには、作成に使用されたすべてのカメラの名前が含まれています：

- **各-カメラごとのリスト、バンド順**（合成画像：背景レイヤー、次に前景；インデックスレイヤー、 そのバンドが由来するカメラ）：`XMP-mapir:SourceCameraSerials`、`SourceCameraModels`、`SourceSensorProfiles`、`SourceProductLines`、`SourceFilterTypes`、`SourceLensModels`、`SourceCalDateUTCs`、`SourceCalBundleSha256s` （各カメラのバンドが通過したキャリブレーションパック。それ以外の場合 — `capture_aligned_multiband()` / `start_aligned_stream()` スタック。このスタックのバンドはどのキャリブレーションパックも通過せず、カメラのパックChlorosがキャッシュしたものであり、 そのキャプチャファイル名と一致するパック）、`SourceLightboxCertIds`、`SourceHalogenLampCertIds`、`SourcePackTiers` — 並列リストであり、各リストの項目 *n* は同じカメラを表す；`-` は、カメラに値がない場合（キャリブレーションパックなし、レンズ不明）である。各カメラは **1つ** の項目である：パックのフィールドに複数の値がある場合 — パックのハロゲンランプ証明書 — これらは `;`（`F-1479-1;F-1480-1;F-1481-1`）で連結され、コンマは決して使用されないため、リストは表示方法にかかわらず、カメラごとに1つの項目として読み取られる。
- **EXIF `SerialNumber`** には、同じシリアル番号がコンマ区切りで格納されています（`214000535,214000551`）。
- 単一のカメラを特定する識別子は**含まれていません**（`XMP-mapir:CameraSerial`、`CalBundleSha256`、…）：これらは1台のカメラとそのパックを指定するものであり、処理の過程でこれらからファイルのキャリブレーションが算出されます。複数のカメラのバンドから算出されたインデックス （`index.tif`、`index_LUT.png`）から算出されたインデックスには、EXIFデータとして`Model` `LATTICE`（リグ）が記録されており、 1台のカメラのモデルである`LATT-…`ではありません。**1台**のカメラのバンドを含むファイル（バンドごとのTIFF、`capture_aligned_multiband(per_camera=True)`フレーム、単一カメラのスタックまたはライブインデックス）には、他のすべてのキャプチャファイルと同様に、そのカメラ固有の識別タグが含まれています。 これは他のすべてのキャプチャファイルと同様です。`capture_aligned_multiband()`のスタック、バンド別ファイル、カメラ別ファイル、および各`start_aligned_stream()`スタックとそのバンド別ファイルには、そのグループのキャプチャ時刻 （基準カメラの露光時刻、UTC）を含んでおり、モノクロカメラの `per_camera` フレームは、スタックと同様に `bit_depth` において 1 プレーンです。**カメラの向き（反転／回転）。** `grab()` / `frame_stream()` からのフレームは、
すべてのレベルにおいて常にカメラのネイティブ向きです。保存されたキャプチャは、カメラの向き
設定（GUI の「Mirror H」/「Mirror V」/回転）に従います：

- **`raw` および `debayered` ファイルは、設定にかかわらず、センサーが読み取ったそのままのデータです**。ベイヤー
  モザイクが回転または反転されると、カラーフィルターのパターンが異なるものとなり、キャリブレーションマップは
  センサー独自のグリッド上にあります。
- **`display`、`radiance`、`reflectance` およびインデックスファイルは、**すべての
  キャリブレーション計算の後に一度回転されています。それらのクランプマスクは同じグリッド上にあります。
- **すべてのキャプチャファイルは、XMPに設定を記録しています**：`XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`）、`ChlorosRotation`（0 / 90 / 180 / 270、時計回り、
  反転処理後）および `ChlorosOrientationApplied` — `false` は生データおよびデベイヤー処理済みデータに対して、`true` は方向補正済み
  データに対して適用されます。 `ChlorosProcessing` は、そのファイルのピクセルに適用された操作を一覧表示します（ネイティブの場合は
  `none`）。 これには、連続記録（カメラ自身の設定、
  未適用）およびマルチカメラストリーム（アイデンティティ：その構成フレームは、
  アレイキャプチャと同様にネイティブ）のすべてのフレームが含まれます。
- **プロジェクトの処理では、記録された向き**が、RAWデータから生成される成果物に適用されます
  （まずセンサー座標でのキャリブレーション、次にアレイアライメントワープ、最後にオリエンテーション）、
  これにより、ライブキャプチャの生成物と一致するようになります。これには、キャリブレーションされていないセンサー応答／ヴィネット補正済みの
  フォールバック生成物も含まれます。デベイヤー処理済みの生成物はネイティブのままです。
- **Chlorosの画像ビューアは、記録情報に基づいて回転されたRAWファイルまたはデベイヤー済みファイルを表示します** — 表示のみであり、
  ファイル自体は変更されません。記録情報のないファイルは、保存されたままの状態で表示されます。
- **記録情報のないキャプチャは、ネイティブとして処理されます。** バージョン1.3.0以前では、シングルカメラによるキャプチャは
  RAWファイルとデベイヤー済みファイルを自身で変換していました。Chlorosは、そのようなRAWファイルとネイティブファイルを区別できず、
  推測も行いません。
- **アレイキャプチャは、デフォルトですべての構成ファイルをネイティブのまま保持し**、その識別情報を記録します。
  `ArraySession.capture(orient=True)`はそれらの積算値を出力します（アレイキャプチャの制御を参照）。

#### バンドごとの出力と混合アレイ

`radiance`および`reflectance`レベルでは、カメラは**そのカメラのバンドごとに1つのプレーン**を返します — M3Cの場合は3つ （3つの未混合バンド、BGRチャネル順）、M3Mの場合は1つ（その
単一の狭帯域フィルター）となります。モノクロカメラの複製されたプレーンは統合されるため、**混合
M3C/M3Mアレイは、シリアルごとに異なるアレイ形状を生成します**。各プレーンが何であるかを確認するには、フレーム
メタデータから `bands` キーを読み取ってください。チャンネル数から推測しないでください：

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

値を使用する前に、必ず `hdr["processing_level"] == "reflectance"` を確認してください。 降格
（DAQ 読み取りなし、キャリブレーションパックなし、バンドレイアウト不明）は、昇格ではなく `processing_level` および
`radiometric_refused` で報告されます。返されるアレイは依然として有効なフレームですが、 単に
要求されたデータではないだけです。降格された保存済みキャプチャの XMP には、
`Chloros:RadiometricRefused` （例：`no-band-layout`、`no-dls-snapshot`）と同じコードが XMP に含まれます。

#### 読み取りごとに 1 回のトリガー — `group_id`

**アレイのメンバーは、各カメラ独自の
グラブバッファからではなく、アレイの同期されたフレームグループから提供されます。** ハードウェア同期されたアレイのカメラはまとめて公開されますが、カメラごとの N 回の読み取りは
まとめて*サンプリング*されるわけではありません。各カメラの最新のバッファされたフレームを読み取ると、トリガーの
境界をまたがり、たった1つのフレームがドロップされただけで、気づかれることなくセット全体の同期が崩れてしまいます。

グループからの提供は必要ですが、それだけでは不十分です。各リクエストは依然として
その瞬間の*最新の*グループをスナップショットとして取得し、その間にグラブループが新しいフレームを公開してしまう可能性がある。
実稼働中の4カメラアレイで測定したところ、**20回の並列読み取りのうち12回が2つのグループにまたがっていた** — 常に
正確に隣接していた。したがって、読み取りは**固定**される：

1. すべてのフレームには、アレイの単調なパブリッシュカウンターである `group_id` が含まれています。
2. `grab()`はマスターを読み取り（マスターが応答できない場合は、次に応答可能なカメラを読み取り）、
   その後、残りのカメラを、その読み取りが着地したグループID（HTTP
   エンドポイント上の`?group_id=N`）に**固定**します。
3. バックエンドは最近のグループを**少なくとも0.5秒間**保持するため、後のパブリッシュによってすでに
   追い抜かれた同等のグループであっても、そのトリガーに対しては引き続き処理されます。これは、9台のカメラからなるフルセンサーアレイであっても、カメラごとの
   9台のカメラからなるフルセンサーアレイであっても、カメラごとの反射率エクスポートの往復通信に十分な時間です。バックエンドが
   もはや保持していないピンは、別のトリガーで黙って応答されるのではなく、`409`および`group_aged_out: true`で拒否され、
   別のトリガーで応答するのではなく、`409`および`group_aged_out: true`で拒否されます。`grab()`はセットを再読み込みします。

ピンニングを適用して再測定した結果：**20回の読み取りのうち20回すべてで、1つのトリガーが返されました。** `grab()`がこの処理を自動的に行います。
HTTPエンドポイントを直接駆動する場合にのみ、`group_id=`が必要となります。

- ピン留めされたグループで処理できないカメラ（グループに含まれていない、またはリクエストを拒否する場合
  （そのカメラが持たない `band`、生成できないレベルなど））は、**別のトリガーから埋められるのではなく、省略されます**
  他のカメラは引き続き返されます。
- `grab()` が返す辞書は、`GrabResult` （単純な `dict` のサブクラス；
  `from chloros_sdk import GrabResult`）です： `.errors`は、各
  省略されたシリアル番号を、その読み取り操作で発生した例外にマッピングし、`.group_id`は共有グループを表します。空の
  `.errors`は、すべてのカメラがこのグループを構成していることを意味します。
- スタンドアロンカメラにはグループがなく、`group_id: None`を報告し、`group_id=`
  ピンを無視するのではなく、**拒否**します。

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

この同じカウンタは、アレイのメンバーの**カメラごとのすべてのMJPEGストリーム**（
プレビュー（`/api/camera/stream/<serial>`）、注釈付きプレビュー（`/stream-annotated`）、および
反射率ストリーム（`/stream-reflectance`）——が、各パートにおいて`X-Chloros-Group-Id`として公開され、 `X-Chloros-Frame-Id`および`X-Chloros-Timestamp-Ns`とは別に、
各パートに`X-Chloros-Group-Id`として公開されます。ストリームは特定の時点の
データではなく連続的なものであるため、パートを収集し、そのヘッダーに基づいてグループ化することで照合されます。設定すべき
ピンはありません。スタンドアロンのカメラにはグループがありません。 詳細は
[カメラごとの MJPEG ストリーム](#per-camera-mjpeg-streams-no-rtsp)を参照してください。

**`frame_stream()`は決して沈黙することはありません。** 何らかの理由で失敗したティックであっても、その原因が自然に解消される可能性がある場合は
スキップされます。カメラストリームの場合は何も返さず、配列ストリームの場合は応答したカメラを
返します （場合によってはゼロ）を返します。その理由としては、まだフレームがないこと、カメラの取得スレッドが
1～2秒間一時停止したこと（`404 not streaming`。これは、別のカメラが接続
または切断されたとき、あるいは設定変更によってプールのバランスが再調整されたときにバックエンドが応答するもの）；バックエンドの再起動または`5xx`；アライメントが
まだ安定していない （`409 … retry shortly`）；および、DAQにまだ測定値がないため反射率の拒否
（`no-dls-snapshot`、古いまたは空の測定値）です。 次のティックで拒否が発生しても、同じことが繰り返されるだけである —
未知の `band`、放射輝度に関する `bit_depth=8`、放射輝度スケールのないキャリブレーションパック、**DAQが全く存在しない**
場合の反射率拒否*（カメラやそのアレイにバインドされていない、バックエンドに接続されていないもの：
拒否の`dls_source`は`"none"`である）、その他の`4xx`など） 拒否の`dls_source`は`"none"`）、その他の`4xx` —
`chloros_sdk.FrameFetchError`を引き起こす （`RuntimeError`、これには`.status`および`.body`が含まれる）を
同時に発生させる。同様に、その間にフレームが存在しない状態で**60秒**のティックがスキップされた場合も、 `fps`が何であれ — アライメントの安定化とバックエンドの再起動を乗り切るには
十分な長さであり、
完全に機能しなくなったカメラがストリームを終了させるには十分な短さです。このギブアップエラーも、カメラの場合も
配列の場合も `FrameFetchError` となり、最後の読み取り時の例外は `.errors` （カメラの場合は `__cause__` として）。
アレイの場合、「refused」はすべてのカメラが拒否されたことを意味し、例外 `.errors` には各カメラの
理由が格納される。

**配列 `frame_stream()` は各トリガーを一度だけ生成する**（`ArrayHandle` および `ArraySession` も同様）。
最新のセットがすでに返されたトリガーであるティックは、同じセットを再度返す代わりに次のトリガーを待機するため、
ストリームは `fps` と配列自身のフレームレートのいずれか遅い方で実行されます
のいずれか遅い方で実行され、連続するセットが `group_id` を共有することは決してありません。パブリッシュを停止した配列は
`FrameFetchError`（「60秒間新しいトリガーなし」）を発生させ、最後のセットを繰り返すことはありません。これがなければ、
配列が公開した量以上のストリームが要求された場合、あるいは接続直後に開始された場合、
重複したデータが配信されてしまいます： ある測定実行では、20セットが11個の異なるグループIDを含んでいました。

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### 単一のバンドのエクスポート — `band`

`grab()` / `frame_stream()` を使用し、`band="<name>"` を指定することで、カメラのバンドのうち1つだけを
単一チャンネルの `(H, W, 1)` 配列としてエクスポートできます としてエクスポートします。これはモノクロカメラが生成する形状と同じであるため、1バンドの出力は、
M3Mからの出力であれ、M3Cの分割によるものであれ、同様に処理されます。FRGNの赤バンド：

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

許容される表記は、完全なレイアウト名（`Red_660`、`NIR_850`、`Green_550`）またはその短縮
ヘッダー （`red`、`nir`、`green`）であり、大文字小文字は区別されません。不明な名前は拒否され、利用可能な
名前がリストされます。

> **選択はチャンネル番号ではなく、名前によって行われます — これは意図的な仕様です。** 「チャンネル 1」は、
> アレイの BGR チャンネル順では `Green_550` ですが、フィルターコードの（`RGN`）では*赤*となり、これら2つは
> 一致しません。間違ったものを選択すると、間違った帯域の、一見妥当に見えるフレームが生成されますが、
> 下流のどの処理でもこれを検出することはできません。 そのため、数値インデックスは存在しません。

#### ストリームの位置合わせ — `align`

`align=True`は、フレームをその配列の共通基準フレームにワープさせます。各カメラは
**同じ**プロファイル解像度とクロップROIを通じてワープされるため、 そのため、個別に開かれたカメラごとのストリームは**相互に位置合わせされた**状態で出力されます。これらをまとめて要求する必要はありません：

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

要件と動作:

- カメラは、アライメント
  プロファイルを備えた**ハードウェア同期**されたアレイ（M8同期ケーブル）に属している必要があります。共同登録では、構成要素が同時に露光されていることが前提となりますが、ソフトウェアまたは
  フリーラン配列では実現できません。これに該当しない場合は、`409`で拒否されます。
- **拒否されます。決して黙ってアライメント未調整のままにはなりません。** キャプチャパスでは、警告を表示した上でアライメント未調整のまま保存される場合があります。
  これは、ファイルがプロヴェナンスタグを保持しており、後で再登録できるためです。一方、ライブストリームには
  そのような再チャンスがないため、プロファイルの欠落はフォールバックではなくエラーとなります。
- **アライメントを要求すると、それが開始される。** プロファイルのないアレイで `align` を要求すると、
  キャリブレーションが開始され、待機中の内容（例：*"214000551 (1/3) で
  露光が安定するのを待機中"*）を含む `409` を返します。再試行すれば、通常は数十秒以内に成功します。
  これはヘッドレス環境において重要です。接続時の自動アライメントは 30 秒後に中断され、
  試行はスキップされるため、露光が
  収束に時間がかかる配列の場合、以前はセッション全体でプロファイルが得られず、デスクトップの結合ビューを開かない限り
  プロファイルを取得する方法がありませんでした。`alignment_status`はまた、
  `ae_align_holdouts`を報告し、まだ安定化中のカメラとその進行状況を明示します。
- 1つのアレイのすべてのストリームに**同じ `max_h`**を割り当ててください。そうしないと、各クロップが異なるピクセル
  グリッド上に配置されてしまいます。
- アライメントはビット深度エンコードの*前*に適用されるため、ワープ処理では全精度の
  値が補間されます。

#### `bit_depth` — 精度と帯域幅のトレードオフ

`grab()` および `frame_stream()` は、`bit_depth=16`（デフォルト）または `bit_depth=8` を指定できます：

| `bit_depth` | 反射率のエンコーディング | コスト |
| --- | --- | --- |
| `16` | uint16、32768 = ρ 1.0、 ρ 2.0 までのヘッドルーム | 完全精度。 |
| `8` | uint8（`0..refl_max` （デフォルト ρ 1.0） | バイト数が半分；デフォルトの天井値での量子化幅は約 0.004 ρ。天井値を超える値はクリップされる — 下記参照。 |

##### 8ビットの上限 — `refl_max`

`refl_max` は、8ビットの範囲がカバーする ρ を設定します。 したがって、`pixel_scale = 255 / refl_max` では ρ は依然として
`pixel / pixel_scale` となります。`grab()` および `frame_stream()` は、カメラと
配列ハンドルの両方でこれをキーワードとして扱います（HTTPエンドポイント上の`?refl_max=`）。これを上げると、スペキュラー
ハイライトのための余裕が得られますが、その際、ステップは比例して粗くなります：

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | ステップ |
| --- | --- | --- |
| `1.0`（デフォルト） | 255 | 0.0039 ρ |
| `1.5` | 170 | 0.0059 ρ |
| `2.0` | 127.5 | 0.0078 ρ |

> **自動露出では8ビットのクリップを修正できず、それを試みるとデータはさらに悪化する。**
> ρ = π·L/E（露出とゲインを分母に除算した値）であるため、ρはシーンの特性であり、
> カメラ設定の特性ではない。ライブアレイで測定したところ、露出を半分にすると平均ρが1.3%変動した。
> 範囲に「合わせる」ために露光を上げると、むしろピクセルが**センサー**の飽和状態に追い込まれ、
> これによりLが過小評価され、エラーが検出されないままρが**低く**偏る（測定値：最大ρ 1.203 → 0.985（
> 露光2倍時）、平均−23%）。 `refl_max`でダイナミックレンジを選択してください。センサーを
> 飽和状態から守ることは、すでにAEの役割であり、それはρの範囲ではなく、その*妥当性*を保護するものです。

**`reflectance_clipped_fraction`をお読みください。 フラグではありません。** `reflectance_clipped_at_1_0`は
たった1ピクセルでトリガーされます — 実際のシーンでは、割合が`0.000001`であるにもかかわらず、`True`と報告されました。これは100万分の1ピクセル
100万分の1の確率で発生した。閾値を引き上げるべきかどうかは、この割合によって判断される。

**`bit_depth=8`は反射率のみを対象とする。** `radiance`に対してこれを要求しても、
処理されるのではなく拒否される： 反射率は8ビット符号化でも維持される0～1の固定範囲を持つ一方、放射輝度には
固定範囲がないため、8ビットに収めるにはフレームごとのパーセンタイル伸縮が必要となり、そのマッピングは
フレームごとに変化する――これは測定値ではなく、単なる画像に過ぎない。

#### ライブ反射率ストリーム (MJPEG)

`GET /api/camera/<serial>/stream-reflectance`は、バンドごとの8ビット反射率を
`multipart/x-mixed-replace`として配信します。M3C形式の場合は3チャンネルのJPEG、M3M形式の場合はグレースケールとなります。クエリパラメータ：
`fps`（デフォルト 10）、`jpeg_q`（デフォルト 95）、 `max_h`（0 = ネイティブ）、`band`（1つの名前付きバンド、
上記と同様）、`align` （`1`を共同登録 — SDKの`align`と同じルール）、`refl_max`（8ビット
エンコーディングの上限、上記と同様）。各パーツには
`X-Chloros-Group-Id`、`X-Chloros-Frame-Id`、`X-Chloros-Timestamp-Ns`、`X-Chloros-Bands`、
`X-Chloros-Pixel-Scale`、`X-Chloros-Aligned`、および`X-Chloros-Reflectance-Clipped`を保持しています。
**異なるカメラからのパーツで、同じ`X-Chloros-Group-Id`を保持しているものは、同じトリガーです** —
このようにして、1つのアレイに対するカメラごとの複数のストリームが再構成され、アレイのメンバーは
同期されたフレームグループから提供されるため、サーバー側でこれが保証されます（
[読み取りごとに1つのトリガー](#one-trigger-per-read--group_id）を参照）。接続時に反射率情報が
利用できない場合は `409` を返し、実行中に利用できなくなった場合はストリームを終了するため、ストリームが停止した
場合は「まだ正常で、単に静止している」のではなく、「反射率が得られなくなった」ことを意味します。

> **JPEGは非可逆圧縮である — これらのピクセルは測定値である。** 圧縮エラーは
> 反射率値に直接影響し、素材の境界となるエッジ部分で最も顕著になる。そのため、品質
> のデフォルト値は95に設定されている。数値をそのまま保持する必要がある場合は、
> `grab(processing="reflectance")` / `frame_stream(...)` を使用してください。これらは、パスにコーデックを含めない状態で
> 同じ出力を返します（なお、コーデックではなく通信コストが制約となる場合は、`bit_depth=8` を使用してください）。

> **EXIFデータはエクスポート先に引き継がれます。** `process()`は、ソースキャプチャのGPSブロック
> **およびそのExifIFD**をすべての製品にコピーするため、エクスポートされたファイルには`FocalLength`、`FNumber`、
> `ExposureTime`、`ISO`、`DateTimeOriginal`、`CameraSerialNumber`に加え、
> 地理参照情報も含まれています。`FocalLength`は、Pix4Dが地上サンプル距離を算出するために使用するデータです。これがなければ、
> 再構築は著しく誤ったスケールになってしまいます（実測例では、411 mの敷地が
> 47.8 kmの敷地となってしまいました）。コピーが意図的に `-all:all` になっていないのは、IFD0 の構造タグが
> LATTICEの出力を破損させるためであり、`ExifImageWidth`/`Height`は、エクスポートされたラスターではなくソースの
> キャプチャを記述しているため除外されています。

キャプチャ段階のサブフラグ（放射レベルに適用される — `radiance`、`reflectance`、 `display`）：

| フラグ | デフォルト | 意味 |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + フラットフィールド + 3x3 アンミックス + NIST 放射測定スケール。 |
| `apply_white_balance` | `True` | WB LUT。カメラにDAQがバインドされている場合はDLSに対応。 |
| `apply_index` | `False` | 植生指数の評価。 |
| `index_expression` | `None` | 計算式のオーバーライド。空でない空でない場合 → インデックスが自動有効化される。 |
| `annotated` | `False` | GUI装飾（ゼブラ／グリッド／ピーキング）のオーバーレイ。`raw`では利用不可。 |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **戻り値は `chloros_sdk.CapturePaths` です。 型は `CapturePathMap` であり、`Dict[str, str]` ではありません。**
> `chloros_sdk.CapturePathMap` は `Dict[str, Union[str, List[str]]]` です：各シリアルに単一レベルの
> `processing`は各シリアル番号に1つのパスを割り当てますが、`processing="all"`は、
> **順序付きリスト**として、そのカメラ用に保存されたすべての製品を返します。`ArrayHandle.capture()`は
> `levels=`を受け付けず、認識できないキーワード引数（`levels`、 `daq`、
> `render_index`、`force_daq`）は無視されるため、ここに示されているもののみを渡してください （`aligned=False`
> は、`ArraySession.capture()`と同様にネイティブファイルを保存します）；それらのコントロールには
> `ArraySession.capture()`を使用してください。ライブの複合コンポジット（ストリーミングされていた場合）は、シリアル番号の下ではなく、追加の
> `"combined"` キーの下に届き、シリアル番号の下には届きません。`str` を前提としたコードは、
> リスト形式では型チェッカーによる警告なしにエラーとなります — アノテーションには、リスト形式の公開後しばらくの間 `Dict[str, str]`
> と記載されていたため、このエイリアスが存在します。フラット形式を使用したい場合は、
> 正規化してください：
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **記述できないもの：`.failed`。** `capture()` が返すマップは、
> `chloros_sdk.CapturePaths` — 単なる `dict` のサブクラスであるため、上記のコードはすべて
> 変更なしで動作します — これには `.failed` も含まれます。これは、バックエンドが書き込みできなかったファイルであり、
> 以下のエントリと同じものです： [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> （`serial`、`level`、 `error`、および `disk_full: True`）と同じエントリが含まれています（ドライブが満杯の場合）。 部分的な
> キャプチャでも、書き込みが行われたすべてのパスが返されます。ドライブが満杯の場合、マップは
> 空になり、`.failed`がその理由を示しています。 `.skipped`はカメラごとのスキップを一覧表示し、
> `.group_seq`は同期グループを、 また、`.alignment` / `.oriented`は
> バックエンドのものであり、`CaptureResult`と同様です。`capture_smart()`はこれらを
> `result["frames"].failed` および `result["failed"]` として返します。
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### 配列のアライメント

`ArrayHandle`は、アライメント領域全体を公開します。プロファイルはデフォルトでセッション限定です。永続化するには、`export_alignment()`を明示的に呼び出してください。

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### 接続時のアライメント

`connect_all(align=...)` は、接続時にすべてのアレイを自動アライメントできます：

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

指定がない場合は、`project.json["config"]["auto_align_on_connect"]` にフォールバックします。

### `SensorHandle`

保存済みの光センサー（DAQ-U、DAQ-M、またはDAQ-E）で、バックエンドの [DAQセンサープール](#daq-sensor-sessions) — `connect_daq_sensor()`および`chloros-cli daq pool-connect`が使用するのと同じプール — から開かれた保存済みの光センサー（DAQ-U、DAQ-M、またはDAQ-E）です。

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` は、センサーの **次の** スペクトルを待機します。呼び出し時にすでにバッファに格納されているものはカウントされません — そして `(spectrum, is_saturated, integration_time, x, y, z)` を返します。5 秒以内に到着しない場合は `None` を返します。 一時停止されていたストリームが再開されます。センサーが校正されている場合、`spectrum`はW/m²/nm単位です。
- `.sensor`はセンサーの[`DAQSensorSession`](#daq-sensor-sessions）の値となります。それ以外の場合は、`latest(n=...)`、`status()`、`record_start()` / `record_stop()`となります。
- `.slot_id`、`.model`、 `.serial`、`.name`、および `.settings` は、`sensors.json` から保存されたエントリです。

**`connect_all`が復元する内容。** 各センサーは、保存されたアドレス（COMポート、Bluetoothアドレス、またはDAQ-Eホスト）で開かれます。プロジェクトでセンサーのIDが記録されている場合、そのセンサーでなければなりません。そのアドレスにある別のユニットは拒否され、接続されません。保存された積分時間、フレーム平均化、自動露出が適用され、キャップ補正も行われます：

| 保存されたキャップ | 接続先 |
| --- | --- |
| キャッププロファイル（例：`fov_30`） | そのプロファイル。 |
| 選択なし | `sunshine_cosine`（接続時のデフォルト）が、想定通り（`cap_id_source=auto_default`）として記録されたため、処理中に警告が表示されます。センサーはキャップなしの状態で出荷されます。装着されているキャップを選択してください。 |
| `none` — GUIで**「なし（センサー本体のみ）」**として選択 | キャップ補正なし。 |

DAQ-E-Sは、保存されていた内容にかかわらずキャッププロファイルを受け付けません。そのディフューザーは校正の一部となっているためです（[DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)を参照を参照）。以前のChlorosで保存されたプロジェクトでは、センサー無キャップが選択肢として記録されていないため、`sunshine_cosine`として再接続されます。GUIで再度**なし（センサー無キャップ）**を選択し、プロジェクトを保存してください。

別のクライアントがすでにプール（CLI、別のスクリプト）で開いているセンサーは、設定を再適用することなくそのまま接続され、 `disconnect_all()`はそのセンサーを稼働させたままにし、`connect_all()`が開いたセンサーは解放されます。`connect_daq_sensor()`と同様に、ChlorosのGUIはプールの一部では**ありません**： スクリプトからプロジェクトを接続する前に、GUIでセンサーを閉じてください。**要件とレポート。** 光センサーには、有料のChloros+プラン（Copper以上）にログインした状態で実行中のChlorosバックエンドが必要ですが、`connect_all()`はこれを起動しません。実行中のバックエンドがない場合、保存されたすべてのセンサーが`report["sensors"]["errors"]`の下に一覧表示され、その旨が示されます。 バックエンドが開けないセンサーも同様にそこに一覧表示され、バックエンド側の理由が記載されます。光センサーの場合、`report["sensors"]["missing"]`は空のままになります。

---

## 直接ハードウェア（バックエンド不要）

LATTICEカメラ（CI、ヘッドレスロボット、組み込み）においてバックエンドへの依存を完全に排除したい場合は、`lattice_sdk`を直接インポートしてください。これは`chloros_sdk`によって再エクスポートされます。 `CAMERA_AVAILABLE`に関する注意点： `lattice_sdk`はSDKホイールに含まれていますが、Arena SDKランタイムとOpenCV（`cv2`）の存在を必要とし、ホイールはこれらをインストールしません。光センサーはバックエンドを経由します — [`connect_daq_sensor()`](#daq-sensor-sessions) またはプロジェクトの `proj.sensors` — を通りますが、DAQ-M バンド振幅ヘルパー [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes)は例外であり、これには`bleak`も必要です。

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### プリセットとトリガー

4つのプリセットのうち3つは**フリーラン**です。カメラは連続して露光を行い、
`capture()`が次のフレームを返します。`triggered`は例外で — これは、
ライン2のハードウェアエッジをトリガーとしてカメラを待機状態に設定するため、エッジが検出されるまでは何も撮影しません。

| プリセット | トリガー | 使用場面 |
| --- | --- | --- |
| `default` | フリーラン | 一般的な用途 |
| `high_speed` | フリーラン | 8ビット、60 fps上限、短時間露光 |
| `high_quality` | フリーラン | 12ビット、fps制限なし — 静止画撮影の標準的な選択 |
| `triggered` | **アーム状態、ライン2** | カメラはM8シンクロケーブルで接続されており、別の何かがシャッターを切る |

`triggered` を選択した場合（または `trigger_mode="On"` を自分で設定した場合）、
ライン 2 を駆動するものが何もないと、すべての `capture()` はタイムアウトします — これは、カメラに待機するよう指示したため、
当然の結果です。SDKでは、この現象が発生した際の説明が記載されています。
[キャプチャ中のSC_ERR_TIMEOUT](#direct-hardware-backend-free)を参照してください。

> **注 — 接続時の「GVSPプローブ」／`SC_ERR_TIMEOUT -1011`メッセージはエラーではありません。**
> 接続時、SDKはスループット向上のために**ジャンボフレーム**（9000バイトのGVSPパケット）のネゴシエーションを試みます。 ダイレクトなポイント・ツー・ポイントのNICリンク（例：リンクローカルの`169.254.x.x`アドレス）では、通常、ネットワークはジャンボフレームを伝送できないため、このプローブはタイムアウトとなり、次のような行がログに記録されます：
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> これは**設計上のフォールバック**です。SDKは自動的に標準の1500バイトパケットに戻り、カメラは通常通り接続を継続します（続く`[chunk-enable …]`の行は、通常の接続シーケンスの一部です）。キャプチャは引き続き機能します。
>
> このプローブはスキップすることもできますが、**これは単にログを出力しないようにするだけのものではなく、ジャンボフレームを無効にするものです。** ネットワークの状態がどれほど良好であっても、カメラは「Don't-Fragment」指定のpingに対して最大1500バイトまでしか応答しないため、pingテストだけではジャンボフレームを検出することはできません。これを検出できるのはこのプローブだけです。これを無効にすると、カメラはどのネットワークにおいても、 ：
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> ジャンボフレームを処理できないと*確実に*分かっているネットワークでのみ有効であり、その場合、カメラ1台あたり接続時間を約1秒短縮できます。 これは単なる見かけ上の違いではなく、実質的なトレードオフであるため、SDKではこれを使用する際にその旨が明記されるようになりました：
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **特別な理由がない限り、設定を変更しないでください。**有効にしたままの場合、各カメラはバックエンドセッションごとに1回測定され、リンクが変更された際（別のアダプタ、別のリンク速度、またはスイッチの切り替えなどによるリンク切断）にも再度測定されます。再接続時には、その測定結果が再利用されます。**スイッチの背後で変更があったものの、スイッチ自体がダウンしていない場合は、バックエンドを再起動して再度測定を行ってください。** 初回接続時、このチェックに数分かかる場合があり、その際はログにその旨が記録されます。
>
> ジャンボパケットのスループットを*確保*したい場合は、エンドツーエンドでジャンボを有効にしてください （NIC MTU 9000 ＋ ジャンボパケットを通すスイッチ）を有効にするか、リンクがジャンボパケットを伝送できることが分かっている場合は `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` で固定してください。ただし、恒久的に設定するよりは、コマンドごとの `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` を優先してください。 固定サイズに設定するとプローブがスキップされ、上流のネットワークへの適応が停止してしまうためです。経路上の**すべての**デバイスがジャンボパケットを通過させられる必要があります。これにはPoEスプリッターやインジェクターも含まれます。これらが原因で、本来ジャンボパケットに対応しているはずの環境でもパケットが通過できないことがよくあります。

> **`SC_ERR_TIMEOUT -1011` が `capture()` / `grab*()` の実行中に発生するのは別の問題です — そちらは実際のエラーです。**
> 上記の注記は、**connect-time プローブ**によって記録された `-1011` についてのみです。**キャプチャ** から発生した同じエラーは、カメラは正常に接続されたものの、画像を送信していないことを意味します：
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> 決定的な手がかりは、*制御*チャネルが正常である（検出が機能し、設定および `[chunk-enable …]` への書き込みがすべて成功している）にもかかわらず、*すべての*フレームでタイムアウトが発生するカメラです。
>
> **一般的な原因は、カメラがハードウェアトリガー用に武装状態になっていることです。** `trigger_mode="On"`および`trigger_source="Line2"`の場合、M8同期ケーブルに電気的なエッジが到着するまで、カメラは一切何も送信しません。 そのラインを駆動するケーブルがない場合、すべてのグラブ処理は永遠に待機したままになります。カメラは故障しておらず、ネットワークにも問題はありません — 指示された通りに動作しているのです。
>
> `CameraSettings()` および `default` / `high_speed` / `high_quality`はフリーランをプリセットしており、アーム状態中にタイムアウトしたグラブは、単に「`-1011`」と表示されるのではなく、その理由が説明されます。`PRESETS["triggered"]`は、設計上、Line2をアームします。
>
> 任意のカメラを強制的にフリーラン状態にするには：
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> `trigger_mode="Off"` を実行してもタイムアウトする場合は、カメラが実際にデータを送信していないことになります。ログと `ip link show` を弊社までお送りください。

#### カラープロファイル（RGBライブプレビュー） — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)`は、RGBカメラでの**ライブプレビュー**に使用するディスプレイのカラープロファイルを選択します（マルチスペクトルカメラはこの設定を無視します）：

| プロファイル | 意味 |
| --- | --- |
| `raw` | 放射測定チェーンを完全にバイパスします。 |
| `linear` | DSNU + フラット + WB、CCMなし、ガンマなし。 |
| `natural` | リニア + 測定済みCCM + sRGBガンマ、簡易処理のみ（彩度平滑化 + ハイライトの彩度低下） — リアルなデフォルト設定。 |
| `enhanced` | `natural` に、フルハブパリティ仕上げ（デフリンジ、ヴィブランス、CLAHE ローカルコントラスト）を追加。より豊かな画質ですが、**1フレームあたりの仕上げコストが約2倍**となるため、LIVEのフレームレートは低下します。 |
| `custom_temp` | `natural` ただし、ホワイトバランスは `custom_cct_k` ケルビン （DLSは無視され、バックエンド側で2000～10000 Kにクリップされます）。 |

このプロファイルは**ライブプレビュー専用**の速度／画質調整機能です。保存されたキャプチャは、選択されたプロファイルに関係なく常に完全で豊かな仕上げが適用されるため、フレーム時間を確保するために`natural`を選択しても、ディスクに書き込まれる画質は低下しません。 未知のプロファイルが選択されると `ValueError` が上昇します。chloros バックエンドに接続可能な場合、その変更はバックエンドにも POST されるため、次のプレビューフレームに反映されます（バックエンドを持たない direct-SDK ユーザーでも、設定の変更は反映されます）。

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### ハイライト（RGBディスプレイ） — `set_highlights`

`LatticeCamera.set_highlights(value)`は、**ディスプレイ**の最も明るく、最も白いピクセルを RGBカメラでレンダリングされる際、最も明るく白いピクセルを抑制します。これは、明るい白い表面が単調な白い塊としてレンダリングされてしまう問題を修正するものです。`value`は、`-100`..`0`（クリップ処理済み）です。`0`はオフになっており、この設定がない場合とまったく同じレンダリング結果になります。設定値を高くすると、ディスプレイの白が低下し（`-100`では約60％まで）、トーンカーブが広がります。 そのため、プレビューの自動輝度調整によって純白に押し上げられていた表面のディテールが回復します。

これはディスプレイ制御ですが、彩度やコントラスト（ライブプレビューのみを変更する）とは異なり、保存される内容も変更します。ライブプレビューおよび**ディスプレイレベル**でのキャプチャはこの設定に従うため、それらのファイルも暗く出力されます（XP003332の設定が反映されます）。 ラディアンスおよびリフレクタンス形式でのエクスポートには影響せず、クリップしたセンサーデータを復元することはできません — その場合は露出を下げてください。マルチスペクトルおよびモノクロカメラはこの設定を無視します。値は `settings.highlights` に保存され、到達可能な chloros バックエンドに POST され、 これは `set_color_profile` と全く同じです。CLI での対応コマンドは `lattice highlights --set N` です。

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### モノクロ (M3M) カメラと `Calibration`

モノクロ **M3M** カメラ (`M3M-<lens>-F<wavelength>`) はシングルバンドです： 1つのグレースケール平面を持ち、ベイヤーモザイクも3×3のスペクトルクロストーク行列もありません。`Calibration`はこれを認識し、`is_mono`フラグを公開します。 反射率は依然としてバンドごとの放射測定マップとして適用されます（アンミックスは単位行列となります）が、単一のカメラに対するマルチバンド演算は、無意味な結果を返すのではなく、適切な処理を行います：

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

モノクロハードウェアから植生指数を構築するには、異なる波長の複数の M3M カメラをアラインメント済みのマルチバンド・スタックに結合し（[アレイのアラインメント](#array-alignment)を参照）、1 台のカメラではなくそのスタック全体に対して指数を計算します。

#### DAQ-Mのバンドマグニチュード — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")`は、独自のBLE MACを使用してBluetooth経由でDAQ-Mに接続します。 これは、バックエンドを経由せずセンサーと直接通信する唯一の光センサークラスであるため、ダイレクトカメラクラスに必要なもの（`CAMERA_AVAILABLE`：Arena SDKランタイムおよびOpenCV）に加え、Bluetooth用の`bleak` （`pip install "chloros-sdk[camera]"` または `pip install bleak`。これがないと、`DLS` が `DLSError` を発生させます）。別の場所に接続された DAQ-M （バックエンドのプール、プロジェクト、GUI）はアドバタイズを停止するため、まずそこで解放してください。 **`mac_address` が必要です**：各ユニットごとに固有の値があるためデフォルトは存在せず、これを指定しない `DLS()` では、接続を試みる前に `DLSError` が発生します。XPR を使用して MAC アドレスを特定してくださいOTX003351（[ディスカバリー](#discovery--finding-an-address-to-connect-with)を参照）または`chloros-cli daq pool-discover --only ble`を使用してMACアドレスを特定してください。

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)`は、カメラの3つのチャネル応答を通じてDAQ-Mスペクトル（`spectrum`が`None`のときの最新の測定値； まだない場合は`DLSError`）をカメラの3つのチャンネル応答を通じて統合し、`{"red": …, "green": …, "nir": …}`を返します — キーはその順序でチャンネル名を指定します。RGBカメラの曲線の場合、3番目（`nir`）が青チャンネルとなります。

- `spectral_response`を指定しない場合、フィルターモデルのフリートカーブ（`filter_model`が構築された`DLS`）が使用されます。これは1台のユニットで測定されたものです。
- `spectral_response`は、代わりにカメラの**独自の**測定カーブを使用します： そのカメラのキャリブレーションパック（`color_correction.spectral_response_smoothed` — RGBカメラ用；マルチスペクトルパックにはこのようなカーブは含まれない）に保存されている辞書（dict）`{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`です。使用できない辞書の場合は、フリートカーブにフォールバックします。

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLIの`chloros-cli lattice dls --camera SERIAL`もバックエンドを通じて同様の処理を行い、読み込まれた、またはキャッシュされたキャリブレーションパックからカメラのカーブを検索し、どのカーブを使用したかを報告します。 HTTP 経由では、`POST /api/dls/cli-read` と `mac` （必須：これがないと、応答はHTTP 400となり、`code` `MAC_REQUIRED`となる）となり、本文には`camera_serial`が含まれます。 応答の `band_magnitudes_source` は、そのカメラ固有のカーブが使用された場合は `camera <serial>` となり、フリートカーブが使用された場合は `model <filter>` となります（カーブのないカメラ：RGB ではない、またはパックが読み込まれていない 、またはキャッシュされていない）の場合は、`model <filter>`となります。

---

## 例外

「Chlorosで発生したあらゆる問題」を処理するために基底クラスをキャッチします：

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` および `ChlorosConfigurationError` は `chloros_sdk` （および `chloros_sdk.exceptions`）の属性ですが、`__all__` には含まれていないため、`from chloros_sdk import *` ではこれらが読み込まれません。名前を指定してインポートしてください。`ChlorosError` ファミリーは、`ChlorosLocal` およびそれに基づいて構築された関数（`process_folder()`、`process_lattice_capture()`）によって派生しています。 スマートコネクトサーフェスは `ChlorosConnectError` を継承します。

階層：

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## エンドツーエンドの例

### 1. カスタムプログレスバーを使用してフォルダを処理する

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. ライブ LATTICE アレイ → 反射率 + DAQ リファレンス

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. プロジェクト主導型のキャプチャキャンペーン

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. マルチカメラのフレームストリーム → NumPyパイプライン

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. ヘッドレス・ダイレクト・ハードウェア（バックエンドなし）キャプチャスクリプト

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. 4カメラアレイ接続前の機能確認

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. キャプチャ・レシピの同等実装（純粋なPython）

`run_project_recipe()`は、CLIレシピをそのまま実行します（[プロジェクトフォルダとレシピ](#project-folder--recipes)を参照）。代わりにPythonで記述した場合、同じ手順は以下の通りです：

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCPサーバー（AIアシスタント）

`chloros-mcp` を使用すると、Claude Desktop、Claude Code、Cursor など、[Model Context Protocol](https://modelcontextprotocol.io) (MCP) を扱う AI アシスタントが、Chloros を操作できるようになります。具体的には、画像の処理（プロジェクトの作成、キャプチャ画像のフォルダのインポート、 設定の選択、処理、実行結果のエクスポート報告）を行い、LATTICEカメラ、アレイ、DAQ光センサー（接続、プレビュー、露出・色の設定、キャプチャ、記録、ファームウェアの更新）を操作できます。そのツールは、このSDKの機能範囲を網羅しています。ユーザーが平易な言葉で指示すると、アシスタントがサーバーのツールを呼び出します：

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

サーバーはこのSDKを基盤として構築されています。画像処理ツールは[`ChlorosLocal`](#image-processing--chloroslocal)を呼び出し、ハードウェアツールは[smart-connectが呼び出す](#smart-connect-for-lattice-cameras)、[DAQセッション](#daq-sensor-sessions)、`ChlorosProject`、および[カメラユーティリティ](#camera-utilities)を呼び出します。これらはすべて、Chlorosアプリが使用するのと同じバックエンドと通信します。そのリクエストには`X-Chloros-Client: sdk`が含まれるため、 そのため、バックエンドはSDKのサインインおよびプランのチェックを適用し、SDKのルールもそれに伴って適用されます（例えば、LATTICEによるキャプチャではTexture Awareが拒否されます）。

### インストール

サーバーは `chloros-sdk` パッケージに含まれており、他にインストールする必要はありません。SDK は `chloros-mcp` コマンドをインストールしますが、これは `python -m chloros_sdk.mcp_server` と同じものです：

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** ChlorosインストーラがSDK環境にSDKをpipでインストールし、そのPythonの`Scripts`フォルダに`chloros-mcp.exe`が配置されます。
- **Linux（.deb）：** このパッケージは、システム用のSDKを`python3`にインストールし、`chloros-mcp`コマンドを`/usr/bin`に配置するため、すべてのユーザーの`PATH`に追加されます。virtualenv内では、`pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`がvirtualenvの`bin`に`chloros-mcp`を追加します。

サーバーは、SDK（Python 3.7 以上）の上に標準ライブラリのみを使用します。MCP クライアントがサーバーを起動し、stdin/stdout 経由で通信を行います。`--help` を使用する場合を除き、ユーザー自身がサーバーを実行する必要はありません。

### クライアントの設定

**Claude Desktop:**サーバーを `claude_desktop_config.json` に追加し（Windows では `%APPDATA%\Claude\claude_desktop_config.json`；**[設定] > [開発者] > [設定の編集]** から開けます）、その後 Claude Desktop を再起動してください：

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claudeコード：**

```bash
claude mcp add chloros -- chloros-mcp
```

**カーソル:** Claude Desktop と同じ `mcpServers` エントリを、`~/.cursor/mcp.json`（すべてのプロジェクト；Windows では `%USERPROFILE%\.cursor\mcp.json`）または `.cursor/mcp.json`に設定されます。

オプションは`args`に指定するか、Claude Codeのコマンド（`claude mcp add chloros -- chloros-mcp --no-auto-start`）の後に指定します。 `chloros-mcp`上のXPPythonのSDKの代わりに特定のPythonのSDKを使用するには、 そのPythonでモジュールを実行します（`PATH`上の最初の`python3`でない場合は、そのフルパスを`command`として指定します）：

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

クライアントはシェルを介さずにサーバーを起動するため、サーバーの起動に失敗したという報告があった場合、クライアントが認識している`PATH`には`chloros-mcp`が存在しない可能性があります。 その代わりにフルパスを指定してください（Windows では `where chloros-mcp`、Linux では `which chloros-mcp`、ここで .deb ファイルのパスは `/usr/bin/chloros-mcp` です）。あるいは、SDK が含まれる Python でモジュールを実行してください。

### オプション

| オプション | デフォルト | 機能 |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | 使用する Chloros バックエンド。 |
| `--no-auto-start` | off | バックエンドを起動しない：ChlorosアプリやLinuxバックエンドサービス、あるいは別のマシンの （[リモートバックエンドモード](#remote-backend-mode-pip-only-host-via-tunnel)を参照；この場合、フォルダパスは両マシンで同じフォルダを指す必要があります）。 |
| `--backend-exe PATH` | Chlorosがインストールする場所 | SDKが検索対象外となっている場所にChlorosがインストールされている場合、バックエンドが実行されていないときに起動するバックエンド。 |
| `--log-level LEVEL` | `WARNING` | `DEBUG`、`INFO`、 `WARNING` または `ERROR`。ログは stderr に出力され、ほとんどのクライアントでは MCP ログに表示されます。stdout にはプロトコルメッセージのみが送信されます。 |

### ツール

各ツールの引数は、そのツールがラップしている SDK 呼び出しの引数と同じです。クライアントは、それぞれの引数を完全なスキーマとともに一覧表示します。パスは絶対パスです。

**処理**

| ツール | SDK 呼び出し | 機能 |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | プロジェクトを作成し、それを開いているプロジェクトとして設定します。処理ツールはこのプロジェクトに対して動作します。単純な名前を指定すると、デフォルトのプロジェクトフォルダ内にフォルダが作成され、絶対パスを指定するとその場所にフォルダが作成されます。 |
| `open_project` | `ChlorosLocal.open_project` | 保存済みのプロジェクトを開き、それを開いているプロジェクトとして設定します。 移動されたプロジェクトのパスは、現在の所在フォルダを指すように再設定されます。`relink_folder`（絶対パス）は、その他の欠落ファイルが現在格納されているフォルダを指定します。 `source_files` および `image_count` を返します。見つかりませんでした：それらの場所のリスト作成にエラーが発生し、`relink_folder` と名付けられました。何も開かれていません。 |
| `import_images` | `import_images` | Survey3の `.RAW` と `.JPG` のペア、「LATTICE」の `.tif`のキャプチャおよびDAQ `.daq`の記録を行い、`count`（画像）および`scan_count`（記録）を返します。 |
| `configure_processing` | `configure` | 渡された設定のみを変更し、それ以外の設定は変更しない（値：[サポートされている値](#supported-values)）； `custom_settings` は、指定された設定全体を送信する。 |
| `get_project_settings` | `get_config` | 開いているプロジェクトの保存済みの処理、エクスポート、およびインデックス設定を読み取ります。 |
| `start_processing` | `process(wait=False)` | 開いているプロジェクトの処理を開始し、直ちに返ります。 |
| `get_processing_status` | `get_status` | `state` (`idle`、`starting`、`processing`、または`finished`）と進行状況を報告します。 完了後、エクスポートの検証ができなかった場合は`summary`、`export_check_failure`を、画像が失われた場合は`image_failure`を記録する。その他の成果物は書き出される。実行を失敗として報告する。 |
| `process_folder` | `process_folder` / `process_lattice_capture` | フォルダ用のプロジェクトを作成し、それをインポートして、設定を行い、1回の呼び出しで実行を開始します （LATTICEキャプチャの場合は`lattice: true`）。実行が進行中の場合、何かを作成する前に処理を拒否し、その間、バックエンドもどのツールからのプロジェクト変更も拒否します。 |
| `export_light_sensor` | `export_light_sensor` | プロジェクトの光センサー記録用に、キャリブレーション済みの `.daq` および `.csv` ファイルを書き込みます 記録用にキャリブレーション済みの `.daq` および `.csv` ファイルを書き込みます。この際、画像の処理は行わず、キャッシュされていない場合は MAPIR Cloud から工場出荷時のキャリブレーションを取得します（[光センサーの記録](#light-sensor-recordings--calibrated-daq--csv) を参照）。 |
| `read_image_audit_tags` | `read_image_audit_tags` | 処理済み画像のChloros監査メタデータ（ソース、キャリブレーション、適用されたステージ）を読み取ります。 |

**カメラとアレイ**

| ツール | SDK呼び出し | 機能 |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`、`list_daq_sensors` | 任意のクライアントで現在接続されているもの（それぞれが `connected_here` を有するもの）、およびサーバーで実行中のジョブ。 |
| `discover_devices` | `discover_lattice_cameras`、`discover_daq_sensors` | 接続可能なカメラおよび光センサー。 |
| `get_device_status` | `CameraSession.status`、`ArraySession.status` / `capability`、 `DAQSensorSession.status` | 1台のカメラ、アレイ（その機能および健全性を含む）、または光センサーのリアルタイムの状態。 |
| `analyze_array_network` | `analyze_array_network` | 提案されたアレイに対して、このホストおよびネットワークが処理可能な容量。 |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | カメラ、または同期されたアレイ（マスターを先頭に）を接続する。 |
| `connect_project_devices` | `open_project(...).connect_all` | 保存されたプロジェクトが記憶しているすべてのデバイスを、その設定とともに接続します。 |
| `disconnect_device` | `disconnect` / `force_disconnect` | カメラ、アレイ、または光センサーを解放する（「動作」を参照）。アレイ上の `keep_settings: true` は各カメラの設定を独自のメモリに保存し、`clear_settings: true` はそのメモリをリセットします （その結果には `camera_memory` が含まれます）。 |
| `disconnect_project_devices`、`align_project_arrays`、`capture_project_devices` | `disconnect_all`、`align_arrays`、`capture_all` | プロジェクトのデバイスをまとめて。 |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | フレンドリー名または GenICamノード名によるカメラ設定の読み取りまたは書き込み。 |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | 配列のレートを固定するか、自動レートに戻す。 |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | 1回のキャプチャ（`fastest`および`wait_for_exposure`は`capture_fastest`および露出が確定したキャプチャです）。`output_dir`がない場合、開いているプロジェクトの`captures`フォルダに保存され、ファイルはプロジェクトに追加されます（処理の実行が終了するまで保留されます）。応答の `project_add` にはその旨が記載されています。 |
| `start_capture_run` | `ArraySession.continuous`、`capture_repeated` | ジョブとしての連続または間隔撮影。 |
| `start_array_recording` | `ArraySession.record` / `burst` | 統合インデックス付きビデオ、または生バーストを、ジョブとして処理します。 |
| `build_burst_video` | `ArraySession.build_video` | バーストをキャリブレーション済みビデオに再処理する（ジョブとして）。 |
| `get_job_status` / `stop_job` | | ジョブの追跡または停止。 |
| `get_camera_preview` | | 各カメラのライブプレビューを画像として返す。 |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | 処理レベルごとの 1 フレーム（または 1 セットの同期データ）を保存せずに取得：プレーンごとの統計情報と各プレーンのPNG画像。 |
| `calibrate_array_alignment`、`get_array_alignment`、`clear_array_alignment`、`nudge_array_alignment`、`export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`、`tweak_alignment`、`export_alignment`、`import_alignment` | 配列のアライメント。 |

**光センサー**

| ツール | SDK呼び出し | 機能 |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | DAQ-U、DAQ-M、またはDAQ-Eを接続します（`cap_id`はキャップ補正を設定します）。 |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | 最新の測定値：タイムスタンプ、飽和度、スペクトル範囲およびピーク、ならびに波長ビンごとに平均化されたスペクトル（各ポイントごとのデータは `full_spectrum`）。 |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | ストリームの開始または停止。 |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | `.daq` ファイルをジョブとして記録します。 `duration_s`は自動的に停止します。 |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | キャップ補正 （`none`：センサー単体の場合）。 |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | 各カメラおよびアレイの反射率データにどのセンサーが供給されるか。 |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | バンドセット、および2つのストリーミングセンサーの環境光＋対象物の反射率と屈折率。 |
| `get_light_sensor_device_info`、`set_light_sensor_network`、`set_light_sensor_auto_stream`、`enable_light_sensor_ptp`、`reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`、`set_auto_stream`、`enable_ptp`、`reboot`、 `update_firmware` | DAQ-E本体に保存されている設定。 |

**カメラユーティリティ** ([カメラユーティリティ](#camera-utilities))

| ツール | SDK呼び出し | 機能 |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`、`camera_color_adjust`、`camera_highlights` およびそれらのセッター | RGBカメラの表示色： プロファイルおよび色温度、彩度／コントラスト、ハイライト。 |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | ワンショットホワイトバランス、またはオート／オフ／固定比率。 |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | カメラのプリセット。 |
| `list_filter_models` | `camera_filter_models`、`camera_filter_model_info` | フィルターモデル、または名称による選択。 |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | ファームウェアのバージョン、およびフラッシュ（デフォルトではジョブ）。 |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`、`wake_camera` | 電源および熱状態；スリープまたはウェイク。 |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | 表示中のキャリブレーションターゲットからの係数。 |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | カメラのネットワークアダプタ、推定値、および修正 （管理者専用）。 |
| `apply_vignette_correction` | `apply_vignette_correction` | 画像フォルダに対するヴィネット補正。 |
| `read_dls` | `read_dls` | 1回のDLS読み取り。 |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | ホストのハードウェアおよびランタイム。 |

**トランジェントカメラ、アライメントプロファイル、およびインデックス** ([トランジェント配列、アライメントプロファイル、およびインデックス](#transient-arrays-alignment-profiles--indices))。ここでカメラを開くすべてのツールは、`keep_settings` / `clear_settings`（デフォルトは false）を受け取ります：[カメラメモリ](#camera-memory--keep_settings--clear_settings)を参照してください。

| ツール | SDK呼び出し | 機能 |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | 1台のカメラを録画するか、複数のカメラを同期してストリーミングするジョブとして実行する （`duration_s`が自動的に停止します）。 |
| `capture_cameras_transient` | `capture_cameras_transient` | アレイを接続せずに、1つの同期キャプチャを実行します。 |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | トリガー同期の確認、および同期ラインに接続されているカメラの確認。 |
| `analyze_array_models` | `analyze_array_models` | これらのモデルのアレイをこのネットワークに接続した場合の動作。 |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | アライメントプロファイルファイルを解析して保存します（`options`には、より詳細なソルバーオプションが格納されています）。 |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | アライメント済みのマルチバンド画像、またはジョブとして処理される一連の画像。 |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | プロファイルファイルの内容。バンドの順序および有効なカメラ。 |
| `list_index_presets`、`compute_index`、`capture_live_index` | `index_catalog`、`compute_index`, `capture_live_index` | プリセットおよびグラデーションのインデックス；マルチバンドTIFFのインデックス；1ステップでのキャプチャからインデックス作成。 |

**時刻同期、プロジェクトフォルダ、およびレシピ** ([PTPグランドマスター](#ptp-grandmaster--time_sync_status)、[プロジェクトフォルダとレシピ](#project-folder--recipes))

| ツール | SDK呼び出し | 機能 |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`、`time_sync_peers`、 `time_sync_cameras` | PTPグランドマスターの健全性、そのピア、および各アレイカメラのPTP状態。 |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | 再起動し、優先順位を設定する。 |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | 新しいプロジェクトが作成されるフォルダ。 |
| `run_project_recipe` | `run_project_recipe` | 保存済みのプロジェクトのレシピを、ログ付きのジョブとして実行します。 最初にチェックが行われます：エラーのあるレシピは拒否され、何も実行されません。接続されていない配列／センサーについては警告が表示されます（接続は行われません：まず `connect_project_devices` が実行されます）。`dry_run` はチェックを返し、何も実行しません。 |

保存されたプロジェクトを開いて処理するためのツールは存在しません。各処理ジョブは `create_project`（または `process_folder`）から開始されます。

### 要件

- **同じコンピュータにChlorosがインストールされていること。** 処理はChlorosのバックエンドが行います。代わりに、トンネル経由でアクセス可能なバックエンドを使用することも可能です（`--backend-url`と`--no-auto-start`を併用。詳細は[オプション](#options)を参照）。
- **そのコンピュータでログイン済みであること**。Chlorosアプリ、または`chloros-cli login`を使用してログインしてください。サーバーが代わりにログインすることはできません。ログアウト状態でツールが呼び出されると、「Chlorosにログインしていません」というメッセージがバックエンドの理由とともに表示され、アシスタントが通知します。
- **有料のChloros+プラン（Copper以上）**。サーバーはSDK経由でアクセスされますが、無料のIronプランにはこの機能がありません。無料プランでツールを呼び出すと、バックエンドの理由を添えて「このChlorosプランでは利用できません」という応答が返されます。
- SDK 3.7 以降（SDK を含む）および、ローカル（stdio）サーバーを起動する MCP クライアント。

### 動作

- **バックエンド。** サーバーの起動やツールのリスト表示だけでは何も開始されません。最初のツール呼び出し時に接続が確立されます。Chlorosアプリ（またはLinuxバックエンドサービス）が実行中の場合、サーバーはそのバックエンドを使用し、実行状態を維持します。 バックエンドが実行されていない場合、サーバーはインストール済みのバックエンドをウィンドウなしモードで起動します。サーバーが終了すると（クライアントを終了するか、サーバーを再起動した場合）、サーバーが開始したキャプチャジョブや光センサーの記録を停止し、接続していたデバイスを解放します。バックエンドは実行され続けるため、 未完了の処理は最後まで継続され、30分間何も使用されていない状態が続くとバックエンドは自動的に停止します。`--no-auto-start`の場合、サーバーはバックエンドを起動しません。バックエンドが実行されていない状態でツールを呼び出すと、Chlorosアプリを開くよう求められ、アプリを開いた後の次の呼び出しで接続が確立されます。
- **開いているプロジェクトは1つ。** バックエンドには開いているプロジェクトが1つあり、Chlorosアプリが開いている間はアプリと共有されるため、アシスタントが動作している間はアプリ内で別のプロジェクトを操作しないでください。
- **渡された設定のみが変更されます。** `configure_processing`は、指定されていない設定についてはすべて保存された値のままにします（SDKの`configure()`は、デベイヤー、ヴィネット補正、反射率キャリブレーション、PPK、およびエクスポート形式が指定されていない場合、それらをデフォルト値にリセットします。 サーバーがプロジェクトからそれらを補完します）。プロジェクトの保存済み設定を読み取れない場合、5つすべてが指定されていない限り、何も変更せずその旨を通知します。`indices`はプロジェクトのインデックスリストを置き換えます。結果には、`settings_sent`として送信された内容を一覧表示し、`warnings`の下には、カメラのバンドでは作成できないインデックス名を列挙します。
- **処理は「開始後ポーリング」方式です。** `start_processing`は実行が開始されるとすぐに返却され、その後数分から数時間かかります。 、アシスタントはそれに続いて `get_processing_status` を実行します。`state` は、`idle`（処理なし）、`starting`（バックエンドがまだ実行を報告していない状態、最大20秒間）、`processing`、または`finished`となり、実行の [サマリー](#post-run-summary--hints)：エクスポートされた内容、および期待された量より少ない場合（画像なし、キャリブレーションターゲットが見つからないなど）のヒント。出力データは、アプリと同様にプロジェクトフォルダの下に書き込まれます。 `start_processing`は実行中に拒否され、応答を停止したバックエンドは完了した実行としてではなく、エラーとして報告されます。
- **アシスタントが認識する内容。** SDKの警告は、`warnings`のツール結果として返され、失敗（サインアウト、プラン、不正なパス）は、アシスタントが読み取ってユーザーに伝達できるエラーテキストとして表示されます。 20項目を超えるリストは最初の20項目に切り詰められ、その全長は「`<key>_total`」として表示されます。パスは絶対パスで指定する必要があります。
- **デバイスは共有されています。** カメラ、 アレイ、光センサーはバックエンドに存在するため、アシスタントはChlorosアプリに接続されたデバイスを操作でき、その変更内容はアプリに反映されます。`disconnect_device`はサーバーが接続したデバイスを公開し、別のクライアント（アプリ、`chloros-cli`、 スクリプト、別の`chloros-mcp`）は、SDKの`force_disconnect`である`force: true`によってのみ解放され、バックエンドが認識している場合は、それを接続したユーザーの名前が拒否情報として表示されます。 バックエンドがまったく保持していないデバイスは、未接続として報告されます（`released: false`、`connected: false`、解放する対象なし）。バックエンドが実行する録画またはストリームは、その `get_job_status` ビューを `counts` — `frames_written`、`frames_dropped`、 `frames_failed`、`drop_reason`（マルチカメラストリームのフレームグループ） — および、実行中の1行の`summary`（「220フレーム書き込み、55フレーム破棄、0失敗：...」）、実行完了時の最終数値、別のクライアントによるそのリスト （`other_clients_jobs`）には同様の数値が記載されており（`list_cameras()`、`list_arrays()`、および`list_daq_sensors()`には各デバイスの`connected_by`：`client`、`label`、 `name`、`at`）を保持します。 各`chloros-mcp`は、バックエンドに対して自身を`chloros-mcp (pid N)`として登録します（`chloros_sdk.connect.CLIENT_LABEL`は、`X-Chloros-Client-Label`として送信されます。任意のSDKスクリプトでこれを設定可能です）。他のクライアントの光センサー記録を停止する場合も、`force` が必要です。
- **長時間実行される処理はジョブです。** `start_capture_run`、 `start_array_recording`、`build_burst_video`、`start_light_sensor_recording`、`start_camera_continuous`、 `start_multi_camera_stream`、`start_aligned_stream`、`run_project_recipe`、 また、`background` に対するキャプチャ、ソルブ、インデックスの要求があった場合は、直ちに `job_id` を返す。続いて `get_job_status` が行われ、`stop_job` で終了する 。実行は、`max_captures`、`duration_s`（またはバーストの場合は`max_frames`）を指定しない限り、停止されるまで継続します; `min_free_gb`は、ドライブの空き容量が不足している場合、起動を拒否します。制限を設定しない限り、制限は適用されません。バックエンドが実行する録画またはストリーム（`start_camera_continuous`、 `start_multi_camera_stream`、`start_aligned_stream`）はバックエンドのジョブIDを保持するため、別の`chloros-mcp`（後から起動されたもの）はそのIDによってそれを検出します： `get_job_status`はそれに続きます（そして、`other_clients_jobs`、`GET /api/camera/cli-jobs`、それぞれに`started_by`といったジョブを一覧表示します）、 そして、`stop_job`は、`force: true`によってのみ停止されます。その他のジョブ （キャプチャ実行、アレイ記録またはバースト、バックグラウンドキャプチャ）は、それを開始した`chloros-mcp`で実行され、それを認識しているのはそのサーバーのみである： 別のサーバーがそれを問い合わせた際に、その旨が返されます。停止されたジョブは、キューを書き出している間は `stopping` を読み取り、その後、最終カウントを含む `finished` を読み取り、 10分間、そして任意の `chloros-mcp` から： 開始したサーバーは、他のサーバーの強制停止も同様に認識します。`get_job_status`は停止を待たず、停止中（自身の`stop_job`、他のクライアントのもの、あるいは自力で終了したジョブのカメラを解放する停止）の最中でも、`stopping`とリアルタイムのカウントを即座に返答し、すでに停止中のジョブの`stop_job`はそれを通知する。終了したジョブは、`stop_reason`に終了理由を記載します：自身の理由（`no_frames`、`disk_full`、`member_missing`、 または配列の連続実行の場合は `count`、`duration`、`array_gone`）、`limit reached` はそのカウントが `count`（またはバーストの`max_frames`、記録の`duration_s`）に達した場合のみ、`error`、`stopped` （`stopped_by`を含む）クライアントによって停止された場合、それ以外の場合は`finished`。
- **キャプチャデータの保存先。** 開いているプロジェクトの `captures` フォルダに保存されます。ただし、`output_dir` が指定されている場合は除きます。
- **画像。** `get_camera_preview` および `grab_frames` は、JSON データに加えて、アシスタントが確認できる画像を返します。`grab_frames` の画像は、表示用にコントラストが調整されていますが、統計値は実際の値です。
- **サーバーが終了すると**、サーバーが開始したジョブは停止し、サーバーが開始した光センサーの記録は閉じられます。サーバーがバックエンドを起動していた場合、バックエンドを停止する前に接続していたデバイスも解放されます。アプリのバックエンドでは、デバイスは接続されたままになります。
- **プロトコル。** stdioのみ（HTTPエンドポイントなし）、ツールのみ（リソースやプロンプトなし）、テキストおよび画像の結果、MCPプロトコルバージョン2024-11-05～2025-11-25。**対象外となる機能。** ログインおよびログアウト（アカウントはユーザー自身のものです。サーバーはログインできません）、およびサーバーが起動していないバックエンドの停止。このSDKが処理やハードウェアに関して行うその他のすべての機能には、対応するツールが用意されています。（DAQ-Eのプロビジョニング――その名前、認証トークン、 OTAパスワード、署名要件、工場出荷時設定へのリセット――およびダイレクトDAQサーバーは`chloros-cli`専用であり、SDK内でも同様です。名前やトークンが間違っていると、すべてのChlorosインストールがユニットからロックアウトされます。）

---

## バックエンドの自動起動

スマートコネクトのエントリポイント――`connect_camera`、`connect_array`、 `connect_daq_sensor`、`discover_lattice_cameras`、および `discover_daq_sensors` — は、バックエンドが `127.0.0.1:5000` （スマートコネクト・サーフェスのデフォルトURL）でリスニングしていることを前提としています。GUI や CLI がすでに実行中の場合は、バックエンドが動作しています。スクリプト単体で実行する場合、 それらが実行されていない可能性があります。そのため、これらの関数は、最初の呼び出しの前に**バンドルされたバックエンドバイナリを自動起動**し（`ChlorosLocal`と同様にウィンドウレスで実行）、起動するまで最大60秒待機します。 `list_cameras()`、`list_arrays()`、`list_daq_sensors()`、 `attach_array()` および `analyze_array_network()` は、バックエンドを起動することはありません。

ルール：

- **生成されるのはローカルURLのみです。** `backend_url`が`localhost` / `127.0.0.1` / `[::1]` を指す `backend_url` のみが対象となります。それ以外のホストは他者のマシンであるとみなされ、決して起動されません。
- **応答する前に終了したバックエンドは、待機を終了させます。** この場合、60秒間待機する代わりに、終了コード、実行されたコマンド、およびバックエンドが出力した最後の行を即座に返します。ただし、終了コード98は例外です： 別のバックエンドがすでにポートまたはロックファイルを保持している（Chlorosアプリ、バックエンドサービス、またはまだ起動中の別のスクリプトによるもの）ため、呼び出しはそのバックエンドからの応答を待ち続けます。`ChlorosLocal`も同様に動作します。
- **バックエンドは再利用のために実行されたままになります**（CLI と同様）— スクリプトが終了しても暗黙的なシャットダウンは行われません。スクリプトを再実行すると、稼働中のバックエンドが再利用されます。
- **これらの呼び出しのいずれかで `auto_start_backend=False`** を使用してこの動作を無効にできます（例：リモートバックエンドを指定している場合、またはバックエンドのライフサイクルを自分で管理している場合）。

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

バンドルされたバイナリが見つからない、または起動できない場合、その後のHTTP呼び出しでは、単なる接続拒否のトレースではなく、対処可能な、**プラットフォームを意識した**`ChlorosConnectError`がスローされます — Windowsではデスクトップアプリまたは`chloros-cli`コマンドを、Linux（GUIなし）では`chloros-cli`コマンドまたは`.deb`を提示します。

---

## 環境とヘッダー

SDKは、すべてのバックエンドHTTP呼び出しに`X-Chloros-Client: sdk`を付与します。バックエンドでは、GUIの無料プランのパスではなく、SDK/CLIのライセンス規則（ログイン**および**有料のChloros+プランが必要）が適用されます。 これはインポート時に自動的に設定されるため、ユーザー側で何かを行う必要はありません。

このヘッダーは、URLに`localhost`または`127.0.0.1`を含むすべての`requests` / `urllib` リクエストに追加されます。URL に `localhost` または `127.0.0.1` が含まれる場合、ポート番号を問わず（自身のローカルサービスも含む） — これにより、ユーザー自身が設定した値が上書きされます。他のホストへの呼び出しは変更されません。

`backend_url=` を渡すことで、バックエンドの URL を上書きできます（`ChlorosLocal` は `backend_url=` または `api_url=`を受け付けます。`process_folder()`および`process_lattice_capture()`は、`api_url=`のみを受け付けます）:

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

（インストールされたバックエンドはループバックでのみリスニングするため、LAN上の `backend_url` からは到達できません。トンネルパターンについては、[リモートバックエンドモード](#remote-backend-mode-pip-only-host-via-tunnel)を参照してください。)

---

## バージョン管理と互換性

- SDKのバージョンは`chloros_sdk.__version__`として公開されます。
- SDKは、同梱されているバックエンドのバージョンに動作を固定しています。古いSDKと新しいバックエンドを併用しても通常は問題なく動作しますが（前方互換性のあるエンドポイント）、新しいSDKと古いバックエンドを併用すると、新しいエンドポイントで`404`エラーが発生する可能性があります。その場合は、デスクトップアプリを対応するバージョンにアップグレードしてください。
- スマートコネクトインターフェース（`connect_camera` / `connect_array` / `connect_daq_sensor`）およびネットワーク分析エンドポイントは、安定したJSONスキーマを返します。新しいフィールドは追加される形となります。

---

## トラブルシューティングのヒント

- **`ChlorosAuthenticationError: Login required`**（`ChlorosLocal` から発生。スマートコネクトの呼び出しでは、同じ原因で `ChlorosConnectError: Login required. …` が発生します） → このマシンで `chloros-cli login EMAIL PASSWORD` を 1 回実行するか、Chloros デスクトップアプリ経由でサインインしてください。
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`**（または、`create_project()` から発生する `Failed to create project: …`）→ このマシンでは Windows の長いパスが無効になっています（Windows のデフォルト設定）。 そのため、パスは260文字を超えることができず、製品はプロジェクトフォルダの下に最大約130文字の位置に配置されます。何も処理されませんでした。このメッセージには、収まる最長のプロジェクトフォルダ名が表示されています。より短いフォルダ名を使用するか、Windowsの長いパスを有効にしてChlorosを再起動してください。 Windowsインストーラーには「Windowsの長いパスを有効にする」というオプションがあり、デフォルトでチェックされています。それ以外の場合は、グループポリシーを使用してください：*コンピューターの構成 > 管理用テンプレート > システム > ファイルシステム > Win32の長いパスを有効にする*。
- **`ChlorosConnectError: No Chloros backend is running …`** → スマートコネクトはローカルバックエンドの自動起動を呼び出すため、このエラーはバンドルされたバイナリが見つからない、または起動できない場合（例：デスクトップパッケージのない pipのみのホストでデスクトップパッケージがない場合など）。このメッセージはプラットフォームに応じて異なります。Windowsではデスクトップアプリを開くか、任意の`chloros-cli`コマンドを実行してください。 Linuxでは、`chloros-cli`コマンドを実行するか（GUIは存在しません）、`.deb`をインストールしてください。リモートバックエンドの場合は、`backend_url=` （および `auto_start_backend=False`）を渡してください。
- **`CAMERA_AVAILABLE == False`** のインポート時 → `lattice_sdk` の読み込みに失敗しました：Arena SDK ランタイム、または OpenCV (`cv2`) は、このPython環境ではインポートできません。バックエンド駆動型のサーフェス（`connect_*`、バックエンドを介して投影されるもの、`ChlorosLocal`）は引き続き動作します。
- **センサー全領域を接続しているがフレームレートが低い**→ 設計上の仕様：自動モードではセンサー全領域を維持し、フレームレートは回線の許容範囲まで低下します。`binning=2` / `4`、あるいはより小さい`width` / `height`を指定してフレームレートを向上させるか、ジャンボフレームを有効にするか、リンクをアップグレードしてください。 ビニングやROIでは、集約的なオーバーサブスクリプション（`oversubscribed: true`、fpsフィールド0）は**解決できません**： 回線容量に対してカメラ数が多すぎる問題は、ビニングやROIでは解決できません。カメラ数を減らすか、ジャンボフレームを有効にするか、より高速なNICに切り替えてください（[オーバーサブスクリプション](#over-subscription-the-per-cam-floor)を参照）。
- **`analyze_array_network()` は、NIC の受信リングが極めて小さい（約 0.26 MB）と報告する／「FRAMES WILL DROP」** と表示されるゲートに接続 → ホスト NIC の受信リングがデフォルト値になっている（NIC ドライバの更新後に 32 にリセットされることが多い）。Realtek USB 10GbE アダプタでは、`ReceiveBufferLen=256` および `PendingReceives=64`（権限昇格）を設定し、リングを再読み込みするためにバックエンドを再起動してください。詳細な手順： [CLIリファレンス → ホストNICの設定と調整](cli-reference.md#host-nic-setup--tuning-lattice-arrays)。
- **再起動/シャットダウン時にホストがハングし、その後WMIで`Invalid class`エラーが発生／NICが有効化されない** → 古い USB 10GbE ドライバーが原因で `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`) が発生しています。アダプターのドライバーを最新バージョン (2026 以上) に更新し、受信リングの設定を再適用してください。 [CLIリファレンス → ホストNICの設定と調整](cli-reference.md#host-nic-setup--tuning-lattice-arrays)を参照してください。
- **HTTP 503「No fresh synced frame group」によりアレイのキャプチャに失敗** → 待機時間内にアレイから新しいフレームグループが公開されませんでした。`arr.status()`: `stalled: True` （または `streaming: False`）と照らし合わせてください。`actual_fps` が 0 の場合、グラブループがアレイの公開を停止していることを意味します — アレイの接続を切断して再接続してください。`stalled`がFalseで、かつ`actual_fps`が0より大きい場合は、再試行してください。 `connect_array()`の直後、キャプチャはすでに最初のグループを最大20秒間待機しています。
- **`status()`は`link_speed_mbps: 4294`を示しています** → Windows がアダプタの速度を読み取れませんでした。これは USB イーサネットアダプタではよくある現象であり、不具合ではありません。[予想フレームレート](#expected-frame-rates) を参照してください。
- **`update_firmware()` が `device never connected back on TCP <port>` で失敗** → ファイアウォールが、DAQ-Eからこのコンピュータへの接続をブロックしています。Windowsインストーラーはルールを追加します（[DAQ-E デバイス設定](#daq-e-device-settings)を参照）。他のファイアウォールを使用している場合は、センサーのサブネットからバックエンドへの着信TCP通信を許可してください。
- **反射率の取得が拒否されました** → 絶対スケールの反射率を取得するには、稼働中のDAQをカメラ（またはアレイ）にバインドする必要があります。GUI経由でバインドするか、ペアリングされたセンサーを必要としない `processing="radiance"` (W/m²/sr/nm) を使用してください。
- **`smart=True`のキャプチャに予想以上に時間がかかる** → AEの収束はシーンの動的状況に依存します。`ArrayHandle.capture_smart()`を使用する場合は、`exposure_tolerance_pct`の幅を広げ、 `stability_window_s`を短縮するか、`settle_timeout_s`を下げると、トリガーがより速く、安定しすぎない状態になります。

---

## 関連項目

- [CLI リファレンス](cli-reference.md) — すべての CLI サブコマンドは、SDK 呼び出しに対応しています。
- [DAQ 光センサー](https://mapir.gitbook.io/chloros/daq-light-sensors)（Chlorosマニュアル内） — センサー、その上限値と校正済み測定範囲、記録、DAQ-E ネットワーク。
- [LATTICEカメラ](https://mapir.gitbook.io/chloros/lattice-cameras/lattice)（Chlorosマニュアル内） — 接続、カメラ設定、キャプチャモード、アレイ。
- オンラインドキュメント: `https://mapir.gitbook.io/chloros/api-python-sdk`
