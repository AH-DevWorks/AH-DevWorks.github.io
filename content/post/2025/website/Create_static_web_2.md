---
title: 自架網站筆記：使用Github + Hugo（中）
date: 2025-04-07
tags: ["hugo","blog","website"]
Description  : "——自架靜態網站的過程紀錄．中篇——"
featured: true
---

#### [【前一回】自架網站筆記：使用Github + Hugo（上）](https://ah-devworks.github.io/notes/website/create_static_web_1/)

---

### PART II : 設定內容倉庫（Contents-Repo）
1. 本地另一個資料夾clone「放網站內容的repo（PART I步驟3建立的Contents-Repo）」
2. 同樣初始化
```bash
touch README.md
echo "# [Contents-Repo]" > README.md
git add README.md
git commit -m "Initial commit"
git push origin main
```
3. 建立內容結構/資料夾
   + 這裡可按照自己需要的層級或theme建議的層級來做
```bash
mkdir content
cd content
mkdir notes post
```
---

{{< alert_box role="danger" title="注意" content="結構資料夾要包含Hugo在building網站時需要的各子資料夾" >}}

---

> 通常是「contents」、「static」這兩個資料夾，但也不一定，建議翻查主題的Documents，有寫最好，沒寫的話就只能自己多hugo server測試幾次。


4. 建立一篇測試文章，如 `echo -e "+++\ntitle = \"First Note\"\ndate = 2025-04-05\n+++\n\nThis is my first note." > content/notes/first-note.md` 或自己寫一個.md檔
5. push上去
+ 注意這裡<b><u>不要</b></u>執行 `hugo new site`，Contents-Repo只負責網站上的內容檔案


<s>
6. <mark>將 Contents-Repo 引入到 Website-Repo</mark>
   + cd回到 PART I 建立好的 Website-Repo 資料夾
7. **加入 Contents-Repo 作為子模組，將 Contents-Repo 的內容直接映射到 content/ 跟 static/ 資料夾**
git submodule add -b main https://github.com/[user_name]/[Contents-Repo].git Contents-Repo
，然後再分別把其下兩個資料夾連結到對應的「content/」「static/」:
ln -s Contents-Repo/content content
ln -s Contents-Repo/static static
+ 注意：這裡可能會出現像 `fatal: 'content' already exists and is not a valid git repo` 等錯誤-->直接把原本資料夾裡有的「content」砍掉（或是裡面資料先備份到其他地方、之後再放回去）
8. 抓取submodule內容
```bash
git submodule update --init --recursive
```
</s>

---

+ [2025/04/11 更新] 
  6. 把Content-Repo的檔案（`content/`資料夾裡的.md文件 ＆ 可能有`static/`裡的圖片等額外檔案）確認都ok後，在這個資料夾先git add -> commit -> push 一次後，手動複製 `content/ static/`兩資料夾到 Web-Repo/ 底下，覆蓋更新原本Web-Repo/裡面的同名資料夾跟檔案
     + 註: 原本submodule引入的方式太複雜，也容易出錯，反正兩個repo的檔案都在同一電腦本機，乾脆版本管理用各自的git，移動用手動的處理比較快。

7. 再次 `hugo server` 測試，正確的話原本在Contents-Repo裡面的文章（步驟4做的）應該會顯示在網站對應的地方（如網站 [Notes] 標籤底下）
8.  沒問題的話記得Web-repo 也要 push提交更改。
9.  之後主要寫日記、文章、圖片都在Content-Repo進行，完成後git版本更新，再複製進Web-Repo/ 後 hugo，Web-Repo/也記得每次都要版本更新。這樣萬一哪次檔案不小心誤刪或蓋掉，還是可以回溯。

---
---

### PART III : 部署網站到 GitHub Pages
1. 生成靜態檔案：在Website-Repo用 `hugo`指令，在其下的 public/ 資料夾生成靜態網站檔案
2. 把public/資料夾裡已生成的靜態網站全部檔案挪到前一層（手動把檔案複製貼上也可以）
```bash
cp -r public/* .
```
3. 提交 
```bash
git add .
git commit -m "Generate static site"
git push origin main
```
4. 檢查網站是否正常顯示（github倉庫-Settings-Pages會顯示「Your site live at `https://...`」）

---

+ 做到這，網站其實就算是架好了，其他細節跟QA放在下篇。

---

#### [自架網站筆記：使用Github + Hugo（下）](https://ah-devworks.github.io/notes/website/create_static_web_3/)