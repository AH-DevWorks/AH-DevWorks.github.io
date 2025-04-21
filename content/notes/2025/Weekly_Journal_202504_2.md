---
Title: 轉職路：AI養成班 Week 2.
Date: 2025-04-20
Tags: ["learning", "journal"]
Description: "當周學習與生活雜記"
draft: false
---

#### 第二週：4/14、4/15、4/17、4/19
![Image by Ravi Kant from Pixabay](/img/notes/pexels-ravikant-1715161.jpg)
> It is not things themselves that disturb us, but our opinions about them.  — Epictetus

---

+ **相同功能/達成結果相同情況下，Recursive (call stack) 比for loop更占記憶體空間（∵Heap）**
	+ `Recursive` 每一次呼叫，程式會分配一塊stack frame(堆疊空間)給該函數，並保留目前狀態，等待下一層遞迴回傳後再繼續執行
		+ 導致每層呼叫都佔據一些記憶體
	+ `for loop` 每個只要一次函數呼叫的堆疊空間，不會有額外的函數呼叫堆疊
		+ 除非每圈都創造很多objects，否則使用空間是固定的
	+ python有堆疊上限: 預設1000層(可用 `sys.getrecursionlimit()` 查看)
		```py
		import sys
		print(sys.getrecursionlimit())
		```
		+ 超過就會 `RecursionError: maximum recursion depth exceeded in comparison` → 無法回傳或Crash或記憶體爆炸

+ *函式也可指派給變數* --> 類似代數中「變數可以代表一個值」的觀念
	+ 函式也是物件（objects）
	```py
	def square(x):
    return x * x

	def cube(x):
	    return x * x * x

	def operate(func, value):
	    return func(value)

	print(operate(square, 3))  # 9
	print(operate(cube, 3))    # 27
	```
	+ **Higher-order Functions(高階函式)**: 「接受其他function作為參數」或「回傳function作為結果」的Function
		+ python內建如map(func, iterable)、filter(func, iterable)、sorted(iterable, key=func)

+ **lambda**
  + Anonymous Function （匿名函式）的一種
  + 結構：`lambda parameters : expression`


---
#### Web Crawling

##### BeautifulSoup
+ *select() vs. find()*
  + select(): CSS 選擇器風格查詢，可單區塊深入查內層
  	+ 可用 CSS selector 語法（ex. `div > span.title`）
  + find(): 條件搜尋式，用 tag 名、class、id 當參數，查詢廣範圍但難深入
  ```py
  # 用 find() 抓到區塊，再用 select() 查裡面的內容
  box = soup.find("div", class_="container")
  title = box.select_one("span.title")  # select_one 會直接回傳單一元素
  ```

##### package(browser-cookie3)
> 針對需要登入才能瀏覽的網站
> 
> 自動fetch瀏覽器cookies
```py
import browser_cookie3, requests

# 從 Chrome 瀏覽器中取得 cookies
cj = browser_cookie3.chrome()  # 回傳一個 cookie jar

# 帶著 cookies 發送 GET 請求
res = requests.get('https://example.com', cookies=cj)
print(res.text)
```
+ 登入了一個網站（例如 Twitter、Google、Facebook 等）後，browser-cookie3可借用(存在本機的)瀏覽器登入的 cookies (session token)去存取相同網站的資料
+ 適用於爬某些私人頁面（如個人FB 、學校系統等），但要小心

##### dynamic website
+ e.g. nownews -- 找點擊「載入更多」後的request url
+ 抓多頁==> 抓網址id規律
	+ 規律不明顯的話就抓大概，搭配去重(都存進set；或用dict，把每篇url最後id當成key也可以)

##### Pandas 儲存格式
+ `parquet format`: 列式儲存格式，I/O效率高、壓縮比高、儲存空間小；通常搭配 pyarrow 或 fastparquet
+ python pickle format: python自己的物件序列化格式；跨語言、跨平台、跨版本python不一定相容；適合快速暫存；*安全風險*--不明來源的pickle可能被植入惡意代碼

---
#### 自補充
##### Separate data (or settings/configuration) from logic 把邏輯和資料分開
+ **「關注點分離」（Separation of Concerns）**
  + 把 Logic(處理流程、資料運算、檔案操作等核心功能) 跟 Settings(target 名稱、輸出位置、參數開關等`使用者可修改`的項目) 分開
  + 良好習慣
+ 不要把一些`可變的`參數（像使用者指定的輸出檔案檔名、ID、URL等）硬寫在程式碼裡面，而是要集中放在一個設定檔（config file）裡管理
  + ❌程式碼中某一行直接寫：
    ```py
    target = "123"  # 請自行更改檔案名稱
    filename = f"{target}.txt"
    ```
  + ✅使用設定檔如 `config.json`，再讓程式讀檔：
    ```json
    {
      "target": "123"
    }
    ```
    ```py
    import json

    with open("config.json", "r", encoding="utf-8") as f:
        config = json.load(f)
    target = config["target"]
    filename = f"{target}.txt"
    ```

##### Prevention事前預防 vs. Recovery事後處理
+ e.g.`讓使用者輸入數字 ==> 避免輸入非數字`
+ `Prevention`: 用regex檢查
	+ 避免 try-except 的開銷（雖然通常微乎其微）
	+ 適合「錯誤不是預期中流程的一部分」的場景（例如輸入驗證）
	+ 缺點：要自己處理各種邊界條件（空字串、負號、小數點、全形字元……）
```py
import re

user_input = input("請輸入數字: ")
if re.fullmatch(r'\d+', user_input):  # re.fullmatch-->整串輸入都要符合規則
    number = int(user_input)
else:
    print("Error: 請輸入合法的整數")
```
+ `Recovery`: try-except
	+ 更符合 **「EAFP」（Easier to Ask Forgiveness than Permission）原則** —— 一種 Python 的設計哲學
	+ 相對於EAFP <---> **LBYL(Look Before You Leap)**
	+ 精簡、直觀：直接嘗試做「你希望做的事」，失敗再處理
	+ 缺點：對於Expected Errors(預期性錯誤，非常常見、幾乎一定會發生)來說，不夠精準（會包住太多可能錯誤）
+ Prevention: 使用者輸入 / 格式驗證；邏輯精密 / 錯誤類型需精細掌握者
+ Recovery: 無法預測的錯誤（網路錯誤、檔案損壞）；簡單任務 / 不在乎精細錯誤類型


---
---

#### 其他自學
+ **[Hello 演算法](https://www.hello-algo.com/zh-hant/)**

+ **`實體書`**
  + 岡野原大輔 (2024). _精確掌握 AI 大趨勢！深度學習技術解密：日本 AI 神人，帶你正確學會從機器學習到生成式 AI 的核心基礎_ (王心薇 & 施威銘研究室, Trans.). 旗標.
  + Ceder, N. (2019). _Python 技術者們：練功！老手帶路教你精通正宗 Python 程式_ (張耀鴻, Trans.). 旗標.

+ **`Udemy線上課程`**
  + `API(Application Programing Interface, 應用程式界面)`
    + API Endpoint: location, URL
    + Request
      + 【library】: `requests`
        + .raise_for_status(): (當如果不是200通過) raise一個requests.exceptions.HTTPError，讓使用者集中處理API請求時的HTTP errors
    + e.g. [International Space Station Current Location](http://open-notify.org/Open-Notify-API/ISS-Location-Now/)
    + http status code:
      + `1xx`: hold on
      + `2xx`: here you go
      + `3xx`: go away
      + `4xx`: you screwed up
      + `5xx`: the site screwed up
  +  [HTML Entities -- HTML特殊字元編碼表/字符實體](https://www.w3schools.com/html/html_entities.asp)
     +  避免跟HTML特殊字元混淆，如 `<` 須改用HTML Entity的寫法：`&lt;`
        ```py
        import html
        unescaped_words = html.unescape(escaped_str)
        ```
  + tkinter
    + `self.false_button = Button(image=self.cross_img, highlightthickness=0, command=self.quiz.check_answer("False"))`:
    後半部不能`command=self.quiz.check_answer("False")`，command要的是method名稱，要到按鈕點擊時才會執行function，原本寫法會等於是「馬上執行 check_answer("False")」
    + --> 可用lambda寫法或是另外定義一個def func引入

+ **[李宏毅【生成式AI時代下的機器學習(2025)】](https://www.youtube.com/playlist?list=PLJV_el3uVTsNZEFAdQsDeOdzAaHTca2Gi)**
+ **[李宏毅【生成式AI導論 2024】](https://www.youtube.com/playlist?list=PLJV_el3uVTsPz6CTopeRp2L2t4aL_KgiI)**
+ **[張成龍【MySQL資料庫從零開始玩轉】](https://www.youtube.com/watch?v=Krrek8onTc8)**

---
