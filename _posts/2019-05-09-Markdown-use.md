---
layout:     post
title:      Markdown的基本使用
subtitle:   Markdonw的基本语法与范例
date:       2019-05-09
author:     Wingin Cheung
header-img: img/markdown-write.jpg
catalog: true
mermaid: true
tags:
    - Markdown
---

# Markdown基本使用

Markdown是一种纯文本格式的标记语言。通过简单的标记语法，它可以使普通文本内容具有一定的格式。

##  1、标题


在文字前面加上"\#"代表标题。几个"\#"代表几级标题，如"\#"代表一级标题，"\#\#"代表二级标题。如：

```markdown
# 一级标题
## 二级标题
### 三级标题
```

效果如下：

> # 一级标题
>
> ## 二级标题
>
> ### 三级标题

*Tips：建议在"\#"与标题文字间添加空格*。

## 2、字体

+ 斜体

  在需要倾斜的文字左右分别添加"\* "号即可，如：

  ```
  *斜体*
  ```

  效果如下：

  > *斜体*

+ 加粗

  在需要加粗的文字左右分别添加两个"\*"号即可，如：

  ```
  **文字加粗**
  ```

  效果如下：

  > **文字加粗**

+ 斜体加粗

  在需要斜体加粗的文字左右分别添加三个"\*"号即可，如：

  ```
  ***斜体加粗***
  ```

  效果如下：

  > ***斜体加粗***

+ 删除线

  在需要添加删除线的文字左右分别添加两个"\~"即可，如：

  ```
  ~~删除文字~~
  ```

  效果如下：

  >~~删除文字~~

## 3、引用

在需要引用的文字前加"\>"即可，可嵌套。如：

```
> 引用嵌套一
>> 引用嵌套二
>>> 引用嵌套三
```

效果如下：

>> 引用嵌套一
>> > 引用嵌套二
>> >
>> > > 引用嵌套三

## 4、分割线

在空白一行用三个或者三个以上"\-"或者"\*"表示分割线，如：

```
分割内容一
---
分割内容二
***
分割内容三
```

效果如下：

>分割内容一
>
>---
>
>分割内容二
>
>***
>分割内容三

## 5、下划线

Markdown并无下划线的原生语法，会与超链接的默认样式产生混淆。但，Markdown语法可兼容HTML语法，所以我们可以借助HTML语法实现下划线：

```
<u>下划线内容</u>
```

效果如下：

><u>下划线内容</u>

*Tips："u"是underline的意思。*

## 6、插入图片

插入图片语法如下：

```
![Atl标签](图片地址 "optional titile悬浮显示文字")
```

如：

```
![Typora.ico](https://dl2.macupdate.com/images/icons256/52992.png?d=1479658505 "Typora Icon")
```

效果如下：

> ![Typora.ico](https://dl2.macupdate.com/images/icons256/52992.png?d=1479658505 ``Typora Icon``)

## 7、超链接

设置超链接语法如下：

```
[需设置超链接的文字](超链接地址 "超链接标题")
```

如：

```
[Markdown中文教程](http://www.markdown.cn "Markdown主页")
```

效果如下：

> [Markdown中文教程](http://www.markdown.cn "Markdown主页")

*Tips：Markdown本身语法不支持在新页面中打开链接。如果实现此操作的话，可用html语言实现*：

```
<a href = "超链接地址" target="_blank">需设置超链接的文字</a>
```

如：

```
<a href = "http://www.markdown.cn" target="_blank">Markdown中文教程</a>
```

效果如下：

> <a href ="http://www.markdown.cn" target="_blank">Markdown中文教程</a>

## 8、列表

 + 无序列表

   在文字前面加"\-"、"\+"、"\*"这三者之一均可实现，**需注意的是，在这三种符号之一的符号与列表内容间，需加一个空格**。如：

   ```
   - 列表内容
   + 列表内容
   * 列表内容
   ```

   效果如下：

   >- 列表内容
   >+ 列表内容
   >
   >* 列表内容

 + 有序列表

   实现有序列表，可使用数字后加".“实现，**序号与列表内容间要有空格**。如：

   ```
   1. 列表内容
   2. 列表内容
   3. 列表内容
   ```

   效果如下：

   >1. 列表内容
   >2. 列表内容
   >3. 列表内容

 + 列表嵌套

   在上一级与下一级之间输入三个空格即可，如：

   ```
   + 一级无序列表内容
     
      * 二级无序列表内容
      * 二级无序列表内容
      * 二级无序列表内容
      
         - 三级无序列表内容
         - 三级无序列表内容
         - 三级无序列表内容
      
   + 一级无序列表内容
      
      1. 二级有序列表内容
      2. 二级有序列表内容
      3. 二级有序列表内容
   
   1. 一级有序列表内容
      
       - 二级无序列表内容
       - 二级无序列表内容
       - 二级无序列表内容
      
   2. 一级有序列表内容
      
       1. 二级有序列表内容
       2. 二级有序列表内容
       3. 二级有序列表内容
   ```

   效果如下：

   >+ 一级无序列表内容
   >  
   >   * 二级无序列表内容
   >   * 二级无序列表内容
   >   * 二级无序列表内容
   >   
   >      - 三级无序列表内容
   >      - 三级无序列表内容
   >      - 三级无序列表内容
   >   
   >+ 一级无序列表内容
   >   
   >   1. 二级有序列表内容
   >   2. 二级有序列表内容
   >   3. 二级有序列表内容
   >
   >1. 一级有序列表内容
   >   
   >    - 二级无序列表内容
   >    - 二级无序列表内容
   >    - 二级无序列表内容
   >   
   >2. 一级有序列表内容
   >   
   >    1. 二级有序列表内容
   >    2. 二级有序列表内容
   >    3. 二级有序列表内容

## 9、表格

表格用"\|"分割各列的内容，用"\-"分割上下两行。

表格内内容文字默认居左，可在文字右侧加"\:"代表文字居右、在文字两侧均加"\:"代表文字居中。

如：

```
序号|姓名|分数|备注
:-:|:-|:-:|-:
1|张三|100|无
2|李四|99|无
3|刘五|98|\-
```

效果如下：

>序号|姓名|分数|备注
>:-:|:-|:-:|-:
>1|张三|100|无
>2|李四|99|无
>3|刘五|98|\-

## 10、代码

 * 单行代码： 代码左右分别加"`"（反引号），如

   ```
   `这是单行代码`
   ```

   效果如下：

   >`这是单行代码`

 * 代码块：代码块前后一行均加三个"`"（反引号），如：

   ```markdown
   ​```c
   
   #include <stdio.h>
   
   int main(int argc, char **argv){
       printf("Hellow World.\n");
       return 0;
   }
   ​```
   ```
   
   效果如下：
   
   ```c
   #include <stdio.h>
   
   
   int main(int argc, char **argv){
       printf("Hellow World.\n");
       return 0;
   }
   ```



## 11、流程图

流程图大致分为两段，第一段定义元素，第二段定义元素间的走向。

1. 定义元素的语法：

    ```markdown
    tag => type: content:>url
    ```

    + tag是元素名字
    + type是元素类型，包括以下几种：
        * start #开始
        * end #结束
        * operation #操作
        * subroutine #子程序
        * condition # 条件
        * inputoutput #输入或者输出
    + content是框图中需要填写的内容
    + url是个链接，与框图中的文本相绑定

2. 连接元素的语法：

    用"->"来连接两个元素。如果是condition类型，有yes和no两个分支，均需要明确指出：

    ```markdown
    cond(yes)->op_yes
    cond(no)->op_no
    ```

    下面，以经典的求1+2+3+…+100的和为例，画出对应的流程图。

    *Tips：以下代码可在Typora中显示流程图，在jekyll中无法正常显示*

    ***请去掉首行````flow \\`中的" \\"***
    
    ```markdown
    ​```flow \
    st=>start: start
    op1=>operation: int i = 0, sum = 0
    op2=>operation: sum+=i
    op3=>operation: i++
    cond=>condition: i>100?
    op4=>operation: printf(sum)
    e=>end: end
    
    st->op1->op2->op3->cond
    cond(yes)->op4->e
    cond(no)->op2
​```
    ```

    *Tips：以下代码用于在jekyll中结合mermaid显示流程图*
    
    ***请去掉首行````html \\`中的" \\"***
    
    ```markdown
    ​```html \
    <div class="mermaid">
    graph TD;
        st(start);
        op1[int i = 0, sum = 0];
        op2[sum+=i];
        op3[i++];
        cond{i>100?};
        op4[printf&#40sum&#41];
        e(end);
        st-->op1;
        op1-->op2;
        op2-->op3;
        op3-->cond;
        cond-->|yes|op4;
        cond-->|no|op2;
    op4-->e;
    </div>
​```
    ```
    
    效果如下：
    
    > <div class="mermaid">
    > graph TD;
    >     st(start);
    >     op1[int i = 0, sum = 0];
    >     op2[sum+=i];
    >     op3[i++];
    >     cond{i>100?};
    >     op4[printf&#40sum&#41];
    >     e(end);
    >     st-->op1;
    >     op1-->op2;
    >     op2-->op3;
    >     op3-->cond;
>     cond-->|yes|op4;
    >     cond-->|no|op2;
>     op4-->e;
    > </div>

    对应的c语言代码为：
    
    ```c
    #include <stdio.h>


​    
    int main(int argc, char **argv){
        int i = 0, sum = 0;
    
        do{
            sum += i;
            i++;
        }while(!(i > 100 ));
    
        printf("sum = %d\n", sum);
    
        return 0;
    }
    ```



## 12、目录

Markdown中可使用"[TOC]"单独一行中插入目录:

```
[TOC]
```
*Tips：在jekyll中，需使用以下语句插入目录，此语法也可用于Typora中：*

```
* toc
{:toc}
```

如在下面插入本文的目录列表，效果如下：

> * toc
{:toc}
