---
title: "Markdown Basic Syntax"
date: 2023-02-01T18:10:41+08:00
categories:
- Markdown
tags:
- Markdown
draft: true
---

## 标题
| Markdown    | HTML                | Rendered Output |
| ----------- | ------------------- | --------------- |
| # 标题一    | \<h1\>标题一\<\/h1\> | <h1>标题一</h1>  |
| # 标题二    | \<h2\>标题二\<\/h2\> | <h2>标题二</h2>  |
| # 标题三    | \<h3\>标题三\<\/h3\> | <h3>标题三</h3>  |
| # 标题四    | \<h4\>标题四\<\/h4\> | <h4>标题四</h4>  |
| # 标题五    | \<h5\>标题五\<\/h5\> | <h5>标题五</h5>  |
| # 标题六    | \<h6\>标题六\<\/h6\> | <h6>标题六</h6>  |

## 段落
<pre>
Don't put tabs or spaces in front of your paragraphs.
 
Keep lines left-aligned like this.
</pre>

## 换行
<pre>
这是第一行.  
这是第二行
</pre>

这是第一行.  
这是第二行

<pre>
> Dorothy followed her through many of the beautiful rooms in her castle.
</pre>

> Dorothy followed her through many of the beautiful rooms in her castle.

<pre>
```cpp
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("Hello, Hugo");
    return 0;
}
```
</pre>

```cpp {linenos=table,linenostart=10,hl_lines=[3,"15-17"]}
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("Hello, Hugo");
    return 0;
}
```

{{< highlight cpp "linenos=table,linenostart=0,hl_lines=[3 15-17]" >}}
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("Hello, Hugo");
    return 0;
}
{{< / highlight >}}

~~doubletilde~~

<br/>
<br/>

**参考文档：**

[Markdown Basic Syntax](https://www.markdownguide.org/basic-syntax)