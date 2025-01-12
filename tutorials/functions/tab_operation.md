---
id: tabs
title: '🥦 多标签页管理'
---

<div class="wwads-cn wwads-horizontal" data-id="317"></div><br/>

浏览器的标签页由 Tab 对象（`ChromiumTab`和`MixTab`）控制。

默认情况下，它们是一一对应关系。

当禁用单例模式后，一个标签页也可以被多个 Tab 对象同时控制。

多个 Tab 对象可以同时操作，不需要切换焦点，也不需要激活到前台。

要操作哪个标签页，只要用属于它的 Tab 对象操作就可以了。

## ✅️️ 获取标签页对象

### 📌 获取最后激活的标签页

`Chromium`对象的`latest_tab`属性返回最后激活的标签页对象。

:::tip 说明
    如`Settings.singleton_tab_obj`为`True`，此属性返回标签页对象的 tab id。
:::

```python
from DrissionPage import Chromium

browser = Chromium()
tab = browser.latest_tab  # 获取最新标签页对象
```

---

### 📌 获取指定标签页

`Chromium`对象的`get_tab()`和`get_tabs()`方法用于获取指定的标签页对象。

可指定标签页序号、id、标题、url、类型等条件用于检索。api 详见 “浏览器对象” 章节。

:::tip 说明
    - 当`id_or_num`不为`None`时，其它参数无效
    - `title`、`url`和`tab_type`三个参数是与关系
    - 如传入序号，序号与标签页视觉排序不一定一致，而是按照激活顺序排列。
:::

```python
from DrissionPage import Chromium

browser = Chromium()
tab1 = browser.get_tab(1)  # 获取列表中第一个标签页的对象
tab2 = browser.get_tab('5399F4ADFE3A27503FFAA56390344EE5')  # 获取指定id的标签页对象
tab3 = browser.get_tab(url='DrissionPage.cn')  # 获取第一个url中带 'DrissionPage.cn' 的标签页对象
tabs = browser.get_tabs(url='DrissionPage.cn')  # 获取所有url中带 'DrissionPage.cn' 的标签页对象
```

:::warning 注意
    Tab 对象默认为单例，即一个实体标签页只有一个`MixTab`对象。`get_tab()`返回的标签页可能是同一个。
:::

---

### 📌 新建标签页并获取对象

`Chromium`对象的`new_tab()`方法用于新建一个标签页，返回其对象。

```python
from DrissionPage import Chromium

browser = Chromium()
browser.new_tab(url='https://DrissionPage.cn')
```

:::tip 说明
    当传入`url`参数时，程序会根据`load_mode`设置访问页面，除了`none`模式，都将等待页面加载完毕。
    如果新建多个标签页不想等待，可批量新建不传入`url`参数的标签页，再遍历使用`get()`。
:::

---

### 📌 获取点击后出现的标签页

在预期点击元素会出现新标签页时，可用元素的`click.for_new_tab()`方法实行点击，点击后会返回新标签页对象。

具体参数见元素交互章节。

```python
from DrissionPage import Chromium

tab1 = Chromium().latest_tab
tab2 = tab1('tag:a').click.for_new_tab()  # 点击获取新tab对象
print(tab2.title)  # 使用新tab对象操作
```

元素对象的`click.middle()`方法可用中键点击`<a>`元素，可强制在新标签页打开链接。

返回新标签页对象。

```python
from DrissionPage import Chromium

tab1 = Chromium().latest_tab
tab2 = tab1('tag:a').click.middle()  # 点击获取新tab对象
print(tab1.title)  # 使用新tab对象操作
```

---

## ✅️️ 多标签页协同

这个示例在一个标签页中遍历列表元素，点击打开新标签页，获取信息后关闭。

```python
from DrissionPage import Chromium

tab = Chromium().latest_tab
tab.get('https://gitee.com/explore/all')

links = tab.eles('t:h3')
for link in links[:-1]:
    # 点击链接并获取新标签页对象
    new_tab = link.click.for_new_tab()
    # 等待新标签页加载
    new_tab.wait.load_start()
    # 打印标签页标题
    print(new_tab.title)
    # 关闭新打开的标签页
    new_tab.close()
```

## ✅️️ 使用多例

默认情况下，Tab 对象是单例的，即一个标签页只有一个对象，即使重复使用`get_tab()`，获取的都是同一个对象。

这主要是防止新手不理解机制，反复创建多个连接导致资源耗费。

实际上允许多个 Tab 对象同时操作一个标签页，每个负责不同的工作。比如一个执行主逻辑流程，另外的监视页面，处理各种弹窗。

要允许多例，可用`Settings`设置：

```python
from DrissionPage.common import Settings

Settings.singleton_tab_obj = False
```

**示例**

```python
from DrissionPage import Chromium
from DrissionPage.common import Settings

browser = Chromium()
browser.new_tab()
browser.new_tab()

# 未启用多例：
tab1 = browser.get_tab(1)
tab2 = browser.get_tab(1)
print(id(tab1), id(tab2))

# 启用多例：
Settings.singleton_tab_obj = False
tab1 = browser.get_tab(1)
tab2 = browser.get_tab(1)
print(id(tab1), id(tab2))
```

**输出：**

```shell
2347582903056 2347582903056
2347588741840 2347588877712
```

可见第一次输出两个 Tab 对象是同一个，第二次输出是独立的。
