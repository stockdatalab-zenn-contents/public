---
title: "【図解】OllamaとOpen WebUI でローカルLLMのRAG環境を構築する手順|RAGをDockerで動かす"
emoji: "👨‍🏫"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ollama", "Open WebUI", "RAG", "生成AI", "環境構築"]
published: true
---
## はじめに
生成AIの利活用が進む中で、LLMをローカル環境で動かしたいというニーズが高まっています。本記事では、**DockerとOllamaとOpen WebUIを組み合わせて、自分だけのナレッジベースを活用できるRAG環境をローカルに構築する手順**を図解を交えて紹介します。

構築する環境のイメージ図は以下です。
![](/images/20250627_tech_env_rag/env0.png =800x)

なお、筆者のPCは「ASUS ゲーミングノートPC：FX707VV-I7R4060A5200」（OS:Windows11、CPU：インテル Core i7-13620H プロセッサー、メモリ：16GB、ストレージ：1TB、GPU：NVIDIA GeForce RTX 4060）です。


## 1. Docker環境を構築する
記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」の「1. WSL2でUbuntuをインストールする」「2. Docker EngineをUbuntuにインストールする」「3. VSCodeやpythonをインストールする」を参照ください。
### ここまでの完成図
![](/images/20250627_tech_env_rag/env1.png =400x)


## 2. ローカルLLMを扱える環境を構築する
記事「[【図解】OllamaとOpen WebUI でローカルLLMの環境構築する手順|大規模言語モデル・生成AIをDockerで動かす](https://zenn.dev/stockdatalab/articles/20250626_tech_env_llm)」を参照ください。
### ここまでの完成図
チャット用モデルは、LlamaやELYZAを指しています。
![](/images/20250627_tech_env_rag/env2.png =800x)


## 3. 任意：Embedding用モデルをセッティングする
### 3-1. Embedding用モデルをコンテナに乗せる
RAGを行うためにはEmbedding（「意味の数値化」を行う処理）用のモデルを使います。デフォルトでは後述の画面で「sentence-transformers/all-MiniLM-L6-v2」が設定されていますが、好みのモデルを設定することもできます。必要に応じて、以下のコマンドのように、[Ollamaのサイト](https://ollama.com/search?c=embedding)からEmbedding用モデル（例：nomic-embed-text:v1.5）を選択してOllamaに乗せます。
```sh:Linux上で実行
$ docker exec ollama ollama pull nomic-embed-text:v1.5
$ docker exec ollama ollama pull mxbai-embed-large:335m
```
### 3-2. 使用するEmbedding用モデルを指定する
Open WebUIで起動されているブラウザから、以下の通り指定します。画面上に注釈がある通り、**Embedding用モデルを変更すると、後続の手順を再度行う必要があるのでご注意下さい**。
![](/images/20250627_tech_env_rag/setting1.png =800x)

### ここまでの完成図
![](/images/20250627_tech_env_rag/env3.png =800x)


## 4. ナレッジを登録する
### 4-1. ナレッジを登録する場所を作成する
Open WebUIで起動されているブラウザから、以下の通り作成します。
![](/images/20250627_tech_env_rag/setting2.png =800x)

### 4-2. ナレッジを登録する
4-1.で作成した場所に、ナレッジを以下の通り登録します。なお、今回は、以下の「RAG_test.txt」をナレッジとして登録しています。
```txt:RAG_test.txt
花子さんは、太郎君のことが好きです。
あやちゃんは、かほちゃんと喧嘩中です。
次郎君は、とても勉強ができます。
けん君は、スポーツがとても得意です。
```
![](/images/20250627_tech_env_rag/setting3.png =800x)

### ここまでの完成図
![](/images/20250627_tech_env_rag/env4.png =800x)


## 5. チャット用モデルとナレッジを繋げる
Open WebUIで起動されているブラウザから、以下の通り設定して繋げます。なお、Enbedding用モデルは、3.の手順を実行した時点で自動的にチャット用モデルと繋がるようになります。
&emsp;④：モデル名は任意の名前を設定
&emsp;⑤：使用したいチャット用モデルを指定
&emsp;⑥：任意の説明を記入
&emsp;⑦：4.の手順で作成したナレッジを選択
![](/images/20250627_tech_env_rag/setting4.png =800x)

### 完成図
![](/images/20250627_tech_env_rag/env0.png =800x)


## 6. 動作確認する
Open WebUIで起動されているブラウザから、5.の手順で作成したモデルを選択します。
![](/images/20250627_tech_env_rag/setting5.png =800x)
登録した「RAG_test.txt」の内容について質問してみると、ちゃんとナレッジの内容に沿った回答が返ってきます。
![](/images/20250627_tech_env_rag/setting6.png =800x)


## おわりに
本記事では、OllamaとOpen WebUIを使って、ローカル環境でRAGを構築する手順を紹介しました。これにより、ローカルPC上で自前のナレッジベースを活用した対話型AIシステムを構築することができ、プライバシーの確保やオフライン動作といった大きな利点があります。

ローカルでの生成AI活用は、個人でも十分手が届く時代になってきました。まずは手元の環境でRAGを動かしてみるところから、ぜひ生成AI活用の第一歩を踏み出してみてください。

