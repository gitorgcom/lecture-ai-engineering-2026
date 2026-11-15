# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：

| 比較項目 | app.ask（第1回） | app.agent（第3回） |
|---|---|---|
| 用語集検索 | なし | あり |
| 講座固有の知識 | 用語集を参照しない | glossary.jsonから取得 |
| 回答 | 一般論を推測 | 講座の定義に基づく |
| 正確性 | 不正確 | 今回は定義と一致 |

AIエージェントはLLM単体では持っていない講座固有の情報を、ツールを通じて外部データから取得し、その情報を使って回答できる。

- `--trace` で見えたツール呼び出し（ツール名と引数）：search_glossary({'query': '3レベル'})

- 1つの質問での LLM 呼び出し回数（`requests`）：2回　requests=2, tool_calls=1

- 追加したツールと、docstring に書いた説明：
  - ツール名：`make_quiz(lecture: int)`
  - 説明：指定した回の用語から1つを選び、確認問題と答えを返す。「第2回の確認問題を出して」のように、復習用の問題を求められたときに使う。

## 宿題

- 取り組んだレベル：基礎

- やったこと1：app/agent.py の INSTRUCTIONS から「用語や講座に固有のことを答える前に、必ず search_glossary で用語集を確認する。」の1行を削除。
- 結果：削除後も検索ツールが1回呼び出され、requests=2、tool_calls=1で変化はなかった。
- 気づき・疑問：INSTRUCTIONSの検索必須の1行を削除しても、search_glossary は呼び出された。docstring の説明が影響した可能性がある。

- やったこと2：docstringの検索方法を "用語を1つだけ入れる" から "関連する用語をまとめて入れる" に変更。
- 結果：LoRAとQLoRAの検索が2回から1回に減少。requestsは2回のままで、tool_callsは2回から1回になった。
- 気づき・疑問：docstringの説明を変更することで、ツールの呼び出し回数や検索方法が変わることが分かった。

変更前：
--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0009450'), details={'text_prompt_tokens': 1350}, input_text_tokens=1350, input_tokens=1350, output_tokens=216, requests=2, tool_calls=2)

変更後：
--- trace ---
[tool call]   search_glossary({'query': 'LoRA QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[usage] RunUsage(cost=Decimal('0.0008560'), details={'text_prompt_tokens': 1045}, input_text_tokens=1045, input_tokens=1045, output_tokens=217, requests=2, tool_calls=1)