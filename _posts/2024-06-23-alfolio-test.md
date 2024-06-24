---
layout: post
title: al-folio 主题功能测试
date: 2024-06-23
description: 使用和测试al-folio主题的所有功能
tags: 配置
categories: sample-posts
tabs: true
featured: true
typograms: true
pseudocode: true

toc:
  beginning: true
---





Hello World!

主题模版[演示](https://alshedivat.github.io/al-folio/)。



# 感觉有用的功能

* **置顶**：在文章信息部分添加`featured: true`
* **显示代码行数**：在`_comfig.yml`中修改`kramdown.syntax_highlighter_opts.block.line_numbers`为true。
* **在blog页面显示插图**：添加`thumbnail: <图片url>` （thumbnail 缩略图）
* 



## table of contents

**sidebar TOC：**

```
toc:
  sidebar: left
```

**beginning TOC：**

```
toc:
  beginning: true
```





## Tabs功能

<font color=red>注意：Tabs的命名好像和最终展示的tab数量有关系。比如定义了tabs code，其中包含c++, java, python3个tab，那么之后使用tabs code时也应该只使用这三个tab。</font>

[Tabs blog](https://alshedivat.github.io/al-folio/blog/2024/tabs/)

代码展示：

{% tabs code %}

{% tab code C++ %}

这是算法的C++版本代码

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

// 查找最长回文子串
string longestPalindromicSubstring(const string& s) {
    int n = s.length();
    vector<vector<bool>> dp(n, vector<bool>(n, false));
    int start = 0, maxLength = 1; // start记录最长回文子串的起始位置，maxLength记录长度

    for (int i = 0; i < n; ++i) {
        dp[i][i] = true; // 单个字符是回文
    }

    for (int i = 0; i < n - 1; ++i) {
        if (s[i] == s[i + 1]) {
            dp[i][i + 1] = true; // 相邻字符相同是回文
            start = i;
            maxLength = 2;
        }
    }

    for (int len = 3; len <= n; ++len) { // len是当前检查的子串长度
        for (int i = 0; i <= n - len; ++i) {
            int j = i + len - 1; // 子串结束位置
            if (s[i] == s[j] && dp[i + 1][j - 1]) {
                dp[i][j] = true;
                start = i;
                maxLength = len;
            }
        }
    }

    return s.substr(start, maxLength);
}

int main() {
    string s = "babad";
    cout << "Longest Palindromic Substring: " << longestPalindromicSubstring(s) << endl;
    return 0;
}
```

{% endtab %}

{% tab code java %}

这是算法的java版本代码



{% endtab %}

{% tab code python %}

这是算法的python版本代码

```python
def longest_palindromic_substring(s):
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    start = 0
    max_length = 1

    for i in range(n):
        dp[i][i] = True

    for i in range(n - 1):
        if s[i] == s[i + 1]:
            dp[i][i + 1] = True
            start = i
            max_length = 2

    for length in range(3, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j] and dp[i + 1][j - 1]:
                dp[i][j] = True
                start = i
                max_length = length

    return s[start:start + max_length]

if __name__ == "__main__":
    s = "babad"
    print("Longest Palindromic Substring:", longest_palindromic_substring(s))

```

除了写代码块之外，Tabs也可以写其他的内容

{% endtab %}

{% endtabs %}





第二个Tabs测试：

{% tabs code %}

{% tab code C++ %}

这是算法的C++版本代码

```cpp
int main() {
    string s = "babad";
    cout << "Longest Palindromic Substring: " << longestPalindromicSubstring(s) << endl;
    return 0;
}
```

{% endtab %}

{% tab code java %}

这是算法的java版本代码 dd



{% endtab %}

{% tab code python %}

这是算法的python版本代码

```python
if __name__ == "__main__":
    s = "babad"
    print("Longest Palindromic Substring:", longest_palindromic_substring(s))
```

除了写代码块之外，Tabs也可以写其他的内容

{% endtab %}

{% endtabs %}



## check list

```
- [x] Brush Teeth
- [ ] Put on socks
  - [x] Put on left sock
  - [ ] Put on right sock
- [x] Go to school
```

- [x] Brush Teeth
- [ ] Put on socks
  - [x] Put on left sock
  - [ ] Put on right sock
- [x] Go to school



---

# 感觉没什么用的功能

## typograms

[typograms blog](https://alshedivat.github.io/al-folio/blog/2024/typograms/)

<font color = red>使用typograms需要在文章信息部分添加`typograms: true`。</font>

```typograms
.------------------------.
|.----------------------.|
||"https://example.com" ||
|'----------------------'|
| ______________________ |
||                      ||
||   Welcome!           ||
||                      ||
||                      ||
||  .----------------.  ||
||  | username       |  ||
||  '----------------'  ||
||  .----------------.  ||
||  |"*******"       |  ||
||  '----------------'  ||
||                      ||
||  .----------------.  ||
||  |   "Sign-up"    |  ||
||  '----------------'  ||
||                      ||
|+----------------------+|
.------------------------.
```



## pseudo code（伪代码）

[pseudo code blog](https://alshedivat.github.io/al-folio/blog/2024/pseudocode/)

<font color = red>使用pseudo code需要在文章信息部分添加`pseudocode: true`。</font>

```pseudocode
% This quicksort algorithm is extracted from Chapter 7, Introduction to Algorithms (3rd edition)
\begin{algorithm}
\caption{Quicksort}
\begin{algorithmic}
\PROCEDURE{Quicksort}{$$A, p, r$$}
    \IF{$$p < r$$}
        \STATE $$q = $$ \CALL{Partition}{$$A, p, r$$}
        \STATE \CALL{Quicksort}{$$A, p, q - 1$$}
        \STATE \CALL{Quicksort}{$$A, q + 1, r$$}
    \ENDIF
\ENDPROCEDURE
\PROCEDURE{Partition}{$$A, p, r$$}
    \STATE $$x = A[r]$$
    \STATE $$i = p - 1$$
    \FOR{$$j = p$$ \TO $$r - 1$$}
        \IF{$$A[j] < x$$}
            \STATE $$i = i + 1$$
            \STATE exchange
            $$A[i]$$ with $$A[j]$$
        \ENDIF
        \STATE exchange $$A[i]$$ with $$A[r]$$
    \ENDFOR
\ENDPROCEDURE
\end{algorithmic}
\end{algorithm}
```



## a post with code diff

[code diff blog](https://alshedivat.github.io/al-folio/blog/2024/code-diff/)

功能就是像git一样，可以显示出两个文件的不同（修改）之处。



# 其他功能

* [引用](https://alshedivat.github.io/al-folio/blog/2024/post-citation/)
* [高级图像（轮播图、图像滑块）](https://alshedivat.github.io/al-folio/blog/2024/advanced-images/)
* [viga lite](https://alshedivat.github.io/al-folio/blog/2024/vega-lite/) 似乎是生成一些数据图
* [geojson](https://alshedivat.github.io/al-folio/blog/2024/geojson-map/)
* 
