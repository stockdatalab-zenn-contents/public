---
title: "■内部用■下書き"
emoji: "🗣"	
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ollama", "Open WebUI", "RAG", "生成AI", "環境構築"]
published: false
---

#SEOも加味して全文の推敲や構成の提案し、リライト版の提示をお願いします。

#以下はブログです。
#仮のタイトルは「【図解】Docker・Ollama・Open WebUI で構築したローカルLLMをLangChainで動かす手順」です。
#理解しましたか。
--------
## はじめに
ローカル環境でLLMを動かすだけで満足していませんか？LangChainを使えば、単なるチャットだけでなく、外部ツールとの連携・ナレッジベースの活用・記憶機能など、本格的なアプリケーションの構築が可能になります。

本記事では、**Docker上に構築したOllama＋Open WebUIベースのローカルLLMを、LangChainで動かす方法**を、LangChainの主な機能の解説や図解を交えて解説します。

構築する環境のイメージ図は以下です。投げられたプロンプトとURLリスト（txtの外部文書）をもとに、ローカルLLMで回答を生成する仕組みを構築します。
![](/images/yyyymmdd_backup/env0.png =800x)

なお、筆者のPCは「ASUS ゲーミングノートPC：FX707VV-I7R4060A5200」（OS:Windows11、CPU：インテル Core i7-13620H プロセッサー、メモリ：16GB、ストレージ：1TB、GPU：NVIDIA GeForce RTX 4060）です。


## 1. Dockerでpythonを扱える環境を構築する
記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」を参照ください。
### ここまでの完成図
![](/images/yyyymmdd_backup/env1.png =400x)


## 2. ローカルLLMを扱える環境を構築する
記事「[【図解】OllamaとOpen WebUI でローカルLLMの環境構築する手順|大規模言語モデル・生成AIをDockerで動かす](https://zenn.dev/stockdatalab/articles/20250626_tech_env_llm)」を参照ください。
### ここまでの完成図
チャット用モデルはLlamaやELYZAなどを、Embedding用モデルはnomic-embed-textなどを指しています。
![](/images/yyyymmdd_backup/env2.png =600x)


## 3. LangChainでOllama上のモデルを扱えるようにする
### 3-1. LangChainでできること
実際に環境を整える前に、LangChainの主要な６つの機能について簡単に触れておきます。公式の説明は[こちら](https://python.langchain.com/api_reference/langchain/index.html)から参照できます。
![](/images/yyyymmdd_backup/langchain.png =800x)
:::details LangChainの主要な６つの機能
#### 1. models
LangChainで使用するモデルを指定する機能です。以下の3種類があります。
  - **chat_models**：チャットモデル用
  - **embeddings**：テキストをベクトル化するモデル用
  - **llms**：大規模言語モデル用
<br>
#### 2. prompts
モデルへの入力を組み立てる機能です。以下の4種類があります。
  - **prompt templates**
  プロンプトを、プログラムで扱いやすいテンプレートの形します。
  - **chat prompt templates**
  チャット形式のプロンプトを、役割や発話単位で構造的にテンプレート化します。
  - **example selectors**
  複数の例から、入力に最も関連するサンプルを動的に選択してプロンプトに挿入します。
  - **output parsers**
  LLMの出力結果を、プログラムで扱いやすい構造に変換します。
<br>
#### 3. chains
model, templates, chainsなどを連結する機能です。
<br>
#### 4. retrieval
ベクトル化（数値化）した外部文書を踏まえてモデルに回答させることができる機能です。
  
<br>
#### 5. memory
モデルにこれまでのやりとりを踏まえた入力をできる機能です。
<br>
#### 6. agents
モデルが様々なツールを選択しながら動作できるようにする機能です。
::: 


### 3-2. ツールやライブラリなどのインストール
LLM内部では以下のような処理を実行します。（スクリーンショットやOCR処理をしなくても、スクレイピングやクローリングでも技術的には目的を達成できますが、サイトの規約で禁止されている場合が多いので、この方法で掲載内容を取得します。）処理に必要なツールやライブラリなどのインストールします。
![](/images/yyyymmdd_backup/action.png =800x)
#### 3-2-1. Tesseractのインストール
OCR処理を実行するために必要なソフトウェアエンジン「Tesseract」をLinuxにインストールします。後続の手順でインストールするpythonライブラリ「pytesseract」に必要なものです。
```bash:★★★
sudo apt update
sudo apt install tesseract-ocr tesseract-ocr-jpn
```
#### 3-2-2. ChromeDriverのインストール
Google Chromeを自動操作するためのWebDriver「ChromeDriver」をインストールします。[こちらのQittaの記事](https://qiita.com/Chronos2500/items/7f56898af25523d04598)の手順に沿ってバージョンに合ったものをダウンロードします。後続の手順でインストールするpythonライブラリ「selenium」に必要なものです。

#### 3-2-3. ライブラリのインストール 
LangChainを扱うためのライブラリを1.で構築したコンテナにインストールします。
```bash:★★★
!pip install pillow pytesseract selenium langchain langchain-community
```
コンテナ上にpyファイルを作成し、ライブラリをインポートします。
```py:langchainAPI.py
import os
import time
from PIL import Image
import pytesseract
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

from langchain.vectorstores import Chroma
from langchain.embeddings import OllamaEmbeddings
from langchain.schema import Document
from langchain.llms import ChatOllama
from langchain.agents import Tool, initialize_agent, AgentType
```


### 3-3. LangChainのmodels機能を使う
利用したいチャット用モデルとEmbedding用モデルを指定します。
```py:langchainAPI.py
llm = ChatOllama(model = "elyza:jp8b")
embed = OllamaEmbeddings(model="nomic-embed-text:v1.5")
```
### ここまでの完成図
![](/images/yyyymmdd_backup/env3.png =800x)


### 3-4. LangChainのprompts機能を使う
利用したいプロンプトを作成します。
```py:langchainAPI.py
prompt = f"このURLの内容を確認して、以下の質問に答えてください。URL: {best_url}\n質問: {user_prompt}"
```
### ここまでの完成図
![](/images/yyyymmdd_backup/env4.png =800x)


### 3-5. LangChainのretrieval機能・chains機能を使う
URLリスト（txtの外部文書）を参照して、関連するURLを特定できるようにします。
```txt:urls.txt
株探（市況）のニュース：https://kabutan.jp/news/marketnews/?category=1
株探（材料）のニュース：https://kabutan.jp/news/marketnews/?category=2
株探（決算）のニュース：https://kabutan.jp/news/marketnews/?category=3
マネー現代（新着）のニュース：https://gendai.media/list/latest/money
マネー現代（マーケット）のニュース：https://gendai.media/list/genre/money/market
マネクリ（新着）のニュース：https://media.monex.co.jp/list/latest
マネクリ（マーケット）のニュース：https://media.monex.co.jp/ud/feature/code/market
マネックスの経済指標カレンダー：https://mst.monex.co.jp/pc/servlet/ITS/report/EconomyIndexCalendar
```
```py:langchainAPI.py
# ----  URLリストを読み込んでベクトル化 ----
def load_url_list(filepath: str) -> list[Document]:
    with open(filepath, encoding='utf-8') as f:
        lines = f.readlines()
    docs = []
    for line in lines:
        if '://' in line:
            title, url = line.strip().split('：')
            content = f"{title}\n{url}"
            docs.append(Document(page_content=content, metadata={"source": url}))
    return docs

# ----  RAGで関連URLを特定 ----
def find_relevant_url(user_query: str, docs: list[Document], persist_dir: str = "./chroma_store"):
    embedding = OllamaEmbeddings(model="nomic-embed-text:v1.5")

    # 初期化または既存ロード（初回は新規作成）
    if not os.path.exists(persist_dir) or not os.listdir(persist_dir):
        vectorstore = Chroma.from_documents(docs, embedding, persist_directory=persist_dir)
        vectorstore.persist()
    else:
        vectorstore = Chroma(persist_directory=persist_dir, embedding_function=embedding)

    # 類似度が最も高い1件を選択して、llmに参照させる
    retriever = vectorstore.as_retriever(search_kwargs={"k": 1})
    llm = ChatOllama(model="elyza:jp8b")
    chain = RetrievalQA.from_chain_type(llm=llm, retriever=retriever, return_source_documents=True)

    result = chain(user_query)
    best_doc = result['source_documents'][0]
    return best_doc.metadata["source"]

```
### ここまでの完成図
![](/images/yyyymmdd_backup/env6.png =800x)

### 3-6. LangChainのmemory機能を使う
LLMに会話履歴を持たせます。使用するメソッドとして、ConversationBufferMemoryやConversationSummaryMemoryが挙げられます。前者は、会話文全文を保持するため、文脈を捉えた回答をしてくれる確率は高いですがメモリを要します。後者は会話を要約して保持するため、文脈を捉える精度はやや劣りますがメモリを比較的圧迫せずに処理できます。
```py:langchainAPI.py
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)
```
### ここまでの完成図
![](/images/yyyymmdd_backup/env7.png =800x)


### 3-8. LangChainのagents機能を使う
agents機能を使い、LLMがtoolsから次のアクションを選択して実行できるようにします。今回はスクリーンショットを取得するtoolしか作成しませんが、将来的には汎用性のあるものにしたいと思っているので、敢えてagents機能を使用します。

```py:langchainAPI.py
# ---- スクリーンショット＋OCRを行うTool ----
def screenshot_ocr_tool(url: str) -> str:
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--disable-gpu')
    options.add_argument('--window-size=1920x1080')

    driver = webdriver.Chrome(options=options)
    driver.get(url)
    time.sleep(3)  # ページの読み込み時間を考慮

    # 全範囲のスクショ取得のために、ページ全体の高さを取得
    page_height = driver.execute_script("return document.body.scrollHeight")
    driver.set_window_size(1920, page_height)
    time.sleep(1)  # サイズ変更後の再描画待ち

    driver.save_screenshot("screenshot.png")
    driver.quit()

    image = Image.open("screenshot.png")
    text = pytesseract.image_to_string(image, lang='jpn')
    return f"[{url} のOCR抽出結果]\n\n{text}"

# ---- LangChain Agentを作成 ----
def build_agent():
    # 使用するモデルを指定
    llm = ChatOllama(model="elyza:jp8b")

    # 使用するツールを作成
    screenshot_tool = Tool(
        name="screenshot_reader",
        func=screenshot_ocr_tool,
        description="指定されたURLのページを開いて、スクリーンショットから日本語の本文をOCRで抽出して読み取ります"
    )

    # 会話履歴を保持するメモリを定義
    memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)

    # agentを構築
    agent = initialize_agent(
        tools=[screenshot_tool],
        llm=llm,
        agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
        tools_return_direct=False,
        memory=memory,
        verbose=True
    )

    return agent
```
### ここまでの完成図
:::details pythonコード全文
```py:langchainAPI.py
import os
import time
from PIL import Image
import pytesseract
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

from langchain.vectorstores import Chroma
from langchain.embeddings import OllamaEmbeddings
from langchain.schema import Document
from langchain.llms import ChatOllama
from langchain.agents import Tool, initialize_agent, AgentType


# ---- URLリストを読み込んでベクトル化 ----
def load_url_list(filepath: str) -> list[Document]:
    with open(filepath, encoding='utf-8') as f:
        lines = f.readlines()
    docs = []
    for line in lines:
        if '://' in line:
            title, url = line.strip().split('：')
            content = f"{title}\n{url}"
            docs.append(Document(page_content=content, metadata={"source": url}))
    return docs


# ---- RAGで最も関連性の高いURLを取得 ----
def find_relevant_url(user_query: str, docs: list[Document], persist_dir: str = "./chroma_store"):

	# 使用するモデルを指定
    embedding = OllamaEmbeddings(model="nomic-embed-text:v1.5")

	# ベクトルストアを構築・起動時に1度初期化
    if not os.path.exists(persist_dir) or not os.listdir(persist_dir):
        vectorstore = Chroma.from_documents(docs, embedding, persist_directory=persist_dir)
        vectorstore.persist()
    else:
        vectorstore = Chroma(persist_directory=persist_dir, embedding_function=embedding)

    retriever = vectorstore.as_retriever(search_kwargs={"k": 1})
    llm = ChatOllama(model="elyza:jp8b")

    # 1件だけ取得
    related_docs = retriever.get_relevant_documents(user_query)
    return related_docs[0].metadata["source"] if related_docs else None


# ---- スクリーンショット＋OCRを行うTool ----
def screenshot_ocr_tool(url: str) -> str:
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--disable-gpu')
    options.add_argument('--window-size=1920x1080')

    driver = webdriver.Chrome(options=options)
    driver.get(url)
    time.sleep(3)  # ページの読み込み時間を考慮

    # ページ全体の高さを取得
    page_height = driver.execute_script("return document.body.scrollHeight")
    driver.set_window_size(1920, page_height)
    time.sleep(1)  # サイズ変更後の再描画待ち

    driver.save_screenshot("screenshot.png")
    driver.quit()

    image = Image.open("screenshot.png")
    text = pytesseract.image_to_string(image, lang='jpn')
    return f"[{url} のOCR抽出結果]\n\n{text}"


# ---- LangChain Agentを作成 ----
def build_agent():
    llm = ChatOllama(model="elyza:jp8b")

    screenshot_tool = Tool(
        name="screenshot_reader",
        func=screenshot_ocr_tool,
        description="指定されたURLのページを開いて、スクリーンショットから日本語の本文をOCRで抽出して読み取ります"
    )

    # 会話履歴を保持するメモリを定義
    memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)

    agent = initialize_agent(
        tools=[screenshot_tool],
        llm=llm,
        agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
        tools_return_direct=False,
        memory=memory,
        verbose=True
    )

    return agent

# ---- メイン処理 ----
def main():
    # ユーザーの質問を受け取る
    user_prompt = input("🧑 質問を入力してください：\n> ")

    # URLリストを読み込んでRAGで最も関連するURLを取得
    docs = load_url_list("./doc/urls.txt")
    best_url = find_relevant_url(user_prompt, docs)

    if not best_url:
        print("関連するURLが見つかりませんでした。")
        return

    print(f"選ばれたURL: {best_url}")

    # Agentを使ってスクリーンショットToolを呼び出す
    agent = build_agent()
    final_prompt = f"このURLの内容を確認して、以下の質問に答えてください。\nURL: {best_url}\n質問: {user_prompt}"

    response = agent.run(final_prompt)
    print(f"\n回答:\n{response}")


if __name__ == "__main__":
    main()

```
:::
![](/images/yyyymmdd_backup/env8.png =800x)


## 4. Open WebUIとLangChain＋Ollamaを繋ぐAPIを作成する
LangChainで構築した処理を、Open WebUIなど外部から呼び出せるようにするためには、FastAPIなどを使ってAPI化する必要があります。このセクションでは、LangChainで作成したRAG処理をAPI化するコードを紹介します。Open WebUIのツール連携機能（Function Calling）を使って、このAPIを呼び出すことが可能になります。

:::details 参考：FastAPIの実装の型
```py:
# FastAPIのインポート
from fastapi import FastAPI

# FastAPIアプリのインスタンス（Webサーバーの"本体"）「app」を作成する
app = FastAPI()

# パスとHTTPメソッドを指定
# 直下の関数がリクエストの処理を担います。
@app.get("/")
def root():
    return {"mock": "Hello World"}
```
:::


Open WebUIはブラウザ上で動作するため、異なるポートのFastAPIへHTTPリクエストを送るにはCORS設定が必要です。たとえば、Open WebUI（ポート3000）とFastAPI（ポート8000）はオリジンが異なると見なされ、CORSを許可しないとブラウザ側でアクセスエラーになります。以下はCORSを許可するpythonコードです。
```py:langchainAPI.py
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # Open WebUI のアドレスのみ許可
    allow_methods=["GET", "POST"],
    allow_headers=["*"],
)
```

### ここまでの完成図
:::details pythonコード全文
```py:langchainAPI.py
import os
import time
from typing import List
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from PIL import Image
import pytesseract
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

from langchain.vectorstores import Chroma
from langchain.embeddings import OllamaEmbeddings
from langchain.schema import Document
from langchain.llms import ChatOllama
from langchain.agents import Tool, initialize_agent, AgentType
from langchain.memory import ConversationBufferMemory

# ---- FastAPI初期化 ----
app = FastAPI()


# ---- CORS設定 ----
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # Open WebUI のアドレスのみ許可
    allow_methods=["GET", "POST"],
    allow_headers=["*"],
)

# ---- リクエストボディ定義（promptを受け取ることを明示） ----
class AskRequest(BaseModel):
    prompt: str


# ---- URLリストを読み込んでベクトル化 ----
def load_url_list(filepath: str) -> List[Document]:
    with open(filepath, encoding='utf-8') as f:
        lines = f.readlines()
    docs = []
    for line in lines:
        if '://' in line:
            title, url = line.strip().split('：')
            content = f"{title}\n{url}"
            docs.append(Document(page_content=content, metadata={"source": url}))
    return docs


# ---- RAGで最も関連性の高いURLを取得 ----
def find_relevant_url(user_query: str, docs: List[Document], persist_dir: str = "./chroma_store"):
    embedding = OllamaEmbeddings(model="nomic-embed-text:v1.5")

    if not os.path.exists(persist_dir) or not os.listdir(persist_dir):
        vectorstore = Chroma.from_documents(docs, embedding, persist_directory=persist_dir)
        vectorstore.persist()
    else:
        vectorstore = Chroma(persist_directory=persist_dir, embedding_function=embedding)

    retriever = vectorstore.as_retriever(search_kwargs={"k": 1})
    llm = ChatOllama(model="elyza:jp8b")
    related_docs = retriever.get_relevant_documents(user_query)
    return related_docs[0].metadata["source"] if related_docs else None


# ---- スクリーンショット＋OCRを行うTool ----
def screenshot_ocr_tool(url: str) -> str:
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--disable-gpu')
    options.add_argument('--window-size=1920x1080')

    driver = webdriver.Chrome(options=options)
    driver.get(url)
    time.sleep(3)  # ページの読み込み時間を考慮

    # ページ全体の高さを取得
    page_height = driver.execute_script("return document.body.scrollHeight")
    driver.set_window_size(1920, page_height)
    time.sleep(1)  # サイズ変更後の再描画待ち

    driver.save_screenshot("screenshot.png")
    driver.quit()

    image = Image.open("screenshot.png")
    text = pytesseract.image_to_string(image, lang='jpn')
    return f"[{url} のOCR抽出結果]\n\n{text}"


# ---- LangChain Agent作成 ----
def build_agent():
    llm = ChatOllama(model="elyza:jp8b")

    screenshot_tool = Tool(
        name="screenshot_reader",
        func=screenshot_ocr_tool,
        description="指定されたURLのページを開いて、スクリーンショットから本文をOCRで読み取る"
    )

    memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)

    agent = initialize_agent(
        tools=[screenshot_tool],
        llm=llm,
        agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
        tools_return_direct=False,
        memory=memory,
        verbose=True
    )

    return agent


# ---- APIエンドポイント：POST /ask ----
@app.post("/ask")
def ask_question(request: AskRequest):
    user_prompt = request.prompt
    try:
        docs = load_url_list("./doc/urls.txt")
        best_url = find_relevant_url(user_prompt, docs)

        if not best_url:
            raise HTTPException(status_code=404, detail="関連するURLが見つかりませんでした。")

        agent = build_agent()
        final_prompt = f"このURLの内容を確認して、以下の質問に答えてください。\nURL: {best_url}\n質問: {user_prompt}"
        response = agent.run(final_prompt)
        return {"result": response, "url": best_url}

    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


# ---- サーバー起動処理 ----
# このpyファイルを実行すると、127.0.0.1:8000 でFastAPI サーバーが起動
# uvicorn はFastAPIのサーバー起動に使うライブラリ
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)

```
:::
![](/images/yyyymmdd_backup/env9.png =800x)

## 5. Open WebUIとLangChain＋Ollamaを作成したAPIで繋ぐ
![](/images/yyyymmdd_backup/memo.png =600x)
### 完成図
![](/images/yyyymmdd_backup/env0.png =800x)

## 6. 動作確認する
![](/images/util/dummy_black.png =150x)

## おわりに
本記事では、Docker環境上でOllamaとOpen WebUIを組み合わせて構築したローカルLLMを、LangChainから活用する方法を解説しました。Open WebUIとAPI経由で接続することで、GUIを維持しながらLangChainの機能を裏側で活用できるようになります。これからローカルLLMとLangChainを組み合わせた応用アプリに挑戦したい方の、第一歩として参考になれば幸いです。


## メモ
このセクションでは、LangChainで作成したRAG処理を /rag?query=... の形式で呼び出せるAPIとして公開するコードを紹介します。
```py:langchainAPI.py
from fastapi import FastAPI

# FastAPIアプリのインスタンス（Webサーバーの"本体"）「app」を作成する
app = FastAPI()

# / というパスに対する GETリクエスト を処理する関数を登録
@app.get("/")
def read_root():
    return {"Hello": "World"}

# /items/{item_id} というURLにアクセスしたときの GETリクエストを処理
@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": f"APIのid：{item_id}"}

# リクエストが「http://127.0.0.1:8000/items/42?q=FastAPI」の場合、
# レスポンスは{"item_id": 42, "q": "FastAPI"}
@app.get("/items/{item_id}")
def read_item(item_id: int, q: Optional[str] = None):
    return {
        "item_id": item_id,
        "q": q  # ← クエリパラメータの値をそのまま返す
    }

# このファイルが直接実行されたときに、uvicorn を使ってFastAPIアプリを起動
# uvicorn はFastAPIのサーバー起動に使うライブラリ
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)
```
