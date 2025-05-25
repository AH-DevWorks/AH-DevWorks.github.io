---
Title: 【ChatGPT】API搭配不同模型之輸出成效評比——2025/05
Date: 2025-05-17
Tags: ["blog","API","OpenAI","model"]
Description: "從「翻譯長篇文章」的角度，簡單比較OpenAI不同模型的成效"
draft: true
---

> 【[前情提要]()】
>
> 根據上次寫的「openai-doc-translator」程式，簡單評比幾個合適/知名模型(model)針對同一任務的翻譯成果。
> 
>> 翻譯原文文本皆為《[THE Velveteen Rabbit OR HOW TOYS BECOME REAL - by Margery Williams](https://www.gutenberg.org/ebooks/11757)》
>
>> 除了model參數以外，其餘參數皆相同
> 
>> 皆"不"使用模型「store=true」自動接續對話功能。


### 1. "gpt-4o-mini-2024-07-18"
+ 漏了開頭「書名、作者名稱」（THE Velveteen Rabbit OR HOW TOYS BECOME REAL by Margery Williams）沒翻譯到
+ 中段某次回應突然出現「抱歉，我無法協助滿足該要求。」<---查了一下這段文字好像是「有違反OpenAI模型使用政策（例如詢問非法、暴力、情色等問題）時」會出現的回應，理應不該出現在正常的回應裡。
  + 不確定是不是「調用API」跟一般使用者以網頁/APP互動時，模型對於input的處理方式不同而導致。
  + 目前只在 4o-mini 碰到，其他模型未出現同樣問題。
+ 中後段原文「And then a strange thing happened. For where the tear...and out of it there stepped a fairy.」精靈現身的這一段沒翻譯到。
  + 特別去翻了翻 OpenAI API 後台 Dashboard - Logs/Response，同一次input還有其他段落都有翻譯到，唯獨漏了這一小段。
+ 【API收費】
+ 【Rate limits】
  + `200,000 TPM｜500 RPM；10,000 RPD｜2,000,000 TPD`
    + TPM：每分鐘token上限（tokens per minute）
    + RPM：每分鐘請求上限（requests per minute）
    + RPD：每天請求上限（requests per day）
    + TPD：每天token數（tokens per day）
+ *【任務速度】
+ *【文句通順程度】
+ *【評分】

> 【任務速度】以xxxxx計時
> 
> 【文句通順程度】＆【評分】皆為個人主觀


### 2. "gpt-4.1-2025-04-14" / "gpt-4.1"

+ 最大問題是Rate limits太低
+ 【Rate limits】
  + 30,000 TPM｜500 RPM｜900,000 TPD