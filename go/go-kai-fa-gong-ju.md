# Go 開發工具

### 一. 開發工具

**1\)sql2go**  
用於將 sql 語句轉換為 golang 的 struct. 使用 ddl 語句即可。  
例如對於創建表的語句: show create table xxx. 將輸出的語句，直接粘貼進去就行。  
http://stming.cn/tool/sql2go.html

**2\)toml2go**  
用於將編碼後的 toml 文本轉換問 golang 的 struct.  
https://xuri.me/toml-to-go/

**3\)curl2go**  
用來將 curl 命令轉化為具體的 golang 代碼.  
https://mholt.github.io/curl-to-go/

**4\)json2go**  
用於將 json 文本轉換為 struct.  
https://mholt.github.io/json-to-go/

**5\)mysql 轉 ES 工具**  
http://www.ischoolbar.com/EsParser/

**6\)golang**  
模擬模板的工具，在支持泛型之前，可以考慮使用。  
https://github.com/cheekybits/genny

**7\)查看某一個庫的依賴情況，類似於 go list 功能**  
https://github.com/KyleBanks/depth

8\)一個好用的文件壓縮和解壓工具，集成了 zip，tar 等多種功能，主要還有跨平台。  
https://github.com/mholt/archiver

**9\)go 內置命令**  
go list 可以查看某一個包的依賴關係.  
go vet 可以檢查代碼不符合 golang 規範的地方。

**10\)熱編譯工具**  
https://github.com/silenceper/gowatch

**11\)revive**  
golang 代碼質量檢測工具  
https://github.com/mgechev/revive

**12\)Go Callvis**  
golang 的代碼調用鏈圖工具  
https://github.com/TrueFurby/go-callvis

**13\)Realize**  
開發流程改進工具  
https://github.com/oxequa/realize

**14\)Gotests**  
自動生成測試用例工具  
https://github.com/cweill/gotests

###  

### 二.調試工具

**1\)perf**  
代理工具，支持內存，cpu，堆棧查看，並支持火焰圖.  
perf 工具和 go-torch 工具，快捷定位程序問題.  
https://github.com/uber-archive/go-torch  
https://github.com/google/gops

**2\)dlv 遠程調試**  
基於 goland+dlv 可以實現遠程調式的能力.  
https://github.com/go-delve/delve  
提供了對 golang 原生的支持，相比 gdb 調試，簡單太多。

**3\)網絡代理工具**  
goproxy 代理，支持多種協議，支持 ssh 穿透和 kcp 協議.  
https://github.com/snail007/goproxy

**4\)抓包工具**  
go-sniffer 工具，可擴展的抓包工具，可以開發自定義協議的工具包. 現在只支持了 http，mysql，redis，mongodb.  
基於這個工具，我們開發了 qapp 協議的抓包。  
https://github.com/40t/go-sniffer

**5\)反向代理工具，快捷開放內網端口供外部使用。**  
ngrok 可以讓內網服務外部調用  
https://ngrok.com/  
https://github.com/inconshreveable/ngrok

**6\)配置化生成證書**  
從根證書，到業務側證書一鍵生成.  
https://github.com/cloudflare/cfssl

**7\)免費的證書獲取工具**  
基於 acme 協議，從 letsencrypt 生成免費的證書，有效期 1 年，可自動續期。  
https://github.com/Neilpang/acme.sh

8\)開發環境管理工具，單機搭建可移植工具的利器。支持多種虛擬機後端。  
**vagrant**常被拿來同 docker 相比，值得擁有。  
https://github.com/hashicorp/vagrant

**9\)輕量級容器調度工具**  
nomad 可以非常方便的管理容器和傳統應用，相比 k8s 來說，簡單不要太多.  
https://github.com/hashicorp/nomad

**10\)敏感信息和密鑰管理工具**  
https://github.com/hashicorp/vault

**11\)高度可配置化的 http 轉發工具，基於 etcd 配置。**  
https://github.com/gojek/weaver

**12\)進程監控工具 supervisor**  
https://www.jianshu.com/p/39b476e808d8

13\)基於**procFile**進程管理工具. 相比 supervisor 更加簡單。  
https://github.com/ddollar/foreman

14\)基於 http，https，websocket 的**調試代理工具**，配置功能豐富。在線教育的 nohost web 調試工具，基於此開發.  
https://github.com/avwo/whistle

**15\)分佈式調度工具**  
https://github.com/shunfei/cronsun/blob/master/README\_ZH.md  
https://github.com/ouqiang/gocron

**16\)自動化運維平台 Gaia**  
https://github.com/gaia-pipeline/gaia

###  

### 三. 網絡工具

![](https://mmbiz.qpic.cn/mmbiz_png/j3gficicyOvatzJibnFP6EXDYRTOzL8ZgT4oXh17qR8vTl5uHKrXtFcp45GS6VyRXMJw4vnSQ3mDibKxL7ficGrPshw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###  

### 四. 常用網站

go 百科全書: https://awesome-go.com/  


json 解析: https://www.json.cn/  


出口 IP: https://ipinfo.io/

redis 命令: http://doc.redisfans.com/  


ES 命令首頁: 

https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

UrlEncode: http://tool.chinaz.com/Tools/urlencode.aspx

Base64: https://tool.oschina.net/encrypt?type=3

Guid: https://www.guidgen.com/

常用工具: http://www.ofmonkey.com/

###  

### 五. golang 常用庫

**日誌**  
https://github.com/Sirupsen/logrus  
https://github.com/uber-go/zap

**配置**  
兼容 json，toml，yaml，hcl 等格式的日誌庫.  
https://github.com/spf13/viper

**存儲**  
mysql: https://github.com/go-xorm/xorm  
es: https://github.com/elastic/elasticsearch  
redis: https://github.com/gomodule/redigo  
mongo: https://github.com/mongodb/mongo-go-driver  
kafka: https://github.com/Shopify/sarama

**數據結構**  
https://github.com/emirpasic/gods

**命令行**  
https://github.com/spf13/cobra

**框架**  
https://github.com/grpc/grpc-go  
https://github.com/gin-gonic/gin

**並發**  
https://github.com/Jeffail/tunny  
https://github.com/benmanns/goworker  
現在我們框架在用的，雖然 star 不多，但是確實好用，當然還可以更好用.  
https://github.com/rafaeldias/async

**工具**  
定義了實用的判定類，以及針對結構體的校驗邏輯，避免業務側寫複雜的代碼.  
https://github.com/asaskevich/govalidator  
https://github.com/bytedance/go-tagexpr

protobuf 文件動態解析的接口，可以實現反射相關的能力。  
https://github.com/jhump/protoreflect

**表達式引擎工具**  
https://github.com/Knetic/govaluate  
https://github.com/google/cel-go

**字符串處理**  
https://github.com/huandu/xstrings

**ratelimit 工具**  
https://github.com/uber-go/ratelimit  
https://blog.csdn.net/chenchongg/article/details/85342086  
https://github.com/juju/ratelimit

**golang 熔斷的庫**  
熔斷除了考慮頻率限制，還要考慮 qps，出錯率等其他東西.  
https://github.com/afex/hystrix-go  
https://github.com/sony/gobreaker

**表格**  
https://github.com/chenjiandongx/go-echarts

**tail 工具庫**  
https://github.com/hpcloud/taglshi

