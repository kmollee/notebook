# Benchmark

Go 語言不只有內建基本的 Testing 功能，另外也內建了 Benchmark 工具，讓開發者可以快速的驗證自己寫的程式碼效能如何, 透過內建工具可以知道程式碼單次執行多少時間，以及用了多少記憶體

### 如何寫 Benchmark

建立 `main_test.go` 檔案

```text
func BenchmarkPrintInt2String01(b *testing.B) {
    for i := 0; i < b.N; i++ {
        printInt2String01(100)
    }
}
```

檔案名稱一定要用 `_test.go` 當結尾 func 名稱開頭要用 Benchmark for 循環內要放置要測試的程式碼 `b.N` 是 go 語言內建提供的循環，根據一秒鐘的時間計算 跟測試不同的是帶入 `b *testing.B`參數

基本的 benchmark 測試也是透過 `go test` 指令，不同的是要加上 `-bench=.`，這樣才會跑 benchmark 部分，否則預設只有跑測試程式，大家可以看到 -4 代表目前的 CPU 核心數，也就是 `GOMAXPROCS` 的值，另外 -run 可以用在跑特定的測試函示，但是假設沒有指定 -run 時，你會看到預設跑測試 + benchmark，所以這邊補上 `-run=none` 的用意是不要跑任何測試，只有跑 benchmark



### Refer

* \[如何在 Go 語言內寫效能測試\]\([https://blog.wu-boy.com/2018/06/how-to-write-benchmark-in-go/](https://blog.wu-boy.com/2018/06/how-to-write-benchmark-in-go/)\)
* \[\[Golang\] 來玩玩Golang的效能評估-Benchmark\]\([https://www.evanlin.com/lets-go-benchmark/](https://www.evanlin.com/lets-go-benchmark/)\)







