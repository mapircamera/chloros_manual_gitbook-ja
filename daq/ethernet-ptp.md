# DAQ-E のネットワーク設定と時刻同期

> センサーの物理的なネットワーク設定（配線、PoE、IP アドレスの割り当て、およびデバイス自体のネットワーク設定）については、**[DAQ ユーザーマニュアル](https://mapir.gitbook.io/daq/daq-e/network-setup)**に記載されています。 このページでは、Chloros 側の内容、すなわち接続、時刻同期、および検出結果が得られなかった場合の対処法について説明します。

DAQ-EはDAQファミリーのイーサネット対応モデルです。PoEで給電され、mDNS（サービス名 `_daq-e._tcp`）を介して検出され、センサーID — `daq-e-<6 hex>.local`、例：`daq-e-def330.local`。このページでは、ネットワーク上でのデータ転送方法およびPTP時刻同期への参加方法について説明します。

## 転送モード

| モード | エンドポイント | 受信側 | 備考 |
| --- | --- | --- | --- |
| **マルチキャスト** (デフォルト) | UDP `239.10.10.10:5002` | 同一LAN上の任意の数が同じストリームを受信 | 各データグラムはCRC-16/CCITTによる検証が行われる |
| **Raw** | TCPポート `5000` | クライアントは正確に1台（排他的） | DAQ-Uとバイト単位で互換性あり |

Chlorosはデフォルトでマルチキャストを使用しており、これによりGUI、CLI、およびSDKがすべて同時に1つのセンサーを監視できるようになります。

## ネットワーク要件

* **同一のブロードキャストドメイン。** Chloros を実行しているマシンは、センサーと同じ L2 ネットワークセグメント上に存在する必要があります。mDNS による検出はルーターを通過しません。
* **Windows によるファイアウォールの確認メッセージ：許可してください。** Chloros が初めてマルチキャストソケットをバインドする際、Windows Defender から一度確認のメッセージが表示されます。 これを許可することで、DAQ-Eデータ（UDP 5002）、mDNS（UDP 5353）、およびPTP（UDP 319/320）がカバーされます。Linuxでは、この処理は静かに実行されます。
* **PoE 給電、ステータス LED なし。** DAQ-E 自体には LED がありません。スイッチまたはインジェクタポートのリンク/PoE インジケータで電源を確認し、電源投入後、起動してネットワークに接続されるまで数秒間お待ちください。

## 接続

**GUI：** [光センサー] タブ → [センサーの接続] → デバイス種別「DAQ-E (イーサネット)」。 検出処理は、接続ダイアログが画面上に表示されている間のみ実行されます（mDNS ブラウズに加え、Windows に対する ARP スウィープ）。これは 15 秒ごとに繰り返され、「更新」ボタンをクリックすると直ちに再スキャンが行われます。検出されたセンサーはドロップダウンリストに表示され、最初に検出されたセンサーが自動的に選択されます。

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI**（バックエンドが実行中）：

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### マルチNICホストおよび起動後の初回接続

アクティブなネットワークインターフェースが複数あるホストでは、起動後の**最初の** `pool-connect --eth` において、センサーが正常であっても結果が空になることがあります。これは、ARPキャッシュがまだ冷えている間に、センサーが存在するインターフェースが検出プロセスで見落とされるためです。 確実な回避策は、ディスカバリをスキップしてアドレスを明示的に指定することです：

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` は mDNS ホスト名または IP アドレスを受け付け、常に正しいセンサーをターゲットにします。これは、スクリプトやヘッドレスインストールにおいて推奨される形式です。 GUI では、接続ダイアログの [Refresh] ボタンを使用し、再スキャンサイクルを実行してください。

## デバイス設定とファームウェア

センサー本体には、ネットワーク設定（静的 IP 対 DHCP ＋リンクローカルアドレス）、デバイス名、起動時の自動ストリーム、OTA パスワードなどが保存されています。 これらのデバイス側の設定は、出荷時の CLI ではコマンドとして公開されていません。これらは、Chloros GUI 上で表示される項目を通じて、または MAPIR サポートを利用して管理されます。

**ファームウェアの更新機能はGUIに組み込まれています。**接続された DAQ-E が、Chloros ビルドに同梱されているイメージよりも古いファームウェアを実行している場合、そのセンサー行にはオレンジ色の**更新あり** ピルが表示され、歯車アイコンの設定モーダルに「～に更新<version>

」ボタン</version>が表示されます<version>

。 アップデートはネットワーク経由で約30秒で完了します。センサーは自動的に再起動して再接続され、転送が中断された場合でも現在のファームウェアはそのまま維持されます。

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP時刻同期

DAQ-E ファームウェア v1.2.0 以降では、IEEE 1588 PTPv2 に通常の（スレーブ専用の）クロックとして参加します。 **ChlorosホストのバックエンドはPTPグランドマスターです**。LAN上のすべてのDAQ-EおよびLATTICEカメラは、ドメイン0においてこれにスレーブとして同期し、すべてのデバイスのタイムスタンプを約1 msの許容誤差内に保ちます。 この共有クロックにより、DAQの測定値とカメラの露光時間をタイムスタンプで照合できるようになります（[記録と.daq形式](recording.md)を参照）。

CLIからの同期情報を確認してください：

| コマンド | 表示内容 |
| --- | --- |
| `chloros-cli time-sync status` | ホストのグランドマスター状態、BMCA優先度、クロックID |
| `chloros-cli time-sync peers` | 検出されたすべてのスレーブ（DAQ-Eセンサー + LATTICEカメラ） |
| `chloros-cli time-sync cameras` | カメラごとのPTPヘルス状態（`PtpStatus`、`PtpOffsetFromMaster`、`PtpMeanPathDelay`） |
| `chloros-cli time-sync restart` | グランドマスタープロセスの再起動 |

GUI では、DAQ-E 設定モーダルに、センサーの現在の PTP 状態を示すリアルタイムの **PTP Sync** 行が表示されます。

厳密な同期を必要とするコンシューマに関する詳細：

* ストリーミングされるすべてのデータグラムにはフラグフィールドが含まれており、**タイムスタンプがPTP同期されているフレームではビット2がセットされます**。カメラとDAQの厳密な同期を必要とするパイプラインは、このビットに基づいてゲート処理を行う必要があります。
* 同期キャプチャを行う前に、センサーが `chloros-cli time-sync peers` に表示されていることを確認してください。 (MAPIRの内部ダイレクトハードウェア・ツール群では、センサーがSLAVE状態になるまで最大15秒待機する`--wait-ptp`フラグを使用して、PTPロック時に記録をゲート制御することも可能です。 このツール機能は、出荷版の CLI には含まれていません。）
* PTP がアクティブにスレーブ状態にある間、センサーは手動によるクロックプッシュを拒否します（「PTP がクロックを供給中」）。これは仕様によるものです。PTP を信頼してください。

## Linux に関する注意事項

* **PTP には、インストール時に `libcap2-bin` が必要です。** `.deb`のpostinstスクリプトは、`/usr/lib/chloros/chloros-backend`上で`cap_net_bind_service=+ep`に権限を付与し、root権限なしでPTPポート319/320をバインドできるようにします。 `libcap2-bin` が存在しない場合、その手順はスキップされ、PTP の起動に失敗します。修正方法：

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **ヘッドレス Jetson / Raspberry Pi:** 初回インストール時、systemd ユニット `chloros-backend.service` は生成されますが、有効化されていません。GUI を使用せずに PTP（および DAQ）を常時稼働させるには：

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  これがない場合、PTPはChloros GUIが開いている間のみ動作します。

## トラブルシューティング：「DAQ-Eデバイスが検出されません」

| 確認項目 | 詳細 |
| --- | --- |
| 電源 | センサーのLEDが点灯しない — スイッチ／インジェクターポートのPoEおよびリンクインジケーターを確認してください。電源投入後、数秒間待ってください |
| ブロードキャストドメイン | ホストとセンサーが同じL2セグメント上にある。mDNSがルーティングされない |
| Windows ファイアウォール | 初回実行時に Defender のプロンプトを受け入れてください（UDP 5002、5353、319/320） |
| マルチNICホスト | 起動直後の初回検出ではセンサーが検出されない場合があります — `--eth-host <ip-or-hostname>`で接続してください |
| GUIによる再スキャン | 検出は接続ダイアログが開いている間のみ実行されます。「更新」を使用してください |</version>
