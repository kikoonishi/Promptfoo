# Promptfoo

## 特徴
- コラボレーション: シェアできる [(リンク)](https://www.promptfoo.dev/docs/usage/sharing/)、Webブラウザでテストの結果が見られる
- プライベート: ローカルでランするため
- 違うモデルの比較をできる
- 複数のプロンプト、変数を表形式で比較できる
- 複数のテストが行える 
- シナリオを作り、変数を変えてテストできる 
- OpenAI prompt formatのメッセージをサポートしている -> 複数のメッセージ(システムプロンプト含む)をセットすることができる 
- 既存のテストケースとプロンプトを使用し、新たなテストケースを含むデータセットを作成できる
- nodeパッケージとして使用可能
- プロンプトはテキスト、JSON、ファイルパス、マークダウン、などで書くことができる。モデルに合わせてプロンプトを変えることができる
- プロンプトファンクション: javascriptやpython を使って、ロジックをプロンプトに組み込むことができる　[リンク](https://www.promptfoo.dev/docs/configuration/parameters/#prompt-functions)
- Nunjucksのフィルターを使用し、プロンプト内の変数にjsのファンクションを使用することができる [リンク](https://www.promptfoo.dev/docs/configuration/parameters/#nunjucks-filters)
- アウトプットをダグすることができる [リンク](https://www.promptfoo.dev/docs/configuration/expected-outputs/#tagging-outputs)
- アウトプットをフォーマットすることができる [リンク](https://www.promptfoo.dev/docs/configuration/expected-outputs/#processing-and-formatting-outputs)
- パフォーマンスデータ?(telemetry) がレコードされる時: init, eval, viewなどがランされた時、アサーションが使用された時　[リンク](https://www.promptfoo.dev/docs/configuration/telemetry/)
- JestやVitest、mocha/Chaiなどのフレームワークと共に使用できる [(Jest, Vitest)](https://www.promptfoo.dev/docs/integrations/jest/),[(mocha/Chai)](https://www.promptfoo.dev/docs/integrations/mocha-chai/)
- 

```
prompts:
 - file://prompt1.txt
 - file://prompt2.txt
providers:
 
- openai:gpt-4o-mini
 - vertex:gemini-pro
tests:
 - vars:
     language: French
     input: Hello world
   assert:
     - type: contains-json
 - vars:
     language: German
     input: How's it going?
```
## コマンドラインでできること　[(ドキュメンテーション)](https://www.promptfoo.dev/docs/usage/command-line/)


- init [directory] - Initialize a new project with dummy files.
- eval - Evaluate prompts and models. This is the command you'll be using the most!
- view - Start a browser UI for visualization of results.
- share - Create a URL that can be shared online.
- cache - Manage cache.
    - cache clear
- list - List various resources like evaluations, prompts, and datasets.
    - list evals
    - list prompts
    - list datasets
- show <id> - Show details of a specific resource (evaluation, prompt, dataset).
- delete <id> - Delete a resource by its ID (currently, just evaluations)
- feedback <message> - Send feedback to the Promptfoo developers.


## アサーション [(ドキュメンテーション)](https://www.promptfoo.dev/docs/configuration/expected-outputs/)
- 必須事項を定義できる。下記の場合、jsonを含んでいないアウトプットは却下される。
  ```
  # jsonを含むか判断する
  # javascriptのコードを文章に対して走らせる
  assert:
       - type: contains-json
       - type: javascript
         value: output.toLowerCase().includes('bonjour')
  ```
- LLM outputに対してJavascriptのコードを走らせることができるその結果をアサーションとして使用できる
- 単語が含まれているかだけでなく、文章の意味的に近いかを判断することができる。どれぐらいのsimilarityを求めるか判断するスレッシュホールドも設定できる。
  ```
  # 意味的に近い文章を判断する
  assert:
       - type: similar
         value: was geht
         threshold: 0.6   # cosine similarity
  ```
- アサーションをcsvファイルから読み込むこともできる
- metricsを使用してそれぞれのアサーションにタグをつけることができる ->このタグは結果の表にも表示される
- もしすでにLLMアウトプットを持っている場合、そのアウトプットにアサーションを使用できる [リンク](https://www.promptfoo.dev/docs/configuration/expected-outputs/#running-assertions-directly-on-outputs)
  ```
  # outputs.jsonが元々あるアウトプットでasserts.yamlが使用したいアサーションを持つファイル
  promptfoo eval --assertions asserts.yaml --model-outputs outputs.json
  ```
  

## シナリオ [(ドキュメンテーション)](https://www.promptfoo.dev/docs/configuration/scenarios/)
```
You're a translator.  Translate this into {{language}}: {{input}}
---
Speak in {{language}}: {{input}}
```

シナリオ内に、変数とテスト、アサーションを入れることができる。
```
scenarios:
  - config:
      - vars:
          language: Spanish
          expectedHelloWorld: 'Hola mundo'
          expectedGoodMorning: 'Buenos días'
          expectedHowAreYou: '¿Cómo estás?'
      - vars:
          language: French
          expectedHelloWorld: 'Bonjour le monde'
          expectedGoodMorning: 'Bonjour'
          expectedHowAreYou: 'Comment ça va?'
      - vars:
          language: German
          expectedHelloWorld: 'Hallo Welt'
          expectedGoodMorning: 'Guten Morgen'
          expectedHowAreYou: 'Wie geht es dir?'
    tests:
      - description: Translated Hello World
        vars:
          input: 'Hello world'
        assert:
          - type: similar
            value: '{{expectedHelloWorld}}'
            threshold: 0.90
      - description: Translated Good Morning
        vars:
          input: 'Good morning'
        assert:
          - type: similar
            value: '{{expectedGoodMorning}}'
            threshold: 0.90
      - description: Translated How are you?
        vars:
          input: 'How are you?'
        assert:
          - type: similar
            value: '{{expectedHowAreYou}}'
            threshold: 0.90
```

## チャット形式の会話/スレッド (リンク)[https://www.promptfoo.dev/docs/configuration/chat/]

例:
```
[
  { "role": "system", "content": "You are a helpful assistant." },
  { "role": "user", "content": "Who won the world series in {{ year }}?" }
]
```
yaml形式
```
- role: system
  content: You are a helpful assistant.
- role: user
  content: Who won the world series in {{ year }}?
```

## データセットの作成 [リンク](https://www.promptfoo.dev/docs/configuration/datasets/)
- ``` promptfoo generate dataset ``` はプロンプトとすでにあるテストケースを使用し、新たなテストケースを作る。
- そのテストケースはさらなるevaluationに使用することができる
- データセットをファイルに書き起こすこともできる
- データセットを編集することもできる
- データセット作成をカスタマイズする [詳細](https://www.promptfoo.dev/docs/configuration/datasets/#customize-the-generation-process)


## テスト
複数のテストを行う際に、.yamlファイルにリストし、それをpromptfooconfig.yamlにファイルパスを入れることができる

```
# promptfooconfig.yaml
prompts:
  # ...

providers:
  # ...

tests: file://path/to/tests.yaml
```
下記のように、さらに別のyamlファイルをテストごとに作成し、リストすることもできる
```
# test.yaml
tests:
  - file://relative/path/to/normal_test.yaml
  - file://relative/path/to/special_test.yaml
  - file:///absolute/path/to/more_tests/*.yaml
```

テストをgoogle sheetsから読み込むこともできる。
テストファイルにアサーションを含むこともできる。[リンク](https://www.promptfoo.dev/docs/configuration/expected-outputs/#load-assertions-from-csv)

## Installation
install grobally npm:
```
npm install -g promptfoo
```
brew:
```
brew install promptfoo
```
use npx to run promptfoo w/o installing:
```
npx promptfoo@latest
```

## Library Usage
Install promptfoo as a library in your project:
```
npm install promptfoo
```

## Verify installation
## インストールを確認
```
promptfoo --version
```

## Initialization
```
promptfoo init
```
This will create a promptfooconfig.yaml placeholder in your current directory
a promptfooconfig.yaml がカレントディレクトリ内に作られる。


