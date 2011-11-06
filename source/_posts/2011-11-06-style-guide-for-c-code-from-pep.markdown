---
layout: post
title: "Style Guide for C Code"
date: 2009-05-07 20:57
comments: true
categories: 
- programming
- python
- C and C++
- translation
---
這份文件的原始出處為 Guido van Rossum 所寫的《Style Guide for C Code》，發表於 [Python Developers Guide](http://www.python.org/dev/) 的 [Python Enhancement Proposals（PEP）](http://www.python.org/dev/peps/)之中，編號為 [PEP 7](http://www.python.org/dev/peps/pep-0007/)。

這份文件主要的內容，一如其 Introduction 一章所言，為統一實作 Python 的程式設計師的寫碼風格而著。對於一般的 C/C++ 程式設計師來說，這份文件也有助於提升程式碼的可讀性。

（可惡，天氣太熱，跑數據打電動都會熱當，閒來無事只好來翻這個東西…）

* * * * *

Introduction 簡介
===============

這份文件設定了一些寫碼慣例，供實作 Python 的設計師們作為參考。在閱讀本文件的同時，請同時參閱 PEP describing style guidelines for Python code [1][1] 一文。

注意，規則存在的意義就在於讓人打破它。至於何時打破這份規則，這邊有兩個不錯的時機：

1. 縱使原始碼的閱讀者亦慣於遵循這份規則，但當遵循規則反而會破壞原始碼的可讀性時，也不要用這份習慣來寫這段程式。
1. 基於歷史遺毒，假若附近的程式碼都不遵循這份規則，那麼新加入的程式碼也不要遵循它──即使這有可能會是一個修正歷史的契機，也不要。

C dialect C 語言的方言
=================

- 使用 ANSI/ISO C 的標準（這邊指的是 1989 年的版本，也就是 C89）。此規則意味著，吾人應該在任何程式區段的開頭宣告所有此區段所需之變數（不必一定得在函式的開頭全部宣告完）。
譯註：這邊指的區段（block）指的是一組大括號（``{}``）所涵蓋之範圍。

- 不要使用 GCC 的附加功能（例如，以多行撰寫同一字串內容時，直接斷行而不加上反斜線跳脫換行字元）。
- 所有的函式宣告、定義之前一定要詳細地把前置宣告寫清楚。意即，在前置宣告中，要明確地標明各參數型態。
- 不要使用 C++ 的單行註解語法（``//``）。（譯註：這條規定與 Scott Meyers 所著的 Effective C++ 之建議洽洽相反──該書內容建議使用新式的單行註解，以避免在開發時的某些註解不對稱問題。（Second Edition, Item 4））
- 不要太依賴某種主流編譯器的特殊功能（gcc、VC++ 等等）。

Code lay-out 程式碼排版
==================

- 如果這份程式碼的前撰寫者已經選用 tab 作為縮排字元，那麼就跟著使用tab，每一層縮排使用一個 tab，而一個 tab 應為 8 個空白字元。如果是自己產生的新程式碼文件，那麼就不要用任何 tab 作為縮排，改用 4 個空白字元來縮排。在某些開發環境下，可能會自動採用 4 個空白字元的縮排法。
- 任何一行程式碼最長應只有 79 個半形字元長。如果你遵循這份寫碼規則，但發現某些程式碼塞不進這樣的長度規定內，那麼你該檢討一下這行敘述的是否太過於複雜了──請考慮增加幾個函式來分攤它的功能。
- 任何一行程式碼都不應以空白字元結尾。如果你認為非得這樣做不可，請再考慮一下──因為某個衰仔下次開啟這份文件的時候，它的編輯器可能自動刪除這些空白字元。
- 宣告函式的風格：函式名稱靠左，包住函式的大括號靠左，區域變數宣告敘述告一段落時留下一個空白行。

{% codeblock lang:c %}
static int
extra_ivars(PyTypeObject *type, PyTypeObject *base)
{
    int t_size = PyType_BASICSIZE(type);
    int b_size = PyType_BASICSIZE(base);
    assert(t_size &amp;gt;= b_size); /* type smaller than base! */
    ...
    return 1;
}
{% endcodeblock %}

- 程式碼結構：在諸如 ``if``、``for`` 之類的關鍵字與其後的左括號之間留下一個空白；在左右括號的內緣不要加入空白；請參考下列範例：

{% codeblock lang:c %}
if (mro != NULL) {
    ...
}
else {
    ...
}
{% endcodeblock %}

- 關於 ``return`` 敘述「不應該」附加多餘的括號：

{% codeblock lang:c %}

    return Py_None; /* 正確的寫法 */
    return(Py_None); /* 錯誤的寫法 */

{% endcodeblock %}

- 函式和巨集的呼叫風格：``foo(a, b, c)`` ──函式名與左括號之間不要加上空白，左右括號的內緣不要加上空白，逗號前面不要有空白，逗號後面留一個空白。
- 在賦值、布林和比較運算子的前後都加上空白。如果在某敘述中需要用到一大堆此類運算子（低優先權運算子）的話，在這些運算子的前後都加上空白。
- 長敘述斷行：可能的話，在一串參數列中，以逗點作為斷行點。把斷行後的每一行都加上相同長度的縮排，例如：

{% codeblock lang:c %}

    PyErr_Format(PyExc_TypeError,
            "cannot create '%.100s' instances",
            type->tp_name);

{% endcodeblock %}

- 如果你欲將一長串的比較運算敘述斷行，則以布林運算子作為斷行點，例如：

{% codeblock lang:c %}
if (type->tp_dictoffset != 0 && base->tp_dictoffset == 0 &&
    type->tp_dictoffset == b_size &&
    (size_t)t_size == b_size + sizeof(PyObject *))
    return 0; /* "Forgive" adding a __dict__ only */
{% endcodeblock %}

- 在每個函式、結構宣告以及主要段落的前後都加上一空白行。
- 在一段程式碼之前加上註解。
- 除非某個函式或全域變數會是整個程式的公開對外介面，否則一律宣告成 ``static``。
- 對於來自外部宣告的函式與變數，則總是在標頭檔資料夾內存在相對應的標頭檔，並以 ``DL_IMPORT()`` 此一巨集宣告之，像是這樣：

{% codeblock lang:c %}
extern DL_IMPORT(PyObject *) PyObject_Repr(PyObject *);
{% endcodeblock %}

Naming conventions 命名法則
=======================

- 公開對外的介面函式名稱一律使用 ``Py`` 作前綴；內部的 ``static`` 函式則完全不要這樣做。``Py_`` 這樣的前綴則留給全域 service routine 使用；特定 routine 群組（例如針對某種特定物件的 API 等）則使用較長前綴，像是專門處理字串的函式們以 ``PyString_`` 前綴。
- 公開的函式與變數們使用 MixedCase 命名法，並在底線字元連接其名稱與前綴，像是：``PyObject_GetAttr``、``Py_BuildValue``、``PyExc_TypeError``。
- 偶爾「內部的」函式可能會需要被外部取用；這時就使用 ``_Py`` 作為前綴，例如：``_PyObject_Dump``。
- 巨集的前綴使用 MixedCase 命名，自身名稱則全大寫，舉例來說：``PyString_AS_STRING``、``Py_PRINT_RAW``。

Documentation Strings 文件字串
==========================

- 如要使用說明字串，則以 ``PyDoc_STR()`` 或 ``PyDoc_STRVAR()`` 巨集來取用，這是為了支援無說明字串選項。如果這份 C 程式碼需要支援 Python 2.3 以前的版本，則請在援引 Python.h 之後使用以下巨集：

{% codeblock lang:c %}
#ifndef PyDoc_STR
#define PyDoc_VAR(name)         static char name[]
#define PyDoc_STR(str)          (str)
#define PyDoc_STRVAR(name, str) PyDoc_VAR(name) = PyDoc_STR(str)
#endif
{% endcodeblock %}

- 各函式的說明字串首行必須要是「簽證行」，指出關於本函式的概要，包括參數及返回值。

{% codeblock lang:c %}
PyDoc_STRVAR(myfunction__doc__,
        "myfunction(name, value) -> bool\n\n\
        Determine whether name and value make a valid pair.");
{% endcodeblock %}

在簽證行與敘述之間總留下一空白行。

- 如果函式返回值總是 ``None``（意即返回值不具任何意義），則不要指定任何返回型態。

如果要使用多行說明字串，則記得在換行時使用反斜線跳脫換行字元，不然請使用多個字串串接的方式撰寫。

{% codeblock lang:c %}
PyDoc_STRVAR(myfunction__doc__,
        "myfunction(name, value) -> bool\n\n"
        "Determine whether name and value make a valid pair.");
{% endcodeblock %}

即使所使用的 C 編譯器允許以非串接形式多行描述同一字串，也不要這樣寫：

{% codeblock lang:c %}
/* 不好的寫法：不要這樣寫！ */    /* 譯註：注意箭頭所指位置！ */
PyDoc_STRVAR(myfunction__doc__,        ↙
        "myfunction(name, value) -> bool\n\n
        Determine whether name and value make a valid pair.");
{% endcodeblock %}

當然，並不是所有的編譯器都允許不好的寫法；像是 MSVC 編譯器，如果你給它這樣的程式碼，它會向你報怨。

References 參考資料
===============

- [PEP 8, Style Guide for Python Code][1], van Rossum, Warsaw

[1]: http://www.python.org/dev/peps/pep-0008/

Copyright 版權聲明
==============

本文張貼於公共空間。

