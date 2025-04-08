---
title: 自架網站筆記：使用Github + Hugo（上）
date: 2025-04-05 
tags: ["hugo","blog","website"]
Description  : "——自架靜態網站的過程紀錄．上篇——"
featured: true
---

> + 第一篇文章，總之先簡單記錄這個靜態網站怎麼做的，不然大概隔幾天又忘光光。
> + 原本只打算寫給自己複習用，所以有些我已經懂的部份（像是Git）寫比較簡單。如果有緣的你無意看到這篇，想看懂，但看不懂，歡迎[私信我](mailto:a.h.devworks@gmail.com>)。
> + 如果你/妳看到這篇文章的時候，網站已經有留言功能，也歡迎留言詢問。

### PART 0 : 在動手之前 - 關於這個靜態網站
預先構想這個網站會由兩大部份（2 repositories）構成：
1. Website-Repo：負責靜態網站呈現（用 Hugo build 出來的 HTML/CSS/JS）。
2. Contents-Repo：放原始內容（markdown，如 content/notes、content/diary）。
+ 未來更新內容主要會在 Contents-Repo，然後將生成的網站部署到 Website-Repo。
+ 也會將其他專案 repo（像 side projects）整合進這個 Hugo 網站。
+ 行有餘力再多加上其他功能（Google Analytics, SEO, comments）


### PART I : 基礎設定
1. 已經有github帳號
2. Windows `winget install Hugo.Hugo.Extended` 安裝Hugo（建議安裝 `Hugo Extended`版本，方便後續自訂模板樣式）
3. github新建兩個repo：一個放內容（Contents-Repo），一個負責網站本身(Website-Repo. -- ex.AH-DevWorks.github.io)，分別clone到本地的資料夾（這時兩個倉庫都還是空的，正常）

---

{{/*< alert_box role="warning" title="建議" content="上述操作需要有「git」以及「github」的基礎理解，如果不懂的話，請先關鍵字google或問AI。" >*/}}

---

4. 初始化git分支
```bash
touch README.md
echo "# [Website-Repo]" > README.md
git add README.md
git commit -m "Initial commit"
git push origin main
```
5. 本地clone放網站內容的repo，`hugo new site .`
   + 設在母資料夾較佳，建議別再額外新增如 `hugo new site [MySite etc.]`，否則後續要每次都手動挪動/public或額外設定自動腳本，會很麻煩。
   + 若母資料夾已有內容（如README.md），則用 `hugo new site . --force`
6. 等Hugo自動建立好內容後，cd到themes資料夾；挑個Hugo的theme，照著說明來裝（例如clone 到 themes資料夾，或用Git submodules處理）
{{/* < info_cards header="suggest" title="這裡以Lightbi為例" content="https://themes.gohugo.io/themes/lightbi-hugo/" > */}}
7. 把這個theme設成預設：
    + 回到步驟5建立好的母資料夾，`vi hugo.toml`
    + 編輯toml內容： A. [baseURL]改成步驟3負責網站本身的repo網址(通常是https://[user_name].github.io/)；B. [title]改隨意（會顯示往站上）；C. [theme]改成前幾步驟挑選的

---
{{/*< alert_box role="success" title="關於hugo.toml參數" content="參數與功能跟主題密切相關，建議先翻看主題給的Documents。" >*/}}
{{/* < info_cards header="suggest" title="以Lightbi為例" content="參數的說明就放在Demo網站的文章裡：[連結](https://lightbi-hugo-theme.netlify.app/en/post/2020/parameters/)" > */}}

---

8. 初步測試：`hugo server`，應該會有空白模板或theme預設範本網頁
   + 有些theme偏舊，這步會出現問題，直接換一個theme比較省事
   + 正確的話bash或cmd會顯示網站生成的頁數、靜態檔案數量等資訊
   + 找到其中Web Server is available at `http://localhost:1313/...`這行，貼到瀏覽器，有出現看起來像網頁的樣子就對
9.  確認ok後要再提交一次更改到github
```bash
git add .
git commit -m "Set up website with theme"
git push origin main
```
 
 ---

##### [【下一回】自架網站筆記：使用Github + Hugo（中）](https://ah-devworks.github.io/notes/website/create_static_web_2/)