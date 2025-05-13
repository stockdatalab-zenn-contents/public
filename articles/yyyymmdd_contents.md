---
title: "【内部管理用】目次"
emoji: "🐕"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false  #変えない
---
## Slug とタイトルの紐づけ
|      No      |         メインタイトル                   |         サブタイトル          |    公開日    |             slug           |
|     ----     |                     ----                 |                     ----      |     ----     |            ----            |
|      01      | はじめまして                             | ー                            | 20250513_Tue | dac1854447a6b0.md          |
|      02      | アラサー個人投資家の未来戦略             | お金と時間の使い方            | 20250513_Tue | 20250513_idea_investor.md  |
|      03      | アラサー個人投資家の未来戦略             | 投資戦略の組み立て方          | 20250514_Wed | 20250515_idea_investor.md  |
|      XX      | アラサー個人投資家の未来戦略             | 子育てを考える                | 2025XXXX_XXX | 202XXXX3_idea_childcare.md |
|      04      | 駆け出しデータサイエンティストの未来戦略 | キャリアを考える              | 20250515_Thu | 20250516_idea_ds.md  |
|      05      | 環境構築                                 | XXXXX                         | 20250518_Sun | 20250518_tech_env.md  |
|      06      | スクレイピング                           | XXXXX                         | 202505XX_XXX | 202505XX_tech_scrapping.md  |
|      07      | youtubeから情報収集                      | XXXXX                         | 202505XX_XXX | 202505XX_tech_youtube.md  |
|      08      | twitterから情報収集                      | XXXXX                         | 202505XX_XXX | 202505XX_tech_twitter.md  |
|      09      | BIツール                                 | XXXXX                         | 202505XX_XXX | 202505XX_tech_bi.md  |
|      10      | AutoMLツール                             | XXXXX                         | 202505XX_XXX | 202505XX_tech_automl.md  |
|      XX      | 気になる業界のニュースを定期的に読む     | XXXXX                         | 202505XX_XXX | 202505XX_tech_automl.md  |
|      XX      | 扱ってみたいデータを使って分析してみる   | XXXXX                         | 202505XX_XXX | 202505XX_tech_automl.md  |
|      XX      | 手法をひとつ選んで、手を動かしてみる     | XXXXX                         | 202505XX_XXX | 202505XX_tech_automl.md  |



## タイトルと概要の紐づけ
|      No      |         メインタイトル                   |         サブタイトル          | 概要 |
|     ----     |                     ----                 |                     ----      | ---- |
|      01      | はじめまして                             | ー                            |自己紹介と直近投稿予定の記事の紹介。 |
|      02      | アラサー個人投資家の未来戦略             | お金と時間の使い方            |「マインドセット」「金融リテラシー」「資源の最適化」の３軸で考える。 |
|      03      | アラサー個人投資家の未来戦略             | 投資戦略の組み立て方          |投資戦略の組み立て方を交えて、アカウントのコンセプトについて説明。 |
|      XX      | アラサー個人投資家の未来戦略             | 子育てを考える                |金融リテラシー・英語・プログラミング |
|      04      | 駆け出しデータサイエンティストの未来戦略 | キャリアを考える              |XXXXX |
|      05      | 環境構築                                 | XXXXX                         |XXXXX |
|      06      | スクレイピング                           | XXXXX                         |PDFをスクレイピング |
|      07      | youtubeから情報収集                      | XXXXX                         |動画をCopilot in edgeで要約する |
|      08      | twitterから情報収集                      | XXXXX                         |XXXXX |
|      09      | BIツール                                 | XXXXX                         |XXXXX |
|      10      | AutoMLツール                             | XXXXX                         |XXXXX |


## 進捗
※進捗・残タスク確認のポイント
- ネタ探し
- 記事の原案作成
- chatGPTによる文章校正
- 画像生成
- 文言最終確認

|      No      |  進捗   |  原案作成  |  画像生成  |  文章校正  |  最終確認  |  公開予約  |    備考    |
|     ----     |   ----  |    ----    |    ----    |    ----    |    ----    |    ----    |    ----    |
|      01      |公開済み |    完　    |    完　    |    完　    |    完　    |    ー　    |    ー　    |
|      02      |公開済み |    完　    |    完　    |    完　    |    完　    |    ー　    |    ー　    |
|      03      |20250514 |    完　    |    完　    |    完　    |    完　    |    完　    |    ー　    |
|      XX      |未作成   |    　　    |    　　    |    　　    |    　　    |    　　    |    ー　    |
|      04      |作成中   |    完　    |    　　    |    着手    |    　　    |    　　    |    校正文書の信憑性を確認・ 「参考：押さえておきたい基礎知識」の構成を検討   |
|      05      |未作成   |    　　    |    　　    |    　　    |    　　    |    　　    |    ー　    |
|      06      |未作成   |    　　    |    　　    |    　　    |    　　    |    　　    |    ー　    |
|      07      |未作成   |    　　    |    　　    |    　　    |    　　    |    　　    |    ー　    |
|      08      |未作成   |    　　    |    　　    |    　　    |    　　    |    　　    |    ー　    |





## 下書き
:::details データ収集に使えるAPIのまとめ

## はじめに
![](/images/util/dummy_white.png =150x)
#### 1. ソーシャルメディア／コミュニケーション系
|                   API名                                                          | 取得できる主なもの |
|                  ----                                                            | ---- |
| [X（旧Twitter）API](https://developer.x.com/en/docs/x-api)                       |投稿、ユーザー、トレンドなど |
| [Facebook Graph API](https://developers.facebook.com/docs/graph-api/)            |投稿、ページ、イベントなど |
| [Instagram Graph API](https://developers.facebook.com/products/instagram/apis/)  |投稿、インサイト、ハッシュタグなど |
| [YouTube Data API](https://developers.google.com/youtube/v3)                     |投稿、ユーザー、トレンドなど |


#### 2. Web／ニュース／コンテンツ系
|                   API名                                        | 取得できる主なもの |
|                  ----                                          | ---- |
| [News API](https://newsapi.org/docs)                           |主要ニュース記事（複数媒体） |
| [NYTimes API](https://developer.nytimes.com/apis)              |ニューヨーク・タイムズの記事、レビューなど |
| [GNews API](https://gnews.io/docs/v4)                          |Googleニュースベースのニュース記事 |
| [Wikipedia API](https://www.mediawiki.org/wiki/API:Main_page/ja)  |記事情報、履歴、リンク構造など |


#### 3. 金融／マーケット系
|                   API名                                                | 取得できる主なもの |
|                  ----                                                  | ---- |
|[Yahoo Finance API（非公式含む）](https://python-yahoofinance.readthedocs.io/en/latest/api.html)          |株価、財務指標、企業情報|
|[Alpha Vantage](https://www.alphavantage.co/documentation/)   |株価、為替、仮想通貨、テクニカル指標|
|[IEX Cloud](https://iexcloud.io/docs/api/)|リアルタイム株価、財務、時系列データ|
|[CoinGecko API](https://www.coingecko.com/en/api)|仮想通貨の価格、時価総額、取引所情報|
|[CoinMarketCap API](https://coinmarketcap.com/api/documentation/v1/)|仮想通貨の価格、時価総額、取引所情報|


#### 4. 経済／統計／公共データ系
|                   API名                                                | 取得できる主なもの |
|                  ----                                                  | ---- |
|[政府統計API（e-Stat APIなど）](https://www.e-stat.go.jp/api/jp)                                                          |日本の統計データ（人口、雇用など）|
|[World Bank API](https://datahelpdesk.worldbank.org/knowledgebase/articles/889392-about-the-indicators-api-documentation) |国際的な経済・開発指標|
|[OECD API]([https://data.oecd.org/api/sdmx-json-documentation/)                                                           |先進国の統計情報|
|[UN Data API](https://data.un.org/Host.aspx?Content=API)                                                                  |国連関連統計|


#### 5. テクノロジー／開発者向けプラットフォーム系
|                   API名                                                | 取得できる主なもの |
|                  ----                                                  | ---- |
|[GitHub API](https://docs.github.com/en/rest)                                                     |リポジトリ、ユーザー、Issue、Pull Requestなど|
|[Stack Exchange API](https://api.stackexchange.com/docs)                                          |質問・回答データ（Stack Overflowなど）|
|[Google Analytics Data API](https://developers.google.com/analytics/devguides/reporting/data/v1)  |ウェブサイトのアクセスログ、ユーザー行動|


#### 6. 位置情報／地理空間データ系
|                   API名                                                | 取得できる主なもの |
|                  ----                                                  | ---- |
|[Google Maps API](https://developers.google.com/maps/documentation)     |位置検索、経路、ジオコーディング|


#### 7. eコマース／レビュー／価格比較系
|                   API名                                                | 取得できる主なもの |
|                  ----                                                  | ---- |
|[Amazon Product Advertising API](https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html)  |商品情報、レビュー、価格|
|[Rakuten API](https://webservice.rakuten.co.jp/documentation/)                                              |商品検索、ジャンル、レビュー|


#### 8. 気象／自然データ系
|                   API名                                                | 取得できる主なもの |
|                  ----                                                  | ---- |
|[OpenWeatherMap API](https://openweathermap.org/api)                    |天気予報、過去の気象データ|
|[Weatherstack API](https://weatherstack.com/documentation)              |リアルタイム・履歴気象データ|
|[NASA APIs](https://api.nasa.gov/)                                      |気象衛星、気候、宇宙関連データ|


#### 9. ヘルスケア／医療系
|                   API名                                         | 取得できる主なもの |
|                  ----                                           | ---- |
|[CDC API（米国疾病予防センター）](https://data.cdc.gov/)         |感染症、健康データ|
|[WHO API](https://www.who.int/data/gho/info/gho-odata-api)       |世界の保健統計（注：公式APIは限定的）|
|[NIH ClinicalTrials API](https://clinicaltrials.gov/api/gui)     |臨床試験のデータ|


#### 10. 交通／モビリティ系
|                   API名                                                                     | 取得できる主なもの |
|                  ----                                                                       | ---- |
|[Google Directions API](https://developers.google.com/maps/documentation/directions/start)   |経路・所要時間|
|[東京メトロ API](https://developer.tokyometroapp.jp/)                                        |時刻表、運行情報|
|[JR東日本 API](https://www.jreast.co.jp/development/)                                        |時刻表、運行情報|
|[FlightAware API](https://flightaware.com/commercial/firehose/firehose\_documentation.rvt)   |フライト情報、到着遅延|
|[AviationStack API](https://aviationstack.com/documentation)                                 |フライト情報、到着遅延|

![](/images/util/dummy_white.png =150x)
## 【内部管理用】記事のステータス
ChatGPTの回答を転記しただけであり、信憑性等については未確認。


:::



:::details XXXXX

## はじめに


## さいごに
:::





## アーカイブ
:::details アーカイブ

:::
