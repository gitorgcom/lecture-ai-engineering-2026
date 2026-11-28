# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：

 - app.agent:   
 search_materials, make_quiz

 - app.mcp_agent:   
 glossary_search_glossary, fetch_fetch, aws_aws___read_documentation, aws_aws___search_documentation, aws_aws___list_regions, aws_aws___get_regional_availability, aws_aws___retrieve_skill

- STEP 2 の [tool call]：  
[tool call] glossary_search_glossary({'query': '3レベル'})

- STEP 3 の [tool call]：  
[tool call] fetch_fetch({'url': 'https://modelcontextprotocol.io/'})  
[tool call] glossary_search_glossary({'query': 'MCP'})

- STEP 4 の [tool call]：  
[tool call] aws_aws___search_documentation({'topics': ['general'], 'search_phrase': 'Amazon S3 バージョニング 概要'})  
[tool call] aws_aws___search_documentation({'search_phrase': 'Using versioning in S3 buckets', 'topics': ['general']})

- 気づき・疑問：  

1つの質問で複数のツールが呼ばれることがあり、[tool call] と [tool return] の流れで処理が見えるようになっているため、挙動がわかりやすい。

外部サーバ接続に関しては、信頼性などの事前調査が必要不可欠、また、送られる情報がどのように利用されるかも確認する必要があるのではないか。  


## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：基礎

- やったこと：用語集のMCPサーバーに、ツールをもう1つ公開

- 結果：

uv run python -m app.mcp_agent --list-tools
```
[glossary_get_lecture_summary]
  指定した回（1始まりの整数）の講義の要点を返す。
  
  「第3回では何を学ぶ？」のように、回の内容や関係を聞かれたときに使う。
  引数: {"lecture": {"title": "Lecture", "type": "integer"}}
```
glossary_get_lecture_summary が一覧に追加されていることを確認した。 

--

uv run python -m app.mcp_agent --trace \
 "第2回では何を学びますか？ "　　

```
第2回では、**ファインチューニングとLoRA**を学びます。

### 主な内容
1. **事前学習とファインチューニングの違い**
   - 事前学習で汎用的な言語能力を身につけたモデルに、目的に合うデータで追加学習させる仕組みや、向く場面・向かない場面を理解します。
2. **SFT（教師ありファインチューニング）**
   - 入力と望ましい出力のペアを用いて学習させる手法や、データの質・形式の重要性を学びます。
3. **PEFTとLoRA**
   - すべてのパラメータを更新するのではなく、小さな低ランク行列だけを効率的に学習させる「LoRA」の考え方を学びます。必要なGPUメモリや保存容量を大きく削減できる仕組みを理解します。
4. **演習**
   - Google Colab上で小さなモデル（Qwen3.5-0.8Bなど）を架空のFAQデータでLoRA学習させ、学習の前後でモデルの出力形式や回答の正確さがどのように変わるかを体験します。

---
関連回：第2回

--- trace ---
[tool call]   glossary_get_lecture_summary({'lecture': 2})
[tool return] glossary_get_lecture_summary -> # 第2回 ファインチューニングとLoRA
```
glossary_get_lecture_summary({'lecture': 2}) が呼ばれていることを確認した。
  

- 気づき・疑問：

サーバー側に `get_lecture_summary` を公開するだけで、エージェント側を変えなくても一覧に反映され、質問時にも呼ばれることが分かった。  

MCP では、サーバー側でツールを公開すれば、エージェント側を変えずに機能を増やせることが確認できた。