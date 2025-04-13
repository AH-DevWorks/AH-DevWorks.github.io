---
Title: 轉職路：AI養成班 Week 1.
Date: 2025-04-13
Tags: ["learning", "journal"]
Description: "當周學習與生活雜記"
---

#### 第一週：4/7、4/8、4/10、4/12
![Image by wal_172619 from Pixabay](/img/notes/blossoms-6991112_640.jpg)

---
+ 自我介紹環節，有不少資訊本科系或做過科技業，甚至還有自費生，真心佩服。
+ 簡單翻了一下歷屆學長姐成果，從電腦視覺到混合應用的都有。該來想想專題的主題了（小組專題＆個人的Side Project）。希望結訓後履歷至少能塞滿一整頁。
---
- 使用者家目錄
    ```
    C:\Users\[USER-NAME]\
    // 動態（隨登入者變化）
    // 登入者(USER)對家目錄有最高權限
    ```
- Adding Python Path to Environment Variables
- Zero Trust
- [pypi.org](https://pypi.org/)
- encode/decode
  - history: ASCII -> BIG-5 -> Unicode(UTF-8/UTF-16...)
  - character set（字元集）
- f-string: {}(Curly Brackets)內必須為`expression` (表達式)
  - statement(敘述句,如`print()`)等無回傳值的不應放入(會None)
- str.format()
  - ("[pattern]".format(a,b,c...))
    ```py
    print("他是 {}，今年 {} 歲。".format("Johnny", 88))
    # 輸出：他是 Johnny，今年 88 歲。
    ```
- float非精準數，比對時以區間比對較佳
    ```py
    a = 100.0
    b = 11.11 / 0.1111
    print(f"a 等於 b ? --> {a == b}")   # a 等於 b ? --> False
    print(f"a: {a} | b: {b}")   # a: 100.0 | b: 99.99999999999999
    ```
- eval()
    ```py
    x = eval(input("Input:"))  # eval()自動評估數字型別＆轉換
    # 不適用非數字
    ```
- match-case
    ```py
    rating = 5
    match rating:
        case 1:
            print("第1名，獎金10,000元！")
        case 2:
            print("第2名，獎金5,000元")
        case 3 | 4 | 5 as number: # 後置宣告 -- as
            print(f"佳作(第{number}名)，獎金500元")
        case _:
            print("下次再接再厲！")
    
    # Tuple
    niceSeasons = ("春", "秋")
    match niceSeasons:
        # 元素順序要一致，否則比對失敗:
        case ("秋", "春"):
            print("溫和季節")
        case _:
            print("季節比對失敗")
    ```

---
#### 自補充
##### f-string 細節
+ 若數字num是正整數如 `7` ，`f"{num:02d}"` 和 `f"{num:0>2}"` 結果看起來會一樣
    ```py
    num = 7
    print(f"{num:02d}")  # 07
    print(f"{num:0>2}")  # 07
    ```
+ 但兩種 format 背後邏輯不一樣：
    1. `num:02d` --> 數值格式化 : 「將變數 num 格式化成一個 `最小寬度為 2 位數` 的`十進位整數(Decimal Integer)`，`如果不足 2 位，則在前面補 0 `。
    2. `num:0>2` --> 字串對齊格式化 : 「把 num 轉成 string 後，`向右對齊(>)`，若num轉成string後，長度沒達到設定的`『最小寬度』 2` ，則在左側補 `0` 直到寬度達標為止」。
+ 因此如果碰到負數等特殊情況，第二種方式可能會出現錯誤的顯示，如：
    ```py
    i = -42
    print(f"{i:05d}")  # ✅-0042
    print(f"{i:0>5}")  # ❗00-42
    ```
+ p.s. `num:0>2` 這種方式，若num本身長度就大於設定的「最小寬度(2)」，則Python 會自動擴展欄位，完整顯示數值，不受限。

##### Disbale path length limit after installed
  - 假如裝 python 時沒勾選。安裝完畢後：
    - `modify the registry key`: Windows Key + R, type “regedit”
    - "Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem\LongPathsEnabled" -->
  set its value to `1`

---

#### Web Crawling
- tags
    ```html
    <label for="name">Name:</label>
    <input type="text" id="name" name="name">
    ```
  + for="xxx" <--> id="xxx"  ∴點擊上面`Name:`的字，連動到`input框`閃爍可輸入，不必一定要點擊input框
  + `type="hidden"` : 僅給開發者使用/確認, or 加上時間戳記,檢核、防爬蟲等
  + 網頁上有`name=""`標籤的通常會被form送出

+ `動態網頁`
- cookie
+ Headers
  + Status Code
+ Rest API
+ payload
+ pyenv -- Python version control
+ ETL
+ `Poetry`


---
---

#### 其他自學
+ [Hello 演算法](https://www.hello-algo.com/zh-hant/)
  + ~ 4.2
  + 從位址計算公式的角度看，索引本質上是`記憶體位址的偏移量`。首個元素的位址偏移量是0 ，因此陣列首個元素的索引為0是合理的
  + 由於陣列的長度是固定的，因此插入一個元素必定會導致陣列尾部元素“丟失”
  + 刪除元素完成後，原先末尾的元素變得“無意義”，所以無須特意去修改
    ```py
    import numpy as np

    # 建立一固定長度的 numpy 陣列
    arr = np.array([10, 20, 30, 40, 50])
    print("原始陣列：", arr)

    # 插入一個元素到index[2]，並移動後面的元素
    # 手動模擬插入（ numpy 不像 list 可以直接 insert）
    arr[3:] = arr[2:-1]  # 往後推一格
    arr[2] = 99  # 插入新值
    print("插入 99 後：", arr)  # 原本的 50 被擠掉了

    # 刪除索引 1 的元素（把後面的往前搬）
    arr[1:-1] = arr[2:]
    print("刪除索引 1 後：", arr)

    # 最後一格雖然還有值，但已經是「無意義」的資料
    print("最尾端元素實際上已經沒用了：", arr[-1])
    ```
+ 自架 `GitHub + Hugo 靜態網站`(https://ah-devworks.github.io//tags/hugo/)：
  + UI(font, i18n/[language] yaml, favicon)、footer調整
  + 技術彙整大致完成，剩餘額外補充待整理
  + 網站尚有改善空間（Google Analytics、SEO...），但反正能用了，先暫時這樣堪用就好
+ `實體書`
  + 岡野原大輔 (2024). _精確掌握 AI 大趨勢！深度學習技術解密：日本 AI 神人，帶你正確學會從機器學習到生成式 AI 的核心基礎_ (王心薇 & 施威銘研究室, Trans.). 旗標.
    + 讀到第二章
    + 順便根據其中2.1節額外找 [最小平方法(Least Squares)教學](https://www.junyiacademy.org/course-compare/math-high/math-high-publisher/s-ma-sanmin/s-ma-sanmin-10/s-ma-s-mg10-b2-3/v/umu0FcmBivg)，雖然現在機器學習應該都不是用OLS了
  + Ceder, N. (2019). _Python 技術者們：練功！老手帶路教你精通正宗 Python 程式_ (張耀鴻, Trans.). 旗標.
    + 中文是根據3rd Edition翻譯的，查了一下原作者Naomi Ceder今年有4th Edition
    + 以英文4th edition為主、中文3rd Edtion為輔，讀到2.3.2
    + 目前比較明顯差異是不再要求裝Anaconda之類的IDE，改用Google Colab hosted Jupyter Notebook放範例
      + Jupyter筆記本由兩種類型的單元格組成：文本單元格和代碼單元格

+ `Udemy線上課程`
  + smtplib
  + 網路託管：[PythonAnywhere](https://www.pythonanywhere.com/)
    + smtplib的涉及個人信箱帳密，還是別放到網上比較好
    + 其他一些無關且須重複執行的project倒是可以託管
  + 進度嚴重落後，這周弄靜態網站花太多時間了🫠

---
