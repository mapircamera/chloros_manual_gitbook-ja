# 対応言語

Chlorosは、**世界38カ国語**のインターフェースを完全に対応しており、世界中のユーザーが利用可能です。 デスクトップ、ブラウザ、CLI、およびPython SDKなど、すべてのインターフェースで即座に言語を切り替えることができます。

Chlorosは以下の言語に対応しています：

| # | 言語 | 現地名 | CLI コード |
|---|----------|-------------|----------|
| 1 | 🇺🇸 英語 | English | `en` |
| 2 | 🇪🇸 スペイン語 | Español | `es` |
| 3 | 🇵🇹 ポルトガル語 | Português | `pt` |
| 4 | 🇫🇷 フランス語 | Français | `fr` |
| 5 | 🇩🇪 ドイツ語 | Deutsch | `de` |
| 6 | 🇮🇹 イタリア語 | Italiano | `it` |
| 7 | 🇯🇵 日本語 | 日本語 | `ja` |
| 8 | 🇰🇷 韓国語 | 한국어 | `ko` |
| 9 | 🇨🇳 中国語（簡体字） | 简体中文 | `zh` |
| 10 | 🇹🇼 中国語（繁体字） | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 ロシア語 | Русский | `ru` |
| 12 | 🇳🇱 オランダ語 | Nederlands | `nl` |
| 13 | 🇸🇦 アラビア語 | العربية | `ar` |
| 14 | 🇵🇱 ポーランド語 | Polski | `pl` |
| 15 | 🇹🇷 トルコ語 | Türkçe | `tr` |
| 16 | 🇮🇳 ヒンディー語 | हिंदी | `hi` |
| 17 | 🇮🇩 インドネシア語 | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 ベトナム語 | Tiếng Việt | `vi` |
| 19 | 🇹🇭 タイ語 | ไทย | `th` |
| 20 | 🇸🇪 スウェーデン語 | Svenska | `sv` |
| 21 | 🇩🇰 デンマーク語 | Dansk | `da` |
| 22 | 🇳🇴 ノルウェー語 | Norsk | `no` |
| 23 | 🇫🇮 フィンランド語 | Suomi | `fi` |
| 24 | 🇬🇷 ギリシャ語 | Ελληνικά | `el` |
| 25 | 🇨🇿 チェコ語 | Čeština | `cs` |
| 26 | 🇭🇺 ハンガリー語 | Magyar | `hu` |
| 27 | 🇷🇴 ルーマニア語 | Română | `ro` |
| 28 | 🇺🇦 ウクライナ語 | Українська | `uk` |
| 29 | 🇧🇷 ブラジルポルトガル語 | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 広東語 | 粵語 | `zh-HK` |
| 31 | 🇲🇾 マレー語 | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 スロバキア語 | Slovenčina | `sk` |
| 33 | 🇧🇬 ブルガリア語 | Български | `bg` |
| 34 | 🇭🇷 クロアチア語 | Hrvatski | `hr` |
| 35 | 🇱🇹 リトアニア語 | Lietuvių | `lt` |
| 36 | 🇱🇻 ラトビア語 | Latviešu | `lv` |
| 37 | 🇪🇪 エストニア語 | Eesti | `et` |
| 38 | 🇸🇮 スロベニア語 | Slovenščina | `sl` |

## 言語の変更方法

### Chloros デスクトップ/ブラウザの場合

1. アプリケーションの設定を開きます
2. 言語選択メニューに移動します
3. リストからお好みの言語を選択します
4. インターフェースが即座に更新されます

### Chloros CLI

`language` コマンドを使用して、CLI インターフェースの言語を表示または変更します：

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

詳細については、[CLI ドキュメント](CLI.md)を参照してください。

### Chloros、Python、SDK

SDK を初期化する際に言語パラメータを設定すると、メッセージや出力を希望の言語で表示できます。

## 対応範囲

以下のすべてにおいて、38言語すべてが完全にサポートされています：

* **Chloros デスクトップ** - GUIの完全な翻訳
* **Chloros ブラウザ** - すべての言語に対応したWebインターフェース
* **Chloros CLI** - コマンドラインインターフェースおよび出力メッセージ
* **Chloros Python SDK** - API メッセージおよびドキュメント

言語サポートにより、世界中のユーザーが障壁なく母国語で効率的に作業できるようになります。
