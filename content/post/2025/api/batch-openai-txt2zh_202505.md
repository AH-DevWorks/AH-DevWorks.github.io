---
Title: 用 OpenAI GPT Batch API 大量翻譯小說 （非同步式）
Date: 2025-05-26
Tags: ["blog","API","OpenAI","model"]
Description: "Batch API工具，用 OpenAI 的AI模型(o4-mini)批次同時處理多筆 .txt 檔案翻譯"
draft: false
---

## batch-openai-txt2zh (非同步 .txt 檔批次翻譯器)
> 透過 OpenAI 模型的 Batch API ，大量批次把你的文件上傳給AI翻譯。
> 
> 雖然標題是寫翻譯「小說」，但其實任何文件——只要能轉成.txt文字檔——都能翻譯

+ 和前一篇[[**同步式翻譯**](https://ah-devworks.github.io/post/2025/api/synchronous-openai-doc-translator_202505/)]方法最大的不同在於「程式開起來跑一次start後，檔案就會上傳給AI處理，本機程式可以關掉，過一陣子再重啟確認翻譯完成了沒」，程式/網路/電腦都不用一直開著

### 檔案連結↓
>[https://github.com/AH-DevWorks/batch-openai-txt2zh](https://github.com/AH-DevWorks/batch-openai-txt2zh)
+ 可以直接點進上面連結，按綠色的「< > Code」按鈕 → 「Download ZIP」→ 解壓縮後照著網頁上的說明或「readme.md」文件進行

---

### 使用教學
> 過程大部份內容都跟前一篇[[同步式](https://ah-devworks.github.io/post/2025/api/synchronous-openai-doc-translator_202505/)]的類似，只有啟動的指令稍微不同

#### **[檔案連結](https://github.com/AH-DevWorks/batch-openai-txt2zh)**
1. 點進上面連結，按綠色的「< > Code」按鈕 → 「Download ZIP」→ 解壓縮（包含「batch-openai-txt2zh」這個最外層資料夾）
   + 解壓縮完的資料夾裡面應該至少會有這些項目：
```bash
batch-openai-txt2zh/
├── main.py                # 程式進入點
├── batch_handler.py       # 建立＆確認batch；任務進度確認
├── check_limits.py        # 估算tokens總數 
├── parse_output.py        # 解析 .jsonl ，將翻譯完畢之內容儲存為 .txt
├── .env                   # API金鑰所在位置
├── input_files/           # ***【請把你要翻譯的 .txt 檔案放入此資料夾】***
├── output_files/
│   └── translated_txts/   # 翻譯完畢的檔案位於此處
├── requirements.txt       # 依賴套件列表
├── batch_files/           # 存放 .jsonl 酬載(payloads)
├── batch_state/           # 留存近期batch_id
├── readme.md              # 【使用說明】
├── finished_files/        # 存放已完成任務之原始文檔
└── sample_docs/           # 範例文件
```

2. **安裝必要套件**
   + 電腦內請先安裝 python ，可到[Python官方網站](https://www.python.org/downloads/)下載
     + 建議使用 python 3.10 以上的版本。照著程式安裝的步驟，全點下一步就能裝好
   + 安裝本專案所需的 Python 套件
     + [Windows] 在前一步驟解壓縮好的資料夾裡面（`main.py所在的資料夾裡`）按住鍵盤 Shift 鍵 + 點滑鼠右鍵 → 選擇「在這裡開啟 PowerShell 視窗」或「在終端中開啟 / 在此處開啟命令提示字元」
     + [Mac] 開啟 Finder，進入`main.py所在的資料夾` → 右鍵 → 點選「服務」→「在終端機中開啟」
     + 輸入或複製貼上以下指令：
      ```bash
      pip install -r requirements.txt
      ```
    + 視窗會跑一堆像是下載＆安裝的資訊出來，放著等全都裝好，視窗沒再動之後就可以關掉

2. **設定 API 金鑰**
   + 用記事本等軟體開啟 .env.example 檔案，修改其中 API_KEY 欄位，填入你自己的 OpenAI API Key(金鑰)，並且將檔案重新命名為「.env」
   + **OpenAI API 金鑰申請**: 進入 [OpenAI Platform](https://platform.openai.com/)，註冊/登入，資料填一填就能 Generate API Key
     + 通常需要先跟OpenAI買一些credits才能用，我自己是先儲值5美金，再把"Billing - Auto recharge"這個功能打開，credit不夠的時候會自動加值，翻譯就不會突然中斷
     + 關於如何取得API Key的過程不贅述，可自行搜尋或參考[How To Get Your OpenAI / ChatGPT API Key (2025)](https://www.youtube.com/watch?v=SzPE_AE0eEo)等教學

3. 回到步驟1解壓縮完的資料夾，找到 [input_files] 資料夾，把要翻譯的檔案全放進資料夾裡
   + 支援格式：純文字（.txt）

4. 執行主程式：在`main.py所在的資料夾`內點選滑鼠右鍵 —> 選擇「在終端中開啟」 —> 輸入 `python main.py start` -> 順利的話就會看到視窗跑出「`[✅] 上傳完成。檔案 ID:xxxxx`」的訊息，表示已經上傳給AI開始翻譯了
   + 這時就可以放心把這個執行中的視窗關閉，或是要斷網、關機都可以，檔案是在 OpenAI 那邊處理的
   + 耗費的時間是依照原文字數＆文件數量的多寡而定
   + 過程中可以隨時打開程式來確認進度：在`main.py所在的資料夾`內點選滑鼠右鍵 —> 選擇「在終端中開啟」 —> 輸入 `python main.py status`

5. 確認進度＆下載完成檔：在`main.py所在的資料夾`內點選滑鼠右鍵 —> 選擇「在終端中開啟」 —> 輸入 `python main.py status`
   + 如果都翻譯完成，應該會跳出一堆勾勾（✅）符號的訊息，最後一則是「`[✅] 翻譯檔案整理完畢：...`」，表示成功
   + 點進 [output_files]-->[translated_txts] 資料夾，裡面就是翻譯好的各檔案；原始原文txt檔則會挪到 [finished_files] 資料夾裡

---

#### 額外補充 --- 操作指令集

| 動作           | 指令                           | 說明                     |
| ------------ | ---------------------------- | ---------------------- |
| 預估 Token 使用量 | `python main.py check-limit` | 送出前先檢查是否超量             |
| 開始翻譯         | `python main.py start`       | 上傳 JSONL → 建立 Batch 任務 |
| 查詢進度/下載結果    | `python main.py status`      | 進度查詢；若完成，則自動下載檔案   |
| 取消目前批次       | `python main.py cancel`      | 向 OpenAI API 發送取消請求    |
> 
> 
---
---

> 簡單說明如上，假如有任何疑問，歡迎[發E-mail給我](mailto:a.h.devworks@gmail.com>) 或到上面檔案連結裡的 github add new issue
>

---

##### 參考資源
+ [OpenAI Batch API Documentation](https://platform.openai.com/docs/guides/batch)

---
##### <i>p.s. 其實用n8n或其他AI Agent八成更快，但手刻的成就感總是高一點</i>