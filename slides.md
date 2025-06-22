---
# You can also start simply with 'default'
theme: the-unnamed
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://images.unsplash.com/photo-1619075120156-f5729c752edf?q=80&w=2942&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
# some information about your slides (markdown enabled)
title: ch 14 處理巢狀資料的函數式工具 Functional tools for nested data
info: ch 14 處理巢狀資料的函數式工具 Functional tools for nested data
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
# seoMeta:
ogImage: https://drek4537l1klr.cloudfront.net/normand/Figures/rabbit.jpg

fonts:
  # 与 css 中的 font-family 一致，你可以使用 `,` 来分割字体名，便于回退
  sans: 'MapleMono-Regular, MapleMonoNormal-NF-CN-Medium,Robot'
  serif: Robot Slab
  mono: 'MapleMono-Regular, MapleMonoNormal-NF-CN-Medium'
  weights: '200,400,600'
  italic: false
  local: MapleMonoNormal-NF-CN-Medium

---

# ch 14 處理巢狀資料的函數式工具

《簡約軟體開發思維用 Functional Programming 重構程式以 JavaScript 為例》 讀書會

2025/06/26

 Ashley



<div class="abs-br m-6 text-xl">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="slidev-icon-btn">
    <carbon:edit />
  </button>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: slide-up
layout: section
---

# 導讀人：Ashley

- 👩‍💻 近 4 年前端開發經驗
- 🛠 擅長開發 React 、React Native
- 💼 目前在新創公司擔任前端工程師

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/features/slide-scope-style
-->

<!--
Here is another comment.
-->

---
transition: slide-up

---

# 前次回顧


---
layout: section

---
#  本章目標

<br/>

- 建⽴⾼階函式操作 hashmap 中的數值。
- 學習⽤高階函式輕鬆處理深度巢狀資料 •
- 瞭解遞迴以及如何安全地執行•
- 判斷何時在深度巢狀結構上套用抽象屏障。

---
layout: default
 
---
# 14.1 用高階函式處理物件內的值

<img
  class="align-center justify-center w-8/9 h-108"
  src="./image/14.1.png"
  alt=""
/>

---
layout: default
---
# 14.2 讓屬性名稱變顯性

為了讓顧客能增加『商品數量』與『衣服尺⼨』，⾏銷部門先撰寫了以下程式碼：

<div grid="~ cols-2 gap-4">


<div >
  
```js
function incrementQuantity(item) {
    var quantity = item['quantity'];
    var newQuantity = quantity + 1;
    var newItem = objectSet(item, 'quantity', newQuantity);
    return newItem;
}
```
<p >函式名稱包含 quantity 的商品屬性。</p>
</div>
<div>

```js
function incrementSize(item) {
    var size = item['size'];
    var newSize = size + 1;
    var newItem = objectSet(item, 'size', newSize);
    return newItem;
}
```
<p >函式名稱包含 size 的商品屬性。</p>
</div>

</div>


---
layout: section

---


<div v-click.hide>含有程式碼異味的程式</div>

<div v-after>將屬性名稱轉為顯性</div>

````md magic-move {at:1, lines: true}


```js
function incrementQuantity(item) {
    var quantity = item['quantity'];
    var newQuantity = quantity + 1;
    var newItem = objectSet(item, 'quantity', newQuantity);
    return newItem;
}
```

```js 
function incrementField(item, field) {
    var value = item[field];
    var newValue = value + 1;
    var newItem = objectSet(item, field, newValue);
    return newItem;
}
```
````


---
layout: section

---
對負責『increment（增加）』、『 decrement（減少）』 、『 double（加倍）』 和 『 halve（減半）』等四項操作的函式做相同重構後，重複的程式碼就又出現了：

<div grid="~ cols-2 gap-4">
```js {*|3,10}
function incrementField(item, field) {
    var value = item[field];
    var newValue = value + 1;
    var newItem = objectSet(item, field, newValue);
    return newItem;
}

function decrementField(item, field) {
    var value = item[field];
    var newValue = value - 1;
    var newItem = objectSet(item, field, newValue);
    return newItem;
}
```

```js {*|3,10}
function doubleField(item, field) {
    var value = item[field];
    var newValue = value * 2;
    var newItem = objectSet(item, field, newValue);
    return newItem;
}

function halveField(item, field) {
    var value = item[field];
    var newValue = value / 2;
    var newItem = objectSet(item, field, newValue);
    return newItem;
}
```
</div>
<div v-click class="text-center font-bold mt-6 ">函式名稱中的隱性引數 </div>

---
layout: default
---

# 14.3 實作更新物件內屬性值的 update()

寫⼀個能更新（update）物件內屬性值的函數式工具，進而消除程式碼重複。

<img
  class="align-center justify-center "
  src="./image/14.3.jpg"
  alt="">



- 將隱性引數轉換為顯性
- 以回呼取代主體實作

<!--
現在剛剛那四個例子就都可以用更高階的 <code> updateField() </code> 
-->


---
layout: section

---

不需要強調此函式修改的是 Field 屬性，因此我們可以將其重命名為 `update()`。

同時，普適化各參數的名稱。

```js {*|2|3|4}
function update(object, key, modify) {
    var value   = object[key]; //取得屬性值
    var newValue = modify(value); //修改屬性值
    var newObject = objectSet(object, key, newValue); //設定新物件
    return newObject;
}
```

<!--
只要傳入物件、屬性名稱和操作函式，就能更新物件內的屬性值。
objectSet 寫入時複製物件
-->

---
layout: default
---

# 14.4 以 update() 修改物件屬性


假設某公司將員工資訊存放在物件中

```js
var employee = {
    name: "Kim",
    salary: 120000
};
```

人事部門想為該員工加薪10%，此時就需要 `raise10Percent()`：該函式接受『加薪前薪資』，並傳回『漲
10%後』的結果：

```js
function raise10Percent(salary) {
    return salary * 1.1;
}
```

---
layout: section
---

```js {monaco-run}
function objectSet(object, key, value) {
    return {
        ...object,
        [key]: value
    };
}

function update(object, key, modify) {
    var value= object[key]; 
    var newValue= modify(value); 
    var newObject = objectSet(object, key, newValue);
    return newObject;
}

var employee = {
    name: "Kim",
    salary: 120000
};

function raise10Percent(salary) {
    return salary * 1.1;
}

var updatedEmployee = update(employee, 'salary', raise10Percent);
console.log('employee', JSON.stringify(employee,null,2)); 
console.log('updatedEmployee', JSON.stringify(updatedEmployee,null,2)); 

```


---
layout: default
---

## update() 會修改原本 hash map 的資料嗎？
<p v-click>
不會，`update()` 不會更動原始 hash map ，會複製原本的物件，並回傳一個新的物件。
</p>
<br/>
<br/>

## 但假如我們在 `update()` 不會修改原始物件，那該怎麼使用？

<p v-click>
只要用 `update()` 回傳的物件取代原本的物件即可。

```js
var employee = {
    name: "Kim",
    salary: 120000
};
employee = update(employee, 'salary', raise10Percent);
```
</p>


---
layout: default
---
# 14.5 重構 3 : 以 update() 取代 『取得、修改、設定』

<div grid="~ cols-2 gap-4">
<div>
重構前

```js 
function incrementField(item, field) {
    var value = item[field];
    var newValue = value + 1;
    var newItem = objectSet(item, field, newValue);
    return newItem;
}
```

1. **取得**物件內的屬性值
2. **修改**該屬性值
3. 依循寫入時複製的原則，利用新的屬性值**設定**物件。

</div>
<div v-click>
重構後
```js
function incrementField(item, field) {
    return update(item, field, function(value) {
        return value + 1;
    });
}
```
</div>
</div>


---
layout: default
---
## 重構 3 的步驟

以 `update()` 取代 『取得、修改、設定』 共包括兩個步驟 ：
1. 辨識「取得、修改、設定』程式段落。
2. 利⽤ `update()` 取代以上三個段落，其中『修改』的部分為回呼。

<br/>

<div grid="~ cols-2 gap-4">

<div>

### 步驟1： 辨識「取得、修改、設定』程式段落

```js {*|2|3|4|*}
function halveField(item, field) {
    var value = item[field]; // 取得
    var newValue = value / 2; // 修改
    var newItem = objectSet(item, field, newValue); // 設定
    return newItem;
}
```
</div>

<div >

### 步驟2： 利⽤ `update()` 取代以上三個段落
```js{*|3|*}
function halveField(item, field) {
    return update(item, field, function(value) {
        return value / 2; //將修改操作以回呼形式傳入 update()
    });
}
```

</div>


</div>
---
layout: default
---

# 14.6 函數式工具 ---  update() 

`update()` 是另一個重要的函數式工具。我們在前幾章學到的函數式工具主要是操作陣列，而這個工具則是操作物件（視為 hash map）。讓我們更深入地了解它：

```js
function update(object, key, modify) {
    var value = object[key];
    var newValue = modify(value);
    var newObject = objectSet(object, key, newValue);
    return newObject;
}
```


---
layout: default
---

`update()` 可接受一個操作函式，並將其套用在物件內指定屬性上。

<img
  class="align-center justify-center"
  src="./image/14.6.jpg"
  alt="">

  `update()` 需要三個參數：
  - (1) 要修改的物件
  - (2) 用於定位要修改值的鍵
  - (3) 用於修改值的函式。

請確保傳入 `update()` 的函式是一個計算函式。該函式會接收一個參數（目前的值），並回傳新的值。

---
layout: center
class: align-center justify-center flex items-center
---
  
```js {*|2,3|4}
function incrementField(item, field) {
    // 將物件（商品）傳入 update()，並指定要修改的屬性（field），將能修改該屬性的回呼函式傳入
    return update(item, field, function(value) {
        return value + 1;// 回呼函式加一後的新屬性傳回
    });
}
```


---
layout: default
---

# 14.7 將 update() 的行為視覺化

<img
  class="align-center justify-center"
  src="./image/14.7.jpg"
  alt="">

⽬標是執⾏以下指令，將上述物件的商品數量加倍：

```js {*|2}
> update(shoes, 'quantity', function(value) {
    return value * 2; // 數值加倍
});
```

---
layout: default
---
⼀步步拆解 `update（ ）`中的操作：

```js {*|2|3|4|*}
function update(object, key, modify) {
    var value = object[key]; // step1 取得屬性值
    var newValue = modify(value); // step2 修改屬性值
    var newObject = objectSet(object, key, newValue); // step3 設定新物件
    return newObject;
}
```

<div v-if="$slidev.nav.clicks === 1">
操作1：取得鍵（key）所指定的物件屬性值（value）
 <img
  class="align-center justify-center"
  src="./image/f0363-02.jpg"
  alt="">
</div>

<div v-if="$slidev.nav.clicks === 2">
操作2：呼叫回呼函式以處理前面取得的屬性值
<img
  class="align-center justify-center"
  src="./image/f0363-03.jpg"
  alt="">
</div>
<div v-if="$slidev.nav.clicks === 3">
操作3：產生具有屬性新值的物件複本
  <img
    class="align-center justify-center"
    src="./image/f0363-04.jpg"
    alt="">
  </div>




---
  layout: default
---
# 練習 14-1

 `lowercase()` 的函式，可以將傳入的字串轉換為小寫。使用者的電子郵件地址存放在 `email` 鍵下。使用 `update()` 將 `email` 屬性的值全部轉換為小寫。


```js
var user = {
    firstName: "Joe",
    lastName: "Nash",
    email: "JOE@EXAMPLE.COM",
    …
};
```


## 解答
<v-click>
```js 
> update(user, 'email', lowercase) 
  
{ 
    firstName: "Joe", 
    lastName: "Nash", 
    email: "joe@example.com", 
    … 
}
```

</v-click>

---
layout: default
---

# 練習 14-2

Mega Mart 的行銷部門想讓顧客更容易大量採購。為此，他們規劃在購買頁面加入一個『10x』按鈕，能直接將目前的商品數量乘以10。你的任務是利用 `update()` 撰寫一個名為 `tenXQuantity()` 的函式，能將商品物件（item）中的數量（quantity）屬性值乘以10。以下是 item 物件的範例：

```js {*|4}
var item = {
    name: "shoes",
    price: 7,
    quantity: 2, // 將此屬性值乘以10
    …
};
```
## 解答

<v-click>
```js
function tenXQuantity(item) { 
    return update(item, 'quantity', function(quantity) { 
        return quantity * 10; 
    }); 
}

```
</v-click>

--- 
layout: default
---
# 練習 14-3
 <div grid="~ cols-2 gap-4">
 <div>
請參考以下資料結構與可使用的3個函式：

```js
var user = {
    firstName: "Cindy",
    lastName: "Sullivan",
    email: "cindy@randomemail.com",
    score: 15,
    logins: 3
};
```
</div>
 <div>
本題組可用的函式：

- `increment()`: 將給定屬性的值加1。
- `decrement()`: 將給定屬性的值減1。
- `uppercase()`: 將給定字串的字母轉為大寫。
</div>
</div>

1. 請問以下程式碼會輸出什麼結果？
<div grid="~ cols-2 gap-4">
```js
> update(user, 'score', increment).score
```
<v-click>
```js
16
```
</v-click>
</div>


2. 請問以下程式碼會輸出什麼結果？
<div grid="~ cols-2 gap-4">
```js
> update(user, 'logins', decrement).score
```
<v-click>
```js
15
```
</v-click>
</div>

3. 請問以下程式碼會輸出什麼結果？
<div grid="~ cols-2 gap-4">
```js
> update(user, 'firstName', uppercase).firstName
```
<v-click>
```js
"CINDY"
```
</v-click>
</div>

---
layout: center
---

<img
  class="align-center justify-center"
  src="./image/14-13.png"
  alt="">


---
layout: section
---

```js {*|4,5,6,7|11,12,13,14,15|}
var shirt = {
    name: "shirt",
    price: 13,
    options: { // shirt 物件中還有 options 物件，形成巢狀結構
        color: "blue",
        size: 3 // 必須存取到options 物件裡面的屬性值
    }
};
 
function incrementSize(item) {
    var options = item.options; //取得
    var size = options.size; //取得
    var newSize = size + 1; // 修改
    var newOptions = objectSet(options, 'size', newSize); // 設定
    var newItem = objectSet(item, 'options', newOptions); // 設定
    return newItem;
}
```
---
layout: default
---
# 14.8 將巢狀資料的 update() 視覺化
<div grid="~ cols-2 gap-4">

```js {*|2,3|4,5|6,7|8,9|10,11|}
function incrementSize(item) {
    //step1
    var options = item.options; //取得
    //step2
    var size = options.size; //取得
    //step3
    var newSize = size + 1; // 修改
    //step4
    var newOptions = objectSet(options, 'size', newSize); // 設定
    //step5
    var newItem = objectSet(item, 'options', newOptions); // 設定
    return newItem;
}
```

<div v-if="$slidev.nav.clicks === 1">
操作 1：取得鍵（key）所指定的物件
 <img
  class="align-center justify-center"
  src="./image/f0368-01.jpg"
  alt="">
</div>

<div v-if="$slidev.nav.clicks === 2">
操作 2：取得鍵（key）所指定的物件屬性值（value）
<img
  class="align-center justify-center"
  src="./image/f0368-02.jpg"
  alt="">
</div>
<div v-if="$slidev.nav.clicks === 3">
操作 3：產⽣屬性新值
<img
  class="align-center justify-center"
  src="./image/f0368-03.jpg"
  alt="">
  </div>
  <div v-if="$slidev.nav.clicks === 4">
操作 4：產生具有數性新值的物件複本
<img
  class="align-center justify-center"
  src="./image/f0368-04.jpg"
  alt="">
  </div>
    <div v-if="$slidev.nav.clicks === 5">
操作 5： 產⽣具有屬性新值的物件複本
<img
  class="align-center justify-center"
  src="./image/f0368-05.jpg"
  alt="">
  </div>

</div>

---
layout: default
---
# 14.9 用 update() 處理巢狀資料

以 `update()`重構 `incrementSize()`
```js 
function incrementSize(item) {
    var options = item.options; // 取得
    var size = options.size; // 取得
    var newSize = size + 1; // 修改
    var newOptions = objectSet(options, 'size', newSize); // 設定
    var newItem = objectSet(item, 'options', newOptions); // 設定
    return newItem;
}
```


---
layout: section
---


重構 3 的步驟

1. 辨識『取得』、『修改 』與『設定』程式段落。
2. 利用 `update()`  取代以上三個段落 ，其中『修改』的部分為回呼 。

<div v-click.hide>原始程式</div>

<div v-after>重構後</div>

````md magic-move {at:1, lines: true}


```js
function incrementSize(item) {
    var options = item.options; // 取得
    var size = options.size; // 取得
    var newSize = size + 1; // 修改
    var newOptions = objectSet(options, 'size', newSize); // 設定
    var newItem = objectSet(item, 'options', newOptions); // 設定
    return newItem;
}
```

```js 
function incrementSize(item) {
    var options = item.options; // 取得


    var newOptions = update(options, 'size', increment); // 修改  
    var newItem = objectSet(item, 'options', newOptions); // 設定
    return newItem;
}

```

```js 

function incrementSize(item) {
    return update(item, 'options', function(options) {
        return update(options, 'size', increment);
    });
}

```
````

---
layout: default
---

# 14.10 實作成普適化的updateOption（）


```js
var shirt = {
    name: "shirt",
    price: 13,
    options: {
        color: "blue",
        size: 3
    }
};
```

```js {*|3|*}
function incrementSize(item) {
    return update(item, 'options', function(options) {
        return update(options, 'size', increment); // 可以看到，函式名稱中又出現了隱性引數，而且還出現了兩個
    });
}

```
由於 size 屬性值位於兩層巢狀結構內（經過兩個物件才能取得該值），故需呼叫兩次 update（）。

**資料的巢狀深度有多少，update（）的巢狀呼叫次數就有多少**

---
layout: section
---
將引數改為顯性：先處理 size（隱性屬性名稱），再處理 increment（隱性修改操作）


````md magic-move {at:1, lines: true}


```js {*|1,2,4|*}
// 1.包含隱性屬性名稱
function incrementSize(item) {
    return update(item, 'options', function(options) {
        return update(options, 'size', increment);
    });
}
```

```js {*|1,2,4|}
// 2.改為顯性屬性名稱
function incrementOption(item, option) { 
    return update(item, 'options', function(options) {
        return update(options, option, increment);
    });
}

```

```js {*|1,2,4|*}
// 3.包含隱性修改操作
function incrementOption(item, option) {
    return update(item, 'options', function(options) {
        return update(options, option, increment);
    });
}

```
```js {*|1,2,4|*}
// 4.改為隱性修改操作
function updateOption(item, option, modify) {
    return update(item, 'options', function(options) { // 函式名稱仍有隱性引數
        return update(options, option, modify);
    });
}

```
````
---
layout: default
---

# 14.11 實作兩層巢狀結構的 update２()

````md magic-move {at:1, lines: true}


```js 
// 1.包含隱性引數
function updateOption(item, option, modify) {
    return update(item, 'options', function(options) {
        return update(options, option, modify);
    });
}
```

```js 
// 2.改為顯性引數
function update2(object, key1, key2, modify) { //函式名稱中的2表示用於兩層巢狀結構
    return update(object, key1, function(value1) {
        return update(value1, key2, modify);
    });
}

```
````

<div v-if="$slidev.nav.clicks === 0">
- 此處的 option 隱藏在函式名稱中，需改成顯性引數
</div>
<div v-if="$slidev.nav.clicks === 1">
將外層的Key 參數名改爲 key1，內層的Key 參數名改爲 key2，並將 item 改為 object
</div>