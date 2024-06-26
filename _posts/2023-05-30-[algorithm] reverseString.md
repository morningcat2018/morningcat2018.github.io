---
layout:     post
title:      旋转字符串
subtitle:   
date:       2023-05-30
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - algorithm
---

- [旋转字符串](https://github.com/julycoding/The-Art-Of-Programming-By-July-2nd/blob/master/ebook/zh/01.01.md)

## 把字符串前面的若干个字符移动到字符串的尾部

```java
/**
 * 给定一个字符串，要求把字符串前面的若干个字符移动到字符串的尾部，
 * 如把字符串“abcdef”前面的2个字符'a'和'b'移动到字符串的尾部，使得原字符串变成字符串“cdefab”。
 * 请写一个函数完成此功能，要求对长度为n的字符串操作的时间复杂度为 O(n)，空间复杂度为 O(1)。
 */
public class ReverseString {

    private static void reverseString(char[] s, int from, int to) {
        while (from < to) {
            char t = s[from];
            s[from++] = s[to];
            s[to--] = t;
        }
    }

    /**
     * 三步反转法:
     * 将一个字符串分成X和Y两个部分，在每部分字符串上定义反转操作，
     * 如X^T，即把X的所有字符反转（如，X="abc"，那么X^T="cba"），
     * 那么就得到下面的结论：(X^TY^T)^T=YX，显然就解决了字符串的反转问题。
     *
     * @param s 字符数组
     * @param n 字符数组长度
     * @param m 前多少字符转移到后面
     */
    private static void leftRotateString(char s[], int n, int m) {
        m %= n;                     //若要左移动大于n位，那么和%n 是等价的
        reverseString(s, 0, m - 1); //反转[0..m - 1]，套用到上面举的例子中，就是X->X^T，即 abc->cba
        reverseString(s, m, n - 1); //反转[m..n - 1]，例如Y->Y^T，即 def->fed
        reverseString(s, 0, n - 1); //反转[0..n - 1]，即如整个反转，(X^TY^T)^T=YX，即 cbafed->defabc。
    }

    public static void main(String[] args) {
        String str = "abcdef";
        char[] s = str.toCharArray();
        leftRotateString(s, s.length, 3);
        System.out.println(new String(s));
    }
}

```