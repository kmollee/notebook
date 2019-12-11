# Linux下你還知道這些特殊文件？

我們都知道Linux下一切皆文件，主要有

* - 普通文件
* d    目錄
* l 符號鏈接
* s    套接字
* b    塊設備
* c    字符設備
* p    管道

這麼幾種文件。  
這裡的前綴字符可以通過ls命令觀察到：

```text
$ ls -l test.log
-rw-r--r-- 1 root root 33 Nov 17 17:03 test.log
```

它的結果最前面是-，因此它是普通文件。

```text
$ ls -al /dev/null
crw-rw-rw- 1 root root 1, 3 Sep 11 20:33 /dev/null
```

它的結果最前面是c，因此它是字符設備。文件簡單介紹幾種字符設備文件，它能在我們功能測試的時候提供很好的幫助。

### /dev/null

/dev/null 可無限接收數據，你可以認為是一個黑洞，因此如果我們需要丟棄某些終端輸出，可以重定向到這裡：

```text
$ echo "shouwangxiansheng" > /dev/null
```

所以如果你有不需要的數據可以盡情的往這裡寫。

### /dev/full

它在讀取時會讀取到連續的NUL（零值）字節流，而在寫入的時候，會返回磁盤空間已滿的結果，後者在測試你的程序的時候會有幫助，即測試磁盤滿的場景：

```text
$ echo "bianchengzhuji" > /dev/full
-bash: echo: write error: No space left on device
```

### /dev/zero

和/dev/null類似，向其中寫入時會丟棄所有數據，但是讀取時，會產生NUL（零值）字節流。

```text
$ cat /dev/zero |od -x 
0000000 0000 0000 0000 0000 0000 0000 0000 0000
```

### /dev/random

/dev/random可以提供隨機數據流，它保證數據的隨機性，但是讀取時會造成等待，例如

```text
$ cat /dev/random | od -x
0000000 2b07 daac 42f4 e1fd fb62 2098 870e e0af
0000020 3022 2099 e5da 4e1c d6db 548b a979 1217
0000040 3777 bb6a 957d 1279 ab29 e8a4 6a36 ecca
0000060 39ec 2285 126c 30ea ea67 1526 5e4a 2dd9
```

稍過會才會出現數據，為了便於查看，我們利用od命令查看其十六進制內容。

### /dev/urandom

從名字就可以看出來，是用來產生隨機數據的。它的產生速度很快，但是數據的隨機性不如/dev/random

```text
cat /dev/urandom | od -x
0547560 f43e 696a 8936 2b27 36c8 4446 2802 1d47
0547600 b8af 249d aae9 edbf 8971 b1d1 0c73 3e2d
0547620 237b 9a81 6348 cb2a 1972 4486 028a 3573
0547640 1690 c388 64e1 aec1 d5f4 1964 bbb9 192f
0547660 f242 7194 51ba 62a3 fc13 ff53 fb50 e3d8
0547700 ef32 3658 b335 75ee 62de 4096 6468 c979
0547720 01b9 c233 878d 12fc 5cfa 5691 89e1 e1f9
```

### /dev/pts

/dev/pts是遠程登陸\(telnet,ssh等\)後創建的控制台設備文件所在的目錄。有什麼用呢？舉個例子，你打開一個終端，獲取到當前的pts：

```text
$ tty
/dev/pts/0
```

然後你又打開一個，輸入：

```text
$ echo "hahahaha">/dev/pts/0
```

你就會發現內容被打印到前面一個終端了。通常我們運行一個程序，其printf的打印都會打印在當前終端。

### 總結

實際上在/dev下還有非常多的特殊文件，但是不一一介紹。

