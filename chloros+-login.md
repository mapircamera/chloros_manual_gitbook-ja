# Chloros+ ログイン

## Chloros および Chloros (ブラウザ) ログイン

ユーザー <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> サイドバーメニューからChloros+アカウントにログインし、追加機能を有効化できます。

ログイン後、アカウント詳細が表示されます：

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI ログイン

Chloros+ の認証情報でログインし、CLI 処理を有効化してください。

**構文:**

```bash
chloros-cli login <email> <password>
```

**例:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**特殊文字**: `$`、`!`、スペースなどの文字を含むパスワードはシングルクォートで囲んでください。
{% endhint %}

**出力:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### プラン有効期限

GUIに表示されるプラン有効期限は、ライセンスが無効になる時期を示します。月額サブスクリプションの場合、有効期限は月末です。年間サブスクリプションの場合、サブスクリプション開始から1年後です。ライセンスチェックには月1回のインターネット接続による検証が必要で、30日間の猶予期間が設けられています。

### デバイス制限

各Chloros+プランでは登録可能なデバイス数が異なります。Chloros+アカウントでログインする各デバイスは登録デバイス数にカウントされます。MAPIR Cloudアカウントページでデバイスの名前変更や削除が可能です。

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+プラン</th><th align="center">COPPER</th><th align="center">ブロンズ</th><th align="center">シルバー</th><th align="center">ゴールド</th></tr></thead><tbody><tr><td align="right">対応デバイス</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
