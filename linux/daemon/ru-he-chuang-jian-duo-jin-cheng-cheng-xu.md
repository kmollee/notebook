# 如何創建多進程程序

但是在C/C++中如何創建進程呢？或者說如何編寫多進程的程序呢？

### 什麼時候需要fork進程

一種可能見到的場景是在服務器程序中，一個請求到來後，為了避免服務器阻塞，fork出一個子進程處理請求，父進程仍然繼續等待請求到來。但這種方式無疑開銷會稍大。

另一種最常見的就是執行一個不同的程序，例如我們在shell終端執行一條命令，實際上就是bash（或者其他）調用fork之後，在執行exec族函數。

### fork

一個現有的進程可以通過fork函數來創建一個新的進程，這個進程通常稱為子進程。fork函數原型如下：

```text
#include<unistd.h>
pid_t fork(void);
```

如果調用成功，它將返回兩次，子進程返回值是0；父進程返回的是非0正值，表示子進程的進程id；如果調用失敗將返回-1，並且置errno變量。

有的朋友可能常常會記不住返回0的時候到底是子進程還是父進程。這裡教給大家一個方法。一個進程可以有多個子進程，但是一個子進程同一時刻最多只有一個父進程。子進程可以通過getppid獲取父進程的進程id，但是父進程卻沒法獲取，因此需要在fork後就得到子進程的進程id。

```text
//公眾號【編程珠璣】，博客 https://www.yanbinghu.com
#include<stdio.h>
#include<unistd.h>
int main(void)
{
    pid_t pid;
    char testVal[128] = {0};
    FILE *fp = fopen("test.txt","w");
    if(NULL == fp)
    {
        printf("open test.txt failed\n");
        return 0;
    }
    if(-1 == (pid = fork()))//等於-1時表明fork出錯
    {
        perror("fork error");
        return -1;
    }
    else if(0 == pid)//子進程
    {
        snprintf(testVal,128,"I am child，father pid is %d\n",getppid());
        fprintf(fp,"%s",testVal);

    }
    else  //父進程
    {
        snprintf(testVal,128,"I am parent,child pid is %d\n",pid);
        fprintf(fp,"%s",testVal);
    }
    printf("fork over,testVal is %s",testVal);
    //為了避免馬上退出sleep一段時間
    sleep(100);
    fclose(fp);
    return 0;
}
```

並在同目錄下創建一個test.txt文件，運行結果：

```text
fork over,testVal is I am parent,child pid is 13008
fork over,testVal is I am child，father pid is 13007
```

需要注意的是，不要對父進程先執行還是子進程先執行做任何假設，因為都有可能。所以，可能出現的運行結果並不一樣。

### fork到底做了什麼

fork被調用後，子進程擁有父進程的副本，因此它擁有父進程的數據空間，堆棧等。但是由於fork之後通常會調用exec函數去執行與原進程不想關的程序，因此fork時直接拷貝父進程的副本顯得沒有必要。為了提高fork的效率，採用了一種**寫時複製**的技術。即fork之後，子進程**名義上**擁有父進程的副本，但是實際上和父進程共用，只有當父子進程中有一個試圖修改這些區域時，才會以頁為單位創建一個真正的副本。

所以我們看到前面的示例程序中，父子進程都對testVal進程了修改，但是互不影響。因為它們修改了不同的區域。

### 子進程繼承了父進程哪些屬性？

由於子進程是父進程的一個副本，所以父進程有的屬性，子進程也都有，這些屬性包括

* 打開的文件描述符
* 會話ID
* 根目錄
* 資源限制
* 工作目錄
* 進程組ID
* 控制終端
* 環境
* …

我們運行前面的示例程序之後，重新打開一個終端，找到打開test.txt文件的進程：

```text
$ lsof test.txt
fork    9919 root    3r   REG  252,1        0 396427 test.txt
fork    9920 root    3r   REG  252,1        0 396427 test.txt
```

lsof命令的用法可以參考《[如何查看linux中文件打開情況？](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649284482&idx=1&sn=ae99d96fab26733cb1a208750f3dd5e8&chksm=f2f9ace5c58e25f397335937c6771ddc2f985783c787c6b3a4dc995622cfad7875ea482eb837&scene=21#wechat_redirect)》

也可以觀察進程打開的文件描述符：

```text
$ ls -l /proc/9919/fd
lrwx------ 1 root root 64 Aug 10 15:38 0 -> /dev/pts/1
lrwx------ 1 root root 64 Aug 10 15:38 1 -> /dev/pts/1
lrwx------ 1 root root 64 Aug 10 15:38 2 -> /dev/pts/1
lr-x------ 1 root root 64 Aug 10 15:38 3 -> /data/workspaces/practices/c/test.txt
```

為什麼這裡要特別說明打開的文件描述符呢？試想以下兩點：

* 父子進程對同一個文件進行寫，將共享文件偏移
* 如果該描述符是一個socket描述符，父進程退出後，子進程仍然打開著，父進程再次啟動，將會出現端口被佔用的問題。

所以如果父子進程的其中一個使用了fclose關閉了文件描述符，實際上還有另外一個進程打開了test.txt文件。

與前面testVal不同的是，如果父子進程都對文件進行寫，並不會產生兩個不同的文件，而是會對同一個文件進行寫，因此運行後會在同一個文件裡出現父子進程寫的內容：

```text
$ cat test.txt
I am parent,child pid is 13008
I am child，father pid is 13007
```

### 父子進程有哪些不同？

* fork之後的返回值不同，進程ID也不同
* 子進程未處理信號設置為空
* 子進程不繼承父進程設置的文件鎖
* 一般子進程會執行與父進程不完全一樣的代碼流程
* …

### 總結

fork用於創建進程，但是需要注意的是，子進程繼承了很多父進程的東西，如果子進程不需要可以進行修改或「丟棄」，例如子進程關閉父進程打開的文件描述符等等。理解了fork的寫時複製思想，也就會明白，實際上fork的速度是非常快的。本文總結點如下：

* fork調用一次，返回兩次
* 一個進程可以有多個子進程，但同一時刻最多只有一個父進程
* 子進程繼承了父進程很多屬性
* 父子進程執行的先後順序不一定

本文僅僅簡單介紹了fork，實際上得到子進程之後，還需要對子進程的狀態進行「監控」，否則會出現其他意想不到的問題。

