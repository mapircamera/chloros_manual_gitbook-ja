# Chloros+ ログイン

## GUI ログイン

ユーザー <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> サイドバーメニューを使用すると、Chloros+ アカウントにログインし、追加機能を利用できるようになります。

**1 台のマシンにつき、ログインは 1 回だけで済みます。** GUI、CLI、Python SDK は同じキャッシュされたセッションを共有しています。デスクトップ GUI 経由でサインインすると、そのマシン上の CLI および SDK も有効になります（`chloros-cli login` 経由でも同様です）。

ログインすると、アカウントの詳細が表示されます：

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## プランの階層

| プラン | `plan_id` | タイプ |
| --- | --- | --- |
| Iron | `0` | 無料 |
| Copper | `1` | 有料（Chloros+） |
| Bronze | `2` | 有料（Chloros+） |
| Silver | `3` | 有料（Chloros+） |
| ゴールド | `4` | 有料（Chloros+） |

各有料プランに含まれる内容については、[プランと料金](https://cloud.mapir.camera/pricing) をご覧ください。

### CLI / SDKへのアクセスには有料プランが必要です

CLIおよびPython SDKへのアクセスには、****Chloros+の有料プラン（Copper以上）****のいずれかが必要です。これは**サーバー側**で強制されます。すべてのCLI/SDKリクエストには、有効なセッションと有料プランの両方が含まれている必要があります：

| HTTPステータス | `error_code` | 意味 | 対処法 |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | このマシンでログインしていません | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | ログイン済みですが、プランのティアが低すぎます（無料のIronティア） | 有料のChloros+プランのいずれかにアップグレードしてください |

`chloros-cli status`は無料プランでもアクセス可能なので、現在のプランやアクセスが拒否された理由をいつでも確認できます。

### プランごとの接続ハードウェア数の上限

各プランには、同時にライブ接続できるLATTICEカメラおよびDAQ光センサーの数に上限があります：

| プラン | LATTICEカメラ | DAQ光センサー |
| --- | --- | --- |
| Iron（無料／未ログイン） | 4 | 2 |
| Copper／Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLIログイン

Chloros+ の認証情報を使用してログインし、CLI処理を有効にしてください。Linux（GUI なし）では、これがライセンスを有効にする唯一の方法です。

**構文：**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDKユーザー**: Python SDKには、キャッシュされた認証情報をクリアするためのプログラムによる`logout()`メソッドも用意されています。詳細については、[SDKリファレンス](reference/sdk-reference.md)を参照してください。
{% endhint %}

**例:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**特殊文字**: `$`、`!`、またはスペースなどの文字を含むパスワードは、一重引用符で囲んでください。
{% endhint %}

**出力:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### 認証情報の保存

キャッシュされた認証情報と設定は、**すべてのプラットフォーム**において、ユーザーのホームディレクトリ内の `.chloros` フォルダに保存されます：

| プラットフォーム | 認証情報のキャッシュパス |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

既存のバージョン上に新しいバージョンをインストールしても、Windows（インストーラー）およびLinux（`.deb`、`chloros-cli update`を含む）ではログイン情報が保持されます。 Chlorosをアンインストールする場合、またはLinuxパッケージを削除する場合は、キャッシュされたログイン情報が消去されるため、新規インストール後は再度サインインする必要があります。

### プランの有効期限とオフライン猶予期間

GUIに表示されるプランの有効期限は、ライセンスが無効になる時期を示しています。 月額定期購読の場合、有効期限は月末となります。年間定期購読の場合は、購読開始から1年後となります。

Chlorosはライセンスをオンラインで検証しますが、猶予期間内であればオフラインでの利用もサポートされています：

* サーバーでの検証が成功した場合は、その結果が**5分間**キャッシュされるため、通常の使用ではライセンスへのアクセス回数はごくわずかです。
* 署名済みでマシンに紐付けられたライセンスキャッシュにより、より長いオフライン期間もカバーされます：**月額プランの場合は30日間**、**年間プランの場合はサブスクリプションの有効期限まで（最大365日間）**です。
* 猶予期間が終了すると、そのマシンがライセンスサーバーに一度接続できるまで、プランは無料の「Iron」ティアに切り替わります。次の検証に成功すると、アクセスが再開されます。

### デバイス数の上限

Chloros+の各プランでは、登録可能なデバイス数が異なります。Chloros+アカウントでログインした各デバイスは、登録デバイス数にカウントされます。MAPIR Cloudのアカウントページで、デバイスの名前変更や削除を行うことができます。

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ プラン</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">対応デバイス</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

アカウントの正確なデバイス登録可能数は、MAPIR Cloudのアカウントページに表示されています。デバイスからログアウトすると、そのスロットは確実に解放されます。また、すでに登録済みのデバイスであれば、アカウントのデバイス登録上限に達している場合でも、いつでも再ログインが可能です。
