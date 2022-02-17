# Markdown 标题
```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```


# Markdown 段落

<color_r>红色文字</color_r>  
<color_g>绿色文字</color_g>  
<color_b>蓝色文字</color_b>  
<color_y>黄色文字</color_y>  

*斜体文本*  
_斜体文本_  
**粗体文本**  
__粗体文本__  
***粗斜体文本***  
___粗斜体文本___  


分割线

----------


测试文本  
~~测试文本~~


<u>带下划线文本</u>


# Markdown 列表
无序列表使用星号(*)、加号(+)或是减号(-)作为列表标记，这些标记后面要添加一个空格，然后再填写内容。有序列表使用数字并加上 . 号来表示

* 第一项
* 第二项
* 第三项

----------------------

+ 第一项
+ 第二项
+ 第三项

----------------------

- 第一项
- 第二项
- 第三项

----------------------

1. 第一项
2. 第二项
3. 第三项

列表嵌套只需在子列表中的选项前面添加四个空格即可：

1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素


# Markdown 区块

> 区块引用  
> 菜鸟教程  
> 学的不仅是技术更是梦想  


----------------------

> 最外层  
> > 第一层嵌套  
> > > 第二层嵌套  


# Markdown 代码

`printf()` 函数


```c++
int a = 0;
```

```
int a = 0;
```

```javascript
$(document).ready(function () {
    alert('RUNOOB');
});
```


# Markdown 链接

链接使用方法如下：


```
[链接名称](链接地址)

或者

<链接地址>
```


# Markdown 图片

![找不到图片时的替代文本](/assets/avatar.png)

![找不到图片时的替代文本](/assets/avatar.png "可选标题")

<img src="http://static.runoob.com/images/runoob-logo.png" width="50%">


# Markdown 表格

Markdown 制作表格使用 | 来分隔不同的单元格，使用 - 来分隔表头和其他行。

我们可以设置表格的对齐方式：

  - -: 设置内容和标题栏居右对齐。
  - :- 设置内容和标题栏居左对齐。
  - :-: 设置内容和标题栏居中对齐。

|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |

----------------------

| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |


# Markdown 高级技巧

支持的 HTML 元素，不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。

```
  目前支持的 HTML 元素有：<kbd> <b> <i> <em> <sup> <sub> <br>等
```

使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑

支持转义字符  
**文本加粗**   
\*\* 正常显示星号 \*\*

Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：

```
\   反斜线
`   反引号
*   星号
_   下划线
{}  花括号
[]  方括号
()  小括号
#   井字号
+   加号
-   减号
.   英文句点
!   感叹号
```


# Markdown 数学公式
嵌入了Katex的话可以使用数学公式

$$
f(x) = \int_{-\infty}^\infty
    \hat f(\xi)\,e^{2 \pi i \xi x}
    \,d\xi
$$

$$
\bar{v} = \begin{pmatrix} \color{red}x \\ \color{green}y \\ \color{blue}z \end{pmatrix}
$$


<https://katex.org/docs/supported.html>  
<https://katex.org/docs/support_table.html>


# Markdown 方框

!!! summary
    "summary"
  
!!! example
    "!!! example"

!!! quote
    "!!! quote"

!!! tip
    "!!! tip"
  
!!! info
    "!!! info"
  
!!! success
    "!!! success"
  
!!! help
    "!!! help"

!!! attention
    "!!! attention"
  
!!! warning
    "!!! warning"
  
  
!!! fail
    "!!! fail"
  
!!! danger
    "!!! danger"
  
!!! error
    "!!! error"
  
!!! bug
    "!!! bug"