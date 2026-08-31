# CLI クイックスタート (pool-*)

同梱の `chloros-cli` は、**`daq pool-*`** コマンドファミリーを介してDAQセンサーを駆動します。これは、HTTPクライアントが、Chlorosバックエンドの永続的なセンサープールを介してセンサーを操作するものです。 トランスポートはバックエンドが所有しているため、GUI、CLI、および SDK スクリプトはすべて、ポートの奪い合いをすることなく、1 つのライブハンドルを共有します。 顧客が必要とするすべての機能（接続、ストリーミング、校正済み `.daq` ファイルの記録、キャッププロファイルの切り替え）は、`pool-*` を通じて利用可能です。

また、`pool-*`は、リリース版ビルドにおいて**唯一の**DAQサーフェスでもあります。`chloros-cli daq --help`は`pool-*`のサブコマンドを一覧表示し、 また、出荷版ビルドでハードウェア直接操作のDAQサブコマンドを実行すると、欠落しているパッケージ名を明示したエラーが表示され、`pool-*`を参照するよう指示して終了します。何も静かに失敗することはありません。 （ダイレクトハードウェアコマンドは、MAPIRのソースチェックアウトからのみ実行可能です。`pip install chloros-sdk`でもこれらのコマンドは提供されていません。）

***

## 前提条件

* **Chloros バックエンドが実行されている必要がある** — `pool-*` コマンドは HTTP のクライアントであり、ハードウェアドライバではない。 Windows では、Chloros デスクトップアプリを起動してください（これによりバックエンドが起動します）。 ヘッドレス版の Linux/Jetson では、サービス（`sudo systemctl enable --now chloros-backend.service`）を有効にしてください。
* **Chloros+（有料プラン）へのログイン**：まず `chloros-cli login` を実行してください。 強制はサーバー側で行われます。ログインしていない場合、コマンドは `401 AUTH_REQUIRED` で失敗します。無料（Iron）プランでは、`403 PLAN_UPGRADE_REQUIRED` で失敗します。
* コマンドはデフォルトで `http://127.0.0.1:5000` を対象とします。バックエンドが別の場所で実行されている場合、`daq pool-*` ファミリーは `CHLOROS_BACKEND_URL` 環境変数を尊重します。

***

## 5分間のセッション

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — プール内のセンサーを開く

| バリエーション | 意味 |
| --- | --- |
| `daq pool-connect` | スマート検出：このマシン上の任意のDAQを検索します。 |
| `daq pool-connect --port PORT` | 特定のシリアルポート上のDAQ-U（例: `COM3`、`/dev/ttyUSB0`）。 |
| `daq pool-connect --ble` | BLE経由のDAQ-M。MACアドレスは自動スキャンされる。 |
| `daq pool-connect --mac MAC` | 既知の BLE MAC アドレスを持つ DAQ-M（`--ble` を意味する）。 |
| `daq pool-connect --eth-host HOST` | 既知のホスト名またはIPアドレスでのDAQ-E — **信頼性の高い経路**。 |
| `daq pool-connect --eth` | 自動検出（mDNS、ARPフォールバック付き）によるDAQ-E。以下の注意事項を参照。 |

調整フラグ（すべてオプション）：

| フラグ | 意味 |
| --- | --- |
| `--integration-time MS` / `-t MS` | 手動の積分時間（ミリ秒単位）。 |
| `--frame-avg N` / `-f N` | 報告されるスペクトルごとの平均フレーム数。 |
| `--no-ae` | 自動露出（AE）を無効化（デフォルトでは有効）。 |
| `--no-stream` | ストリームを開始せずに接続する（後で `pool-stream --start` を使用して再開）。 |
| `--cap-id CAP` | キャップ補正プロファイル。バックエンドのデフォルトは `sunshine_cosine` です。 [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap)を参照。 |

{% hint style="warning" %}
**`--eth` 自動検出に関する注意点。** マルチホームホスト（アクティブなネットワークインターフェースが複数ある場合）では、起動後の*最初の* `pool-connect --eth` は、センサーが正常であっても空の結果となる可能性があります。これは、ARP キャッシュが冷えている間に、検出スキャンがセンサーのインターフェースを見逃してしまうためです。 `--eth`で何も見つからない場合は、再試行するか、`--eth-host <ip-or-hostname>`を使用して検出を完全にスキップしてください。これがマルチホームマシンでの確実な方法です。 DAQ-Eのホスト名は`daq-e-<id>.local`（例：`daq-e-def330.local`）です。IPアドレス単体でも動作します。
{% endhint %}

## `pool-list` — 接続されている機器の確認

バックエンドプール内のすべてのセンサーを表示します。これには、他のすべてのコマンドで必要となる `sensor_id` も含まれます：

| モデル | `sensor_id` 形式 | 例 |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5オクテットのハイフン区切り | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — スペクトルフレームの読み取り

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

最新のフレーム、または最新の `--recent N` フレームを返します。`--json` は、スクリプト処理用の機械可読出力を出力します。 フレームは、135 ポイント、340～1010 nm のグリッド上で、センサーのキャッププロファイルがすでに適用された、放射測定的に校正されたスペクトル放射照度（W/m²/nm）です。 定量的な放射照度値を得るには、少なくとも15秒分のフレームを平均化してください。これは機器の特性であり、不具合ではありません。

## `pool-stream` — ストリーミングの一時停止または再開

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — `.daq` ファイルの記録

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| フラグ | デフォルト | 意味 |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | 録画時間（秒単位）； `0` は、`--stop` を発行するまで実行し続けることを意味します。 |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | 出力ディレクトリ。**バックエンドを実行しているマシン**上で解決されます。 |
| `--device-name NAME` | — | 録画に保存されるラベル。 |
| `--stop` | — | 進行中の録画を停止します。 |

{% hint style="info" %}
録画はバックエンドで行われるため、 そのため、`.daq`ファイルは**バックエンドマシンの**ファイルシステムに保存されます。デフォルトでは`~/Documents/DAQ Live View/`に保存され、必ずしもCLIを実行した場所とは限りません。 ファイル名には、センサーIDとタイムスタンプが含まれます。
{% endhint %}

## `pool-set-cap` — 装着されたキャップを指定する

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

キャップ ID は、すべてのスペクトルに適用される工場測定の補正プロファイルを選択するものであり、**センサーに物理的に取り付けられているキャップと一致している必要があります**。センサーもソフトウェアも単独ではキャップを検出できないため、この選択情報はすべての `.daq` ファイルに記録されます。 すべての環境におけるデフォルトは `sunshine_cosine` です（すべての DAQ には、Sunshine コサイン補正キャップが取り付けられた状態で出荷されており、設計上約 12 倍の減衰があります。キャップの変更を宣言しないと、スペクトルがおよそその倍率分だけ誤って補正されてしまいます）。

| `--cap-id` | 対応機種 |
| --- | --- |
| `sunshine_cosine` (デフォルト) | DAQ-U、DAQ-M、DAQ-E |
| `fov_15`、`fov_45`、`fov_90` | DAQ-U、DAQ-E |
| `fov_30`、`fov_60` | DAQ-U のみ |
| `none` | DAQ-E のみ — 注を参照 |

センサーのセット範囲外のキャップIDは、接続時に明確なエラーとして拒否されます。 `none` (DAQ-E) は、キャップが物理的に取り外されていることを意味します。DAQ-Eの凹型ガラスディフューザーには依然として工場出荷時の幾何学的プロファイルが適用されるため、これは無操作（no-op）ではありません。また、キャップなしのDAQ-Eはベンチ構成であり、サポート対象の現場構成ではありません。 （キャップなしのDAQ-Uは完全な「ベア」状態であり、補正プロファイルは一切必要ありません。DAQ-MはSunshineキャップと組み合わせて使用されます。）

## `pool-disconnect` — センサーの解放

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## コマンド概要

| コマンド | 目的 |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | バックエンドプール内のセンサーを開く。 |
| `daq pool-list` | プール内のすべてのセンサーを、その `sensor_id` とともに表示します。 |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 校正済みのスペクトルフレームのうち、最新の N 個を表示します。 |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | ストリーミングの再開／一時停止。 |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | `.daq` 記録の開始／停止（バックエンド側）。 |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 実行時にキャップ補正プロファイルを切り替える。 |
| `daq pool-disconnect --sensor-id ID [--all]` | 1つのセンサー、またはすべてのセンサーを解放する。 |

***

## 初めての DAQ-E 接続時のトラブルシューティング

1. DAQ-E にはステータス LED がありません。スイッチまたはインジェクタのポートにある PoE/リンクインジケータで電源が供給されていることを確認し、電源投入後、起動してネットワークに参加するまで数秒間お待ちください。
2. バックエンドマシンは、センサーと**同じブロードキャストドメイン**にある必要があります。mDNSはルーターを通過しません。
3. Windowsでは、初回起動時にDefenderファイアウォールのプロンプトを許可してください（mDNS：UDP 5353、DAQ-Eデータ：UDP 5002、PTP：UDP 319/320）。
4. `--eth`からまだ反応がない場合は、`--eth-host`を使用して、ユニットのホスト名（`daq-e-<id>.local`）またはIPアドレスを指定してください。特にマルチホームホストでは、これが確実な方法です。

***{% hint style="info" %}**AIアシスタントへのヒント。** このマニュアルの各ページは生のMarkdown形式で提供されています。ページの小文字のスラッグ（このページの場合：URL）の末尾に `.md` を追加してください（このページの機械可読インデックスは `https://mapir.gitbook.io/chloros/llms.txt` です）； 機械可読なインデックスは `https://mapir.gitbook.io/chloros/llms.txt` となります。 `chloros-cli daq` およびその他のすべてのコマンドファミリーに関する完全なフラグレベルのドキュメントについては、[CLI リファレンス](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`)を参照してください。 Python のパスは、[SDK リファレンス](../reference/sdk-reference.md) 内の `chloros_sdk.connect_daq_sensor()` です。
{% endhint %}
