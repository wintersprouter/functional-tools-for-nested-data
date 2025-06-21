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

Hover on the bottom-left corner to see the navigation's controls panel, [learn more](https://sli.dev/guide/ui#navigation-bar)

## Keyboard Shortcuts

|                                                     |                             |
| --------------------------------------------------- | --------------------------- |
| <kbd>right</kbd> / <kbd>space</kbd>                 | next animation or slide     |
| <kbd>left</kbd>  / <kbd>shift</kbd><kbd>space</kbd> | previous animation or slide |
| <kbd>up</kbd>                                       | previous slide              |
| <kbd>down</kbd>                                     | next slide                  |

<!-- https://sli.dev/guide/animations.html#click-animation -->
<img
  v-click
  class="absolute -bottom-9 -left-7 w-80 opacity-50"
  src="https://sli.dev/assets/arrow-bottom-left.svg"
  alt=""
/>
<p v-after class="absolute bottom-23 left-45 opacity-30 transform -rotate-10">Here!</p>

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

<div v-after>將屬性名稱轉為顯性 </div>

````md magic-move {at:1, lines: true}


```js
function incrementQuantity(item) {
    var quantity = item['quantity'];
    var newQuantity = quantity + 1;
    var newItem = objectSet(item, 'quantity', newQuantity);
    return newItem;
}
```

```ts 
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
layout: center
---
⼀步步拆解 `update（ ）`中的操作：

<img
  class="align-center justify-center"
  src="./image/figure_14-1.png"
  alt="">

---
layout: default
---
# 操作1：取得鍵（key）所指定的物件屬性值（value）

<img
  class="align-center justify-center"
  src="./image/f0363-02.jpg"
  alt="">



---
layout: default
---

# 操作2：呼叫回呼函式以處理前面取得的屬性值

<img
  class="align-center justify-center"
  src="./image/f0363-03.jpg"
  alt="">

---
layout: default
---
# 操作3：產生具有屬性新值的物件複本

<img
  class="align-center justify-center"
  src="./image/f0363-04.jpg"
  alt="">


---
  layout: default
---
# 練習 14-1

We have a function called lowercase() that will convert a string to lowercase. Users have an email address under the 'email' key. Using update(), modify the user record by applying lowercase() to their email.