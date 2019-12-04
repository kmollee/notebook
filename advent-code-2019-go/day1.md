# File Read/Write

開啟檔案

```text
os.Open(path)
os.OpenFile(path, os.O_RDONLY, 0644)
os.Create(path)
```



比較常見使用如下

* ioutils
  * ReadFile
  * ReadAll
  * WriteFile
* bufio
  * NewWriter
  * ...
* 或直接對 file handle 做 read/write

不過最後還是記得他們最後都可以  `io.Reader`  `io.Writer` 方式串接

```go
package main

import (
	"bufio"
	"bytes"
	"io"
	"io/ioutil"
	"os"
)

const inputPath = "./input.txt"

// 利用ioutil.ReadFile 直接從文件讀取到 []byte 中
func read0(path string) (string, error) {
	b, err := ioutil.ReadFile(path)
	return string(b), err
}

// 先從文件讀取到file中，在從file讀取到buf, buf在追加到最終的[]byte
func read1(path string) (string, error) {
	f, err := os.Open(path)
	if err != nil {
		return "", err
	}
	defer f.Close()

	// read buffer
	var chunk []byte
	buf := make([]byte, 1024)
	for {
		n, err := f.Read(buf)
		if err != nil {
			return "", err
		}

		if n == 0 {
			break
		}
		chunk = append(chunk, buf[:n]...)
	}
	return string(chunk), nil
}

// 先從文件讀取到file, 在從file讀取到Reader中，從Reader讀取到buf, buf最終追加到[]byte
func read2(path string) (string, error) {
	// get file handle
	f, err := os.OpenFile(path, os.O_RDONLY, 0644)
	if err != nil {
		return "", err
	}
	defer f.Close()

	reader := bufio.NewReader(f)
	var chunk []byte
	buf := make([]byte, 1024)

	for {
		n, err := reader.Read(buf)
		if err != nil {
			return "", err
		}

		if n == 0 {
			break
		}
		chunk = append(chunk, buf[:n]...)
	}
	return string(chunk), nil

}

// 讀取到file中，再利用ioutil將file直接讀取到[]byte中
func read3(path string) (string, error) {
	// get file handle
	f, err := os.OpenFile(path, os.O_RDONLY, 0644)
	if err != nil {
		return "", err
	}
	defer f.Close()

	b, err := ioutil.ReadAll(f)
	if err != nil {
		return "", err
	}

	return string(b), nil

}

// 使用 io.WriteString 寫入文件
func write(write io.Writer, content string) (int, error) {
	var b bytes.Buffer
	n, err := io.WriteString(&b, content)
	return n, err
}

// 使用 ioutil.WriteFile 寫入文件
func write1(path string, content string) error {
	return ioutil.WriteFile(path, []byte(content), 0644)
}

// 使用 File(Write,WriteString) 寫入文件
func write2(path string, content string) (int, error) {
	// get file handle
	f, err := os.OpenFile(path, os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		return 0, err
	}
	defer f.Close()

	return f.Write([]byte(content))
	// return f.WriteString(content)
}

// 使用 bufio.NewWriter 寫入文件
func write3(path string, content string) (int, error) {
	// get file handle
	f, err := os.OpenFile(path, os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		return 0, err
	}
	defer f.Close()
	writer := bufio.NewWriter(f)
	defer writer.Flush()
	return writer.WriteString(content)
}

func main() {

}

```





