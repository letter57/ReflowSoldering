# ReflowSoldering

## 為什麼要做這個專案
* 以前如果要手焊PAD在元件底部的，老實說有難度.如果要焊這種元件，一般都是要開鋼板跟買加熱平台.在台灣製作鋼板及買加熱平台的話，花下來的金額也不少.在社群看到有前輩分享，而且成本也很低.本著MAKER的精神，也想自己嘗試做看看.

## 設計脈絡
* 製作這個加熱平台，我希望可以有個Display，用來顯示目前溫度及按鍵來控制選單 & 調整溫度. 如果用按鍵的話，至少要三個按鍵，這...太佔版面，於是發現有五向按鈕，解決了至少要三個按鍵的問題.但操作上好像沒那麼順暢.於是看到有人使用旋轉編碼器，操作上感覺更直覺.
* 溫度偵測有PTC & 熱電偶，PTC可量測的溫度大約300 ~ 350度C. K Type 熱電偶可量測的溫度可達1000度C左右，在此選擇 K Type 熱電偶的原因是希望可相容較多的加熱板.
* 加熱板的控制一般有繼電器 及 SSR. 這裡選用SSR，原因是如果控制較頻繁，也不會聽到嗲嗲的聲音.

## 硬體方面
* 大腦的核心，我選擇使用Nordic 的 [nRF52840](https://shopee.tw/nRF52840-%E9%96%8B%E7%99%BC%E6%9D%BF-i.26640381.23644275212?sp_atk=af0a1c73-030b-4a82-b1d5-a3a14a383f4f&xptdk=af0a1c73-030b-4a82-b1d5-a3a14a383f4f)，主要是為了讓自己更熟悉這顆晶片，另一方面也是工作上的需求.
* [顯示屏](https://shopee.tw/ST7735-1.8%E5%90%8B-128-x-160-%E8%BF%B7%E4%BD%A065K%E5%85%A8%E5%BD%A9-IPS%E6%B6%B2%E6%99%B6%E8%9E%A2%E5%B9%95-SPI%E9%80%9A%E8%A8%8A-TFT-LCD%E9%A1%AF%E7%A4%BA%E5%99%A8-i.656213378.16646367723?sp_atk=32532fe5-c0e8-4631-abc3-5c7d1c5ff266&xptdk=32532fe5-c0e8-4631-abc3-5c7d1c5ff266)我選擇1.8吋的IPS屏，會選擇IPS屏主要是希望像我一樣，360度無死角.
* 溫度偵測我使用[MAX6675 + K Type 熱電偶](https://shopee.tw/MAX6675-K%E5%9E%8B%E7%86%B1%E9%9B%BB%E5%81%B6-%E6%BA%AB%E5%BA%A6%E6%B8%AC%E9%87%8F%E6%A8%A1%E7%B5%84-SPI%E9%80%9A%E8%A8%8A-%E9%99%84K%E5%9E%8B%E7%86%B1%E9%9B%BB%E5%81%B6-i.656213378.15647588556?sp_atk=dd1b2b24-ee44-4bc8-a935-d48bbbb22976&xptdk=dd1b2b24-ee44-4bc8-a935-d48bbbb22976)，會選擇這個原因是因為可偵測的溫度範圍較廣，再加上Zephyr的driver(SENSOR_CHAN_AMBIENT_TEMP)有支援.
* 操作方面，我希望簡單直接一點，所以我選用[旋轉編碼器](https://shopee.tw/EC11-360%E5%BA%A6%E9%80%A3%E7%BA%8C%E6%97%8B%E8%BD%89-%E7%B7%A8%E7%A2%BC%E5%99%A8%E6%A8%A1%E7%B5%84-%E6%95%B8%E4%BD%8D%E8%A8%8A%E8%99%9F-D%E5%AD%97%E9%A0%AD-%E9%99%84%E4%B8%8B%E5%A3%93%E6%8C%89%E9%88%95%E6%97%8B%E8%BD%89%E7%B7%A8%E7%A2%BC%E5%99%A8-KY-040-i.656213378.15847628322?sp_atk=33de71a6-a8f4-4521-9540-e4a7d6687c94&xptdk=33de71a6-a8f4-4521-9540-e4a7d6687c94).同時Zephyr的driver(SENSOR_CHAN_ROTATION)也有支援.

## 軟體方面
* 這整個系統我使用了[Zephyr RTOS](https://www.zephyrproject.org/)來幫我管理，各線程的溝通使用IPC.
* MAX 6675 及 旋轉編碼器都是使用Zephyr支援的Sensor driver.
* 顯示的部分則使用開源的[LVGL](https://lvgl.io/)，這部份Zephyr也有整合好了，搭配shield讓你簡單到懷疑人生.UI的部份搭配NXP的[GUI Guider](https://www.nxp.com/design/design-center/software/development-software/gui-guider:GUI-GUIDER).這部分也可以使用[SquareLine Studio](https://squareline.io/).
* ncs版本: 2.5.2

## 實作過程
### UI規劃
* 上電會有開機畫面，開機畫面所呈現icon有Nordic、Zephyr、LVGL、WFEGO，2秒後會進入主選擇.   
* 主選單的icon有『顯示目前偵測到的溫度及目標溫度』、『設定目標溫度』、『目前溫度曲線圖』、『Info』，可用旋轉編碼器來選擇.
* 『顯示目前偵測到的溫度及目標溫度』: 顯示目前max 6675偵測到的溫度，取到小數點第二位.溫度太高或太低可以調整旋轉編碼器來達到目標溫度.
* 『設定目標溫度』: 預設溫度為150度，如果每次都要180度的話，每次都要去旋轉編碼器很麻煩，這裡使用NVS來記錄設定溫度值.
* 『目前溫度曲線圖』: 將偵測到的溫度繪製成曲線，由於顯示屏不夠大，顯示效果有限.
* 『Info』: 描述藍牙Mac Address及韌體版本.   
  * 到時也可實作將偵測到的溫度透過藍牙傳送出來，可用來觀察溫度的變化.

### 實作過程所遇到的問題
* 由於有使用到浮點數，所以需要開啟`浮點數的config`.
* 當開啟藍牙功能又使用NVS時，會發生讀寫失敗，但卻回應成功.解法是定義使用者的Partition.
* max 6675最快的polling time為220ms，這部份要注意一下.
* 旋轉編碼器一開始沒辦法控制的很好，主要是polling的時間，這個時間要快一點，目前使用2ms.旋轉編碼器轉太快會造成顯示畫面跟不上，這邊有做一個限制，第一筆資料與第二筆資料時間在200ms以內，忽略第二筆.這部份還有改善的空間.

## [Demo](https://youtu.be/ZhiblBvzqyQ)

## 結論
* 如果只是想要加熱什台的話，直接淘寶買一個，因為自己做的話，其實沒有便宜太多，如果要練功的話，我覺得蠻不錯的.
* 如果想要使用Zephyr練功的話，非常推薦使用Nordic的晶片.
  * [開發工具齊全](https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-Desktop).
  * 有[教學文件](https://academy.nordicsemi.com/).
  * 有問題的話，可以上Nordic的[Devzone](https://devzone.nordicsemi.com/)，大部份都可以解決問題.

  
