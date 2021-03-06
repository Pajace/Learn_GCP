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

直接到這裡 -> [freenom] 就可以申請面費的 DNS

### 下面是申請過程的步驟:

連上 [freenom] 首頁會看到下面的圖示，直接在 **發現一個新的免費功能變數** 框框裡面輸入你所想要申請的 Domain name

![request dns step1][00-request-dns-01]

在這裡我想要申請的 Domain name 是 `aeon-pool`，所以我就輸入 `aeon-pool` 並且按下 **檢查可用性**

![request dns step1][00-request-dns-02]

接著我們就可以發現他列出一堆可以使用的網址是以 `aeon-pool` 開頭的，前面都是免費的，後面就是要付費的。這時候就可以選擇想要的點選  **馬上獲取**

![request dns step1][00-request-dns-03]
![request dns step1][00-request-dns-04]

因為我選擇的是 `.tk` 這個網域名稱, 所以我申請的網域名稱就會是 `aeon-pool.tk`，如果可用，他就會變成 **已選擇** ，然後按下付款按鈕(因為它是免費的所以按下後費用一樣會是 0)

![request dns step1][00-request-dns-05]

選擇付款後，最長可以選到 12 個月都是免費的，如果選一年含以上就開始要付費了。(在這裡我是選 12 個月，其實就跟一年是一樣的意思，但他仍舊是免費的 XD)

![request dns step1][00-request-dns-06]

選完有效期限之後，他就會要求你填寫一些你的個人資料(這裡不會要求你填寫信用卡資料，所以可以放心填寫)，填寫完畢後直接將底下的 *have read and aggree to the Terms & Conditions* 打勾並且按下 **Complete order**

![request dns step1][00-request-dns-07]
![request dns step1][00-request-dns-08]

出現這個畫面時就代表你已經申請成功了

![request dns step1][00-request-dns-09]

## 設定 Cloud DNS

GCP 的 Cloud DNS 設定非常的容易, 首先是進到 GCP 的主控台, 接著點選 **Cloud DNS**

![enter-cloud-dns][01-enter-cloud-dns]

接著是選擇建立區域

![create domain][02-create-domain]

填入可以提供自己識別的 **區域名稱** 和 我們剛剛申請的 **DNS 名稱** `elegant-aeon.tk` 接著點選 **建立**

關於 DNSSEC 就保持停用即可，關於 DNSSEC 可以參考 [這裡][dnssec]

![dns domain created][03-dns-domain-created]

接著我們就可以看到 Cloud DNS 預設就為我們建立了兩筆紀錄, 分別是 **NS** 以及 **SOA**, NS 必須要先寄下來，等會兒會用到，SOA 不用管他，放著就好。

接著我們就可以把我們的主機 IP 給填上了，點選 **新增紀錄集**, 在分別輸入 DNS 名稱和 IPv4 位置, 因為我們想要使用的是 DNS 對應到 IPv4, 所以我們在資源紀錄類型就選擇 A, TTL 和 TTL 單位用預設的就可以了。

在這裡我會新增兩筆資料:

1. elegant-aeon.tk 對應到 (A) 35.201.139.239
1. elegant-aeon.tk 有一個別名 (CNAME) www.elegant-aeon.tk
1. mine.elegant-aeon.tk 對應到 (A) 35.201.139.239

![dns record][04-add-dns-record00]
![dns record][04-add-dns-record01]
![dns record][04-add-dns-record02]

新增好兩筆紀錄之後就會在 區域詳細資料看到我們剛剛新增的那兩筆

![show dns detail][05-show-dns-detail]

做到這邊，在 GCP Cloud DNS 的設定就差不多了，但還不會生效，我們必須要把我們 domain name 裡的 NS 資料填到 我們申請 DNS 的主機上面才會真的生效。

## 設定 nameserver

因為我們在這個練習裡面是使用 [freenom] 的免費 DNS, 所以我們就要回到 [freenom] 去設定我們在 GCP - Cloud DNS 裡的 NS 位址。

首先，在登入 [freenom] 之後選擇 **Service** -> **My Domains**

![setup NS for freenom][setup-ns-on-freenom01]

接著在我們要設定的 Domain 上面點選 **Manage Domain**

![setup NS for freenom][setup-ns-on-freenom02]

進到我們欲修改的 **Domain** 之後，選擇 **Management Tools** -> **Nameservers**

![setup NS for freenom][setup-ns-on-freenom03]

最後一步就是選擇 **Use coustom nameservers(enter below)** 並且將我們的 GCP Cloud DNS 的 NS 填入下方的 Nameserver 1 ~ 4, 再單擊 **Change Nameservers**, 就完成了。

!! 注意 !! GCP 的 Cloud DNS 所給予我們的 NS 每次都很有可能會不一樣，所以如果有重新設定還是需要再留意一下! 在下圖中是我第一次設定時 Cloud DNS 給的 nameserver.

基本上一分鐘左右就會生效了，如果遲遲沒有生效，可以檢查一下是不是 [freenom] 沒有設定好，因為我遇過幾次明明設定了，但就是沒有存進去，目前還不知道是為什麼，但再重新設定一次就好了。

![setup NS for freenom][setup-ns-on-freenom04]

下圖是使用 nslookup 驗證的結果

![nslookup for DNS][nslookup-dns-reuslt]

[freenom]: http://www.freenom.com
[dnssec]: https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F%E5%AE%89%E5%85%A8%E6%89%A9%E5%B1%95

[00-request-dns-01]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_00_request_new_domain01.png
[00-request-dns-02]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_00_request_new_domain02.png
[00-request-dns-03]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_00_request_new_domain03.png
[00-request-dns-04]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_00_request_new_domain04.png
[00-request-dns-05]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_00_request_new_domain05.png
[00-request-dns-06]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_00_request_new_domain06.png
[00-request-dns-07]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_00_request_new_domain07.png
[00-request-dns-08]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_00_request_new_domain08.png
[00-request-dns-09]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_00_request_new_domain09.png
[01-enter-cloud-dns]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_01_click_cloud_dns.png
[02-create-domain]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_02_select_create_domain.png
[03-dns-domain-created]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_03_create_dns_domain.png
[04-add-dns-record00]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_05_add_record_00.png
[04-add-dns-record01]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_05_add_record_01.png
[04-add-dns-record02]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_05_add_record_02.png
[05-show-dns-detail]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_06_dns_detail.png
[setup-ns-on-freenom01]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_07_setup_ns_01.png
[setup-ns-on-freenom02]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_07_setup_ns_02.png
[setup-ns-on-freenom03]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_07_setup_ns_03.png
[setup-ns-on-freenom04]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_07_setup_ns_04.png
[nslookup-dns-reuslt]: https://raw.githubusercontent.com/Pajace/Learn_GCP/master/images/01_cloud_dns/learn_gcp_cloud_dns_08_nslookup_dns.png
