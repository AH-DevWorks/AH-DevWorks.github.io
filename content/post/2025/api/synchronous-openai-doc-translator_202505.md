---
Title: 用 OpenAI GPT API 翻譯小說 （同步式）
Date: 2025-05-20
Tags: ["blog","API","OpenAI","model"]
Description: "簡單的API工具，用 OpenAI 的AI模型(gpt-4.1)來把 .txt 檔案翻譯成繁體中文"
lastmod: 2025-05-25
featured: true
draft: false
---

![Image by AH-DevWorks](/img/post/Sample_20250522.jpg)
> Comparison - Before-and-After Text
 
---
 
## OpenAi Doc Translator (OpenAI 文件翻譯器)
> 透過 OpenAI 模型 API ，快速翻譯你自己的文件。
> 雖然標題是寫翻譯「小說」，但其實任何文件——只要能轉成.txt文字檔——都能翻譯

## >> [檔案連結](https://github.com/AH-DevWorks/synchronous-openai-doc-translator)
+ 可以直接點進上面連結，按綠色的「<> Code」按鈕 → 「Download ZIP」→ 解壓縮後照著網頁上的說明或「readme.md」文件進行

---
 
+ 這是個人Side Project的副產物，核心是呼叫OpenAI的API，搭配簡單的前處理（原文段落自動拆分）＆循環串接，達到「翻譯文本連貫性」的效果
+ 內容儘量寫得簡單，不懂程式的人大概也能照著操作，拿自己的OpenAI金鑰來翻譯文件

---

### 特色
+ 從原文前處理到譯文串接都是在本機程式上進行，只有「翻譯」階段的文本才會連網傳到 OpenAI Model上面進行AI翻譯
+ 最大好處就是只會經過 OpenAI ，不會再有其他第三方服務拿到你的資料或權限
+ 用你自己的API，除了你自己以外，也不會有其他人知道你翻譯了些什麼
  +  P.S. OpenAI API 服務的帳號後台 [https://platform.openai.com/](https://platform.openai.com/) 上面也提供使用者手動確認歷史紀錄、Credits用量、自動Credits儲值等功能
+ 當然，操作上肯定沒有其他像是「沉浸式翻譯」之類的網路外掛套件來得方便簡單，功能也沒那麼豐富
  + 懂程式的人也可以拿這個程式當基底，自己擴展像是讀取圖片、單純聊天對話、摘要PDF等功能，反正只要OpenAI有給API的都能擴展

---

### OpenAi Doc Translator 使用教學
> 基本就只是照搬readme.md內容，多加上幾張示意圖

#### [檔案連結](https://github.com/AH-DevWorks/synchronous-openai-doc-translator)
1. 點進上面連結，按綠色的「<> Code」按鈕 → 「Download ZIP」→ 解壓縮（包含「synchronous-openai-doc-translator-main」這個最外層資料夾）
   + 解壓縮完的資料夾裡面應該至少會有這些項目：
```bash
synchronous-openai-doc-translator-main/
├── main.py                # 程式進入點
├── files_handler.py       # 處理檔案（讀取／儲存）
├── text_segmenter.py      # 拆分原文txt檔成為段落
├── api_translator.py      # 呼叫OpenAI API處理翻譯
├── .env                   # API金鑰所在位置
├── input_files/           # *** 把要翻譯的 .txt 檔放進這裡 ***
├── output_files/          # 翻譯完畢的檔案會出現在這個資料夾
├── requirements.txt       # 依賴套件列表
├── README.md              # 【使用說明】
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

4. 執行主程式：在`main.py所在的資料夾`內點選滑鼠右鍵 —> 選擇「在終端中開啟」 —> 輸入 `python main.py` -> 順利的話就會看到視窗開始跑進度條，表示已經開始翻譯了
   + 記得「不能」把這個執行中的視窗關閉（可以縮小或放到畫面旁邊），也儘量不要斷網
   + 耗費的時間是依照原文字數＆文件數量的多寡而定，可參考底下我的簡單估計

5. 視窗顯示都翻譯完畢後，可以點進 [output_files] 資料夾，裡面以「[Translated]...」開頭的就是翻譯好的檔案了

---

#### 額外補充
1. 根據個人經驗，翻譯一本原文（英文）字數 95,000 左右的小說，大約需要 40～60 分鐘
2. 但實際 AI 在「讀」文字的時候，看的不是「原文有幾個字」，而是會把字句拆成 `Token` (符記)：一個英文單詞如果是由比較多個字母組成，就可能被拆成2～3個token；中文字則通常是一個字一個token。如下面示意的：
<img src="/img/post/Tokenizer_20250525.png" width="1000"/>

   + [Tokenizer | OpenAI Platform](https://platform.openai.com/tokenizer): 把原文複製貼上，就能推估有多少tokens

3. OpenAI API是以token來計價，以這個程式呼叫的「gpt-4.1-mini」來說：
   + Input Tokens(輸入的tokens，也就是每次翻譯的原文＆程式告訴AI要做翻譯的中文指令)：每一百萬個tokens=美金$0.40元
   + Output Tokens(輸出的tokens，也就是AI翻譯好、回傳給我們的中文內容)：每一百萬個tokens=美金$1.60元

4. 從上面就可以看出，「用程式來跟AI模型交流」的方式，如果不是用量很兇，算起來會比直接登入ChatGPT、訂閱Plus等方案來得划算
5. 但有一點要注意：這個 OpenAi Doc Translator 程式原始目標是「小說的翻譯」，「小說」非常重視邏輯、語氣、人名的前後一致，所以程式裡用了一種「能讓翻譯連貫」的方法，而這個方法會讓 Input Tokens 提昇
   + 這裡不講得太複雜，只說結論：如果你要翻譯的是「超長篇鉅作（比如魔戒三部曲）」，建議不要一次就把整本將近500頁原文、快要20萬字的小說放進 [input_files] 資料夾裡
   + ==> 但你可以把每一本再拆成2～3個txt檔（像是按照章節去拆，每一份txt檔案裡面包含3～5章之類的）。例如原本魔戒有3大本txt檔，拆好之後變成12個分章的txt檔案，這時後再一起放進 [input_files] ，總tokens數量在AI看來就會比較少，簡單來說就是比較省錢
   + 要是擔心這樣拆分，人名/地名會不連貫的話，可以預先在每一份小txt檔內文的最前面，自己手動加上「對照表」、告訴AI：
    ```bash
    ☆【人名/地名對照表，請按照對照表來翻譯】☆
    Frodo Baggins：佛羅多·巴金斯
    Gandalf：甘道夫
    Aragorn：亞拉岡
    Legolas：勒苟拉斯
    Gollum：咕嚕
    ☆【以上是人名/地名對照表】☆
    The Fellowship of the Ring
    J.R.R. Tolkien
    Chapter I: A Long-expected Party
    When Mr. Bilbo Baggins of Bag End announced that he would shortly be celebrating his eleventy-first birthday with a party of special magnificence, there was much talk and excitement in Hobbiton.
    Bilbo was very rich and very...
    ```

--- 

> 簡單說明如上，假如有任何疑問，歡迎[發E-mail給我](mailto:a.h.devworks@gmail.com>) 或到上面檔案連結裡的 github add new issue

---

##### 參考資源
+ [OpenAI Responses API Documentation](https://platform.openai.com/docs/api-reference/responses)
+ [Models](https://platform.openai.com/docs/models)
+ [Tokenizer](https://platform.openai.com/tokenizer)