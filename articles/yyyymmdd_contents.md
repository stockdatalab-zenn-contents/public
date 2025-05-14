---
title: "【内部管理用】目次"
emoji: "🐕"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false  #変えない
---

## タイトルと概要の紐づけ（No★：原案レベル）
|      No      |         メインタイトル                   |         サブタイトル                    | 概要 |
|     ----     |                     ----                 |                     ----                | ---- |
|      01      | はじめまして                             | ー                                      |自己紹介と直近投稿予定の記事の紹介。 |
|      02      | アラサー個人投資家の未来戦略             | お金と時間の使い方                      |「マインドセット」「金融リテラシー」「資源の最適化」の３軸で考える。 |
|      03      | アラサー個人投資家の未来戦略             | 自分に合った投資戦略の組み立て方        |投資戦略の組み立て方を交えて、アカウントのコンセプトについて説明。 |
|      XX      | アラサー個人投資家の未来戦略             | 子育てを考える                          |金融リテラシー・英語・プログラミング |
|      04      | 駆け出しデータサイエンティストの未来戦略 | 得意分野をどう決める？キャリア設計入門  |XXXXX |
|      ★      | 駆け出しデータサイエンティストの未来戦略 | キャリアを考える（深堀版）              |どのスキルを身に着けるか具体的に整理 |
|      05      | 環境構築                                 | XXXXX                                   |XXXXX |
|      06      | スクレイピング                           | XXXXX                                   |PDFをスクレイピング |
|      07      | youtubeから情報収集                      | XXXXX                                   |動画をCopilot in edgeで要約する |
|      08      | twitterから情報収集                      | XXXXX                                   |XXXXX |
|      09      | BIツール                                 | XXXXX                                   |XXXXX |
|      10      | AutoMLツール                             | XXXXX                                   |XXXXX |
|      ★      | No4の記事とか目次のメモに沿ったネタ      | XXXXX                                   |XXXXX |
|      ★      | 気になる業界のニュースを定期的に読む     | XXXXX                                   |XXXXX |
|      ★      | 扱ってみたいデータを使って分析してみる   | XXXXX                                   |XXXXX |
|      ★      | 手法をひとつ選んで、手を動かしてみる     | XXXXX                                   |XXXXX |


## Slug とタイトルの紐づけ
|      No      |         メインタイトル                   |         サブタイトル                    |    公開日    |             slug           |
|     ----     |                     ----                 |                     ----                |     ----     |            ----            |
|      01      | はじめまして                             | ー                                      | 20250513_Tue | dac1854447a6b0.md          |
|      02      | アラサー個人投資家の未来戦略             | お金と時間の使い方                      | 20250513_Tue | 20250513_idea_investor.md  |
|      03      | アラサー個人投資家の未来戦略             | 自分に合った投資戦略の組み立て方        | 20250514_Wed | 20250515_idea_investor.md  |
|      04      | 駆け出しデータサイエンティストの未来戦略 | 得意分野をどう決める？キャリア設計入門  | 20250515_Thu | 20250516_idea_ds.md  |
|      05      | 環境構築                                 | XXXXX                                   | 20250518_Sun | 20250518_tech_env.md  |
|      06      | スクレイピング                           | XXXXX                                   | 202505XX_XXX | 202505XX_tech_scrapping.md  |
|      07      | youtubeから情報収集                      | XXXXX                                   | 202505XX_XXX | 202505XX_tech_youtube.md  |
|      08      | twitterから情報収集                      | XXXXX                                   | 202505XX_XXX | 202505XX_tech_twitter.md  |
|      09      | BIツール                                 | XXXXX                                   | 202505XX_XXX | 202505XX_tech_bi.md  |
|      10      | AutoMLツール                             | XXXXX                                   | 202505XX_XXX | 202505XX_tech_automl.md  |


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
|      03      |公開済み |    完　    |    完　    |    完　    |    完　    |    完　    |    ー　    |
|      04      |作成中   |    完　    |    完　    |    完　    |    　　    |    　　    |    ー　    |
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



:::details 分析手法
#### 学習形態によるアプローチ分類
|      カテゴリ      |                     概要                 |              代表例             |
|     ----           |                     ----                 |              ----               |
|教師あり学習        |正解ラベル付きで予測モデルを学習          |線形回帰、決定木、SVM、深層学習など|
|教師なし学習        |データ構造の発見・パターン認識            |クラスタリング（k-means）、PCA、Autoencoder|
|強化学習            |試行錯誤から報酬最大化を学ぶ              |Q学習、DQN、Policy Gradient|
|半教師あり学習      |ラベル付き＋未ラベルの混合学習            |Semi-supervised GANなど（補足程度）|
|自己教師あり学習    |ラベルを自動生成して学習                  |SimCLR、BYOLなど（補足程度）|


#### モデル構造・アルゴリズムによる分類
|      カテゴリ      |                     概要                 |              代表例             |
|     ----           |                     ----                 |              ----               |
|線形モデル系|入力と出力を線形関係で表現|線形回帰、ロジスティック回帰、Ridge|
|決定木系|ルールベースでの分岐学習|決定木、ランダムフォレスト、XGBoost|
|SVM系|マージン最大化の境界学習|SVM、SVR|
|ベイズ系|確率モデルに基づいた推論|Naive Bayes、ベイズネット|
|ニューラルネット系|多層・非線形表現の学習|MLP、CNN、RNN、Transformer（深層学習）|
|その他の非パラメトリック系|シンプルな距離ベースなど|k-NN・LDA|


#### 応用領域（分析対象）による分類
|      カテゴリ      |                     概要                 |              代表例             |
|     ----           |                     ----                 |              ----               |
|自然言語処理|人間の言語をコンピュータで理解・生成する技術。テキストの分類、要約、感情分析などを行う。|RNN、Transformer、Naive Bayes|
|音声処理|音声データの解析と生成を行う技術。音声認識（音声→テキスト）や音声合成（テキスト→音声）などを行う。|RNN、WaveNet、Transformer|
|画像処理|デジタル画像の解析・加工技術。画像の特徴抽出（分類）や物体認識、フィルタリングなどを行う。|CNN、Vision Transformer|
|時系列処理|時間に沿ったデータの解析技術。トレンドや季節性を見つけ、未来の予測を行う。|ARIMA、LSTM、Prophet|
|グラフデータ処理|ノードとエッジからなるグラフデータの解析技術。|GCN、GraphSAGE（ニューラルネット系）|


#### 分析アプローチの目的別の分類
|      カテゴリ      |                     概要                 |              代表例             |
|     ----           |                     ----                 |              ----               |
| 探索的データ分析（EDA）|データの構造や傾向を把握するための初期分析。仮説なしでも実施可能。|ヒストグラム、散布図、相関分析|
| 記述分析|現状を正確に把握・報告する。業務報告やKPIモニタリングに有用。|集計表、レポート、BIダッシュボード|
| 仮説検証分析|特定の仮説の正しさを検証する。要件定義や施策評価と相性が良い。|回帰分析、ABテスト、因果推論|
| 予測分析|将来の数値や行動を予測する。モデル精度・運用体制が重要。|機械学習モデル、時系列予測、深層学習|
| 最適化・改善分析|結果をもとに行動を最適化・自動化。PDCAや業務オペレーションと密接。|グロースハック、施策最適化、強化学習|


:::



:::details 分析対象の具体例
- **構造化データ（表形式のデータ）**
売上記録、ユーザー行動履歴、センサーデータなど
- **自然言語（テキストデータ）**
顧客レビュー、問い合わせ履歴、SNS投稿など
- **画像**
医療画像、製造現場のカメラ映像、商品写真など

:::



:::details template

## はじめに


## さいごに
:::






## アーカイブ
:::details アーカイブ

:::
