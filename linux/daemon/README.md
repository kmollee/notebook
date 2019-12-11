# Daemon

### 如何讓程序真正地在後台運行 <a id="activity-name"></a>





如何實現一個守護進程？如何讓程序在後台運行?這是後台開發面試常問的一道題，那麼守護進程到底是什麼？又該如何實現？

### 守護進程

守護進程通常生存期長，很多是在系統啟動時啟動，系統退出時才關閉。它們的特點通常沒有控制終端，後台運行。有人可能會會心一笑，後台運行程序，我知道呀。還有兩種方式呢

```text
$ ./hello &
```

看，多麼簡單。但是運行之後，你試著關閉當前終端，你會發現程序會停止運行，因為一旦關閉終端，程序會收到一個信號SIGHUP，而收到該信號默認的動作就是程序退出。沒關係啊，我還有招：

```text
$ nohup ./hello & #注意這裡&是必要的，否則不會變成後台進程
$ jobs
[1]+  Running                 nohup ./hello &
```

我使用nohup命令總可以了吧？挺好的，nohup會忽略SIGHUP命令，並有了&的加持，即便終端關了，也能繼續執行。但它的終端輸出還會記錄默認還在nohup.out文件中，同時，如果將huponexit關閉，它同樣難逃命運：

```text
$ shopt -s huponexit  #shopt -u huponexit 設置為off
$ shopt |grep onexit
huponexit       on
```

一旦終端退出（ctrl+D）後，nohup也救不了。下面要介紹的守護進程一一種完全脫離終端，有著自己的會話。  
如果你在你的Linux系統中執行下面的命令：

```text
$ ps -elf
```

就會發現一些進程的tty列是？，當然這並不是說明它們是守護進程，而那些用\[\]括起來的，是內核守護進程想像一下，如果沒有任何人登錄的服務器上面的運行程序，難道每次執行的時候都要使用nuhup+&？況且，一旦系統的huponexit選項是打開的，這種方式仍然無法避免終端關閉程序就退出的命運！那麼就需要實現用戶守護進程了，或者說daemon化。

### 如何實現

其實現過程基本遵循以下原則：

* 調用umask設置文件模式，通常設置為0。為了便於後續創建文件，不使用繼承而來的父進程的設置，需要設置新的權限掩碼。
* 調用fork，創建子進程，並且父進程退出
* 調用setdid創建新的會話（一個或多個進程組的集合），由於當前進程不是一個進程組的組長，因此會創建一個新的會話，卻成為組長進程，同時沒有控制終端。
* 將當前工作目錄切換為根目錄。同樣的，其工作目錄可能是從父進程繼承而來的，可以自己另立山頭。
* 關閉不需要的文件描述符。同樣的，可能從父進程繼承了一些打開的文件描述符，而這些描述符可能再也不需要，因此可以關閉。
* 重定向標準輸出，標準輸入和標準錯誤到/dev/null（相關閱讀：[Linux下你還知道這些特殊文件？](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649285306&idx=2&sn=afdebad474f0665eeb0b3fcdb44b1dff&chksm=f2f991ddc58e18cb86399b98d5602af8c31cd8b12628998bd1d3b7030c0737c46265ddd41f76&scene=21#wechat_redirect)）

實際上，從上面的描述可以發現，這些規則都有幾乎相同的目標，那就是不想成為富二代，擺脫父親的控制。（在[fork的介紹](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649284871&idx=1&sn=675b7d077ab356f65510552827f99df8&chksm=f2f99260c58e1b76b5f01302f6a273570bc50a573f369bb4b8c833d8153dfbfedfe33e472042&scene=21#wechat_redirect)中，我們說到，兒子從父親那裡繼承了很多東西）

* 重新設置權限掩碼，避免受父進程影響
* 創建新的會話，脫離終端
* 使用新的工作目錄
* 關閉不需要的文件描述符
* 關閉標準輸入，標準輸出和標準錯誤

所以通過這些也可以明白，有些規則並不是完全強制的，可根據實際程序的情況進行設置，不過按照常規做法是一個比較好的選擇。

### 具體實現

參考代碼如下：

```text
//來源：公眾號【編程珠璣】
//https://www.yanbinghu.com
#include<stdio.h>
#include<sys/resource.h>
#include<fcntl.h>
#include<unistd.h>
#include<stdlib.h>
/*實現僅供參考，可根據實際情況調整*/
int daemonize()
{
    /*清除文件權限掩碼*/
    umask(0);

    /*父進程退出*/
    pid_t pid;
    if((pid=fork()) < 0)
    {
        /*for 出錯*/
        perror("fork error");
        return -1;
    }
    else if(0 != pid)/*父進程*/
    {
        printf("father exit\n");
        exit(0);
    }
    /*子進程，成為組長進程，並且擺脫終端*/
    setsid();

    /*修改工作目錄*/
    if(chdir("/") < 0)
    {
        perror("change dir failed");
        return -1;
    }


    struct rlimit rl;
    /*先獲取文件描述符最大值*/
    if(getrlimit(RLIMIT_NOFILE,&rl) < 0)
    {
        perror("get file decription failed");
        return -1;
    }
    /*如果無限制，則設置為1024*/
    if(rl.rlim_max == RLIM_INFINITY)
        rl.rlim_max = 1024;

    /*為了使得終端有輸出，保留了文件描述符0，1，2;實際上父進程可能沒有打開2以上的文件描述符*/
    int i;
    for(i = 3;i < rl.rlim_max;i++)
        close(i);
    return 0;
}
int main(void)
{
    if(0 == daemonize())
    {
        printf("daemonize ok\n");
        sleep(20);
    }
    else
    {
        printf("daemonize failed\n");
        sleep(20);
    }
    return 0;
}
```

編譯運行，你就會發現，它已經可以歡脫地運行啦。代碼中有幾個點，需要關注一下。為了保留printf的輸出，我在daemonize函數中，並沒有關閉所有的文件描述符，0，1，2可以參考《[如何理解 Linux shell中「2&gt;&1」？](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649284005&idx=1&sn=dc9e9db84ec363d5a0ed7f84bc6ec866&chksm=f2f9aec2c58e27d42eee09ae646e493530d8d0deda822df2ffa2e5153d210b709a6d69272957&scene=21#wechat_redirect)》，當然了，如果想讓printf的輸出保存到文件，也有方法，可以參考《[如何優雅地將printf的打印保存在文件中？](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649285312&idx=1&sn=2a1dd548fd5a17decd303966322bbf4c&chksm=f2f991a7c58e18b1d8d49f8ece1e84a859a5c29212c40e36d62b9bb0e51320465b8e936aab60&scene=21#wechat_redirect)》，這裡就不再贅述了。

### 實際實現

實際上，已經有一個接口可以幫我們做這些事情：

```text
#include <unistd.h>
int daemon(int nochdir, int noclose);
```

即daemon函數，它有兩個參數

* nochdir 為0時，表示修改其根目錄為/，否則不變
* noclose，為0時，表示將標準輸入，標準輸出，標準錯誤重定向到/dev/null。

簡單例子：

```text
//來源：公眾號【編程珠璣】
//https://www.yanbinghu.com
#include<stdio.h>
#include<unistd.h>
int main(void)
{
    if(0 == daemon(1,1))
    {
        printf("daemon ok\n");
        sleep(20);
    }
    else
    {
        printf("daemon failed\n");
        sleep(20);
    }
    return 0;
}
```

如果你還要實現單例化，可以參考《[如何讓你的程序同時只能運行一個？](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649285343&idx=2&sn=7dfa30d8cc682f349529435902e7c867&chksm=f2f991b8c58e18ae49b4dd52a84c527cdc007fc89e7f2c73ac21d4d9a983d682bb22ad0460f8&scene=21#wechat_redirect)》，使得同時只有一個該進程運行。

### 總結

以上就進程後台運行以及是守護進程實現的介紹，關鍵點有

* 創建子進程，父進程退出
* 創建新的會話，脫離終端

附上daemon的源碼：

```text
int daemon(nochdir, noclose)
    int nochdir, noclose;
{
    int fd;

    switch (fork()) 
    {
    case -1:
        return (-1);
    case 0:
        break;
    default:
        _exit(0);
    }

    if (setsid() == -1)
        return (-1);

    if (!nochdir)
        (void)chdir("/");

    if (!noclose && (fd = open(_PATH_DEVNULL, O_RDWR, 0)) != -1) 
    {
        (void)dup2(fd, STDIN_FILENO);
        (void)dup2(fd, STDOUT_FILENO);
        (void)dup2(fd, STDERR_FILENO);
        if (fd > 2)
            (void)close (fd);
    }
    return (0);
}
```

