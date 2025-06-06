---
title: "■内部用■下書き"
emoji: "🗣"	
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["クローリング", "スクレイピング", "銘柄", "個別株", "投資"]
published: false
---

#以下の内容でブログを書こうとしています。
#仮のタイトルは「自然言語処理と生成AI・LLMを組み合わせるために必要な知識：初心者向け」です。
#理解しましたか。
--------



こんにちは、データサイエンティスト兼個人投資家の「マチ」です。個別株を投資していて、テーマで銘柄を検索できるサイトがありますが、「そもそもどんなテーマがあるのか分からん」と思ったことはないでしょうか。そこで、今回は銘柄とキーワードを紐づけて一覧を作ろうと思います。


1.データ収集
テーマで銘柄を検索できるサイトやテーマ一覧を提供しているサイトの多くは、残念ながら、スクレイピングやクローリングが利用規約で禁止とされています。なので、APIやダウンロードできる以下の資材を使用します。
https://www.jpx.co.jp/markets/statistics-equities/misc/01.html
https://qiita.com/XBRLJapan/items/27e623b8ca871740f352#1-edinet-api%E3%81%A8%E3%81%AF
https://zenn.dev/sre_holdings/articles/dc0909ad62429f

- [日本取引所グループ|東証上場会社情報サービス](https://www2.jpx.co.jp/tseHpFront/JJK010010Action.do?Show=Show)
- [みんかぶ｜株テーマ一覧](https://minkabu.jp/theme)



