---
layout: post
title: "Go 記憶體空間模型"
date: 2011-04-24 22:43
comments: true
categories: 
- programming
- golang
---
譯序
==

本文譯自 [The Go Memory Model](http://golang.org/doc/go_mem.html)，翻譯時原文刊載於 [Go Programming Language Official Website](http://golang.org/)，受 [Creative Commons Attribution 3.0 License](http://creativecommons.org/licenses/by/3.0/) 保護。

本繁體中文版翻譯時參考了[柴树杉與 Bian Jiang 合譯的簡體中文版本](http://bbs.golang-china.org/viewtopic.php?f=4&t=24)，該文刊載於 [Go 语言中文论坛](http://bbs.golang-china.org/)。

曾經有人問為何會翻這篇文章，其實在翻譯當時我只想知道 Go 的執行期 memory layout 長成什麼樣子，但顯然這篇主要討論的方向不在此，於是一切都只是一場美麗的誤會 :P

Introduction – 簡介
=================

本《Go 記憶體空間模型》一文所描述的內容為，針對同一變數，在一個 goroutine 上對該變數進行寫入，如何得以保證在另一 goroutine 上得以正確讀取前述寫入值。

<a name="happens_before"></a>Happens Before – 在…之前
=====================

就單一 goroutine 的執行，只保證所有的讀／寫行為就結果上應等同於其原始碼設計；也就是說，如果表現的行為不變，編譯器和處理器得以任意排列同一 goroutine 內的所有記憶體讀／寫順序。也因為允許如此重新排序，即使執行的內容相同，每個不同的 goroutine 上彼此看見對方的執行順序都可能有所不同。舉例來說，如果一個 goroutine 的程式碼寫著 ``a = 1; b = 2;``，實際執行時在另一個 goroutine 上回頭一望，可能會看到 ``b`` 的值先被更新，而後 ``a`` 的值才被改變。

以下為了描述程式對於讀／寫記憶體的請求，對於部份程式區段的記憶體操作，我們先定義何為「在…前發生」。*如果事件 e1 在事件 e2 之前發生，那麼我們也可以說 e2 在 e1 之後發生。同理，如果 e1 不在 e2 之前發生，且亦不在 e2 之後發生，那麼我們說 e1 與 e2 同時發生。*

在單一的 goroutine 內，記憶體操作的順序是與原始碼相同的。

在以下條件下，針對變數 v 的讀取事件 r，可以取得同對變數 v 的寫入事件 w 的內容：

1. w 在 r 之前發生。
1. 在 w 之後、 r 之前的時間區間內，沒有其它對於 v 的寫入事件 w’ 發生。

為了保證一項對於變數 v 的讀取事件 r 不但*可以*感知寫入事件 w、且 r 實際讀取的值即為 w 所寫入的值，我們還必須保證 w 是唯一一個可以被 r 感知的寫入事件：

1. w 在 r 之前發生。
1. 所有其它對於共享變數 v 的寫入事件都發生在 w 之前或 r 之後。

後述這組條件較前述一組來得強健；它進一步限制其它寫入事件不得與 w 或 r 同時發生。

在單一 goroutine 內，沒有任何事件可以同時發生，所以此二條款等價於以下敘述：同對於變數 v，一個讀取事件 r 將讀取最近一次寫入事件 w 之結果。而在多 goroutine 同對共享變數 v 進行操作時，必須利用一些同步操作才得以建立前述的有序環境，保證對於變數的讀取事件皆對讀到相應的寫入事件。

對於變數 v 的零值初始化動作，在本記憶體模型內為一寫入事件。

如果讀／寫事件的內容值比單機器字長（machine word）來得長，那麼視為多次、與機器字長等長的不定順序讀／寫操作。

Synchronization – 同步
====================

Initialization – 初始化
--------------------

整個程式的所有初始化行為都在單一 goroutine 內進行，而其中產生的新的 goroutine 會在整個初始化結束之後才真正開始運行。

如果一個 package **p** 引用了另一個 package **q**，則 **q** 的所有 `init` 函式將在 **p** 的任何 `init` 函式開始前結束。

主函式 `main.main` 在所有 `init` 函式結束後才開始執行。

在某一 `init` 函式內的 goroutine 創建，會等到所有 `init` 函式全部結束後才真正開始創建。

Goroutine creation – 創建 Goroutine
---------------------------------

go 描述式會在 goroutine 內的程序開始執行之前創建新的 goroutine。

舉例而言，在下列的程式中：

{% codeblock lang:go %}
var a string

func f() {
	print(a)
}

func hello() {
	a = "哈囉，世界"
	go f()
}
{% endcodeblock %}

呼叫 `hello` 會在稍後時間點印出`「哈囉，世界」`（甚至可能在 `hello` 返回之後）

Channel communication – Channel 通訊
----------------------------------

channel 通訊是 goroutine 之間主要的同步手段。對等定的通道的所有發送動作都一定要有相應的接收，而這一送一收的動作通常發生在兩個不同的 goroutine 上。

對一個 channel 的發送行為一定會在相應的接收行為完成之前完成。

參考以下程式碼：

{% codeblock lang:go %}
var c = make(chan int, 10)
var a string

func f() {
	a = "哈囉，世界"
	c <- 0
}

func main() {
	go f()
	<-c
	print(a)
}
{% endcodeblock %}

上述程式將被保證印出「`哈囉，世界`」。這是因為，對於 `a` 的寫入行為在對 `c` 發送訊息之前完成，而對 `c` 的發送行為必在其接收行為完成之前發生，又接收行為必在 `print` 之前。

對一不帶緩衝區的 channel，接收行為必在發送完成之前發生。

參考以下程式碼（與前例相似，但將送收角色對換、且改採無緩衝 channel）：

{% codeblock lang:go %}
var c = make(chan int)
var a string

func f() {
	a = "哈囉，世界"
	<-c
}

func main() {
	go f()
	c <- 0
	print(a)
}
{% endcodeblock %}

上述程式亦可保證印出「`哈囉，世界`」。對 `a` 的寫入行為早於對 `c` 的接收行為，而此接收行為必在發送行為完成之前開始，而發送行為又早於 `print`。

如果使用帶緩衝的 channel（意即宣告 `c = make(chan int, 1)`），則本例無法保證可以正確印出「`哈囉，世界`」。（它可能頂多印出空字串；它不會印出「`再見，宇宙`」或是讓程式崩潰。）

Locks – 同步鎖
-----------

`sync` 這個 package 實作了兩種類型的同步鎖，`sync.Mutex` 與 `sync.RWMutex`。

對於任一 `sync.Mutex` 或 `sync.RWMutex` 變數 `l`，且 n < m，則第 n 次叫用 `l.Unlock()` 必在第 m 次的 `l.Lock()` 返回之前發生。

參考以下程式碼：

{% codeblock lang:go %}
var l sync.Mutex
var a string

func f() {
	a = "哈囉，世界"
	l.Unlock()
}

func main() {
	l.Lock()
	go f()
	l.Lock()
	print(a)
}
{% endcodeblock %}

此程式可保證印出「`哈囉，世界`」。第一次呼叫 `l.Unlock()`（位於 f 內）必在第二次 `l.Lock()`（位於 `main`）返回之前，意即於 `print` 之前呼叫。

對於任何 `sync.RWMutex` 變數 `l`，若在第 n 次呼叫 `l.Unlock` 之後調用 `l.RLock`，則其相應的 `l.RUnlock` 必發生於第 n+1 次呼叫 `l.Lock` 之前。

Once – 單次執行
-----------

`sync` package 保證多 goroutine 間的得以安全地進行初始化，提供一個變數型態 `Once` 以控制之。針對特定函式 `f`，多個執行緒可以同時調用 `once.Do(f)` 試圖執行之，但 `f()` 只會被實際執行一次，且在被實際執行的 `f()` 返回前，其它的 `once.Do(f)` 調用者都會被鎖住。

利用 `once.Do(f)` 呼叫的 `f()` 必在任何 `once.Do(f)` 返回前單次執行並返回。

參考以下程式碼：

{% codeblock lang:go %}
var a string
var once sync.Once

func setup() {
	a = "哈囉，世界"
}

func doprint() {
	once.Do(setup)
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
{% endcodeblock %}

呼叫 `twoprint` 會印出兩次「`哈囉，世界`」。只有第一次呼叫 `twoprint` 會執行 `setup`。

Incorrect synchronization – 錯誤的同步方式
-----------------------------------

注意，如果讀取事件 r 和寫入事件 w 同時發生，r 或許能夠讀取 w 寫入的值；即便如此，這項事實並不意味著在 r 之後的讀取事件可以讀到 w 之前發生的寫入事件的值。

參考以下程式碼：

{% codeblock lang:go %}
var a, b int

func f() {
	a = 1
	b = 2
}

func g() {
	print(b)
	print(a)
}

func main() {
	go f()
	g()
}
{% endcodeblock %}

此例中的 `g` 有可能先印出 `2` 再印出 `0`。（譯註：參考[《Happens Before》一節](#happens_before)，「如果表現的行為不變，編譯器和處理器得以任意排列同一 goroutine 內的所有記憶體讀／寫順序」，所以函式 `f` 可能被重排成 `b = 2; a = 1`）

這個特性會使得一些過去常用的同步做法失效。

二次檢查鎖過去被用來迴避同步所造成的系統負擔。舉例來說，前例的 `twoprint` 函式可能被錯誤地寫成：

{% codeblock lang:go %}
var a string
var done bool

func setup() {
	a = "哈囉，世界"
	done = true
}

func doprint() {
	if !done {
		once.Do(setup)
	}
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
{% endcodeblock %}

但是這樣的同步機制現在無效了。理想上，在 `doprint` 裡，籍由觀測 `done` 是否被寫入值意味著 `a` 是否完成寫入；實際上，「`哈囉，世界`」可能不被印出，取而代之的是空字串。

另外一個過去常用的同步手段是忙碌鎖，像是這樣：

{% codeblock lang:go %}
var a string
var done bool

func setup() {
	a = "哈囉，世界"
	done = true
}

func main() {
	go setup()
	for !done {
	}
	print(a)
}
{% endcodeblock %}

一如前例，在 `main` 中觀查 `done` 是否被寫入，無法保證 `a` 是否被賦值，所以同理可能印出空字串。更糟的是，因為在兩個執行緒之間並沒有明確的同步事件，所以就連 `done` 的寫入事件也無法保證能被 `main` 所觀察，因此有機會在 `main` 造成無窮迴圈。

在此還有個微妙的變種，參見以下程式：

{% codeblock lang:go %}
type T struct {
	msg string
}

var g *T

func setup() {
	t := new(T)
	t.msg = "哈囉，世界"
	g = t
}

func main() {
	go setup()
	for g == nil {
	}
	print(g.msg)
}
{% endcodeblock %}

即便 `main` 觀察到 `g != nil` 發生而決定是否離開迴圈，`g.msg` 也不一定已經完成它的初始化。（譯註：因為 `setup` 可能被重排成 `g = new(T); g.msg = "哈囉，世界"`，因此 `g != nil` 時 `g.msg` 可能尚未被賦值。）

綜觀以上數例，其解決方案始終如一：以明確的方式進行同步。
