# ROS 1與ROS 2之間的差異

[原文連結](http://design.ros2.org/articles/changes.html)

本文概觀了ROS 2之於ROS 1所做的改變

原文作者: [Dirk Thomas](https://github.com/dirk-thomas)

## 前言

針對已經對ROS 1熟悉的讀者, 這裡的內容在能提供足夠背景及理論基礎的前提下, 儘量以簡要為原則. 而如果有外部資訊, 也會提供連結.

```
內容中如果有提到尚未提供的功能, 會以⏳標示
```

## 平台及相依性 (Platforms and dependencies)

### 平台

ROS 1僅在Ubuntu上做持續整合 (CI, continuous integration)測試. 其他Linux平台及OS X的支援由社群資源提供

ROS 2目前支援包含Ubuntu Xenial, OS X El Capitan以及Windows 10等平台, 並在上述平台上做持續整合測試(參照[[ci.ros2.org](http://ci.ros2.org/)]).

### 程式語言

#### C++標準版本

ROS 1以C\++03為其核心, 在其API並沒有利用C\++11的功能. ROS 2則廣泛運用了C\++11標準, 並部分運用了C\++14的版本功能. 未來如果C\++17在其他主要平台上也支援的話, ROS 2或許也會納入

#### Python

ROS 1主要支援Python 2. ROS 2則支援Python 3.5以上版本

### 運用現有中介軟體(middelware)
ROS 1使用自定義的序列化格式, 傳輸協定及中心發掘機制. ROS 2則是透過抽象化中介軟體介面([abstract middleware interface](http://design.ros2.org/articles/ros_middleware_interface.html))來支援序列化, 傳輸及發掘機制. 目前所有的介面皆是透過DDS標準來實現. 這讓ROS 2能夠提供在各種不同的網路上改善通訊品質的服務質量作為([Quality of Service policies](http://design.ros2.org/articles/qos.html))

## 編譯系統

更多關於編譯系統的資訊, 請參照此關於[ament](http://design.ros2.org/articles/ament.html)
的文章

### 除CMake外, 亦支援其他編譯系統

所有的ROS package皆可視為一CMake專案. ROS 2也支援其他編譯系統. 目前除CMake外, 編譯工具也支援單純的Python package

### Python packages

ROS 1中, setup.py檔案是以CMake的自定義邏輯處理, 因此包含Python程式碼的ROS package只能使用一部分setup.py檔案中的功能子集. 而在ROS 2中, 由於Python package可以透過`python3 setup.py install`呼叫, 因此可完整地運用setup.py檔案, 例如:entry points.

### 環境配置

在ROS 1中, 在使用編譯完成的ROS packages前, 必須透過source指令使用編譯工具產生的腳本(scripts)來配置運行環境. 這種作法只能在ROS packages是以ROS特定的編譯工具來進行編譯的情形才成立.

在ROS 2中, 環境配置分成套件特定腳本(package-specific scripts)及工作區特定腳本(workspace-specific scripts).每個package都具備了讓各自在編譯後可運作的腳本(scripts). 編譯工具僅呼叫工作區特定腳本(workspace-specific scripts), 再讓該腳本來呼叫套件特定腳本(package-specific scripts)

### 沒有非獨立編譯

在ROS 1中, 多個packages可在單一個CMake狀況下編譯. 這可以讓編譯的速度更快, 每個package也必要確認各package間的相依性定義正確. 此外, 同個namespace下的packages也會有名稱衝突的問題

在ROS 2中, 只支援獨立的編譯, 也就是說, 每個package都單獨編譯. 而安裝的空間可以是獨立或是合併的

### 沒有 devel space

在ROS 1, package可以在沒有安裝的情形下編譯. 在搭配source space的devel space中, 系統就可以運作. 但每個package都必須支援該devel space, 例如: enviroment hooks及CMake程式碼

而在ROS 2, package在編譯後必須先安裝, 才能使用

devel space在ROS 1存在的一個理由是讓開發者能修改檔案, 例如Python及lauch程式檔, 而能在未經重新編譯的情形下, 使用這些修改的程式檔. ROS 2則是以軟連結(symlinks)取代安裝步驟中的複製操作來達到相同的效果.

### 支援catkin_simple的使用

在ROS 1中, catkin_simple的目的在於簡化ROS package中的CMake程式撰寫. 但在很多時候, 由於要符合一些像是支援devel space功能的設計需求限制下, catkin_somple並沒有達到原有的目的.

在ROS 2, 則針對這樣的使用案例, 重新架構了CMake API

### 對於無描述資料(manifest)的package的基本支援

在ROS 1中, 編譯系統只會處理具有描述資料檔(manifest files)的packages. 在ROS 2, 編譯系統是可以查找出無描述資料的package. 如果package是依慣常寫法寫成, 系統甚至可能偵測出遺漏的描述資料(像是相依性, dependencies)

## Messages, Services

詳細資訊可參考[ROS interface definition](http://design.ros2.org/articles/interface_definition.html)

### C\++命名空間(namespaces)的劃分

在ROS 1, .msg及.srv檔案可以有相同的檔名, 但是會造成程式碼衝突. 相同的情形也會發生在services的request及response程序中.

在ROS 2, 產生的程式碼會劃分彼此的命名空間, 來避免衝突

### Python中名稱相同

在ROS 1及ROS 2, 在service及message中產生的Python程式碼都使用相同的module及class名稱. 因此他們無法在同一應用中載入. 這樣的作法如果有需要或許會重新考量.

### Message定義允許預設值設定

在ROS 2中, 建立message時可以對基本資料型態設定預設值. 不過在非基本型態資料(non-primitive fields), 例如字串陣列及巢狀訊息(array of strings, nested message), 目前還不適用(⏳).

### 可設定陣列及字串上限

對於資料長度非固定的message, 如此便能計算並預先配置所需的最大記憶體空間. 這對於系統效能提升及其他像是有實時要求的應用案例都有好處.

### 統一的時程(duration)及時間(time)類型

ROS 1, 時程及時間都在客戶端函式庫(client libraries)中定義. 這些資料的名稱在C\++(sec, nsec)及Python(secs, nsecs)中也不一樣.

ROS 2把這此資料類型定義成meessage, 因此名稱在各程式語言中都統一

### 將表頭訊息(Header message)中的sequence欄位刪除

此欄位棄用已久, 在ROS 1中的設置也不一致

## 客戶端函式庫(Client libraries)

### 跨程式語言
#### Topic 命名空間(⏳)

目前由於DDS有效字元的限制, ROS 2並沒有支援topic的命名空間. [這篇文章](http://design.ros2.org/articles/topic_and_service_names.html)討論了未來加入的作法.

#### 通知
ROS 1中關於ROS graph的資訊都必需透過master取得. 而ROS 2可透過publish的方式針對變動發出通知. 例如: 參數變動時可發送通知

#### 元件生命週期
在ROS 1中每個節點都有其主要函式. 而ROS 2則是建議從週期性的元件中繼承.

這樣的週期可便於像是roslaunch類型的工具來啟動多個元件組成的系統(⏳).

詳細資訊可參照[node life cycle](http://design.ros2.org/articles/node_lifecycle.html).

#### 參數及動態配置
在ROS 1中全域參數(global parameters)及節點動態配置參數(node-specific dynamic reconfigure parameters)是兩種不同的概念. ROS 2則統一了作法. 以類似動態配置的作法, 利用一個名為"全域參數伺服器(global parameter server)"(⏳)的節點來接收並設定參數值. ROS 1中以查看的方式來取得參數變動資訊, 而ROS 2則以"publish"方式來通知其他個體相關的變動資訊.

詳細資訊可參照[parameter design](http://design.ros2.org/articles/ros_parameters.html)

#### Actions(⏳)
ROS 2目前沒有actions的概念. 往後會加入一種結合先佔式伺服(preemptible)及回饋發佈器(feedback publisher)的作法

#### 執行緒
ROS 1開發者只能從單一執行緒或多執行緒作法擇一. ROS 2則可在C\++中使用粒化執行(granular execution)的作法, 而客製化執行器的運用也更容易. 不過在Python的執行模式則尚末建立.

#### ROS Graph
ROS 1中, nodes跟topics對應只能在啟動期間來描繪. ROS 2的重新描繪機制尚未建立(⏳). 但目標是在啟動期間(startup time)及執行期間(runtime)都可以使用的描繪(remapping)及混疊(aliasing)功能.

### C及C\++
#### 支援 real-time

ROS 1並不支援real-time的程式碼, 而是仰賴像是Orocos的外部架構來達到類似要求. 而在ROS 2中, 如果使用適合的RTOS並有周詳的軟體搭配, 是可以實現real-time節點的

### C\++
#### Node vs. Nodelet
ROS 1中nodes及nodelets的API是不同的, 開發者必須在編寫程式時決定nodes與執行程序的對應關係. 在ROS 2中, 建議將個別的元件編譯成共同的函式庫, 再將其載入個別程序或是將同一程序與其他元件共享(如同ROS 1的nodelets). 如此在配置期間就可以擇定程序規劃.

#### 允許同一程序中多個nodes
ROS 1中, 在單一程序中不能有多個nodes. 這導因於API本身及內部建置時的決定. 而在ROS 2則允許單一程序的多個nodes

## 工具
### roslaunch(⏳)
在ROS 1中, roslaunch檔在XML中定義, 而僅有基本的功能. 在ROS 2中, roslaunch是以Python撰寫, 因此可以使用像是判斷式等較複雜的邏輯. 目前的狀態是僅提供較少的功能來做多程序的測試功能.

## 資源查找(Resource lookup)
ROS 1中不同的資源(packages, messages, plugins...等)是基於ROS\_PACKAGE\_PATH來查找檔案系統. 而當ROS\_PACKAGE\_PATH太龐大時, 這樣的作法會造成系統的效能低落.

在ROS 2中, 資源會在編譯時生成索引, 而在執行時有效地查詢. 更多資訊可參照[resource index](https://github.com/ament/ament_cmake/blob/master/ament_cmake_core/doc/resource_index.md)
文件. 

## Packaging
### ABI 版本化
ROS 1會重新編譯所有下游packages, 因為其預設ABI之間不相容. 而為了降低系統負擔(overhead), ROS 2的package可以宣告其ABI來儘量降低下游packages的重新編譯

### Windows的binary packages
ROS 1只能編譯來自Windows的源碼(source), 這還只在少數ROS套件, 且無技術支援的情形下提供. ROS 2則會提供基於Chocolatey的binary package














