# 如何使用 Google Cloud Platform - Cloud DNS

DNS, Domain Name Service, 是一種以文字代替 IP 位置的一種查詢服務，讓我們可以使用 文字 來取代 IP 位置。

例如 我們想要到 google search 的網頁時，我們可以輸入 https://www.google.com.tw 而不是 https://172.217.24.3 就是利用 DNS 幫我們把 https://www.google.com.tw 轉換成 https://172.217.24.3 , 因為在網路上真正使用的是 IP Address, 但由於 IP Address 非常不利於我們記憶，所以就有了 DNS 這樣一個服務。

## 關於 DNS 有以下幾種類型：

1. SOA (Start Of Authority):
   每一個特定的網域名稱都必須有一個 SOA, 這裡會紀錄該網域名稱的主要名稱伺服器(Primary DNS Server)和次要的名稱伺服器(Secondary DNS Server)。SOA 也紀錄著一個網域名稱的起點和與時效性有關的參數。
1. NS (Name Server)
   NS 是用來指定操作的 DNS 伺服器主機名稱，並且不可以用 IP 位置表示。
1. A (Address)
   這個是用來將 DNS 網域名稱對應到 IPv4 的 IP Address. 例如：我們這次要練習的是新增一台 Compute Engine, 就可以把他的 IP 和對應到的網址填在這裡。
1. AAAA
   與 A 的功能相同，唯一不同的是，AAAA 是用來對應 IPv6 128bit 的 位置，而 A 是用來對應 IPv4 32bit 的位置。
1. PTR (Pointer)
   PTR 主要的功能是定義某個 IP 對應的 Domain name, 也就是將 IP 位址轉換成 FQDN。
1. CNAME (canoical name)
   主要的目的是可以為一部主機設定許多的別名, 例如: elegant-aeon.tk 的別名可以是 mine.elegant-aeon.tk 和 www.elegant-aeon.tk
1. MX (Mail Exchanger)
   設定區域中擔任郵件伺服器的主機，所有要送往那部機器的 mail 都要經過 mail exchanger 轉送。而數字則是該主機郵件傳遞時的優先次序，此值越低表示有越高的郵件處理優先權。

## 申請免費的 DNS

[freenom] 可以申請免費的 DNS

## 設定 Cloud DNS

GCP 的 Cloud DNS 設定非常的容易, 首先是進到 GCP 的主控台, 接著點選 **Cloud DNS**

![](images/learn_gcp_cloud_dns_01_click_cloud_dns.png)

接著是選擇建立區域

![](images/learn_gcp_cloud_dns_02_select_create_domain.png)

填入可以提供自己識別的 **區域名稱** 和 我們剛剛申請的 **DNS 名稱** `elegant-aeon.tk` 接著點選 **建立**

關於 DNSSEC 就保持停用即可，關於 DNSSEC 可以參考 [這裡][dnssec]

![](images/learn_gcp_cloud_dns_03_create_dns_domain.png)

接著我們就可以看到 Cloud DNS 預設就為我們建立了兩筆紀錄, 分別是 **NS** 以及 **SOA**, NS 必須要先寄下來，等會兒會用到，SOA 不用管他，放著就好。

接著我們就可以把我們的主機 IP 給填上了，點選 **新增紀錄集**, 在分別輸入 DNS 名稱和 IPv4 位置, 因為我們想要使用的是 DNS 對應到 IPv4, 所以我們在資源紀錄類型就選擇 A, TTL 和 TTL 單位用預設的就可以了。

在這裡我會新增兩筆資料:

1. elegant-aeon.tk 對應到 (A) 35.201.139.239
1. elegant-aeon.tk 有一個別名 (CNAME) www.elegant-aeon.tk
1. mine.elegant-aeon.tk 對應到 (A) 35.201.139.239

![](images/learn_gcp_cloud_dns_05_add_record_00.png)

![](images/learn_gcp_cloud_dns_05_add_record_01.png)

![](images/learn_gcp_cloud_dns_05_add_record_02.png)

新增好兩筆紀錄之後就會在 區域詳細資料看到我們剛剛新增的那兩筆

![](images/learn_gcp_cloud_dns_06_dns_detail.png)

做到這邊，在 GCP Cloud DNS 的設定就差不多了，但還不會生效，我們必須要把我們 domain name 裡的 NS 資料填到 我們申請 DNS 的主機上面才會真的生效。

## 設定 nameserver

因為我們在這個練習裡面是使用 [freenom] 的免費 DNS, 所以我們就要回到 [freenom] 去設定我們在 GCP - Cloud DNS 裡的 NS 位址。

首先，在登入 [freenom] 之後選擇 **Service** -> **My Domains**

![](images/learn_gcp_cloud_dns_07_setup_ns_01.png)

接著在我們要設定的 Domain 上面點選 **Manage Domain**

![](images/learn_gcp_cloud_dns_07_setup_ns_02.png)

進到我們欲修改的 **Domain** 之後，選擇 **Management Tools** -> **Nameservers**

![](images/learn_gcp_cloud_dns_07_setup_ns_03.png)

最後一步就是選擇 **Use coustom nameservers(enter below)** 並且將我們的 GCP Cloud DNS 的 NS 填入下方的 Nameserver 1 ~ 4, 再單擊 **Change Nameservers**, 就完成了。

!! 注意 !! GCP 的 Cloud DNS 所給予我們的 NS 每次都很有可能會不一樣，所以如果有重新設定還是需要再留意一下! 在下圖中是我第一次設定時 Cloud DNS 給的 nameserver.

基本上一分鐘左右就會生效了，如果遲遲沒有生效，可以檢查一下是不是 [freenom] 沒有設定好，因為我遇過幾次明明設定了，但就是沒有存進去，目前還不知道是為什麼，但再重新設定一次就好了。

![](images/learn_gcp_cloud_dns_07_setup_ns_04.png)


下圖是使用 nslookup 驗證的結果

![](images/learn_gcp_cloud_dns_08_nslookup_dns.png)


[freenom]: http://www.freenom.com
[dnssec]: https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F%E5%AE%89%E5%85%A8%E6%89%A9%E5%B1%95