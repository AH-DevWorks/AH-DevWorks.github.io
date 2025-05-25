---
Title: 【Side Project】用ChatGPT API來翻譯小說
Date: 2025-05-17
Tags: ["blog","API","OpenAI","tools"]
Description: "能透過OpenAI Responses API大量翻譯長文本的簡單程式"
draft: true
---

> 算是在做 Deep Learning 過程中的副產品（?
>> 雖然標題寫「翻譯小說」，但只要不違反OpenAI的使用規範，從小說、散文到論文都能翻譯
> 

※這篇較屬於技術面說明，實際成果請見：
1. [synchronous-openai-doc-translator]()
2. [asynchronous-openai-doc-trnaslator]()


---

### Responses API vs. Chat Completions API
+ 根據 OpenAI 的 [Document](https://platform.openai.com/docs/guides/responses-vs-chat-completions)，跟 OpenAI 模型互動的方式有兩種：Responses API ＆ Chat Completions API
+ 兩者差異基本如下：
    |Capabilities|Chat Completions API|Responses API|
    |---|---|---|
    |Text generation|O|O|
    |Audio|X|Coming soon|
    |Vision|O|O|
    |Structured Outputs|O|O|
    |Function calling|O|O|
    |Web search|X|O|
    |File search|X|O|
    |Computer use|X|O|
    |Code interpreter|X|Coming soon|

### 選 Responses API 的原因
+ Chat Completions API 需要手動追蹤各狀態間的變化
+ 如同OpenAI在Document裡自己寫的，「Multi-step conversational logic and reasoning are easier to implement with the Responses API」。
+ 在 Conversation State (對話狀態) 上， Response 有個方便的參數(parameter)「**previous_response_id**」用於串連每次的回應，也就是能建立連貫的對話，不讓模型每次的回應都像是開一個全新的聊天。
  + 這一點在「不使用本文接續方式」的長篇文章翻譯時特別重要，能保持語氣、措辭、人名地名等詞彙前後一致


#### previous_response_id 參數應用範例
+ 以下是根據[OpenAI給的範例](https://platform.openai.com/docs/guides/conversation-state?api-mode=responses#openai-apis-for-conversation-state)（1.先讓模型講個笑話｜2.再讓它自己解釋剛才生成的笑話哪裡好笑）稍微修改後的程式碼以及輸出，供參考：
```py
import os
from dotenv import load_dotenv
from openai import OpenAI

# 從環境變數 .env 匯入 API_KEY，並建立OpenAI連線
load_dotenv()
API_KEY = os.getenv("API_KEY")
client = OpenAI(api_key=API_KEY)

if __name__ == "__main__":
    
    # 第一次呼叫
    response = client.responses.create(
        model="gpt-4.1-nano-2025-04-14",  # 用最便宜的gpt-4.1-nano模型
        input="繁體中文跟我講一個笑話。",
    )
    print(f"第一次回覆：{response.output_text}")
    print(response.usage)   # 每次model的回覆都帶有usage屬性，可呼叫來看這輪的input/output tokens數量

    print("=" * 50) # 分隔線

    # 第二次呼叫
    second_response = client.responses.create(
        model="gpt-4.1-nano-2025-04-14",
        previous_response_id=response.id,
        input=[{"role": "user", "content": "解釋為什麼這個笑話好笑。"}],
    )
    print(f"第二次回覆：{second_response.output_text}")
    print(second_response.usage)
```
```bash
// 輸出
第一次回覆：當然可以！這裡有一個簡單的笑話給你：

有一隻螃蟹走進酒吧，跟吧台的魚說：「你知道我為什麼喜歡海嗎？」  
魚回答：「為什麼呀？」  
螃蟹說：「因為我在那裡可以海啦！」

希望你喜歡！還要聽另一個嗎？
ResponseUsage(input_tokens=18, input_tokens_details=InputTokensDetails(cached_tokens=0), output_tokens=92, output_tokens_details=OutputTokensDetails(reasoning_tokens=0), total_tokens=110)
=======================================
第二次回覆：這個笑話好笑之處在於它運用了語言的巧妙雙關和出乎意料的轉變。

1. **雙關語**：螃蟹說「我在那裡可以海啦」，其實是將「海」與「嘿」（嘿呀）或「嗨」的發音聯想起來，形成一個語音遊戲。同時，它也暗示螃蟹喜歡在海裡，這是它的熟悉的環境。

2. **出乎意料的轉折**：通常人們預期螃蟹會說「我喜歡海」（因為它生活在海裡），但它卻說「可以海啦」，帶有一點調皮和幽默的語氣，讓情境變得輕鬆有趣。

3. **動物擬人化**：把螃蟹擬人化，讓它有意識地跟魚聊天，增加了趣味性，讓聽眾產生意料之外的聯想。

總之，這個笑話巧在語言的雙關以及設定出乎意料的角色行為，使得聽者產生輕鬆愉快的笑意。
ResponseUsage(input_tokens=130, input_tokens_details=InputTokensDetails(cached_tokens=0), output_tokens=285, output_tokens_details=OutputTokensDetails(reasoning_tokens=0), total_tokens=415)
```

#### 既然 previous_response_id 那麼方便，那為什麼實際synchronous-openai-doc-translator程式裡不使用？
　　原因很簡單：對於這次長篇小說翻譯而言，token會產生類似「費波那契式增長」的效果——總 token 數會隨對話輪數累積，出現非線性式的增長——導致每次的input token看起來差不多，但其實會像滾雪球，越滾越大。
+ 從前述「讓AI講笑話後自己解釋」的程式也可以證明：
  + 姑且不管笑話好不好笑，程式裡面可以看到兩次input的字數差不多，理論上Tokens數量也應該不會差太多。
  + 但前後兩次「ResponseUsage」裡面的「input_tokens」卻有明顯落差（18-->130）
  + 合理推測，`第二次的input_tokens = 130 可能等於：第一次的total_tokens（110）＋ 第二次「解釋為什麼這個笑話好笑。」這幾個字的tokens（tokens:20）`
+ 「`每輪請求的 token 數 ≈ 歷史所有 input + output tokens + 本輪 input tokens`」這個情形，不只會導致「不知不覺就花了很多＄＄」（畢竟API是按tokens計價的），更大的問題是「可能超過**模型的 Context Window 限制**」。


### Context Window（上下文視窗）限制
> Reference: [Conversation state](https://platform.openai.com/docs/guides/conversation-state?api-mode=responses)
　　打個比方：每個人的能力有高有低，但都有極限，比如小明10分鐘內只能「記下10個英文單字 ＋ 思考怎麼用這10個單字寫幾組英文短句 ＋ 把句子寫出來」，這就是他的極限了。
　　AI的`context window`概念就有點類似：不管是哪家的AI（ChatGPT、Grok、Claude...）模型，都有一個「context window」，可以想成是模型的「短期記憶上限」。我們每次跟AI模型交談，都會有我們使用者這次的「`Input tokens`」，以及模型這次的「`Output tokens`」——如果是會深入思考的模型種類，還要加上使用者通常看不到的，它在回覆前的「腦中推論(`Reasoning tokens`)」。
　　對模型來說，上述這些「tokens」全部加起來，不能超過它的「context window」上限，否則就會被「截斷」——比如這次它的回答變得不完整，話講到一半就忽然斷了。
+ 就像我們今天硬要小明想辦法記下20個英文單字然後造句，明顯就超過他目前的能力極限
　　也因為這樣，context window 是衡量模型好壞的指標之一，通常越新的模型，context window　越大（e.g. GPT-4o: 128,000 | o3: 200,000），代表它更具備「長文章理解」、「長篇對話」、「讀取並理解使用者單次輸入大量的 prompt 」等能力。

### 模型用於長文章連貫翻譯上的限制
+ 回到原本的問題：目前專案API搭配的參數 previous_response_id 在使用上有個重大限制：`「Input Tokens（所有累積的歷史問答紀錄＆這一次的new_input_tokens）＋ 模型這次的Reasoning Tokens ＋ Output Tokens」==> 可能直接超過 Context Window 的 Limit`

#### 解決 Context Window Limit 問題
　　之前想到的方法有這幾種：
1. 使用者自己手動控制歷史 Token 總數（例如刪除比較早的紀錄） ==> 明顯不佳，違背原本「自動化」的懶人初衷（
2. 用模型摘要前幾輪對話，再用簡化過的摘要續接新輸入 ==> 不佳，要再多套上其他模型
3. 使用 system role 提示模型記住關鍵背景：仍需要人工確認「哪些是關鍵背景」且較難自動化


### openai.RateLimitError: Error code: 429 問題
```bash
...raise self._make_status_error_from_response(err.response) from None
openai.RateLimitError: Error code: 429 - {'error': {'message': 'Rate limit reached for gpt-4.1-mini in organization org-xxxxxxx on tokens per min (TPM): Limit 200000, Used 124325, Requested 81594. Please try again in 1.775s. Visit https://platform.openai.com/account/rate-limits to learn more.', 'type': 'tokens', 'param': None, 'code': 'rate_limit_exceeded'}}
```
+ ※失敗的請求也會計入模型每分鐘限制（TPM & RPM），所以一失敗就立刻重新request也沒用

#### 解法1 - Exponential Backoff (指數退避 / 指數輪詢)
+ 使用 OpenAI Documents 官方建議的 [**Exponential Backoff**（指數退避法）](https://platform.openai.com/docs/guides/rate-limits#retrying-with-exponential-backoff)，加上 **Jitter**（隨機抖動）
+ Exponential Backoff 的作法：
  1. 當請求失敗時，等待一小段時間（例如2秒，可自訂）後重試
  2. 若仍失敗，則將等待時間拉長再重試，時間拉長的方式也可自訂
  4. 持續上述1~2流程，直到請求成功或達到重試上限為止
  5. 加入 jitter (例如`sleep_time = delay * (1 + random.random())`)，避免多個請求在相同時間重試
+ ex.[api_translator.py] 內 translate_text function 修改：
  ```py
  def translate_text(ori_text: str, translated_paragraphs: str, prompts: str=prefix_prompt, store: bool=True, max_retries: int = 8, initial_delay: float = 1.0, backoff_factor: float = 2.0, max_delay: float = 60.0) -> str :
    attempt = 0
    delay = initial_delay
    while attempt < max_retries:
        try:
            response = client.responses.create(
                model=MODEL,
                input=f"{prompts}\n{translated_paragraphs}\n{ori_text}",
                store=store
            )
            return response.output_text
        except RateLimitError as e:
            attempt += 1
            print(f"[WARNING] RateLimitError，第 {attempt} 次重試...")
            sleep_time = delay * (1 + random.random())
            sleep_time = min(sleep_time, max_delay)
            print(f">> [Waiting] {sleep_time:.2f} 秒後重試...")
            time.sleep(sleep_time)
            delay *= backoff_factor
        except Exception as e:
            print(f"[ERROR] 非速率限制錯誤：{e}")
            raise e
    raise Exception(f"[Fail] 超過最大重試次數（{max_retries} 次），仍無法完成請求。")
  ```

+ ==> **實際成效**???
<!-- （仍碰過[Fail] 超過最大重試次數（8 次）情況），若是 RateLimitError，使用 Exponential Backoff 等了最多60秒，理論上不會再次跳 Exception 才對
後台檢查也不是「單次請求的 Token 數量過大（上下文內容過長）」、「模型之間共享 TPM / RPM 的限制」等其他問題 -->

#### 解法2 - Batch API 批次處理
+ 前述程式都是 synchronous（同步）方式一段一段翻譯，但 Batch API 可以 asynchronous（異步）處理大量請求
+ 根據 OpenAI 的 [Batch API Documents](https://platform.openai.com/docs/guides/batch)，用 Batch API 不只有更寬的rate limits，成本可降低50%，還能在 24 小時內取得處理結果。
+ 「非時間敏感」（例如不需要立即得到回覆）的請求——像是Running evaluations、Classifying large datasets、Embedding content repositories等任務——用 Batch API 比較好

+ 技術簡述
  + import sys ==> 用sys.argv列表讀取/儲存使用者執行commands時輸入的所有參數，根據啟動main.py時的參數來呼叫對應功能
    + sys.argv[0]: 腳本本身的名稱（如main.py）
    + sys.argv[1]開始: 使用者輸入的參數（如start）


##### Batch API 缺點
+ 缺點顯而易見，實際跑過一次就知道：儘管OpenAI宣稱用Batch API能顯著提昇Rate Limit（以o4-mini為例，100,000-->1,000,000），但實際model能處理的token長度感覺並沒有什麼提昇。就算文件total tokens長度遠低於Batch Limit，模型還是會直接罷工（如下o3-mini跟o4-mini）。
  + `o4-mini` - 要做不做的感覺
    ```bash
    1. [知道有好幾章，但只翻譯第一章] ...我屏息傾聽，他則清晰地逐點陳述他那近乎匪夷所思、卻又合情合理的推論。──未完，請續閱下回。
    2. [完全無視原文] 明白您的要求，已準備就緒。請隨時提供原文段落，我將忠實、通順、完整地翻譯成繁體中文。
    3. [直接擺爛] 很抱歉，但我無法協助您進行此請求。不過，我可以為您提供該作品的摘要或針對其中某些段落提供說明與分析。
    + [其他幾本都只翻譯第一章]
    ```
  + `o3-mini` - 直接跟你說不行
    ```bash
    1. 抱歉，但您提供的文本過於龐大，目前無法在一次回覆中完整翻譯這整部小說全文。請您分段提供原文，我將逐段提供完整、通順且忠實於原文的繁體中文翻譯。
    2. 抱歉，但您提供的原文文本太長，超出了我目前一次處理與回覆的字數限制。請您將文本分成較小的段落並分次提供，以便我能完整、通順地翻譯成繁體中文。
    3. 抱歉，但您提供的原文文本太長，超出了我目前一次處理與回覆的字數限制。請您將文本分成較小的段落並分次提供，以便我能完整、通順地翻譯成繁體中文。
    ……
    ```

+ 到這裡或許會想：「那就跟 synchronous 的程式一樣，把原文拆一拆不就好了？」
+ 但 Batch API ——或者說 asynchronous 非同步—— 本身「批次同時送出處理」的特性，較適合「無上下文依賴關係」的任務，所以才希望能一次把整本書塞給model，不然就算一本只拆成上下集，還是很有可能出現「不一致（人物名詞／專有名詞／翻譯語境語氣……）」的情況。
+ 此外，OpenAI Batch API 本身不支援像是previous_response_id的參數；再加上[Doc](https://platform.openai.com/docs/guides/batch)也提到「the output line order may not match the input line order (輸出順序可能跟輸入順序不同)」，簡單來說就是回傳結果可能整個亂掉。

#### 小結
1. **[synchronous-openai-doc-trnaslator]()**
  + 用previous_response_id參數可以輕易做到連貫對話，基本等於沒界面的ChatGPT，用多少付多少(credits計價)，整體算起來應該會比訂閱ChatGPT Plus便宜
    + GUI界面其實也可以用tkinter之類的套件、土法煉鋼刻出來；最近似乎還有[AnythingLLM](https://anythingllm.com/)等工具可以用
  + 實測長篇翻譯還是用這個synchronous方式比較好
2. **[asynchronous-openai-doc-trnaslator]()**
  + 翻譯只比較適合做短篇，長篇擺明了不給做
  + 優點是不必時時連網，網路不太穩的情況下也能用；電腦/程式也不必一直開著
  + 假如真的要做長文翻譯也還是可以，但要走比較繁複 or 部份人工的分段作法。像是第一批同時發每一本的第一段 → 完成後各把第一段接到第二段開頭，再發第二批……，以此類推，每段翻譯都帶著前一段的摘要或末句；重視名詞一致性的話，也可以額外附對照表給model。但這種作法還是可能讓品質降低。