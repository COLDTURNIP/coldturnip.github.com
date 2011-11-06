---
layout: post
title: "關於例外處理的一則小故事"
date: 2005-07-12 18:59
comments: true
categories: 
- C and C++
- history
- programming
---
{% blockquote Steve Oualline , C++ 的風格與藝術 (Practical C++ Programming 黃吉霈譯) %}
1996 年 6 月 4 日，一個電腦程式啟動了。30 秒過後，assert 敘述失敗並且關閉系統。儘管在多數情況下關閉系統是最安全的，但是這次卻是一件壞事。

這是因為該電腦是在 Ariane 5 火箭的頂部，其任務是保持火箭的正確方向。

結果顯然是系統當機（火箭在著地之前爆炸）。

應該注意，在 99.99% 的情況下，異常終止程式是正確的。只有極少數的程式應像 Ariance 5 的導航軟體，無論如何都得繼續執行。
{% endblockquote %}
