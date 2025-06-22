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
---
layout: default
---

比較一下重構後的版本：

對 options 的 size 屬性加 1 的操作：

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


<p v-if="$slidev.nav.clicks === 0">原始程式</p>
<p v-if="$slidev.nav.clicks === 1"> 改以 update2() 實作</p>

````md magic-move {at:1, lines: true}


```js 
function incrementSize(item) {
    var options = item.options;
    var size = options.size;
    var newSize = size + 1;
    var newOptions = objectSet(options, 'size', newSize);
    var newItem = objectSet(item, 'options', newOptions);
    return newItem;
}
```

```js 
function incrementSize(item) {
 
    return update2(item, 'options', 'size', function(size) {
        return size + 1;
    });
 
 
}

```
````
---
layout: default
---

# 14.12 視覺化說明 update2（）如何操作巢狀物件

## 路徑 (path) 

用來定位巢狀物件中之屬性值的鍵序列，其中每一個鍵對應一個巢狀層。

> 本例的目標是『對 size 屬性值加 1』。
為此  `update2（）`需先取得 item 物件，
接著進入『'options'』鍵所指定的物件，然後才能存取到『'size'』鍵的值。

```js

> return update2(shirt, 'options', 'size', function(size) {
        return size + 1; // 對 size 屬性值加 1
    });
```

'options', 'size' 通往目標屬性的路徑

---
layout: center

---


<div class="flex gap-4">


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



  <img
    class="align-center justify-center"
    src="./image/f0372-01.jpg"
    alt="">
</div>

---
layout: section

---

## 進入巢狀結構 （取得一>取得一>修改）

<img
    class="align-center justify-center"
    src="./image/f0372-02.jpg"
    alt="">

---
layout: section
---

## 離開巢狀結構（設定 ⼀> 設定）

<img
    class="align-center justify-center"
    src="./image/f0372-03.jpg"
    alt="">

---
layout: default
---
當遇到三層巢狀結構？

<div class="flex gap-4">
```js
var cart = {
    shirt: {
        name: "shirt",
        price: 13,
        options: {
            color: "blue",
            size: 3
        }
    }
}
```

<img
    class="align-center justify-center"
    src="./image/f0373-02.jpg"
    alt="">
</div>

---
layout: default

---

# 14.13 函式incrementSizeByName()的4種實作方法
此函式能接受購物⾞ cart 參數和商品名 name 參數，然後將購物⾞內對應商品的 size 屬性值（位於 options 物件中）加 1。

<div grid="~ cols-2 gap-1">

<div>
<p class="text-sm"> 方法1: 使⽤ `update()` 和 `incrementSize()`</p>

```js
function incrementSizeByName(cart, name) {
    return update(cart, name, incrementSize);
}
```
</div>
<div>
<p class="text-sm"> 方法2：使⽤ `update()` 和 `update2()`</p>

```js
function incrementSizeByName(cart, name) {
    return update(cart, name, function(item) {
        return update2(item, 'options', 'size', function(size) {
            return size + 1;
        });
    });
}
```
</div>


</div>
---
layout: section
---

<p class="text-sm"> 方法3：只⽤ `update()` </p>
```js
function incrementSizeByName(cart, name) {
    return update(cart, name, function(item) {
        return update(item, 'options', function(options) {
            return update(options, 'size', function(size) {
    return size + 1;
            });
        });
    });
}
```


<p class="text-sm">方法4：⾃⾏實作所有『 取得 、 修改 、 設定 』步驟</p>

```js
function incrementSizeByName(cart, name) {
    var item    = cart[name];
    var options  = item.options;
    var size    = options.size;
    var newSize  = size + 1;
    var newOptions = objectSet(options, 'size', newSize);
    var newItem  = objectSet(item, 'options', newOptions);
    var newCart  = objectSet(cart, name, newItem);
    return newCart;
}
```

---
layout: default
---

# 14.14 實作三層巢狀結構的 update3 ()

<div class="flex gap-4">
<div>
<p v-if="$slidev.nav.clicks === 0">方法二</p>
<p v-if="$slidev.nav.clicks === 1">重構後</p>

````md magic-move {at:1, lines: true}
```js
function incrementSizeByName(cart, name) {
    return update(cart, name, function(item) {
        return update2(item, 'options', 'size', function(size) {
            return size + 1;
        });
    });
}

```

```js
function incrementSizeByName(cart, name) {
    return update3(cart, name,'options', 'size',
     function(size) {
        return size + 1;
    });
}

function update3(object, key1, key2, key3, modify) {
    return update(object, key1, function(object2) {
        return update2(object2, key2, modify);
    });
}
```
```

````
</div>
<div v-if="$slidev.nav.clicks === 0">

1. 辨識出函式名稱裡的隱性引數，擷取成 `update3()` 。
2. 加入新參數以接收顯性輸入。
3. 利用新參數取代函式實作中的固定值

</div>

<div v-if="$slidev.nav.clicks === 1">
`update3()`參數

- `object`：購物車物件
- `key1`：商品名稱
- `key2`：商品名稱內的 options 物件
- `key3`：商品名稱內 options 物件中的屬性
- `modify`：用於修改值的函式

`update3()` 相當於 『 在 `update()` 中呼叫 `update2()` 』。
</div>
</div>

---
layout: default
---

# 練習 14-4

## update4()

<v-click>

```js
function update4(object, k1, k2, k3, k4, modify) { 
    return update(object, k1, function(object2) { 
        return update3(object2, k2, k3, k4, modify); 
    });  
}
```
</v-click>


## update5()

<v-click>

```js
function update5(object, k1, k2, k3, k4, k5, modify) {
    return update(object, k1, function(object2) {
        return update4(object2, k2, k3, k4, k5, modify);
    });
}
```
</v-click>

---
layout: default
---
# 14.15 實作任意與狀深度的 nestedUpdate()

觀察以下規律：
```js

function update3(object, key1, key2, key3, modify) {
    return update(object, key1, function(value1) {
        return update2(value1, key2, key3, modify);
    });
}

function update4(object, key1, key2, key3, key4, modify) {
    return update(object, key1, function(value1) {
        return update3(value1, key2, key3, key4, modify);
    });
}
```

要定義 `updateX()` ，只要在 `update()` 中呼叫 `updateX - 1()` 即可︔
此時 `update()` 會使⽤第⼀個鍵，剩餘的鍵則按順序、連同 `modify` 函式引數一起傳給
`updateX - 1()`。

X 剛好等於鍵的個數、而這些鍵又共同組成路徑，可以將 X 解釋成『路徑長度』或『巢狀深度』。

---
layout: full
---
`update2()` 的實作如下︰
```js
function update2(object, key1, key2, modify) {
    return update(object, key1, function(value1) {
        return update1(value1, key2, modify);
    });
}
```
`update1()` 又如何呢？注意 X - 1 會變成 0
```js
function update1(object, key1, modify) {
    return update(object, key1, function(value1) {
        return update0(value1, modify);
    });
}
```
`update0()`無法套用該模式： 

1. `update0()` 中沒有鍵，因此無法呼叫 `update()`（沒有『第一個鍵』可供 `update()` 使用）。 
2. 此處的 X - 1 會變成 - 1，這並非合理的路徑長度。
```js
function update0(value, modify) {
    return modify(value);
}
```
---
layout: statement

---
從剛剛的觀察發現程式再度飄出『函式名稱中的程式碼異味』：

# 函式名稱中的數字總是和鍵的數量相同

---
layout: default

---

## 將隱性引數轉為顯性

```js
function update3(object, key1, key2, key3, modify) {
    return update(object, key1, function(value1) {
        return update2(value1, key2, key3, modify);
    });
}

```
<v-click>
加入⼀個代表『 巢狀深度 』 的 depth 參數 ：

```js
function updateX(object, depth, key1, key2, key3, modify) {
    return update(object, key1, function(value1) {
        return updateX(value1, depth-1, key2, key3, modify);
    });
}
```
以上改寫確實讓『巢狀深度』變顯性了，
但如何確保 depth 的值應該要和鍵的數量相同呢？
</v-click>

<v-click>
假如將所有鍵按順序存成一個陣列（也就是將 key1、key2、key3、…，改用一個 keys 陣列取代），那就不需設額外的 depth 參數了，也就是說該陣列的長度（即：元素數量）就是『巢狀深度』！

```js
function updateX(object, keys, modify) {} //keys 包含所有鍵的陣列
```
</v-click>
---
layout: default
---

先將陣列中的第⼀個鍵取出並傳入 `update（）`，然後把排在後面的那些鍵存成「restOfKeys」陣列再傳給 `updateX（）`。注意！restOfKeys 陣列的長度為 X - 1 ：

```js {*|2|3|*} 
function updateX(object, keys, modify) {
    var key1 = keys[0]; //呼叫 update（）時傳入第一個鍵
    var restOfKeys = drop_first(keys); //在遞迴呼叫之前，先把第一個鍵從 keys 陣列中刪除
    return update(object, key1, function(value1) {
        return updateX(value1, restOfKeys, modify);
    });
}
```

除了 `update0()` 以外，以上實作可適用於所有正整數。
---
layout: default   
---
## update0()

```js
function update0(value, modify) {
    return modify(value);
}
```

只需多加一個判斷式：若 keys 陣列的長度為零（即：沒有鍵），則直接呼叫 `modify()`，否則遞迴呼叫 `updateX()`。

````md magic-move {at:1, lines: true}
```js
function updateX(object, keys, modify) {
 
 
    var key1 = keys[0];
    var restOfKeys = drop_first(keys);
    return update(object, key1, function(value1) {
        return updateX(value1, restOfKeys, modify);
    });
}
```

```js
function updateX(object, keys, modify) {
    if(keys.length === 0)
      return modify(object);
    var key1 = keys[0];
    var restOfKeys = drop_first(keys);
    return update(object, key1, function(value1) {
        return updateX(value1, restOfKeys, modify);
    });
}
```
```

````

<br/>

### 遞迴基本條件(base case)

所有遞迴呼叫皆應收斂到某種不涉及遞迴呼叫的情況，稱為基本條件。

---
layout: default
---
## 修改函式名稱

將updateX()改成 nestedUpdate()

````md magic-move {at:1, lines: true}
```js
function updateX(object, keys, modify) {
    if(keys.length === 0)
      return modify(object);
    var key1 = keys[0];
    var restOfKeys = drop_first(keys);
    return update(object, key1, function(value1) {
        return updateX(value1, restOfKeys, modify);
    });
}
```

```js
function nestedUpdate(object, keys, modify) {
    if(keys.length === 0)
      return modify(object);
    var key1 = keys[0];
    var restOfKeys = drop_first(keys);
    return update(object, key1, function(value1) {
        return nestedUpdate(value1, restOfKeys, modify);
    });
}
```
```

````
---
layout: section

---



# 什麼是遞迴 ？

<v-click>

定義⼀個函式時，你可以在實作中呼叫任何東西，包括你正在定義的函式本身。這種『在定義中呼叫自己』的做法就叫做遞迴（recursive），
`nestedUpdate()`  就是很好的例子。

```js
function nestedUpdate(object, keys, modify) {
    if(keys.length === 0)
      return modify(object);
    var key1 = keys[0];
    var restOfKeys = drop_first(keys);
      return update(object, key1, function(value1) {
      return nestedUpdate(value1, restOfKeys, modify);
    });
}
```
</v-click>

---
layout: section
---

# 為什麼要⽤遞迴這麼難懂的方式寫程式？

<v-click>

遞迴寫法特別適合對付巢狀資料。

操作巢狀資料時，往往需對每個層做類似處理，而遞迴函式剛好能做到這一點
（每一層都呼叫該函式，只不過傳入不同引數）。
</v-click>
<v-click>

# 難道不能用較好理解的 for 迴圈達到同樣的目的嗎?
</v-click>

<v-click>

遞迴可利用函式呼叫堆疊（stack）追蹤每一輪的引數與傳回值。

要是使用for週圈，則必須自行處理此問題。

總而言之，有了JavaScript的堆疊功能，我們不必手動push和pop（將資料放進堆疊稱為push，移出則稱為pop），故省去了許多麻煩
</v-click>
---
layout: section
---

# 但遞迴不會很危險嗎︖ 萬一產生無限迴圈或記憶體溢位怎麼辦？

<v-click>

遞迴的確有可能造成無限迴圈。

此外，根據你的實作和所用語言，函式呼叫致於有可能遞迴太多次以至於堆疊空間不足。

</v-click>

---
layout: default
---
# 14.16 安全的遞迴需要具備什麼？


## 1 .⼀定要有基本條件

<p>要防止遞迴無限循環下去，就一定要<span v-mark="{  color: 'red', type: 'underline' }">定義不含任何遞迴呼叫</span>的 <span v-mark="{  color: '#234', type: 'circle' }">基本條件</span>，並以此當做終點。</p>

  ```js {*|2,3,|*}
  function nestedUpdate(object, keys, modify) {
      if(keys.length === 0) // 基本條件
        return modify(object); //其中不含任何遞迴呼叫
      var key1 = keys[0];
      var restOfKeys = drop_first(keys);
        return update(object, key1, function(value1) {
        return nestedUpdate(value1, restOfKeys, modify);
      });
  }
  ```

基本條件通常發生在 『 當某次呼叫的傳入引數為空陣列 』 、 『 遞減變數值變成 0 』
或 『 答案已經找到 』時 。 

---
layout: default

---

## 2. 弄清楚遞迴的條件

<p>遞迴函式的定義中<span v-mark="{  color: 'red', type: 'underline' }">至少要包含一次遞迴條件</span>，也就是包含了遞迴呼叫的敘述。</p>
此處同樣以
`nestedUpdate()`為例:
```js {*|5,7|*}
function nestedUpdate(object, keys, modify) {
    if(keys.length === 0) 
      return modify(object); 
    var key1 = keys[0];
    var restOfKeys = drop_first(keys);// 每經過一次遞迴呼叫，restOfKeys 就會少一個元素
      return update(object, key1, function(value1) {
      return nestedUpdate(value1, restOfKeys, modify);//遞迴呼叫
    });
}
```

<v-click>


## 3. 確定函式呼叫有朝著基本條件前進

必須確保其中至少有⼀個引數正在『變小』，並且使得呼叫條件越來越接近基本條件。

舉例：假如『以空陣列做遞迴呼叫』是基本條件，那每次遞迴時就要移除該陣列的一個元素。

</v-click>

---
layout: default
---

# 14.17 將 nestedUpdate() 的行為視覺化

```js
> nestedUpdate(cart, ["shirt", "options", "size"], increment)
```

下方為巢狀結構執行狀態的堆疊（Stack）圖，每呼叫一次 `nestedUpdate()`就往下堆疊一層，直到執行`modify()`，然後再一層一層執行`objectSet()`從堆疊中移除。

<img
  class="align-center justify-center"
  src="./image/f0383-01.jpg"
  alt="">
  <img
  class="align-center justify-center"
  src="./image/f0383-02.jpg"
  alt="">

---
layout: full
---

<img
  class="align-center justify-center"
  src="./image/f0383-03.jpg"
  alt="">
  <img
  class="align-center justify-center"
  src="./image/f0383-04.jpg"
  alt="">
<img
  class="align-center justify-center"
  src="./image/f0383-05.jpg"
  alt=""><img
  class="align-center justify-center"
  src="./image/f0383-06.jpg"
  alt="">
---
layout: full
---

  <img
  class="align-center justify-center"
  src="./image/f0383-07.jpg"
  alt="">
  
---
layout: default
---

# 14.18 比較遞迴和迴圈

## 迴圈

迴圈⾛訪陣列時，程式會從索引0開始， ⼀邊處理其中的元素，⼀邊將⽣成的結果加到傳回陣列末端，如下圖所示：

<img
  class="align-center justify-center"
  src="./image/f0384-02.jpg"
  alt="">

---
layout: full

---


巢狀資料的操作⽅式則不⼀樣：我們需要先⼀層⼀層進⾏『取得』，接著『修改』⽬標屬性值，最後再循環相反方向完成每⼀層的
『設定』。此外，因為使用了『寫入時複製』，故『設定』其實會產生資料複本：

<img
  class="w-5/9 h-5/9"
  src="./image/f0384-03.jpg"
  alt="">


事實上，『取得、修改、設定』的巢狀執行方式恰好反映巢狀結構，而這樣的構造很難不使用遞迴和呼叫堆疊實現。
<img
  class="align-center justify-center w-1/3 h-1/3"
  src="./image/f0384-04.jpg"
  alt="">

---
layout: section
---

# 練習 14-5
在 14.13 節已介紹過 incrementSizeByName() 的四種實作方法。在此請各位利用
nestedUpdate() 寫出第五種實作：

```js
> function incrementSizeByName (cart, name) {}
```

<v-click>
```js
function incrementSizeByName(cart, name) { 
    return nestedUpdate(cart, [name, 'options', 'size'],
        function(size) {
            return size + 1;
    });
}
```
</v-click>

---
layout: default
---

# 14.19 遇到深度巢狀資料時的設計考量

用 `nestedUpdate()` 處理深度巢狀資料時，可能會遇到以下問題：

需要傳入一長串鍵作為路徑，但我們很難記得這些鍵到底指的是什麼？

ex:

```js

httpGet("http://my-blog.com/api/category/blog", function(blogCategory) {
    renderCategory(nestedUpdate(blogCategory, ['posts', '12', 'author', 'name'], capitalize));
});

```

每一個巢狀層都有自己的資料結構，必須記得這些結構，才能了解路徑的意思。

---
layout: default

---

# 14.20  為巢狀資料建立抽象屏障

## 抽象屏障（abstraction barrier）:

是有效隱藏實作細節的函式層，有了它，使用屏障中的函式時，完全不需要了解函式的底層實作。
<br/>

舉例：建立可操作目標資料結構的函式，並且賦予這些函式有意義的名稱。

寫⼀個能根據給定貼⽂編號（ ID ）修改部落格貼⽂（ post ）的函式（貼⽂儲存在 category 巢狀物件的『posts』鍵下）

```js
function updatePostById(category, id, modifyPost) { //不需知道 posts 和 category 的關係也能用
    return nestedUpdate(category, ['posts', id], modifyPost); // 位於屏障上層的程式不必知道 category 的資料結構
}
```
變更作者資訊（user，存放於『author』鍵下）
```js
function updateAuthor(post, modifyUser) {
    return update(post, 'author', modifyUser);
}
```

---
layout: default
---

將作者名字改為大寫（利用 capitalize() 完成），所以需再實作能取得「name」屬性的函式，並傳入 capitalize() 

```js
function capitalizeName(user) {
    return update(user, 'name', capitalize);
}
```
結合以上，就可以得到：

```js
updatePostById(blogCategory, '12', function(post) {
    return updateAuthor(post, capitalizeUserName);
});
```
<br/>

## 使用抽象屏障優點

- 函式的名稱簡單易懂
- 不知道鍵的名稱也能夠順利修改目標屬性

---
layout: default
---

# 14.21 總結高階函式的應用

## 在⾛訪陣列時取代for迴圈

`forEach（）`、`map（）`、`filter（）`、與`reduce（）`，皆是能操作陣列的⾼階函式。你可以將它們串連成鏈，以實現更複雜的計算。

## 有效處理巢狀資料

要改變巢狀資料中的值非常麻煩，不僅需進行多次『取得』，還得對每一層的物件做『寫入時複製』。為此，我們實作了 `update（）` 和 `nestedUpdate（）` — 無論巢狀結構有多深，這兩個高階函式都能精準更改指定屬性值。

---
layout: section
---

## 套用『寫入時複製』

『寫入時複製』的步驟是固定的（即：產生複本、修改複本、傳回複本），故實作時可能產生重複程式碼。但只要使用 `withArrayCopy（）` 和 `withObjectCopy （）`，就能把任意操作協調成『寫入時複製』版本。以上兩個高階函式是將固定程式標準化的極佳範例。

## 將 try/catch 敘述標準化

我們曾寫過名為 `wrapLogging（）` 的高階函式：其可接受任意函式`f`，並傳回功能和「相同，但多了 `try/catch` 錯誤捕捉能力的新函式。這個例子讓我們瞭解：高階函式可改變其他函式的行為。

---
layout: default
---

# 重點整理

- 函數式工具 `update()` 能修改物件中的指定屬性值，為我們免去手動取得屬性值、修改，然後再將結果設定回物件中的麻煩。

- `nestedUpdate()` 能處理深度巢狀資料。當知道目標屬性值在巢狀物件中的路徑（由一系列的『鍵』構成）時，就能用此函數式工具輕鬆修改該值。

- 一般來說，迴圈比遞迴容易理解。但面對巢狀資料時，遞迴則比較好用。

- 函式在呼叫自己之前，遞迴會利用函式呼叫堆疊來追蹤目前進度，這使得遞迴函式的構造能反映巢狀資料結構。

- 深度巢狀結構會造成理解困難上的不便。要操作此類資料，你必須記得每一巢狀層的資料結構為何。

- 你可以更簡化關鍵資料結構設計上的抽象不便，藉此降低需要記憶的細節。這麼做能讓巢狀資料操作更簡單。
