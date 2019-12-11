# setsid

####  測試 setsid\(\)

  
 假設我們一開始建立了一個行程\(process\)，那麼該行程若以 fork\(\)複製出一個相同的行程，稱之為子行程\(child proccess\)，而產生子行程的行程則稱為父行程\(parent process\)，這是行程之間的父子關係。  
  
 最早的父行程具有行程領導者\(process leader\)的地位，行程領導者的 PID \(Process ID\)就是該行程家族群組的群組 ID \(group ID\)。總之，只要都是最早的父行程或其後代所產生的子行程都會繼承相同的群組 ID，而 Linux kernel 會根據該群組 ID，將訊號送給各行程群組。  
    
 - system call  


> getpgid\(\) 與 getpgrp\(\)：取得該行程的群組 ID  
>  getpid\(\): 取得該行程的 PID  
>  getppid\(\): 取得該行程之父行程的 PID  
>  setpgid\(\): 將指定的 PID 設定為該行程的群組 ID，若 PID 為 0，則使用該行程 PID。  
>  setsid\(\): 將該行程設定為 session leader 與行程領導者。  
>  getsid\(\)

  
 /\* man 2 setsid for details \*/  


> ```text
> #include <stdlib.h>
> #include <sys/types.h>
> #include <unistd.h>
>
> int main(void)
> {
>     if(fork()>0) {
>       printf("p getpid: %d\n", getpid());
>       printf("p geppid: %d\n", getppid());
>       printf("p getpgrp: %d\n", getpgrp());
>       printf("p setsid: %d\n", setsid());
>       printf("p getpgrp: %d\n", getpgrp());
>       printf("p geppid: %d\n", getppid());
>     } else {
>       printf("gepid: %d\n", getpid());
>       printf("geppid: %d\n", getppid());
>       printf("getpgrp: %d\n", getpgrp());
>       printf("setsid: %d\n", setsid());
>       printf("getpgrp: %d\n", getpgrp());
>       printf("geppid: %d\n", getppid());
>     }
>
>       return 0;
> }  
> ```

> \# gcc test.c -o test

> \# ./test  
>  gepid: 3899  
>  geppid: 3898  
>  getpgrp: 3898 &lt;-----  
>  setsid: 3899  
>  getpgrp: 3899 &lt;-----  
>  geppid: 3898  
>  p getpid: 3898 &lt;-----  
>  p geppid: 3697  
>  p getpgrp: 3898  
>  p setsid: -1  
>  p getpgrp: 3898  
>  p geppid: 3697

  
由結果觀察到，經過 setsid 之後的子行程已經自己成家立業，有自己的行程群組，而不再與原本的父行程同一個行程群組。  
  
 PS: 這裡所謂的行程群組與行程所屬的使用者群組是不一樣。

