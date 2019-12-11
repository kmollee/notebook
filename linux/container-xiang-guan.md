# Container 相關

Container 與虛擬機類似,都是在原作業系統提供一個環境給另外一個作業系統來使用,虛擬機器 Virtual Machine 透過 VMM \(Virtual Machine Monitor,也可以稱作 Hybervisor\) 的方式來建立一個虛擬化的環境給虛擬機的作業系統來使用,而 Container 是在原作業系統中透過資源共享的方式,建立出一個獨立空間,這環境有自己的 file system, process 與 block I/O ,network.

傳統的 VMM 優點是所有的作業系統我們都可以模擬,但需要透過 VMM 效能會比較差一點,Container 只能使用在 Linux 同值性的作業系統下,透過資源共享,效能佳,也不會消耗過多的系統資源.

但 Container 要如何隔離主系統與 Containers 之間的存取? 與限制 Containers 使用的資源?  
 這就需要透過 cgroup & namespace 這兩項功能 \(Linux 核心從 2.6.24 之後的版本內建了 Control Group 和 Namespaces \)



#### **Cgroup \(Control Group\)**

\*\*\*\*

