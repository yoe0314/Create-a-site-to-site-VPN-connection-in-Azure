
# Create a site-to-site VPN connection in Azure

###### tags:`Azure Network` `VPN` 

[toc]

本篇內容將透過Azure Portal來建立site-to-site VPN連線。

![](https://i.imgur.com/MpsjF35.png)

## 1/ 建立VNet、Subnet
* 建立虛擬網路、子網路，可以新建一個資源群組存放此次建立S2S VPN的相關資源。
![](https://i.imgur.com/wclMlsZ.png)
```
假設VNet網段為：
192.168.0.0/22

Subnets網段為：
192.168.0.0/26 - HubSubnet01
192.168.0.64/26 - AzureBastionSubnet
192.168.0.128/26 - GatewaySubnet
192.168.1.0/26 - AppSubnet01
192.168.2.0/26 - DBSubnet01
```
* `虛擬網路IPv4位址空間`輸入192.168.0.0/22。
* `新增子網路`輸入上述5個Subnets。
![](https://i.imgur.com/rlBZdx3.png)


## 2/ 建立VPN Gateway

* 為VPN Gateway取一個`名稱`。
* `閘道類型`選擇VPN。
* `SKU`選擇VpnGw1。
    >  建立虛擬網路閘道時，根據工作負載、輸送量、功能和SLA的類型，選取符合需求的 SKU。[查看 Gateway SKUs](https://docs.microsoft.com/zh-tw/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#gwsku)
* `虛擬網路`選擇剛才建立的VNet。
* 為VPN Gateway建立一個`公用IP位址`。
* 填完相對應資訊後點選建立。
![](https://i.imgur.com/NWFXiGO.png)

:::warning
:warning: **注意 !**
* 建立VPN Gateway通常可能需要**40分鐘左右或者更久**，視選取的**Gateway SKU**而定。
* 等待的同時我們可以先去建立Local Network Gateway。
:::

## 3/ 建立Local Network Gateway
* 上方Search bar搜尋區域網路閘道或Local Network Gateway。
![](https://i.imgur.com/arYk1kg.png =80%x)
* 在建立Local Network Gateway的時候需要地端的一些資訊，例如：地端VPN設備的對外IP、地端要連線上雲端的IP網段。
:::warning
:warning: **注意 !** 
* 地端的網段不能和雲端的網段重疊。
* [建立VPN連線必要條件。](https://docs.microsoft.com/zh-tw/azure/vpn-gateway/tutorial-site-to-site-portal#prerequisites)
:::

```
假設地端VPN設備對外IP為：
1xx.xx.xx.125

地端IP網段為：
10.xx.xxx.x/24
10.xx.xxx.x/24
10.xx.xxx.x/24
10.xx.xxx.x/24
10.xx.x.x/16
10.xx.x.x/16
192.xxx.x.x/24
```
* `IP位址`填入地端VPN設備對外IP。
* `位址空間`填入地端要連上雲端的IP網段。
![](https://i.imgur.com/DisFh23.png =90%x)

## 4/ 設定地端VPN設備
* 設定地端 VPN 設備時，需要下列值：
    * ==共用金鑰==：在建立S2S VPN Connection的時候也需要相同的共用金鑰。 
    * ==VPN Gateway的Public IP==。
    ![](https://i.imgur.com/EZrJnWk.png)
    * >:memo: [**設定VPN設備**](https://docs.microsoft.com/zh-tw/azure/vpn-gateway/tutorial-site-to-site-portal#VPNDevice)

## 5/ 建立Connection
* 上方Search Bar搜尋`連接`。
![](https://i.imgur.com/o7S6VDr.png =70%x)

* 基本
    * `連接類型`選擇`站對站(IPSec)`。
![](https://i.imgur.com/1fLNrZ6.png =90%x)
* 設定
    * `虛擬網路閘道`選擇剛才建立的VPN Gateway。
    * `區域網路閘道`選擇剛才建立的Local Network Gateway。
    * `共用金鑰(PSK)`輸入在地端VPN設備設定的金鑰。
    * 其餘選項使用預設即可。
![](https://i.imgur.com/dz2blpi.png =90%x)

* Connection建立完後，若地端VPN設備沒有同步設定完成，則Connection狀態會顯示`未連接`，`連入資料`和`連出資料`都會是0B。
![](https://i.imgur.com/CduUgkC.png)

* 地端VPN設備有同步設定成功的話，Connection狀態會顯示`已連接`，且`連入資料`和`連出資料`過一段時間會開始有流量顯示。
![](https://i.imgur.com/X1aOzVX.png)

:::info
:memo: **測試連線**
* Connection設定成功後，可以建立一台VM去Ping地端的機器是否可以Ping通，來驗證我們建立的S2S VPN服務是否正常。
:::

## 6/ 建立S2S VPN所需創立的資源
* 可以在`資源群組`中看到我們建立S2S VPN所需的資源有哪些。
![](https://i.imgur.com/pAmqMO9.png)
    - [x] ==VNet、Subnet==
    - [x] ==VPN Gateway==
    - [x] ==VPN Gateway Public IP==
    - [x] ==Local Network Gateway==
    - [x] ==Connection==
:::info
:bulb: **提醒**
* 如果只是嘗試建立S2S VPN的話，測試完畢後，記得將資源群組刪除，才不會有額外的費用產生。
:::
## Reference
* [在Azure入口網站中建立站對站VPN連線](https://docs.microsoft.com/zh-tw/azure/vpn-gateway/tutorial-site-to-site-portal)
* [使用Azure入口網站建立和管理VPN閘道](https://docs.microsoft.com/zh-tw/azure/vpn-gateway/tutorial-create-gateway-portal)

