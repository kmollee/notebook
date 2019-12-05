# 軟體開發的那些真理

### 不要太在意 Code 行數

你可能聽到過很多有關`Code 行數`的瘋狂理論，但請不要把它們當真。基於代碼行數來做技術決策是一件很荒謬的事情。code 行數能夠爲我們提供的信息是很有限的。實際上，在大多數情況下，行數能夠爲我們提供的信息爲零。基於代碼行數來做技術決策無異於基於一本書的頁數來判斷書的質量。

有人認爲，項目的代碼越少就越容易讀懂，但這個觀點只說對了一部分。我認爲，具有可讀性的代碼應該具備以下這些特點：

* Code should be consistent 一致性
* Code should be self descriptive 自我描述
* Code should be well documented 良好的文檔
* Code should utilize stable modern features 使用了穩定新特性 e.g.  c++17
* Code shouldn't be unecessarily complex 不會太複雜
* Code shouldn't be un-performant \(don't write intentionally slow code\) 性能不會太差

### 不一定要把程式語言分出“好壞”

人們經常會這樣說：

> C 語言比某某語言好，因爲它的性能更好。
>
> Python 比某某語言好，因爲它更簡潔。
>
> Haskell 比某某語言好，因爲它是異類。

使用一句話來評判和比較一門編程語言是對語言本身的侮辱。它們是程式語言，可不是什麼口袋怪獸。

程式語言沒有好壞, 每個語言都有獨特的優勢, 你要做的就像是在工具箱內找到對應的工具完成你的工作!

程式語言之間確實存在差別，而且很少存在“沒有用”的編程語言（除了那些過時或者已經死掉的語言）。每一門程式語言都在某些方面做出了權衡，它們就像工具箱裏的工具。螺絲起子可以做錘子做不到的事情，但你能說起子比錘子更好嗎？

程式語言的評估

* Density of available online resources \(StackOverflow density\) 是不是有很多相關資源\(stackoverflow 討論程度\)
* Development Velocity \(vroom vroom\) 開發速度
* Bug proneness \(eeek\) 出現 bug 的機率
* Quality and breadth of package ecosystem \(yea npm, it says quality\) 庫生態系統的質量和廣度
* Performance characteristics \(more dots\) 性能
* Hirability \(sorry COBOL\) 好不好招人



### 閱讀別人的 Code 是個麻煩事

閱讀別人的代碼其實並非易事。Robert Martin 在《Clean code》裏提到過這個問題：

> 事實上，人們花在閱讀代碼和花在寫代碼上的時間比率超過了 10 比 1。閱讀舊代碼是寫新代碼的一個組成部分……所以，容易讀懂的代碼會讓寫新代碼的工作變得更容易些。

有很長一段時間，我被閱讀別人的代碼這件事所困擾。後來發現，其實有很多人都跟我一樣，每天都要面對這個問題。閱讀別人的代碼就像是在閱讀一本用外文寫的書，即使代碼是用你熟悉的語言寫的，代碼的風格和架構仍然會不一樣。這個問題不太好解決，不過我發現下面這些做法可能會對你有所幫助。

評審別人的代碼有助於提升閱讀代碼的能力。在過去兩年中，我評審了很多 GitHub PR\(pull request\)。每評審完一個 PR，我就越能夠適應閱讀別人的代碼。GitHub PR 很適合用來提升代碼閱讀能力，因爲：

* 隨時都可以評審，只要找到你想加入的開源項目；
* 在劃定的範圍內進行練習（一個功能、一個 bug）；
* 需要專注細節，迫使你仔細查看每一行代碼。

第二種方式有點特別，這也是我一直在踐行的，可以幫我節省很多時間。在瞭解了某個項目的代碼風格之後，就用這種風格來寫代碼，這樣可以提升閱讀這種風格代碼的能力。因爲你已經體驗過類似的風格，所以再去閱讀這樣的代碼就不會感到陌生。





