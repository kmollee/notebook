# 如何讓你的程序同時只能運行一個？

有些程序我們希望在一台機器上只有一個實例在運行，我在windows下也遇到過很多類似這樣的程序，如QQ，它只允許同時運行一個。那麼我們在Linux該如何實現這樣的單例運行的程序呢？  


### 思路

實現這樣的程序方法很多，但是總體思路都是類似的：

* 1.啟動程序，檢測標誌，判斷是否有同樣的程序運行，是則2，否則3
* 2.程序退出
* 3.程序啟動，並設置標誌，以便下次啟動時檢測

### 實現方法

按照這種思路，實現的方法有很多種，例如使用ps等命令獲取該進程的進程數，大於0 表示已有運行；啟動後寫一個臨時文件，如果下次啟動時發現有該文件，則直接退出；創建一個文件並加鎖，退出時刪除文件，新的程序啟動時試圖加鎖，如果失敗，則說明已有實例運行……除了上面說到的這些，可能還有一些其他的實際做法，但是本文介紹一種實用並且也是非常通用的做法，即文件鎖的方法。

### 基本原理

程序在啟動後，打開一個program.pid文件（無則創建），然後試圖去設置文件鎖（如果還不理解鎖的概念，可以簡單理解為，一旦a寫鎖定了，b就無法進一步寫操作了，除非a釋放鎖），如果設置成功，就將該程序的進程ID寫入該文件；如果加鎖失敗，那麼說明已經有另外一個實例在運行了，則退出此次啟動。而當前已經運行的程序如果退出了，該文件會自動解除鎖定。實際上，我們觀察一下/var/run/目錄下，有很多類似這樣的文件：

```text
$ ls -l /var/run/*.pid
-rw-r--r-- 1 root root 5 11月 24 08:19 /var/run/acpid.pid
-rw-r--r-- 1 root root 5 11月 24 08:19 /var/run/atd.pid
-rw-r--r-- 1 root root 5 11月 24 08:19 /var/run/crond.pid
-rw-r--r-- 1 root root 5 11月 24 09:08 /var/run/dhclient-wlp3s0.pid
-rw-r--r-- 1 root root 4 11月 24 08:19 /var/run/docker.pid
```

不過這個位置通常只有root用戶能夠寫入。

### 實現

在看代碼實現之前，先看下文件鎖（記錄鎖）和fcntl函數。  
flock結構至少包含以下字段：

```text
struct flock {
    short l_type; /*鎖類型 F_RDLCK,F_WRLCK, F_UNLCK*/
    short l_whence;  /* 偏移開始的位置 SEEK_SET, SEEK_CUR, SEEK_END */
    off_t l_start;   /* 開始鎖定區域*/
    off_t l_len;     /* 鎖定字節數 */
    pid_t l_pid;     /* 記錄鎖的進程ID*/
};
```

從結構體成員可以看到，它不僅可以鎖整個文件，還可以鎖某個區域，但是本文僅用於鎖定整個文件。

```text
int fcntl(int fd, int cmd, ... /* arg */ 
```

fcntl函數的cmd選項也很多，本文只用到F\_SETLK（或F\_SETLKW，），即設置鎖。  
結合以上兩者，參考代碼如下：

```text
//來源：公眾號【編程珠璣】
//博客：https://www.yanbinghu.com
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <errno.h>
#include <sys/stat.h>
#define PROC_NAME "single_instance"
#define PID_FILE_PATH "/var/run/"
static int lockFile(int fd);
static int isRunning(const char *procname);
int main(void)
{
    /*判斷是否已經有實例在運行*/
    if(isRunning(PROC_NAME))
    {
        exit(-1);
    }
    printf("run ok\n");
    sleep(20);//避免程序立即退出
    return 0;
}
/*鎖文件還可以使用flock，目的是類似的。不過是它是BSD系統調用，並且某些版本不支持NFS，出於移植性考慮，使用fcntl*/
static int lockFile(int fd)
{
    struct flock fl;
    fl.l_type   = F_WRLCK;//設置寫鎖
    fl.l_start  = 0;
    fl.l_whence = SEEK_SET;
    fl.l_len    = 0;
    fl.l_pid = -1;//鎖定文件，設置為-1

    return(fcntl(fd, F_SETLK, &fl));
}

static int isRunning(const char *procname)
{
    char buf[16] = {0};
    char filename[128] = {0};
    sprintf(filename, "%s%s.pid",PID_FILE_PATH,procname);
    int fd = open(filename, O_CREAT|O_RDWR );//可讀可寫，不存在時創建
    if (fd < 0) {
        printf("open file %s failed!\n", filename);
        return 1;
    }
    if (-1 == lockFile(fd)) /*嘗試加鎖*/
    {                                                  
        printf("%s is already running\n", procname);
        close(fd);
        return 1;
    } 
    else 
    {
        ftruncate(fd, 0);/*截斷文件，重新寫入pid*/
        sprintf(buf, "%ld", (long)getpid());
        write(fd, buf, strlen(buf) + 1);
        return 0;
    }
}
```

### 編譯運行

```text
$ gcc -o single_instance single_instance.c
$ ./single_instance  #注意root權限運行，或者調整pid文件位置
run ok
```

查看pid文件目錄下已經有了pidfile：

```text
$ ls -al /var/run/single_instance.pid
-rw-r--r-- 1 root root 6 11月 24 11:36 /var/run/single_instance.pid
```

在另外一個終端再次嘗試運行：

```text
$ ./single_instance
single_instance is already running
```

如果你想控制同一個目錄下的bin文件只能運行一個，那麼可以設置pid文件的位置為當前目錄。這種方式有什麼特點呢：

* 簡單可靠
* 可讀可見，相比於信號量或共享內存，它更容易觀察
* 無性能要求，啟動時加鎖，結束釋放。
* 一旦出現異常沒有釋放，也可以手動刪除文件

當然對於BSD系統，還可以使用下面的接口來完成：

```text
 struct pidfh *
     pidfile_open(const char *path, mode_t mode, pid_t *pidptr);
int pidfile_write(struct pidfh *pfh);

int pidfile_close(struct pidfh *pfh);

int pidfile_remove(struct pidfh *pfh);
```

本文就不過多介紹了。

### 總結

單例運行的基本原理是類似的，而pid文件是一種常見的單例運行的方式，很多知名的開源組件都是使用類似的方式。對於shell腳本，還可以使用flock命令進行類似的操作。

