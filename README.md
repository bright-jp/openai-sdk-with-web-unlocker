# 高パフォーマンスのために OpenAI Agents SDK を Web Unlocker と統合する方法

[![Bright Data Promo](https://github.com/bright-jp/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/)

本ガイドでは、OpenAI の Agents SDK と Web Unlocker API を組み合わせて、Webサイトからデータを取得・処理できる堅牢な AI エージェントを Python で作成する方法を解説します。

- [OpenAI Agents SDK とは？](#what-is-openai-agents-sdk)
- [この AI エージェント手法における主な課題](#major-challenges-with-this-ai-agent-approach)
- [Agents SDK を Web Unlocker API と統合する](#integrating-agents-sdk-with-a-web-unlocker-api)
  - [ステップ #1: プロジェクトのセットアップ](#step-1-project-setup)
  - [ステップ #2: プロジェクトの依存関係をインストールして開始する](#step-2-install-the-projects-dependencies-and-get-started)
  - [ステップ #3: 環境変数の読み込みを設定する](#step-3-set-up-environment-variables-reading)
  - [ステップ #4: OpenAI Agents SDK を設定する](#step-4-set-up-openai-agents-sdk)
  - [ステップ #5: Web Unlocker API を設定する](#step-5-set-up-web-unlocker-api)
  - [ステップ #6: Webページのコンテンツ抽出関数を作成する](#step-6-create-the-web-page-content-extraction-function)
  - [ステップ #7: データモデルを定義する](#step-7-define-the-data-models)
  - [ステップ #8: エージェントのロジックを初期化する](#step-8-initialize-the-agent-logic)
  - [ステップ #9: 実行ループを実装する](#step-9-implement-the-execution-loop)
  - [ステップ #10: すべてを統合する](#step-10-put-it-all-together)
  - [ステップ #11: AI エージェントをテストする](#step-11-test-the-ai-agent)

## OpenAI Agents SDK とは？

[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) は、OpenAI によって作成されたオープンソースの Python ライブラリです。エージェントベースの AI アプリケーションを、分かりやすく効率的で、本番運用に耐える形で構築できるようにします。このライブラリは、OpenAI の以前の実験的プロジェクトである [Swarm](https://github.com/openai/swarm) を洗練させたバージョンです。

OpenAI Agents SDK は、抽象化を最小限に抑えつつ、いくつかの重要なコンポーネントを提供します。

- **Agents**: 特定の指示とツールを組み合わせてタスクを実行する LLM
- **Handoffs**: 必要に応じてエージェントが他のエージェントへタスクを引き継げるようにする仕組み
- **Guardrails**: エージェントの入力が期待される形式や要件に適合することを確認するための仕組み

これらの中核要素と Python の汎用性を組み合わせることで、エージェントとツール間の高度な相互作用を作成しやすくなります。

また SDK にはトレーシング機能が組み込まれており、エージェントのワークフローを可視化し、トラブルシューティングし、評価できます。特定のユースケース向けのモデルのファインチューニングもサポートしています。

## この AI エージェント手法における主な課題

多くの AI エージェントは、コンテンツの抽出やページ要素との相互作用など、Webページ上の操作を自動化することを目的としています。つまり、本質的には Web をプログラムでナビゲートする必要があります。

AI モデル自体による誤解釈の可能性に加えて、これらのエージェントが直面する最も重要な[障害](https://brightdata.jp/blog/web-data/anti-scraping-techniques)は、Webサイトの防御メカニズムへの対処です。多くのサイトがアンチボットやアンチスクレイピング技術を実装しており、AI エージェントを制限したり、誤った方向へ誘導したりする可能性があるためです。これは特に現在、[アンチ AI CAPTCHA や高度なボット検知システムがますます一般化している](https://hackernoon.com/ai-agent-browsers-are-failing-and-its-not-just-because-of-captchas)ことから、非常に重要です。

これらの障害を克服するには、[Bright Data's Web Unlocker API](https://brightdata.jp/products/web-unlocker) のようなソリューションと統合して、エージェントの Web ナビゲーション能力を強化する必要があります。このツールは、インターネットに接続する任意の HTTP クライアントやソリューション（AI エージェントを含む）で利用でき、Webアンロック用のゲートウェイとして機能します。任意のWebページから、クリーンでブロックされていない HTML を提供します。CAPTCHA、IP 制限、アクセス不能なコンテンツに悩まされることはありません。

## Agents SDK を Web Unlocker API と統合する

このガイド付きセクションでは、OpenAI Agents SDK と Bright Data's Web Unlocker API を統合し、次のことができる AI エージェントを構築する方法を学びます。

1. 任意のWebページのテキストから要約を作成する
2. eコマースサイトから構造化された商品情報を取得する
3. ニュース記事から重要な詳細を収集する

これを実現するために、エージェントは OpenAI Agents SDK に対して、任意のWebページのコンテンツを取得する仕組みとして Web Unlocker API を利用するよう指示します。コンテンツを取得したら、エージェントは AI ロジックを適用し、各タスクで必要な形にデータを抽出・整形します。

> **免責事項**:
> 
> 上記の3つのユースケースは単なる例です。ここで提示する方法論は、エージェントの挙動をカスタマイズすることで、他の多数のシナリオにも拡張できます。

最適なパフォーマンスを得るために、OpenAI Agents SDK と Bright Data's Web Unlocker API を使用して Python で AI スクレイピングエージェントを開発するには、次の手順に従ってください。

### 前提条件

このチュートリアルを開始する前に、以下を用意してください。

- コンピュータに Python 3 以上がインストールされていること
- 有効な Bright Data アカウント
- 有効な OpenAI アカウント
- HTTP リクエストに関する基礎理解
- Pydantic モデルに関する基本的な知識
- AI エージェントの動作に関する一般的な理解

### ステップ #1: プロジェクトのセットアップ

まず、システムに Python 3 がインストールされていることを確認してください。インストールされていない場合は、[Python をダウンロード](https://www.python.org/downloads/)して、お使いの OS に応じたインストール手順に従ってください。

ターミナルを起動し、スクレイピングエージェントプロジェクト用の新しいディレクトリを作成します。

```sh
mkdir openai-sdk-agent
```

`openai-sdk-agent` ディレクトリには、Python ベースで Agents SDK を利用するエージェントのすべてのコードが配置されます。

プロジェクトディレクトリに移動し、[仮想環境](https://docs.python.org/3/library/venv.html)を作成します。

```sh
cd openai-sdk-agent
python -m venv venv
```

お好みの Python IDE でプロジェクトディレクトリを開きます。[Python 拡張機能付き Visual Studio Code](https://code.visualstudio.com/docs/languages/python)や[PyCharm Community Edition](https://www.jetbrains.com/pycharm/download/#section=windows)が優れた選択肢です。

`openai-sdk-agent` ディレクトリ内に、`agent.py` という名前の新しい Python ファイルを作成します。ディレクトリ構成は次のようになります。

![The file structure of the AI agent project](https://github.com/bright-jp/openai-sdk-with-web-unlocker/blob/main/images/The-file-structure-of-the-AI-agent-project.png)

現時点では `scraper.py` は空の Python スクリプトですが、まもなく必要な AI エージェントロジックが含まれるようになります。

IDE のターミナルで仮想環境を有効化します。Linux または macOS では次のコマンドを実行します。

```sh
./env/bin/activate
```

同様に Windows では次を実行します。

```powershell
env/Scripts/activate
```

### ステップ #2: プロジェクトの依存関係をインストールして開始する

本プロジェクトでは、以下の Python ライブラリを使用します。

- [`openai-agents`](https://openai.github.io/openai-agents-python/): OpenAI Agents SDK。Python で AI エージェントを作成するために使用します。
- [`requests`](https://requests.readthedocs.io/en/latest/): Bright Data's Web Unlocker API に接続し、AI エージェントが処理するためのWebページの HTML コンテンツを取得するために使用します。詳しくは、[Python Requests ライブラリを使いこなす](https://brightdata.jp/blog/web-data/python-requests-guide)ガイドをご覧ください。
- [`pydantic`](https://docs.pydantic.dev/latest/): 構造化された出力モデルを定義し、エージェントが明確で検証された形式でデータを返せるようにします。
- [`markdownify`](https://python.langchain.com/docs/integrations/document_transformers/markdownify/): 生の HTML コンテンツをクリーンな Markdown に変換するために使用します。（この利点は後ほど説明します。）
- [`python-dotenv`](https://github.com/theskumar/python-dotenv): `.env` ファイルから環境変数を読み込むために使用します。ここに OpenAI と Bright Data の認証情報を保存します。

有効化された仮想環境で、次のコマンドですべてインストールします。

```sh
pip install requests pydantic openai-agents openai-agents markdownify python-dotenv
```

次に、`scraper.py` を以下の import と async のボイラープレートコードでセットアップします。

```python
import asyncio
from agents import Agent, RunResult, Runner, function_tool
import requests
from pydantic import BaseModel
from markdownify import markdownify as md
from dotenv import load_dotenv 

# AI agent logic...

async def run():
    # Call the async AI agent logic...

if __name__ == "__main__":
    asyncio.run(run())
```

### ステップ #3: 環境変数の読み込みを設定する

プロジェクトディレクトリに `.env` ファイルを作成します。

![Adding a .env file to your project](https://github.com/bright-jp/openai-sdk-with-web-unlocker/blob/main/images/Adding-a-.env-file-to-your-project.png)

このファイルには、API キーやシークレットトークンなどの環境変数を保存します。`.env` ファイルから環境変数を読み込むには、`dotenv` パッケージの `load_dotenv()` を使用します。

```python
load_dotenv()
```

これで、[`os.getenv()`](https://docs.python.org/3/library/os.html#os.getenv) を使って特定の環境変数にアクセスできます。

```python
os.getenv("ENV_NAME")
```

Python 標準ライブラリの [`os`](https://docs.python.org/3/library/os.html) を import することを忘れないでください。

```python
import os
```

### ステップ #4: OpenAI Agents SDK を設定する

OpenAI Agents SDK を使用するには、有効な OpenAI API key が必要です。まだ生成していない場合は、[OpenAI の公式ガイド](https://help.openai.com/en/articles/4936850-where-do-i-find-my-openai-api-key)に従って API key を作成してください。

取得したら、`.env` ファイルに次のように追加します。

```python
OPENAI_API_KEY="<YOUR_OPENAI_KEY>"
```

`<YOUR_OPENAI_KEY>` プレースホルダーは、実際のキーに置き換えてください。

追加のセットアップは不要です。`openai-agents` SDK は `OPENAI_API_KEY` 環境変数から API key を自動的に取得するよう設計されています。

### ステップ #5: Web Unlocker API を設定する

まだお持ちでない場合は、[Bright Data アカウントを作成](https://brightdata.jp/?hs_signup=1)してください。すでにお持ちの場合は、[ログイン](https://brightdata.jp/cp/start)してください。

次に、API トークンを取得するために [Bright Data の公式 Web Unlocker ドキュメント](https://docs.brightdata.com/scraping-automation/web-unlocker/quickstart)を参照してください。もしくは、次の手順に従ってください。

Bright Data の「User Dashboard」ページで、「Get proxy products」オプションを選択します。

![Clicking the "Get proxy products" option](https://github.com/bright-jp/openai-sdk-with-web-unlocker/blob/main/images/Clicking-the-Get-proxy-products-option.png)

製品テーブルで「unblocker」とラベル付けされた行を見つけ、クリックします。

![Clicking the "unblocker" row](https://github.com/bright-jp/openai-sdk-with-web-unlocker/blob/main/images/Clicking-the-unblocker-row.png)

「unlocker」ページで、クリップボードアイコンを使って API トークンをコピーします。

![Copying the API token](https://github.com/bright-jp/openai-sdk-with-web-unlocker/blob/main/images/Copying-the-API-token.png)

また、右上のトグルが「On」になっていることを確認し、Web Unlocker 製品がアクティブであることを確認してください。

「Configuration」タブで、最適な効果のために次のオプションが有効になっていることを確認してください。

- [Premium domains](https://docs.brightdata.com/scraping-automation/web-unlocker/configuration#premium-domains)
- [CAPTCHA Solver](https://brightdata.jp/products/web-unlocker/captcha-solver)

![Making sure that the premium options for effectiveness are enabled](https://media.brightdata.com/2025/04/Making-sure-that-the-premium-options-for-effectiveness-are-enabled.png)

`.env` ファイルに次の環境変数を追加します。

```python
BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN="<YOUR_BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN>"
```

プレースホルダーは実際の API トークンに置き換えてください。

### ステップ #6: Webページのコンテンツ抽出関数を作成する

次の処理を行う `get_page_content()` 関数を作成します。

1. `BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN` 環境変数を読み込む
2. `requests` を使って[提供された URL を使用し Bright Data's Web Unlocker API にリクエストを送信する](https://docs.brightdata.com/scraping-automation/web-unlocker/send-your-first-request)
3. API から返される生の HTML を取得する
4. HTML を Markdown に変換して返す

上記のロジックを次のように実装します。

```python
@function_tool
def get_page_content(url: str) -> str:
    """
    Retrieves the HTML content of a given web page using Bright Data's Web Unlocker API,
    bypassing anti-bot protections. The response is converted from raw HTML to Markdown
    for easier and cheaper processing.

    Args:
        url (str): The URL of the web page to scrape.

    Returns:
        str: The Markdown-formatted content of the requested page.
    """

    # Read the Bright Data's Web Unlocker API token from the envs
    BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN = os.getenv("BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN")

    # Configure the Web Unlocker API call
    api_url = "https://api.brightdata.com/request"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN}"
    }
    data = {
        "zone": "unblocker",
        "url": url,
        "format": "raw"
    }

    # Make the call to Web Uncloker to retrieve the unblocked HTML of the target page
    response = requests.post(api_url, headers=headers, data=json.dumps(data))

    # Extract the raw HTML response
    html = response.text

    # Convert the HTML to markdown and return it
    markdown_text = md(html)

    return markdown_text
```

**注 1**: 関数には [`@function_tool`](https://openai.github.io/openai-agents-python/tools/) で注釈を付ける必要があります。この特殊なデコレーターにより、OpenAI Agents SDK はこの関数を、エージェントが特定のアクションを実行するためのツールとして使用できることを認識します。このケースでは、この関数が、エージェントが処理するWebページのコンテンツを取得するために利用できる「エンジン」として機能します。

**注 2**: `get_page_content()` 関数は入力型を明示的に宣言する必要があります。  
省略すると、次のようなエラーに遭遇します: `Error getting response: Error code: 400 - {'error': {'message': "Invalid schema for function 'get_page_content': In context=('properties', 'url'), schema must have a 'type' key.``"`

生の HTML を Markdown に変換すると、パフォーマンス効率と費用対効果が向上します。これは HTML が非常に冗長で、スクリプト、スタイル、メタデータといった不要な要素を含むことが多いためです。AI エージェントはこれらのコンテンツを必要としません。エージェントがテキスト、リンク、画像などの要点のみを必要とする場合、Markdown ははるかにクリーンでコンパクトな表現を提供します。

具体的には、HTML-to-Markdown 変換により入力サイズを最大 99% 削減でき、次の両方を節約できます。

- トークン数（OpenAI モデル利用時のコスト削減）
- 処理時間（モデルは入力が小さいほど高速に動作します）

詳しくは、「[Why Are the New AI Agents Choosing Markdown Over HTML?](https://hackernoon.com/why-are-the-new-ai-agents-choosing-markdown-over-html)」の記事をご覧ください。

### ステップ #7: データモデルを定義する

正しく動作するために、OpenAI SDK のエージェントは、出力データの期待される構造を定義するための Pydantic モデルを必要とします。ここで構築するエージェントは、次の3つのいずれかの出力を返すことを思い出してください。

1.  ページの要約
2.  商品情報
3.  ニュース記事情報

対応する3つの [Pydantic モデル](https://docs.pydantic.dev/latest/concepts/models/)を定義しましょう。

```python
class Summary(BaseModel):
    summary: str

class Product(BaseModel):
    name: str
    price: Optional[float] = None
    currency: Optional[str] = None
    ratings: Optional[int] = None
    rating_score: Optional[float] = None

class News(BaseModel):
    title: str
    subtitle: Optional[str] = None
    authors: Optional[List[str]] = None
    text: str
    publication_date: Optional[str] = None
```

**注**: `Optional` を使用すると、エージェントをより汎用的で多用途にできます。すべてのページがスキーマで定義された全データを含むわけではないため、この柔軟性はフィールドが欠けている場合のエラーを防ぐのに役立ちます。

[`typing`](https://docs.python.org/3/library/typing.html) から `Optional` と `List` を import することを忘れないでください。

```python
from typing import Optional, List
```

### ステップ #8: エージェントのロジックを初期化する

`openai-agents` SDK の [`Agent`](https://platform.openai.com/docs/guides/agents) クラスを使用して、3つの特化エージェントを定義します。

```python
summarization_agent = Agent(
    name="Text Summarization Agent",
    instructions="You are a content summarization agent that summarizes the input text.",
    tools=[get_page_content],
    output_type=Summary,
)

product_info_agent = Agent(
    name="Product Information Agent",
    instructions="You are a product parsing agent that extracts product details from text.",
    tools=[get_page_content],
    output_type=Product,
)

news_info_agent = Agent(
    name="News Information Agent",
    instructions="You are a news parsing agent that extracts relevant news details from text.",
    tools=[get_page_content],
    output_type=News,
)
```

各エージェントは次の特徴を持ちます。

1. 意図する機能を説明する明確な instruction 文字列を含みます。OpenAI Agents SDK はこれを使用してエージェントの挙動を誘導します。
2. 入力データ（つまりWebページのコンテンツ）を取得するツールとして `get_page_content()` を使用します。
3. 先ほど定義した Pydantic モデル（`Summary`、`Product`、`News`）のいずれかで出力を返します。

ユーザーのリクエストを適切な特化エージェントへ自動的に振り分けるために、上位レベルのエージェントを定義します。

```python
routing_agent = Agent(
    name="Routing Agent",
    instructions=(
        "You are a high-level decision-making agent. Based on the user's request, "
        "hand off the task to the appropriate agent."
    ),
    handoffs=[summarization_agent, product_info_agent, news_info_agent],
) 
```

これが `run()` 関数内で問い合わせて AI エージェントロジックを駆動するエージェントです。

### ステップ #9: 実行ループを実装する

`run()` 関数に次のループを追加して、AI エージェントロジックを起動します。

```python
# Keep iterating until the use type "exit"
while True:
    # Read the user's request
    request = input("Your request -> ")
    # Stops the execution if the user types "exit"
    if request.lower() in ["exit"]:
        print("Exiting the agent...")
        break
    # Read the page URL to operate on
    url = input("Page URL -> ")

    # Routing the user's request to the right agent
    output = await Runner.run(routing_agent, input=f"{request} {url}")
    # Conver the agent's output to a JSON string
    json_output = json.dumps(output.final_output.model_dump(), indent=4)
    print(f"Output -> \n{json_output}\n\n")
```

このループは継続的にユーザー入力を監視し、各リクエストを適切なエージェント（summary、product、news）にルーティングして処理します。ユーザーのクエリと対象 URL を結合してロジックを実行し、[`json`](https://docs.python.org/3/library/json.html) を使用して構造化された結果を JSON 形式で表示します。次のように import してください。

```python
import json
```

### ステップ #10: すべてを統合する

これで `scraper.py` ファイルには次の内容が含まれているはずです。

```python
import asyncio
from agents import Agent, RunResult, Runner, function_tool
import requests
from pydantic import BaseModel
from markdownify import markdownify as md
from dotenv import load_dotenv
import os
from typing import Optional, List
import json

# Load the environment variables from the .env file
load_dotenv()

# Define the Pydantic output models for your AI agent
class Summary(BaseModel):
    summary: str

class Product(BaseModel):
    name: str
    price: Optional[float] = None
    currency: Optional[str] = None
    ratings: Optional[int] = None
    rating_score: Optional[float] = None

class News(BaseModel):
    title: str
    subtitle: Optional[str] = None
    authors: Optional[List[str]] = None
    text: str
    publication_date: Optional[str] = None

@function_tool
def get_page_content(url: str) -> str:
    """
    Retrieves the HTML content of a given web page using Bright Data's Web Unlocker API,
    bypassing anti-bot protections. The response is converted from raw HTML to Markdown
    for easier and cheaper processing.

    Args:
        url (str): The URL of the web page to scrape.

    Returns:
        str: The Markdown-formatted content of the requested page.
    """

    # Read the Bright Data's Web Unlocker API token from the envs
    BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN = os.getenv("BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN")

    # Configure the Web Unlocker API call
    api_url = "https://api.brightdata.com/request"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN}"
    }
    data = {
        "zone": "unblocker",
        "url": url,
        "format": "raw"
    }

    # Make the call to Web Uncloker to retrieve the unblocked HTML of the target page
    response = requests.post(api_url, headers=headers, data=json.dumps(data))

    # Extract the raw HTML response
    html = response.text

    # Convert the HTML to markdown and return it
    markdown_text = md(html)

    return markdown_text

# Define the individual OpenAI agents
summarization_agent = Agent(
    name="Text Summarization Agent",
    instructions="You are a content summarization agent that summarizes the input text.",
    tools=[get_page_content],
    output_type=Summary,
)

product_info_agent = Agent(
    name="Product Information Agent",
    instructions="You are a product parsing agent that extracts product details from text.",
    tools=[get_page_content],
    output_type=Product,
)

news_info_agent = Agent(
    name="News Information Agent",
    instructions="You are a news parsing agent that extracts relevant news details from text.",
    tools=[get_page_content],
    output_type=News,
)

# Define a high-level routing agent that delegates tasks to the appropriate specialized agent
routing_agent = Agent(
    name="Routing Agent",
    instructions=(
        "You are a high-level decision-making agent. Based on the user's request, "
        "hand off the task to the appropriate agent."
    ),
    handoffs=[summarization_agent, product_info_agent, news_info_agent],
)

async def run():
    # Keep iterating until the use type "exit"
    while True:
        # Read the user's request
        request = input("Your request -> ")
        # Stops the execution if the user types "exit"
        if request.lower() in ["exit"]:
            print("Exiting the agent...")
            break
        # Read the page URL to operate on
        url = input("Page URL -> ")

        # Routing the user's request to the right agent
        output = await Runner.run(routing_agent, input=f"{request} {url}")
        # Conver the agent's output to a JSON string
        json_output = json.dumps(output.final_output.model_dump(), indent=4)
        print(f"Output -> \n{json_output}\n\n")


if __name__ == "__main__":
    asyncio.run(run())
```

### ステップ #11: AI エージェントをテストする

AI エージェントを起動するには、次を実行します。

```sh
python agent.py
```

例えば、[Bright Data の AI services hub](https://brightdata.jp/ai) のコンテンツを要約したいとします。次のようにリクエストを入力してください。

![The input to get a summary of Bright Data's AI services](https://github.com/bright-jp/openai-sdk-with-web-unlocker/blob/main/images/The-input-to-get-a-summary-of-Bright-Datas-AI-services.png)

受け取る JSON 形式の結果は次のとおりです。

![The summary returned by your AI agent](https://github.com/bright-jp/openai-sdk-with-web-unlocker/blob/main/images/The-summary-returned-by-your-AI-agent.png)

次に、[PS5 listing](https://www.amazon.com/PlayStation%C2%AE5-console-slim-PlayStation-5/dp/B0CL61F39H/) のような Amazon の商品ページから商品データを抽出したいと想像してください。

![The Amazon PS5 page](https://github.com/bright-jp/openai-sdk-with-web-unlocker/blob/main/images/The-Amazon-PS5-page.png)

通常であれば、[Amazon の CAPTCHA とアンチボットシステム](https://brightdata.jp/blog/web-data/bypass-amazon-captcha) によってリクエストがブロックされるはずです。Web Unlocker API を使えば、AI エージェントはブロックされることなくページにアクセスして解析できます。

![Getting Amazon product data](https://media.brightdata.com/2025/04/Getting-Amazon-product-data.gif)

出力は次のようになります。

```json
{
    "name": "PlayStation\u00ae5 console (slim)",
    "price": 499.0,
    "currency": "USD",
    "ratings": 6321,
    "rating_score": 4.7
}
```

最後に、[Yahoo News の記事](https://www.yahoo.com/news/pope-francis-dies-88-080859417.html) から構造化されたニュース情報を取得したいとします。

![The target Yahoo News article](https://github.com/bright-jp/openai-sdk-with-web-unlocker/blob/main/images/The-target-Yahoo-News-article.png)

次の入力で実行できます。

```
Your request -> Give me news info
Page URL -> https://www.yahoo.com/news/pope-francis-dies-88-080859417.html
```

結果は次のようになります。

```json
{
    "title": "Pope Francis Dies at 88",
    "subtitle": null,
    "authors": [
        "Nick Vivarelli",
        "Wilson Chapman"
    ],
    "text": "Pope Francis, the 266th Catholic Church leader who tried to position the church to be more inclusive, died on Easter Monday, Vatican officials confirmed. He was 88. (omitted for brevity...)",
    "publication_date": "Mon, April 21, 2025 at 8:08 AM UTC"
}
```

## 結論

OpenAI SDK と Bright Data's [Web Unlocker API](https://brightdata.jp/products/web-unlocker) を組み合わせることで、ほぼあらゆるWebページ上で信頼性高く動作できる AI エージェントを開発できます。これは、Bright Data の製品とサービスが高度な AI 統合をどのように支援できるかを示す一例に過ぎません。

自律型 AI エージェント、垂直型 AI アプリ、基盤モデル、マルチモーダル AI、データプロバイダー、データパッケージなどを含む、[AI 製品](https://brightdata.jp/ai/products-for-ai)の全ラインナップをご覧ください。

Bright Data アカウントを作成し、AI エージェント開発向けの当社製品・サービスを今すぐお試しください！