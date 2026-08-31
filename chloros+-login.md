# Chloros+ ログイン

## GUI ログイン

<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">のサイドバーメニューから、Chloros+ アカウントにログインし、追加機能を利用できるようになります。

**各マシンでログインする必要があるのは1回のみです。** GUI、CLI、および Python SDK は、同じキャッシュされたセッションを共有しています — デスクトップGUI経由でサインインすると、そのマシン上のCLIおよびSDKも有効化されます（`chloros-cli login`経由でも同様です）。

ログインすると、アカウントの詳細が表示されます：

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## プランの階層

| プラン | `plan_id` | タイプ |
| --- | --- | --- |
| Iron | `0` | 無料 |
| Copper | `1` | 有料 (Chloros以上) |
| Bronze | `2` | 有料 (Chloros+) |
| シルバー | `3` | 有料 (Chloros+) |
| ゴールド | `4` | 有料 (Chloros+) |

各有料プランの内容については、[プランと料金](https://cloud.mapir.camera/pricing)をご確認ください。

### CLI / SDKへのアクセスには有料プランが必要です

CLI および Python、SDK へのアクセスには、**有料の Chloros+ プラン（Copper 以上）**が必要です。 これは**サーバー側**で強制されます。すべての CLI/SDK リクエストには、有効なセッションと有料プランの両方が含まれている必要があります：

| HTTP ステータス | `error_code` | 意味 | 対処法 |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | このマシンでログインしていない | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | ログイン済みですが、プランのティアが低すぎます（無料の Iron ティア） | 有料の Chloros+ プランのいずれかにアップグレードしてください |

`chloros-cli status`は無料プランでも引き続き利用可能ですので、現在のプランやアクセスが拒否された理由をいつでも確認できます。

### プランごとの接続ハードウェア数の上限

各プランには、同時にライブ接続できるLATTICEカメラおよびDAQ光センサーの数に上限があります：

| プラン | LATTICEカメラ | DAQ光センサー |
| --- | --- | --- |
| Iron（無料／未ログイン） | 4 | 2 |
| Copper／Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI ログイン

Chloros+ の認証情報を使用してログインし、CLI 処理を有効にしてください。 Linux（GUIなし）では、これがライセンスを有効にする唯一の方法です。

**構文：**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK ユーザー**: Python および SDK では、キャッシュされた認証情報をクリアするためのプログラムによる `logout()` メソッドも提供されています。 詳細については、[SDK リファレンス](reference/sdk-reference.md)を参照してください。
{% endhint %}

**例：**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**特殊文字**: `$`、`!`、またはスペースなどの文字を含むパスワードは、一重引用符で囲んでください。
{% endhint %}

**出力:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### 認証情報の保存

キャッシュされた認証情報と設定は、**すべてのプラットフォーム**において、ユーザーのホームディレクトリ内の `.chloros` フォルダに保存されます：

| プラットフォーム | 認証情報キャッシュのパス |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### プランの有効期限とオフラインの猶予期間

GUI に表示されるプランの有効期限は、ライセンスが無効になる時期を示しています。月額定期購読の場合、有効期限は月末となります。年間定期購読の場合、有効期限は購読開始から 1 年後となります。

Chlorosはライセンスをオンラインで検証しますが、猶予期間内であればオフラインでの使用もサポートされています：

* サーバーによる検証が成功した場合は、その結果が**5分間**キャッシュされるため、通常の使用ではライセンスへの呼び出し回数はごくわずかです。
* 署名済みでマシンに紐付けられたライセンスキャッシュにより、より長いオフライン期間に対応しています：**月額プランの場合は30日間**、**年間プランの場合はサブスクリプションの有効期限まで（最大365日間）**です。
* 猶予期間が終了すると、そのマシンがライセンスサーバーに一度接続できるまで、プランは無料の「Iron」ティアに切り替わります。次の検証に成功すると、アクセスが再開されます。

### デバイス数制限

各 Chloros+ プランでは、登録可能なデバイス数が異なります。 Chloros+アカウントでログインする各デバイスは、登録済みデバイスの数にカウントされます。MAPIR Cloudアカウントページから、デバイスの名前変更や削除を行うことができます。

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ プラン</th><th align="center">COPPER</th><th align="center">ブロンズ</th><th align="center">シルバー</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">対応デバイス</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>アカウントの正確なデバイス登録可能数は、MAPIR Cloudのアカウントページに表示されています。デバイスからログアウトすると、そのスロットは確実に解放されます。また、すでに登録済みのデバイスは、アカウントのデバイス登録上限に達している場合でも、いつでも再ログインが可能です。
