---
Title: 轉職路：AI養成班 Week 4.
Date: 2025-05-12
Tags: ["learning", "journal"]
Description: "當周學習與生活雜記"
draft: true
---

#### 第四週：5/3～5/7、5/9、5/10
![Image by Ravi Kant from Pixabay](/img/notes/pexels-ravikant-1715161.jpg)
> The soul becomes dyed with the color of its thoughts.  — Marcus Aurelius

---

+ **MongoDB NoSQL**
  + Lazy Loading
    + mongodb 搭配 python 或 javascript 的 driver 時可能出現
        ```py
        cursor = collection.find({"status": "active"})  # --> 此時並未真正執行查詢
        for doc in cursor:
            print(doc)  # --> 直到這裡才真正觸發查詢、逐筆取出data
        ```
    + find() 回傳一個 iterator
    + 當開始遍歷 cursor 時，MongoDB 才真正從DB取出資料
    + Data是一批一批從 server 傳回（依據 batch size）
  + Update
  + Write Concern
    + 控制「寫入操作需要多少確認」的機制，確保data寫入時的可靠性＆一致性
    + 決定 MongoDB 執行寫入（insert, update, delete...）後，要等多少個節點有收到＆記錄這筆資料，才算是寫入成功
    + w0, w1, w2.... ; w: "majority"
  + Sharding 水平擴展
    + 將資料分散儲存到多個伺服器（shard）
    + shard key 影響 data 如何分佈、query效率、I/O效能
      + 選Cardinality高的
      + 避免 hot shard 特定幾個 shard 高負載 （e.g. `{ gender: "male" / "female" }` --> 只有兩種值，會集中在兩個 shard 上）

+ **AI Basic**
  + Edge Computing
  + Processing Unit
    + CPU（通用運算）, GPU（大量平行運算）, TPU（針對ML加速tensor運算）
  + AI
    + Machine Learning
      + Supervised Learning
        + Classification, Regression
      + Unsupervised Learning
        +  Clustering
      + Reinforcement Learning
    + Robotics
  + Microsoft Power Bi
  + Reinforcement Learning --- 投資組合問題（Single-armed Bandit 吃角子老虎）

+ **Data Mining**
  + Data ---[Selection]---> Target Data ---[Preprocessing]---> Preprocessed Data ---[Transformation]---> Transformed Data ---[Data Mining]---> Pattern / Models ---[Interpretation Evaluation]---> Knowledge
  + Pivot Table

+ **Python Flask**
  + Decorator, host, port
  + static
  + HTML + Jinja2(模板語言): `讓HyperText Markup Language(hutml) 在伺服器端處理流程控制`

---

#### Side Project
+ 主題：[AI相關]（名稱與具體內容暫不揭露）
+ 進度：[███░░░░░░░░░░░░░░░░░] 15%
  + [✔] 初步規劃與構想
  + [✔] 選定資料來源
  + [✔] 完成並執行腳本（資料抓取/預處理）
    + 副產物：[synchronous-openai-doc-translator]()、[asynchronous-openai-doc-trnaslator]()
  + [✔] 進一步 Data Cleaning
  + [◎] 進行中：標註／格式／語言轉換
  + [　] ...

---