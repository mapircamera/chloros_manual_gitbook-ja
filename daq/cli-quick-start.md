# CLI クイックスタート (pool-*)

同梱の `chloros-cli` は、**`daq pool-*`** コマンドファミリーを通じて DAQ センサーを駆動します。これは、Chloros バックエンドの永続的なセンサープールを介してセンサーを操作する、軽量な HTTP クライアントです。 トランスポートはバックエンドが管理しているため、GUI、CLI、SDKスクリプトはすべて、ポートの競合を起こすことなく、1つのアクティブなハンドルを共有します。 顧客が必要とするすべての機能（接続、ストリーム送信、校正済み `.daq` ファイルの記録、キャッププロファイルの切り替えなど）は、`pool-*` を通じて利用可能です。

`pool-*` は、リリース版でサポートされている DAQ サーフェスです。`chloros-cli daq --help` は `pool-*` サブコマンドの一覧を表示し、`daq discover` / `daq list` は、`pool-discover` と同じスキャンを実行します。 ハードウェア直接操作の DAQ サブコマンドが利用できない場合、欠落しているパッケージ名を明示したエラーメッセージが表示され、`pool-*` への戻りが指示されます。エラーが黙って発生することはありません。 （`pip install chloros-sdk` も、ハードウェアへの直接アクセスコマンドは提供していません。）

***

## 前提条件

* **Chlorosバックエンドが実行されている必要があります** — `pool-*`コマンドはHTTPクライアントであり、ハードウェアドライバではありません。Windowsでは、Chlorosデスクトップアプリを起動してください（これによりバックエンドが起動します）。 ヘッドレス環境のLinux/Jetsonでは、自身のユーザーとしてサービスを有効にしてください：`sudo systemctl enable --now chloros-backend@$USER.service`（[Linuxのインストール](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)を参照）。
* **Chloros+（有料プラン）へのログイン**：まず `chloros-cli login` を実行してください。 この制限はサーバー側で適用されます。ログインしていない場合、コマンドは `401 AUTH_REQUIRED` で失敗します。無料（Iron）プランでは、`403 PLAN_UPGRADE_REQUIRED` で失敗します。
* コマンドはデフォルトで `http://127.0.0.1:5000` を対象とします。バックエンドが別の場所で実行されている場合、`daq pool-*` ファミリーは `CHLOROS_BACKEND_URL` 環境変数を尊重します。

***

## 5分間のセッション

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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
| `daq pool-connect --port PORT` | 特定のシリアルポート上のDAQ-U（例：`COM3`、`/dev/ttyUSB0`）。 |
| `daq pool-connect --ble` | BLE経由のDAQ-M。MACアドレスは自動スキャンされる。 |
| `daq pool-connect --mac MAC` | 既知の BLE MAC アドレスを持つ DAQ-M（`--ble` を意味する）。 |
| `daq pool-connect --eth-host HOST` | 既知のホスト名またはIPアドレスでのDAQ-E — **信頼性の高い経路**。 |
| `daq pool-connect --eth` | 自動検出（mDNS、ARPフォールバック付き）によるDAQ-E。以下の注意事項を参照。 |

チューニングフラグ（すべてオプション）：

| フラグ | 意味 |
| --- | --- |
| `--integration-time MS` / `-t MS` | 手動の積分時間（ミリ秒単位）。 |
| `--frame-avg N` / `-f N` | 報告されるスペクトルごとの平均フレーム数。 |
| `--no-ae` | 自動露出（AE）を無効化（デフォルトでは有効）。 |
| `--no-stream` | ストリームを開始せずに接続する（後で `pool-stream --start` を使用して再開）。 |
| `--cap-id CAP` | キャップ補正プロファイル。バックエンドのデフォルトは `sunshine_cosine` です。 [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap)を参照。 |

すでにプールに登録されている DAQ-E を、別のアドレス（ホスト名 `daq-e-<id>.local` または IP アドレス）で接続すると、そのユニットを再度開くのではなく、既存のプールエントリが返されます。

{% hint style="warning" %}
**`--eth` 自動検出に関する注意点。** マルチホームホスト（アクティブなネットワークインターフェースが複数ある場合）では、起動後の*最初の*`pool-connect --eth`は、センサーが正常であっても空の結果となる可能性があります。これは、ARPキャッシュが冷えている間に、検出スキャンがセンサーのインターフェースを見逃してしまうためです。 `--eth`で何も検出されない場合は、再試行するか、`--eth-host <ip-or-hostname>`を使用して検出を完全にスキップしてください。これがマルチホームマシンにおける確実な方法です。 DAQ-Eのホスト名は`daq-e-<id>.local`（例：`daq-e-def330.local`）です。IPアドレス単体でも動作します。
{% endhint %}

## `pool-discover` — 接続せずにスキャンする

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

接続可能なセンサーをスキャンし、実際に接続することなく、接続先のアドレスを表示します。`--only` は、`usb,ble,eth` のコンマ区切りのサブセットを受け取ります。 `--timeout SEC`は、トランスポートごとのスキャン時間を設定します（デフォルトは5秒、1～20秒の範囲に制限されます）。 プール内で既に開かれているセンサーは一覧に表示されません（接続済みのDAQ-Mはアドバタイズを停止します）。それらについては、`pool-list`を使用してください。 出荷時のビルドでは、`chloros-cli daq discover` および `daq list` がこの同じスキャンを実行します。

## `pool-list` — 接続されているものを確認する

バックエンドプール内のすべてのセンサーを表示します。これには、他のすべてのコマンドで必要となる `sensor_id` も含まれます：

| モデル | `sensor_id` 形式 | 例 |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5オクテットのハイフン区切り | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

PTPがオフに設定された（設定は本機に保存される）DAQ-Eは、`pool-list`で`!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.`としてフラグが立てられ、PTPを再びオンにするコマンドが指定されます。詳細は [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on)を参照。

## `pool-latest` — スペクトルフレームの読み取り

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

最新のフレーム、または最新の `--recent N` フレームを返します。`--json` は、スクリプト処理用に機械可読形式の出力を生成します。 フレームは、135 ポイント、340～1010 nm のグリッド上で放射測定的に校正された分光放射照度（W/m²/nm）であり、センサーのキャッププロファイルがすでに適用されています。 定量的な放射照度の数値を得るには、少なくとも 15 秒間のフレームを平均化する必要があります。これは機器の特性であり、不具合ではありません。

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | 出力ディレクトリ。 このコンピュータ上のバックエンドの場合、相対パスはコマンドを実行したフォルダを基準に解決されます。リモートバックエンドの場合、パスは**バックエンドを実行しているマシン**上で解決されます。 |
| `--device-name NAME` | — | 記録とともに保存されるラベル。 |
| `--stop` | — | 進行中の記録を停止します。 |
| `--confirm-after SEC` | `20` | 開始後、最初の行が書き込まれるまでこの時間まで待機し、何も到着しない場合は大きな警告（`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`）を出します。 `0` を指定すると、このチェックは無効になります。 |
| `--heartbeat SEC` | `30` | `--duration` を指定すると、SEC 秒ごとに行数を表示し、増加が止まった場合に警告を出力する。 `0` を有効にすると、このチェックが無効になります。 |

{% hint style="info" %}
記録はバックエンドで行われるため、`.daq`ファイルは**バックエンドマシンの**ファイルシステムに保存されます。デフォルトでは、`~/Documents/DAQ Live View/`そこに保存され、CLIを実行した場所とは限らないことに注意してください。 ファイル名には、センサーIDとタイムスタンプが含まれます。
{% endhint %}

## `pool-set-cap` — 装着されたキャップの指定

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

キャップIDは、すべてのスペクトルに適用される工場測定済みの補正プロファイルを選択するものであり、**センサーに物理的に装着されているキャップと一致している必要があります**。センサーもソフトウェアも単独ではキャップを検出できないため、この選択情報はすべての`.daq`ファイルに記録されます。 すべての環境におけるデフォルトは `sunshine_cosine` です（すべての DAQ には、Sunshine コサイン補正キャップが取り付けられた状態で出荷されており、設計上約 12 倍の減衰率があります。キャップの変更を宣言しないと、スペクトルはおよそその倍率分だけ誤って補正されてしまいます）。

| `--cap-id` | 対応機種 |
| --- | --- |
| `sunshine_cosine` (デフォルト) | DAQ-U、DAQ-M、DAQ-E |
| `fov_15`、`fov_45`、`fov_90` | DAQ-U、DAQ-E |
| `fov_30`、`fov_60` | DAQ-U のみ |
| `none` | DAQ-E のみ — 注を参照 |

センサーのセット範囲外のキャップIDは、接続時に明確なエラーとして拒否されます。 `none` (DAQ-E) は、キャップが物理的に取り外されていることを意味します。DAQ-Eの凹型ガラスディフューザーには依然として工場出荷時の幾何学的プロファイルが適用されるため、これは無操作（no-op）ではありません。また、キャップなしのDAQ-Eはベンチ構成であり、サポート対象の現場構成ではありません。 （キャップなしの DAQ-U は完全な「ベア」状態であり、補正プロファイルは一切必要ありません。DAQ-M は Sunshine キャップと組み合わせて使用されます。）

## `pool-disconnect` — センサーの解放

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect`は、誰が開放したかに関わらずセンサーを解放するため、クラッシュしたスクリプトがプールに残したセンサーも解放します（そのセンサーでの記録が先に完了している場合）。 プールに存在しないセンサーは解放する対象がないため、このコマンドは `<sensor-id>: already released (not in the pool)` を出力して終了します `0`。バックエンドが拒否した、または確認しなかった解放処理は、`1` で終了します。

## `pool-enable-ptp` — DAQ-EのPTPを再び有効にする

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

DAQ-EはPTPのオン/オフ設定を自身のメモリに保持しているため、PTPがオフにされたユニットは、再びオンにされるまでオフのままであり、そのタイムスタンプはPTPに同期されません。 `pool-list`は、そのようなユニットをフラグ付けし、そのユニットに対してこのコマンドを出力します。Chlorosは、オフ状態が意図的なものである可能性があるため、ユニットを自動的に再有効化することはありません。1.3.0での新機能。

## DAQ-E デバイス設定

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info`は、センサーのファームウェア（およびこのChlorosリリースに新しいファームウェアが含まれているかどうか）、そのアドレスと取得方法、現在実行中の設定と異なる場合に保存されているネットワーク設定、電源投入時のストリーミング、およびPTPを表示します。 `pool-set-network`は、変更を適用するためにセンサーを再起動し、以前の設定で再接続します。`--no-reboot`は設定を保存するだけです。 静的アドレスはこのコンピュータのネットワーク上に存在している必要があり、そうでない場合、Chlorosはそれを拒否します。`--force`は、アドレスの再割り当てまたはそのアドレスへのルーティングが行われるコンピュータに限り、この制限を無効にします。 センサーが記録中は再起動は拒否されます。`pool-update-firmware`は<version\>、コマンドラインにおける</version\>GUIの**「\**に更新<version\>」に相当し、GUIのないLinux上でDAQ-Eを更新する方法です。1.3.0で追加されました。***

## コマンド概要

| コマンド | 目的 |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | バックエンドプール内のセンサーを開く。 |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 接続せずにセンサーをスキャンする（DAQ-MのBLE MACを検出）。 |
| `daq pool-list` | プール内のすべてのセンサーを、それぞれの `sensor_id` とともに表示します。 |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 校正済みのスペクトルフレームのうち、最新の N 個を表示します。 |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | ストリーミングの再開／一時停止。 |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | `.daq` 記録の開始／停止（バックエンド側）。 |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 実行時にキャップ補正プロファイルを切り替える。 |
| `daq pool-disconnect --sensor-id ID [--all]` | 1つのセンサー、またはすべてのセンサーを解放する。 |
| `daq pool-enable-ptp --sensor-id ID` | PTPがオフになっているDAQ-EのPTPを再びオンにする。 |
| `daq pool-info --sensor-id ID [--json]` | DAQ-Eのファームウェア、アドレス、保存されたネットワーク設定、電源投入時のストリーミング、およびPTP。 |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP または静的アドレス；センサーを再起動して再接続します。 |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | 電源投入時のストリーミング。 |
| `daq pool-reboot --sensor-id ID` | DAQ-Eを再起動し、再接続します。 |
| `daq pool-update-firmware --sensor-id ID` | 旧型のセンサーに、このリリースのDAQ-Eファームウェアを書き込みます。 |

***

## 初めてのDAQ-E接続時のトラブルシューティング

1. DAQ-EにはステータスLEDがありません。スイッチまたはインジェクタのポートにあるPoE/リンクインジケータで電源が供給されていることを確認し、電源投入後、数秒間待って起動し、ネットワークに接続されるのを待ちます。
2. バックエンドマシンは、センサーと**同じブロードキャストドメイン**にある必要があります。mDNSはルーターを通過しません。
3. ファイアウォールは、mDNS（UDP 5353）、DAQ-Eデータ（UDP 5002）、およびPTP（UDP 319/320）を通過させる必要があります。 Windowsインストーラーはこれに対応するルールを追加し、Linuxの`.deb`は、インストール時に有効だった`ufw` / `firewalld`でこれらを開放します。 サードパーティ製のファイアウォール、または後で有効化されたファイアウォールでは、これらのポートを通過させる必要があります。
4. それでも `--eth` から反応がない場合？ `--eth-host` に、デバイスのホスト名（`daq-e-<id>.local`）または IP アドレスを指定して使用してください。これにより、検出プロセスがスキップされます。

***
{% hint style="info" %}
**AIアシスタント向けのヒント。** このマニュアルの各ページは生のMarkdown形式で提供されています。ページの小文字のURLスラッグの末尾に「`.md`」を追加してください（このページの場合：`https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`）。 機械可読なインデックスは `https://mapir.gitbook.io/chloros/llms.txt` となります。 `chloros-cli daq`およびその他のすべてのコマンドファミリーに関する完全なフラグレベルのドキュメントについては、[CLIリファレンス](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`) を参照してください； Pythonパスは、[SDKリファレンス](../reference/sdk-reference.md)内の`chloros_sdk.connect_daq_sensor()`です。
{% endhint %}
</version\>
