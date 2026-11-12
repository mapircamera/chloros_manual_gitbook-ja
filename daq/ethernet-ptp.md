# DAQ-E のネットワーク接続と時刻同期

> センサーの物理的なネットワーク設定（配線、PoE、IP アドレスの割り当て、およびデバイス自体のネットワーク設定）については、**[DAQ ユーザーマニュアル](https://mapir.gitbook.io/daq/daq-e/network-setup)**に記載されています。 このページでは、Chloros側の内容、すなわち接続、時刻同期、および検出に失敗した場合の対処法について説明します。

DAQ-EはDAQファミリーのイーサネット対応モデルです。PoEで給電され、mDNS（サービス `_daq-e._tcp`）を介して検出され、センサーIDから派生したホスト名 — `daq-e-<6 hex>.local`、例：`daq-e-def330.local`。このページでは、ネットワーク上でのデータ転送方法およびPTP時刻同期への参加方法について説明します。

## 転送モード

| モード | エンドポイント | 受信側 | 備考 |
| --- | --- | --- | --- |
| **マルチキャスト** (デフォルト) | UDP `239.10.10.10:5002` | 同一LAN上の任意の数が同じストリームを受信 | 各データグラムはCRC-16/CCITTによる検証が行われる |
| **Raw** | TCPポート `5000` | クライアントは正確に1台（排他的） | DAQ-Uとバイト単位で互換性があり、そのまま利用可能 |

Chlorosはデフォルトでマルチキャストを使用しており、これによりGUI、CLI、SDKのすべてが1つのセンサーを同時に監視できるようになります。

## ネットワーク要件

* **同一のブロードキャストドメイン。** Chlorosを実行するマシンは、センサーと同じ L2 ネットワークセグメント上に存在する必要があります。mDNS による検出はルーターを通過しません。
* **ファイアウォール。** Windowsでは、インストーラーがすべてのネットワークプロファイルに対して、バックエンド用の**Chloros Backend (UDP-In)**ファイアウォールルールを追加します。このルールは、DAQ-Eデータ（UDP 5002）、 mDNS（UDP 5353）、PTP（UDP 319/320）をカバーします。また、ファームウェア更新中にDAQ-Eがコンピュータへ接続する際のために、ローカルサブネットに限定された**「Chloros Backend (TCP-In)」**ルールも追加されます。 Linuxでは、インストール時にそのファイアウォールが有効になっている場合、`.deb`が`ufw`または`firewalld`のUDPポートを開きます。 サードパーティ製のファイアウォール、または後から有効化されたファイアウォールについても、同じポートを通す必要があります。
* **PoE 給電、ステータス LED なし。** DAQ-E 自体には LED がありません。スイッチまたはインジェクタポートのリンク/PoE インジケータで電源を確認し、電源投入後、起動してネットワークに参加するまで数秒間お待ちください。

## 接続

**GUI:** [光センサー] タブ → [センサーの接続] → デバイス種別「DAQ-E (イーサネット)」。検出処理は、接続ダイアログが画面に表示されている間のみ実行され（Windowsでは mDNS ブラウズと ARP スウィープ）、15 秒ごとに繰り返されます。「更新」ボタンを押すと、直ちに再スキャンが行われます。 検出されたセンサーはドロップダウンリストに表示され、最初に検出されたセンサーが自動的に選択されます。

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI**（バックエンド実行中）：

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

プールにすでに存在するDAQ-Eは、ホスト名とIPアドレスの両方で識別されます。別のアドレスで再度接続しても、ユニットを2度目として開くのではなく、既存のプールエントリが返されます。

### マルチNICホストと起動後の最初の接続

アクティブなネットワークインターフェースが複数あるホストでは、起動後の**最初の** `pool-connect --eth` が、センサーが正常であっても空になることがあります。これは、ARPキャッシュがまだ初期状態の間に、センサーが存在するインターフェースが検出プロセスで見落とされるためです。 確実な回避策は、ディスカバリをスキップしてアドレスを明示的に指定することです：

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host`はmDNSホスト名またはIPアドレスを受け付け、常に正しいセンサーをターゲットにします。これは、スクリプトやヘッドレスインストールにおいて推奨される形式です。GUIでは、接続ダイアログの「更新」ボタンを使用し、再スキャンサイクルを待ってください。

## デバイス設定とファームウェア

センサー本体には、ネットワーク設定（DHCP（工場出荷時の設定。DHCPサーバーからの応答がない場合は169.254.x.xのリンクローカルアドレスにフォールバック）または静的アドレス）が保存されています。また、電源投入時にストリーミングを開始するかどうかという設定も保持されています。 これらの設定は、GUI（センサーの歯車アイコンの設定、**ネットワーク**：**変更…**、**電源投入時にストリーミング**、**センサーの再起動**；[「DAQ」タブ](gui.md)を参照）、またはCLIおよびSDKから`pool-set-network`、 `pool-set-auto-stream` および `pool-reboot` （`DAQSensorSession.set_network()`、`set_auto_stream()`、`reboot()`；[CLIリファレンス](../reference/cli-reference.md#daq-e-device-settings)を参照）から設定できます。 ネットワークの変更を反映するためにセンサーが再起動し、Chlorosが再接続を行います。静的IPアドレスは、Chlorosを実行しているコンピュータのネットワーク上に存在する必要があります。Chlorosは、再起動後に到達できないアドレスを拒否します。センサーのIDおよびその`daq-e-<id>.local`ホスト名は決して変更されません。 1.3.0での新機能。

**ファームウェアの更新機能がGUIに組み込まれています。** 接続されたDAQ-Eが、お使いのChlorosビルドに同梱されているイメージよりも古いファームウェアを実行している場合、そのセンサー行に黄色の**「更新あり」**アイコンが表示され、歯車アイコンの設定モーダルに「...に更新<version>」ボタン</version>

が表示されます。<version> アップデートはネットワーク経由で約30秒で完了します。センサーは自動的に再起動して再接続され、転送が中断された場合でも現在のファームウェアは保持されます。 CLIやSDKから、またGUIのないLinux環境では、`chloros-cli daq pool-update-firmware --sensor-id <id>`（`DAQSensorSession.update_firmware()`）を実行してください。

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP 時刻同期

DAQ-E ファームウェア v1.2.0 以降では、IEEE 1588 PTPv2 に通常の（スレーブ専用）クロックとして参加します。 **ChlorosホストのバックエンドがPTPグランドマスターとなります**。LAN上のすべてのDAQ-EおよびLATTICEカメラは、ドメイン0においてこれにスレーブとして同期し、すべてのデバイスのタイムスタンプを約1 msの許容誤差内に保ちます。 この共有クロックにより、DAQの測定値とカメラの露光時間のタイムスタンプを一致させることができます（[記録と.daq形式](recording.md)を参照）。

CLIから同期状態を確認します：

| コマンド | 表示内容 |
| --- | --- |
| `chloros-cli time-sync status` | ホストのグランドマスター状態、BMCA優先度、クロックID、および `health`: `OK`、または `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` および、そのパケットがネットワークに到達しない理由を説明する `health_detail` |
| `chloros-cli time-sync peers` | 検出されたすべてのスレーブ（DAQ-Eセンサー＋LATTICEカメラ） |
| `chloros-cli time-sync cameras` | カメラごとのPTPヘルス状態（`PtpStatus`、`PtpOffsetFromMaster`、`PtpMeanPathDelay`） |
| `chloros-cli time-sync restart` | グランドマスタープロセスの再起動 |

GUI の DAQ-E 設定モーダルには、センサーの現在の PTP 状態を示す **PTP 同期** 行がリアルタイムで表示されます。**PTPがオフになっているDAQ-Eは、オフのままとなります。** ユニットはPTPのオン/オフ設定を自身のメモリに保持しているため、PTPがオフにされていたユニットは、再接続や電源再投入後もオフのままとなり、そのタイムスタンプはPTPに同期されません。 `chloros-cli daq pool-list`は、そのようなユニットをフラグ付けし（`!! PTP is switched OFF on this unit …`）、問題を修正するコマンドを出力します：

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chlorosは、オフ状態が意図的なものである可能性があるため、ユニットを自動的にオンに戻すことはありません。

厳密な同期を必要とするユーザー向けの詳細：

* ストリーミングされるすべてのデータグラムにはフラグフィールドが含まれており、**タイムスタンプがPTP同期されているフレームではビット2がセットされます**。カメラやDAQの厳密な同期を必要とするパイプラインでは、このビットをゲート条件として使用する必要があります。
* 同期キャプチャを行う前に、センサーが `chloros-cli time-sync peers` に表示されていること、および `chloros-cli daq pool-list` でその PTP がオフになっているとフラグが立てられていないことを確認してください。 (MAPIRの内部用ダイレクトハードウェアツールでは、センサーがSLAVE状態になるまで最大15秒待機する`--wait-ptp`フラグを使用して、PTPロック時に記録をゲート制御することも可能です。ただし、このツールは出荷版のCLIには含まれていません。)
* PTPがアクティブにスレーブ動作している間、センサーは手動によるクロックプッシュを拒否します（「PTPがクロックを供給中」）。これは仕様によるものです。PTPを信頼してください。

## Linuxに関する注意事項

* **root権限なしでPTPポートを使用する場合。** PTPはUDP 319/320にバインドします。 `.deb`は`/etc/sysctl.d/60-chloros-ptp.conf`（`net.ipv4.ip_unprivileged_port_start = 319`）を書き込むため、バックエンドはそれらをユーザーの権限で実行してバインドします。 バージョン 1.3.0 以降、このパッケージはバックエンドにファイル権限 (`setcap`) を設定せず、`libcap2-bin` も不要です。自分で権限を追加しないでください。追加すると、展開されたバックエンドが起動しなくなります。 PTP がポート 319 にバインドできない場合は、[Linux インストール](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319) を参照してください。

* **ヘッドレス Jetson / Linux ホスト:** このパッケージはバックエンドサービスをインストールしますが、いずれも有効化しません。コマンドを実行するユーザーがいない状態で PTP（および DAQ）を常時稼働させるには、ご自身のユーザーとしてサービスを有効にしてください:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  これを行わない場合、PTPは、`chloros-cli`コマンドやSDKスクリプトなど、何かがバックエンドを起動している間のみ実行されます。 root `chloros-backend.service` およびそこから切り替える方法については、[Linux インストール](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) を参照してください。

## トラブルシューティング：「DAQ-E デバイスが見つかりません」

| 確認項目 | 詳細 |
| --- | --- |
| 電源 | センサーのLEDが点灯していない — スイッチ／インジェクターポートのPoEおよびリンクインジケーターを確認してください。電源投入後、数秒待ってください |
| ブロードキャストドメイン | ホストとセンサーが同じL2セグメント上にある。mDNSがルーティングされない |
| ファイアウォール | Windows：インストーラーの**Chloros Backend (UDP-In)**ルールにより、DAQ-Eストリーム（UDP 5002）、mDNS（5353）、PTP（319/320）の受信が許可されています。サードパーティ製ファイアウォールでも同様の設定が必要です。 Linux：`.deb`は、インストール時にファイアウォールが有効だった場合にのみ、`ufw` / `firewalld`でこれらのポートを開きます。後で有効にした場合は、手動で許可してください |
| マルチNICホスト | ディスカバリーはmDNSを検索し、ARPをフォールバックとして使用します。`--eth-host <ip-or-hostname>`はディスカバリーをスキップして直接接続します |
| GUIによる再スキャン | ディスカバリーは接続ダイアログが開いている間のみ実行されます。「更新」を使用してください |
