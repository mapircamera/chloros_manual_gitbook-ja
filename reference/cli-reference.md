# Chloros CLI リファレンス

**バージョン:**

1.3.0**生成日:**2026-07-29 19:19 ·**改訂日:** 2026-11-06**対象読者:** LLMによる利用に最適化されていますが、人間が読める形式です。**範囲:** `chloros-cli`のユーザー向けサブコマンドすべて。オプションおよびコピー＆ペースト可能な例を含みます。

このドキュメントは、MAPIR Chloros に同梱されているコマンドラインツール「`chloros-cli`」の完全なリファレンスです。LLM（または人間）がソースコードを確認することなく、以下のリストからサポートされているあらゆるワークフローを構築できるよう、意図的に網羅的に作成されています。

要点のみを確認したい場合は、以下の項目へ進んでください：
- [5分でできるクイックスタート](#five-minute-quickstart)
- [LATTICEカメラの初回接続ワークフロー](#lattice-camera-first-connect-workflow)
- [DAQセンサーの初回接続ワークフロー](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [キャプチャモード、レコーダー、オフライン再処理](#capture-modes-recorders--offline-reprocess)

> **AIアシスタントからChlorosを操作しますか？** Claude Desktop、Claude Code、Cursor、およびその他のMCPクライアントは、`chloros-mcp`を介して、処理、LATTICEカメラおよびアレイ、DAQ光センサーを実行できます。 Python SDK とともにインストールされる MCP サーバー（および Linux .deb パッケージ、`/usr/bin`）である `claude mcp add chloros -- chloros-mcp` を通じて、処理、LATTICE カメラおよびアレイ、DAQ 光センサーを実行できます。セットアップ、オプション、およびツールについては、SDK リファレンスの [MCP サーバー (AI アシスタント)](sdk-reference.md#mcp-server-ai-assistants)に記載されています。シェルを備えたアシスタントを使用すれば、このページに記載されているすべてのコマンドを直接実行することも可能です。

> **

1.3.0での変更点**（既存の1.2.0スクリプトやワークフローが検知するもの）。
>
> **

1.2.0スクリプトで対応が必要となる可能性のある変更点：**
>
> - **`--preset`が適用されます。** `lattice capture`、`continuous`、`calibrate`、`multi-capture`、`multi-stream`、 `align-calibrate`、`align-apply`、および `align-stream` は、バージョン 1.2 で削除されました。0で、カメラ自身の設定で動作していました。現在は、プリセットのフォーマット、露出、ゲイン、フレームレートを設定します。`--preset high_speed`は、5 ms、6 dB、 60 fpsで8ビット撮影します。`lattice cam-connect --preset`もプリセットのフォーマットを維持します（バージョン1.2.0では、プリセット適用後にカメラが12ビットに設定されていました）。[共通オプション](#common-options-shared-by-most-subcommands)を参照してください。
> - **M3M（モノクロ）カメラにおけるベイヤー画素フォーマットとは、その深度を指します。** プリセット名はベイヤーフォーマットを指定していますが、モノクロセンサーではこのフォーマットを撮影できません。バージョン1.2.0では、カメラがこのフォーマットを拒否し、元のフォーマットを維持したため、 そのため、`--preset high_speed`はそこで8ビットにはなりませんでした。現在は同じ深度のモノクロフォーマットになっています：`high_speed`および`--pixel-format BayerRG8`は、`Mono8`となり、`high_quality`、 `triggered`、`BayerRG12`は、それぞれ`Mono12`を生成します。M3CとM3Mが混在する配列では、各カメラは自身のファミリー内で共有フォーマットを採用し、`network-analysis` / `analyze-array --pixel-format`は、各カメラをその深度で投影します。逆の場合も同様です。モノクロフォーマットを要求されたM3Cは、その深度のベイヤーフォーマットを採用するため、マスターがM3Mであるアレイは、M3Cカメラをモノクロで再オープンしなくなります。詳細は [Mono (M3M) シングルバンドカメラ](#mono-m3m-single-band-cameras) を参照してください。
> - **`lattice array-capture`のエクスポートデータは、アプリと同様にデフォルトでアラインメント処理されます。** 各メンバーの`debayered`、 `display`、`radiance`、`reflectance`、およびインデックスファイルは、マスターカメラのフレームに合わせて歪み補正され、メンバー間のオーバーラップ部分 （カメラのオフセット分だけセンサーより小さい、`XMP-mapir:AlignmentJSON` `"applied": true`）にトリミングされるため、CLIでのキャプチャは、同じシーンをChlorosアプリでキャプチャしたものと同じファイルになります。`raw`はネイティブのままです。以前は、`--aligned`を使用しないキャプチャでは、すべてのファイルがネイティブサイズで、ワープ処理されずに保存されていました。`--no-aligned`では、この動作が維持されています。同じデフォルト設定が、`--smart`、 `--continuous`、`--interval`、およびSDKと`chloros-mcp`にも適用されます。[`array-capture`モード](#array-capture-modes)を参照してください。
> - **`lattice array-capture --compression` のデフォルトは `none` です。** ファイルは、最も高速な持続レートを実現するために非圧縮で書き込まれます（フル解像度のフレームあたり約 6.3 MB）。バージョン 1.2.0 では `deflate`（約 4.1 MB）が書き込まれました。 ファイルサイズを小さくするには、`--compression deflate`を指定してください。どちらもロスレスです。
> - **PNGおよびJPEGは、`display` / `preview`およびインデックス付き製品のみを受け付けます。** 8ビットファイルで表現できないレベルを持つ`--format png`または`jpg`のキャプチャは、キャプチャが開始される前に拒否されます（HTTP 400、`1`で終了）、 また、`--processing all`と`--format png`/`jpg`を組み合わせたデフォルト設定も同様に拒否されます。[キャプチャのエクスポートレベル](#capture-export-levels-the-all-default)を参照してくださいを参照してください。
> - **`lattice dls` には `--mac` が必要です。**デフォルトの MAC アドレスはもはや存在しません（以前は 1 ユニットのアドレスでした）。`daq pool-discover --only ble` はセンサーの MAC アドレスを検出します。**`lattice dls --camera SERIAL`** は新規追加されました。
> - **その他のエラーは `1` で終了します。** 実行時のエクスポートが検証できなかった場合や、イメージが失われた場合（他の製品は書き込まれているにもかかわらず）、`process` が発生します。書き込みに失敗したファイルを含むキャプチャコマンド （他のファイルは正常に保存される）； `lattice continuous` および `multi-stream` はドライブが満杯の場合に発生します。`array-burst` / `array-record` は、レコーダーがエラーを報告した際に発生します。 要求された製品がスキップされたか、要求通りに構築されなかった場合：`array-build-video`；キャプチャが途中で失敗した場合：`project burst` / `stream`； `project sensor log`：録画に読み取りデータが含まれていない場合；`lattice exposure --set`、`gain --set`、および`configure --set`：カメラが値を受け入れない場合。 `lattice network-analysis --json`は、サマリーと同様に終了します （`1` は `needs_force_slip` の場合）。その逆の場合：プールから除外されたセンサーの `daq pool-disconnect` は、`0` を終了させますXとして終了します。[終了コード](#exit-codes)を参照してください。
> - **処理が遅い`lattice array-connect`は、配列が接続されている限り失敗することはありません。** 最大 600 秒間待機し（バージョン 1.2.0 では 180 秒、`project connect` では 90 秒）、その間バックエンドのステップを出力します。その時点で接続が完了していない場合、バックエンドによって接続は解除され — すべてのカメラは検出時の状態に戻されます — したがって、`Error:`（終了コード `1`）が表示された場合は、アレイが接続されていないことを意味します。 バージョン 1.2.0 では、カメラの電源投入直後の接続処理が待機時間を超えて続き、コマンドが失敗した後にアレイが接続されてしまうことがありました。現在は、Ctrl+C キーを押すことで接続をキャンセルできます。[永続的な接続](#persistent-connections-smart-prep-gui-equivalent-flow) を参照してください。
> - **`lattice array-connect` はセンサー全体を維持します。** `--width` / `--height` / `--binning` を指定しない場合、すべてのカメラはフルセンサーに設定され、フレームレートはこのコンピュータとネットワークが提供できる範囲まで低下します。接続出力には、より高速なオプション （ビニングまたはより小さなROI）と、その予測レートがリストされます。1.2.0では、レートを維持するためにフレームを縮小またはビニングしていました。[`array-connect` オプション](#array-connect-options)を参照してください。
> - **ネットワーク速度が遅くても、アレイが拒否されることはなくなりました。** 各カメラは回線の帯域幅に応じてペースが調整され、それに伴ってフレームレートも低下します。カメラのペースをそれほど遅くできない場合、アレイはシリアルエミット (`slip-emit-and-capture`) で動作します。すべてのフレームが到着しますが、カメラは同時に露光されません。`--force-tier sim-capture-sim-emit` は同時露光を必須とします。[アレイの fps およびバーストモデル](#array-fps--burst-model)を参照してください。
> - **`lattice network-analysis` / `analyze-array`は、connectが適用される内容を反映します**: `--pixel-format`がない場合、connectが設定するフォーマット（Mono12 / BayerRG12、以前はBayerRG8）および[パックされた12ビット](#packed-12-bit-transport---packed)の決定。 したがって、同じコマンドでも、バージョン 1.2.0 とは異なるフレームレートが指定される場合があります。
> - **相対パスは、コマンドを実行した場所を基準とします。** `-o` およびバックエンドが開くその他のパス （`--input`、`--burst-dir`、`--profile`、`--save`、`--mask`)、レシピステップの `output`、 および `lattice index --live` のデフォルト値である `output/` は、`chloros-cli` を実行するフォルダを基準に解決されます。 バージョン 1.2.0 では、バックエンドは自身のインストールフォルダ内でこれらを解決していましたが、インストール済みのビルドでは「アクセスが拒否されました」というエラーが発生していました。[規約](#conventions)を参照してください。
> - **`lattice index`は、他の`lattice`コマンドと同様に、あらゆる形式でバックエンド上で実行されます**。そのため、まずバックエンドを起動してください。オフラインインデックス (`--input`)はバックエンドによって計算されます。バックエンドはTIFFを読み込み、その横または`-o`に結果を書き込みます。 また、`--list-presets` / `--list-gradients`はバックエンドのテーブルを出力します。Windowsでは、これらが現在動作します： 以前は、WindowsのCLIでは、`lattice index`や`--live`を含むあらゆる形式のコマンドに対して、`Error: lattice_sdk not available`という応答が返されていました。Linuxでは、 オフラインインデックスは、以前はバックエンドもログインも設定せずにCLI内で実行されていましたが、現在は他のすべてのCLIコマンドと同様に、両方が必要となります。[インデックス / 植生計算](#index--vegetation-maths)を参照してください。
> - **成果物の横に新しいファイルが追加されました。** バンドのアンミックス処理でピクセルがクリップされたすべての放射輝度、反射率、およびインデックスプロダクトには、プロダクトのフォルダ横にある `clamp_masks/` フォルダ内にマスクが生成され、キャプチャ出力には `clamp mask:` の行が記録されます。 [アンミックス・クランプ・マスク](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)を参照してください。
> - **カメラの向き。** シングルカメラでのキャプチャでは、`raw`および`debayered`がセンサーが読み取ったままの状態で保存され、カメラの反転／回転情報がXMPに記録されます。`process`は、生成される製品にこの情報を適用します。 新しい `--orient` を渡さない限り、配列の構成ファイルはネイティブ形式のまま保持されます (`array-capture`、`array-burst`、`project capture` / `burst` / `stream`）を指定しない限り、配列のメンバーファイルはネイティブ形式のまま保持されます。[カメラの向き](#what-a-captures-folder-looks-like)を参照してください。
> - **1以外のエクスポートレベル名は拒否されます。** `lattice capture --processing` およびすべての `--levels` リストは、`raw`、`debayered`、`radiance`、`reflectance`、または `display`（あるいは `preview` などのエイリアス；`--processing` および `all`）。カメラが操作される前に、それ以外の名前は拒否されます（HTTP 400 `UNKNOWN_LEVEL`、終了 `1`）； 1.2.0 はそのために `raw` を保存し、終了した `0`。
> - **`daq pool-set-cap` には `--cap-id` が必要です。** それがなければ、コマンドは終了します `2`; 1.2.0 ではキャップ補正が削除され、素のセンサーが残されました。 `--cap-id none`は、この素のセンサーを要求します。
> - **`lattice resolution --binning-mode`は削除されました。** ビニングは常に「平均」となります。このフラグは依然として受け付けられますが、警告を表示するだけで何もしません。 [センサーのチューニング](#sensor-tuning)を参照してください。
> - **LATTICEキャプチャでは、`process --debayer texture-aware`は拒否されます。** LATTICEキャプチャが格納されたフォルダの場合、`process`はインポート後に`1`を終了し、処理を開始する前にその数を表示します。 LATTICEキャプチャには常に標準のデモザイク処理が適用されるため、バージョン1.2.0ではこのオプションを受け付けたものの無視していました。Chlorosアプリでも、LATTICEキャプチャに対しては同様のオプションを提供していません。[`chloros-cli process`](#chloros-cli-process)を参照してください。
> - **カメラは、検出された状態に戻ります — `--keep-settings` の場合を除く。** `lattice array-disconnect`、すべての一時的なコマンド（配列を指定しない `multi-capture`、`multi-stream`、 `multi-test`、`align-calibrate`、`align-apply`、 `align-stream`、`index --live`）および`lattice continuous`は、各カメラを元の状態に戻します： トリガーおよび同期ラインの配線、PTP、そしてセッション中に変更されたすべてのカメラ設定（露出とゲイン、AEターゲット、リミットとダンピング、ホワイトバランス、ブラックレベル、ピクセルフォーマット、ビニングとROI、反転、フレームレート）を復元します。一方、`--keep-settings`は各カメラの各カメラの設定をカメラ自身のメモリに保存するため、電源投入時にそれらの設定で起動します。`--clear-settings`は、そのメモリを工場出荷時の設定にリセットします。いずれの場合も、カメラはフリーラン状態のままになります。プロジェクトは、カメラを接続する際に、自身の保存済み設定に基づいてカメラを設定します。 [カメラのメモリ](#camera-memory----keep-settings----clear-settings)を参照してください。
> - **どのモードでも、プロジェクトへの保存方法は同じです。** プロジェクトに接続されたカメラ、アレイ、または光センサーに対する変更（アプリ、CLI、SDK、MCPサーバーからの変更を含む）は、すべてそのプロジェクトの`cameras.json` / `sensors.json`に保存され、どのモードでプロジェクトを開いても、そこからデバイスが設定されます。以前は、アプリのみがプロジェクトを保存しており、次の保存時に他のすべてのデータが上書きされていました。[1つのプロジェクトレコード、すべてのモード](#one-project-record-every-mode)を参照してください。
> - **複数のカメラから作成されたファイルには、各カメラの名前が付けられます。** マルチバンドスタック（`align-apply`、`align-stream`、`index --live --save-multiband`）、複数のカメラのバンドから作成されたライブインデックス、およびアレイキャプチャの `composite/` 画像は、それぞれソースとなるカメラのシリアル番号、モデル、およびキャリブレーションパックをリストとして保持しています（`XMP-mapir:SourceCameraSerials`、 `SourceCameraModels`、`SourceCalBundleSha256s`、…）、またEXIF `SerialNumber`にはシリアル番号がカンマ区切りでリストされています。 以前は、カメラ情報が全く含まれていなかったり、最初のカメラのシリアル番号のみが含まれていたりしました。[複数のカメラから作成されたファイル](#files-made-from-several-cameras)を参照してください。
> - **カメラのキャリブレーションタグが含まれているのは、`raw`ファイルのみです。** MicaSense互換のセット（`BlackLevel`、`RadiometricCalibration`、チャンネルごとの`Band{R,G,B}RadiometricCalibration`、 `MicaSenseVignetting*`セット）およびペイロード`CalibrationJSON`は、センサーの生データを記述しているため、キャプチャファイル`raw`にのみ含まれます。 それ以外のすべてのファイル（キャプチャの `debayered`、`display`、`radiance`、 `reflectance`およびインデックスファイル、`lattice continuous`および`multi-stream`フレーム、 およびすべての `process` プロダクトには、カメラの識別情報（`XMP-mapir:CameraSerial`、`CameraModel`、`FilterType`、 `LensModel`、`CalBundleSha256`、…）を保持しており、撮影時にはそのフレーム固有の記録（`XMP-mapir:FrameJSON`：露光、ゲイン、温度、ピクセルフォーマット、撮影時刻）も保持します。 バージョン1.2.0では、RAWデータセットも含まれており、これを適用するツール （Pix4D、Metashape）が、RAW値ではないピクセルに適用していました。`process`は、そのシリアル番号と`CalBundleSha256`に基づいて、そのようなファイルのキャリブレーションを特定します。
>
> **その他の変更点：**
>
> - **アレイの自動フレームレートは、アレイが出力するレートに安定します。** トリガー数の1％以上を失ったレートは、1回ではなく数回の長時間の監視に基づいて判断され、段階的に引き下げられます。また、引き下げられた元のレートは、時折再試行されます。手動で設定したレートは変更されません。
> - **自動露出がより安定しました。** 光量が一定の場合はその状態を維持し、変化後は数秒以内に安定します。また、わずかなハイライトは無視し、強い日差しの下で応答しなくなったカメラを復旧させます。 [自動露出の動作について](#how-auto-exposure-behaves)を参照してください。
> - **`process`は、画像と録画データのみをカウントします。** プロジェクトの `camera_cal/` が格納されているフォルダで再実行しても、それらのキャリブレーションファイルはカウントされなくなります（90枚の画像セットで「96枚の画像が見つかりました」と表示される）。また、すでにそれらを取り込んだプロジェクトでは、それらのファイルが削除されます。
> - **`lattice multi-test`は、カメラの電源投入状態を引き継ぎます。** 実行中にカメラのPTPを有効にし、最初のトリガーが発行される前にクロックがロックされ安定するのを待ちます。各カメラのロックには120秒（電源投入から約2分）かかります （`--ptp-settle SECONDS`が設定）、これはロック直後のクロックが約2分間、最大約200マイクロ秒の誤差を持つためです。その後、各カメラのPTPを元の状態に戻します。以前のテストでロックされたままのカメラについては、再度待機することはありません。 以前は、DAQ-Eが応答しない単一スイッチのネットワークではPTPを無効にしたままにし、各カメラのフリーランニングクロックを、ロックされないPTPクロックとして読み取っていました（`PTP is not locked`、各実行時）。[マルチカメラ（一時的なセッション）](#multi-camera-transient-sessions)を参照してください。
> - **`.daq`の記録には、キャップが選択されたか、あるいは想定されたかが示されます。** DAQ-U、DAQ-M、およびDAQ-Eにはキャップが付属しておらず、Chlorosはキャップが装着されているかどうかを判別できないため、キャップ名が指定されずに接続されたセンサーには、デフォルトで`sunshine_cosine`が割り当てられ、これは*仮定*。すべてのChloros記録（スキーマ1.30）は、現在、`cap_id`の横に`cap_id_source`を保存するようになりました： キャップに名前を付けた場合は `operator`（`none` を含む）、デフォルトの場合は `auto_default` となります。キャップが「仮定」された記録を処理する際、プロジェクトの DAQ キャップ設定を「Auto」のままにするとに設定したまま処理すると、ファイル名を `[daq-dls] WARNING` として出力されます。日差し補正を適用していない DAQ-U のみの場合、読み取り値が約 30 倍高くなります。接続時に **適合させたキャップに名前を付ける** か、プロジェクトの DAQ キャップを設定すれば、この警告は表示されなくなります。
> - **Windows での pip コンソールウィンドウの表示がなくなりました。** Python がインストールされた PC では、バックエンドとその各イメージワーカーが起動時に `pip` を 1 回ずつ実行し、 それぞれがコンソールウィンドウを開いていましたが、現在はそうなりません。
> - **`.daq`によるライブ記録は停電後も維持されます。**記録は、センサーの背後で、コミットごとにジャーナルとディスクの完全同期を行って書き込まれます。 そのため、ディスクの速度が遅くてもファイルの書き込みが遅れるだけで、測定値の記録が遅れることはありません。コミット途中で停止した記録（停電、強制終了）の横に `<name>.daq-journal` が残っている場合があります：**この2つを一緒にコピーしてください。**Chlorosは、ファイルを最初に読み込む際にジャーナルをロールバックするため、記録ファイルを開くと停止前のすべてのデータがコミットされた状態になります（最後の約3秒分が失われます）。バージョン1.2.0では、不適切なタイミングで停電が発生すると、ファイル全体が読み取れなくなる可能性がありました（「データベースのディスクイメージが不正な形式です」というエラー）。
> - **負荷の高いディスク上での記録処理でも、センサーが停止しなくなりました。** `daq pool-record`は読み取りデータをキューに入れ、専用のスレッドで書き込みを行います。書き込みできなかった読み取りデータは、 （`daq pool-list`は、最後のエラーが発生した`N write error(s)`を示しています）センサーがフリーズすることはなくなりました（4台のカメラがTIFFを保存している傍らで、1回の書き込みに8.4秒かかった例があります）。
> - **`lattice multi-capture`は、配列を使用しない場合、トリガーのタイムアウト時に保存済みのフレームを失うことがなくなりました。** タイムアウトしたカメラのトリガー（`SC_ERR_TIMEOUT -1011`、1つの1 GbEスイッチに接続された4台のカメラで確認）は、そのフレームグループで実行を終了します。すでに保存されたグループは`Saved:`として一覧表示され、その後エラーが表示され、 exit `1`と表示されます。バージョン1.2.0では、ファイルがディスク上に存在していたにもかかわらず、リストが表示されないHTTP 500エラーが発生していました。
> - **ソフトウェアトリガー方式のカメラは、トリガーを1回おきにドロップしなくなりました。** ソフトウェアトリガーモードのカメラは、読み出し中に到着したトリガーをキューに格納するようになりました（あるいは、Chlorosが準備完了を待機します）。 これは、ケーブル同期カメラがすでにそうしていたのと同じ動作です。
> - **`daq pool-discover`は、DAQ-E-Sの`DAQ-E-S`** （また、`--json`の`model`も同様）。すべてのイーサネットセンサーに対して`DAQ-E`と表示されていたが、 約2秒以内に身元確認に応答しないセンサーであっても、`DAQ-E`として依然としてリストに表示されています。
> - **LATTICEインデックス製品**（NDVIおよびその他のインデックス）が、すべての処理モードにおいて、`process`から改善されました。
> - **OCNカメラ：日照下における放射輝度、反射率、およびインデックス製品の精度が向上しました。** バンド分離後にフリート補正が適用されます。RGNカメラは変更なく、OCNの値はバージョン1.2.0とは異なります。これは、以前のバージョンで撮影されたOCNデータについても、撮影時に使用されたキャリブレーションが保存されているコンピュータで処理される場合に適用されます （Chlorosは、カメラが一度接続されるとそのキャリブレーション情報を保持します）。
> - **`lattice continuous`は生データを記録します。** `tiff`の実行により、デモザイク処理済みの3プレーンのフレームを書き出しましたが、`process`はこれを起点として処理を開始できません。そのフレームは、現在、アレイキャプチャが書き出す`raw`ファイルとなっています。これは、センサー固有の値を持つ1つのモザイクプレーンで、非圧縮であり、キャリブレーションパケットが含まれています。 これらのファイルから3つのプレーンを読み込むスクリプトは、1つしか読み込みません。
>
> - **GUIと同様に、すべてのキャプチャでは、使用したDAQの読み取り値が保存されます。** `lattice capture`、 `lattice continuous`（1フレームにつき1つの`.daq`）、`array-capture`、および`multi-capture`は、接続された配列（およびSDK）を使用すると、`.daq` およびその `.match.json` を書き込みます。これには、DAQ がバインドまたはプールされた場合、および生データのみのキャプチャが含まれます。バージョン 1.2 では、0 では、生データまたはデベイヤー処理済みのキャプチャが保存された場合、`--force-daq` でのみ 1 つが保存されます。`--no-daq` はこれを無効にし、`--daq-tolerance-ms MS` はそのキャプチャのマッチング許容誤差を設定し、新しい `daq pool-bind` は、CLI からカメラまたはアレイを DAQ にバインドします。`.daq` によるキャプチャの保存は、`process` において、指定されたカメラとのみ照合されます。 キャプチャは、露光前後の読み取り値をブレンドできるように、DAQの次の読み取りを待機するようになりました。 「キャプチャが保存する DAQ 読み取り値」(#the-daq-reading-a-capture-saves) および「露光から離れた場所での DAQ 読み取り」(#a-daq-reading-far-from-the-exposure) を参照してください。
> - **`process --indices NAME`** は、各カメラのバンドごとにプロジェクトにインデックスを追加します（新しいプロジェクトが含まれます）。バージョン 1.2.0 では、`Unknown index name(s) ignored` により、プロジェクトにまだリストされていない名前は無視されていました。
> - **ライブアレイの横にある**`process`**：**カメラやDAQを保持しているバックエンドを再起動しなくなり、終了しつつある実行に対しては最大15秒待機するようになりました。**Ctrl+C** では、CLIだけでなく、バックエンドの実行も停止します。 [`process` 注意事項](#notes)を参照してください。
> - **Windowsのパス制限（260文字）を超える深さのプロジェクトフォルダ**については、何も書き込まない実行を行う代わりに、制限内に収まる最長のフォルダ名を使用して、最初から拒否されるようになりました。 Windowsインストーラーには、「Windowsの長いパスを有効にする」オプションと、バックエンド用のMicrosoft Defender除外設定が追加され、どちらもデフォルトでチェックされています。[Windows (.exe)](#windows-exe)を参照してください。
> - **`lattice array-capture --continuous`** は、バックエンド独自の連続キャプチャを実行します。2 秒ごとに進行状況バーが表示され、ファイルごとの一覧表示の代わりに要約が表示され、書き込みできなかったファイルがある場合やドライブが満杯になった場合は、終了コードが 0 以外になります。 [`array-capture` モード](#array-capture-modes)を参照してください。
> - **`array-connect`の直後に実行される最初の`array-capture`**は、HTTP 503「Array still warming up」エラーで失敗する代わりに、アレイの最初のフレームグループが到着するまで最大20秒待機します。
> - **ROI 内のアレイの露光時間：** 小さな ROI であっても、アレイ構成要素の露光時間がセンサーのフリーラン期間（768 × 304 の場合 4.9 ms）に制限されることはなくなりました。[アレイ構成要素の露光時間](#exposure-on-an-array-member)を参照してください。
> - **自動露光によって、アレイの自動フレームレートが低下しなくなりました。** ケーブル同期されたアレイは、ネットワークおよびカメラの読み出しが許容するレートで動作します。自動露光は、そのレートによって与えられる期間を埋めた後、ゲインを上げます。 その期間（`lattice exposure --set`、`configure --set`）を超えるマニュアル露光の場合、露光が拒否されるのではなく、その露光時間に合わせるためにトリガー速度が低下し、メンバーが自動モードに戻ると元のレートに戻ります。詳細は [アレイメンバーでの露出](#exposure-on-an-array-member)を参照してください。
> - **アレイによる不完全なフレームグループの発生が減少**し、503エラー（「新しい同期済みフレームグループなし」）の発生も減少します。各スレーブカメラのトリガー入力は、グリッチ除去処理が施されるようになりました。
> - **パックされた12ビット転送**（`--packed`、デフォルトは`auto`）により、フルセンサーアレイのフレームレートが約3分の1向上します。 `array-connect --packet-size`が適用されるようになりました（1.2.0では受け入れられていましたが、無視されていました）。 [パケット化された12ビットトランスポート](#packed-12-bit-transport---packed)を参照してください。
> - **カメラの電源再投入後に自動的に回復するアレイ。** スイッチの電源再投入後、`array-connect`のアレイは- カメラが応答するとカメラを再オープンし、PTPを再有効化してストリーミングを再開します。1.2.0では完全に停止してしまい、新しい`array-connect`が必要でした。`array-status`は、カメラがPTPオフで読み取りを行っている間、`ptp_enabled: False`（および `ptp_off:`）を報告しますが、以前はアレイが接続されていた設定を報告していました。詳細は [永続的な接続](#persistent-connections-smart-prep-gui-equivalent-flow) を参照してください。
> - **`lattice array-record`は、GUIが開いていない状態で記録を行います。** GUIの統合ビューが閉じられている場合、バックエンドはビュー自体（アレイのNDVI、または新しい`--expr`で渡されたインデックス。フレームを書き込まない記録は、現在失敗します（`no-frames: …`、`1`で終了）。 以前は、GUIの統合ビューが開いている間のみ記録が行われ、 、何も書き込まない録画でも `0` を終了していました。[`array-record`](#array-record--combined-index-videogif-monitoring-grade) を参照してください。
> - **Chlorosがすでに保持しているカメラは、二度と開かれません。** それを開こうとするコマンドは、代わりにHTTP 409 `CAMERA_HELD`を返し、そのカメラを保持しているものと、それを解放するコマンドの名前を指定します。 これには、現在別のコマンドによって使用中のカメラも含まれます（カメラを開いている `multi-test` や `multi-capture`、あるいは `lattice capture` など）。最初のコマンドが完了するまで、2番目のコマンドは拒否されます。 再度開こうとすると、保持しているカメラのストリームが停止したり、バックエンドがクラッシュしたりする可能性があります。[Persistent Connections](#persistent-connections-smart-prep-gui-equivalent-flow)を参照してください。
> - **一時的なマルチカメラコマンドは、すべてのカメラを配信するか、失敗します。** 配列が接続されていない `lattice multi-capture`、`multi-stream`、`multi-test`、`align-calibrate`、 `align-apply`、`align-stream`、および `index --live` は、フレームを送信しなかったカメラ名を指定して `1` を終了します。 ストリームは、カメラが欠落しているフレームグループをスキップし、3回連続して欠落すると停止する。`multi-test`はトリガーごとに1行を出力し、`(no frame)`はカメラがトリガーを逃した箇所を、`align-stream`もジョブエラー時に`1`がジョブエラーで終了します。その後、各実行では、すべてのカメラを検出時の状態（トリガーおよび同期ラインの配線、PTP (`PtpEnable`)、および実行中に変更されたすべてのカメラ設定（露出とゲイン、AEターゲット、リミットとダンピング、ホワイトバランス、ブラックレベル、ピクセルフォーマット、ビニングとROI、反転、フレームレート）を、`--keep-settings` / `--clear-settings` を除き（[カメラメモリ](#camera-memory----keep-settings----clear-settings)を参照）。 以前は、これらのコマンドは取得されたフレームをすべて保持したまま`0`を終了していました（`align-stream`は`Note:`としてジョブエラーを出力していました）。 すべてのカメラを `TriggerMode` `On` に残していました。また、フレームを保持する前のシーン（最大数秒間）に基づいて各カメラの露出を設定し、接続されたアレイが各カメラを以下の設定で開始するようにしていました：40 マルチスペクトル、 60RGB（`--brightness N`が設定）で各カメラを起動し、実行中はそれを維持する。 Chlorosが保持していないカメラの単一の`lattice capture`も同様です。以前は、各カメラの最初のフレームは、光量にかかわらず、カメラ自身の露出制限（約100 ms）で撮影され、しばしばクリップされていました。 [マルチカメラ（一時セッション）](#multi-camera-transient-sessions)を参照。
> - **一時的なマルチカメラコマンドは、`array-connect`と同様にネットワークを共有し、`--serials`のみを開きます。** 各カメラはホストリンクの割り当て分に応じてペースが調整されるため、1 Gbpsのリンク上で4台のカメラが動作します。以前は、4台で合計125 MB/s（1 GbEの全帯域）を要求していたため、すべてのフレームでパケットが失われていました。 ネットワークが処理できないアレイは、どのカメラもストリーミングを開始する前に拒否されます。`--serials`は現在、カメラを選択し、最初にトリガーを発動させたカメラを優先します。以前は、`--serials`の指定にかかわらず、すべてのコマンドがネットワーク上のすべてのカメラを開いていました。 GVSPパケット損失による障害は、M8同期ケーブルを原因とするのではなく、その旨が明示されるようになりました。[マルチカメラ（一時的なセッション）](#multi-camera-transient-sessions)を参照してください。
> - **`lattice array-disconnect`は、検出された状態のままカメラをフリーランニング状態にします。** `TriggerMode`、`Off`、および接続時に検出されたその他のすべての設定 — トリガーおよび同期ラインの設定（`TriggerSource`、 `TriggerActivation`、`TriggerOverlap`、`TriggerDelay`、`LineMode` / `LineSource`）、PTP、およびセッション中に変更されたカメラ設定——つまり、元メンバーに対して単一の元メンバーに対する1回のカメラコマンドでフレームを一度に取得でき、マスターはもはや `TriggerSource` `Software` に残されることはありません。`--keep-settings` は、セッションの設定を各カメラのメモリに保存しますし、`--clear-settings`はそのメモリをリセットします（[カメラのメモリ](#camera-memory----keep-settings----clear-settings)を参照）。 また、`lattice continuous`は、武装状態にあるカメラをそのラン中はフリーランモードに設定し（ラン終了後に元の状態に戻します）、フレームがないランは`1`で終了します。
> - **`lattice configure --get` / `lattice exposure`** は、`note:` を出力し、そこにはカメラの実際の露出値またはゲインを`note:`として出力します。拒否された`lattice exposure --set` / `configure --set`は、カメラが許容する範囲を示し、自動露出を無効にしたままにするのではなく、自動モードに戻します。詳細は [センサー調整](#sensor-tuning)を参照してください。
> - **新しいカメラ制御コマンド：**`lattice highlights`（RGB表示のレンダリングにおいて、明るい白い表面の輝度を下げる）および `lattice exposure --brightness-auto`。**`lattice multi-detect`**は、1回のプローブスキャンで有線カメラを検出し、マスターをデフォルトでRGBカメラに設定します。
> - **新しいDAQコマンド：** `daq pool-bind`; `pool-enable-ptp`；およびDAQ-E固有の設定用として`pool-info`、`pool-set-network`、`pool-set-auto-stream`、 `pool-reboot`、および `pool-update-firmware`。詳細は [DAQ-E デバイス設定](#daq-e-device-settings) を参照してください。
> - **データ送信を停止した DAQ-E は、バックエンドによって再接続されます。** プールされた DAQ-E のストリームが静止状態になった場合（電源供給用スイッチの電源再投入、 カメラリンクの切断など）によりストリームが途絶えた場合、バックエンドは以前の設定で再接続を行い、接続が途絶えている間は間隔を徐々に広げながら再試行を行います。そのデバイスでの録画は同じ `.daq` で継続され、ファイル内に途切れが記録され、 また、`pool-record --stop`には各中断箇所が一覧表示されます。バージョン1.2.0では、センサーは`pool-disconnect` + `pool-connect`になるまで無通信状態が続き、これにより録画も終了していました。 [データ送信を停止する DAQ-E](#a-daq-e-that-stops-delivering)を参照してください。
> - **`time-sync status`** は、PTP グランドマスターのパケットがネットワークに到達しているかどうかを報告します（`health`）。到達していない場合は、stderr に `WARNING:` が出力されます。 [`time-sync`](#chloros-cli-time-sync)を参照してください。
> - **Linux .deb:** カメラストリームに必要なカーネルのソケット受信バッファを拡張し（カーネルのデフォルト設定では、フレームやフレームレートが失われる）、ユーザーとして実行されるバックエンドサービスを追加し、`lattice network` がバッファをチェックします。 CLIは、サービスとして実行されているバックエンドを停止することは決してなく、アップグレード時には有効なバックエンドサービスが再起動されます。詳細は、[バックエンドをサービスとして実行する](#running-the-backend-as-a-service-headless) および [Linuxホスト — ソケット受信バッファ](#linux-hosts--socket-receive-buffers)を参照してください。
> - **`lattice array-capture`は、アラインメントができなかった場合に次のように表示します：**キャプチャがアラインメントされていない状態で保存された各理由（アラインメントプロファイルがない、メンバーの解像度が異なる、配線同期されていない配列、 プロファイルにマッピングできないフレームなど）は、`Alignment:` 行として出力されます。**`chloros-cli daq help`**は`daq --help`（`2`で終了）であり、Linuxのヘルプには、そのビルドに含まれる直接ハードウェア操作用の`daq`コマンドが がリストされています。
> - **Jetson:** パッケージは `chloros_1.3.0_arm64_jp6.deb`（JetPack 6 用）であり、その GPU 処理は JetPack の CUDA PyTorch を使用して Orin 上で実行されます。詳細は [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6)を参照してください。

---

## 規約

- すべてのコマンドには「`chloros-cli`」という接頭辞が付きます。Windowsではバイナリ名は「`chloros-cli.exe`」となり、Linux/Jetsonでは`chloros-cli`となります。
- オプションの引数は`--flag`のように表記されます。必須の位置指定引数は括弧なしで表記されます。
- デフォルト値が指定されている場合、フラグを省略するとその値が使用されます。
- CLIは、Chlorosバックエンド（デスクトップアプリも使用する`127.0.0.1:5000`上のサーバー）を経由する簡易なHTTPクライアントです。**これを起動するコマンド：**
  - 主要コマンド — `process`、`login`、`logout`、`status`、 `export-status`、`time-sync`、および `selftest` — これらは実行中のバックエンドを使用するか、バックエンドが実行されていない場合は起動し、処理が完了すると停止します。
  - **`lattice`、`project`、および `daq` コマンドは、バックエンドを起動することはありません。** これらは、すでに実行中のバックエンドを必要とします。Windowsデスクトップアプリのもの、Linuxの [バックエンドサービス](#running-the-backend-as-a-service-headless)、または自分で起動したもの（[バックエンドの起動](#start-the-backend)を参照）。バックエンドがない場合、`Error: Chloros backend not reachable at http://127.0.0.1:5000`というメッセージと、それを起動するコマンドが表示され、`1`で終了します。
- `CHLOROS_BACKEND_URL=<url>` は、**`lattice`**、**`project`**、および**`daq pool-*`** コマンドファミリーを別のバックエンドに割り当てています（`lattice network-analysis`は代わりに`--backend-url`を割り当てます）。コアコマンドは意図的に`http://127.0.0.1:<port>`を固定し、これを無視しています （IPv4リテラルを使用することで、Windowsにおける `localhost`→`::1` によるリクエストあたり約2秒のパフォーマンス低下を回避するため）。[環境変数](#environment-variables)を参照してください。
- **相対パスは、`chloros-cli`を実行したフォルダを基準に解決されます。** バックエンドは出力および入力フォルダを開くプロセスであるため、パスを送信する前にCLIはそれを絶対パスに変換します： `-o` / `--output`（そのデフォルト値である `output` を含む）、`--input`、 `--burst-dir`、`--profile`、`--save`、および `--mask`、ならびに [レシピ](#recipe-dsl) ステップの `output`（デフォルトは `output`）、および `lattice index --live` のデフォルトの `output` フォルダです。したがって、`-o captures/` は `<current folder>/captures/` に書き込みを行い、出力されるパスは絶対パスとなります。 `~` は展開されます。リモートバックエンド（別のマシンを指定する `CHLOROS_BACKEND_URL`）に対しては、パスは入力されたまま送信され、そのマシン上で解決されます。
- すべての SDK/CLI 呼び出しには、Chloros+ アカウントへのログインが必要です（各マシンで `chloros-cli login` を 1 回実行してください。結果は `~/.chloros/` にキャッシュされます）。
- 例ではLinuxのパスを使用しています。Windowsでは、`/home/user/...`を`C:/Users/.../...`に置き換えてください。DAQ-U'のシリアルポートは、Windowsでは `COM3` 形式、Linuxでは `/dev/ttyUSB0` / `/dev/ttyACM0` 形式です。
- 概要では、排他的なフラグを `{--a | --b}`として記述し、オプションのフラグは`[...]`として記述します。

---

## トップレベルの概要

```
chloros-cli [global options] COMMAND [command options]
```

### グローバルオプション

| フラグ | 説明 |
| --- | --- |
| `--backend-exe PATH` | 自動検出されたバックエンド実行ファイルを上書きします。 |
| `--port N` | バックエンドのHTTPポート（デフォルト: `5000`）。コアコマンドでのみ使用されます（`process`、`login`、`logout`、`status`、`export-status`、`time-sync`、 `selftest`）のみが使用されます。`lattice`、`project`、および`daq`は、`CHLOROS_BACKEND_URL`（デフォルトの `http://127.0.0.1:5000`）を使用します。 |
| `-v, --verbose` | 詳細出力を有効にします。 |
| `--restart` | バックエンドを強制再起動します（実行中の `backend_server.py` をすべて終了させます）。systemd サービスによって実行されているバックエンド （Linux .deb）によって実行されているバックエンドは停止されません。その場合、コマンドは代わりに `sudo systemctl restart <unit>` を出力し、`1` で終了します。 |
| `--version` | バージョン情報（`Chloros CLI 1.3.0`）を表示します。 |
| `--help` | トップレベルのヘルプを表示します。 |

### コマンド索引

| コマンド | 目的 |
| --- | --- |
| [`process`](#chloros-cli-process) | Survey3またはLATTICEでキャプチャされたフォルダをエンドツーエンドで処理します。 |
| [`login`](#chloros-cli-login) | このマシンをChloros+アカウントで認証します。 |
| [`logout`](#chloros-cli-logout) | キャッシュされた認証情報をクリアします。 |
| [`status`](#chloros-cli-status) | 現在のライセンス／認証ステータスを表示します。 |
| [`export-status`](#chloros-cli-export-status) | `process`の実行中のThread-4エクスポートの進行状況をリアルタイムで表示します。 |
| [`language`](#chloros-cli-language) | CLIの表示言語を設定または一覧表示します（38言語に対応）。 |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | デフォルトのプロジェクトフォルダ（GUIと共有）。 |
| [`update`](#chloros-cli-update) | Chlorosの更新を確認してインストールします（Linux/Jetson）。 |
| [`selftest`](#chloros-cli-selftest) | システム診断およびスモークテスト。 |
| [`time-sync`](#chloros-cli-time-sync) | PTPグランドマスターのステータス確認／制御。 |
| [`lattice`](#chloros-cli-lattice) | LATTICEカメラの制御およびキャプチャ（45以上のサブコマンド）。 |
| [`daq`](#chloros-cli-daq) | DAQスペクトルセンサー制御（DAQ-U / DAQ-M / DAQ-E）。 |
| [`project`](#chloros-cli-project) | 保存済みのChlorosプロジェクト（カメラ＋DAQ）を開いて実行します。 |

---

## インストール

`chloros-cli`は、サポートされているすべてのプラットフォームにおいて、Chlorosデスクトップインストーラー内に同梱されています — 別途CLI版をダウンロードする必要はありません。プラットフォームパッケージをインストールすると、デスクトップアプリおよびそれを駆動するバックエンドバイナリとともに、`chloros-cli`が`PATH`に追加されます。

最新のダウンロード： [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> インストーラーには、すぐに使える CLI シェルを開く便利なランチャースクリプト (`Chloros_CLI.bat` / `Chloros_CLI.ps1`、`Launch_CLI.*`、 `chloros-cli.sh`）が同梱されており、これらはすぐに使えるCLIシェルを起動します。

### Windows (.exe)

1. ダウンロードページからWindowsインストーラーをダウンロードします。
2. `Chloros Installer 1.3.0.exe`を実行し、ウィザードの指示に従います。 デフォルトのインストールパスは `C:\Program Files\Chloros\` です（CLI は `C:\Program Files\Chloros\cli\` に配置され、インストーラーによって PATH に追加されます）。
3. 更新された `PATH` が認識されるよう、新しいターミナル（`cmd.exe`、PowerShell、または Windows Terminal）を開きます。

```powershell
chloros-cli --version
```

インストーラーは自動的に `chloros-cli.exe` をシステムの `PATH` に追加し、LATTICE カメラのドライバーをバンドルします。バックエンドは `C:\Program Files\Chloros\backend\chloros-backend.exe`（インストールフォルダ内の `backend` フォルダ）であり、`PATH` には含まれていません。

**システム設定**。インストーラの「*追加タスクの選択*」ページにある **システム設定**の下には 2 つのオプションがあり、どちらも**デフォルトでチェックされています**：

| オプション（インストーラ上の表記） | 機能 | サイレントインストールオプトアウト |
| --- | --- | --- |
| **Windowsの長いパスを有効にする**（プロジェクトフォルダを260文字を超える階層に配置可能；推奨） | Windowsのシステム全体の設定である`LongPathsEnabled`を1に設定します。Windowsではこの設定はデフォルトでオフになっており、 この設定を有効にすると、パスが 260 文字を超えることはできません。Chloros 製品はプロジェクトフォルダの約 130 文字下まで配置されるため、それより深い階層のプロジェクトフォルダは拒否されます（[トラブルシューティング](#troubleshooting-pointers)を参照）。インストール後に起動したプログラムから有効になります。他のプログラムがこれに依存している可能性があるため、アンインストール後も有効のままにします。 | `/MERGETASKS="!longpaths"` |
| **Microsoft DefenderのスキャンからChlorosバックエンドを除外する**（カメラキャプチャが大幅に高速化されます。推奨） | インストールフォルダ内の`chloros-backend.exe`に対してDefenderの**プロセス**除外を追加します。これにより、バックエンドが書き込むファイルを、書き込み先に関わらず、リアルタイム保護がスキャンしなくなります。 他のプログラムは除外されません。この除外設定はアンインストール時に削除されます。Defender が存在しない場合やポリシーによってロックされている場合、この手順はエラーを出さずに失敗し、インストールは続行されます。 | `/MERGETASKS="!defenderexclusion"` |

**Defender オプションが重要な理由：** キャプチャレートが高い場合、キャプチャファイルのリアルタイムスキャンにはキャプチャレートの約 30 % の処理負荷がかかります（9 台のカメラによる連続キャプチャで測定： 77.7 キャプチャ/秒が除外され、52.7 がスキャンされました）。`lattice array-capture --continuous`は、キャプチャ処理中にDefenderが1つ以上のCPUコアを使用していると検出した場合、注意メッセージを表示します。その場合は、このオプションにチェックを入れてインストーラーを再実行してください。

サイレントインストール（`/SILENT` または `/VERYSILENT`）では、オプトアウト（例：`"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`）しない限り、両方が適用されます。

**Windows ファイアウォール。** インストーラーは、すべてのネットワークプロファイルに対して、常に `chloros-backend.exe` 用の 2 つの受信ルールを追加し、アンインストーラーはそれらを削除します。`Chloros Backend (UDP-In)` は、カメラストリーム、PTP、 mDNS、およびDAQ-Eデータストリーム用の`Chloros Backend (UDP-In)`、ならびにDAQ-Eファームウェア更新用の`Chloros Backend (TCP-In)`（ローカルサブネットに限定され、センサーがこのコンピュータに再接続して画像を取得する場合）です（[DAQ-Eデバイス設定](#daq-e-device-settings)を参照）。バックエンドのHTTPAPIは、引き続き`127.0.0.1`にバインドされたままとなります。

### Linux amd64 (.deb)

Ubuntu 22.04 LTS 以降、または Debian ベースの x86_64 ワークステーション向け。

> **Ubuntu 20.04 はサポートされていません。** パッケージの依存関係リストは、
> バックエンドが実際にリンクしているものに基づいており、そこには `libc6 (>= 2.35)` が含まれます。
> focal には glibc 2.31 が同梱されています。 `apt` は、実行時に失敗するのを許すのではなく、
> インストールを拒否します。 このリストには `libgl1` も記載されています。これは同梱の OpenCV が読み込むもので、最小構成の
> サーバーイメージには含まれていない可能性があります。`apt install` がこれを提供します。

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

この .deb パッケージは以下をインストールします：
- `chloros-cli` から `/usr/bin/chloros-cli` へ、および `chloros-backend`（バックエンドを起動）を `/usr/bin/chloros-backend` に更新
- コンパイル済みのバックエンドとCLIを、`/usr/lib/chloros/backend/` および `/usr/lib/chloros/cli/` に展開
- LATTICEカメラ用ドライバ
- ノイズ除去モデル、キャリブレーションバンドル、および更新チャネルの設定
- システム用PythonSDK：`python3`（`import chloros_sdk`）。`apt install`には、その依存関係である`python3-numpy`および`python3-requests`も含まれています（推奨ですが必須ではありません。上記のプログラムはスタンドアロン型であるため、 `dpkg -i`はこれらをスキップし、インストール出力で不足しているものが明示されます）。virtualenv内では：`pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`。これらは、カメラの ストリーム用のカーネルソケット受信バッファを32 MBに増やし、これを即座に適用します（すでにこれより高い値が設定されている場合は変更されません）。パッケージを削除すると、このファイルも削除されます。[Linuxホスト — ソケット受信バッファ](#linux-hosts--socket-receive-buffers)を参照してください。
- バックエンド用の 2 つのオプションの systemd サービス。いずれも有効化されていません（次のセクション）。

#### バックエンドをサービスとして実行する（ヘッドレス）

サービスがない場合、バックエンド（およびそれに伴う PTP グランドマスター）は、何かによって起動されている間のみ実行されます。 ターミナルでの ``chloros-backend``、SDKスクリプト、またはコア ``chloros-cli`` コマンドの実行期間中のみです。ヘッドレスマシン上でバックエンドを常時稼働させるには、**自分のユーザーとして**サービスを有効にしてください：

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service`は、そのユーザーとしてバックエンドを実行します。バックエンドが書き込むキャプチャデータはそのユーザーのものであり、詳細なログはそのユーザーの`~/.cache/chloros/logs`に記録されます。 また、そのユーザーの `chloros-cli login` (`~/.chloros`) を使用するため、まずそのユーザーとして一度ログインしてください。PTP には root 権限は必要ありません。インストールされたすべてのバックエンドと同様に、`127.0.0.1:5000` でのみリスニングを行います。

`chloros-backend.service`は、バックエンドを**root**として実行します。 これを有効にしたマシンが稼働し続けるよう、依然としてインストールされていますが、rootバックエンドが書き込むすべてのデータはrootに属するため、ユーザーは自身のキャプチャを削除することはできません。自分のユーザーに切り替えるには：

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

root サービスの詳細ログは `/var/log/chloros` にあり、`adm` グループのメンバーが読み取ることができます（サービスは `CHLOROS_LOG_DIR` を設定します。 [環境変数](#environment-variables)を参照)）。パッケージをインストールすると、ユーザー名が埋め込まれた`enable`コマンドが表示され、rootサービスが有効になっている場合は切り替えコマンドも表示されます。

**バックエンドユニットを1つ有効にする。** 一度に実行されるバックエンドは1つだけです。2つ目の有効化されたユニット（`chloros-backend@<user>.service`のほかに`chloros-backend.service`、または2人のユーザーの`@`ユニットなど）は は起動を拒否し、終了ステータス98の状態で`failed`のままとなり、systemdはこれを再起動しません。 その `systemctl status` が理由を示しています：`Another instance is already running. Exiting.`、次にバックエンドを保持しているもの（ポート 5000、またはロックファイルとその所有ユーザー）、 さらに`systemd will not restart <unit> (exit status 98)`。再起動後、どちらのユニットが実行されるかは場合によります。切り替えるには、上記の手順に従って使用しない方を無効にしてください。**CLIは、サービスのバックエンドを停止させることはありません。** `chloros-cli`は、GUIやSDKと同様に、それをそのまま使用します。`process`は、その上で動作し、最後のプロジェクトの代わりに自身のプロジェクトを読み込みます。 CLIであればバックエンドを再起動する場面（`--restart`、またはインポートで何も見つからなかった`process`）において、 このコマンドはユニットの名前を指定し、`sudo systemctl restart <unit>`と表示して`1`を終了します。 これは、その方法で停止されたバックエンドは停止したままになってしまうためです。`GET /api/status`はユニットの名前を`service`と指定します（サービスが実行されていないバックエンドの場合は`null`）。**アップグレードと削除。** パッケージのアップグレードにより、すべてのバックエンドサービスが停止した後、有効になっている各サービスが順次再起動されます。最初に再起動されるのは `@<user>` ユニットです（手動で起動されたものの、有効化されていないユニットは、再起動後と同様に停止したままになります）。 有効化されているものが複数ある場合、1つだけが実行され、他は前述のように失敗状態のままとなります。アップグレード時には、`Chloros: N backend units are enabled: …` および `Keep one enabled: sudo systemctl disable --now <unit>` が表示されます。削除すると、それらは停止し、無効化されます。 パージすると、`/var/log/chloros`も削除されます。

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

amd64 .deb と同じレイアウトで、受信バッファおよび [バックエンドサービス](#running-the-backend-as-a-service-headless) が含まれており、**JetPack 6** (L4T R36) 上の Jetson Orin / Orin NX / Orin Nano 向けです。 含まれる PyTorch は、Orin 向けの JetPack 6 の CUDA ビルドであるため、GPU 処理やテクスチャ対応のデベイヤーは、Jetson の GPU 上で実行されます。 このパッケージにはCUDA自体は含まれていません。JetPack 6がインストールするCUDAおよびcuDNNライブラリを使用します（これらが含まれていないイメージの場合は`sudo apt install nvidia-jetpack`）。 CUDA対応GPUが利用できない場合、処理はCPU上で実行され、テクスチャ対応モードでは標準のデベイヤーが使用されます（[`process`](#chloros-cli-process)を参照）。

### マシンごとに1回認証

すべてのプラットフォームにおいて、SDK/CLI呼び出しが機能するには、Chloros+への初回ログインが必要です：

```bash
chloros-cli login user@example.com 'YourPassword'
```

認証情報は`~/.chloros/user_session.json`にキャッシュされます。

### バックエンドの起動

`lattice`、`project`、および `daq` コマンドは、すでに実行中のバックエンドと通信します。これらのコマンドはバックエンドを起動することはありません（[規約](#conventions)を参照）。これらのコマンドを使用する間は、バックエンドを常時実行しておいてください：

| プラットフォーム | 方法 |
| --- | --- |
| Windows | Chlorosデスクトップアプリを開く（そのバックエンドはCLIにも対応しています）、または専用のターミナル（インストールフォルダの`backend`フォルダ内）で`"C:\Program Files\Chloros\backend\chloros-backend.exe"`を実行し、そのウィンドウを開いたままにしておく。 |
| Linux / Jetson | 専用のターミナルで `chloros-backend` を実行するか、[バックエンドサービス](#running-the-backend-as-a-service-headless) を使用して常に1つを実行し続けてください。 |

一度に実行できるバックエンドは 1 つだけです。すべてのクライアント（GUI、CLI、SDK スクリプト）がこれを共有します。 主要なコマンド（`process`、`login`、`status`、…）もこれを使用し、実行中のバックエンドがない場合は起動します。

**バックエンドはコマンドの実行間も動作し続けます**。そのため、次のコマンドが実行される際、再度起動処理が行われることはありません。 サインアウト時（`chloros-cli logout`、ただしChlorosアプリが開いている場合や、実行またはオフラインビルドが進行中の場合は除く）、`chloros-cli backend stop`を実行したとき、あるいは何も使用されていない状態で30分経過すると自動的に停止します： コマンド、スクリプト、アプリ、実行中またはオフラインビルド、接続されたカメラや光センサーが一切ない状態。`backend stop`は、実行中またはオフラインビルドが進行中の場合は拒否し、その名前を通知します。`--force`は直ちに停止させ、その実行またはビルドは失われます （バーストデータは保持され、`array-build-video`が再ビルドを行います）。 バックエンドが起動する前に `CHLOROS_BACKEND_IDLE_MIN` を設定して、30 分という時間を変更してください（`0`：決して停止しない）；[バックエンドサービス](#running-the-backend-as-a-service-headless)は、決して自発的に停止することはありません。`chloros-cli backend status`は、バックエンドが実行中かどうか、何がそれを使用しているか、および実行中のオフラインビルドがあるかどうかを表示します。`process`は、Chlorosアプリがプロジェクトを開いた状態で起動している場合を除き、実行中のバックエンドでそのプロジェクトを開きます 開いている場合は、その操作を拒否します。その場合は、アプリから処理を行うか、まずそのプロジェクトを閉じてください。以前の`process`やSDKスクリプトによって開かれたままになっているプロジェクトは、単に上書きされます。

### インストールの確認

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest`は、7つのチェックすべてに合格した場合にのみ`0`を終了します。最後のチェックにはCUDA GPU（NVIDIA GPU、 またはJetson Orin）とノイズ除去モデルが必要です。そのため、これらがないコンピュータでは`Failed: CUDA + Denoiser`と表示され、`1`で終了します。ただし、GPU専用のテクスチャ対応デベイヤーを除くすべての機能はそこで動作します。

> **Chloros+ サブスクリプションが必要です。**CLI を使用するには、有効な Chloros+ プランが必要です。**Copper**は Chloros+ のエントリーティアです。有料の Chloros+ ティアはすべて CLI/SDK へのアクセス権を有していますが、無料の**Iron** ティアのみアクセス権がありません。（プラン ID 対応表：`0`=Iron/無料、 `1`=Copper、`2`=Bronze、`3`=Silver、`4`=Gold。） [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing) でアップグレードしてください。
>
> この下限は、CLIだけでなくバックエンドによっても強制されます。有料プランがない状態でSDKやCLIのフラグが設定されたリクエストは、それが`chloros-cli`、Python SDK、あるいは手作りのHTTPクライアントからのものかに関わらず、`403 PLAN_UPGRADE_REQUIRED`で拒否されます。 ログアウトしている呼び出し元には、代わりに `401 AUTH_REQUIRED` が返されます。プランの猶予期間中（月額プランの場合は毎月30日間、年間プランの場合は有効期限まで）はオフラインでもアクセスが可能ですが が経過すると停止します。`chloros-cli status`は引き続き動作し、その理由を確認できます（これは、ティア制限の対象外となっている唯一のSDK/CLIルートである`GET /api/license-status`です）。

---

## 5分でできるクイックスタート

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process`（ステップ 2）は、バックエンドが実行されていない場合に独自の実行用のバックエンドを起動し、処理が完了すると停止します。そのため、ステップ 4 以降ではバックエンドを起動してください。 [バックエンドの起動](#start-the-backend)を参照してください。

---

## `chloros-cli process`

画像が格納されたフォルダを、Chlorosのフルパイプライン（ターゲット検出 → キャリブレーション → ビネット → 反射率 → インデックスのエクスポート）で処理します。

### 概要

```
chloros-cli process INPUT [OPTIONS]
```

### 位置引数

| 引数 | 説明 |
| --- | --- |
| `INPUT` | `.raw + .jpg`（Survey3）、`.tif/.tiff`（LATTICE）、または`.dng`ファイルを含む入力フォルダのパス。 |

### 共通オプション

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `-o, --output PATH` | デフォルトのプロジェクトフォルダ内に、`-n`という名前で新しいフォルダを作成 | プロジェクトフォルダ。 プロジェクト名はこれに準じて命名され、すでに `project.json` を含むフォルダがある場合は、上書きされるのではなく、そのフォルダに `_1` / `_2`という同階層のフォルダが作成され、上書きされることはありません。デフォルトのプロジェクトフォルダは、`set-project-folder`（またはGUI）が選択したフォルダ、それ以外の場合は`Documents/MAPIR/Chloros Projects`となります。 |
| `-n, --project-name NAME` | 自動 (タイムスタンプ) | プロジェクト名。**`-o`**がない場合にのみ使用されます（**`-o`** がある場合は、プロジェクト名はフォルダ名になります）。 |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware`は、Survey3画像に対してChloros+ニューラルデベイヤーを使用します。処理速度は遅くなりますが、画質は向上します。これはSurvey3専用です： LATTICEによるキャプチャは常に標準のデモザイクを使用するため、LATTICEキャプチャを含むフォルダの場合、`process`はインポート後に`texture-aware`を拒否し、その枚数を示すエラーメッセージを表示して終了します。`1`で終了します（Chlorosアプリもこれらに対して「Texture Aware」機能を提供していません）。CUDA対応GPUが必要です（Windows / Linux x64ではNVIDIA GPU、JetsonではOrinのGPU）： GPUがない場合、実行では`standard`が使用され、バックエンドにはその理由を説明する1行のログ（`[DEBAYER] … needs a CUDA GPU …`）が出力されます（`--verbose`で表示）。 |
| `--vignette / --no-vignette` | `--vignette` | ヴィネット補正。 |
| `--reflectance / --no-reflectance` | `--reflectance` | 反射率キャリブレーション（パネルターゲットが見つかった場合はそれを使用、LATTICEの場合はNISTのシリアルごとのキャリブレーションを使用）。LATTICEマルチスペクトルの場合、これは反射率**積**としても機能します トグル — [製品ごとのエクスポート・トグル](#per-product-export-toggles-lattice-multispectral)を参照。 |
| `--ppk` | off | サイドカーファイルからの PPK GNSS 補正を適用します。 |
| `--exposure-pin-1 CAMERA_MODEL` | none | PPK 時刻同期用に、露光ピン 1 に接続されたカメラモデル。 |
| `--exposure-pin-2 CAMERA_MODEL` | なし | PPK 時刻同期用に、露出ピン 2 に接続されたカメラモデル。 |
| `--recal-interval SECONDS` | `0` | 再校正の最小間隔（秒単位）（プロジェクトの *再校正の最小間隔*）。 |
| `--timezone-offset HOURS` | `0` | 光センサーのUTCからのタイムゾーンオフセット（時間単位）（プロジェクトの *光センサーのタイムゾーンオフセット*）。 |
| `--format FORMAT` | `TIFF (16-bit)` | `TIFF (16-bit)`、 `TIFF (32-bit, Percent)`、`TIFF (8-bit)`、`PNG (8-bit)`、`JPG (8-bit)`のいずれか。Chlorosは画像のビット深度を絶対に上げません： すべて8ビットの画像の場合、より深い形式の画像は `TIFF (8-bit)` として書き込まれ、実行時に一度その旨が表示されます。 8ビットと12/16ビットのキャプチャが混在するプロジェクトでは、各キャプチャは自身のビット深度、またはそれ以下の深度でエクスポートされます。Radianceは常に32ビット浮動小数点です。 |
| `--indices NAME [NAME ...]` | なし | プリセット名による植生指数 (`NDVI`、`GNDVI`、`OSAVI`、`SAVI`、 `EVI`, `GCI`, …)、大文字小文字は区別されません。それぞれが、そのカメラのバンドに対してプロジェクトに追加され、新しいプロジェクトが含まれます。プリセットではない名前、またはカメラのバンドでは作成できない （例：`NDRE`にはREとNIRが必要ですが、これらを単一のカメラで提供できるものはありません）は、警告として表示され、インデックス画像は生成されません。 |
| `--input-level {auto,raw,debayered,processed}` | `auto` | LATTICE TIFF に対するパイプラインのエントリポイントを強制します（Survey3.raw には影響しません）。 また、**RAWデータなし**のキャプチャを処理できるようにする回避策 — [キャプチャフォルダの構成](#what-a-captures-folder-looks-like)を参照。 |
| `--debayered / --no-debayered` | on | 線形デベイヤー処理済み出力を出力します（`Debayered_Images`）。[製品ごとのエクスポート切り替え](#per-product-export-toggles-lattice-multispectral)を参照してください。 |
| `--preview / --no-preview` | オン | 表示プレビュー（`Preview_Images`）を出力します：RGB = ホワイトバランス（利用可能な場合はDAQ光源、それ以外はグレイワールド）＋ガンマ；マルチスペクトル = 偽色伸張。 |
| `--radiance / --no-radiance` | オン | float32 ラディアンスを出力 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | LATTICE反射率プロダクトの基準： `auto` = QAを通過したフレーム内のターゲットが絶対基準（QAフラグを持つターゲットは、測定スコアにおいてDAQと競合する）、DAQダウンウェリング（ρ = π·L/E）によるフォールバック； `target` = 厳格（DAQによる置換なし）；`daq` = DAQを優先。［製品ごとのエクスポート切り替え］(#per-product-export-toggles-lattice-multispectral)を参照。 |
| `--target-reflectance-dir DIR` | なし | ユニットごとの**測定済み**ターゲット反射率スキャンのディレクトリ（`<serial>.csv`）； 取得に失敗した場合は、公称の T3/T4P スペクトルにフォールバックします。 |
| `--array-alignment / --no-array-alignment` | オン | LATTICE アレイ： 各キャプチャの `Chloros:Alignment*` XMP に記録されたモジュール間位置合わせ情報を、すべての処理済みプロダクト（デベイヤー処理済み／プレビュー／放射輝度／反射率／インデックス）に適用します。タグのない画像に対しては何も行いません。 |
| `--array-alignment-crop / --no-array-alignment-crop` | トリミング | アライメント済みのエクスポートデータを、アレイの共通オーバーラップ領域にトリミングし、すべてのモジュールが同一のフットプリントを共有するようにする。`--no-…`はセンサーのキャンバス全体を維持する（ソース領域の外側は黒で塗りつぶされる）。 |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | アライメントワープのためのリサンプリング。`nearest`はソースのDNを正確に保持します（放射測定値のピクセル間混合なし）。 |

### ターゲット検出オプション

| フラグ | 説明 |
| --- | --- |
| `--min-target-size PIXELS` | 検出器におけるキャリブレーションターゲットの最小サイズ（ピクセル単位）。 |
| `--target-clustering 0-100` | ターゲットのクラスタリング率（デフォルトは `60`）。 |
| `--target / --targets` | 入力データの `target` または `targets` サブフォルダ内でのみキャリブレーションターゲットを検索する（処理が高速）。 |

### 例

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### 製品ごとのエクスポート切り替え (LATTICE マルチスペクトル)

LATTICE 処理は、**1回のパスで該当するすべての製品**に展開されます。4つのタイプごとの切り替えオプション — `--debayered`、 `--preview`、`--radiance`、`--reflectance` — はすべて**デフォルトでON**になっています。いずれかを無効にするには、`--no-<type>`形式を使用してください。 RGBマスターカメラは、常にデベイヤー処理済みの+プレビューのみを出力し（バンドごとの放射輝度/反射率は出力しない）、 そのため、`--radiance`/`--reflectance`はこれらに対してはノーオペレーションとなります。Survey3の`.raw`（標準の反射率／ターゲットパスに従う）では、これらのトグルは無視されます。*（旧`--radiometric-output {reflectance,radiance,sensor-response}`フラグは**削除**され、これらのトグルに置き換えられました。`sensor-response`レベルはもはや存在しません。）*

| 製品 | 出力 | DAQダウンウェル処理が必要か？ |
| --- | --- | --- |
| `--debayered` | 線形デモザイク (`Debayered_Images`)：キャプチャが `debayered` レベルで保存するファイルと同じもの — キャリブレーションパックのダークオフセット（DSNU）をRAWデータから差し引き、その後OpenCVのエッジ認識デモザイク処理を行う。線形LUT、フラットフィールド、アンミックスは行わない。 | 不要 |
| `--preview` | プレビュー表示（`Preview_Images`）：RGB = WB + ガンマ；マルチスペクトル = 偽色伸張。 | なし |
| `--radiance` | 完全な放射測定チェーン（`Radiance_Images`）から得られた float32 単位の W/m²/sr/nm。 | 番号 |
| `--reflectance` | uint16 反射率 ρ（`32768` = 1.0）、 Pix4D対応。 | **はい**。ただし、QAに合格したフレーム内のターゲットによって固定されている場合は除く（以下を参照）。 |

`--reflectance-source`は反射率の基準を選択します：**`auto`**（デフォルト）は、QAに合格したフレーム内のターゲットを**絶対基準**とします — ターゲットにアンカーされた経験的ラインチェーンは、除外されたパネル上でクロススコアされ、測定された勝者がバンドごとに適用されます。DAQがキャリブレーションできないバンド、またはDAQスロープのフィッティングに失敗したバンドは、ターゲット自身のラインを採用します （`ReflectanceElBandMethods`が対象を指定） — フラグ（パネルの欠落、バンドの順序異常）付きでQAに合格したターゲットも、その測定スコアにおいてDAQの分岐値と競合し、敗れたフレームには`ReflectanceChain=daq`がスタンプされる — ターゲットが存在しない場合やQAに失敗した場合は、DAQのダウンウェル・ディバイド（ρ = π·L/E）にフォールバックする；**`target`**は厳格な動作（DAQによる置換なし）；**`daq`**はDAQを優先する動作を選択する。ターゲットの幾何学的構成（ArUco／固定ROI／ストリップ）はプロジェクトのターゲット設定から取得される；`--target-reflectance-dir DIR`は、ターゲットユニットのシリアル番号／QRごとに**測定** スキャン（`<serial>.csv`）を保持し、ターゲットユニットのシリアル番号/QRコードに基づいて検索され、フォールバックとして公称T3/T4Pスペクトルが使用される。

DAQ反射率パスは、記録された**`.daq`**（DAQ-U/M/E）**または、画像データと共に存在するDAQ-Mネイティブの`.csv`**から、**タイムスタンプが一致するダウンウェルリング**を自動的に特定します。カメラごとまたはDAQごとのキャリブレーションバンドルがローカルにキャッシュされていない場合、 パイプラインは初回使用時に**AWSから自動的に取得**します（1回のみインターネット接続が必要；`~/.chloros/`としてキャッシュされます）。**どのカメラの測定値か。**キャプチャが保存する `.daq` は、それが記述するカメラの測定値です。そのカメラ名は、隣にある `<file>.daq.cameras.json` サイドカーに （`lattice capture`、`multi-capture`、 `array-capture`およびGUIも同様です。RAWバーストは、マニフェスト内で各メンバーの読み取り値を個別に指定します）――そして`process`は、**それらのカメラからの画像にのみ**これを照合します**にのみ一致します。したがって、独自の読み取り値を持たないスタンドアロンカメラは、他のカメラのDAQを借りることができなくなりました。DAQをそのカメラにバインドするか、DAQ自身の録画 （`daq pool-record`、またはGUIで開始された記録）、あるいはそのサイドカーを持たない任意の`.daq`（古いキャプチャを含む）は、依然としてすべてのカメラに対応します。

#### 反射率ピクセルの読み取り（Pix4D / Metashape / 独自のスクリプト）

反射率は整数DNとして保存されており、**ρ = 1.0 に対応するDNはソースカメラによって異なります**：

| ソース | ρ = 1.0 に対応する値 | 判別方法 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768`（ρ 2.0までの余裕あり） | ファイルに「XMP `Chloros:PixelScale=32768`」とスタンプが押されている。 |
| Survey3 | `65535`（ρ 1.0でクリップ） | `Chloros:*`のXMPタグがない — その欠如こそが*手がかり*です。 |

**定数であると仮定するのではなく、`Chloros:PixelScale`を読み取り、それを除算する**。 このタグは uint16 ドメインで定義されているため、再スケーリングを行う出力形式においても `32768` のまま維持されます — `TIFF (16-bit)`、`TIFF (8-bit)`、 `PNG (8-bit)`、`JPG (8-bit)`、`TIFF (32-bit, Percent)`はすべて自己記述型です （保存されたデータ型をまず uint16 に正規化します：8 ビットからは ×257、float からは ×65535）。

> **古いビルドのファイルにはスケール情報が含まれていない場合があります。** 8ビットの反射率TIFFは再スケーリングされ、他のファイルと同様に`Chloros:PixelScale`という名前が付けられます。バージョン1.3.0以前のビルドでは、代わりに0～255に*クリップ*されていたため、ρ≈0.008を超えるすべての値が255に平坦化され、 さらに、`Chloros:PixelScale`と`MicaSense:RadiometricCalibration`のタプルも両方とも省略されていました。**LATTICE反射率ファイルにこのタグがない場合、スケールが設定されていると仮定せず、再処理を行ってください**。もともと割り切れないピクセルを無理に割り算しようとするのは避けてください。

#### インデックスピクセルの読み取り（NDVIおよびその他の植生指数）

**整数インデックスラスターは、画像ごとのコントラスト伸張であり、−1…1のマップではありません。** `TIFF (16-bit)`、`TIFF (8-bit)`、 `PNG (8-bit)`、および `JPG (8-bit)` といったインデックス製品（8ビットキャプチャとして書き出される8ビットTIFFを含む）は、

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

ここで、`IndexMin` / `IndexMax` は **そのフレーム固有の最小および最大の有限インデックス値** です。 DN 0 は −1 ではなく、フルスケールは +1 ではありません。

Chlorosは、マッピングが可逆性を保つように、両端点にスタンプを付けます：

| タグ | 意味 |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — 以下の規約 |
| `Chloros:IndexMin` | DN 0 として格納されるインデックス値 |
| `Chloros:IndexMax` | フルスケールとして格納されるインデックス値 |

これらを読み取り、逆変換を行い、保存されたDNを、保存されたデータ型の最大値で0…1の範囲に正規化します。これにより、1つの式ですべての整数形式を網羅できます：

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

> **`-config pix4d.config`** は、そのフレームのすべてのピクセルが同じインデックス値を持っていたことを意味します。ラスタは中間スケールで埋められており、除算する対象がないため、端点値を採用します。

> **`Chloros:Index*`タグがない場合は、指定されたマッピングがないことを意味します。** LUTエクスポートとは、ピクセルがLUTのしきい値ウィンドウに基づいてグラデーションカラーで着色されたレンダリングであり、有限のインデックス値を持つピクセルがないフレームには、指定すべき範囲が存在しません。 いずれの場合もマッピングを想定しないでください。これは、`PixelScale`が欠落している場合と同じルールです。**ストレッチ処理は画像ごとに適用されるため、1回のフライトの2つのフレームをDNとして比較することはできません**— 各フレームは独自の端点を持つため、同じNDVI値でも画像によって異なるDNに割り当てられます。フライト全体にわたる定量的な作業（ゾーン統計、固定閾値、時系列分析）を行う場合は、**`TIFF (32-bit, Percent)`**をエクスポートしてください。インデックスラスターは、リスケールやタグを必要とせず、**インデックス値そのもの**がfloat32としてそこに書き込まれます。フォーマット名とは異なり、インデックスプロダクトは×100ではありません。そのスケーリングは整数データ型の反射率にのみ適用されます。

#### アンミックス・クランプ・マスク（どのピクセルでバンドがクランプされたか）

3×3スペクトルアンミックスにより、バンドが負の値になる場合があります。これは、光にそのバンドがほとんど含まれていない場合や、シーンがカメラのアンミックスマトリックスが記述する範囲外にある場合です。 これは実際の値ではないため、エクスポートでは0に固定され、ライブビューにも同じ0が表示されます。バンドは数ピクセルでクリップされることもあれば、ほぼすべてのピクセルでクリップされることもあり、マスクにはそのいずれかが記録されます。 `Chloros:Band{R,G,B}UnmixClampFraction`は、フレームのどの程度の範囲でそれが発生したかを示し、クランプマスクは**どのピクセル**で発生したかを示します：

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— 製品のフォルダ**の横**にある`clamp_masks/`フォルダ内に保存され、決してその内部には保存されません（`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; `process`のエクスポートでは、`tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`）— すべての放射輝度、 反射率および屈折率の各データセットについて、少なくとも1ピクセルがクリップされたもの：キャプチャおよび`process`のエクスポートデータ（および`array-build-video --save-tiffs`のフレームごとのTIFFファイル）を含みます。シングルバンドの`uint8`は、 そのプロダクト独自のピクセルグリッド上で生成されます（向き、ROI、ビニング、配列のアライメントを含みます）。各マスクには、そのプロダクトのカメラ名（`Make` / `Model` / シリアル番号）、撮影時刻、レンズタグが記載されており、そのビットが表すバンドがリストされています （`XMP-Camera:BandName`、ビット0が先頭）。`ImageDescription`および`Software`は、これをクランプマスクとして識別しており、キャリブレーション情報は含まれていません。
- **ビット**：ビット 0 = 製品の R チャンネルにおけるバンド (`Chloros:BandRName`)、ビット 1 = G (`BandGName`)、ビット 2 = B （`BandBName`）。ビットがセットされている場合、そのバンドにおける当該ピクセルの値が 0 にクランプされたことを意味します。このビットおよびこれに基づいて構築されたインデックスは、信頼度が低いものとして扱ってください。
- **`Chloros:UnmixClampMask`** は、アンミックス処理が実行されたすべての製品に付与されます。これは、製品自身のフォルダを基準としたマスクのパス（`../clamp_masks/<product folder>/<product name>_clampmask.tif`、 常に `/` で区切られています。画像のフォルダにこのパスを追加してマスクを開いてください）、またはピクセルのクリッピングが行われていない場合は `none` となります。 タグがない場合は、その製品でアンミックス処理が実行されていないことを意味します（RGBまたはモノクロカメラ、RAW/デベイヤー処理済みレベル）。
- `lattice capture` / `array-capture` は、保存されたファイルの下にマスクのパスを表示します （`clamp mask: …`）の下にマスクのパスを表示し、キャプチャ結果ではそれをフレームの `clamp_mask` としてリストします。
- `lattice index --live --save-multiband`は、マルチバンドスタック用にソースカメラごとに1つのマスクを書き込みます — [`lattice index` — マルチバンドスタックマスク](#lattice-index--multiband-stack-masks)を参照してください。動画にはマスクがありません。その代わり、`array-build-video`は各フレームのクランプ率を報告します（[`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)を参照）。

どの階層の製品フォルダにもマスクが含まれていないため、製品フォルダ（`radiance/`、`reflectance/`、 `process` エクスポートの `*_Images/` フォルダ）は、そのままの状態で Pix4D、Metashape、またはその他の写真測量ソフトウェアに渡すことができます — サブフォルダも含めて — そのまま渡すことができ、これらのソフトウェアは画像のみを取り込むためです。その上のフォルダ（そちらには `clamp_masks/` やその他のすべてのエクスポート形式も含まれています）ではなく、プロダクトフォルダそのものを渡してください。Chloros 独自のインポート機能では、マスクはスキップされます。マスクの読み取り： `(mask & 1) != 0`は「ここでRチャンネルの値がクリップされた」、`& 2`はG、`& 4`はBを表します。

#### エクスポート時に引き継がれる EXIF

`process` は、ソース画像の **GPS ブロックとその ExifIFD** を をすべての出力ファイルにコピーするため、
エクスポート時には `FocalLength`、`FNumber`、`ExposureTime`、`ISO`、 `DateTimeOriginal`、および
`CameraSerialNumber`が地理参照情報とともに含まれます。

**`FocalLength`は、写真測量において必須です。** Pix4Dは、焦点距離と高度から
地上サンプル距離（GSD）を算出します。このタグがない場合、著しく誤ったスケールで再構築されてしまいます。ある
オレンジ農園での49枚の撮影からなる飛行データにおいて、このタグの欠如により、411 m × 160 mの敷地が、
47.8 km × 13 kmの領域として再構築されてしまいました。これは主に「nodata」で構成される455 MPのオルソ画像であり、GSDを確認するまでは、タイリングの問題や
BigTIFFの問題と誤認されていました。もしオルソ画像が不自然な
スケールで出力された場合は、 、まずエクスポートされたデータに対して`exiftool -FocalLength`を実行してください。

このコピーは意図的に**`-all:all`**ではありません：IFD0'の構造タグは、コピーされるとLATTICEの出力を
破損させるためです。また、`ExifImageWidth` / `ExifImageHeight`は、
*ソース*のキャプチャを記述しているため除外されています。これらは、サイズ変更されたエクスポートデータであり、そのまま使用すると
XMPはコピーではなく直接書き込まれます。これは、ExifToolが
XMPブロックのコピー時に同一呼び出しのXMPタグを破棄するためです（これによりMAPIRの
キャリブレーションタグが失われてしまうため）。

### 出力の保存先

生成されたファイルは、**プロジェクトフォルダの下に、カメラごとにグループ化され、さらにファイル形式ごとに分類されて**書き込まれます：

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE用のカメラフォルダは`LATT-<sensor>-<lens>-F<filter>`（キャプチャのEXIF
`Model`）で、Survey3の場合は`<model>_<filter>`となります。センサーとフィルターを共有するが、
レンズが異なる2台のカメラは、ヴィネット、視野、歪みが異なるため、別々のツリーに保存されます。フォーマット
フォルダ構成は `--format`: `tiff16`、`tiff8`、`png8`、`jpg8`、あるいは `tiff32` から
`TIFF (32-bit, Percent)` といった形式になります。

> **エクスポートされたすべてのファイルは、ソースファイルの名前を保持します。** > `capture_…_raw.tif` のラディアンス・エクスポートは、依然として `capture_…_raw.tif` という名前ですが、実際には
> `tiff32/Radiance_Images/` フォルダ内に保存されています。**製品を識別するのはファイル名ではなくフォルダ名です**。 そのため、
> `*radiance*.tif` をグロブ検索しても何も見つかりません。代わりにディレクトリで一致させるようにしてください。

### 光センサーの記録 — キャリブレーション済み `.daq` + `.csv`

`process`は、入力フォルダ内の`.daq`の記録も処理しますが、**その処理には**
画像データは一切必要ありません。DAQ-U / DAQ-M / DAQ-Eを単独で飛行させただけでも完全な
キャプチャとなり、`.daq`ファイルのみを含むフォルダも有効な入力となります。

DAQは、キャリブレーション**なし**でも記録可能です。これは、一般に公開されている
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) レコーダー
(`record_daq.py`) がデフォルトで行っていることです。これらは生のセンサーカウントを書き込み、ファイルにタイムスタンプを付けることで、
Chlorosがそのセンサーの工場出荷時キャリブレーションを**シリアル経由**（まずローカルキャッシュ、
次にMAPIRクラウド）からそのセンサーの工場出荷時校正データを取得し、適用します。`process`は結果を書き戻します：

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv`は、1回の測定ごとに1行を格納します：UTCタイムスタンプ、積分時間、総パワー、
明所視／暗所視ルクス、PPFD（およびその青／緑／赤の分割値）、ピーク波長、そして
センサー独自の波長グリッドに基づく全スペクトル。`.daq`は、
再度キャリブレーションを行うことなく再インポートされます。

`<name>`は、その記録固有のファイル名です。 異なるフォルダにある2つの記録でも、
2台のカメラによって同じミリ秒に保存された
1つの「キャプチャごとの測定値」（`<capture>/daq/daq_data_<sensor id>_<ts>.daq`）を共有でき、
それぞれが独自の成果物を取得します。最初の記録は
`<name>`を保持し、次のものは名前**の後に**区別するためのフォルダ名を追加します
（`<name>__<capture>_daq_calibrated.daq`）。したがって、録画の日付と時刻が依然として先頭に表示されます。

成功した場合、実行結果は `Light-sensor products written: N (calibrated .daq + .csv)` と報告されます。
括弧内の記述は実際に書き込まれた内容を表しており、具体的には
`(RAW COUNTS — this sensor has no calibration bundle)`（バンドルのないセンサーの場合）や
`(N calibrated, M raw counts)`（両方を格納するフォルダの場合）となります。 バックエンド独自の
`[DAQ-EXPORT]` および `[RUN-SUMMARY]` という見出しも、同様の方法で表現が導き出されています。これら
3つはいずれも、未加工のエクスポートデータを「校正済み」と呼ぶことはできません。

キャリブレーション・バンドルを取得できない DAQ-U / DAQ-M / DAQ-E の記録（オフラインであるか、
そのセンサーのキャリブレーションがファイルに存在しない場合）は、**理由を付してスキップ**され、
`[DAQ-EXPORT]` 行に記録されます。生カウントを含む「校正済み」ファイルとして書き出されることは決してありません。
インターネットに接続して再実行してください。理由は、リーダーが実際に
そのファイルに対して特定したものです（スキーマが読み取れない、バンドルがない、書き込みエラーなど）。また、実行
サマリーには**個別の**理由が一覧表示されます。1つの原因で20ファイルがスキップされた場合、それは1つの
原因として扱われ、同じ原因が20回繰り返されたことにはなりません。 CLIは、実行の最後にスキップされた各記録に
その理由を付けて名前を付けます：

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

**1つも**エクスポートされない記録のフォルダは失敗します：
[画像を書き出さない実行は失敗する](#a-run-that-writes-no-images-fails)を参照してください。

ファイル名が `_calibrated` または `_raw` (`field_raw.daq`) で終わる記録は **インポートされません**：これらは `process` が自身の生成物に付ける名前であり、生成物が記録として読み込まれると二重にカウントされてしまいます。
インポートされません**：これらは `process` が自身の生成物に付与する名前であり、生成物を
記録として読み戻すと、2 回カウントされてしまいます。 CLIにはその旨が表示され、空のフォルダであると報告する代わりに、ファイル名 —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— が表示されます。このような記録は、処理する前に名前を変更してください。

#### DAQ-Aの記録は生カウントとしてエクスポートされる

**DAQ-A** シリーズは、シリアル番号ごとのバンドルシステムが導入される以前に開発されたものであり、取得すべきキャリブレーションバンドルを
持ちません。代わりに、現場で反射率ターゲットを用いてキャリブレーションが行われるため、
バンドルは不要でした。これらの記録を拒否すると、データを
出力する手段が全く失われてしまうため、**別の名前**でエクスポートされます：

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

ファイル内のフラグではなく別のファイル名となっているのは、この情報（ファイル名）が
単なる名前としてメールで転送されても失われないようにするためです。 `.csv`というヘッダーには
`raw spectral sensor counts (NOT irradiance)`と記載されており、値はファイル**内**でのみ比較可能である
という警告が表示されています。これはまさに、ターゲットベースのキャリブレーションがそれらを
使用する目的であり、センサー間で比較するためのものではありません。 電力依存の測光カラム（総電力、明所視および
暗所視ルクス、PPFD）は、カウントから積分されるのではなく**NULL**として書き込まれ、実行
サマリーには `RAW COUNTS` と記載されているため、ログ内の「エクスポートされた」と表示されるため、これらを放射照度として解釈することはできません。

旧式の**v1.01 / v1.02**記録（DAQ-A-SDがこれらを書き込みます）には、読み取りごとのエポック情報がなく、
ファイルの書き込み時刻のみが含まれています。 画像↔下向き放射のマッチングツールは依然としてこれらを受け付けません（
フレームを書き込み時刻と照合すると、目に見えない形で誤りが生じるため）が、エクスポーターはこれらを読み込み、
CSVファイルには「`clock=daq_created_on`」と出力されるため、どのクロックに基づいているかがわかります。

### 注意事項

- `process`は、フォルダがSurvey3、LATTICE、またはその混合であるかを自動的に検出します。
- 進行状況はServer-Sent Eventsを介してストリーミングされます。CLIでは、スレッドごとの進行状況（検出中、分析中、処理中、エクスポート中）がリアルタイムで表示されます。
- Linux/Jetsonの場合、CLIはスワップ領域を確認し、大容量のフォルダを処理する前に警告を表示することがあります。また、テクスチャ対応のデベイヤーは、低消費電力のJetson（Nano、Orin Nano）において、GPUの周波数制限を自動的に適用します。
- 処理が成功すると、実行レポートに書き出された画像プロダクトの数が表示されます（`Image products written: N`。アンミックス処理でピクセルがクリップされた場合は`(plus M clamp masks)`）。。
- **`process`は、実行中のバックエンドを安全に共有します。** すでに実行中のバックエンドが、以前のプロジェクトが読み込まれた状態でアイドル状態であり、カメラ、配列、DAQを保持していない場合にのみ、そのバックエンドを自動的に再起動します。 これら（GUIによるアレイのストリーミング、SDKスクリプト、DAQの記録）のいずれかを保持しているバックエンドは、そのまま使用され、再起動されることはありません。そのため、`process`はライブアレイと並行して実行できます。バックエンドが実行中であることを報告した場合、`process`は、その実行が終了するまで最大15秒待機します（終了したばかりの実行がクリアされるまでには約1秒かかります）、その後、`Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).` エラーを返して拒否します。グローバルフラグ `--restart` は依然として新しいバックエンドを強制し、その上で実行中のカメラ、アレイ、または DAQ セッションをすべて終了させます。Linux の systemd サービスとして実行されているバックエンドは、決して再起動されません： `process`がそれを使用し、代わりに`--restart`が`sudo systemctl restart <unit>`を出力します（[バックエンドをサービスとして実行する](#running-the-backend-as-a-service-headless）を参照）。Chlorosアプリ独自のバックエンドも同様です。アプリが開いている状態では、`process`がそれを使用し、アプリのプロジェクトの代わりに自身のプロジェクトを読み込みます（その場所にバックエンドを起動すると、開いているアプリがロックアウトされてしまいます）。
- **実行中の処理は、そのプロジェクトを維持します。** 処理の実行が進行中である間（アプリ、`process`、SDKスクリプト、または`chloros-mcp`によって開始されたもの）、バックエンドは、ファイルの作成、開く、インポート、追加、削除、移動、 削除、またはプロジェクトの処理設定の変更を拒否し、HTTP 409 `PROCESSING_IN_PROGRESS` および `A processing run is in progress on project '<name>', so ... now is refused` を返します。これを受け取った `process` は `1`で終了します。表示設定（サムネイルサイズ）は引き続き保存されます。処理モードおよびプロジェクトのターゲット反射率設定についても同様に拒否されます（実行は開始時のモードを維持します）。 実行中にプロジェクトに保存されたキャプチャ（アプリ、CLI、SDK、または`chloros-mcp`からのもの）は通常通り保存されますが、プロジェクトのファイルリストに追加されるのは実行が終了したときのみです。 その旨が応答に示されています（`project_add`）。また、プロジェクト内で停止された光センサーの記録についても同様です。以前は、2台目のクライアントが実行中のプロジェクトを切り替えることができ、実行中のランが別のプロジェクトにエクスポートされていました。
- **Ctrl+C は、CLI だけでなく、バックエンドの実行も停止させます。** CLIはバックエンドに処理の停止を要求し、`Waiting for the backend to stop (Ctrl+C again to stop waiting)...`を出力した後、`Processing stopped.`が返されるまで最大120秒待機するため、`chloros-cli process … && next`は、まだ実行中のバックエンド上で`next`は開始されません。2回目のCtrl+Cで待機が終了します。その後、中断されたコマンドと同様に、`process`は`130`を終了させます。

#### 画像を出力しない実行は失敗する

プロダクトの出力を要求したにもかかわらず、実行結果が **なし** — つまり `project.json` と
`calibration_data.json` のみ — であった場合、`process` はこれを失敗とみなします。その際、
`Processing finished but wrote no image products.` を出力し、**ゼロ以外の値で終了**するため、スクリプトで
これを検出できます。 メッセージには、プロジェクトフォルダ名と一般的な原因が記載されています：

- 入力フォルダがキャプチャとして認識されなかった（レイアウトと `--input-level` を確認してください）、または
- 要求されたすべてのプロダクトが、それらのカメラには適用できないとしてスキップされた（例：
  RGB専用カメラから放射輝度／反射率を要求した場合など）。

`--verbose`を指定して再実行し、バックエンドログで`[LATTICE-EXPORT]` / `[EXPORT-CHECK]`の行を確認してください。
これらは、CLIの出力には通常表示されない、カメラごとのスキップ理由を説明しています。

意図的にメタデータのみを出力する実行（すべてのプロダクトをオフにし、`--indices`を使用しない）は、
**成功**とみなされます。これは、この場合、空の画像が出力されるのが正しい結果だからです。

#### エクスポートされたファイルの検証ができなかった実行は失敗となります

実行後、`process`はExifToolを使用してエクスポートされたファイルのタグを確認します。ExifToolが実行できなかった場合、
`エクスポート検証に失敗しました: ExifToolを実行できなかったため、M個のエクスポートファイルのうちN個のタグは
は確認されませんでした（バックエンドログの [EXIFTOOL] 行を参照）` and `出力ファイルは書き込まれ、
その場所に残ります。確認するには再実行してください。`, does not print `処理完了！`と表示し、**非ゼロで終了**します。
出力ファイルは保持されます。

#### 実行でキャプチャされる内容

`process`が実行される前に完了したキャプチャは含まれます。実行中に保存されたキャプチャは含まれません。`process`は
除外されたキャプチャの数を警告し、再実行するとそれらを含めることができます。

#### 画像が失われた実行は失敗する

実行中に画像の取得に失敗した場合、`process`は最も
一般的な原因を記載した`N image(s) failed and were not exported`を出力し、`Processing complete!`は出力せず、**非ゼロで終了**します。 その他の出力データは
書き込まれ、そのまま残ります。また、バックエンドのログには各失敗が記録されます。

**光センサーのみの実行**も同様です： `.daq`の記録ファイルが含まれるフォルダには、定義上、エクスポート可能な画像が存在せず、
その実行は、代わりに書き込まれた較正済みの`.daq` / `.csv`に基づいて評価されます。
もし**何も**書き込まれなかった場合（すべての記録がスキップされた場合）、
`Processing finished but wrote no light-sensor products.` を出力し、各記録にスキップされた
理由を名前として付け、**非ゼロで終了**します。

---

## `chloros-cli login`

このマシンをChloros+クラウドアカウントで認証します。認証情報は`~/.chloros/user_session.json`に安全にキャッシュされます。

```
chloros-cli login EMAIL PASSWORD
```

### 例

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$`（パスワードの一部を削除または複製して実行）。401エラーが発生した場合、CLIは自動的に`$$`を再- その後、重複を排除したパスワードの半分で再試行します。再試行が成功するとログインされ、次回使用する正しい一重引用符の構文が表示されます。

> **ヘッドレス／スクリプトでの使用：キャッシュされたセッションがない場合、ログインプロンプトが表示されます。** キャッシュされたセッションなしで実行されるコアコマンド（`process`、`status`、 `export-status`、`time-sync`、…）は、キャッシュされたセッションなしで実行されると、処理を続行する前に標準入力（stdin）から `Email:` / `Password:` の入力を求めます。 stdinが閉じられている場合（cron、systemdユニットなど）、`login cancelled`で即座に失敗します。stdinがオープンなターミナルまたはパイプである場合は、入力を待ちます。ヘッドレス作業をスケジューリングする前に、マシンごとに1回、`chloros-cli login EMAIL PASSWORD`を実行してください。

---

## `chloros-cli logout`

キャッシュされたセッションをクリアし、次回の呼び出し時に新規ログインを強制します。

処理の実行中にサインアウトしても、その処理は停止しません。実行は開始時の処理モードのまま完了し、次の実行では標準処理が使用されます。`logout`（`The processing run in progress finishes in <mode> mode; the next run uses standard`）にもその旨が記載されており、サインアウトした際にもアプリに同様のメッセージが表示されます 。

```bash
chloros-cli logout
```

---

## `chloros-cli status`

現在のライセンス階層（Iron/Copper/Bronze/Silver/Gold）、認証済みユーザー、およびデバイスバインディング数を表示します。

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Thread-4のエクスポート進行状況をリアルタイムでポーリングします。別のシェルから`process`を実行している**最中**でも安全に呼び出せます。

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

CLIの表示言語を設定します（CJK、RTL、インド系言語を含む38言語に対応）。スクリプトをレンダリングできないレガシーコンソールでは、英語にスムーズに切り替わります。

```
chloros-cli language [LANG_CODE] [--list]
```

### 例

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## プロジェクトフォルダコマンド

これらは、デフォルトのプロジェクトフォルダの場所（GUIと共有）を管理します。

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` は、新規プロジェクトが保存されるフォルダを出力します。設定されたフォルダが存在しなくなった場合（削除された、名前が変更された、または接続されていないドライブ上にある場合）、 CLIはGUIと同様にデフォルトのフォルダに切り替わります。デフォルトのフォルダ名が標準出力（stdout）に表示され、存在しないフォルダ名を指定した警告が標準エラー出力（stderr）に出力され、終了コードは`0`となります。

---

## `chloros-cli update`

Linux／Jetson のみ。`version_url` から `/etc/chloros/update.conf` までのバージョンを確認し、新しいバージョンが存在する場合、 対応する `.deb` をダウンロードし、`sudo dpkg -i` を使用して直ちにインストールします（確認のプロンプトは表示されません。`sudo` ではパスワードの入力を求められる場合があります）。 Jetson では、JetPack 6 システムにのみ更新が提供されます。Windows では、このコマンドは何も実行せず、「`up to date`: 新しいインストーラーで更新してください」と報告します。

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linux/Jetson では、CLI も **起動のたびに自動更新チェック** （ノンブロッキングで、コマンドの実行を遅らせることはありません）：`/etc/chloros/update.conf`を読み取り、結果を1時間キャッシュし（`~/.chloros/update_cache.json`）、新しいバージョンが存在する場合は`Update available: vX.Y.Z / Run: chloros-cli update`を出力します。 エラーが発生した場合やWindowsでは、何も表示せずにスキップされます。`CHLOROS_DISABLE_UPDATE_CHECK=1`を設定すると、この機能は無効になります。1時間のキャッシュがあるため、その1時間以内に公開されたリリースであっても、最新の状態として読み取られる場合があります。

---

## `chloros-cli selftest`

7段階のスモークテストを実行します：バージョン、ポートの可用性、バックエンドの起動、`/api/test`、`/api/system-info`（GPU/CUDA/PyTorch）、ノイズ除去モデルの有無、 CUDA＋ノイズ除去機能の準備状況。実行中のバックエンドを使用するか、テスト用にバックエンドを起動して、終了後に停止します。`N/7 checks passed`と失敗したチェック項目を出力し、7項目すべてに合格した場合にのみ`0`で終了します。 CUDA GPUを搭載していないコンピュータでは、CUDA+ノイズ除去チェックが失敗します（[インストールの確認](#verify-the-install)を参照）。

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTPグランドマスターのステータスと制御。ChlorosホストがPTPグランドマスターを実行し、LATTICEカメラおよびDAQ-Eユニットは、デバイス間のタイムスタンプを同期させるためにこれにスレーブとして接続されます （デバイス間で約1 ms）を実現します。PTPは**タイムスタンプ**を同期させます。アレイのカメラが同時に露光するのは、PTPによるものではなく、M8ハードウェアトリガーがカメラを起動するためです（モジュール間の遅延は100 µs未満）。 サブコマンドを指定しない`chloros-cli time-sync`は、`status`を実行します。

| サブコマンド | 説明 |
| --- | --- |
| `status` | グランドマスターの状態、BMCAの優先順位、クロックID、およびそのパケットがネットワークに到達しているかどうかを表示します：`health`は、グランドマスターが稼働している場合は`OK` （グランドマスターが実行されていない場合は `STOPPED`）、または `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` であり、かつ `health_detail` が存在する場合、 その後、`WARNING:`という行がstderrに出力されます（stdoutはJSONのままです）。`sync_sent` / `announce_sent` は送信されたメッセージのみをカウントし、`*_send_errors` および `last_send_error` は送信されなかったメッセージをカウントします。 カメラリンクが切断されて復旧した後、グランドマスターは自動的にソケットを再構築します（`rebinds`、`last_rebind`）。 |
| `peers` | Delay_Req を通じて検出されたスレーブの一覧（カメラおよび DAQ-E センサー）。 |
| `cameras` | カメラごとのPTPヘルス状態（`PtpStatus`、`PtpOffsetFromMaster`、`PtpMeanPathDelay`）。 |
| `restart` | グランドマスタープロセスを再起動します。 |
| `set-priority --priority1 N --priority2 N` | BMCAの優先順位を上書きします。 |

グランドマスターは、カメラのNIC上でアナウンスを行います （`169.254.x.x`アドレスを含む任意のアダプタ）でアナウンスを行います。アクティブなアダプタがない場合、デフォルトルートのアダプタにフォールバックしますが、これは有線接続の場合に限られ、Wi-Fiや仮想アダプタ（Tailscale、WSL）は対象外です。これは、カメラやDAQ-Eがそれらの背後に配置されていないためです。アナウンスする対象がない場合、ソケットなしで動作し（`health` `NO_SOCKETS`）、次にカメラやDAQが接続されると、その時点でアクティブなNIC上で再構築されます。`time-sync restart`はこれを即座に行います。

グランドマスターは、バックエンドとは別に独自のプロセスで実行されます（`own_process` / `process_pid` in `time-sync status`）の横で独自のプロセスとして実行され、Windows上では高優先度（`process_priority`）で動作するため、バックエンドがビジー状態（処理の実行、エクスポート、 複数のコマンドを同時に実行）によってタイムスタンプの付与が遅延することはありません。以前は、そのような遅延が一度にすべてのカメラの時計をずらす原因となっていました。このプロセスはバックエンドで終了します。もし開始できない場合、グランドマスターは以前と同様にバックエンド内で実行されます（`own_process` false、バックエンドのログにもその旨が記録されます）。

### 例

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICEカメラ制御。すべてのサブコマンドはChlorosバックエンドを経由します。バックエンドがカメラプールを管理しているため、後続のCLI呼び出しでは同じオープンハンドルが再利用されます。

### 共通オプション（ほとんどのサブコマンドで共有）

| フラグ | 説明 |
| --- | --- |
| `-d, --device N` | カメラインデックス（デフォルト：0）。 |
| `-s, --serial SN` | 特定のシリアル番号。`--device`を上書きします。 |
| `--serials SN1,SN2,…` | マルチカメラ操作用の、カンマ区切りのシリアル番号。 [一時的なマルチカメラコマンド](#multi-camera-transient-sessions) は、これらのカメラのみを開き、最初のカメラがトリガーを発動します。 |
| `--all` | 検出されたすべてのカメラに対して操作を実行します。 |
| `--exposure US` | 露光時間（マイクロ秒単位）。 |
| `--gain DB` | ゲイン（dB単位）。 |
| `--pixel-format FMT` | 例：`BayerRG8`、`BayerRG12`；`Mono8`、 M3M では `Mono12`。この名前は、カメラのファミリー内でのビット深度を設定します： M3Mでは、ベイヤー形式の名前がその深度のモノクロ形式を決定します（`BayerRG8`は`Mono8`を設定します）。M3Cでは、モノクロ形式の名前がベイヤー形式を決定します （`Mono12`は`BayerRG12`を設定します）。 |
| `--width N` / `--height N` | 画像サイズ。 |
| `--preset {default,high_quality,high_speed,triggered}` | 設定プリセットの適用：ピクセル形式、露光、ゲイン、フレームレート（`high_speed`: `BayerRG8`, 5 ms、6 dB、60 fps；`high_quality`：`BayerRG12`、20 ms、0 dB；`lattice presets`に一覧表示）。 M3Mでは、フォーマットは同じ深度のモノラル形式となります：`high_speed`は`Mono8`をキャプチャし、 `high_quality`、`triggered`、および`Mono12`をキャプチャします。これと共に指定されたフラグ （`--exposure`、`--gain`、`--pixel-format`）はプリセット設定よりも優先され、プリセットで指定されていない設定についてはカメラ自身の設定が維持されます。すべてのフリー実行されますが、`triggered`は例外です。これはライン2のハードウェアエッジに対してカメラを待機状態にします。そのラインに信号が入力されない場合、キャプチャを行わず、永久に待機し続けます。 |
| `-o, --output DIR` | 出力ディレクトリ （デフォルト：`output`）。 |
| `--packet-size {auto,jumbo,standard,N}` | GVSP パケットサイズ。`auto` は ICMP+GVSP プローブを実行します。`jumbo` = 9000; `standard` = 1500。 |

### LATTICE カメラの初回接続ワークフロー

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### サブコマンドリファレンス

#### 検出と情報

| サブコマンド | 目的 |
| --- | --- |
| `lattice info` | 接続済みのカメラを一覧表示（ベンダー、モデル、シリアル番号、IP、MAC）。 |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | ホストシステムを分析し、最適なカメラ設定を決定します。`--no-discover` はカメラの検出をスキップします（処理が高速で、NIC のみの分析となります）。 |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | NIC設定の確認・修正（Linuxでは、カーネルのソケット受信バッファも対象 — [Linuxホスト](#linux-hosts--socket-receive-buffers)を参照）；帯域幅およびFPSの推定。 |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | 安定スキーマのバックエンドネットワーク機能 + アレイの推奨（`status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`）。`--width` / `--height`はデフォルトでフルセンサーとなり、 `2048` × `1536`。`0`は`ok`、`auto_capped_fps`および`auto_shrunk`、また`1`から`needs_force_slip`および`error`へ移行し、 `--json`の有無にかかわらず。これは`--backend-url`（デフォルトは`http://127.0.0.1:5000`）でバックエンドに到達し、`CHLOROS_BACKEND_URL`では到達しません。 `auto_capped_fps`は要求された解像度を維持しますが、ターゲットfpsに上限を設けます。`recommended.recommended_target_fps`を読み取り、接続ターゲットとして渡してください。これを成功とみなしてください。 エラーとは見なさない。`auto_shrunk`は、要求されたフレームがいかなるフレームレートでも配信できない場合にのみ表示される。connectは、独自にフレームを縮小することは決してない。 |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | カメラを開かずに実施する「仮定分析」（`--width` / `--height` デフォルト `2048` × `1536`）。これと `network-analysis` の両方は、connect がストリーミングする内容を予測します。`--pixel-format` を指定しない場合、 connectが適用するカメラごとのフォーマット（Mono12 / BayerRG12）が使用され、`--packed auto`（デフォルト）では、[パックされた12ビット](#packed-12-bit-transport---packed）という、このコンピュータ上でconnectが下すであろう決定が適用されます。したがって、記載されているfpsは、このアレイが動作するレートです。どちらもパッキングの決定を出力します。**`--n-active`は、このアレイのみではなく、ネットワーク上のカメラの総数を表します**— スタンドアロンのカメラが同時にストリーミングする場合、または回線の帯域幅予算がカメラ数を過小評価した需要に基づいて計算されている場合は、この値を増やしてください（デフォルト：`len(--models)`）。 常に、集計値である `Wire budget:`（要求される MB/s 対 衝突防止の上限）と `Max cameras:` の行を出力し、アレイがワイヤの容量を超過している場合は `** OVER-SUBSCRIBED**` をフラグとして表示します — [アレイの fps およびバーストモデル](#array-fps--burst-model)を参照。 |
| `lattice gpu` | GPU のステータスを表示します。 |
| `lattice firmware [--update] [--force] [-y\|--yes]` | カメラのファームウェアを確認または更新します。 ローカルの `.fwa` 選択は固定されています。`firmware/<MODEL_PREFIX>/` 内の、ビルドの `MIN_FIRMWARE_VERSION` と一致するファイルが存在する場合、そのファイルがフラッシュされます （フォールバックとして最高バージョンのみが対象）ため、ディスク上にステージングされた新しいベンダーイメージは、そのピンがバンプされるまで無効なままとなります。意図的に新しいリリースは、署名付きAWSマニフェストを介してユニットに配信され、新しいバージョンがある場合はこちらが優先されます。 |
| `lattice presets [--apply NAME]` | カメラプリセットの一覧表示または適用。`--apply` は、プリセットが定義する内容（ピクセル形式、露出、ゲイン、フレームレート、トリガー）を設定し、カメラが読み取った設定値を以下のように出力します： M3M上では、プリセットの深度のモノクロ形式；動作するフレームレート（プリセットで指定された値より低くなる場合がある。`high_speed`は60 fpsを指定するが、1 Gbpsリンク上のフルフレームカメラでは約35 fpsで動作する）。 そのほか、openコマンドで書き込まれたすべての設定は、検出されたままの状態で復元されます。`--apply default`は、カメラを工場出荷時のデフォルト設定にリセットします。具体的には、Line2でのフリーランニング自動露出、リンクスループット制限および欠陥補正の無効化、RGBカメラの場合はキャリブレーション済みのホワイトバランスが保持されます（その他のカメラではユニティが適用されます）。 カメラのメモリには何も保存されないため、次回の電源投入時には以前と同様にカメラ自身のメモリから起動します。キャプチャや接続時の `--preset` として、`default` はプリセットではありません。 |
| `lattice status` | カメラのライブステータス。 |

#### キャプチャ

| サブコマンド | 目的 |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | 単一フレーム。 **デフォルトではすべてのエクスポート形式が保存されます**(`--processing all`); [キャプチャのエクスポートレベル](#capture-export-levels-the-all-default)を参照してください。 `--processing` は、`all`、`raw`、`debayered`、 `radiance`、`reflectance`、または `display` を受け付けます（`preview` は `display`）については`preview`が受け入れられます。カメラに触れる前（`1`を終了する前）に、それ以外の名前が指定された場合は拒否されます。`--levels`は明示的なサブセットを保存します（`--processing`を上書きします）。 Chlorosが保持していないカメラがキャプチャ用に開かれ、その露出は[一時コマンド](#multi-camera-transient-sessions)と同様に、まずシーンから設定されます（最大数秒間）。 キャプチャが終了すると（終了方法にかかわらず）、そのカメラは検出時の状態（フォーマット、露出、カラーパイプライン、キャプチャによって変更されたすべての設定 — [マルチカメラ（一時セッション）](#multi-camera-transient-sessions) のリストを参照）に戻されます。 使用されたキャプチャを読み込んだDAQのデータは、`.daq`というサイドカーとして保存され、生データのみのグラブが含まれます。`--no-daq` / `--daq-tolerance-ms`でこれを調整します （[キャプチャを読み込むDAQが保存する内容](#the-daq-reading-a-capture-saves)）。`--force-daq`も引き続き使用可能です。`--jpeg-quality` = JPEG画質 1–100 （デフォルトは95）。**開いているプロジェクトに保存されたファイルは、そのプロジェクトに追加されます。** アプリと同様に、Chlorosで開いているプロジェクトのフォルダ内にファイルが保存されたキャプチャは、そのプロジェクトのファイルリストに追加されます。他の場所に保存されたファイルはそのまま残されます。この動作を示す1行のメッセージがコマンド出力されます (`Project: ...`)、あるいは処理の実行中に、ファイルが保留されており、実行終了時に追加される旨のメッセージを表示します。`multi-capture`、`array-capture` （`--smart`も同様）および`project capture`も同様です。 |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Ctrl+Cが押されるまでディスクへストリーム出力します。`tiff`の実行のフレームは、`raw`ファイルとして、そのカメラのキャプチャ配列が書き出されます。これは、センサーのモザイクであり、各プレーンが独自の値を持ち、非圧縮で、キャリブレーションパケット （2048×1536の12ビットフレームは、シーンにかかわらず6.0 MiBにパケットが加算されます）が含まれており、`process`はこれらを基に処理を開始します。 `png` および `jpg` は 8 ビットを保持しているため、これらのランは表示用のデモザイク処理済みフレームであり、`process` はそのデータから開始しません。DAQ にバインドされている場合 （`daq pool-bind`）またはプール処理が行われる場合、各フレームには、ランの `daq/` フォルダ内に独自の `.daq` および `.match.json` が割り当てられ、 `lattice capture`が1つを保存し、`--no-daq`がこれを無効にすると、停止時に保存されたフレーム数が表示されます。各フレームは完全なメタデータと共に1回書き込まれます。 Ctrl+Cを押すと、カメラが解放される前に、すでに取得済みのフレームが書き込まれます。ドライブが満杯になると処理を停止し、`Failed:` / `Disk full:`と出力して終了し、`1`となります。書き込みできなかったフレームがある場合も、`1`で終了します。ハードウェアトリガー用にアームされたままのカメラ （`TriggerMode` `On`）は、実行中はフリーランモードに設定されます。実行が終了すると、カメラは検出時の状態に戻されます。つまり、トリガー設定および実行中に変更されたすべての設定 （露光、ゲイン、フォーマットなど）はすべて変更前の状態に戻されます。ただし、実行中の設定をカメラ自身のメモリに保存する `--keep-settings`（フリーラン）や、そのメモリを工場出荷時の設定にリセットする `--clear-settings` を除きます。 フレームを記録できなかったランは失敗とみなされます。10秒間フレームがない場合、自動的に停止し、理由を記載した「`No frames:`」を出力して、「`1`」で終了します。 ライターが追いつけなかったフレームはドロップされたものとみなされ、停止時にはその理由（カメラのフレームレート、ライターのフレームレート、およびキュー）を記載した「`Dropped:`」が出力されます。フレームレートを下げたり、8ビットのピクセル形式を使用したりすれば、すべてのフレームを保持できます。 このコマンドは2秒ごとにバックエンドにチェックインを行います。録画を停止せずにコマンドが終了した場合（`kill -9`、メモリ不足）、バックエンドは最後のチェックインから30秒後に録画を停止し、カメラを元の状態に戻します （`stop_reason`、`client_gone`、およびバックエンドログへの1行の記録）。`multi-stream`および`align-stream`も同様に動作します。 カメラの起動に、コマンドの待機時間（60秒；`multi-stream` / `align-stream`の場合は180秒）よりも長くかかる場合（カメラがまだ起動中の状態）、バックエンドによってその起動処理は取り消されます： カメラは元の状態に戻され、閉じられ、録画は一切実行されません（HTTP 504 `CLIENT_DEADLINE_PASSED`）。|
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | ブラウザベースのライブ MJPEG プレビュー：カメラに接続し、ストリームURLを表示してブラウザで開きます。`--ae-damping` は自動露出のダンピング（0.4～100）を設定します。 |

##### ライブストリーム（MJPEG；RTSP非対応）

Chlorosは、**HTTP経由のMJPEG**（`multipart/x-mixed-replace`、境界 `frame`）で配信します。**RTSPはサポートされていません。** ブラウザでURLを開くか、任意のMJPEGクライアント（OpenCVの`cv2.VideoCapture(url)`、ffmpeg、VLC）で読み込んでくださいで読み取ることができます。カメラは、まずバックエンドで接続されている必要があります（`lattice cam-connect`、`lattice array-connect`、`lattice viewer`、またはGUI）。

| URL (`http://127.0.0.1:5000` + …) | ストリーム |
| --- | --- |
| `/api/camera/stream/<serial>` | プレビュー表示。最大約30 fps。配列の要素の場合は、 `?max_h=`（デフォルトは `720`）および `?jpeg_q=`（デフォルトは `80`）は、高さと JPEG 画質を設定します。 これは、URL `lattice viewer` が開くURLです。 |
| `/api/camera/<serial>/stream-annotated` | GUIのオーバーレイが描画された同じプレビュー： `zebra=1`、`crosshair=1`、`grid=3x3`（または `9x9`）、 `histogram=1`、`peaking=1`、`spot=1`；クエリなしの場合、そのカメラに対してプロジェクトに保存されているオーバーレイ設定が適用されます。 |
| `/api/camera/<serial>/stream-reflectance` | バンドごとの8ビット反射率：ρ = ピクセル ÷ `X-Chloros-Pixel-Scale`ヘッダー（`refl_max`が設定されていない場合は255）。`X-Chloros-Pixel-Scale`ヘッダー（`refl_max`が設定されていない場合は255）。`fps`（デフォルトは`10`）、 `jpeg_q`（デフォルトは`95`）、`max_h`（デフォルトは`0` = ネイティブ）、 `band=<name>`、`align=1`。JPEG形式のため、非可逆圧縮です。カメラに反射率データがない場合（RGBカメラで、キャリブレーションパックがなく、DAQの読み取り値もない場合）、HTTP 409で拒否されます。|

- **カメラ間のフレーム照合。**3つのストリームすべてにおいて、**配列の要素**が送信する各データには、`X-Chloros-Group-Id`（配列の同期グループカウンター）、`X-Chloros-Frame-Id`、および `X-Chloros-Timestamp-Ns` ヘッダーを含みます。異なるカメラから送信されたパートで、同じ `X-Chloros-Group-Id` を持つものは、同じトリガーから送信されたものです。スタンドアロンカメラにはグループがなく、グループ ID を送信しません。 ブラウザはパートごとのヘッダーを無視します。マルチパートストリームを自ら解析するクライアントがそれらを読み取ります。
- **アレイビューが開いている間は一時停止します。** アレイのメンバーの `/api/camera/stream/<serial>` は、アレイの統合ライブストリーム（例：GUI のアレイビュー）が開いている間、静的な「一時停止」フレームを送信し、それが閉じると再開します。

#### センサーの調整

| サブコマンド | 目的 |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | 任意の GenICam ノードの読み取り/書き込み。 `--set ExposureTime=…`（`Gain=…`）は、`ExposureAuto`（`GainAuto`）スイッチなしで、まず自動モードをオフにします。 `lattice exposure --set`のように動作する。カメラが受け入れない値が指定された場合、stderrに`Failed to set <node>: …`を出力して`1`を終了する。`--get`および デフォルトのグループ化ビューは、その自動モードの実行中に `note:` を `ExposureTime` / `Gain` の下に表示します — [自動モードでの露出とゲインの読み取り](#reading-exposure-and-gain-under-auto-mode)を参照。 |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | 露出とAE。`--set`の露出値または`--brightness`のターゲット値があなたの設定として保持されます （Chlorosの自動露出制御はこれをそのままにします）。ゲインは変更されないため、オートモードでは、`lattice gain`を設定するまで、ターゲットに向けて調整が続きます。 `--auto`は露出制御を元に戻し、`--brightness-auto`はターゲットをChlorosの自動ターゲット設定に戻します。フラグが設定されていない場合、露出ノードを読み取り、 `note:`は`configure --get`と同じです。カメラが拒否する`--set`値の場合、stderrに`Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)`を出力し — 書き込みによってオフにされていた自動モードは、オフのままにされるのではなく復元される — `Exposure: … us (auto OFF)`行は出力されず、`1`で終了する。 |
| `lattice gain [{--auto \| --off \| --set DB}]` | ゲインおよびオートゲイン。 カメラが `--set` の値を拒否した場合、stderr に `Failed to set Gain: …` を出力し、`Gain: … dB (auto OFF)` 行は出力されず、`1` で終了します。 |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | センサーのROIおよびビニング。ビニングは常に**平均**です — 下記を参照。 |
| `lattice format [--set FMT] [--list]` | ピクセル形式。 |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | ハードウェア／ソフトウェアトリガー。 |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (フラグなし = ワンショットWB) | WB処理。 RGB/ベイヤー方式のカメラのみ。モノクロM3Mではノーオペレーション（スキップ）となる。ワンショット処理では、算出されたホワイトバランス（`BalanceWhiteAuto`およびゲイン）が保持される。それ以外の設定については、オープン書き込みが行われた値が元のまま戻され、ワンショット処理が実行できない場合はすべての設定値が元のまま戻される。 |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB表示用カラーパイプライン。`natural`（デフォルト）は低コストなライブ処理です； `enhanced`は、デフリンジ＋ヴィブランス＋CLAHEローカルコントラストを追加し、完全なハブパリティのルックを実現します。フレームごとの処理コストは約2倍となるため、**ライブ**フレームレートは低下します。ただし、保存されたキャプチャにはいずれの場合も完全な処理が適用されます。RGB/ベイヤー方式のカメラのみ対応； モノクロ M3M ではスキップされます。 |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | ライブプレビューの彩度/コントラスト（RGBフィルター搭載カメラ）；`100`はニュートラル、`--reset`は両方を100に設定します。 保存されたキャプチャには影響しません。モノクロ M3M ではスキップされます。 |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]`（フラグなし = 読み取り） | **ハイライト**の表示（RGBフィルターカメラ）：最も明るく、最も白いピクセルを暗くする — 例：明るい白い表面が、単調な白い塊として表示される場合。`0` = オフ、 `-100` = 最強。ライブプレビューおよび表示レベルのキャプチャはこの設定に従いますが、放射輝度／反射率は決して従わず、クリップされたセンサーデータを復元することはできません（その場合は露出を下げてください）。モノクロ M3M ではスキップされます。 |
| `lattice filter [--list]` | Chlorosが認識するフィルターモデルを一覧表示します（`RGN-IMX265`、`OCN`、`NGB`、…）。カメラのフィルターはモデル文字列から読み取られるものであり、設定することはできません：`--set NAME`はモデルを1つ検索するだけで、その後`1`を終了し、`setting the filter model is not supported`に移行します。 |
| `lattice power [--sleep]` | 電源／熱ノードをプローブし、低消費電力アイドル状態を切り替えます。 `--sleep` はカメラをスリープ状態にします（リンクスループット制限が有効化され（`DeviceLinkThroughputLimitMode` `On`）、その下限値に設定され、カメラに省電力モードがある場合はそれが有効になります） その他の設定については、検出時の状態がそのまま復元される； スリープに失敗した場合は、設定をすべて元の状態に戻す。ウェイクアップ時には、スリープ状態でのリンク設定（制限値、モード、およびパケット遅延 `GevSCPD`。制限値が有効な間、カメラはこの値を 0 に保持する。この記録はこのコンピュータに保存されるため、 そのため、バックエンドの再起動後のウェイクアップでもその状態が検出される）、あるいは、このコンピュータがスリープ状態にしなかったがスリープ状態に見えるカメラ（制限が最低値で有効）については、制限を無効にし、`GevSCPD`を80 （工場出荷時の設定）で0のまま残されている場合；フロア値より上に設定されたリミットはそのまま残されます。プローブとウェイクは、オープン時に書き込んだ内容を元に戻します。 |

> **ビニングは常に「平均」です。** `lattice resolution --binning N`は平均ビニングを設定し、
> モード切り替え機能は提供しません。 「Sum」ビニングは各DNにN²を乗算するため、ビニングステップごとに約2ストップ分の
> ハイライトのヘッドルームを消費します。Chlorosは、
> 放射測定チェーンの前にその係数を除算して元に戻しますが、「Sum」によってセンサーの天井値まで押し上げられたピクセルはクリップされ、
> 復元することはできません。 これは「平均」では得られない利点は何もない――インデックスはスケール不変であり、放射輝度
> スカラーは「平均」スケールで校正されている。カメラがすでに「合計」に設定された状態で到着した場合（
> 他のツールによって、あるいは工場出荷時設定へのリセットによって）、 Chlorosはノードの読み出しからそれを検出し、
> 補正を行います。`lattice resolution --binning N`を実行すると、Averageに戻ります。1.2.0
> `--binning-mode`フラグは引き続き受け入れられるため、古いスクリプトも実行されますが、警告が表示されるだけです：
> ビニングは「Average」のままです。

> **カメラのビニング、ピクセル形式、トリガーを変更しても、設定は保持されません。** カメラで
> バックエンドがオープン状態を維持しストリーミングを行っている場合（`cam-connect`、GUI）、`lattice resolution --binning`
> は反映されず（ビンニングが依然として有効であることが表示される、`Binning: 1x1`）、
> `lattice format --set` および `lattice trigger --mode` はカメラによって拒否される
> （`SC_ERR_ERROR -1001`）。 まず解放してください（`lattice cam-disconnect -s <SN>`）、または
> 接続時に設定してください（`cam-connect --pixel-format …`、`array-connect --binning …`）。

##### オートモードでの露出とゲインの読み取り

カメラの `ExposureAuto` (`GainAuto`) が実行されている間、その `ExposureTime` (`Gain`) ノードは、カメラの現在の動作状況ではなく、**最後に書き込まれた値**を維持します。 バックエンドが開いたままにしているカメラ（`cam-connect`、`array-connect`、GUI）、`lattice configure --get`、デフォルトの `lattice configure` ビュー、および `lattice exposure`（フラグなし）は、ノードの値と、その下にカメラの最新のフレームを示す注記を表示します：

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

注記からフレーム値を読み取ってください。ノード値は最後に書き込まれた値のみです。HTTP経由では、同じ`POST /api/camera/<serial>/read-nodes`応答により、`nodes`は変更されず、`frame_values` （`{"ExposureTime": <us>, "Gain": <dB>}`、最新のフレーム）と `notes`（上記の文）が追加されます。これら両方は、自動モードが `Off` ではなく、かつフレームが既知である場合にのみ存在します。 読み取り専用で開かれたカメラにはフレームが存在しないため、どちらも取得されません。`lattice gain`（フラグなし）はノードのみを出力します。注記については `lattice configure --get Gain` を使用してください。

##### アレイのメンバーに対する露光

アレイがストリーミングしている間、各メンバーは**`AcquisitionFrameRateEnable`**で実行され、そのフレームレートは配列のトリガーレートよりわずかに高い値となります。その周期は、トリガー周期から15％を差し引いた値（最低3 ms）か、トリガーレートがすでに最大レートに近い場合は、カメラ自身の最大レートとなります。 したがって、メンバー上の`configure --get`は、その設定において`AcquisitionFrameRateEnable`を`True`および`AcquisitionFrameRate`として読み取ります。アレイが 停止すると、イネーブルはオフに戻ります。**アレイメンバーにこれら2つのノードを書き込まないでください** — アレイはこれらをトリガーと同期させ、上書きします。

この設定は、メンバーの露光上限でもあります：10 fpsのトリガー（100 msの周期）では、**どのROIやビニング設定でも**約85 msとなります。バージョン1.3.0以前では、たとえアレイのトリガーがどれほど遅くても、小さなROIを設定すると、メンバーの露光時間はセンサーのフリーラン周期（768 × 304で4.9 ms）に制限されていました。**上限値を超える手動の`ExposureTime`は、`lattice exposure --set`または`configure --set`（あるいはGUI）で書き込まれた場合、**その上限値に合わせるためにトリガーの速度が低下します**。アレイはまず、そのメンバーのシーリングをその値に合わせて開き、低速レートで数トリガー周期動作させるため、書き込みに数秒長くかかることがあります。その後、トリガーは露光が許容する範囲を超えて速く発火することはありません。自動レートアレイでは、手動露光によって速度が低下していた各メンバーが自動露光に戻るたびに、以前の動作レートが回復します。 カメラ自体が撮影できない値は依然として拒否され、エラーメッセージに受け入れ可能な範囲が表示されます。また、その書き込みのためだけにオフにされていたメンバーの自動露出は、元の状態に戻されます。レシピ `apply` の露出はトリガーを遅らせません。上限値に固定されます。 Chlorosは、より高速なトリガーに合わせて手動露光時間を短縮することはありません。露光時間が許容する速度よりも速い固定レート（`array-connect --target-fps F`）が設定された場合、そのメンバーには露光時間に見合った十分な長さの上限が設定され、そのメンバーは露光時間が許容する速度以上のトリガーを受け付けません。

#### キャリブレーションとセンサー

| サブコマンド | 目的 |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | 反射率ターゲットからキャリブレーションを行う。最大 `--attempts` フレーム（デフォルトは `30`）を検索する。 不完全なフレーム（GVSPパケット損失）は、実行回数ではなく試行回数を1回消費する。フレームが完全に到着しなかった場合は、`no target detected in N attempts`、 または、完全なフレームが1つも到着しなかった場合は`no complete frame in N attempts: <last error>`で失敗します。 |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | DAQ-Mのスペクトルおよびバンドの振幅値を、 バックエンドを介してBluetooth（BLE）経由で読み取られます。`--mac`はセンサー固有のBLE MACアドレスであり、以下の3つのアクションのいずれかを実行する際に必要です。各ユニットが独自のMACアドレスを持つため（`daq pool-discover --only ble`で一覧表示されます）、デフォルト値は存在しません； これが指定されない場合、コマンドは終了します（`1`）。アクションが指定されていない場合は、ヘルプを表示します。`--camera SERIAL`は、フィルタモデルのフリート曲線ではなく、そのカメラ固有の測定スペクトル応答を用いて、表示されたバンド振幅を積分します — RGBカメラの場合、カメラのキャリブレーションパックが読み込まれている（接続されている）か、このコンピュータにキャッシュされている必要があります。見出しには、どのカーブが使用されたかが表示されます：`Band magnitudes (camera <SERIAL>):`、またはカメラにカーブがない場合（RGBカメラでない、 またはパックが読み込まれていない、あるいはキャッシュされていない）場合は、代わりにフリートカーブが使用されたことを示します。HTTP経由では、`POST /api/dls/cli-read`が`camera_serial`を受け取り、`band_magnitudes_source`を返します。 |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | 既存の画像にヴィネット補正を適用します。各画像には、そのレンズ固有のフラットフィールドが （EXIFの「Model」フィールドから取得）を適用します。`--lens-model`は、「Model」にカメラ名が指定されていない画像に適用されます。Modelがこれと矛盾する画像はスキップされます。 |

#### マルチカメラ（一時的なセッション）

一時的なコマンドは、1回の実行分だけカメラを開き、終了時に閉じる：`multi-capture`（永続的な配列が接続されていない場合）、`multi-stream`および`multi-test`、 また、以下に示す `align-calibrate`、`align-apply`、`align-stream`、および `index --live` も同様です。これらすべてに以下のルールが適用されます：

- **`--serials SN1,SN2,…`がカメラを選択します。** 選択されたカメラのみが開かれ、最初のカメラがトリガーを発動させます （`align-calibrate`、`--reference`が指定されている場合）。ネットワーク上に存在しないシリアル番号は、どのカメラも開かれる前に実行に失敗します。`--serials`がない場合、ネットワーク上のすべてのカメラが開かれます。 Chlorosがすでに保持しているカメラ（GUI、`cam-connect`、`array-connect`）は、`--serials`で指定されているかどうかにかかわらず、実行を拒否します。 そのカメラは、実行が対象とするネットワークを共有しているためです。（`--serials`がない場合の`index --live`は、代わりにそのプロファイルで有効になっているカメラを開きます。詳細は[Index / Vegetation Maths](#index--vegetation-maths)を参照してください。）
- **カメラは、`array-connect`がネットワークを共有するのと同じ方法でネットワークを共有します。** 各カメラは、ホストリンクが伝送するデータ量のうち、そのカメラに割り当てられた割合に合わせてペースが調整されます（これはパケット間の遅延であり、カメラのスループット上限ではありません。1 GbE上の4台のカメラは、すでに31.25 MB/sという下限を超えています）。 したがって、4台のカメラは1 Gbpsのリンク上で動作します。カメラが到達可能な最も遅いペースで同時に送信できない場合、各カメラは順番にトリガーされ、それぞれがリンクを独占して動作し、 この場合、コマンドは「`Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`」と表示されます。一度に1台でもカメラが収まらない場合、どのカメラもストリーミングを開始する前に実行は拒否されます： `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`、HTTP 409、終了 `1`。
- **すべてのカメラがフレームを送信しなければ、実行は失敗します。** フレームを送信しないカメラはエラーとなります： コマンドは「`Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`」と表示し、ここで「`<why>`」はそのカメラのフェッチエラー、続いてその原因を示す情報が表示され、その後「`1`」で終了します。 GVSPパケット損失（`IncompleteBufferError: GVSP packet loss …`）を伴って到着したフレームは、ネットワークがカメラの合計帯域幅（`Every missing frame arrived with GVSP packet loss: …`）を処理しきれていないことを意味します：ホストおよびスイッチのアップリンクのリンク速度を確認し（`lattice network`）、ジャンボフレームを使用するか、 または、`--serials`で起動するカメラの数を減らしてください。何も送信しなかったカメラは、通常、マスターからのトリガーを受信していません。そのM8同期ケーブルを確認し、その後`lattice multi-detect`を実行してください。カメラが欠落しているフレームグループに対して各コマンドがどのような処理を行うかは、各行に記載されています。
- **各カメラは、検出された状態のまま復元されます。** 実行が終了すると、すべてのカメラは、実行開始時に持っていた状態に戻ります：`TriggerMode`、`TriggerSource`、 `TriggerActivation`、`TriggerOverlap`、`TriggerDelay`、 および `Line2` と `Line3` 上の `LineMode` / `LineSource` （および、配列がスレーブのラインに与えるトリガー入力のデグリッチ、`LineFilterSelector` / `LineFilterWidth`）； その `PtpEnable`（および、これとセットで設定される `PtpSlaveOnly` / `PtpDomainNumber`）； `ExposureAuto` / `GainAuto` （および、手動値が設定されていた `ExposureTime` / `Gain`）； `TargetBrightness`、`ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`、 `GainAutoUpperLimit`、`ExposureAutoDamping`；`BalanceWhiteAuto`（およびホワイトバランスが手動設定だった各`BalanceRatio`）；`BlackLevelAuto` / `BlackLevel`; `PixelFormat`（ビニング）、`Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable`（および、ガンマがオンだった`Gamma`）、 `LUTEnable`、`ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; パケットサイズ（`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`）；ペーシング、`DeviceLinkThroughputLimitMode`（およびモードが`On`であった場合の`DeviceLinkThroughputLimit`）、各カメラに対するパケット間遅延 `GevSCPD` および開始時間ずれ `GevSCFTD` を、各カメラごとに配列が指定する；各ストリームが有効にするチャンクデータ （`ChunkModeActive`、各 `ChunkEnable` はその `ChunkSelector` の下にある）； `AcquisitionFrameRateEnable` （および `AcquisitionFrameRate` では、有効化がオンでした。オフの場合、レートは読み取り専用となり、ペーシングが許容する最大値が読み取られるため、設定にはなりません）。 異なる部分のみが書き込まれます。カメラは最初に停止し、フレーム期間を必要とする露光前のペーシングとフレームレート（カメラは `ExposureTime` を 1 フレーム期間に制限します）が設定されます。 `--keep-settings`では、トリガーおよび同期ラインの配線が元に戻り、カメラはフリーラン状態となり、残りの設定は実行セットで指定された通りにカメラ自身のメモリに保存されます。`--clear-settings`では、そこに工場出荷時の設定が書き込まれます。
- **露出は、最初のフレームが記録される前のシーンに基づいて設定されます。** ラン処理では、同期されたフレーム群を取得し、それらを破棄しながら、各カメラの露出とゲインをそれらのフレームに基づいて設定します。目標は、カメラのターゲットにおける平均輝度を、フレームのクリッピングが1％以下に抑えることです。ハイライトが明るいシーンでは、ターゲットに達する前に処理を終了します。 これには通常、2～6つのフレームグループが必要で、最大でも5秒かかります。
  - 各カメラは、接続されたアレイが開始時に設定したターゲットを、そのフィルターに基づいて狙います。マルチスペクトルカメラ（RGN、OCN、NGB）ではターゲット輝度 **40**、RGBカメラでは**60** です。 これは、露出目標値 80 の半分（RGB では 4 分の 3）に相当するため、明るい反射率パネルでもクリップが発生しにくくなります。`--brightness N` (0-255) は代わりに各カメラのターゲットを設定します（`multi-test` には `--brightness` がありません。その自動露出はオフになっているためです）。 バックエンドのログには、各カメラのターゲット名が記載されます：`[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`。
  - 露出が優先され、カメラ自身の自動露出上限まで設定されます。ゲインは、残りの範囲のみをカバーします。
  - 算出された露出とゲインは実行全体を通じて保持されるため、`multi-stream`の各フレームはすべて同じ露出で撮影されます。光の変化に対応する必要があるキャプチャは、接続された配列（`array-connect`）に配置する必要があります。 その自動露出は継続的に調整されます。
  - `--exposure` および `--gain` は、指定された通りに厳密に保持されます。 両方が指定されている場合、どのフレームも破棄されません。
  - 実行が終了すると、各カメラの自動露出が再び有効になります。
  - バックエンドのログには、結果として `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` または `exposure NOT solved after …`（理由付き）が記録され、適用されなかった各カメラについては `(not settled)` が記録されます。その場合でも、フレームは撮影されます。
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` は 5 秒の制限を変更し、`0` はこの機能を無効にします（[環境変数](#environment-variables) を参照）。

**Chlorosがカメラ**（GUI、`cam-connect`、または接続されたアレイ）を保持している間、`multi-stream`、`multi-test`、 `multi-detect`、`align-apply`、`align-stream`、および`index --live`は実行を拒否します。これは、これらのコマンドがネットワーク上のすべてのカメラを開こうとし、保持中のカメラを再度開こうとすると 保持中のカメラを再度開こうとすると、バックエンドがクラッシュする可能性があるためです。（`index --live`は、開こうとするカメラが保持されている場合に実行を拒否します。対象となるのは、`--serials`またはそのプロファイルのカメラ、およびカメラを指定していないプロファイルのカメラです。） バックエンドは、HTTP 409 `CAMERA_HELD`を返し、保留中の各カメラと解放コマンド （`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`）を列挙し、コマンドは終了します `1`。 配列が接続されている場合、`array-capture --continuous`は`multi-stream`の代わりにそれを記録し、`array-capture` （デフォルトでアラインメント調整済み）は、`align-apply`の代わりにアラインメント調整済みのセットをその場に保存します。`multi-capture`および`align-calibrate`は、カメラを開く代わりに、それらのカメラを保持する接続済み配列を使用します。

| サブコマンド | 目的 |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | 同期ロールを持つすべてのカメラを一覧表示します。`--serials`を指定すると、それらのカメラのみが表示され、`MASTER`を指定すると最初のカメラのみが表示されます。 |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | 各カメラから 1 フレームずつ同期したデータを取得します。永続的な配列が接続されている場合、**デフォルトですべてのエクスポート形式**を保存し、使用された DAQ 読み取り値も保存します（[キャプチャが保存する DAQ 読み取り値](#the-daq-reading-a-capture-saves)）； 一時的なアレイなしのフォールバックは**デベイヤー処理のみ**が行われます（残りの処理については、まず `array-connect` を実行してください）。 `--brightness`はフォールバックにのみ適用されます：接続されたアレイは独自の自動露出を維持します。フォールバックでは`.daq`は保存されません （`--no-daq` / `--daq-tolerance-ms`はここでは効果がない）、また`multi-capture`は`--force-daq`を受け入れない。フォールバックでは、 フレームを送信しないカメラはキャプチャに失敗し（`Error: multi-capture failed: no frame from …`、終了 `1`）、そのフレームグループのファイルは保存されません。 |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | ストリーム同期フレーム（一時的）、 `--fps` デフォルトの `3` を、Ctrl+C または `--count` グループが指定されるまで処理します。カメラから何も送信されなかったフレームグループは書き込まれず、 3つ連続してスキップされるとストリームが終了します。いずれかのグループがスキップされた場合、実行は `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)` を出力して終了し、さらに `Error: the stream stopped: …` でストリームを終了したカメラ名を表示した後、`1`Xで終了します。ドライブが満杯になると自動的に停止し、`Failed:` / `Disk full:`を出力して`1`で終了します。 書き込みに失敗したファイルがある場合も、`1`で終了します。 |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO同期タイミングテスト（過渡現象）、`--count` デフォルトの`10`トリガー。ネットワークの共有状況を表示 （`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`、およびカメラが次々とトリガーされた場合の`Serial emit:`行。この場合、タイムスタンプにはその時間差が含まれる）、同期グループごとに1行の`Sync group: master SN:<serial> -> SN:<serial>, …`行（同期ケーブルに接続されていないカメラについては`(no slaves: not synced)`となり、これは単独でトリガーされる）、 その後、トリガーごとに1行：`[  0] SN:<serial>=<timestamp> \| …`。そのトリガーで何も送信しなかったカメラについては`SN:<serial>=(no frame)`。 続いて、各カメラごとに1行ずつ、`Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):`および`SN:<serial>: mean <±x> us, jitter <y> us`を記述します。これらは、1クロック以内にマスターに対して送信されたすべてのカメラのデータです。カメラのデバイスクロックとPTP （PTPがオフ、またはまだロック中）のトリガーは除外され、`Note:`がそれらをカウントします。 フレームが1つでも見逃されるとテストは不合格となります（`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`、`1`で終了）。3回連続で不完全なトリガーが発生すると、テストは早期に終了します。 また、すべてのカメラが 1 クロックで同期しているトリガーが 2 つ未満の場合（`… the clocks cannot be compared`）、またはクロックが決して一致しない場合（ PTPクロックがマスターから1ミリ秒以上（シリアル層のずれ分を含む）ずれている場合（`… PTP is not locked`）、またはトリガーの間にデバイスのクロックオフセットがそれ以上変動する場合。 PTPグランドマスターが実行されていない場合はこれを起動し、すべてのカメラのPTPを有効にし（`array-connect`が実行するように、ネットワークの状態にかかわらず）、最初のトリガーが発生する前に、すべてのカメラのPTPクロックがグランドマスターにロックされ、安定するのを待ちます： `PtpStatus` 1つのグランドマスター上のスレーブが、グランドマスターから100 us以内の誤差で、かつ3秒間安定していること。また、各カメラが電源投入から120秒間（約2分間）——中断することなく——その状態でロックされていること。ロック直後のクロックは、「Slave」と読み取られてから約2分間、最大約200 usのずれが生じますが、カメラ自身の`PtpOffsetFromMaster`にはそれが反映されません。120秒経過すると、カメラのクロックは安定状態のクロックから約10 us以内の誤差に収まります。あるカメラは このバックエンドが認識した限りでは、PTPをオンにしたままの以前のテスト以降もロック状態を維持しており、 そのロック時点からカウントを続けるため、連続したテストでは再度待機することはない。テストで電源が入ったカメラ、またはPTPがオンになっている状態でこのバックエンドがロックを確認していないカメラ（別の場所で電源が入ったもの、またはバックエンドの再起動前に電源が入ったもの）は、120秒待機する。両方のカメラが混在する配列では、ロックしたばかりのカメラは依然として待機を続ける。 スレーブ状態を離脱したカメラ、待機中に100 usを超えたカメラ、または別のグランドマスターを3回連続（1.5秒）追従したカメラは、カウントを再開する。そのような読み取り値が1回または2回記録され （`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`）として記録され、カウントは継続されます。また、読み取れないグランドマスターIDは、変更ではなく「不明」として扱われます。 すべてのカメラのカウントが同時に（1秒以内に）再開始されるのは、カメラ側の時間ではなく、グランドマスター側の時間が動いているためである。バックエンドのログには `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …` と記録され、 `ptp_lock`はこれを`grandmaster_steps`にリストし、その後タイムアウトした待機は`every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`でエラーを開始します。 この処理のために制限時間は延長されません。`--ptp-settle SECONDS`は実行のセトル時間を設定します（0～600；セトル待機なしの場合は`0`）。待機時間は、安定化時間＋90秒（デフォルトは210秒、最低90秒）に制限されており、コマンドはバックエンドに対して240秒＋安定化時間 （デフォルトは360秒）待機します。`PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`が出力され、バックエンドのログには5秒ごとに、各カメラの状態とオフセット、ロックされてからの経過時間、および残り時間が記録されます： `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`。すべてのカメラはPTPをオフにして電源が入ります。 そこから約10秒でクロックがロックされます。その後、各カメラはテストで検出された状態に戻されます — PTP、自動露出、およびテスト実行中にオフに設定されたゲインはそのまま維持され、その他（`--keep-settings`の場合を除く） — したがって、PTPがオンのままのカメラ（`lattice configure -s <serial> --set PtpEnable=True`、または`--keep-settings`が設定された実行）はロックされた状態を維持し、このバックエンドが安定時間だけロック状態を確認すれば、その後のテストではそれを待たずに処理を進めます。 待機中にクロックが安定しない場合、トリガーされる前に失敗となります（`PTP: NOT locked …`、終了 `1`） 、そして対処方法を示します：一度もロックされなかったカメラ（`PtpStatus Listening`）は、グランドマスターがそれに到達できていないことを意味します（`time-sync status`）；ロック状態が頻繁に切断されたカメラは、ロックされていた期間で名前が付けられます （`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`）；クロックがまだ収束中の場合は、`time-sync status`を確認し、 テスト前にロック状態にするには、PTPをオンにしてそのままにしておく（各カメラごとに`lattice configure -s <serial> --set PtpEnable=True`、または`--keep-settings`でテストを実行；テストは検出結果を元に戻すため、PTPはオンのままになる）、その後テストを実行する。 |
| `lattice multi-detect [--line LINE] [--json]` | M8同期ライン上のカメラと、デフォルトのマスターを特定します。RGBカメラ（モデル名の末尾が`-FRGB`）がケーブル接続されている場合はそれ、そうでない場合はシリアル番号順で最初のケーブル接続カメラがマスターとなります。 M8ケーブルは1つの分岐点で合流しているため、**配線済みのカメラならどれでもマスターになる** — 印刷されている「`MASTER`」はデフォルト値であり、配線上の事実ではありません。各カメラが順番にラインを駆動し、他のカメラがそのパルスを検知するかどうかを確認します。配線されたグループ内で既に検出済みのカメラは再度プローブされません。 したがって、完全に配線されたアレイでは1回のスキャンで完了します。`--line`がない場合は、`Line2`をスキャンし、 次に `Line3` をスキャンします。`MASTER: SN:…`、配線済みのカメラ1台につき1つの `SLAVE: SN:…`、および `(probed on <line>)` を出力します。`--json`が完全な結果（`master`、`slaves`、 `no_cable` — パルスが到達しなかったカメラ — `line`、およびカメラごとの `results`）を合計します。。処理が完了すると、各カメラは元のトリガーおよび同期ライン設定（上記と同じノード）に戻ります。 |

#### アライメント

| サブコマンド | 目的 |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — 検出器／マッチャーノブ `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`、 RANSACパラメータ `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`、マルチフレーム結合 `[--averaging mean\|median\|inlier_weighted]`、幾何学的制約 `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`、空間的制約 `[--roi X0,Y0,X1,Y1] [--mask PATH]`、およびスレーブごとの上書き `[--per-cam-override SN:KEY=VALUE]`（繰り返し可能） | ライブカメラからアライメントプロファイルを計算します。デフォルト：`--method orb`、`--model rigid`、 `--frames 3`、`--save alignment_profile.json`（実行元のフォルダ内）、 `--max-features 2000`、`--ratio-threshold 0.75`、`--matcher bf`、`--knn-k 2`、`--ransac-threshold-px 3.0`、 `--ransac-iters 2000`、`--ransac-confidence 0.99`、`--averaging mean`。（`project align calibrate`には独自のデフォルト設定があります。 [その表](#project-align-calibrate-options)を参照のこと。) `--prefilter`のデフォルト設定は`gradient` （エッジマップ；GUI／アレイアライナーと一致 — エッジはスペクトルバンドをまたいで保持される）。`--matcher flann`は、特徴数が約5000以上になると効果を発揮する； `--averaging median`は1つの不良キャプチャに対して頑健であり、`inlier_weighted`は一致数に基づいて重み付けを行い、`--lock-scale`は最も近い回転（スケールなし）に投影し、 `--lock-axis`は1つの並進成分をゼロに設定します； `--mask`はすべてのカメラに適用されます（カメラごとの設定には`--per-cam-override`を使用してください。例：`--per-cam-override 214701292:method=phase`）。 `--rms-threshold-px`は、再投影のRMSが許容範囲を超えるキャリブレーションの保存を拒否します。`--method manual`は、`--points PATH`からポイントを取得します： JSONオブジェクトである`{"<serial>": [[x, y], ...], ...}`から、各カメラにおける同一特徴点のピクセル座標（基準カメラを含む、カメラあたり少なくとも2つ）を取得します。`--frames`の各項目は、これらの点から算出されます。 接続された配列にカメラが含まれている場合、キャリブレーションはその配列のライブフレームを使用し、プロファイルもその配列のキャプチャに対するアライメントとなります。そうでない場合は、[一時配列](#multi-camera-transient-sessions)を開きます： `--frames`のいずれにおいてもフレームを送信しないカメラはキャリブレーションに失敗し（`1`を終了し、その名前を指定）、プロファイルは保存されません。 |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | アライメント済みのマルチバンドフレームを1枚キャプチャします。`--bit-depth`はデフォルトでカメラに合わせます； `--no-crop`はフルフレームを維持します（黒でパディング）；`--interpolation`（デフォルトは`linear`）および`--border-mode`/`--border-value`（デフォルトは `constant`/0）は CPU ワープを制御します。いずれの場合も GPU パスは双線形補正となります。[一時配列](#multi-camera-transient-sessions) を開きます。フレームを送信しないカメラはキャプチャに失敗し（`1` をその名前で終了）、何も書き込まれません。 |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | ストリームアラインされたマルチバンドフレーム（`align-apply`と同じワープ設定）、`--fps` デフォルト、`3`。[一時配列](#multi-camera-transient-sessions) を開きます：カメラから何も送信されなかったフレームグループは保存されず、3回連続して何も送信されないとストリームが終了します。 いずれかのグループがスキップされた場合、実行は `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` および `Error: …` を出力して終了し、 その後、`1`で終了します。その他のエラーで終了するストリームも、`Error: …`を出力し、`1`で終了します。 |
| `lattice align-info --profile PATH [--json]` | プロファイルの詳細を表示します。 |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | レイヤーの順序を変更します。 |

#### 索引 / 植生計算

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel`は、プリセットのシンボルをバンドにマッピングします** （NDVI：`red`、`nir`）、それぞれ1つの`--channel SYMBOL=BAND`。

- **`--live`: スタックが対応できないシンボルにのみ適用されます。** `--input` と同様に、プリセットのシンボルを省略した場合、そのシンボルの名前で始まるスタック内の 1 つのバンド（プロファイルで有効になっているカメラのバンド）が割り当てられます。 これは、カメラが開く前にチェックされます。どのバンドも、あるいは複数のバンドも対応していないシンボルは拒否され、それらの名前を指定して `1` を終了します。 カスタム `--formula` には、シンボルごとに `--channel` が必要です。
- **`--live` 製品は、それがどの製品に属するかを示します。** `index.tif` および `index_LUT.png` は、スタックに含まれるカメラ（メーカー、モデル、シリアル番号）、その撮影時刻（参照カメラの露光時間、UTC） およびレンズ情報を保持しており、両者とも、`--input` 製品と同様に、インデックスを単一のバンドとして指定しています（`XMP-Camera:BandName`、`NDVI`；カラー PNG ファイルでは、色チャンネルではなくインデックスが名前として指定されます）。 一方、複数のカメラのバンドからなるインデックスは、それぞれのバンド名を個別に指定し、その `Model` は `LATTICE` となります — [複数のカメラから作成されたファイル](#files-made-from-several-cameras)を参照してください。
- **`--input`: ファイルが回答できないシンボルの場合にのみ使用します。** 省略したシンボルについては、ファイル内の名前がそのシンボルで始まるバンドが採用されます： `red`は`Red_660`または`Red_660_RGN`を、`nir`は`NIR_850`を採用します。 これらの名前はファイル自体から取得されます。つまり、スタック自身のバンドリスト、あるいは Chlorosの放射輝度／反射率キャプチャ、あるいは`process`プロダクトの、その`Chloros:Band{R,G,B}Name`／`XMP-Camera:BandName`タグから取得されます。したがって、`lattice index --input <reflectance capture> --preset NDVI`には `--channel`を必要としません。
- **1つのシンボルに対応する2つのバンドは、その間で推測されません**（9台のカメラによるスタックには2つのNIRバンドがあります）： そのバンドに名前を付けます。どのシンボルにも対応しないものは `channel_map missing entries` で拒否され、`1` で終了します。**スタック内のバンド名**にはカメラのフィルター情報が含まれます：M3Cカメラのバンドは、`<band>_<filter>` — `NIR_850_RGN`、`Green_550_RGN`、 RGNでは`Red_660_RGN`、`NIR_808_OCN`、`Cyan_490_OCN`、 OCNでは`Orange_615_OCN`；RGBでは`Blue_475_RGB`、`Green_550_RGB`、`Red_625_RGB`；NGBでは`Blue_475_NGB`、`Green_550_NGB`、`NIR_850_NGB`はNGB上で — そしてM3Mカメラののシングルバンドは `Mono_<filter>` です（`Mono_660`、`Mono_850`）。`lattice align-info --profile PATH` は、プロファイルのバンド一覧を示しています。

完全なフラグセット：`--input PATH | --live --profile PATH`、`--preset NAME`（NDVI／NDRE／EVI／SAVI／GNDVI／…）、 `--formula EXPR`、`--channel SYM=BAND`（繰り返し可能）、`--capture-level raw|debayered|radiance|reflectance|unknown`（ソースTIFFに記録されたキャプチャレベルを上書き；デフォルト： TIFFメタデータから読み込み、Chlorosファイルの`Chloros:ProcessingLevel`にフォールバックするため、反射率キャプチャでは`reflectance`が報告される）、`--output PATH` / `-o`（オフライン時：パス。画像拡張子がある場合、ファイル名はそのステム — `-o ndvi_FRGN.png` は `ndvi_FRGN.tif` および `ndvi_FRGN_LUT.png` を書き込みます — 拡張子がない場合は接頭辞となり、`-o ndvi` → `ndvi.tif`、`ndvi_LUT.png`。入力の横にある場合はデフォルト値として、 `<input name>_<preset>`のように；`--live`があり、`-o`がない場合は、コマンドを実行したフォルダの下に`output/`フォルダが作成されます）、 `--output-format all|raw|tif|colorized|lut|png`、`--gradient NAME|JSON`、`--vmin/--vmax/--percentile LO,HI`、 `--bg-mode clip|transparent|indexColor|backgroundColor`、`--colorize`、`--list-presets`、 `--list-gradients`。`--live` では、アライメント・ワープ・ノブも適用されます：`--save-multiband`、`--gpu/--no-gpu`、`--no-crop`、 `--bit-depth 8|12|16`、`--vignette`、`--interpolation nearest|linear|cubic|lanczos`、`--border-mode constant|replicate|reflect|wrap`、`--border-value N`、 `--brightness N`（カメラの自動露出ターゲット。一時的なルール参照）、`--serials SN1,SN2,…`。`--live`は[一時配列]（#multi-camera-transient-sessions）を開く：フレームを送信しないカメラは実行に失敗する（`1`をその名前で終了する）、スタックやインデックスは書き込まれません。`--serials`を開きます（最初のものがトリガーを発火させます）。そうでない場合は、プロファイル内で`layer_order`エントリが有効になっているカメラ（参照カメラを最優先）— 他のカメラはこれにアラインされているため、バンドが無効になっていても参照カメラはトリガーを発火させます それに合わせられており、トリガーを発火させるためです。バンドが無効化されているカメラは決してキャリブレーションされないため、プロファイルで無効化されたRGBカメラは、もはや反射率インデックス（`no-absolute-radiance-scale`）の失敗を引き起こしません。カメラを指定していないプロファイルは、以前と同様にネットワーク上のすべてのカメラを開きます。

すべてのフォームはバックエンド上で実行されるため、バックエンドが稼働している必要があります。 Chloros GUI から起動するか、CLI がバックエンドに接続できない場合に表示するコマンドを使用して起動してください。オフライン時、バックエンドは `--input` を読み込み、浮動小数点形式の TIFF および色付けされた PNG 自体を書き込みます。そのため、相対パスである `--input` や `-o`は、コマンドを実行したフォルダを基準に解決されます（[規約](#conventions)を参照）。 どちらのオフライン生成物も、`process`インデックスエクスポートと同様に、入力のカメラ記録を含みます：メーカー／モデル／シリアル番号、GPS、レンズタグ、撮影時刻（入力データ自身のもの。EXIFに「なし」と記載されている場合は、記録された露出瞬間またはファイル名のタイムスタンプ、 UTC）およびMAPIRキャリブレーションID（インデックスには適用されないraw-DNキャリブレーションは含まれません）に加え、`Chloros:ProcessingLevel=index`およびインデックス式を含みます。浮動小数点数のTIFFは、1つのバンド、つまりプリセットを指定します。 `--list-presets` および `--list-gradients` は、バックエンドのプリセットおよびグラデーションテーブルを出力します。 バックエンドが拒否した式、またはTIFFに存在しない `--channel` バンドがある場合、バックエンドの理由を伴って `1` を終了します。

> **`--channel` 記号は大文字と小文字が区別されます。** シンボル側はプリセットのチャンネル名と完全に一致している必要があります（プリセットでは小文字が使用されます。例：NDVI = `red`、`nir` — `--list-presets` を確認してください）。また、バンド側はアラインされたスタック内のバンド名と一致している必要があります（オフラインモードの場合は、0を基点とするバンドインデックスでも可）。`--channel red=Red_660_RGN --channel nir=NIR_850_RGN`は正常に動作しますが、`--channel RED=660`は`channel_map missing entries` エラーが発生します。

#### `lattice index` — マルチバンド・スタック・マスク

`--capture-level radiance` または `reflectance` において、各カメラの 3×3 アンミックスは、バンドを 0 にクリップすることができます （[アンミックス・クランプ・マスク](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)を参照）。マルチバンド・スタックには複数のカメラのバンドが含まれており、各カメラが自身のバンドをクリップするため、`--live --save-multiband`はスタックの横に**ソースカメラごとに1つのマスク**を書き込みます：

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- 各マスクは、**スタック**のピクセルグリッド上（同じ位置合わせワープおよびクロップを経た）のシングルバンド `uint8` であり、そのカメラ固有のビットを持ちます：ビット 0 = そのカメラの R チャンネルのバンド、ビット 1 = G、 ビット2 = B。カメラのチャンネル*c*から取得されたスタックバンドは、そのカメラのビット*c*（R = 0、G = 1、B = 2）が設定されている箇所でクリップされます。
- 各バンドに埋め込まれたプロヴェナンスには、そのカメラのマスク名 （`unmix_clamp_mask`、スタックのフォルダを基準とする、または `none`）と、読み取るべきビット（`unmix_clamp_mask_bit`）を指定します。
- アンミックス処理でクリッピングが発生しなかったカメラにはファイルが生成されません。モノクロまたはRGBカメラはアンミックス処理を実行しないため、同様にファイルは生成されません。CLIはスタックの下に各マスク（`clamp mask SN…: …`）を出力し、応答ではそれらを`clamp_masks`として一覧表示します。
- 同じ実行のインデックス TIFF / PNG は、同じスタックから、同じグリッド上で計算されるため、マスクはこれらも記述します。`--capture-level raw` はアンミックスを実行せず、マスクも書き込みません。
- スタックとインデックスは `--output` に直接格納されるため、マスクは `--output` 内の `clamp_masks/` に格納されます。指定したフォルダの上位には決して配置されません。 このフォルダには、スタック、インデックス、カラー化レンダリングなど、異なる種類のデータが混在しています。フォルダ全体をフォトグラメトリーソフトウェアに渡すのではなく、必要なファイルのみを選択してください。

#### 永続的な接続（Smart-Prep、GUIと同等のフロー）

これらのコマンドは、CLIの呼び出しをまたがって、バックエンドプール内のカメラを開いたままにします。

| サブコマンド | 目的 |
| --- | --- |
| `lattice cam-connect [--serial SN]` | プールにカメラを 1 台追加します（単一カメラ、配列なし）。[共通オプション](#common-options-shared-by-most-subcommands) `--exposure`、`--gain`、`--preset`、 `--pixel-format`、`--width` / `--height` および `--packet-size`（`standard` = 1500、`jumbo` = 9000、またはバイト単位；`auto`プローブ）は、接続時に適用される （`--exposure` または `--gain` は自動モードをオフにします）。 カメラのストリームが開始されると（その開始まで約15秒、最初のバッファが完全に満たされるまでさらに10秒待機し）、返却され、ストリームが何も送信されなかった場合は `stream: streaming` または `stream: no_frames -- connected, no frames yet: …` を出力します — ジャンボパケットを使用しているカメラの場合、`--packet-size standard`で再接続するようアドバイスが表示されます。あるカメラが接続している間（初回接続ではキャリブレーションデータのダウンロードに約70秒かかります）、カメラを必要とする別のコマンドは最大5秒間待機し、 その後、接続中のカメラ名を指定したHTTP 503エラー（`CAMERA_POOL_BUSY`）で失敗します。しばらくしてから再実行してください。`lattice status`およびカメラリストは待機しません。接続中のカメラは、そのフェーズとともに「接続中」と表示されます。 |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | 解放。 カメラは、接続時に検出された状態（トリガー配線、PTP、設定）に戻されます。`--keep-settings`は設定を自身のメモリに保存し、`--clear-settings`はそのメモリを工場出荷時の設定にリセットします （[カメラメモリ](#camera-memory----keep-settings----clear-settings)を参照）。アレイのメンバーであるカメラは、アレイ全体を解放します。 |
| `lattice cam-list` | プール内のカメラを一覧表示します。 |
| **`lattice array-connect`**|**永続的な同期アレイを接続します（推奨されるエントリポイント）。** GUI によるスマートプレップフロー全体を実行します。カメラ 9 台の場合、約 1 分かかります。 カメラの電源投入直後は数分かかる場合があります。待機中は、バックエンドが到達した各ステップが `    <s> s: <step>`（例：`    41 s: open SN:214000535 (4 of 9)`）として出力され、 また、バックエンドのログにはステップごとに1行のタイムスタンプ（`[array-connect <id>] <step>: <s> s …`）と、最も時間がかかったステップを示す最終行が記録されます。 待機時間は最大600秒です。その時点で接続が完了していない場合、バックエンドによって接続は解除されます。バックエンドが開いていたすべてのカメラは、検出時の状態に戻され、閉じられます。 コマンドは到達したステップを含む「`Error:`」を出力して終了し（`1`）、アレイは接続されません（HTTP 504 `CLIENT_DEADLINE_PASSED`）。 Ctrl+C でも同様に接続がキャンセルされます（`Interrupted: …`）。アレイがすでに接続されていた場合、その旨がメッセージに表示され、接続を解放する `array-disconnect` の名前が示されます。待機中に強制終了されたコマンド (`kill -9`) により、バックエンドから30秒間応答がない場合、接続は解除されないままとなります。起動しているが、まだ同期されたフレームグループを持たないアレイも、接続済みとして報告され、そのストリームは `no_frames` として表示されます。 |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | アレイを解放する。 すべてのカメラはフリーランニング状態（`TriggerMode` `Off`）のままとなり、それ以外の設定（トリガーおよび同期ラインの設定（マスターの`TriggerSource`は元の状態に戻り、その同期ラインは入力に戻ります）、PTP、およびセッションで変更されたカメラ設定はすべて維持されます。これにより、メンバーに対する単一カメラコマンドが即座に機能します。`--keep-settings`は、各カメラの設定を、セッション終了時の状態のままカメラ自身のメモリに保存します。`--clear-settings`は、そのメモリを工場出荷時の設定にリセットします。[カメラのメモリ](#camera-memory----keep-settings----clear-settings)を参照してください。 |
| `lattice array-list` | 接続されたアレイを一覧表示します。 |
| `lattice array-status [--array-id ID]` | ライブfps、PTP、 直近のエラー。`ptp_enabled`は、すべてのメンバーがPTPを「オン」と読み取っている場合にのみ`True`となります。`ptp_off:`は、PTPを「オン」と読み取っていないメンバーの名前を示します。設定の再起動中に`state:`行は、設定の再起動中に`restarting (applying settings)`を読み取り、電源が切れたメンバーが再起動される際に`recovering (…)`を読み取り（以下の注を参照）、 `stalled (…)`は、グラブループが実行されているがフレームグループを公開していない間に表示され、`stopped (…)`は、グラブループが処理を断念した後に表示されます（アレイを再接続してください）。 |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | ライブアレイからの同期キャプチャ 1 回 — シングル / 連続 / インターバル / 最速。**デフォルトは `all`**（カメラごとに該当するエクスポート形式ごとに1ファイル）、アプリのキャプチャと同様に**アラインメント済み**（ネイティブファイルの場合は `--no-aligned`）。スキップされたカメラ（例： RGBは放射輝度／反射率から除外）は `Skipped: SN:<serial> (<reason>)` で報告されます。使用されたキャプチャを読み込んだ DAQ の読み取り値は併せて保存され、RAW データのみのキャプチャが含まれ、`DAQ: <path>` で報告されます（[キャプチャを読み込んだ DAQ が保存するデータ](#the-daq-reading-a-capture-saves))を参照してください。[キャプチャモード、レコーダー、およびオフライン再処理](#capture-modes-recorders--offline-reprocess)を参照してください。 |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | ライブの複合インデックスビューを動画/GIFとして記録します（モニタリンググレード；GUIビューが開いていない場合、バックエンドは配列のNDVIをレンダリングします、または`--expr`）。 |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | 高フレームレートの生ベイヤーバースト （分析用；オフラインで再処理）。`array-record`および`array-capture --continuous`と同様に、各ポーリングでバックエンドにチェックインを行う：停止せずに終了（`kill -9`）、バックエンドは最後のチェックインから30秒後にこれを停止させる（`stop_reason`、`client_gone`；停止時と同様に、バーストのマニフェストが確定され、結合インデックスの構築が開始される）が、アレイは接続されたままとなる。 |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | 保存された生のバーストを、キャリブレーション済みの動画に再処理する。 |

> **電源やリンクを失ったカメラは、自動的に復旧する。** メンバーが応答しなくなった場合（スイッチの電源再投入により、電源を供給されているすべてのカメラが再起動される）、アレイはそのメンバーを待機し、再び応答し始めたら接続を再開し、 `array-connect`で設定された内容（PTP、トリガーおよび同期ライン、ネットワークペーシング、ピクセルフォーマット、フレームサイズ、自動露出）をそのカメラに復元します。その後、新たな`array-connect`を設定することなく、再びストリーミングを開始します。 その間、`array-status`には`state: recovering (...)`が表示され、`last_error`には待機中のカメラが示されます。その後、`re-opened:`が出力されます。 再起動したカメラはPTPがオフの状態で復帰するため、再オープンされるまでは、`ptp_enabled`は`False`となり、`ptp_off:`がそのカメラ名を指定します。

> **保留中のカメラは、二度と開かれることはありません。** 2 度目に開くと、保持者のストリームが停止し、設定が上書きされ、バックエンドがクラッシュする可能性があります。 したがって、プールが保持しているカメラ（GUI または `cam-connect`）、配列の要素、またはバックグラウンドジョブが保持しているカメラ（実行中の `lattice continuous`、 `multi-stream` または `align-stream`、自身のものか他のクライアントのものかを問わず）、`lattice continuous`、 `lattice capture`、`cam-connect`、`power --sleep`、`calibrate`、1ショットである`white-balance`および`presets --apply`は、HTTP 409で拒否し、`CAMERA_HELD`は、所有者と解放コマンドを指定し （`chloros-cli lattice cam-disconnect -s <SN>`、`chloros-cli lattice array-disconnect --array-id <ID>`、またはジョブの場合はそのIDと開始者）を指定して、終了する `1`。 また、別のコマンドが現在使用しているカメラ（`multi-test` または `multi-capture` がそのカメラを開いている場合、別のワンショット）についても同様です。ここにあるすべてのコマンドについて、エラーメッセージは、そのコマンドが終了するまで待つよう指示しています。 `lattice power` は、`--sleep` なしで、ホルダー自身のハンドル（プール、配列、ジョブの開いているカメラ）を読み取り、別のコマンドが使用中のカメラについては拒否されます。 すべての `CAMERA_HELD` 応答には、`code` および `error_code` の両方にコードが含まれています。すべてのカメラを開くコマンドについては、[マルチカメラ（一時セッション）](#multi-camera-transient-sessions)を参照してください。

##### `array-connect` オプション

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--serials SN1,SN2,…` | すべての LATTICE カメラを自動検出（2台以上必要） | 最初のシリアルが MASTER となります。省略した場合、検出は LATTICE (M3C / M3M) モデルに絞り込まれ、それらすべてが接続されます。 |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO同期ライン。 |
| `--target-fps F` | 自動 | マスタートリガーの発射レート。 |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | 自動 | ティアピッカーを上書きします。 |
| `--wire-ceiling-mbps MB_PER_S` | 自動検出 | **ホストの持続的なワイヤ帯域幅（MB/s） — アレイ全体の割り当てがこの数値に依存します。** アレイがGVSP破損フレームを報告した場合は、この値を下げてください。自動設定値はNICが通知するリンクレートに基づいて算出されますが、USBアダプタ、帯域の狭いPCIeレーン、および負荷の高い共有ファブリックでは過大評価される傾向があります。プロジェクトのアレイキャプチャブロックに永続化されるため、再オープン／CLI／SDKによる再接続で復元されます。 [アレイの健全性](#array-health--which-subsystem-is-losing-frames)を参照してください。 |
| `--binning {1,2,4}` | なし（フルセンサー） | ハードウェア・ビニング（常に平均）。 |
| `--width N` / `--height N` / `--pixel-format FMT` | フルセンサー；カメラごとのフォーマット | フルセンサーおよびカメラごとのフォーマット選択（Mono12 / BayerRG12）の代わりに、すべてのカメラに対して明示的なフレームサイズまたはピクセルフォーマットを指定します。 |
| `--no-recommend` | オフ | ネットワーク解析ステップをスキップ （これにより、各カメラはフルセンサーに設定される代わりに、現在のサイズを維持します）。 |
| `--no-ptp` | オフ | PTPを無効化（これにより、カメラ間のタイムスタンプは**比較できなくなります**）。 |
| `--packed {auto,on,off}` | `auto` | 12ビットフレームをパックしてストリームし、このコンピュータでアンパックする — フルセンサー時でも1秒あたりのフレーム数が増加し、データ内容は同一。`auto`は、 処理に有効で、かつこのコンピュータが処理能力に余裕がある場合にのみパックを行います； `off`は決してパックを行いません。[パックされた12ビット転送](#packed-12-bit-transport---packed)を参照してください。 |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | アレイ内の各カメラに対するGVSPパケットサイズ。 `auto`は各カメラにネットワークのプローブを許可します。`jumbo` = 9000、`standard` = 1500、またはバイト単位のサイズを指定すると、その値に固定されます。 バージョン 1.3.0 以前では、`array-connect` はこのフラグを受け付けましたが、無視していました。 |

> **デフォルトではフルセンサー。** `--width` / `--height` / `--binning` /
> `--pixel-format`、`array-connect`が指定されていない場合、すべてのカメラはフルセンサー設定になります —
> ビニングなし、 ROIクロップなし——となり、フレームレートを維持するためにフレームサイズを縮小するのではなく、このコンピュータと
> ネットワークが提供できる範囲までフレームレートが低下するよう設定されます。
> ネットワーク分析行には予測フレームレートが表示され、次の行には
> より高速なオプション（ビニングまたはより小さなROI）とその予測フレームレートがリストされています。
> `--binning` または `--width`/`--height` を実行して、いずれかを選択してください。Connect は、
> 独自にビニングやクロップを行うことはありません。

##### パックされた12ビット転送（`--packed`）

センサーのフル解像度では、アレイのフレームレートは各カメラが
フレームを送信できる速度によって決まります。 12ビット形式（`BayerRG12`、`Mono12`）は通常、16ビット
ワードとして転送されますが、**パック**転送では、同じ12ビットを1ピクセルあたり1.5バイトで送信し、
このコンピュータ上で展開するため、各フレームの送信時間が25％短縮され、
**フルセンサー時で1秒あたりのフレーム数が約33％増加**します（測定値：9台のアレイで10.7 → 14.0 fps
）。 **保存されるデータは同一**です：値も、
16ビットのファイル形式も、メタデータもすべて同じです。

- `auto`（デフォルト）は、**メリットがあり**、かつこのコンピュータが
  アンパック処理に余裕がある場合にのみパック処理を行います。 接続時、バックエンドは自身の解凍速度、CPUコア数、
  現在のCPU負荷、およびネットワーク帯域幅を測定するため、余剰コアがあるデスクトップPCでは
  フルセンサーで圧縮して送信し、負荷が高いまたは低電力のコンピュータでは解凍済みの状態でストリーミングします。ROIが小さく、フレームがビンニングされている場合は
  ほとんど圧縮されません。
- `off`は決して圧縮を行いません。診断中に圧縮を無効にする場合や、
  他の作業のためにCPUリソースをすべて確保する必要があるコンピュータで使用してください。
- `on`は常に12ビット形式でパックします。

connectの出力には、その決定と理由が表示されます（`packed 12-bit: on (auto:
予測値 11.0 → 14.4 fps, …)`); `/api/camera/array/status`は
`packed_transport`および`packed_decision`を報告します。毎回フラグを渡す代わりにマシン全体のデフォルトを設定するには、バックエンドの環境で`CHLOROS_ARRAY_PACKED12=off` （または
`on`）をバックエンドの環境変数に設定してください。ただし、明示的に指定された `--packed on|off` が優先されます。
8ビットおよび16ビット形式は、決してパックされません。

### 1つのプロジェクトレコードで全モードに対応

1つのプロジェクトフォルダには、すべてのデバイスの設定が格納されます：`cameras.json` — 各カメラの設定（`cameras[].settings`、 単一のカメラ用および各アレイ構成員用）と、各アレイ全体に適用される設定（`arrays[].capture`：フォーマット、ROI、ビニング、トリガーレート、ワイヤ予算）——ならびに`sensors.json`、 各光センサーの設定（積分時間、フレーム平均、上限値）。アプリ、CLI、SDK、およびMCPサーバーはすべて、同じファイルを同じ方法で読み書きします：

- **プロジェクトに接続されたデバイスは、そのプロジェクト内に保存され、すべての変更がそこに記録されます。** 「プロジェクトの下」とは、バックエンド（アプリ側）でプロジェクトが開かれている状態、あるいは — SDK からは — 独自のフォルダ名を指定する `ChlorosProject` を通じて接続されている状態を意味します。プロジェクトにまだ登録されていないカメラ、アレイ、または光センサーは、接続されるとそのファイルに追加されます （`lattice cam-connect`、`array-connect`、`daq pool-connect`、アプリ、SDK、`chloros-mcp`）、その設定とともに。 いずれかのモード（`lattice exposure`、`gain`、`configure --set`、`resolution`、アレイの設定やトリガーレート、光センサーのキャップ、アプリのコントロールなど）の変更は、その直後にそのプロジェクトのファイルに保存されます。 カメラで設定された露出、ゲイン、またはAEターゲットは「保持」状態として保存されるため、プロジェクトを再度開くとその設定が再び適用されます。
- **プロジェクトを開くと、すべてのモードにおいてそのプロジェクトの設定がデバイスに適用されます**：独自の設定がない状態で接続されたカメラ、アレイ、または光センサーには、プロジェクトに保存された設定が適用されます （プロジェクトを開いている状態の `lattice cam-connect` / `array-connect`、`chloros-cli project connect`、`open_project().connect_all()`、アプリの再起動時）。 新しいIDで接続されたアレイは、プロジェクトに保存されている同じカメラを持つアレイを更新します。異なるカメラを持つアレイは、その横に追加されます。
- **アプリ独自の保存処理は、デバイスのライブ設定を**古いコピーで上書きすることは決してなく、CLIやSDKの呼び出し元によって保存されたメンバーごとの上書き設定を維持します。
- **設定したピクセル形式は、プロジェクトに保持されます。** 「配列設定」で設定された形式、`--pixel-format`または`connect_array(pixel_format=)`によって設定された形式（8ビットを含む）は、正確に保存され、再接続時にもそのまま反映されます。 何も設定されていない場合、各カメラは 12 ビットのデフォルト設定（モノクロセンサーでは `BayerRG12`、`Mono12`）で接続されます。フォーマット名はビット深度を表します。ファミリーはセンサーのものであるため、カラーカメラがモノクロフォーマットに設定されたり、モノクロカメラがベイヤーフォーマットに設定されたりすることは決してありません。フォーマットがマッピングされた際には、行 `[PixelFormat <serial>] ... written` にその旨が記載されます。8ビットキャプチャのエクスポートデータは、反射率データを含め8ビットとなります。これらは再スケーリングされ、`PixelScale`という識別子が付与されます。
- プロジェクトに紐付けられていないデバイスは、何も記録しません。
- **記録は、その後に作成されたプロジェクトに従います。デバイスを開くと、そのプロジェクトが保持されます。**プロジェクトが作成されると（SDKの`create_project()`、`chloros-mcp`の`create_project`、またはアプリの**[新規プロジェクト]**— または複製（アプリの**「複製」**機能。これによりコピーが開かれます）が行われた際、その時点でまだ以前開いていたプロジェクトへの書き込みが進行中であれば、記録は継続され、書き込まれた内容は新しいプロジェクト内の同じ場所（下部）に保存されます。既存のプロジェクトを開く場合 — `chloros-cli process`、`project devices --reconnect`、 `ChlorosProject.process()`、アプリの**プロジェクトを開く**）場合、録画は元のプロジェクトへの書き込みを継続します。録画はそこで完了し、そのファイルは当該プロジェクトのファイル一覧に追加されます（アプリはプロジェクトを開く前に、すべての録画を停止します）。
  - `array-record` および `array-burst`：録画が停止すると、動画、または全フレームを含むバーストフォルダが丸ごとそこに移動します（その `--duration`、 Ctrl+C）で停止すると、動画または全フレームを含むバーストフォルダが丸ごとそこに移動します。出力される`Burst dir`が新しいファイルであり、`--build`および`array-build-video`はこれを基に動作します。ドライブが移動を拒否した場合、録画データは書き込まれた場所に残ります。終了時の末尾のログにその理由が記載されており、停止時のHTTP応答（`relocation_error`）にも同様の情報が含まれます。停止されたバーストの生フレームと`.daq`は、そのバーストが最終的に属することになるプロジェクトのファイルリストに追加されます。 動画（`array-record`、バーストから生成された動画）は、プロジェクトのファイルリストには追加されません。
  - `array-capture --continuous`：変更後のすべてのキャプチャは新しいプロジェクトに保存されます。変更前に保存されたものは元の場所に残ります。
  - `daq pool-record` および `project sensor log`（およびアプリ独自の光センサー録画）：録画が停止すると、`.daq` は新しいプロジェクトの`light_sensor`フォルダに移動し、そのプロジェクトのファイル一覧に追加されます。

  開いているプロジェクト外のフォルダ（別の場所にある `-o` など）への録画 は元の場所に残り、`lattice continuous`および`multi-stream`は指定されたフォルダのままとなります。
- **アプリ内では、新しいプロジェクト作成後のキャプチャはそのプロジェクトに保存されます。** プロジェクトの作成または複製を行っても、すべてのキャプチャ実行および録画は継続され、それ以降に行われた各キャプチャ（「すべてキャプチャ」、「キャプチャ開始」）は新しいプロジェクトに保存され、そのファイル一覧に追加されます。アプリ内で既存のプロジェクトを開くと、まずそれらすべてが停止します。
- **プロジェクトは、録画中は移動されません。**開いているプロジェクトを別の作業ディレクトリ（アプリの**設定 → 作業ディレクトリ**； CLIコマンドではプロジェクトを移動できません。`/api/move-project`にPOSTリクエストを送信するHTTPクライアントのみが移動可能です）は、アレイ記録やバースト、アレイ連続キャプチャ、 `lattice continuous` / `multi-stream`ジョブ、または光センサー記録がそのプロジェクトに書き込みを行っている間は、HTTP 409 `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`で拒否されます。アプリも同様に、 フォルダを要求する前に、アプリ独自の「すべてキャプチャ」、「キャプチャ開始」、または光センサーによる録画が実行されている間は、同様に拒否されます。

これは、切断時にカメラがどのような状態になるかとは別問題です（[カメラのメモリ](#camera-memory----keep-settings----clear-settings)を参照）：プロジェクトは、開く際にカメラの設定を独自に行います。

### カメラのメモリ — `--keep-settings` / `--clear-settings`

アレイが接続されると、そのカメラはアレイの設定を引き継ぎ、その後、設定を変更することができます。Chlorosがカメラを解放するとき — `array-disconnect`、一時的なコマンドの終了時 （配列を伴わない `multi-capture`、`multi-stream`、`multi-test`、`align-calibrate`、 `align-apply`、`align-stream`、`index --live`）、 `lattice continuous`の停止、`lattice capture`の終了（カメラが保持していないもの）—— 接続前に検出された状態に戻します： トリガーおよび同期ラインの配線、PTPおよびカメラの設定（完全なリストは[マルチカメラ（一時セッション）](#multi-camera-transient-sessions)に記載されています）。

プロジェクト外でカメラを使用する場合、これらのコマンドのいずれかに2つのフラグを指定することで、カメラの**内蔵メモリ**（どのソフトウェアで起動した際の状態でも）に書き込むことができます：

- **`--keep-settings`** は、セッション終了時の各カメラの設定を、そのカメラのメモリ（UserSet2、起動時の設定）に保存します。 カメラは引き続きフリーランニング状態（`TriggerMode` `Off`）のままとなり、同期ラインの配線も現状のまま維持されます。UserSet2がないカメラはUserSet1に保存されます。このコンピュータはそれを記憶しているため、Chlorosのアレイの自己修復機能 — これにより、Chlorosのベースラインから逸脱したUserSet1がリセットされますが、それ以外はそのままにされます。
- **`--clear-settings`**は、各カメラのメモリを工場出荷時の設定（Chlorosのベースライン、つまりアレイの自己修復機能によって定着された — `TriggerMode`、`Off`、`TriggerSource`、`Line2` による工場出荷時のデフォルト設定、 自動露出、RGBカメラのキャリブレーション済みホワイトバランスは保持されます — UserSet1に保存され、ブートセットとして設定されます。
- これら2つを組み合わせることはできません。 各カメラに1行ずつ割り当てられます：`SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` または `SN:…: camera memory reset to the factory settings.`。応答の `camera_memory` も、カメラごとに同じ形式になります。

このアプリではこれらのフラグは提供されていません。

**どの終了経路でも、カメラは元の状態に戻されます。**上記のコマンドだけでなく、各終了経路で、発見時の状態への復元処理が実行されます：`cam-disconnect`（およびアプリのカメラ**X**）； 配列上または任意の配列要素の行にあるアプリの**X**（配列は要素なしでは実行できないため、配列全体が解放される）；アプリを終了する際（バックエンドに対し、すべてのカメラ、 `lattice continuous`をすべて戻すための猶予を最大20秒与えます。その時点でまだ実行中のコマンドがある場合 — カメラを開いたりPTPクロックを待機している一時的なコマンド、ワンショットの`lattice capture`、 `power`、`white-balance`、または`presets --apply`など）は、次のセーフポイントで停止し、最大8秒間かけてまずカメラを元の状態に戻し、 HTTP 503 `SHUTTING_DOWN`を返す。一方、まだ実行中の接続配列 — `lattice array-connect`（アプリの接続） — は、次のカメラに到達する前に停止し、開いていたカメラを最大10秒間元に戻し、 HTTP 503 `SHUTTING_DOWN`を返す；また、シャットダウン中に到着した接続は何も開かず：HTTP 503 `SHUTTING_DOWN`）；さらに、強制終了されたりターミナルが閉じられたりしたCLIコマンド（SIGTERM / SIGHUPによりジョブが停止し — カメラを元の状態に戻してから — 終了します（`143` / `129`、Ctrl+Cと同様）。 `lattice continuous`、`multi-stream`、または`align-stream`が即座に強制終了（SIGKILL）された場合、何も停止させることができませんが、最後のチェックインから30秒後にバックエンドによってジョブが停止され、カメラが元の状態に戻されます 。クラッシュしたり強制終了されたりしたバックエンドは、何も復元できません。電源の再投入により各カメラはメモリの状態に戻り、次のChloros接続時に、準備完了状態のカメラは再配線されます。**プロジェクトはカメラの設定を自身で行います。** プロジェクトが開かれている状態でアレイが接続されると、各メンバーはプロジェクトが保存した設定を受け取ります。これには、そのカメラに設定された露出、ゲイン、またはAEターゲット（「自身の設定」として保持されるため、自動露出はこれを変更しません）およびホワイトバランスモード（Chlorosによる補正；カメラ自身のホワイトバランスは無効のまま）が含まれます。これらは、カメラのメモリに保存されている内容にかかわらず適用されます。

### スマートAE / スマートキャプチャ

LATTICEアレイは接続されるとすぐにバックグラウンドで連続AEを実行しますが、新たに撮影対象を指定したシーンでは、収束するまで少し時間がかかります。 `array-capture --smart`は、**利便性をパッケージ化した**機能です。アレイ内のすべてのカメラでAEが安定するのを待ち、その後キャプチャをトリガーします。セッション中にシーンを変更する際に使用してください。

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

デフォルトの安定化ポリシーは保守的です：タイムアウト5秒、安定性ウィンドウ1.5秒、露出ばらつきの許容範囲±5％。自動化とは異なる動作が必要な場合は、SDK（`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`）を介して調整してください。

#### 自動露出の動作

- **露出を優先し、ゲインは後回し。** ゲインが「自動」設定の場合、露出が許容される最長値に達するまで、ゲインは 0 dB のまま維持されます。
- **手動露出設定時でも、ゲインを変更するか手動に設定するまでは、ゲインは「自動」のままです。** 再び「自動」に戻すと、ゲインの上限は以前の状態に戻ります。
- **わずかなハイライトはシーンを暗くしません。** フレームの 1 % までのスペキュラー光は無視され、既知のキャリブレーションターゲットのパネル内では、パネルの 0.5 % までが対象外となります。
- **DAQ の露出上限は、キャリブレーションターゲットが視野内にある間のみ適用されます。** これにより、白いパネルのクリッピングが防止されます。パネルが視野にない場合、 カメラはシーンに合わせて露出を調整します。
- **一定の光量では露出が安定し、光量の変化後は数秒で安定します。** 光量が一定の場合、露出とその上限は変わりません。光量が大きく変化した後は、数秒以内に表示が安定します。クリッピングによって露出が恒久的に制限されることはなくなり、ROI（関心領域）が変更されても露出が再調整されます。
- **輝度ターゲットは制御可能な範囲内に収まります。** カメラが露出上限に達し、ゲインをこれ以上上げられない状態では、輝度ターゲットは上昇しないため、光が戻ってもフレームがフレア状態になることはありません。
- **自動露出が反応しなくなったカメラは、その時点の光量に応じたレベルで維持されます。** 強い日差しの下では、カメラが最短露出時間まで下がり、フレームが暗くなったままその状態を維持することがありますが、 設定値はすべて正常を示している状態です。Chlorosはフレームを監視し、その光量に合わせてカメラを適切なレベルに維持し、光量が変化した際に設定を解除します。光センサーが暗くなったり遮られたりしても、この動作は妨げられません。

### キャプチャ・エクスポート・レベル（`all`のデフォルト）

本リリース（`lattice capture`）より、 `lattice multi-capture`、および `lattice array-capture` **は `--processing all` がデフォルト** — 各カメラに適用されるエクスポートタイプごとに1つの保存ファイルが作成され、GUIの「すべてキャプチャ」の動作と一致します。レベルは以下の通りです：

| レベル | 出力 | 適用対象 |
| --- | --- | --- |
| `raw` | センサーから直接取得したシングルチャンネル・ベイヤー（モノクロカメラ：単一バンド）。 | すべてのカメラ。 |
| `debayered` | 3チャンネルカラー、**RGB**チャンネル順で保存（TIFF 形式の場合は RGB 測光解釈が適用され、PNG/JPEG 形式も同様に RGB として開かれる）：RAW データからキャリブレーションパックのダークオフセット（DSNU）を差し引き、その後 OpenCV のエッジ対応デモザイク処理 — 線形化LUT、フラットフィールド補正、およびアンミックス処理は行わない（モノクロカメラ：1チャンネルグレースケール）。Python SDKが返すメモリ内デベイヤー処理済みフレーム（`grab()`、`frame_stream()`）は**BGR**（OpenCVの順序）ですが、 保存されたファイルのみがRGB形式となります。 | すべてのカメラ。 |
| `radiance` | 完全な放射測定チェーンによる float32 W/m²/sr/nm。 | マルチスペクトル（M3C/M3M）のみ — **RGBフィルター付きカメラの場合は RGBフィルター搭載カメラの場合は**スキップ**。 |
| `reflectance` | uint16 ρ（`32768` = 1.0）、Pix4D対応。 | マルチスペクトルのみ、かつ **DAQがバインドされ、かつカメラがキャリブレーションされている場合のみ**；それ以外の場合はスキップされる。 |
| `display` | 完全なGUIプレビューチェーン （カメラのプロファイルに基づくCCM + WB + ガンマ）。`preview/`フォルダに保存されます。`lattice capture`は、これに対して`preview`という名前も受け付けます； `array-capture` / `multi-capture` は `display` のみを受け付けます。 | すべてのカメラ。 |

1つのレベルのみを指定して、その1つだけを保存します（`--processing debayered`）。`all`を指定した場合、特定のカムに適用されないレベルはスキップされ （その旨が報告されます）が、エラーにはなりません。接続されていない、またはキャリブレーションされていないカメラでも、`raw` / `debayered` / `preview`が取得されます。

#### キャプチャで読み込まれた DAQ の値の保存

DAQ がカメラやアレイにバインドされたり、プールされたりすると、キャプチャで使用されたダウンウェル測定値は、キャプチャフォルダの `daq/` サブフォルダ内の **`.daq`**ファイルに書き込まれます。このファイルには、`daq/` （測定値が露光からどの程度離れていたか）および `.cameras.json`（どの測定値か）が記録され、`DAQ:` 行に報告されます。 これは、RAW またはデベイヤー処理済みのキャプチャを含め、すべてのレベルに当てはまるため、どのキャプチャも後で放射輝度、反射率、またはインデックスに再処理することができます。 GUI、CLI、SDKのいずれからキャプチャされたものであっても、ファイル名やサイドカーは同じです。DAQの読み取り値がまったくない場合、`.daq`は存在せず、接続された配列がない`multi-capture` （過渡的なフォールバック）の `multi-capture` は、何も保存しません。

| フラグ | 効果 |
| --- | --- |
| `--no-daq` | レベルでこれを使用していないキャプチャ（`raw`、`debayered`）の読み取り値を保存しない。`radiance`、`reflectance`、または`display`レベルでは、再インポートにその製品が作成された際の測定値が必要であるため、依然として書き込みが行われます。 |
| `--daq-tolerance-ms MS` | 許容範囲内とみなされるために、測定値が露光値からどの程度離れていてもよいか（デフォルト 500）。判定のみが変更される （`.match.json`の`in_tolerance`および`tolerance_ms`、反射率の`DlsInTolerance`）、使用される測定値自体は決して変わりません。 [露光値から大きく外れたDAQ測定値](#a-daq-reading-far-from-the-exposure)を参照してください。 |
| `--force-daq` | RAW専用キャプチャで`.daq`を要求する旧方式。現在はこれがデフォルトとなっており、`lattice capture`および`array-capture`では依然としてこのフラグを受け付けます（`multi-capture`は受け付けません）。`--no-daq`がこれに優先します。 |

どのDAQか：カメラまたはアレイにバインドされているもの（GUIの光センサー選択、プロジェクトの保存済みバインド、または`chloros-cli daq pool-bind`）、それ以外の場合はバックエンドが持つ最新のもの。RAWバースト （`array-burst`）は、以前と同様に、バーストフォルダ内の個別の読み取りごとに独自の `.daq` を保持します。

**PNGおよびJPEGはチャンネルあたり8ビットであるため、`display` / `preview`およびインデックスデータのみを使用します。** `--format png` または `jpg` で、保持できないレベルが指定された場合は、キャプチャが行われる前に拒否され、そのレベルと、それを保持できるフォーマットとしてTIFFが指定されます：

- `radiance`（float32）および `reflectance`（uint16、`32768` = 1.0） — 常にこのため、`--processing all`（デフォルト）と `--format png`/`jpg` の組み合わせも拒否されます： `--format tiff`、または `--levels display` を使用してください。
- 8 ビットを超えるストリームを送信するカメラからの `raw` および `debayered` （12ビットのM3C / Mono12 M3Mフレーム）— CLIはカメラのビット深度を認識できないため、バックエンドはこれらを拒否します（HTTP 400）。単一の-フレームの `lattice capture --processing debayered` は、PNG/JPEG 用に 8 ビットのフレームを取得するため拒否されず、永続配列を持たない `multi-capture`（8 ビットのデベイヤー処理済み）も同様に拒否されません。

すべてのキャプチャルートは同じように応答します （`error_code`、`FORMAT_CANNOT_HOLD_LEVEL`）：`lattice capture`、`multi-capture`、`array-capture`、 `project capture` / `burst`、およびそれらの背後にあるHTTPルート。

### キャプチャフォルダの内訳

各エクスポート形式は、`-o`の下にある**専用のサブフォルダ**に保存されるため、多階層のキャプチャでも形式が混在することはありません。 図は、**`all`による1台のカメラのデフォルトキャプチャ（`lattice capture`）**を示しています。カメラごとに複数のレベルがあるため、すべてのファイルに`_<level>`というサフィックスが付いています：

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

ファイル名のバリエーション：

- **カメラフィールド — シリアル番号、その後にフィルター：** すべてのキャプチャファイルには、撮影したカメラ名が `SN<serial>-<filter>` の形式で付きます。ここで、`<filter>` はカメラモデルのフィルター部分です： `SN261902142-F450` の場合、`M3M-L87-F450`、 `M3M-L87-F450`の場合は`SN213800234-FRGN`、RGBカメラの場合は`SN214701292-FRGB`となります。フィルターはシリアル番号と結合され、`-`となり、 `_` とは異なるため、名前は以前と同様に `_` で区切られたフィールド（プレフィックス、日付、時刻、ミリ秒、カメラ、レベル）で構成されます。 モデルを読み取れないカメラ（DeviceUserIDが消去されているもの）は、単に「`SN<serial>`」という名前になります。以前のリリースからのキャプチャには名前にフィルターが含まれていませんが、Chlorosは両方を読み取ります。
- **プレフィックス：** `lattice capture`はファイルを`capture_…`と命名します。接続されたアレイを持つ`array-capture`、`multi-capture`、および`project capture` / `burst` / `stream` といった配列名の場合は、`sync_<ts>_SN<serial>-<filter>…` と命名されます。
- **レベルサフィックスは、複数のレベルがある場合のみ使用されます。**1回のキャプチャで**複数の**レベルを保存するカメラ（`all`、または `--levels a,b`）は、`_<level>` を追加します。**1つ** レベルのみを保存するカメラは、サフィックスなしの単純な名前を維持します（`--processing reflectance` は `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` を書き出します）。これにより、単一レベルのキャプチャ向けに作成されたスクリプトは引き続き動作します。 レベルは依然としてフォルダ名によって示されます。
- **クランプマスクは製品名の後に続きます：** `<product name>_clampmask.tif`。1レベルの反射率キャプチャは `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif` を書き出し、`all` のキャプチャは `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif` を書き出します。
- **拡張子**は `--format` に続きます。デフォルトの `tiff` は `.tiff` を生成し、`--format tif` は `.tif` となります（クランプマスクは常に `.tif` です）。
- **1つの例外：** `lattice capture --processing debayered`（または `--levels debayered`）は、高速な単一-フレームのパスを受け取り、`capture_<ts>_SN<serial>-<filter>.tiff`をサブフォルダを経由せずに直接`-o`に書き込みます。
- **ストリームとバースト**は同じフィールドを持ちます： `lattice continuous`は、`frame_<seq>_SN<serial>-<filter>.tiff`、`lattice multi-stream`、`stream_<seq>_SN<serial>-<filter>_<frame id>.tiff`、および生のバーストデータである`<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`を書き込みます。

`<ts>`はキャプチャタイムスタンプ、`<serial>`はカメラのシリアル番号であるため、1つの同期グループでは
カメラ間でタイムスタンプが共有されます。**1つの非対称性に注意してください：** `display` レベルは
「`preview/`」という名前のフォルダに保存されますが、マルチレベルファイルでは `_display` が名前 — フォルダ名と接尾辞が異なるのは
そのレベルに限られます。不明なレベルの場合は、そのレベル名と同じ名前のフォルダに保存され、サブフォルダが
作成できない場合でも、ファイルが失われることなく出力ルートに書き込まれます。

**キャプチャフォルダの再処理：**`chloros-cli process` を**キャプチャのルート**
(`output/`) に向けます。`process` は通常、指定したフォルダのみをインポートしますが、そのフォルダに
画像がなく、サブフォルダが存在する場合、自動的に階層を辿ります。そのため、各レベルのサブフォルダと
`daq/`サブフォルダが一度にすべて取り込まれます。キャプチャの各レベルは、レベルごとに1つの画像としてではなく、
単一の画像としてインポートされ、他のレベルはモードとして利用可能になります。

**レベルサブフォルダ**を直接指定（例：`output/raw/`）することも可能です。この場合、`daq/`は
残ってしまうため、再インポートする際には、DAQの読み取りデータをコピーするか、その場所を指定して併せて読み込んでください`raw/`から放射測定
プロダクトを導出する際は、`daq/`も併せてコピーするか、DAQ読み取り先を指定してください。そうしないと、タイムスタンプの一致を照合する対象がなくなります。

**処理は常に`raw`から開始されます。** 各キャプチャ内では、生データフレームがパイプラインのソースとなります。
`debayered`、`radiance`、`reflectance`、および`preview`は表示可能なモードとして存在しますが、パイプラインに
フィードバックされることはありません。派生プロダクトを再処理すると、そのピクセルにすでに焼き込まれているヴィネット、CCM、および
放射輝度の計算が再適用されてしまうため、Chlorosは二重処理を避けるために
これを拒否します。知っておくべき2つの結果：

- `index/` および `composite/` のレンダリングは **決して** 処理されません。これらはキャプチャではなく出力であり、
  NDVI LUT レンダリングには意味のある放射輝度の解釈がありません。
- `raw` **を含まない**状態でエクスポートされたキャプチャフォルダ（例：`array-capture --processing reflectance`）には
  正当なパイプラインソースがありません。これらのキャプチャは通常通りインポートおよび表示されますが、`process`は
  それらをスキップし、次のように表示します：

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  派生製品を確実に通過させる必要がある場合 —
  `demosaic`が有効な状態でキャプチャされたハブセッションや、レガシーフォルダなど）を確実に通過させる必要がある場合、`--input-level {raw,debayered,processed}`はエントリ
  ポイントを強制し、スキップ設定を上書きします。このフラグは意図的に設けられた回避策であり、`auto` （デフォルト）
  は、生データ（raw）を含まないキャプチャを処理することはありません。

**キャリブレーションタグ：`raw`でのみ有効。** キャプチャの `raw` ファイルには、カメラのキャリブレーションパケット（`CalibrationJSON` ペイロードおよび MicaSense 互換タグ（`BlackLevel`、 `RadiometricCalibration`、`Band{R,G,B}RadiometricCalibration`、`MicaSenseVignetting*`）を含んでおり、これらはセンサーのRAW値を記述しています。 それ以外のレベルのファイル（`debayered`、`display`、`radiance`、 `reflectance`、`index/`のレンダリング）およびすべての`lattice continuous`／ `multi-stream`）の各フレームには、カメラの識別情報（`XMP-mapir:CameraSerial`、`CameraModel`、`FilterType`、 `LensModel`、`CalBundleSha256`、…）と、フレーム固有のレコードである`XMP-mapir:FrameJSON`（露光時間、ゲイン、温度、ピクセルフォーマット、撮影時刻、 DAQマッチ）を含み、すべての `process` 製品には識別子が含まれています。`process` は、そのシリアル番号に基づいてChlorosのキャリブレーションキャッシュ内から該当するファイルのキャリブレーションを検索し、 コンピュータにキャッシュがない場合、そのパックを一度ダウンロードします。

#### 複数のカメラから作成されたファイル

複数のカメラのバンドから作成されたファイル — マルチバンド・スタック（`align-apply`、`align-stream`、`index --live --save-multiband`）、 複数のカメラから作成されたライブインデックス（`index.tif`、`index_LUT.png`）、アレイキャプチャの `composite/` 画像 — これらには、作成に使用されたすべてのカメラの名前が含まれています：

- **カメラごとのリスト（バンド順）**（合成画像：背景レイヤー、次に前景；インデックスレイヤー、そのバンドのソースとなったカメラ）：`XMP-mapir:SourceCameraSerials`、 `SourceCameraModels`、`SourceSensorProfiles`、`SourceProductLines`、`SourceFilterTypes`、`SourceLensModels`、 `SourceCalDateUTCs`、`SourceCalBundleSha256s`（各カメラのバンドが通過したキャリブレーションパック、それ以外の場合 — `align-apply` / `align-stream`のスタック（このスタックのバンドはどのキャリブレーションも受けていない） — カメラのパックChlorosがキャッシュしたもので、そのキャプチャファイルの名前にあるもの）、 `SourceLightboxCertIds`、`SourceHalogenLampCertIds`、`SourcePackTiers` — 並列リストであり、各リストの*n*番目の項目は同じカメラを表す。`-`は、カメラに値がない場合（キャリブレーションパックなし、レンズ不明）を示す。 各カメラは**1つの**項目です。パックの特定のフィールドに複数の値がある場合（パックのハロゲンランプ証明書など）、それらは `;` (`F-1479-1;F-1480-1;F-1481-1`) で連結され、決してコンマで区切られることはありません。 したがって、リストは表示形式にかかわらず、カメラ1台につき1項目として読み取られます。
- **EXIF `SerialNumber`** には、コンマで区切られた同一のシリアル番号が含まれています（`214000535,214000551`）。
- これには、**単一の**カメラを特定する識別子は含まれていません（`XMP-mapir:CameraSerial`、`CalBundleSha256`、…）： これらは1台のカメラとそのパックを識別するものであり、`process`はそれらからファイルのキャリブレーションを算出します。複数のカメラのバンドから計算されたインデックス（`index.tif`、 `index_LUT.png`）から算出されたインデックスには、EXIF情報として`Model` `LATTICE` （リグ）が記録されており、単一のカメラの `LATT-…` モデルではありません。**1台**のカメラのバンドからなるファイル — バンドごとの TIFFファイル（`--per-band`）、1つの`align-apply --per-camera`フレーム、単一カメラのスタックまたはライブインデックスなど——は、他のすべてのキャプチャファイルと同様に、そのカメラ固有の識別タグを含んでいます。 `align-apply`のスタック、バンド別ファイル、カメラ別ファイル、および各`align-stream`スタックとそのバンド別ファイルには、そのグループのキャプチャ時刻（基準カメラの露光時刻、UTC）が記録されており、 また、モノクロカメラの `--per-camera` フレームは、スタックと同様に `--bit-depth` 形式の単一プレーンです。**カメラの向き（反転／回転）。** カメラの向き設定（GUI の「Mirror H」／
Mirror V／回転）は、キャプチャ時に保存されるデータに影響を与えます：

- **`raw` および `debayered` は、設定にかかわらず、センサーが読み取ったままの状態で保存されます**。
  ベイヤーモザイクを回転または反転させると、カラーフィルターのパターンが異なるものとなり、キャリブレーション
  マップ（ダークオフセット、フラットフィールド）はセンサー独自のグリッド上にあります。
- **`preview`/`display`、`radiance`、 `reflectance`および`index/`のレンダリング画像は**、すべてのキャリブレーション計算後に
  1回回転されているため、カメラの構図通りに表示されます。これらの
  `clamp_masks/…_clampmask.tif`ファイルは同じグリッド上にあります。
- **すべてのキャプチャファイルには、XMPに設定が記録されています**： `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`)、`ChlorosRotation` (0 / 90 / 180 / 270、反転処理後の
  時計回り）および `ChlorosOrientationApplied` — `false`（RAWおよびデベイヤー処理済み（ネイティブピクセル））、
  `true`は方向補正済みデータに対するものです。`ChlorosProcessing`には、そのファイルの
  ピクセルに適用された処理が一覧表示されます（ネイティブの場合は`none`）。これには、`lattice continuous`がストリーミングするすべてのフレーム（
  カメラ自身の設定、未適用）および`lattice multi-stream`のすべてのフレーム（アイデンティティ：その
  構成フレームは、配列キャプチャと同様にネイティブである）が含まれます。 `lattice align-stream` / `align-apply`
  スタックは、マスターカメラの設定（未適用）を記録します。これらのバンドはマスターの
  フレームにワープされますが、回転はされません。
- **`process`は、RAWデータを処理する際に記録された方位を適用します**：まずセンサー座標系でダークオフセット、フラット
  フィールド、デモザイク、アンミックスを行い、次にアレイ位置合わせのワープを行い、
  向きを最後に適用します。これにより、プレビュー、放射輝度、反射率、および屈折率の各出力は、
  ライブキャプチャと全く同じ状態になり、`ChlorosOrientationApplied=true`として記録されます。未校正のフォールバック
  出力（`Sensor_Response_Images`、`Vignette_Corrected_Images`）は、
  フラットフィールド処理の後、同じ向きに調整されています。デベイヤー処理済みのプロダクトはネイティブのままです。
- **Chloros画像ビューアは、記録順に回転されたRAWファイルまたはデベイヤー処理済みファイルを表示します**（表示のみ —
  ファイル自体は変更されません）。そのため、そのプロダクトのように見えます。 サムネイル、フルサイズ表示、
  カーソル読み出し、およびインデックス／LUTサンドボックスはすべて、同じ回転角度を使用します。レコードのないファイルや、
  （すでに方向が調整済みの）プロダクトは、保存されたままの状態で表示されます。 サンドボックスへのエクスポートでは、
  ピクセルが持つ向きが記録されます（`ChlorosOrientationApplied=true`；記録のないレイヤーの識別子）。
- **レコードのないキャプチャはネイティブとして処理される**（動作に変更なし）。1.3.0以前では、
  反転や回転が設定されたシングルカメラキャプチャの場合、RAWファイルおよびデベイヤー済みファイルが自動的に回転されていたが、
  Chlorosは、ネイティブ形式からすでに-回転済みのRAWデータをネイティブデータと区別できず、推測も行わないため、
  保存されたままの状態で処理します。キャリブレーションマップはセンサー独自のグリッド上に存在するため、キャリブレーションが重要な場合は、
  1.3.0で撮影されたキャプチャを使用してください。
- **配列はデフォルトでは構成ファイルを回転させません**（アライメントワープおよび`Alignment*` XMPは、
  カメラ固有の、回転されていないフレームで保存されています）し、アイデンティティを記録します。 `array-capture --orient`は、アライメントワープ後に各
  メンバーの処理結果を回転させます。その際、メンバーが
  そのフレームにワープされた際の**マスター**の向き（これによりメンバー間の位置合わせが維持されます）で回転させるか、そうでない場合は各カメラ自身の向きで回転させます。 各
  ファイルは、自身のピクセルが存在するフレームの向き（RAWデータ、自身のカメラの向き）を記録し、
  アライメント変換を含むRAWデータには、`ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`：マスターのフレームにワープされた製品の場合、マスターの座標系
  。`process`は、各プロダクトをそのピクセルに属する値だけ回転させます：
  RAWをワープする際にアライメントが適用されている場合はアライメント済み値（`--array-alignment`、デフォルト — ライブアライメント
  キャプチャの生成物）、そうでない場合はカメラ固有の座標系を使用します（`--no-array-alignment` — ライブ
  非アラインメントキャプチャのもの）。同じオプションは、`array-burst`、`project capture` / `burst` /
  `stream`、レシピ、およびSDKについても同様のオプションが用意されています。

### 混合フィルターアレイにおけるスキップされたキャプチャ

1つのアレイにRGBカメラとマルチスペクトルカメラを混在させる場合、`array-capture --processing radiance`（または`reflectance`）はマルチスペクトルフレームを保存し、**RGBカメラのキャプチャはスキップ**RGBカメラのキャプチャを**スキップ**します。ブロードバンドセンサーでは、ベイヤー単位の放射輝度は意味をなさないためです。 CLIは、保存された各ファイル（エクスポートレベルを含む）、書き込まれた各`.daq`、および各スキップを明示的に表示するため、ファイル数が予想外になることはありません：

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

スキップ理由のトークンは `<level>-not-applicable-to-rgb-cam` というパターンに従います。キャリブレーションパックが読み込まれていないカメラは、`radiance` / `reflectance` をスキップし、`<level>-needs-calibration (connect the camera so its cal pack loads)`でスキップし、使用可能なDAQの反射率測定値がない場合は`reflectance-skipped-no-dls`でスキップします。また、反射率のスキップは`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`でも発生し、 また、バンドの大部分がDAQ光センサーの放射測定校正範囲（約374～974 nm）外にある場合、`dls-uncalibrated-band-<nm>`でも同様の現象が発生します。出荷されているSKUの中では、F988のみが対応しており、そのサポート対象となるワークフローは反射率パネルワークフローです。

フィルターの種類に関係なくすべてのカメラを含めるには `--processing debayered`（または `display`）を使用し、カメラごとに適用可能なすべてのレベルを一度に取得するにはデフォルトの `all` を使用してください。

### 露光値から大きく外れたDAQ測定値

キャプチャごとに保存される各 `.daq` は、その露光値に対応するDAQ測定値であり、その横にある `.match.json` には、両者の差 (`gap_ms`)、およびそれが許容範囲内かどうか（`in_tolerance`、キャプチャ設定で指定されていない限り500 ms）を記録します。 許容範囲外の測定値も使用されるため、フレームが失われることはなく、その値にはラベルが付けられます。反射率ファイルの XMP には、`DlsInTolerance=False` および `DlsMatchGapS` が含まれます。 また、`lattice capture`、`multi-capture`、および`array-capture`も、関連する`.daq`の下にこれを表示し、説明するとDAQ'ストリームの障害が発生します。その理由を説明すると：

**読み取り値と露光時間の対応関係。** キャプチャは、露光終了後のDAQによる最初の読み取り値を待機し、その前後の読み取り値を合成できるようにします（`method` `interpolated`）。DAQ-Eは、日中は0.4秒ごと、屋内では1～2秒ごとに読み取り値を送信するため、この待機時間はコマンドに対して（露光時間ではなく）それらの間隔の約1.5倍分が加算されます。送信を停止したDAQについては待機しません。DAQ-Eの測定値は、前回の測定値以降のインターバルで平均化された光量であるため、そのインターバルの中間に配置されます。つまり、`gap_ms`は露光開始からその中間点までの時間を表し、`bracket_stamps_ns`は測定値自体のタイムスタンプを示します。 `array-capture --continuous`は、キャプチャの速度を落とさずに待機します。そのRAWファイルは即座に書き込まれ、読み取りデータが到着次第、`.daq`が更新されます。 `--interval` では、各パスが単一のキャプチャと同様に待機します。

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

その読み取り値から反射率ファイルが書き込まれなかった場合、最初の行は代わりに `saved anyway; its .match.json says in_tolerance: false` で終わります。

キャプチャは依然として `0` で終了します：すべてのファイルが書き込まれ、その内容が示されています。ストリームが停止した DAQ は最後の読み取り値を出し続けるため、そのキャプチャの反射率を信頼する前に、`chloros-cli daq pool-list` で確認してください。 バックエンドは、データ送信を停止したプールされたDAQ-Eを再接続します （[データ送信を停止した DAQ-E](#a-daq-e-that-stops-delivering) を参照）。許容範囲外の測定値は、保持された測定値（後述）でない限り、その `DAQ:` 行のみが出力されます。

**DAQが利用できなくなった後も保持された測定値。** ある露光に対してDAQが測定値を提供できない場合（バインドされたDAQが利用できない、またはプールされたDAQのいずれも測定値を持っていない場合、例えば`daq pool-disconnect`の後など）、キャプチャはChlorosが最後に持っていた測定値を再利用し、その`.match.json`は、この露光に対するその測定値を次のように記述します：`method`、`retained`、`gap_ms`を、その測定値が得られたサンプルに照らし合わせて記述し、`in_tolerance`はそのギャップからの値、そして`retained_from_epoch_ns`は、その測定値が最初に照合された露光です。したがって、DAQが停止してからかなり時間が経ってから行われたキャプチャでは、上記の行が印刷され、その下には さらに1行が追加され、これは測定値がまだ許容範囲内にある間にも出力されます：

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

その`.daq`には、依然としてその測定値を取得したDAQの名前と、その測定が行われた際のキャリブレーションおよびキャップが記載されているため、再インポート時にどのDAQ、キャリブレーション、キャップによって生成されたものかがわかります。

### 書き込みに失敗したファイル

ファイルの書き込みに失敗したカメラ（ドライブの容量不足や権限エラーなど）があっても、他のカメラのファイルの保存は妨げられません。すべてのキャプチャコマンド（`lattice capture`、`multi-capture`、 `array-capture`; `project capture`, `burst`, `stream`; およびレシピの `capture`、`burst`、`stream`）は、保存に失敗した各ファイルを stderr に出力し、 その後、残りのファイルは正常に保存されたにもかかわらず、**`1`で終了**します：

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

キャプチャに含まれるすべてのファイルが対象となっており、唯一の-フレームのデベイヤー処理済みファイル（`lattice capture --levels debayered`）も含まれています。`.daq`の読み取りデータのうち、`.match.json`の由来を示すサイドカーファイルが書き込めなかったものが、`[match]`としてリストされます（`.daq`自体は保存されています）。

ドライブが満杯の場合、繰り返しキャプチャ（`array-capture --continuous` / `--interval`、`project burst` / `stream`、レシピ `burst` / `stream`）**そのパスで停止**し、残りの実行期間中、同じドライブに何も書き込まない。その他のエラーは発生した各パスで報告され、実行は最後まで継続された後、`1`で終了する。`array-capture --continuous` （バックエンドの連続キャプチャ）には、失敗したファイルを含め、ファイルは一切リストされません。ファイル数は進行状況および要約行でカウントされます （`N failed`、`N file(s) failed to write.`）でそれらをカウントし、`1`で終了します — [連続モード](#continuous-mode---continuous)を参照のこと。

レコーダー（`array-burst`および`array-record`）は、自身の失敗を報告します。 `array-burst`は、ドライブが満杯になったとき、または満杯になる直前に自動的に停止します。どちらも、その他の書き込み失敗を無視して処理を続行します。いずれかがエラーを報告すると、そのエラーはカウンタの後に表示され、コマンド **`1`**：

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:`が`Disk full:`の代わりに表示される場合、バーストがドライブの容量を 。停止前に記録されたフレームは保持されます。`array-burst --build`は、空き容量が不足した際にビルドをスキップします。これは、動画が同じドライブに書き込まれるためです。まず空き容量を確保してから、`array-build-video --burst-dir …`を実行してください。その他のエラーが発生した場合でも、ビルドは実行され続けます。

`array-record`でフレームが1つも書き込まれなかった場合も、録画失敗となります （`Recorder error: no-frames: the combined view published no frame to record …`、`1`を終了） — [`array-record`](#array-record--combined-index-videogif-monitoring-grade)を参照してください。

---

## キャプチャモード、レコーダー、およびオフライン再処理

これらはすべて **永続配列** 上で動作します（まず `array-connect` を実行してください）。これらは GUI のキャプチャパネルを反映しています。

### `array-capture` のモード

`array-capture` は、4つのシャッターモードと一連のエクスポート切り替えオプションを備えた単一のコマンドです：

| モード | フラグ | 動作 |
| --- | --- | --- |
| **シングル** *(デフォルト)* | (なし) | 1つの同期キャプチャグループを実行した後、終了します。 |
| **連続** | `--continuous` | `Ctrl+C`、`--count N`によるキャプチャ、 `--duration S`、またはドライブが満杯になるまでキャプチャを連続して実行します。バックエンド独自の連続キャプチャ（GUIの「連続モード」）を実行します — 以下の[連続モード](#continuous-mode---continuous)を参照してください。 |
| **間隔** | `--interval S` | `S` 秒ごとに 1 パス（各パスの開始時から計測）、範囲は同じ。 |
| **最速** | `--fastest` | 生データのみ + 割り当てられたDAQ測定値 + 複合インデックス；放射度/反射率/表示の計算をスキップするため、フレームの処理が高速化されます。`--processing raw --force-daq`を前提とします。保存された`.daq`は、後で校正済みプロダクトに再処理してください。 |

エクスポート切り替え（どのモードとも組み合わせ可能。すべてGUI/SDKエンドポイントを共有）：

| フラグ | 効果 |
| --- | --- |
| `--array-id ID` | キャプチャ元の配列（接続されているのが1つだけの場合は省略）。 |
| `-o, --output DIR` | 出力フォルダ（デフォルトは `output`、コマンドを実行したフォルダの下）。 |
| `--format {tiff,tif,png,jpg,jpeg}` | ファイル形式（デフォルトは `tiff`）。 PNG / JPEG 形式の場合は、`display` およびインデックスレンダリングのみが対象となります。[キャプチャのエクスポートレベル](#capture-export-levels-the-all-default)を参照してください。 |
| `--processing LEVEL` | 単一のエクスポートレベル (`raw`、`debayered`、`radiance`、`reflectance`、`display`)、または `all` （デフォルト）。 |
| `--levels L1,L2,…` | エクスポート型の明示的なサブセット（例：`raw,radiance,reflectance`）；**`--processing`を上書き**します。 |
| `--aligned` / `--no-aligned` | **デフォルト：アラインメントあり。これは、Chlorosアプリが同じ配列をキャプチャする際の設定と同じです**（Chlorosアプリの配列ごとの**アラインメントあり**キャプチャ設定は、無効にしない限りオンになっています）。各メンバーの非RAWエクスポート（`debayered`、`display`、`radiance`、`reflectance`、インデックスレンダリング） は、マスターカメラのフレームにワープされ、メンバーの共通オーバーラップ領域にトリミングされます。そのため、ファイルはピクセル単位で位置合わせされ、アプリのファイルと同じサイズ、トリミング、および `Alignment*` タグで出力されます — センサー （またはROI）よりも小さくなります。ROIの下では、変換はウィンドウに基づいて再計算されます。`XMP-mapir:AlignmentJSON`は`"applied": true`を指定し、`crop_roi`を生成します。 Rawはネイティブサイズのままワープ処理されませんが、メタデータ（`"applied": false`）に変換情報が含まれます。`--no-aligned`は、すべてのファイルをネイティブサイズで、ワープ処理せずに保存します。 `--aligned`は、明示されたデフォルト設定です。ワープが適用されるかどうかに関わらず、配列にアライメントプロファイルが設定されると、各メンバーファイルはそのカメラの`Alignment*`変換を記録します — ただし、プロファイルとアラインメントできないカメラ（除外されたもの、またはソルブ失敗によりアイデンティティ変換に強制されたもの）および、他のカメラがアラインメントされていない場合の参照カメラは例外であり、これらには`Alignment*`タグは付かない。配列にプロファイルがない場合、キャプチャはアラインメントされていない状態で保存され、CLIは、ワープが適用できなかった他のすべてのケース（異なる解像度のメンバー、ケーブル同期されていない配列、プロファイルにマッピングできないフレーム）と同様に、その理由を`Alignment:`行にその理由を表示します。これは、ワープを適用できなかった他のすべてのケース（メンバーの解像度が異なる、アレイがケーブル同期されていない、プロファイルをマッピングできないフレームなど）でも同様です。アレイの自動アライメントは、露出が安定するとプロファイルを作成します。 `align-calibrate`はオンデマンドでプロファイルを作成します。 |
| `--orient` | 各メンバーの**プロダクト**— `display`、`radiance`、 `reflectance`、`index/`のレンダリング、および`composite/` — を、アライメントワープ処理後にカメラの向きに合わせて変換します。つまり、**マスターの**向きで、マスターのフレームにワープされたもの（デフォルト）。これにより、各メンバーファイルの位置関係が維持されます。アライメントされていないもの（`--no-aligned`、またはワープを適用できなかったキャプチャ）は、自身のカメラの向きで保持されます。**デフォルト：オフ — すべてのメンバーファイルはネイティブのまま**です。これは、配列キャプチャでは常にそうであるように。 `raw` および `debayered` は、いずれの場合もネイティブのままです。すべてのファイルには、その製品が持つ向きが記録されます（[カメラの向き](#what-a-captures-folder-looks-like)を参照）。`Alignment*`のXMPファイルには、オリエンテーションの前に適用されたネイティブフレーム変換が保持されるため、`process` （ワープ、その後オリエンテーション）は、ライブの生成データを再現します。 |
| `--index` / `--no-index` | カメラごとに設定されている植生インデックスのオーバーレイを保存／スキップします。デフォルト：レンダリングします。 |
| `--no-daq` | 選択されたレベルでDAQが使用されていない場合（RAWデータのみの取得）、`.daq`サイドカーをスキップします。 デフォルトでは測定値が保存されるため、フレームをオフラインで反射率/指数に再処理できます。`--force-daq`も引き続き有効です（デフォルト）。 |
| `--daq-tolerance-ms MS` | これらのキャプチャに対するDAQマッチング許容誤差 （デフォルト 500 ms）。 |
| `--smart` | トリガーする前に、すべてのカメラで AE が安定するのを待ちます（[Smart-AE / Smart-Capture](#smart-ae--smart-capture) を参照）。 |
| `--compression {deflate,none}` | TIFFピクセル圧縮。`none`（バージョン1.3.0以降ではデフォルト；1.2.0では`deflate`と記述）= 非圧縮 — センサーのピクセルを読み取ったままの状態で保存され、フル解像度のフレームあたり約6.3 MB、最も高速な持続転送レートを実現します。`deflate` = ロスレス zlib L1 + 水平予測方式、フレームあたり約4.1 MB、処理速度は遅くなります。どちらもロスレスであり、インポート時の読み取り結果は同一です。 |

> **シングルライトTIFF + 持続転送レートモデル。**すべてのファイルは**1回**のみ書き込まれます：ピクセル、 XMPパケット、IFD0の「メーカー/モデル/カメラシリアル番号」、およびEXIFの`ExposureTime`／`SerialNumber`タグを1回のパスで書き込み、その後何も上書きされることはありません。`--continuous`Xは連続した同期グループをキャプチャします。各キャプチャは前のグループの直後のグループを取得し（そのキャプチャがまだ保存中の間に公開されたグループも含む）、そのため配列のレートで実行されます。DEFLATEはPythonのGILを保持するため、圧縮書き込みはカメラごとのライタースレッド間で**並列化されません**。 センサーレートでのフル解像度キャプチャを継続するには、非圧縮書き込み（デフォルト）**および** NVMeクラスのディスク（9カメラアレイで約10 fpsの場合、持続転送速度約600 MB/s）が必要です。同じ設定項目は、`POST /api/camera/array/capture`上で`compression`として公開されています。

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### 連続モード (`--continuous`)

`array-capture --continuous`（`--interval`または`--smart`を指定しない場合）は、処理を**バックエンド独自の連続キャプチャ**（GUIの「連続モード」が実行されるセッション）に引き継ぎます。 バックエンドは、同期されたグループをすべて自身で保存し、2つのキャプチャを同時に処理し、グループ間のCLIからのリクエストを排除することで、キャプチャごとに1回のリクエストでは処理が追いつかない高速アレイにも対応します（512ピクセルのROIを持つ9台のカメラアレイで、73～77キャプチャ/秒を計測）。 上記のエクスポート設定は、各キャプチャに個別に適用されます。

- **範囲（Bounds）：** `--count N`は、N回のキャプチャ（1回のキャプチャ＝1つの同期済みグループ：メンバー全員のファイル）後に停止します; `--duration S`はS秒後に停止します。いずれも指定しない場合、`Ctrl+C`、ドライブの容量がいっぱいになる、エラーが発生する、またはアレイが切断されるまで実行されます。`Ctrl+C`はバックエンドのセッションを停止し、 その後、要約を出力します。
- **進行状況**は2秒ごとに表示され、値が0以外の場合には`, N failed`および`, N groups skipped`が追加されます（スキップされたグループとは、アレイが公開したものの、どのキャプチャも取得しなかったグループのことです）：

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **ファイルごとの一覧表示はありません。** 実行は、1行の要約（キャプチャ数、レート、ファイル数、出力フォルダ）で終了し、値がゼロでない場合はその後に `N file(s) failed to write.` / `N group(s) skipped.` が続きます：

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **停止**：ドライブが満杯の場合は `Continuous capture stopped: the drive is full.`、エラーまたはアレイが消失した場合は `Continuous capture stopped: <reason>.` で停止します（いずれも stderr に出力されます）。
- **終了ステータス：** キャプチャが保存され、すべてのファイル（`Ctrl+C`を含む）の書き込みが完了した場合は `0`、書き込みに失敗したファイルがある場合、 ドライブの容量不足、エラー、アレイの消失、またはキャプチャが全く行われなかった場合。
- **Microsoft Defender:** Defenderがキャプチャによって書き込まれるファイルのスキャンに1つ以上のCPUコアを消費した場合、実行時にstderrに一度だけ「`Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.`」と出力されます。インストーラの [システム設定](#windows-exe)を参照してください。
- 1つのアレイにつき、一度に1つの連続キャプチャのみ実行可能です。2つ目のキャプチャは `a continuous capture is already running for this array` エラーで拒否されます。 HTTP経由の場合、セッションは`POST /api/camera/array/continuous/start`（`/api/camera/array/capture`の本文に`max_captures`および`duration_s`が追加されたもの）となり、 `GET /api/camera/array/continuous/status?array_id=ID`および`POST /api/camera/array/continuous/stop`で構成されます。

**Interval**（`--interval S`、`--continuous`の有無にかかわらず）および**`--continuous --smart`**はリクエストごとのループを維持します：1パスにつき1回のキャプチャリクエスト —**Interval**の場合は各スケジュールされた時刻に、**`--smart`**の場合はAEが安定した後 — 各パスのファイルはシングルモードと同様にリストされ、末尾に `Done — N capture pass(es).` 行が記載され、`--count N` でパス数がカウントされる。

### `array-record` — 統合インデックス動画/GIF（監視用）

配列の **ライブ複合インデックスビュー** を `.avi`（およびオプションで `.gif`）に記録します。GUI の複合ビューが開かれている場合、表示されているそのビューをそのまま記録します。 何も開いていない場合、バックエンドがビュー自体をレンダリングします。つまり、`--expr`で渡されたインデックス、あるいは配列のアライメントが完了した時点でのデフォルトのNDVIが使用されます。 デフォルトのNDVIは、ライブビューと同様に、660 nmおよび850 nmに最も近い赤色バンドとNIRバンドを使用します。記録中に開かれたGUIビューは、開かれている間、記録されます。2秒ごとに進行状況をポーリングし、`--duration`、 `Ctrl+C`が検出されたとき、またはレコーダーが自動的に終了したときに停止します。

フレームが書き込まれない記録は失敗となります。レコーダーは `no-frames: …` を報告し（配列の位置合わせや露出の安定化がまだ進行中である可能性があります）、コマンドは **`1` で終了** します。

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--array-id ID` | 配列のみ | ターゲット配列（接続されているのが 1 つのみの場合は省略）。 |
| `-o, --output DIR` | `output` | 出力ディレクトリ。相対パスは、 コマンドを実行したフォルダを基準とします。 |
| `--fps F` | `10` | 録画フレームレート。 |
| `--duration S` | Ctrl+C まで | `S` 秒後に自動停止。 |
| `--gif` | off | アニメーション GIF も書き出す。 |
| `--gif-only` | オフ | GIFのみを書き出す（`.avi`はなし）。 |
| `--expr EXPR` | 配列のNDVI | GUIビューが開いていない場合に記録するインデックス式。配列のバンド名で指定（例：`'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; [バンド名](#index--vegetation-maths)を参照)。短いカラー名（`NIR`、`Red`）は、配列の1つのバンドにそれがある場合にのみ機能します。開いているGUIビューは、代わりに表示されているとおりに記録されます。 |

### `array-burst` — 生データ（Bayer）高フレームレート連写（分析用）

グラブループの同期グループバッファを直接読み込みます — **キャリブレーションチェーン、exiftool、ライブビューは不要** — そのため、カメラの最大グラブレートで動作します。RAWフレーム + フレームごとのマニフェスト + `.daq`を、`<output>/bursts/<base>/`の下にある個別のDLS読み取りごとに書き込みます。 オフラインで再処理（次のコマンド）するか、`--build`を指定して停止時に即座に実行します。

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--array-id ID` | 配列のみ | ターゲット配列。 |
| `-o, --output DIR` | `output` | 出力ディレクトリ（バーストデータは `<DIR>/bursts/<base>/` に保存されます）。 |
| `--duration S` | Ctrl+C まで | `S` 秒後に自動停止。 |
| `--max-frames N` | 無制限 | `N` 個の生フレーム後に自動`N` 生のフレーム後に自動停止。 |
| `--orient` | オフ | このバーストからビルドが生成するプロダクトは、カメラの向きによって回転されます。カメラごとのプロダクトは、そのカメラ自身の向きで、 結合インデックスはマスターの向きに従って回転されます — `array-capture --orient` と同様に。RAW フレームはネイティブのまま保持され、 バーストの `manifest.json` は、各メンバーの向きとオプションを記録します。 |
| `--build` | オフ | 停止後、直ちにバーストを再処理します（`array-build-video` と同様）。 |
| `--products …` | `combined:index` | `--build` 指定時：どの動画を作成するか（以下を参照）。 |
| `--fps F` | `10` | `--build`と併用時：出力動画のfps。 |
| `--save-tiffs` | オフ | `--build` と併用時：フレームごとのキャリブレーション済み TIFF ファイルも保存（各ファイルには、ピクセルがクリップされた際の [アンミックス・クランプ・マスク](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on）を各フレームごとに保存します。ビルド結果では、これらは `clamp_masks` として一覧表示されます）。 |
| `--gif` | off | `--build` を使用する場合：アニメーション GIF も書き出します。 |

### `array-build-video` — 保存済みのバーストをオフラインで再処理

各RAWフレームを、最も近い保存済みの`.daq`測定値と時間的に照合し、**インポートパイプラインと同じ放射度／反射率／屈折率の処理チェーン**を通過させて、1つ以上の動画をレンダリングします。

`--products` は `kind:level` 項目のカンマ区切りリストであり、`kind` ∈ `per_cam` | `combined` であり、かつ `level` ∈ `radiance` | `reflectance` | `index` です。 単なる `level`（`kind:` なし）の場合、デフォルトは `per_cam` となります。デフォルトは `combined:index` です。

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--burst-dir DIR` | (必須) | バーストフォルダへのパス (`…/bursts/<base>/`) へのパス。 |
| `--products …` | `combined:index` | `kind:level` リスト（上記と同様）。 |
| `--fps F` | `10` | 出力動画のfps。 |
| `--save-tiffs` | off | 動画とともに、フレームごとのキャリブレーション済みTIFFも保存する。 |
| `--gif` | off | アニメーションGIFも書き出す。 |

**動画にはクランプマスクは含まれない**（1つの画像ストリーム、8ビット）。その代わりに、ビルド結果には `clamp_fractions` が含まれる：動画出力ごとに、 各フレームのピクセル分について、そのR/G/Bチャンネルが保持する帯域内でアンミックス処理によりクランプされた値 — すなわち、そのフレームの放射度／反射率の積にスタンプされるはずの値（統合インデックスについては、各構成カメラの値）が、`Chloros:Band{R,G,B}UnmixClampFraction`として保持されます。 この実行では、製品ごとの最悪のフレームが `Clamp:` 行に出力されます。`--save-tiffs` は TIFF 用のピクセル単位のマスクを追加し、 TIFFファイルの`<level>/`フォルダ横の`clamp_masks/<level>/`に追加します。**ビルドが途中で中断されても、破損した動画は残りません。** 各動画は `<name>.partial.avi` として書き込まれ、完了して初めて最終的な名前が付けられるため、その名前の古い動画は、置き換えられる動画が完全になるまで残ります。残された `.partial.avi` を置き換えるには、ビルドを再度実行してください。

実行時には、書き出されたすべての動画が表示され、その後、`Skipped:` 形式の出力結果（要求されたカメラ／レベル、または複合インデックスにおいて動画が生成されなかったもの。その理由として、例えば「キャリブレーションバンドルの欠如」、 反射率用のDLS測定値がない、または複合インデックス用の赤/NIRバンドペアがないなど）および`Note:`を出力します。放射度または反射率が要求されたRGBカメラには、代わりに表示用動画が提供され、カメラのキャリブレーションバンドルが利用可能な場合はホワイトバランスが調整されます。 キャリブレーションバンドルなしでレンダリングされたカメラは、`Note:`で指定されます。要求されたプロダクトがスキップされた場合、または要求通りに構築されなかった場合（設定されたインデックス式がオフラインで評価されない場合）、**非ゼロで終了**します。構築された動画は引き続き書き込まれ、一覧に表示されます。

> **適切なレコーダーを選択してください。** `array-record`は*モニタリンググレード*です。これは、レンダリングされたライブコンポジット（GUIの統合ビュー、またはGUIが開いていない場合はバックエンド独自の NDVI（または `--expr`）ビューを記録します。`array-burst` → `array-build-video` は *分析グレード* です — フルレートで生センサーデータを保存し、その後、ライブビューを必要とせずに、校正済みの放射輝度／反射率／インデックスの動画を再構築します。

### モノクロ（M3M）シングルバンドカメラ

**M3M**シリーズは、ベイヤー方式の**M3C**のモノクロ版にあたります。カメラごとに 1 つの狭帯域干渉フィルター（`M3M-<lens>-F<wavelength>`、例：`M3M-L87-F685`）が搭載されており、 そのため、センサーはベイヤーモザイクのない**単一のグレースケールバンド**を出力します。デモザイク処理の必要も、チャンネル間のクロストークを分離する必要も、ホワイトバランスの設定も一切ありません。RGB表示のカラー処理パイプライン全体が、そもそも適用されないのです。

CLI上での意味は以下の通りです：

- **`lattice white-balance`、`lattice color-profile`、`lattice color`**はモノクロカメラを検知し、無意味な設定を適用する代わりに**1行のメッセージを表示してスキップ**します。 同じセッション内でRGB/ベイヤー方式のM3Cカメラに対しては、通常通り動作します。
- **ベイヤー方式のピクセルフォーマットは、その深度を意味します。** M3Mはモノクロ形式のみを提供するため、`--pixel-format`や`--preset`からのベイヤー形式は、同じ深度のモノクロ形式を設定します：`BayerRG8`および`--preset high_speed`からは`Mono8`が生成され、`BayerRG12`、`high_quality`、および`triggered`からは`Mono12` となる。 パックされた名前はパックされたままとなる（`BayerRG12p`は`Mono12p`となる）。ファイルおよび`lattice format`はMono形式を指定する。M3CカメラとM3Mカメラが混在する配列は、 各M3Cにはベイヤー、各M3Mにはそのモノクロ深度が割り当てられる。名前はそのビット深度に等しい。逆もまた真であり、 したがって、M3Cがモノフォーマットに配置されることは決してありません（`--pixel-format Mono12`は、配列のマスターとなるカメラがどちらであっても、M3C上では`BayerRG12`となります）。 また、`lattice format --set`も同様にマッピングされ、設定されたフォーマットで出力されます。
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** は依然として機能します。放射輝度と反射率は*バンドごと*の放射測定マップであり、1バンドに対しては完全に明確に定義されています。モノクロフレームは**恒等**センサー応答行列（3×3のアンミックスなし）を保持しているため、この平面はキャリブレーション計算を無変更で通過します。
- **単一のモノクロカメラでは植生指数を算出できません。**NDVI／NDRE／その他には、少なくとも2つのバンド（例：Red＋NIR）が必要です。モノクロハードウェアから指数を得るには、異なる波長の**複数の** M3Mカメラを向け、それらを1つのマルチバンドスタックとしてアラインし、*その*スタックから指数を算出します：

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live`はカメラを独自に起動するため（[一時セッション](#multi-camera-transient-sessions)）、最初は何も接続しないでください：`--serials`。そうしないと、プロファイルで有効化されたカメラが （[インデックス / 植生計算](#index--vegetation-maths)を参照）。Chlorosがカメラ（GUI、`cam-connect`、`array-connect`）を保持している間 、HTTP 409 `CAMERA_HELD` で拒否され、`1` で終了します。`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all` でそれらを解放してください。

`--channel` シンボルは、プリセットのチャンネル名と **完全に** 一致している必要があります（大文字と小文字を区別します。NDVI は小文字です `red`、`nir` — `--list-presets`を参照）、またバンド名は、アラインされたスタック内のバンドを指定する必要があります（オフラインモードでは、0を起点とするバンドインデックスも使用可能です。例：`--channel red=0 --channel nir=1`）。

スタック全体を通じての識別子は、モデル文字列内の `M3M` トークンであり（`M3C` 文字列には決して出現しません）、GUI/SDK には `is_mono` として表示されます。

---

## ホストNICの設定と調整（LATTICEアレイ）

LATTICEカメラはホストのイーサネットアダプタを介してGVSPをストリーミングするため、マルチカメラアレイの場合、アダプタの**ドライバ**および**受信リングサイズ**は、リンク速度と同様に重要です。設定が誤っていると、アレイ設定パネルで `FRAMES WILL DROP` / `Reduce ROI to enable` ゲートとしてアレイ設定パネル（および `lattice network-analysis` / SDKの `analyze_array_network()`）に表示されます。これは、カメラ自体が正常に動作している場合でも発生します。

### USB 10GbE アダプタ — Realtek RTL8157（「Realtek USB 10GbE Family Controller」）

以下の設定は、Windowsのドライバのプロパティです。Linuxでは、USBアダプタにも帯域制限が設定されます（x86_64では400 MB/s、 NVIDIA Jetsonでは200 MB/s）という制限が設けられており（[配列のfpsとバーストモデル](#array-fps--burst-model)を参照）、すべてのアダプタにおいてカーネルのソケット受信バッファが重要となります — [Linuxホスト](#linux-hosts--socket-receive-buffers)を参照してください。

| 項目 | 必要な値 | 重要な理由 |
| --- | --- | --- |
| **ドライババージョン**|**v10.67 以上 (2026年1月)**, INF `rtump64x64sta.inf` | レガシーな**2016年**版ドライバ (v10.65, `rtump64x64.inf`）は、シャットダウン／再起動／スリープ時に**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**による電源切断やバグチェックを不適切に処理します。 移行処理がハングし（約5分のタイムアウト）、ユーザーが強制的に電源を切ると、不適切なシャットダウンが繰り返されることで**WMIリポジトリが破損**し（PowerShellやツールが`Invalid class`エラーで動作しなくなる）、**USBスタックが固着** （NICが有効化されず、USBドライブの列挙が停止する）。正常な再起動を期待する前に、realtek.com（またはドングルベンダー）から更新を行ってください。 |
| **受信バッファ**— キーワード `ReceiveBufferLen` |**256**（ドライバの最大値） | NICのRXリング。ドライバのデフォルト値である**32**では、使用可能なリング容量が約0.26 MBしか残らない — マルチカメラのバーストには明らかに小さすぎる — ため、アレイパネルは`Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB`を報告し、接続をブロックする。**256**ではリングは十分に大きく（**ラボの 10GbE ホストで測定した結果、約 13.5 MB**）、RX パイプラインにマルチカメラ GVSP バーストのための十分な余裕が生まれます。（特定の構成が実際に *接続* できるかどうかは、**ドレインを考慮した**アドミタンスチェックと**集計オーバーサブスクリプション**チェックの2つのチェックによって決定され、単純なバースト対リングの比較によるものではありません；[アレイのfpsおよびバーストモデル](#array-fps--burst-model)を参照してください。) |
| **受信URB**— キーワード `PendingReceives` |**64** (最大) | 処理中のUSBリクエストブロック。バースト吸収のため、受信バッファと併せて増やす。 |
| **ジャンボフレーム** — キーワード `*JumboPacket` | **9014** | 9000バイトのGVSPパケットに必要（1500バイトの場合に比べて、フレームあたりのパケット数が6分の1）。 |

> ⚠️ **NIC ドライバの更新により、これらの高度なプロパティはデフォルト値にリセットされます。**アダプタドライバの更新または交換後は、**再度適用**してください `ReceiveBufferLen=256` および `PendingReceives=64` を**再適用**してください。そうしないと、「ハードウェアに変更はない」にもかかわらず、アレイパネルが再びゲート状態になります。これは、以前は正常に動作していたリグが突然接続を拒否する最大の原因です。**管理者権限**で実行したPowerShellから適用してください（アダプタ名（例：`"Ethernet 5"`）を置き換えてください）：

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix`はUSB 10GbEアダプタに対応しています。**これにより、アダプタの種類が検出され、適切な受信リングキーワードが設定されます：PCIe NIC（Intel I219、 など）、またはRealtek**USB** 10GbEコントローラ（`*ReceiveBuffers`を公開しないもの）の場合は、`ReceiveBufferLen`→256 + `PendingReceives`→64となります。。設定値は各ドライバが報告する最大値（`NumericParameterMaxValue`）に制限されるため、範囲外の値が書き込まれることはありません。 **管理者権限**のターミナルから実行してください。レジストリベースの調整と同様に、変更はアダプタの再起動またはシステムの再起動後に有効になります。上記の手動による `Set-NetAdapterAdvancedProperty` コマンドも有効な代替手段であり、再起動なしで即座に適用されます（アダプタを再バインドします）。

### Linuxホスト — ソケット受信バッファ

Linux（NVIDIA Jetson を含む）では、カーネルのソケット受信バッファは、NIC の受信リングと同様に重要です。 カーネルのデフォルト設定（`net.core.rmem_max` / `net.core.rmem_default` = 212,992 バイト）では、カメラからの GVSP パケットのバーストにより、Chlorosがそれを読み取る前にストリームソケットがオーバーフローしてしまいます。 NIC自体はパケット損失を報告しないものの、パケットはカーネル内でドロップされ、アレイはフレームグループを失い、フレームレートを低下させてしまいます。 USB 10 GbE アダプタの背後に接続された 4 台のカメラからなるアレイで測定したところ、両方を 32 MB に引き上げた結果、接続後のアレイのフレームレートは 4.96 fps から 7.52 fps に、連続生キャプチャは 4.7 キャプチャ/秒から 7.6 キャプチャ/秒に上昇し、カーネルでのパケットドロップは発生しませんでした。

- **.deb パッケージがこれらの設定を行います。** パッケージをインストールすると、`/etc/sysctl.d/60-chloros-gige.conf`（両方とも `33554432`）が書き込まれ、即座に適用されます。 すでにそれより高い値が設定されている場合は、変更されません。
- **`lattice network` はこれらをチェックします。** いずれかの値がこれより低い場合、*Issues found* の下に `socket receive buffers too small for GigE Vision (…)` が表示され、最初のカメラまたはアレイの接続時にバックエンドのログに同じ警告が記録されます。
- **手動で値を引き上げるには**（次回の再起動まで有効。パッケージのファイルは起動のたびに適用します）：

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` もこれらを上昇させ、維持しますが、これはバックエンドが root として実行されている場合に限られます。ユーザーとして実行されるバックエンド（`chloros-backend` または [`chloros-backend@<user>`X](#running-the-backend-as-a-service-headless))で起動されたもの）として実行されるバックエンドは、カーネルの設定を変更できず、代わりに `sudo sysctl` 行を出力します。

**自動アダプタ調整は、接続済みのアダプタにのみ適用されます。** カメラが最初に接続された際、root 権限で実行されているバックエンドは、受信リングが小さい有線アダプタを調整します。 現在では、リンクが確立されているアダプタのみを調整するため、接続されていないオンボードポートは MTU を維持し、受信バッファは変更されません。`lattice network` は、接続されていないアダプタのMTUを問題としてリストしなくなりました。`lattice network`、`--fix`、`lattice probe`およびアダプタ調整機能は、有線イーサネットポートのみを対象とし、 JetsonのCANバス（`can0`）や、ホストPCに提供するUSBデバイスモードのリンク （`usb0`、`usb1`）は対象外ですが、カメラ用のUSBイーサネットアダプタは依然として対象となります。

### ネットワークの基礎（すべてのLATTICEリンク）

- **アドレス指定：** リンクローカル `169.254.0.0/16`（GigE Vision LLA）。ホストは静的アドレス `169.254.x.x/16` を割り当て、カメラおよび DAQ-E は同じ範囲内で自動割り当てを行います。 DHCPやゲートウェイは不要です。
- **パケットサイズ：**ジャンボ（9000）を推奨しますが、自動プローブに検出させます。GVSPプローブにより、カメラの1500バイトICMP上限を無視して検出するため、回線が実際に伝送可能な場所であればどこでもジャンボサイズになります。**各カメラは、バックエンドセッションごとに1回測定され、リンクが変更された場合（別のアダプタ、別のリンク速度、またはスイッチの切り替えなどによるリンク切断）にも再度測定されます。再接続時には測定結果が再利用されます。スイッチの背後で変更があったもののスイッチ自体が稼働し続けている場合は、バックエンドを再起動して再度測定を行ってください。**このチェックでは、初回接続に数分かかることがあり、その場合はログにその旨が記録されます。プローブよりも正確な情報を把握している場合にのみ、`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`で固定し、恒久的な設定よりもコマンドごとの設定を優先してください。固定するとプローブがスキップされるため、 そのため、経路が実際に9000を伝送できない場合、**すべての**キャプチャで`SC_ERR_TIMEOUT -1011`によるタイムアウトが発生します（[環境変数](#environment-variables)を参照）。
- **RXリングは`ReceiveBufferLen`に比例して拡大します：**デフォルトの `32` では、使用可能なリング容量は約 0.26 MB です（マルチカメラバーストには小さすぎます）。最大値の `256` では、 （ラボの 10GbE ホストで測定した結果、約 13.5 MB）となり、実質的な余裕が生まれます。設定が接続可能かどうかは、ドレインを考慮したアドミタンスチェック**および** 以下の集約オーバーサブスクリプションチェックによって決定され、単純なバースト対リングの比較によるものではありません。

### アレイのfpsおよびバーストモデル

「アレイ設定」パネル（および`lattice analyze-array`／SDKの`analyze_array_network`）の読み方：

- **バーストは、各カメラの実際のピクセルフォーマットに基づいてカメラごとに合計されます。**モノクロ**M3M**カメラは**Mono12 (2 B/px)**をストリーミングします。または、`--pixel-format` を使用する場合は、その深度のモノクロフォーマットでストリーミングします （`BayerRG8`は`Mono8`であり、 1 B/px）のモノクロフォーマットでストリーミングされます；**M3C**ベイヤーカメラは 8 ビットまたは 12 ビットでストリーミングします（M3C ファームウェアは、BayerRG8 が要求された場合でも、黙って BayerRG12 を出力します）。したがって、4台のカメラによるフル解像度のフレームは、**すべて8ビットの場合は約12.6 MBですが、12ビットのモノカメラが3台の場合は約25 MB**になります。 この投影処理は、各カメラのモデル情報（アイデンティティキャッシュ）からフォーマットを判別するため、バーストデータは「一律BayerRG8」という仮定ではなく、実際に通信ラインを流れるデータと一致します。
- **USBイーサネットアダプタは、銘板に記載されている仕様に関わらず、転送速度は200 MB/sが上限です。** リンクレートを持続転送速度に変換する効率表はPCIeに基づいており、USB NICは*イーサネット*のリンクレートを広告しているものの、実際にはUSBバスとそのドライバによって制限されています。かつて使用されていたUSB 10GbEドングルは、約1063 MB/s 「持続」——実際には検証されたことのない数値——を記録していましたが、その結果生じたペーシングにより、フレームの6～18％が破損していたにもかかわらず、正常な目標fpsが報告されていました。USB接続のNICは現在、絶対値として上限が設定されています：WindowsおよびNVIDIA Jetsonでは**200 MB/s**、 Linux x86_64では**400 MB/s**に制限されています（Linuxでは、USB 2.0ポートに接続されたアダプターの割り当ては**30 MB/s**となっており、機能情報にはUSB 3ポートへの接続が推奨されています） （この制限はホスト側のUSBバスおよびドライバによるものであり、定格値に応じてスケーリングされるわけではない。USB 1 GbEアダプタは約80 MB/sの転送速度を発揮し、この制限の影響を受けない）。Linuxでは、アダプタはsysfsから識別される（`ethtool -i`の`bus-info`にフォールバックする）。各数値は実測値です：Windowsでは、同じ4台のカメラアレイを使用した場合、200 MB/sを超えるとフレームの6～10％に破損が見られました；Linux x86_64では、同じアダプタが480 MB/sまでのあらゆる上限速度で問題なく動作しました（273 MB/sが実測値で、これはカメラ自体の制限値です）。NVIDIA Jetson上では、同じアダプタは200 MB/sでのみ正常に動作し、それ以上の速度では断続的にフレームを損失した。機能記録の`wire_ceiling_source`にはその旨が明記されており、`nic_is_usb`ではフラグが立てられている。いずれの場合も、`--wire-ceiling-mbps`で上書きできます。
- **アドミタンスはドレインを意識しており、バースト全体対リングの比較ではありません。** 同時バーストは、バースト全体ではなく、*過渡的なバックログ* = `max(0, Σ per-cam arrival − host drain) × emit_window` に収まればよい。高速ホスト／低速カメラのファブリック（**PCIe**10Gホスト + 4× 1 GbE CAM：到着 ≈ 320 MB/s、ドレイン ≈ 1063 MB/s）では、ホストのドレイン速度がカメラのフィル速度を上回るため、バックログ ≈ 0 となり、25 MB のバーストが 13.5 MB のリング容量を超えているにもかかわらず、フル解像度のシミュレーション送信は**許可**される。 Windows上で、同じ4台のカメラを**USB**10GbEアダプタの背後に接続すると、ドレインは1063ではなく200 MB/sとなる。到着がドレインを上回り、ロスはフレームレートの低下ではなく、破損したフレームとして現れる。 1 GbEホストでは、カメラの31.25 MB/sというDLThrの下限値により、到着データがドレイン処理能力を上回ります → システムは正しく**ブロック**します（*この*種類のブロックについては、ROIを縮小するか、ビンニングを2以上にしてください）。 アドミタンスは**2つ**ある接続ゲートのうちの1つであり、もう1つは後述の集計オーバーサブスクリプションチェックである。
- **予測fpsは、最も遅いカメラのサイクルに準じます。**同時エミットでは、すべてのカメラが同時にフレームを送信するため、アレイのサイクルは、最も遅いメンバーのセンサー読み出し時間とエミット時間のうち、長い方に準じます（フレームバイト数 ÷ そのカメラのエミットレート：カメラの**アクセスリンク**、1 GbE ≈ 80 MB/s、またはホストのワイヤ上限がそれより小さい場合はそのシェア）。各露光は前のフレームの読み出しと重なるため、露光はサイクルに含まれません： 自動露光は、レートによって定められた周期を埋め尽くした後、ゲインを上昇させ、暗いシーンでもレートが低下しなくなる（バージョン1.2.0では低下していた）。その周期よりも長い手動露光のみがトリガーを遅らせる（[アレイ構成要素での露光](#exposure-on-an-array-member)を参照）。カメラを追加しても、それらのワイヤ共有幅が縮小するまではアレイの速度は低下しません。シリアルエミット（`slip-emit-and-capture`）では、代わりに各カメラのエミット時間が合算され、各メンバーの露光時間はスタッガに織り込まれます。 PCIe 10 GbE ホストを搭載した 9 台のカメラアレイで、フルセンサー、12 ビットで測定した結果：10.7 fps、または [パック転送](#packed-12-bit-transport---packed) を使用した場合 14.0 fps。
- **連続キャプチャ終了後、プロジェクションにはその間の処理状況が表示されます。** 自動レートで30秒以上キャプチャした場合、このコンピュータ上のそのアレイサイズで安定したレートが記録されます。レートコントローラによる高速レートでの短時間の試行は除外されます。 それ以降、`array-connect`の行および「アレイ設定」パネルでは、そのレートがモデルのレートを下回る場合、そのレートと制限要因が記載されます：`clock-bound`（アレイは安定したレートで動作し、 グラブ・ループがトリガクロックを待機中）、`loop-bound`（グラブ・ループの時間が残っていなかった）、`capture-bound`（キャプチャが遅れ、グループをスキップした）、または`measured` （原因が特定されていない）。モデルの数値は `wire_fps_max` に示されている。配線が伝送できる値まで、配列は依然として上昇し続ける。
- **配列はどのネットワークでも動作する — 制限となるのはフレームレートだ。**各カメラは、GevSCPDに合わせて調整された**衝突防止の回線上限**（`sustained × sim_emit_factor`）を均等に割り当てられ、すべてが同じトリガーで露光を行う。 低速な回線では割り当て量が少なくなり、それに伴ってフレームレートも低下します。ジャンボフレームを使用しない1 GbE回線上で、フル解像度の12ビットカメラ8台を接続した場合、各カメラの帯域幅は52 / 8 = 6.5 MB/sとなり、**約1 fps**（パック処理時は約1.3 fps）となります。一方、10Gホストでは11～14 fpsで動作します。カメラをそれほど低速にペース制御できない場合（各カメラのGevSCPD範囲を読み込んで確認してください）、アレイは**シリアル**送信モードに切り替わります (`slip-emit-and-capture`) モードに切り替わります。この場合、カメラはフルリンクレートで順番に通信線を使用し、次々とトリガーされるため、すべてのフレームは到着しますが、メンバーは**同時に公開されません**。 `analyze-array` および connect の出力はモードを示します。フォールバックを拒否するには、`--force-tier sim-capture-sim-emit` をピン留めしてください。フレームサイズはこの共有には影響しないため、**ビニングや ROI の縮小はフレームレートを向上させますが、アレイが収まるかどうかに変化はありません**。 シリアル伝送ですら配列を伝送できないワイヤのみが**オーバーサブスクライブ**状態となります（例：1本のワイヤが処理できる容量をはるかに超えるカメラ数、またはそのワイヤ上でストリーミング中の他の多くのカメラなど）。その場合の症状は、レート低下ではなくGVSPパケット損失です： `analyze-array`は達成可能なfps値をゼロにし、`**OVER-SUBSCRIBED**`を出力します。接続は大きな警告を伴って進行し、HTTP接続では`pin_resolution: true`（厳格モード）が**拒否**されます。対処法としては、カメラの台数を減らす、エンドツーエンドでジャンボフレームを使用する、またはより高速なNICを使用することが挙げられます。`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`は、ベンチマーク作業においてこの拒否を「大きな警告」に格下げします — [環境変数](#environment-variables)を参照してください。

### アレイの健全性 — どのサブシステムでフレームが失われているか

接続されたアレイの `GET /api/camera/array/<array_id>/capability` には、
`health` ブロックが含まれており、**10 秒**のローリングウィンドウで再評価されます。 これにより、フレーム損失は、
原因を特定せずに単一の「不完全」
レートを報告するのではなく、対照的な修正を必要とする2つの原因に
分類されます：

| 項目 | 意味 | 対象サブシステム |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (シリアルごと) | フレームが**到着したが、構造的に不正だった**— GVSPパケット損失。 |**ネットワーク**：ワイヤ予算、ペーシング、NIC RXリング、MTU |
| `never_arrived_rate_pct`（シリアルごと） | フレームが**まったく到着しなかった**— カメラが起動しなかったか、何も送信されなかった。 |**トリガー／同期**：M8ケーブル、`--line`、`TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 各カメラの最低レート。 | — |
| `per_cam_rate_pct` | カメラごとの不完全レート合計（両方の原因を合わせたもの）。 | — |
| `stable_for_seconds` | 各カメラが 0.01 % 未満の状態を維持していた期間。 | — |

5％を超えると、バックエンドはスプリット名を記載した `[array-health <id>] WARN` 行をログに記録します。
これは、最初の違反時、重大度帯の変更時、状態が継続している間は1分ごとに、そして状態が解消された際に1回記録されます。
破損した半分のシステムは、各カメラおよび原因ごとの最初のヒット時に `[gvsp-corrupt <SN>]` をバックエンドのログファイルに書き込み、 破損した半分の処理は、カメラごとおよび理由ごとの
最初のヒット時に `[gvsp-corrupt <SN>]` をバックエンドログファイルに書き込み、その後 60 秒ごとに集計値を記録します。アプリのログパネルには、代わりに
`[array-health <id>]`という行が表示されます。すべての評価結果は依然としてバックエンドのログファイルに記録され、
出力内容にかかわらず、バッファごとにカウンタは更新されます。

同じレコードには、割り当て全体が依存している数値が報告されています：

| フィールド | 意味 |
| --- | --- |
| `wire_ceiling_mbps` | ホストの有効な持続ワイヤ帯域幅（MB/s）。 |
| `wire_ceiling_source` | その数値の出典（文字列） — 例：`USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` （Linux x86_64 では `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`）または `user override 120 MB/s (auto said 200)`。 |
| `wire_ceiling_is_user_set` | `true`（ただし、`--wire-ceiling-mbps`（またはGUIの**Wire Budget**フィールド）で設定された場合）。 |
| `nic_is_usb` | USBイーサネットアダプタの場合は`true` — 上記のUSB制限を参照。 |

**解釈：** `gvsp_corrupt_rate_pct` が 0 以外で、`never_arrived_rate_pct` が 0 の場合
は、トリガーと同期ケーブルの状態が完璧であり、損失の100％がネットワーク
経路に起因していることを意味します — `--wire-ceiling-mbps`を下げて再接続してください。逆のパターンは、
同期ケーブルまたはトリガーラインに問題があることを示しています。

> **`--target-fps`は、フレーム破損の決定要因ではありません。** GevSCPDのペーシングは
> 接続時に一度だけ設定されるため、トリガーレートを下げてもデューティサイクルは変化しますが、
> 同時送信バーストレートは変化しません。測定上、要求量を5倍削減しても改善は見られませんでした；
> ワイヤのシーリングを240 MB/sから200 MB/sに下げたところ、同じリグの
> 破損率が10.4 %から0.00 %に低下しました。

> **LATTICE (M3C / M3M) ファームウェアでは、稼働中のアレイは自身で再計画を行うことができません。** そのため、
> 自身でこの問題を修正することはできません。接続を切断して再接続し、接続時間ピッカーが
> 新しい上限値に基づいて再計画を行えるようにしてください。

### 症状 → 対処法

| 症状 (アレイ設定 / 接続 / `analyze_array_network`) | 原因 | 対処法 |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`、 `Reduce ROI to enable` | `ReceiveBufferLen`が32にリセットされる（通常はドライバ更新後） | `ReceiveBufferLen`→256に設定し、`PendingReceives`→64; パネルを再起動（バックエンドが古いリングサイズをキャッシュしている場合は再起動） |
| 再起動/シャットダウンでハング; その後 `Invalid class` WMI エラー、 NICが有効化されない、USBドライブが認識されない | 旧式の2016年製Realtek USB 10GbEドライバ → BSOD `0x9F` → 強制電源オフ | アダプタドライバをv10.67以上 (2026) に更新し、上記の受信リング設定を再適用 |
| アレイはフルセンサーで接続されるが、フレームレートが低い | 「自動」設定ではフルセンサーを維持し、レートは回線の許容範囲まで低下する | 接続出力リストにある高速なオプションのいずれかを選択する（`--binning 2` / `4`、またはより小さい`--width`／`--height`）のいずれかを選択するか、ジャンボフレームを有効にするか、リンクをアップグレードしてください |
| Arrayが正常な目標fpsを報告しているにもかかわらず、その一部しか提供されていない場合；`health.gvsp_corrupt_rate_pct`が0以外、 `never_arrived_rate_pct` 0 | ホストの推定ワイヤ・バジェットが、実際に維持できる値よりも過大に算出されている（USBイーサネットアダプタ、帯域の狭いPCIeレーン、または共有ファブリックでよく見られる現象） | `--wire-ceiling-mbps`の値を低く設定して再接続し、ヘルスブロックを再確認してください。 **`--target-fps` ではない** — GevSCPD のペーシングは接続時に固定される |
| Linux：NIC がドロップを報告していないにもかかわらず、アレイがフレームグループを喪失し、フレームレートを低下させている； `lattice network` が `socket receive buffers too small` を一覧表示 | カーネルのソケット受信バッファがデフォルト設定であるため、カメラパケットがカーネル内でドロップされる | `net.core.rmem_max` / `rmem_default` を 32 MB に引き上げ — [Linuxホスト](#linux-hosts--socket-receive-buffers) を参照。.deb パッケージはインストール時にこれを実行します |
| 公開グループにカメラが登録されていない；`health.never_arrived_rate_pct` が 0 以外、 `gvsp_corrupt_rate_pct` が 0 | トリガー／同期パス — カメラが作動していない（ネットワークの問題ではない） | M8 同期ケーブルと `--line` を確認し、すべてのメンバーが武装状態であることを確認する (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` が `analyze-array` で上限を超えているか、厳格モードでの接続拒否 (`pin_resolution`; `array over-subscribes the wire`) | カメラが到達するどのペーシングにおいても、配線が配列を処理できない — 配線が共有できる容量をはるかに超えるカメラ数、またはその配線でストリーミング中の他のカメラが存在する | カメラ数を減らす、エンドツーエンドでジャンボフレームを使用する、またはより高速なNICを使用する。 **ROI/ビニングでは解決しません**（共有はフレームサイズに依存しません）。`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`はベンチマーク上で優先されます（パケット損失を受け入れます） |
| 低速ネットワーク上では、`slip-emit-and-capture`（シリアル）でアレイが接続される | カメラの動作間隔を、回線を同時に共有できるほど十分に遅くすることはできないため、カメラは順番に動作する。フレームは同一の瞬間に露出されない | ジャンボフレームまたはより高速なNICを使用すれば、同時キャプチャが回復する； `--force-tier sim-capture-sim-emit`はこれを強行し（その結果、パケットを損失する） |

---

## `chloros-cli daq`

スペクトルセンサーのコマンド。 `pool-*` コマンドは、バックエンドの永続プールを通じてセンサーを駆動する簡易な HTTPクライアントであるため、バックエンドが実行されている必要があります（[バックエンドの起動](#start-the-backend)を参照）。バックエンドがトランスポートを管理しているため、CLIおよびSDKスクリプトは、シリアルポートを奪い合うことなく、1つのアクティブなハンドルを共有します。`pool-discover`は、DAQ-MのBLE MACを取得する唯一の方法です。

> **`chloros-cli daq --help`**（または `-h`、あるいは `daq help`）は、`pool-*` のサブコマンド一覧を表示し（Linuxでは、ビルドに含まれるハードウェア直接操作コマンドも併せて表示されます）、 また、`chloros-cli daq <subcommand> --help`（または `daq help <subcommand>`）は各サブコマンドのオプションを示しています。これらは、接続、 ストリーム、校正済み `.daq` ファイルの記録、キャッププロファイルの交換、DAQ とカメラの紐付け、および DAQ-E の場合のネットワーク設定、PTP、再起動、ファームウェアの更新などが含まれます。 また、DAQはPythonから`chloros_sdk.connect_daq_sensor()`を使用して制御することも可能です。これには同じプールされたパスが使用されます。
>
> **すべてのプラットフォームで`pool-*`コマンドを使用してください。** これらはWindows、Linux、Jetsonで同じように動作します。Windowsでは、 `daq discover` および `daq list` は `pool-discover` の別名であり、`daq` 以外のすべての `pool-*`以外の`daq`コマンドは、`1`を`DAQ direct-hardware commands require the full daq package`で終了させます。LinuxおよびJetsonでは、 `daq discover`、`daq list`、およびその他の`pool-*`以外の`daq`コマンドは、DAQツールの直接ハードウェアコマンドです。これらは、CLI独自のプロセス内でセンサーを開き、 バックエンドのプール外（つまり、バックエンド、GUI、またはスクリプトが保持しているセンサー上ではない）でセンサーを開き、`daq discover` では `--only` および `--timeout` は受け付けますが、`--json`はサポートされません。このリファレンスでは、`pool-*`コマンドのみを説明しています。

### DAQセンサーの初回接続ワークフロー

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` リファレンス

| サブコマンド | 目的 |
| --- | --- |
| `daq pool-connect` (smart-detect) | バックエンドプール内のセンサーを開きます。 |
| `daq pool-connect --port PORT` (`-p`) | 特定のシリアルポート上のDAQ-U。 |
| `daq pool-connect --ble` | BLE経由のDAQ-M、MACアドレスの自動スキャン。 |
| `daq pool-connect --mac MAC` (`-m`) | 既知のMACアドレスでのBLE経由のDAQ-M (`--ble`を前提とする)。 |
| `daq pool-connect --eth-host HOST` | 既知のホスト名またはIPアドレスでのイーサネット経由のDAQ-E（`--eth`を前提とする）。 |
| `daq pool-connect --eth` | イーサネット経由のDAQ-E、ホストの自動検出（mDNS + ARPフォールバック；WindowsおよびLinuxのARPキャッシュが空の状態でも動作）。 |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | 積分ウィンドウ／AE状態の調整。 |
| `daq pool-connect --no-stream` | 接続するが、まだストリーミングを開始しない （`pool-stream --start`で再開）。 |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | キャップ補正プロファイル。バックエンドのデフォルトは`sunshine_cosine`です； `none`はセンサー本体のみです。[DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)はこれを無視します（固定ディフューザー）。**センサーはキャップなしの状態で出荷されるため、取り付けたキャップに名前を付けてください：** デフォルトは (`cap_id_source=auto_default`)として記録され、処理時に警告が表示されます。これは、日射補正が適用された状態でベア状態のDAQ-Uを使用すると、測定値が約30倍高くなるためです。 |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 接続せずに、接続可能なセンサーをすべてのトランスポートでスキャンします。 **これがDAQ-MのBLE MACアドレスを確認する方法です。** `--timeout`はトランスポートごとに設定されます（デフォルト5秒、1～20）。プール内で既に開かれているセンサーは一覧に表示されません（接続されたDAQ-Mはアドバタイズを停止するため）。それらの場合は`pool-list`を使用してください。 `usb`はUSBシリアルデバイスのみを一覧表示します。コンピュータ自身のシリアルポート（`COM1`、Linuxの`/dev/ttyS*`）はセンサーにはなり得ないため、除外されます。 |
| `daq pool-list` | バックエンドプール内のすべてのセンサーを表示します。 |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | GUIの光センサー選択機能と同様に、カメラまたは配列をDAQにバインドします： そのキャプチャは、そのDAQの測定値を使用、照合、保存します。`SENSOR`は、`pool-list`からのセンサーID、またはDAQへのアクセス先アドレスです。`--clear` （`SENSOR`はなし）はバインドを解除し、 キャプチャは最新のDAQに切り替わります。バインディングは、バックエンドが再起動するか、そのカメラ、アレイ、またはDAQが切断されるまで持続します。切断（ここではSDKからの切断、またはアプリ内での切断）が発生すると、切断されたハードウェアに対するすべてのバインディングが解除されます。設定した主体に関わらず、アレイのメンバーのバインディングもアレイ自体と共に解除されます。 カメラは、開いているプロジェクト内に接続されているか、またはこのコンピュータ上で以前に開かれている必要があり、アレイも接続されているか、開いているプロジェクト内にある必要があります。それ以外のシリアル番号やIDは名前によって拒否され、何もバインドされません。 |
| `daq pool-disconnect {--sensor-id ID \| --all}` | 解放 — センサーを開いたユーザーに関係なく実行されるため、クラッシュしたスクリプトがプールに残したセンサーも解放されます（そのセンサーでの記録は先に完了している必要があります）。プールにないセンサーには解放すべきものはありません： `ID: already released (not in the pool)` を出力して終了します。バックエンドが拒否した、または確認しなかった解放処理の場合は、`0` で終了します。バックエンドが拒否した、または確認しなかった解放処理の場合は、`1` で終了します。 SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 最新の N 個のスペクトルフレーム（デフォルトは 1）。 |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | ストリーミングの再開／一時停止。 |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | .daq 記録の開始／停止。 `-o` は、バックエンドのコンピュータ上で `~/Documents/DAQ Live View` がデフォルトとなります（相対パスは、コマンドを実行するフォルダを基準とします）。`--duration`（`-d`、デフォルトは `0` = `--stop` まで）。 `--confirm-after S` は、開始から S 秒経過しても測定値が到着しない場合に警告を出します（デフォルトは 20、0 に設定すると無効になります）。`--duration` では、 `--heartbeat S`はS秒ごとに読み取りカウントを出力し、増加が止まった際に警告を発します（デフォルトは30；0に設定すると無効になります）。DAQ-Eでの記録が途絶えた場合、バックエンドが再接続されると同じファイル内で記録が再開され、`--stop`は途切れた部分を一覧表示します（[データ送信を停止したDAQ-E](#a-daq-e-that-stops-delivering))。Ctrl+Cを押すと`--duration`の記録を早期に終了し、その旨を報告します。 `--duration` を指定しない場合、Ctrl+C（`130` を終了）後も、`--stop` まで記録は継続します。 |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 実行時にキャップ補正プロファイルを切り替えます。`--cap-id` が必要です。`--cap-id none` はキャップ補正を無効にします （センサー単体）を解除します。DAQ-E-Sは、設定内容にかかわらず`as_recorded`を維持します。 |
| `daq pool-enable-ptp --sensor-id ID` | PTPがオフになっているDAQ-EのPTPを再びオンに切り替えます。設定はユニットに保存されるため、 そのため、一度オフにしたユニットは、このコマンドを実行するまでオフのままとなり、そのタイムスタンプはPTP同期されません。 `pool-list`は、そのようなユニット（`!! PTP is switched OFF`）にフラグを立て、そのユニットに対してこのコマンドを出力します。Chlorosは、オフ状態が意図的なものである可能性があるため、決して自動的にユニットのPTPを再有効化することはありません。 |
| `daq pool-info --sensor-id ID [--json]` | DAQ-E 本体の設定：ファームウェア（およびこの Chlorosリリースに新しいものが含まれているかどうか）、アドレスとその取得方法 （DHCP／リンクローカル／静的）、センサーが実行するまでの保存済みネットワーク設定（`--no-reboot`に含まれる保存済み設定）、電源投入時のストリーム、およびPTP。ストリームがこのコンピュータに届いていないセンサーについては、新規接続時にセンサーを読み取り、その旨を通知します。 |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | DAQ-EをDHCP （`--auto`）または静的アドレス（`--mask`、デフォルトは`255.255.255.0`）に設定する。センサーは設定を適用するために再起動し、以前の設定で再接続されます。この処理には最大 `--timeout` 秒（デフォルトは 45 秒）かかります。 `--no-reboot` は設定を保存するのみです。[DAQ-E デバイス設定](#daq-e-device-settings)を参照してくださいを参照してください。 |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | ソフトウェアが接続されていない状態で、DAQ-Eが電源投入時にストリーミングを開始するかどうか（センサーに保存されます）。Chlorosはどちらの場合でもストリーミングを自動的に開始します。 |
| `daq pool-reboot --sensor-id ID [--timeout S]` | DAQ-Eを再起動し、以前の設定で再接続する （最大 `--timeout` まで待機、デフォルトは 45 秒）。 |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | 古いバージョンを実行しているセンサーに、この Chlorosリリースに含まれる DAQ-E ファームウェアを書き込み、その後再接続する （約1分間。電源を切らないでください。復旧までのデフォルト時間は `--timeout` 45秒です）。すでに最新バージョンであるセンサーには何も行いません。 |

### データ送信を停止した DAQ-E

プール内の DAQ-E が 20 秒間スペクトルデータを送信しない場合、バックエンドによって再接続されます。これは、センサーに電力を供給するスイッチの電源を入れ直すことで行われます。これによりセンサーが再起動し、指示があるまでデータストリームを再開しません （ただし、`pool-set-auto-stream --on`が設定されている場合は除く）。

- **まず、それが同じセンサーであるかを確認します。** センサーが以前持っていたアドレスに対して、設定を変更することなく、センサー名、MACアドレス、シリアル番号を問い合わせます。 また、センサーの `daq-e-<id>.local` 名でも問い合わせを行いますが、その回答がセンサー自身のネットワークアダプタ（MAC）から送信されたものである場合にのみ、その回答を信頼します。それ以外の回答元に対しては、再接続や書き込みは行われません。
- **セッションの設定（**露光時間、フレーム平均化、自動露出、キャッププロファイル**）を、センサーの電源投入時のデフォルト設定ではなく、以前の状態のまま維持して再接続します。PTPがオンだったセンサーがオフの状態で復帰した場合は、PTPを再びオンに切り替えます。その後、ストリームと、それを受信するコンピュータのレシーバーを再起動します。
- **再試行の間隔を徐々に延ばします：** まず直ちに、次に5、10、20、40秒後、そしてセンサーが接続されていない間は60秒ごとに試行します。 各試行では、センサーに対してその識別情報を問い合わせるのみです。応答はあるものの、3回の再接続後もデータを送信しないセンサーについては、そのまま放置され、その状態が報告されます（`auto-reconnect GAVE UP`）。`pool-stream --start`は再試行を行います。
- **`pool-list`は現在の状況を表示します。** センサーから応答がない間、その`STREAM FAULT`行には再接続の処理状況が表示されます。 最初のスペクトルが表示されるとその行は消え、センサーは `reconnected automatically N time(s)` と読み取られます。
- **記録は同じ `.daq` で継続される。** ファイルは各データ欠落箇所を 2 行でマークする：欠落前の最後のスペクトルで `event_type` 5、欠落後の最初のスペクトルで 6、それぞれにタイムスタンプのみが付く。`pool-record --duration`は、センサーが不在の間に`!! STREAM LOST …`を出力し、戻ってきた際に`stream resumed after …`を出力します。`pool-record --stop`は各ギャップを列挙し、 空のファイルの場合と同様に（測定値が含まれない記録は `1` で終了します）。
- **1つの記録には1つのキャリブレーションが保持されます。** 記録の実行中にセンサーが異なるキャリブレーションで復帰した場合、記録の残りの部分は開始時のキャリブレーションを維持し、停止警告にもその旨が表示されます。 `pool-disconnect` および `pool-connect` を実行すると、新しいキャリブレーションが適用されます。
- **ユーザーのコマンドが優先されます。** `pool-disconnect`で停止させたり、`pool-stream --stop`で一時停止させたりしたセンサーは、たとえ再接続がすでに進行中であっても、再接続されることはありません。
- **応答のないセンサーに対する `pool-info`** は、新規接続時にセンサーを読み取り、`!! stream NOT reaching this computer` を追加します。 応答しないセンサーは直ちに報告されます。`pool-list`は、応答を停止したセンサーのPTP状態を表示しなくなりました。
- DAQ-U および DAQ-M は対象外です。DAQ-M は Bluetooth 経由で自動的に再接続しますが、USB から取り外された DAQ-U には、再度 `pool-connect` が必要です。

### DAQ-E-S — 固定式ディフューザーおよび傾斜センサー

**DAQ-E-S** は、ソフトウェアの観点から重要な2つの違いがある DAQ-E です。それ以外（検出、接続、PTP、記録、以下のデバイス設定など）は、DAQ-E とまったく同じように動作します。**そのディフューザーは固定式であるため、キャップ補正は不要です。** サンシャインスタイルのディフューザーは内蔵されており、工場出荷時の校正測定時にすでにユニットに取り付けられていたため、その校正値はすでに真の下向き放射照度を示しています。Chlorosは、要求内容にかかわらず、そのキャップを`as_recorded`（プロファイルなし）に設定します： このデバイス専用のキャップ名（`sunshine_cosine`、`fov_*`、`none`）が指定されても、メッセージが表示されて無視され、記録データには `cap_id_source=model`となるため、このデータに対しては「想定キャップ」の警告は決して発動しません。また、手動でキャップを適用することも避けてください。上部に`sunshine_cosine`を配置すると、読み取り値が約11倍高くなってしまいます。

**このデバイスには3軸加速度センサーが搭載されています。** サンシャインディフューザーのコサイン応答は、水平な状態でのみ有効です。そのため、センサーが傾いていると、他の症状は現れず、誤った放射照度が報告され、その誤差が反射率にも引き継がれます。したがって、各DAQ-E-Sの測定値にはその姿勢情報が含まれています：

| フィールド | 意味 |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | 角度（度単位）。ファームウェアが角度を未定義（自由落下、デッド軸） |
| `x_mg`、`y_mg`、`z_mg`、 `mag_mg` | 加速度（ミリg）。`mag_mg`が約1000から大きく外れている場合、ユニットが移動していたことを意味し、そのサンプルにおける角度には意味がありません。 |
| `cal_applied` | 設定済み：軸と角度はディフューザーの座標系に基づいています（ユニットの取り付け校正が適用されています）。クリア：ボードの生の座標系。この場合、実際には存在しない最大約2.3°の傾きが表示されることがあります。 |
| `sample_age_ms` | スペクトルが送信された時点での加速度計サンプルの経過時間。 |
| `temp_raw` | キャリブレーションされていないダイ温度の**傾向** — 差分のみであり、温度計の測定値そのものではありません。 |

`daq pool-latest --json`は、各フレームの`imu`キー（DAQ-U、DAQ-M、または通常のDAQ-Eでは`null`）として返されます。DAQ-E-Sから記録された`.daq`は、その`imu_*`列（`imu_tilt_deg`、`imu_roll_deg`、`imu_pitch_deg`、`imu_mag_mg`、 `imu_cal_applied`、…）に同じ値を格納するため、読者は雲と三脚の倒れを区別できます。他のすべてのモデルでは、これらの列は空欄です。

姿勢データは個別に報告される：`daq pool-list`は、ダウン中は`!! ATTITUDE STREAM FAULT (spectra unaffected)`を出力し、その後回復した障害の数を示す。この部分の障害によって、スペクトル記録に欠落が生じることはない。 `daq pool-discover`は、当該ユニットを`DAQ-E-S`としてリストします（約2秒以内にIDチェックに応答しない場合は、`DAQ-E`としてリストされます）。

### DAQ-E デバイス設定

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` に `--gateway` を指定しない場合、ゲートウェイは保存されません**（以前に設定されたゲートウェイはすべて破棄されます）。これは、ルーターのないカメラネットワークでは適切な動作です。
- **このコンピュータが属するネットワーク上に静的IPアドレスが設定されている必要があります。** Chlorosは、センサーのネットワーク上でこのコンピュータのアドレスが含まれていないサブネットを拒否します。再起動後、センサーにアクセスして設定を元に戻すことができません。このコンピュータが新しいサブネットに再アドレス指定されるか、ルーティングされる場合のみ、`--force`を指定してください。
- **センサーが録画中の再起動は拒否されます。** まず録画を停止してください（`pool-record --stop`）。再起動の直前に開始された録画は、再起動によって終了し、 `pool-set-network`、`pool-reboot`、および`pool-update-firmware`は、記録ファイル名を指定する`Warning:`行で終了します （その後、センサーが復帰しなかった場合でも同様です）。
- **再起動後、Chlorosはセンサー本体のみを再接続します。** センサーがあった場所――そして `--auto` の後は `daq-e-<id>.local` という名前でも――で、センサーの名前、 ホスト名、モデルを含む応答を、センサーが以前持っていたネットワークアダプタ（MAC）から待ち、応答があったアドレスで再接続します。 その名前での応答は、そのMACアドレスからのもののみ受け入れられるため、ルーターを経由してセンサーに到達するコンピュータ、またはデフォルトのNATネットワーク環境にあるWSL2からは、DHCPによって新しいアドレスに移動したセンサーは再接続されません： コマンドはその旨を通知し、以前のシャッター速度と露出設定でセンサーを復旧させる`pool-connect --eth-host <its address> …`コマンドを出力します。
- **`pool-update-firmware`は、ファームウェアバージョンを読み取れないセンサーを拒否します。** フラッシュ後に再接続されるまでの時間が待機時間を超えても、センサーは故障していません。応答があれば、新しいファームウェアの設定が反映された状態で再接続され、コマンドは応答が遅かったことを警告します。
- **ファームウェアの更新には、着信TCP接続が必要です。** センサーは画像を取得するためにこのコンピュータに再接続します。Chlorosは、センサーへ接続するアダプタでリスニングを行い（どのアダプタか判別できない場合はすべてのアダプタで）、そのセンサー自身のアドレスに対してのみサービスを提供します。 それ以外の接続は処理されずに閉じられます。Windowsでは、インストーラーの`Chloros Backend (TCP-In)`というファイアウォールルールによって接続が許可されます（[Windows (.exe)](#windows-exe)を参照); これをブロックするファイアウォールでは、`… device never connected back on TCP <port> …`エラーにより更新に失敗します。
- **WSL2 では、ミラーリングネットワークを使用してください**（`networkingMode=mirrored` および `%UserProfile%\.wslconfig`）。 デフォルトのNATモードでは、センサーのネットワーク上のこのコンピューターのアドレスはWSL自身のものとなるため、すべての静的アドレスが拒否され（`--force`を除く）、ライブストリームがWSLに届かず、センサーがイメージを受信するためにコンピューターに再接続するため、ファームウェアの更新を完了できません。
- **センサーIDとホスト名は変更されません。** アドレスが何であれ、DAQ-Eは依然として`daq-e-<id>`および`daq-e-<id>.local`であるため、`--eth-host daq-e-<id>.local`は引き続きそれを検出します。
- **GUIの設定は同じままです**：「Light Sensors」タブ、センサーの設定（歯車アイコン）、ネットワーク。GUIはセンサーとの独自の接続を維持するため、GUIから開いているセンサーの設定を変更してください。 CLIから再起動すると、GUIのそのセンサーとのセッションが終了します。
- SDKにおける対応するIDは、`DAQSensorSession.device_info()`、`set_network()`、`set_auto_stream()`、 `reboot()`、および `update_firmware()` です。

---

## `chloros-cli project`

保存済みの Chlorosプロジェクト （`cameras.json` + `sensors.json` + `project.json` を含むフォルダ）を開き、接続し、実行します。すべてがバックエンドを経由するため、GUIとCLIで生成されるハードウェアの状態は同一になります。

### サブコマンドリファレンス

| サブコマンド | 目的 |
| --- | --- |
| `project open PATH [--relink FOLDER]` | プロジェクトのデバイスマニフェスト（カメラ、アレイ、センサー）を出力し、そのファイルから読み込み、ソースファイルを確認します（[移動されたプロジェクト](#a-moved-project---relink)を参照）。 バックエンドでプロジェクトを開くことはありません。バックエンドで開かれているプロジェクト、およびそのデバイスや録画データは、そのままの状態を維持します。プロジェクトのソースファイルが1つも見つからない場合は、`1`で終了します。 |
| `project devices PATH [--reconnect]` | 検出の一覧表示または再実行。 |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | 保存されているすべてのカメラ／アレイ／センサーに接続します。各アレイは `lattice array-connect` と同様に接続されます： 最大600秒間、その手順はstderrに出力されます（`[array <id>]    <s> s: <step>`；stdoutはJSONレポートです）。 その時間内に接続されなかったアレイは元に戻され（そのカメラは検出時の状態に戻される）、`errors`の下で報告され、接続されたままになることはありません。`--packed`は、各アレイのパックされた12ビット転送を設定します （デフォルトは `auto`；プロジェクトは配列ごとの設定を保存しません；[12 ビットパック転送](#packed-12-bit-transport---packed) を参照）。各光センサーは、保存されたアドレスで、保存された積分時間、フレーム平均化、自動露出、およびキャップ設定とともに、バックエンドのDAQプールで開かれます： キャップが選択されていないセンサーには `sunshine_cosine` が割り当てられ、保存時に「ベア」状態だったもの（`none`）はそのまま「ベア」のままであり、DAQ-E-S はキャップを設定しません。 GUIのDAQパネルはプールの一部ではありません。センサーを閉じる場合は、まずそこで閉じてください。 |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | 指定されたカメラまたはアレイからの単一キャプチャ（`--prefix`がデフォルト、`capture`）。`--orient` （アレイ）：アレイは生データを保存し、そのデータはネイティブ形式のまま保持されます。これにより、`process`がそこから生成するプロダクトは、`array-capture --orient`のようにオリエンテーションが設定されることが記録されます。デフォルト：ネイティブ。 `project burst` / `stream` およびレシピの `capture` / `burst` / `stream` (`orient: true`）は、同じオプションを受け付けます。 |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | 指定されたカメラまたはアレイからのNフレームバースト（`-n/--count` デフォルト 5；`-i/--interval` フレーム間の秒数、デフォルト 0）。 アレイバーストは、重複する同期グループを重複排除（鮮度ウォッチドッグ）するため、部分的に動作しているアレイが 1 フレームの N コピーを返すことができません。 反復ごとの結果を出力します。途中で失敗したキャプチャはバーストを停止し、`1`を終了するとともに、それまでに保存されたフレーム数を報告します。 |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | ループ内でディスクへのキャプチャ：`--fps F`は、 （デフォルトは3）、カメラからのキャプチャごとに1フレーム、アレイからの同期済みフレームグループを1つ（`project burst`と同じ古さウォッチドッグを使用）、`-n N`がキャプチャするまで （デフォルトの `0` は Ctrl+C が入力されるまで。この際、保存されたフレーム数が表示され、`0` が終了する。または、保存されたフレームがない場合やファイル作成に失敗した場合は `1` が終了する。 `-n N`がキャプチャを完了する前にCtrl+Cが入力されると、`130`が終了します）。キャプチャが途中で失敗すると、ストリームが停止し、`1`が終了し、その時点で保存されたフレーム数が表示されます。`--poll-interval`は非推奨であり無視されます： ただし、引き続き受け入れられるため、古いスクリプトは実行されますが、stderrに注意書きが表示されます。 |
| `project sensor read PATH NAME [--json]` | 最新のスペクトルフレーム。 |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | `--seconds`の.daqファイルを記録します （デフォルト 10）用に .daq ファイルを記録します。測定値が含まれていない記録は stderr に報告され、`1` が終了します。 |
| `project run PATH RECIPE.yaml` | YAML/JSONキャプチャレシピを実行します。 `--dry-run` は実行せずに検証を行います。 |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | 配列のアライメントを計算します — [以下のフラグ一覧](#project-align-calibrate-options)を参照してください。 |
| `project align status PATH NAME [--json]` | 現在のアラインメントプロファイルを表示します。 |
| `project align clear PATH NAME` | キャッシュされたプロファイルを削除します。 |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | 1つのスレーブのトランスフォームを微調整します （デフォルトは `0`、`0`、`0`、`1.0`：変更なし）。 |
| `project align export PATH NAME --to FILE` | プロファイルをJSON形式で保存する。 |
| `project align import PATH NAME --from FILE [--no-validate]` | 保存済みのプロファイルを読み込む。 |

#### `project align calibrate` オプション

| フラグ | デフォルト | 説明 |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | アラインメント方法。**これらの表記は、短縮形 `orb` / `akaze` / `phase` といった短縮形を受け付けますが、このフラグに関しては、これら2つのコマンドは互換性がありません。 |
| `--model {translation, rigid, affine, homography}` | `affine` | モデルを適合させるように変形する。 |
| `--frames N` | `1` | 同期されたフレームのスナップショットを 平均化。 |
| `--reference SN` | マスター | 参照カメラのシリアル番号。他のすべてのメンバーはこれにワープされる。 |
| `--max-features N` | `5000` | ORB特徴点数の上限。 |
| `--ratio-threshold F` | `0.75` | ローのs比テスト。 |
| `--ransac-threshold-px F` | `3.0` | RANSACのインライヤー閾値。 |
| `--min-matches N` | `15` | **品質ゲート** — この内点一致数未満の解は拒否する。 |
| `--max-reproj-err-px F` | `4.0` | **品質ゲート** — このRMS再投影誤差を超える解は却下する。 |
| `--checkerboard RxC` | — | `--method checkerboard`のボード形状。例：`9x6`。 |
| `--name PROFILE` | 空 | 保存されたJSONに埋め込まれたプロファイル名。 **配列名ではありません** — それは位置指定の `NAME` です。 |

2つの品質ゲートが存在するため、キャリブレーションが解の算出に成功しても、
保存が拒否される場合があります。いずれかのゲートに失敗したプロファイルは、
その後のすべてのキャプチャで黙って位置合わせの誤りを引き起こすため、
保存されるのではなく拒否されます。

### 移動されたプロジェクト (`--relink`)

プロジェクトの `project.json` は、各ソースファイル（すべての画像および光センサーの
記録）をフルパスで指定しています。プロジェクトフォルダ、またはプロジェクトフォルダ全体を
別のドライブに移動しても、それらのパスは依然として元の場所を指したままになります。

プロジェクトを開くと、アプリが独自に修正できる範囲で修正が行われます。プロジェクトの
フォルダ内にあったファイルは、プロジェクトが現在置かれているフォルダ内で検索され、そこで見つかったすべてのパスは
再指定されて保存されます （`project.json`はまず
`project.json.before-relink-<date>-<time>`としてバックアップされます）。これは、アプリ、
`chloros-cli project open`、 `chloros-cli process`を既存のプロジェクトに読み込む際、SDKの
および`chloros-mcp`の各フォルダについても同様です。パスの再設定は、その場所に存在するファイルに対してのみ行われます。

プロジェクトフォルダの外にあったファイル（ハブセッションや「フォルダの追加」によるインポートなど）は、
その方法では見つかりません。それらのファイルが現在入っているフォルダ名を指定してください：

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink`は、ファイルが移動したフォルダ、またはその親フォルダを指定します。欠落している
各ファイルのフォルダは、末尾のフォルダ名に基づいてその下で検索されます
（`Hub Projects/<session>/raw/<serial>`、次に `<session>/raw/<serial>`、…）、そして各
フォルダ内のファイルは、最も多くのファイルが見つかった場所から取得されます。`project open`は、
再指定された内容、見つかったソースファイルの数、および残りのファイルが
あったフォルダを出力します。 一部がまだ欠落している場合：警告が表示され、`0`が終了します。残りのファイルがあればプロジェクトは正常に開きます。
見つからない場合：それらのフォルダ名と`--relink`を指定するエラーが表示され、`1`が終了します。バックエンドも同様に
、`process`や`project devices --reconnect`といったプロジェクトの開くことを拒否します（HTTP 409 `PROJECT_SOURCES_MISSING`）。

### 例

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### レシピ DSL

`project run RECIPE.yaml` は、一連のアクションを記述した YAML または JSON ファイルを受け付けます：

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

サポートされているアクション：`apply`、`wait`、`capture`、`stream`、 `burst`、`sensor`。 `apply`は、`cameras.json`、`settings`、`exposure_us`、 `auto_exposure`、`gain_db`、`auto_gain`、`target_brightness`、… （旧名称である `exposure_time`、`exposure_auto`、`gain`、`gain_auto` は別名として受け入れられます）。 この方法で設定された露出、ゲイン、またはターゲットは、ユーザー固有の設定として保持されます。Chlorosの自動露出制御（DAQによる測光を含む）は、これらを変更しません。 手動露出の場合、ゲインはそのまま維持されます。自動モードでは、`gain_db` または `auto_gain: "Off"` を設定するまで、カメラはターゲットの輝度に向けて（全範囲にわたって）ゲインを調整し続けます。 その後、`auto_exposure: "Continuous"`を実行すると露出が自動制御に戻り、`target_brightness_auto: true`を実行すると固定されたターゲットがChlorosの自動ターゲットに戻ります。 `burst` アクションは、`name`（必須）、`count`（デフォルト 5）、 `interval`（秒、デフォルト 0）、`output`、`format`、 および `settings`（`apply` と同じカメラごとの設定形状）を受け取ります。アレイバーストは、`project burst` と同じ、新たに同期されたグループのウォッチドッグを使用します。

また、`capture`、 `burst` または `stream` ステップの `output` は、デフォルトで `output` になります。 相対パス指定の `output` （デフォルトで含まれています）は、レシピファイルのフォルダやプロジェクトフォルダではなく、`chloros-cli project run` を実行したフォルダを基準に解決されます（バージョン 1.2.0 では、バックエンドのインストールフォルダに配置されました）。

実行方法：

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**実行のたびに、まずプロジェクトに対してチェックが行われます。**各アクションのキーは、上記の動詞のいずれか1つでなければなりません — `- capture: {name: FrontLeft}` であり、`- action: capture` や `- type: capture` ではありません — 例えば、`name`の場合、プロジェクトの`cameras.json` / `sensors.json`にはカメラまたは配列が、`apply` / `capture`／`burst`／`stream`用アレイ、および`sensor`用の光センサーを備えている； `stream`には`count` > 0が必要であり、`wait`には継続時間が必要です。このようなエラーを含むレシピは、**一切**のアクションを実行しません （`[recipe] error: …`、`[recipe] not run: …`、終了 `1`）、たとえ `stop_on_error: false` が指定されていても同様です。 ステップで使用されていないキーは `[recipe] warning:` であり、レシピが制御する、現在接続されていないすべての配列や光センサーも同様です。 レシピはデバイスを接続することはありません。まず `chloros-cli project connect PATH` を実行してください。`--dry-run` はレシピを出力し、その後同じエラーと警告を表示し、`[recipe] valid: …` / `NOT valid: …` を実行し、何も実行せずに終了します。また、レシピにエラーがある場合は `1` で終了します。

---

## 環境変数

| 変数 | 効果 |
| --- | --- |
| `CHLOROS_BACKEND_URL` | バックエンドのURLを上書きする（デフォルトは `http://127.0.0.1:5000`） — **`lattice`、`project`、および`daq pool-*`コマンドファミリーでのみ有効** （`lattice network-analysis`は代わりに`--backend-url`を受け付けます）。 コアコマンド（`process`、`login`、`logout`、`status`、 `export-status`、`time-sync`、`selftest`）は`http://127.0.0.1:<port>`をピン留めし、この変数を無視します （IPv4のリテラルを使用することで、Windowsの`localhost`→`::1`におけるリクエストあたり約2秒のペナルティを回避できるため）、常にローカルマシンをターゲットとします。 |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | バックエンドの環境変数として設定されます。 `1`は、オーバーサブスクリプションによる拒否を緩和します。つまり、`pin_resolution`を指定した配列接続、または[一時的なマルチカメラコマンド](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — を「大きな警告」に緩和し、GVSPパケット損失を許容します。 テスト目的のみ — [アレイの fps およびバーストモデル](#array-fps--burst-model)を参照。 |
| `CHLOROS_ARRAY_PACKED12` | バックエンドの環境で設定： `on` または `off` を設定すると、すべてのアレイに対して [パックされた 12 ビット転送](#packed-12-bit-transport---packed) がデフォルトとして設定されます （デフォルトは `auto`）のデフォルトとして [パケット化された 12 ビット転送](#packed-12-bit-transport---packed) を設定します。明示的な `--packed` が指定された場合は、そちらが優先されます。 |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` （または `true` / `yes`）を指定すると、起動時に Linux CLI の [自動更新チェック](#chloros-cli-update) が無効になります。 |
| `CHLOROS_CLI_MODE` | CLI自体によって設定され、バックエンドに並列処理を有効にするよう指示します。 |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0`は、GVSPフォールバックプローブをスキップします （ICMPの結果のみ）。**これはジャンボパケットを無効にするものであり、単にログの出力を抑制するだけではありません** — カメラはすべてのパスにおいて1500までのDF pingにのみ応答するため、このプローブだけがジャンボパケットを検出できる唯一の手段です。 接続ごとにカメラ1台あたり約1秒を節約します。ネットワークがジャンボパケットを伝送可能だった場合、コストは回線上限の約1.45倍になります。設定時にSDKから警告が表示されます。 |
| `CHLOROS_GVSP_SESSION_REUSE` | バックエンドの環境で設定：`0`は、カメラ1台あたりバックエンドセッションごとに1回ではなく、接続のたびにパケットサイズを測定します。 |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | GVSPパケットサイズをNバイトに固定します。プロービングを完全にスキップします。恒久的に設定するよりも、コマンドごとの設定（`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`）を優先してください： サイズを固定すると、その先のネットワークへの適応が行われなくなり、ジャンボパケットを伝送できない経路で9000に固定すると、`SC_ERR_TIMEOUT -1011`を使用した場合、**すべての**キャプチャでタイムアウトが発生します。 |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | バックエンドの環境変数で設定：[一時的なマルチカメラコマンド](#multi-camera-transient-sessions)、またはカメラが保持していない `lattice capture` が、フレームを保持する前に前のシーンから各カメラの露出を設定するのに費やす最大秒数（デフォルトは `5`）。 `0` を設定するとこの機能は無効になります。その場合、各カメラの自動露出は、最初のフレームからその露出制限値に基づいて動作します。 |
| `CHLOROS_LOG_DIR` | バックエンドの環境変数で設定：詳細ログの保存先フォルダ。 Windows ではデフォルトで `%LOCALAPPDATA%\Chloros\logs`、Linux では `~/.cache/chloros/logs`（`$XDG_CACHE_HOME/chloros/logs` が設定されている場合はそれ）です。 .deb パッケージのルートサービスが `/var/log/chloros` を設定し、`adm` グループが読み取り可能になります — [バックエンドをサービスとして実行する](#running-the-backend-as-a-service-headless) を参照してください。 |
| `TMPDIR` (Linux) | 一時ファイルの保存先。CLIは、`/mnt/ssd/tmp`が存在する場合、それを自動的に使用します。 |

---

## 終了コード

| コード | 意味 |
| --- | --- |
| `0` | 成功。 |
| `1` | 一般的な失敗（ほとんどのサブコマンドのエラー）。 また、キャプチャコマンドは、他のファイルは正常に保存されたにもかかわらず、書き込みを指示されたファイルのいずれかの書き込みに失敗した場合、`1` で終了し、`array-burst` / `array-record` の場合、レコーダーがエラーを報告した際に `1` で終了します — [書き込みに失敗したファイル](#files-that-fail-to-write) を参照してください。 また、実行のエクスポートが検証できなかった場合や、イメージが失われた場合にも、`process` は `1` を終了します。ただし、他の製品は書き込まれます。 |
| `2` | 引数のエラー（不明なフラグ、またはフラグの欠落。優先度がない場合は `time-sync set-priority` も同様）。 |
| `130` | Ctrl+C による中断：実行を中断したコマンド — `lattice`、`project`、 `daq`、および `process` も同様（バックエンドが実行を停止した後の `process` については、 [`process` 注記](#notes)を参照)。Ctrl+C で通常停止するコマンド — `lattice array-capture --continuous` / `--interval`、`lattice viewer`、`lattice continuous`、`multi-stream`、`align-stream`、 `array-record`、`array-burst`、および `project stream` から `-n 0` まで — 実行結果に応じて終了します： `0` はすべての書き込みが完了した際、`1` は何かが失敗した際に終了する。 |
| `143` / `129` | コマンドのクリーンアップ処理（実行中のジョブが停止し、カメラが元の状態に戻され、バックエンドでの処理が停止）が完了した後、SIGTERM / SIGHUP（128 + シグナル番号）によって強制終了された。 `lattice continuous` や、通常は Ctrl+C で停止するその他の録画も、`143` を終了させますが、`0` は終了させません。 |

`lattice network-analysis`は、判定結果が`needs_force_slip`および`error`の場合、`1`を終了させ、 また、`selftest`は、7つのチェックすべてに合格しない限り、`1`を終了させます。

---

## トラブルシューティングのヒント

- **「ログインが必要です」** → このマシンで `chloros-cli login EMAIL PASSWORD` を一度実行してください。
- **「このコンピュータではプロジェクトフォルダの階層が深すぎます」** → このマシンでは Windows の長いパスが無効になっている（Windows のデフォルト設定）ため、パスは 260 文字を超えてはならず、製品はプロジェクトフォルダから最大で約 130 文字下にある必要があります。 何も処理されませんでした。メッセージには、条件を満たす最長のプロジェクトフォルダが表示されています。より短いフォルダを使用するか（`-o` または `chloros-cli set-project-folder`）、Windowsの長いパスを有効にして Chloros を再起動してください。 Windowsインストーラーには「Windowsの長いパスを有効にする」というオプションがあり、デフォルトでチェックされています。それ以外の場合は、グループポリシーを使用してください：*コンピューターの構成 > 管理用テンプレート > システム > ファイルシステム > Win32の長いパスを有効にする*。
- **「バックエンドに接続できません」** → Chloros デスクトップアプリ（Windows）を起動するか、バックエンドを手動で起動してください：Windows では `"C:\Program Files\Chloros\backend\chloros-backend.exe"`（インストールフォルダの `backend` フォルダ内； `PATH`にはありません）、Linuxでは`chloros-backend`。リモート接続の場合は`CHLOROS_BACKEND_URL`を確認してください。
- **`lattice` コマンドが「LATTICE カメラドライバが見つかりません」というエラーで失敗する** → バックエンドが、インストール時に同梱されている LATTICE カメラランタイムを読み込めませんでした。 すべての `lattice` コマンドはバックエンド上で実行されます。CLI 自体にはカメラドライバは含まれていません。Chloros を再インストールしてください。Windows インストーラおよび .deb パッケージは、バックエンドの隣にランタイムをインストールします。
- **「Array connect」または「Array Settings」で「FRAMES WILL DROP」または「Reduce ROI to enable」と表示される** → ホスト NIC の受信リングが小さすぎます（通常、NIC ドライバーの更新後に 32 にリセットされます）。 [ホストNICの設定と調整](#host-nic-setup--tuning-lattice-arrays) を参照してください — `ReceiveBufferLen=256`、`PendingReceives=64`を設定してください。
- **再起動/シャットダウン時にマシンがハングし、その後 WMI `Invalid class` が表示される／NIC が有効化されない／USB ドライブが認識されない** → 古い USB 10GbE アダプタドライバが原因で `DRIVER_POWER_STATE_FAILURE` が発生 (BSOD `0x9F`)。アダプタドライバを更新してください — [ホスト NIC の設定とチューニング](#host-nic-setup--tuning-lattice-arrays)を参照してください。
- **Linux: キャプチャは root に属しているため削除できず、バックエンドのログが `~/.cache` には存在しません** → バックエンドは root 権限で実行されています（`chloros-backend.service`）。 `chloros-backend@<user>.service` に切り替えてください。このディレクトリでは、バックエンドがユーザーとして実行されます — [バックエンドをサービスとして実行する](#running-the-backend-as-a-service-headless)を参照してください。 それまでは、root サービスのログは `/var/log/chloros` にあります（`adm` グループが読み取り可能です）。
- **Linux: バックエンドユニットが `failed` で、「別のインスタンスがすでに実行されています」（終了ステータス 98）と表示される場合** → 別のバックエンドがすでにそのポートを占有しています。一度に実行できるのは 1 つだけです。 2つのバックエンドユニットが有効になっている場合は、1つだけを残してください。使用しない方のユニットは `sudo systemctl disable --now <unit>` として設定します — [バックエンドをサービスとして実行する](#running-the-backend-as-a-service-headless) を参照してください。 このユニットのみが有効になっている場合、systemd 以外（`chloros-backend`、`chloros-cli`、または SDK スクリプトによって）起動されたバックエンドがそのポートを占有しています。そのバックエンドを停止してから、`sudo systemctl start <unit>` を実行してください。
- **Jetson スワップ警告** → ファイルベースのスワップを追加します。CLIには、正確な `fallocate` / `swapon` コマンドが表示されます。
- **`lattice`、`project`、または `daq` コマンドからの「バックエンドに到達できません」** → バックエンドが実行されていません。これらのコマンドではバックエンドは起動されません。 `chloros-cli backend status` を実行し、[バックエンドの起動](#start-the-backend)を参照してください。
- **「`daq net ...`はChlorosでは利用できません」**（LinuxおよびJetson） → DAQ-E固有の設定の一部は、MAPIRサービス用に予約されています。 ネットワーク設定、再起動、ファームウェアの更新は、それぞれ `pool-set-network`、`pool-reboot`、`pool-update-firmware` で行います。それ以外については、MAPIR サポートにお問い合わせください。
- **「DAQの直接ハードウェアコマンドには、daqパッケージ全体が必要です」**（Windows）→ バックエンドを介して同じセンサーを駆動する`pool-*`、またはPythonからの`chloros_sdk.connect_daq_sensor()`を使用してください。

---

## 関連項目

- [Python SDK リファレンス](sdk-reference.md) — すべての CLI コマンドに相当するプログラム記述。
- [DAQ 光センサー](https://mapir.gitbook.io/chloros/daq-light-sensors)（Chlorosマニュアル内） — センサー、そのキャパシティと校正済み測定範囲、記録、DAQ-E ネットワーク。
- [LATTICEカメラ](https://mapir.gitbook.io/chloros/lattice-cameras/lattice)（Chlorosマニュアル） — 接続、カメラ設定、キャプチャモード、アレイ。
- オンラインドキュメント: `https://mapir.gitbook.io/chloros/cli`
