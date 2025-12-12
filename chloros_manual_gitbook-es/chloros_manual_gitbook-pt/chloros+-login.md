# クロロス+ ログイン

## クロロスとクロロス（ブラウザ）ログイン

ユーザー <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> サイドバー メニューを使用すると、Chloros+ アカウントにログインし、追加機能のロックを解除できます。

ログインすると、アカウントの詳細が表示されます:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLIログイン

Chloros+ 認証情報を使用してログインし、CLI 処理を有効にします。

**構文：**

```bash
chloros-cli login <email> <password>
```

**例：**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% ヒント スタイル="警告" %}
**特殊文字**: `$`、`!`、スペースなどの文字を含むパスワードは一重引用符で囲みます。
{% エンドヒント %}

**出力：**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### プランの有効期限

GUI のプランの有効期限は、ライセンスがいつ無効になるかを示します。定期的な月次サブスクリプションの場合、有効期限は月末になります。年間サブスクリプションの場合、サブスクリプションを開始してから 1 年後となります。ライセンス チェックでは、月次のインターネット接続を確認する必要があり、30 日間の猶予期間があります。

### デバイス制限

Chloros+ プランごとに提供される登録デバイスの数は異なります。 Chloros+ アカウントでログインした各デバイスは、登録済みデバイスの数としてカウントされます。 MAPIR Cloud アカウント ページでデバイスの名前を変更したり削除したりできます。

<table><thead><tr><th width="168.5999755859375" align="right">クロロス+計画</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">デバイスサポート済み</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
