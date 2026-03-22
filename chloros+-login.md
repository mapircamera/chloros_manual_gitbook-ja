# Chloros+ ログイン

## Chloros および Chloros (ブラウザ版) ログイン

ユーザー <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> サイドバーメニューから、Chloros+ アカウントにログインし、追加機能を利用できるようになります。

ログインすると、アカウントの詳細が表示されます：

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI ログイン

Chloros+ の認証情報を使用してログインし、CLI の処理を有効にしてください。Linux（GUIなし）では、これがライセンスを有効にする唯一の方法です。

**構文:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK ユーザー**: Python SDK では、キャッシュされた認証情報をクリアするためのプログラムによる `logout()` メソッドも提供されています。 詳細については、[Python SDK ドキュメント](api-python-sdk.md#logout)を参照してください。
{% endhint %}

**例:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**特殊文字**: `$`、`!`、またはスペースなどの文字を含むパスワードは、一重引用符で囲んでください。
{% endhint %}

**出力:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### 認証情報の保存

キャッシュされた認証情報は、プラットフォーム固有の場所に保存されます:

| プラットフォーム | 認証情報キャッシュのパス |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### プランの有効期限

GUIに表示されるプランの有効期限は、ライセンスが無効になる時期を示しています。月額定期購読の場合、有効期限は月末となります。年間購読の場合、購読開始から1年後となります。ライセンスの確認には、30日間の猶予期間を設けて、毎月インターネット接続が必要です。

### デバイス制限

各Chloros+プランでは、登録可能なデバイス数が異なります。Chloros+アカウントでログインする各デバイスは、登録デバイス数にカウントされます。MAPIRクラウドアカウントページから、デバイスの名前変更や削除を行うことができます。

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ プラン</th><th align="center">COPPER</th><th align="center">ブロンズ</th><th align="center">シルバー</th><th align="center">ゴールド</th></tr></thead><tbody><tr><td align="right">対応デバイス</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
