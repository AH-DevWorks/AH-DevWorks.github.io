---
Title: Python 實作 Bubble Sort
Date: 2025-04-14
Tags: ["learning", "algorithm", "sorting", "bubble sort"]
Description: "氣泡排序法（泡沫排序法）練習"
---

## Python實作 Bubble Sort

> ※排序演算法(Sorting Algorithm)是將資料依特定規則進行重新排列的演算法，通常是依照資料的大小進行排列，Bubble Sort是其中之一。

> ※資料排序後，就能進一步對其進行更有效率的分析或搜尋。如「二分搜尋法」在資料數較多時，效率高於循序搜尋，但執行二分搜尋之前，必須先將原始資料排序。


### 程式碼
```py
def user_input_check(message):
    while True:
        try:
            user_input = input(message)
            if '.' not in user_input:
                return int(user_input)
            else:
                return float(user_input)
        except ValueError:
            print("請輸入正確的數字...")


def bubble_sort(number_list):
    for i in range(1, len(number_list)):
        print(f"### 第{i}輪 ###")
        for j in range(len(number_list) - i):
            if number_list[j] > number_list[j + 1]:
                number_list[j], number_list[j + 1] = number_list[j + 1], number_list[j]
        print(f">>資料狀態: {number_list}")
        print("-------------------")
    return number_list


print("———請輸入以下資訊———")
data_length = 0
while data_length <= 0:
    data_length = int(user_input_check("資料總數量：\n"))

data_list = []
for num in range(data_length):
    data_list.append(user_input_check(f"第{num + 1}筆資料: "))

print(f"\n\n原始資料:{data_list}")
print("-------------------")

sorted_list = bubble_sort(data_list.copy())

print(f"排序後資料: {sorted_list}")
```
### 執行畫面
{{< figure src="/img/post/Bubble-Sort-2025-04-14.png" width="400px" >}}


### 說明
1. 上面程式碼是自己練習寫的，可能不是最好的bubble sort寫法
2. `user_input_check(message)`: 輸入檢查，跟bubble sort無關，只是習慣放
3. `bubble_sort(number_list)`: 排序法本體，外層 for loop 控制排序的輪數，每輪最後會把一個最大值排到最右側；內層 for loop 進行相鄰元素比較與交換
   + 值得一提是中間 `number_list[j], number_list[j + 1] = number_list[j + 1], number_list[j]` ，這種寫法是Python特有的，涉及 tuple 型別的 packing / unpacking ，一行就能交換兩變數值
   + 如果是其他程式語言，要兩數互換大多需要再一個temp變數，分三行如 `temp=var1`、`var1=var2`、`var2=temp`
4. 從巢狀for loop就能看出bubble sort的時間複雜度是O(n²)，效率差，實務上基本沒人會用。但就入門迴圈邏輯＆資料操作思維來說，應該還是有幫助。至少我希望有幫助...✏️