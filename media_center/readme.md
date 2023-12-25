[Home](https://www.leishi.io/)[Blog](https://www.leishi.io/blog)[About](https://www.leishi.io/about)

利用NAS实现全自动观影追剧
==============

30 December, 2021

想象这样一个场景：

> 打开一个网页，点开搜索框输入想看的电影/电视剧，点一个按钮，过几分钟对应的高清资源已经呈现在你的Emby/Plex影视库中，刮削完毕，字幕配好，你可以随时在家里的大屏幕或手机的小屏幕上欣赏

观影、追剧、刮削、字幕，全部自动化，本文就教你如何在NAS上实现这一套流程。

  

*   [基本流程简介](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%9F%BA%E6%9C%AC%E6%B5%81%E7%A8%8B%E7%AE%80%E4%BB%8B)
*   [名词解释](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%90%8D%E8%AF%8D%E8%A7%A3%E9%87%8A)
    *   [1\. BT/PT/Usenet 资源中心](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#1-btptusenet-%E8%B5%84%E6%BA%90%E4%B8%AD%E5%BF%83)
        *   [BT](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#bt)
        *   [PT](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#pt)
        *   [Usenet](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#usenet)
    *   [2\. Jackett/Prowlarr 资源索引中心](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#2-jackettprowlarr-%E8%B5%84%E6%BA%90%E7%B4%A2%E5%BC%95%E4%B8%AD%E5%BF%83)
    *   [3\. Radarr/Sonarr 资源周转中心](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#3-radarrsonarr-%E8%B5%84%E6%BA%90%E5%91%A8%E8%BD%AC%E4%B8%AD%E5%BF%83)
    *   [4\. Overserr/Ombi Radarr/Sonarr的前端](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#4-overserrombi-radarrsonarr%E7%9A%84%E5%89%8D%E7%AB%AF)
    *   [5\. Bazarr/ChineseSubFinder 资源搜索引擎](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#5-bazarrchinesesubfinder-%E8%B5%84%E6%BA%90%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E)
    *   [6\. Emby/Plex 影视库](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#6-embyplex-%E5%BD%B1%E8%A7%86%E5%BA%93)
*   [万事开头难 让Docker跑起来](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E4%B8%87%E4%BA%8B%E5%BC%80%E5%A4%B4%E9%9A%BE-%E8%AE%A9docker%E8%B7%91%E8%B5%B7%E6%9D%A5)
    *   [Docker简介](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#docker%E7%AE%80%E4%BB%8B)
    *   [安装Docker](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85docker)
    *   [安装docker-compose](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85docker-compose)
    *   [docker-compose基本操作](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#docker-compose%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C)
*   [你已经成功大半了 准备工作](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E4%BD%A0%E5%B7%B2%E7%BB%8F%E6%88%90%E5%8A%9F%E5%A4%A7%E5%8D%8A%E4%BA%86-%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)
    *   [准备docker-compose配置文件](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%87%86%E5%A4%87docker-compose%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
    *   [准备影视库文件夹和下载目录](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%87%86%E5%A4%87%E5%BD%B1%E8%A7%86%E5%BA%93%E6%96%87%E4%BB%B6%E5%A4%B9%E5%92%8C%E4%B8%8B%E8%BD%BD%E7%9B%AE%E5%BD%95)
    *   [站住警察 准备好你的ID](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E7%AB%99%E4%BD%8F%E8%AD%A6%E5%AF%9F-%E5%87%86%E5%A4%87%E5%A5%BD%E4%BD%A0%E7%9A%84id)
    *   [回顾](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%9B%9E%E9%A1%BE)
*   [准备你的影视库](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%87%86%E5%A4%87%E4%BD%A0%E7%9A%84%E5%BD%B1%E8%A7%86%E5%BA%93)
*   [安装下载器](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85%E4%B8%8B%E8%BD%BD%E5%99%A8)
    *   [启动下载器](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%90%AF%E5%8A%A8%E4%B8%8B%E8%BD%BD%E5%99%A8)
    *   [配置qBittorrent](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEqbittorrent)
    *   [配置nzbget](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEnzbget)
    *   [回顾](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%9B%9E%E9%A1%BE-1)
*   [Prowlarr/Jackett](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#prowlarrjackett)
    *   [安装Prowlarr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85prowlarr)
    *   [配置Prowlarr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEprowlarr)
    *   [安装Jackett](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85jackett)
    *   [配置Jackett](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEjackett)
    *   [（可选）配置flaresolverr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%8F%AF%E9%80%89%E9%85%8D%E7%BD%AEflaresolverr)
    *   [回顾](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%9B%9E%E9%A1%BE-2)
*   [Radarr/Sonarr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#radarrsonarr)
    *   [安装Radarr/Sonarr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85radarrsonarr)
    *   [配置Radarr/Sonarr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEradarrsonarr)
        *   [配置刮削](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AE%E5%88%AE%E5%89%8A)
        *   [配置下载器](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AE%E4%B8%8B%E8%BD%BD%E5%99%A8)
            *   [配置远程文件映射](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AE%E8%BF%9C%E7%A8%8B%E6%96%87%E4%BB%B6%E6%98%A0%E5%B0%84)
        *   [配置Indexer](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEindexer)
            *   [Prowlarr配置](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#prowlarr%E9%85%8D%E7%BD%AE)
            *   [Jacker配置](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#jacker%E9%85%8D%E7%BD%AE)
        *   [配置媒体目录](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AE%E5%AA%92%E4%BD%93%E7%9B%AE%E5%BD%95)
        *   [恭喜配置完成](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E6%81%AD%E5%96%9C%E9%85%8D%E7%BD%AE%E5%AE%8C%E6%88%90)
*   [配置Overseerr/Ombi](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEoverseerrombi)
    *   [安装Overseerr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85overseerr)
    *   [配置Overseerr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEoverseerr)
    *   [安装Ombi](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85ombi)
    *   [配置Ombi](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEombi)
    *   [使用Overseerr/Ombi](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E4%BD%BF%E7%94%A8overseerrombi)
*   [安装字幕下载器](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85%E5%AD%97%E5%B9%95%E4%B8%8B%E8%BD%BD%E5%99%A8)
    *   [安装Chinesesubfinder](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85chinesesubfinder)
    *   [配置Chinesesubfinder](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEchinesesubfinder)
    *   [安装Bazarr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E5%AE%89%E8%A3%85bazarr)
    *   [配置Bazarr](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#%E9%85%8D%E7%BD%AEbazarr)

基本流程简介
------

[![flow](https://www.leishi.io/static/8a26eae7c1dabc38b4224546ac504645/72e01/flow.jpg "flow")](https://www.leishi.io/static/8a26eae7c1dabc38b4224546ac504645/0d465/flow.jpg)

整套系统的流程如上图，左上角是输入端，用户经由`Overserr`/`Ombi`来记录下想看的电影/电视剧，`Overserr`/`Ombi`作为整套系统的前端，会自动将请求录入`Radarr`/`Sonarr`。资源录入后，`Radarr`/`Sonarr`作为中间站，会从Indexer -- `Jackett`/`Prowlarr`搜索对应的资源，然后将下载任务推送到下载器 -- `qBittorrent`/`nzbget`。待下载器下载对应的资源后，`Radarr`/`Sonarr`会自动生成硬连接，刮削好nfo，并通知观影前端 -- `Emby`/`Plex`资源已经就绪，于此同时，字幕自动下载工具 -- `Bazarr`/`Chinesesubfinder`会自动扫描目录，并下载好对应的字幕。

  
被这一堆名词吓到了？没事，下面我们将一步步讲解，并配置好所有的工具，让这套系统运行起来！

> **注意**：本文默认读者已经拥有NAS或者能7/24运行的大容量PC设备/云端服务器，会用ssh并且具备利用命令行运行软件、修改文件的能力，同时具有影视库（Emby/Plex等）搭建的能力。

> **注意2**：本文不解决任何软件的汉化问题

> **注意3**：本文不解决Docker/TMDB等的国内访问问题

> **注意4**：如果无意或者不想使用Docker，那么本文不适合你

> **注意5**：本文不解决任何影视资源获取的问题，但聪明的你肯定能从本文找到如何获取大量影视资源的入口

名词解释
----

如果你已经熟悉流程图中出现的各种名词，并且大概明白它们各自是什么、做什么，那么你可以跳过本章节。

### 1\. BT/PT/Usenet 资源中心

#### BT

相信大部分人对[BT](https://zh.wikipedia.org/wiki/BitTorrent_(%E5%8D%8F%E8%AE%AE))并不陌生，相较于中心化的下载服务器，BT利用P2P（Peer-to-Peer）技术，使每个在P2P网络中的用户都成为了资源的提供方；而资源则以分块的形式，散落在每个人的硬盘中，除非拿到种子文件，否则任何人都不（太）可能从网络中拿到完整的文件。正是这些特性，使得BT自一发布就流行至今（也成了盗版资源传播的高速公路）。流程图中的`qBittorrent`即是一款BT下载器，另外比较常用的还有`Transmission`/`Deluge`/`uTorrent`/`aria2`等（除aria2外，其余软件均也是PT常用的下载器）

比较常用的公网BT影视资源站有：

*   [Pirate Bay](https://thepiratebay.org/index.html)，大名鼎鼎的海盗湾
*   [RARBG](https://rarbgmirror.org/)，主要是影视资源，质量参差不齐
*   [Nyaa](https://nyaa.si/)，主要是动漫类资源，追番必备
*   [dmhy](https://dmhy.org/)，动漫花园，同上

#### PT

PT（Private Tracker）在技术与BT并无本质不同，唯一的区别是，BT连接的是公开的Tracker，而PT通常需要邀请注册，并且用自己密钥才能连上Tracker。

PT相比BT，入门有一定的门槛，高质量的站点要么只通过熟人间口口相传，要么需要捐赠注册，价格从几十到几百不等。拥有帐号后还需要一定的设备和动手能力来保持帐号的活跃。

新入门PT建议从[PT吧](https://tieba.baidu.com/f?kw=pt)或者[Reddit](https://www.reddit.com/r/trackers/)起步，从开放注册的PT站点或者有人大量发放的新手站点玩起，慢慢熟悉规则，再逐步进入更好的站点。

新手可以从以下站点起步：

*   [hd.ai](https://www.hd.ai/signup.php)，常年开放注册
*   [HDAtmos](https://hdatmos.club/)，PT吧内有大量邀请
*   [HDDolby](https://www.hddolby.com/signup.php)，要是有一定的PT知识，可以答题入站

顶级站点

*   [皮](https://passthepopcorn.me/)[妞](https://broadcasthe.net/)[堡](https://hdbits.org/)

#### Usenet

[Usenet](https://zh.wikipedia.org/wiki/Usenet)是早期的一种互联网形式，它甚至比现在的互联网[www](https://zh.wikipedia.org/wiki/World_Wide_Web)更早。在早期的语境中，Usenet一般指[新闻组](https://zh.wikipedia.org/wiki/%E6%96%B0%E9%97%BB%E7%BB%84)，类似于BBS论坛。相对于Usenet的讨论组性质，在本文中提到的Usenet，注重的是利用Usenet网络，存储和分发大容量二进制内容（即盗版影视资源）的特性。

相对于BT资源的良莠不齐，PT的高不可攀，Usenet兼具高质量资源和低门槛（有不少人不间断地从topsite/PT下载资源上传到Usenet），你只要有钱就行，并且也不是很多钱（大约$30+/年），同时还不需要像PT一样维护帐号活跃/长时间做种。

中文语境下很少有Usenet的相关内容，如果你有兴趣了解，可以去[阮一峰的Blog](https://www.ruanyifeng.com/blog/usenet/)或者[Reddit](https://www.reddit.com/r/usenet/wiki/index)学习和讨论。事实上，如果仅从影视资源角度，Usenet上的资源比任何一个顶级PT都要多、全，甚至有不少玩PT的人，使用Usenet完全是为了下载上面的资源再上传到PT上去；Usenet另一大优势是极快的下载速度，不像BT/PT，资源的下载速度严重依赖于做种人数的多少，Usenet的资源存在服务器上，几乎任何时刻你都可以跑满你的大部分带宽。

想要玩转Usenet，你需要3部分来写共同协作来下载资源：

*   `Indexer`，类似于种子搜索网站，提供下载Usenet资源需要的[nzb](https://en.wikipedia.org/wiki/NZB)文件。Indexer有免费的，有收费的，也有类似于PT一样进入门槛很高的，本文不作推荐，新手可以从免费的玩起，熟悉流程后再寻找质量更高的Indexer。你可以从[这个链接](https://www.reddit.com/r/usenet/wiki/indexers)起步寻找适合你的Indexer。
*   `Provider`，负责提供连接Usenet的帐号。Usenet的内容存储是分布式的，但是你需要一个Provider才能连上Usenet网络，Provider大部分收费，月付费$3-$5左右，这是使用Usenet最大的花销。你可以从[这个链接](https://www.reddit.com/r/usenet/wiki/providers)起步寻找适合自己的Provider。
*   nzb下载软件，主要推荐是用[nzbget](https://nzbget.net/)和[sabnzbd](https://sabnzbd.org/)

### 2\. Jackett/Prowlarr 资源索引中心

有了资源获取渠道，那自然有专门的软件来提供一个统一、规范的方式来搜索、索引这些资源，这被称为`Indexer`（对，Usenet也有个Indexer，Usenet的Indexer索引Usenet的资源，这个Indexer索引BT/PT/Usenet的资源）。[Jackett](https://github.com/Jackett/Jackett)/[Prowlarr](https://github.com/Prowlarr/Prowlarr)是最常用的两个Indexer，你可以按自己的喜好随便选择一个，但是Prowlarr具有同步配置到`Radarr`/`Sonarr`的能力，如果你同时使用大量的Indexer，使用上会方便很多。

### 3\. Radarr/Sonarr 资源周转中心

[Radarr](https://radarr.video/)/[Sonarr](https://sonarr.tv/)，一个负责电影，一个负责电视剧，是整套系统能够全自动的关键。它们整合了Indexer，拥有自动搜索资源/追剧的能力（它们自己也拥有一定的Indexer能力，如果你只使用Usenet或者拥有它们支持的PT站点的帐号，你可以不需要独立的Indexer）；整合了下载器，当找到了资源时，自动推送到下载器中下载资源；当下载完成时，自动刮削供影视库使用并调用API来更新影视库内容；同时，你还可以配置各种通知手段，来提醒你电影下好了/剧更新了等等。

### 4\. Overserr/Ombi Radarr/Sonarr的前端

`Radarr`/`Sonarr`虽好，但是是两个独立的软件，要想体验不割裂，我们还需要一个统一的前端来整合`Radarr`/`Sonarr`。[Overserr](https://overseerr.dev/)/[Ombi](https://ombi.io/)就是这样的软件，提供了一个统一的搜索框，让你不用关心到底是Sonarr还是Radarr负责这类资源，你只需要提供个名字就好。它们同时还整合了影视库，让你知道什么样的资源已经在库中。`Overserr`/`Omni`提供了美观、简单易上手的界面使得你可以很容易把它们提供给家人们使用，想看什么电影电视剧，搜一下，点一下，整套系统就可以运作起来。

### 5\. Bazarr/ChineseSubFinder 资源搜索引擎

网上的资源很多并不自带中文字幕，[Bazarr](https://github.com/morpheus65535/bazarr)/[ChineseSubFinder](https://github.com/allanpk716/ChineseSubFinder)就是补全资源搜索整合的最后一块拼图，有了Bazarr/ChineseSubFinder，当资源下载完毕后，中文字幕会根据资源的名称自动匹配，并下载下来。

### 6\. Emby/Plex 影视库

[Emby](https://emby.media/)/[Plex](https://www.plex.tv/)（当然还有[Jellyfin](https://jellyfin.org/)/[Kodi](https://kodi.tv/)）是大多数人开始家庭影视库搭建最常听说也最常被使用的软件。虽然略有不同，但他们都提供了解析资源文件/文件夹，利用IMDB/TMDB/TVDB等影视索引网站索引你的本地资源（即刮削），并播放的功能。如果你读到了这里，并且完全没有听说过这其中任一软件，那么我真诚建议先去尝试在自己的NAS/PC上搭建一个影视库，熟悉一下相关的各种概念和流程，然后再来本篇文章继续全自动化影视库的搭建。

万事开头难 让Docker跑起来
----------------

学习完了理论，让我们马上动手起来。本文大部分的软件都建议利用Docker运行，如果你没听过Docker，以下又是一段学习材料（笑）。如果你已经熟悉Docker和docker-compose，请再浏览下[dokcer-compose基本操作](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#docker-compose%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C)，确保你已经熟悉所有接下来会用到的操作。

### Docker简介

[Docker](https://www.docker.com/)是一款容器运行软件，通过封装和隔离，能够让众多的软件同时运行在宿主系统上，同时不妨碍/污染宿主机的环境，简单来说，Docker让普通人具备了运行/维护大型软件的能力。如果没有Docker，即使是一个有经验的专业运维人员，想要完整且正确地运行本文所提及的所有软件，恐怕也要花费大半天的时间。而利用Docker，配合上[docker-compose](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/%60https://docs.docker.com/compose/%60)，跑起整套系统的时间可以缩短到，几秒钟。

大部分时候，你拉取的都是别人写好的镜像（Image），而镜像（大部分）是从[Dockerfile](https://docs.docker.com/engine/reference/builder/)生成。你可以在[Docker Hub](https://hub.docker.com/)上找到本文提到的所有镜像，但是因为Docker的政策，很多大公司[倾向于使用自己的Docker Registry](https://www.linuxserver.io/blog/wrap-up-warm-for-the-winter)。但是国内的网络访问这些Registry速度通常不太理想，可以的情况下，尽量使用代理来拉取镜像。

拉取完镜像，当你把镜像跑起来的时候，这被称为一个容器（Container）。容器是一个隔离的系统，里面通常包含了运行你的软件所需要的最简化的依赖，以及你提供的环境变量，映射进去的Volume等等。一定情况下，你可以把容器理解成一个虚拟机（当然它们的工作原理完全不一样）。

在开始使用Docker前，有3个知识点你需要知道：

1.  `环境变量`，软件在操作系统中运作时，可以读取环境变量，人们通常用环境变量来自定义软件的各种行为，如运行监听的端口、webui的用户名/密码等等。在Docker中使用参数`-e`来定义，在docker-compose中，环境变量定义在`environments`这一栏。
2.  `端口映射`，通常你在Docker或者docker-compose的配置中会看到形如`8000:8000`这样的配置，这项配置冒号左边代表在宿主机上打开的端口，冒号右边代表容器内部监听的端口，为减少麻烦人们通常在宿主机和容器内部使用同样的端口。在Docker中使用参数`-p`来定义，在docker-compose中，端口映射定义在`ports`这一栏。
3.  `挂载目录`，相对于端口，你同样可以映射宿主机的目录进入容器中，使得容器和宿主机读取同一目录，任何在容器内和宿主机上的文件操作都会反应在两端。目录挂载非常适合用来加载软件运行需要的配置文件。在Docker中使用参数`-v`来定义，在docker-compose中，挂载目录定义在`volumes`这一栏。

### 安装Docker

Docker的安装依据各种系统各有不同。成熟的NAS系统，如群晖/威联通/Unraid等，会有图形化的界面（如群晖的套件中心）来安装Docker。

Linux系统，则可以使用Docker官网的一键脚本，全自动安装

    curl -fsSL https://get.docker.com -o get-docker.sh
    sh get-docker.sh

如果是其他系统，可以根据[官方文档](https://docs.docker.com/get-docker/)自行安装，为和本文保持一致，强烈建议将本系统安装在运行商业NAS系统或Linux的机器上。

安装完成后，在命令行检查下Docker是否正常运行（是的，即使是群晖等的商业NAS系统，你仍然可以通过命令行来使用Docker）

    sudo docker run hello-world

### 安装docker-compose

[docker-compose](https://docs.docker.com/compose/)是Docker官方推出的一款统筹/运行多个容器的软件，即使你只有一个容器，我也建议把容器的配置写到docker-compose的配置（`docker-compose.yml`，下文说编辑docker-compose配置，即指编辑这个文件）中，便于修改、更新和管理。

你可以根据[官方文档](https://docs.docker.com/compose/install/)来安装`docker-compose`，如果你是群晖/Linux，也可以直接

    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose

测试一下安装是否成功

    sudo docker-compose --version

### docker-compose基本操作

docker-compose最最最基本的操作有两个：

*   启动所有的容器

    sudo docker-compose up -d

*   停止所有的容器

    sudo docker-compose down

> 所有docker-compose的操作都需要在含有\``docker-compose.yml`文件的文件夹内运行 别急，我们马上会创建

**请牢记这两个操作，你会在后面用到很多次。**

  
docker-compose其他常用的命令

你已经成功大半了 准备工作
-------------

### 准备docker-compose配置文件

首先，先创建个文件夹，这个文件夹将会存放这套系统所有的配置文件，你可以很方便地复制这个文件夹到新的系统上，继续使用这套配置。 以我为例，我比较喜欢直接放在我对应用户的HOME文件夹下，你也可以选择放在你已经开启了smb/nfs共享的文件夹下，这样你就可以从你的电脑/笔记本上访问这个文件夹，利用你喜欢的文本编辑器编辑配置，而不需要熟悉命令行下的文本编辑器了。

    cd ~ # 这行请根据你自己的位置调整
    mkdir -p media_center # media_center可以改成你自己喜欢的名字
    cd media_center
    touch docker-compose.yml # 创建docker-compose的配置文件

如前所述，`docker-compose.yml`是docker-compose的配置文件，你可以把几乎所有容器的配置都写在这一个文件里。

### 准备影视库文件夹和下载目录

然后在你存储所有影视资源的硬盘分区上，新建（如果没有的话）如下的几个文件夹

*   Downloads，用来存储所有的未整理的资源，下载器下载的所有文件将会存在这里
*   Movies，用来存储整理好的**电影**资源，Radarr/Sonarr会从Downloads文件夹里把资源硬连接到这个文件夹，另外存储刮削后的文件
*   Series，用来存储整理好的**电视剧**资源。

取决于你影视库的规划，你可以增加更多的文件夹（如新番、合集等等），或者改成其他的名字。但宗旨就是，你需要有一个**下载**文件夹，一个**电影**文件夹，和一个**剧集**文件夹。下文会以Downloads/Movies/Series这3个文件夹名字为准，请根据自己的情况自行调整你的配置。

> **注意**：如果你的NAS上有多个分区（存储空间），每个分区上都需要有这3个文件夹

> **小技巧**：如果你有多个NAS，建议把所有的文件夹共享，然后挂载（建议使用NFS）所有文件夹到同一台机器下，再在这台机器上建立这套系统

### 站住警察 准备好你的ID

Docker镜像会有权限泄漏的问题，我建议，如果可以的话，所有的镜像都是用非root用户运行。因此，我们需要得到你现运行的，非root用户的uid和gid

    id -a
    uid=1026(lei) gid=100(users) groups=100(users),101(administrators)

如上，我的uid是1026，gid是100，你可能会得到不用结果，但是请把这两个ID记在心里，稍后配置的时候使用你对应的ID

### 回顾

至此，准备工作完成，你的机器上现在需要已经安装好Docker和docker-compose，建立好3个文件夹（Downloads、Movies和Series），建立好一个配置文件夹，里面有个名为`docker-compose.yml`的空文件，并且你已经知道你现运行的用户的uid和gid。以下的所有步骤都将围绕`docker-compose.yml`这个文件进行。

准备你的影视库
-------

本文默认你已经有一定的影视库搭建能力，或者你已经有类似Emby/Plex已经搭建完成的影视库。如果你还没有搭建好的话，建议根据你的NAS系统，先去找对应的教程搭建好影视库再来。本文提供Emby/Plex的Docker搭建配置供参考，但大部分情况下，还是建议直接用宿主机搭建

> 请务必将上步操作的电影/剧集文件夹设为影视库的电影/剧集文件夹

Emby配置

Plex配置

安装下载器
-----

根据你资源的来源，你需要[BT下载器](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#bt)或者[Usenet下载器](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#usenet)，或者两者。如果是群晖/威联通等系统，一般自带的软件（套件）中心会带有这些软件，你也可以选择Docker安装。如果你已经安装并配置好你需要的下载器，你可以略过本章。本文以安装`qBittorrent`和`nzbget`为例：

首先，我们编辑`docker-compose.yml`：

    ---
    version: "3"
    services:
      # ... 其他容器配置
      
      qbittorrent:
        image: lscr.io/linuxserver/qbittorrent
        container_name: qbittorrent
        restart: unless-stopped
        network_mode: host
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
          - WEBUI_PORT=8080 # QB webui的端口，你可以换成其他端口
        volumes:
          - ./qbittorrent:/config
          - /volume1/Data/Downloads:/downloads   # 替换成你自己的下载文件夹
          - /volume2/Data2/Downloads:/downloads2 # 替换成你自己的第二个分区的下载文件夹
    
      nzbget:
        image: lscr.io/linuxserver/nzbget
        container_name: nzbget
        restart: unless-stopped
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
        volumes:
          - ./nzbget:/config
          - /volume1/Data/Downloads:/downloads   # 替换成你自己的下载文件夹
          - /volume2/Data2/Downloads:/downloads2 # 替换成你自己的第二个分区的下载文件夹
        ports:
          - 6789:6789

几个注意点：

*   注意替换配置中的`PUID`和`PGID`
*   `environment`即环境变量，可以看到在qbittorrent的环境变量中，我们定义webui的端口。
*   `volumes`即你需要挂载的目录
    *   可以看到我们为各自的配置文件新建了文件夹，并映射到了下载器的config这个存储所有配置的目录，**这两个文件夹需要另行建立**。
    *   我们把所有的下载目录也挂载了进去
*   qbittorrent的`network_mode`是`host`，这是为了ipv6和UpnP方便。

> **注意**：如果是多台NAS，建议在每台NAS上分别安装下载器，以获得最好的写入速度

### 启动下载器

在你的配置文件所在的文件夹下运行

    # 移动到你的配置文件夹下
    # cd ~ 
    # 创建下载器所需要的配置文件夹
    mkdir -p qbittorrent nzbget
    sudo docker-compose down && docker-compose up -d 

等命令运行完毕，可以分别去`NAS_IP:8080`和`NAS_IP:6789`查看下载器有没有运行成功

> qBittorrent的默认用户名密码是`admin/admin`

> nzbget的默认用户名密码是`nzbget/tegbzn6789`

### 配置qBittorrent

在qBittorrent的Web UI，点击设置，点击Web UI，然后禁用**启用跨站请求伪造(CSRF)保护**[![22 57 12 2x](https://www.leishi.io/static/1f8d3f4efe02ac96519bfb0962df48a6/2bef9/22.57.12%402x.png "22 57 12 2x")](https://www.leishi.io/static/1f8d3f4efe02ac96519bfb0962df48a6/b1584/22.57.12%402x.png)

### 配置nzbget

首先，我们确认下下载文件夹是否是我们希望以后下载所有文件的路径

点击`Settings`，然后点击`PATHS`，最后确认下`MainDir`是否是你**映射后**的文件夹[![23 07 47 2x](https://www.leishi.io/static/7eb108b97fc63cd58ec8a9c420529235/2bef9/23.07.47%402x.png "23 07 47 2x")](https://www.leishi.io/static/7eb108b97fc63cd58ec8a9c420529235/ad00e/23.07.47%402x.png)

  

然后我们配置下我们的Usenet `Provider`。

点击`Settings`，然后点击`NEWS-SERVERS`，然后根据你的实际情况填写，最主要的几个参数是`Host`、`Port`、`UserNamne`、`Password`和`Connections`，你的Provider应该会提供详细的连接教程[![23 12 56 2x](https://www.leishi.io/static/38cbf32a6d29c6464d45bb1b5ccdc109/2bef9/23.12.56%402x.png "23 12 56 2x")](https://www.leishi.io/static/38cbf32a6d29c6464d45bb1b5ccdc109/67a79/23.12.56%402x.png)[![23 05 25 2x](https://www.leishi.io/static/65357db7af6576ef34a5aa7378a3c8a0/2bef9/23.05.25%402x.png "23 05 25 2x")](https://www.leishi.io/static/65357db7af6576ef34a5aa7378a3c8a0/eba85/23.05.25%402x.png)

  
Transmission配置

sabnzbd配置

[![21 45 55 2x](https://www.leishi.io/static/7bec999869ededecc1a2c0d386630e27/2bef9/21.45.55%402x.png "21 45 55 2x")](https://www.leishi.io/static/7bec999869ededecc1a2c0d386630e27/764d7/21.45.55%402x.png)

### 回顾

截止到这步，你应该

1.  有至少1个配置好的下载器，无论使用Docker安装的还是使用自带软件中心安装的
2.  你有对应的用户名密码，能够通过web访问它们的控制界面
3.  下载器们能够访问你的下载文件夹，可以正常下载种子文件/nzb文件。

如果你是用docker-compose安装，你的`docker-compose.yml`应该长这样：

    ---
    version: "3"
    services:
      qbittorrent:
        ...
    
      nzbget:
        ...
    
      # transmission:
      #   ...
    
      # sabnzbd:
      #   ...
    
      ...其他下载器

> 你只需要一个BT下载器和一个Usenet下载器即可

Prowlarr/Jackett
----------------

本章节你可以任选其中一个Indexer，无需同时安装两个，但两个Indexer互为补充，如果你有很多PT站点，Jackett可能更适合你，如果你玩Usenet，Prowlarr会比较好。

> 你需要在`docker-compose.yml`文件中继续添加service

### 安装Prowlarr

    ---
    version: "3"
    services:
    
      # ... 此处是其他容器的配置
    
      prowlarr:
        image: ghcr.io/linuxserver/prowlarr:develop
        container_name: prowlarr
        restart: unless-stopped
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
        volumes:
          - ./prowlarr:/config
          - /volume1/Data/Downloads:/downloads   # 替换成你自己的下载文件夹
          - /volume2/Data2/Downloads:/downloads2 # 替换成你自己的第二个分区的下载文件夹
        ports:
          - 9696:9696

配置编写完成后，在命令行运行

    # 移动到你的配置文件夹下
    # cd ~ 
    # 创建Indexer所需要的配置文件夹
    mkdir -p prowlarr
    sudo docker-compose down && sudo docker-compose up -d

### 配置Prowlarr

浏览器打开`NAS_IP:9696`，然后点击侧边栏的`Indexers`，然后点击`Add Indexer`[![23 43 31 2x](https://www.leishi.io/static/8aaf607a0f29aba42e18dac420fc4637/167b5/23.43.31%402x.png "23 43 31 2x")](https://www.leishi.io/static/8aaf607a0f29aba42e18dac420fc4637/167b5/23.43.31%402x.png)

Indexer主要分类3类

*   第一类即BT资源，对应的Privacy为Public，基本可以直接添加[![23 47 32 2x](https://www.leishi.io/static/c8d9c928ef48e522c5947061ee729e8e/2bef9/23.47.32%402x.png "23 47 32 2x")](https://www.leishi.io/static/c8d9c928ef48e522c5947061ee729e8e/f98ee/23.47.32%402x.png)
    
*   第二类是PT资源，对应的Privacy为Private，你需要站点的用户信息才能添加，一般需要站点的Cookies，或者用户名密码，或者Api Key等[![23 48 19 2x](https://www.leishi.io/static/fda7a60dd87e446035d660477617350b/2bef9/23.48.19%402x.png "23 48 19 2x")](https://www.leishi.io/static/fda7a60dd87e446035d660477617350b/21cdd/23.48.19%402x.png)
    
*   第三类是Usenet资源，对应Protocol为nzb的选项，你需要填写你的Usenet Indexer，如果列表中没有对应的，你也可以使用如图的`Generic Newzanb`选项，然后填写`URL`和`Api Key`[![23 51 05 2x](https://www.leishi.io/static/c8aef8dd5145f6f1a185104c62ec563a/2bef9/23.51.05%402x.png "23 51 05 2x")](https://www.leishi.io/static/c8aef8dd5145f6f1a185104c62ec563a/41870/23.51.05%402x.png)
    

### 安装Jackett

    ---
    version: "3"
    services:
    
      # ... 此处是其他容器的配置
    
      jackett:
        image: lscr.io/linuxserver/jackett
        container_name: jackett
        restart: unless-stopped
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
          - AUTO_UPDATE=true
        volumes:
          - ./jackett:/config
          - /volume1/Data/Downloads:/downloads   # 替换成你自己的下载文件夹
          - /volume2/Data2/Downloads:/downloads2 # 替换成你自己的第二个分区的下载文件夹
        ports:
          - 9117:9117

配置编写完成后，在命令行运行

    # 移动到你的配置文件夹下
    # cd ~ 
    # 创建Indexer所需要的配置文件夹
    mkdir -p jackett
    sudo docker-compose down && sudo docker-compose up -d

### 配置Jackett

浏览器打开`NAS_IP:9117`，然后划到最底下，进行一些最基本的配置

1.  首先设置个访问密码，防止其他人直接使用你的Tracker
2.  （选填）其次你可以设置个代理，方便访问OMDB/TMDB等国内访问不畅的网站
3.  （选填）最后可以配置下OMDB，让Jackett自动获取影视的详细信息[![23 57 43 2x](https://www.leishi.io/static/60043cfc67ec7ea606b1d44dcd28ffbc/2bef9/23.57.43%402x.png "23 57 43 2x")](https://www.leishi.io/static/60043cfc67ec7ea606b1d44dcd28ffbc/2a08f/23.57.43%402x.png)

配置完成后，点击页面最上方的`Add Indexer`，开始配置Indexer。

Indexer主要分为2类

*   第一类是BT资源，对应的Type是Public，可以直接添加[![00 02 48 2x](https://www.leishi.io/static/7fdbf4f35b28ee754539446ca7d2153c/2bef9/00.02.48%402x.png "00 02 48 2x")](https://www.leishi.io/static/7fdbf4f35b28ee754539446ca7d2153c/1e1c3/00.02.48%402x.png)
    
*   第二类是PT资源，对应的Type是Private，需要站点相应的信息才能添加，一般是Cookies，或者用户名密码，或者是Api Key等[![00 07 10 2x](https://www.leishi.io/static/c643ae0ac00ec591a01fdff693462e3c/2bef9/00.07.10%402x.png "00 07 10 2x")](https://www.leishi.io/static/c643ae0ac00ec591a01fdff693462e3c/807a0/00.07.10%402x.png)
    

> **注意**：Jackett不支持Usenet的Indexer，需要到Radarr/Sonarr另行配置

### （可选）配置flaresolverr

因为很多PT站点都开了CF盾，这些Indexer可能不会正常工作，所以我们还需要flaresolverr来解决CF5秒盾的问题

> flaresolverr实质上是跑一个浏览器，会对内存有较大的消耗，非必要不需要配置

    ---
    version: "3"
    services:
    
      # ... 此处是其他容器的配置
    
      flaresolverr:
        image: ghcr.io/flaresolverr/flaresolverr:latest
        container_name: flaresolverr
        restart: unless-stopped
        ports:
          - 8191:8191

配置编写完成后，在命令行运行

    # 移动到你的配置文件夹下
    # cd ~ 
    sudo docker-compose down && sudo docker-compose up -d

对于Jackett，flaresolverr的配置很简单，拉到最底下加上`http://flaresolverr:8191/`就行了[![21 18 18 2x](https://www.leishi.io/static/432b72ad4f4eaeddf91aea88e9a1085a/b97f6/21.18.18%402x.png "21 18 18 2x")](https://www.leishi.io/static/432b72ad4f4eaeddf91aea88e9a1085a/b97f6/21.18.18%402x.png)

对于Prowlarr，需要现在Settings（设置）-> Indexers，点击`FlareSolver`[![21 20 57 2x](https://www.leishi.io/static/cedc3a1c70ca5f93b68ada3f328e9d9b/2bef9/21.20.57%402x.png "21 20 57 2x")](https://www.leishi.io/static/cedc3a1c70ca5f93b68ada3f328e9d9b/f4fb1/21.20.57%402x.png)

然后取个名字和Tag，填上`http://flaresolverr:8191/`，保存好[![21 20 34 2x](https://www.leishi.io/static/e67222287ac716935a790696978747c9/2bef9/21.20.34%402x.png "21 20 34 2x")](https://www.leishi.io/static/e67222287ac716935a790696978747c9/f3abf/21.20.34%402x.png)

然后在你需要flaresolver的Indexer配置里，加上对应的Tag就行[![21 24 03 2x](https://www.leishi.io/static/c3bde6534979b43eb2fb65aaf13a63d8/2bef9/21.24.03%402x.png "21 24 03 2x")](https://www.leishi.io/static/c3bde6534979b43eb2fb65aaf13a63d8/f3a19/21.24.03%402x.png)

### 回顾

本章结束后，你应该

1.  安装并配置好至少一个Indexer
2.  在Indexer里添加你的资源来源（BT/PT/Usenet）

现在你的`docker-compose.yml`应该长这样：

    ---
    version: "3"
    services:
      
      ...下载器容器
    
      prowlarr:
        ...
    
      jackett:
        ...

Prowlarr/Jackett也应该至少有一个资源来源[![22 56 33 2x](https://www.leishi.io/static/57f33354935cd492feb3a99987ec571e/2bef9/22.56.33%402x.png "22 56 33 2x")](https://www.leishi.io/static/57f33354935cd492feb3a99987ec571e/8e6e2/22.56.33%402x.png)[![22 57 25 2x](https://www.leishi.io/static/6144f5fccc3229a1fd76ba766be66f28/2bef9/22.57.25%402x.png "22 57 25 2x")](https://www.leishi.io/static/6144f5fccc3229a1fd76ba766be66f28/cea38/22.57.25%402x.png)

Radarr/Sonarr
-------------

Radarr/Sonarr是本文的重点，在配置上最为繁琐，如果你把Radarr/Sonarr配置完成并弄明白，那么整套系统就可以全自动地工作起来了。

### 安装Radarr/Sonarr

我们继续编辑你的`docker-compose.yml`文件：

    ---
    version: "3"
    services:
      
      ...下载器容器
    
      ...Indexer容器
    
      radarr:
        image: lscr.io/linuxserver/radarr
        container_name: radarr
        restart: unless-stopped
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
        volumes:
          - ./radarr:/config
          - /volume1:/volume1 # 注意这里不再是下载文件夹，需要是下载/电影/剧集文件夹的母文件夹
          - /volume2:/volume2 # 注意这里不再是下载文件夹，需要是下载/电影/剧集文件夹的母文件夹
        ports:
          - 7878:7878
      sonarr:
        image: lscr.io/linuxserver/sonarr
        container_name: sonarr
        restart: unless-stopped
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
        volumes:
          - ./sonarr:/config
          - /volume1:/volume1 # 注意这里不再是下载文件夹，需要是下载/电影/剧集文件夹的母文件夹
          - /volume2:/volume2 # 注意这里不再是下载文件夹，需要是下载/电影/剧集文件夹的母文件夹
        ports:
          - 8989:8989

配置编写完成后，在命令行运行

    # 移动到你的配置文件夹下
    # cd ~ 
    # 创建Radarr/Sonarr所需要的配置文件夹
    mkdir -p sonarr radarr
    sudo docker-compose down && sudo docker-compose up -d

然后浏览器分别打开`NAS_IP:7878`和`NAS_IP:8989`就可以访问Radarr/Sonarr了。

### 配置Radarr/Sonarr

两款软件系出同门，UI也大同小异，我会把文字描述放在Radarr上，然后Sonarr的配置用截图示意。

#### 配置刮削

首先我们设置电影信息的语言，Settings（设置）-> UI -> Language（语言）[![11 13 51 2x](https://www.leishi.io/static/4a00bbd6da765887072a02ad5fc8bb2e/2bef9/11.13.51%402x.png "11 13 51 2x")](https://www.leishi.io/static/4a00bbd6da765887072a02ad5fc8bb2e/00e09/11.13.51%402x.png)

然后我们设置刮削，Settings（设置）-> Metadata（元文件） -> 根据你的影视库选择Emby或其他[![11 17 38 2x](https://www.leishi.io/static/a21f7c548a2cf703b477b0b7cd42f7d6/2bef9/11.17.38%402x.png "11 17 38 2x")](https://www.leishi.io/static/a21f7c548a2cf703b477b0b7cd42f7d6/e1250/11.17.38%402x.png)

在弹出框，我们选择启用，然后选择你需要的刮削语言，点击保存就好[![11 20 40 2x](https://www.leishi.io/static/520a4916fa2e2a9887cdea24cf19f28b/2bef9/11.20.40%402x.png "11 20 40 2x")](https://www.leishi.io/static/520a4916fa2e2a9887cdea24cf19f28b/a83dd/11.20.40%402x.png)

#### 配置下载器

进入WebUI，我们点击Settings（设置）->Download Clients（下载器），然后点击那个加号，余下的就根据你自己的下载器自行添加[![22 25 08 2x](https://www.leishi.io/static/ae9b287d4a443e7ea2f552b068361956/2bef9/22.25.08%402x.png "22 25 08 2x")](https://www.leishi.io/static/ae9b287d4a443e7ea2f552b068361956/5ab15/22.25.08%402x.png)

配置完成后你的下载器配置应该差不多长这样：[![22 32 32 2x](https://www.leishi.io/static/1d30744ed9406b5b920a791faafee1a5/2bef9/22.32.32%402x.png "22 32 32 2x")](https://www.leishi.io/static/1d30744ed9406b5b920a791faafee1a5/09262/22.32.32%402x.png)

> BT（PT）和Usenet各添加一个下载器即可，图示各使用了2个下载器是**错误**做法！

##### 配置远程文件映射

如果你的下载器是NAS宿主机安装的，那么可以略过这步，如果是Docker安装的，或者Radarr/Sonarr和下载器不在一台机器上，我们需要配置`Remote Path Mappings`。

为什么需要映射（mapping）呢？如果你还记得的话，我们在用Docker安装下载器的时候，把宿主机的目录挂载进了容器的目录里（你可以温习下[Docker挂载目录](https://www.leishi.io/blog/posts/2021-12/home-nas-media-center/#docker%E7%AE%80%E4%BB%8B)），两个目录并不是同样的名称：

宿主机目录

Docker容器内的目录

/volume1/Data/Downloads

/downloads

/volume2/Data2/Downloads

/downloads2

而在Docker容器内，它只认识`/downloads`，它完全意识不到这其实是NAS上的`/volume1/Data/Downloads`。而对于Radarr/Sonarr，我们直接把`/volume1`映射到了容器里的`/volume1`，所以Radarr/Sonarr容器内的`/volume1/Data/Downloads`，仍然是宿主系统上的`/volume1/Data/Downloads`。知道了这个原理，我们的映射就好配置了：[![22 43 56 2x](https://www.leishi.io/static/384015d1f257732945edb28f066c07d6/2bef9/22.43.56%402x.png "22 43 56 2x")](https://www.leishi.io/static/384015d1f257732945edb28f066c07d6/3126c/22.43.56%402x.png)

现在Radarr/Sonarr就知道了，下载器内的`/downloads`和`/downloads2`其实就是自己容器内的`/volume1/Data/Downloads`和`/volume2/Data2/Downloads`

> 对于Radarr/Sonarr和下载器在不同机器的，原理也大致相同，核心就是将Radarr/Sonarr容器内能访问到的下载目录和下载器的下载目录对应起来即可

对于Sonarr，同样也配置好[![22 50 47 2x](https://www.leishi.io/static/87ab4c845c73ba121ef57ad1a630b9d3/2bef9/22.50.47%402x.png "22 50 47 2x")](https://www.leishi.io/static/87ab4c845c73ba121ef57ad1a630b9d3/74d4a/22.50.47%402x.png)

#### 配置Indexer

配置完了下载器，Radarr/Sonarr就认识了下载器，也能正确分配下载任务给下载器了。那么任务从哪来了，这就要配置Indexer了

##### Prowlarr配置

两个Indexer的配置略有不同，对于Prowlarr来说，如果你还记得的话，Prowlarr能够自动同步Indexer配置到Radarr/Sonarr，所以我们先需要拿到Radarr/Sonarr的API Key：[![23 00 31 2x](https://www.leishi.io/static/aa42f105bdb612597a210856145f0f82/2bef9/23.00.31%402x.png "23 00 31 2x")](https://www.leishi.io/static/aa42f105bdb612597a210856145f0f82/a9577/23.00.31%402x.png)

> Sonarr与Radarr的API Key在同样的位置，此处不再截图

然后我们回到Prowlarr，网页打开`NAS_IP:9696`[![23 10 16 2x](https://www.leishi.io/static/7a916394bc462ffe49bef666fd8689dd/2bef9/23.10.16%402x.png "23 10 16 2x")](https://www.leishi.io/static/7a916394bc462ffe49bef666fd8689dd/49b61/23.10.16%402x.png)

在弹出框里，点击Radarr和Sonarr，我们两个都需要各自配置一遍：[![23 13 17 2x](https://www.leishi.io/static/cf3376b8df92906ad33322066f5677ab/2bef9/23.13.17%402x.png "23 13 17 2x")](https://www.leishi.io/static/cf3376b8df92906ad33322066f5677ab/a2792/23.13.17%402x.png)

> 你肯定注意到了Prowlarr和Radarr的地址很奇怪，你也可以选择填写你的NAS IP，或者如果他们在一个docker-compose里，那么service的名称（这里容器们的名称恰好就是prowlarr，radarr，sonarr）可以用来当作地址，这跟`localhost`有点类似

当你添加完成后，点击一下`Sync App Indexers`，然后回到Radarr/Sonarr的Indexer页面，你会看到Prowlarr的配置已经同步了过来[![23 19 39 2x](https://www.leishi.io/static/620afa659bfa5446097e45a409168308/2bef9/23.19.39%402x.png "23 19 39 2x")](https://www.leishi.io/static/620afa659bfa5446097e45a409168308/dcb79/23.19.39%402x.png)[![23 23 53 2x](https://www.leishi.io/static/bb0695ef02865f3d5424e2ab9f8195a4/2bef9/23.23.53%402x.png "23 23 53 2x")](https://www.leishi.io/static/bb0695ef02865f3d5424e2ab9f8195a4/b88bf/23.23.53%402x.png)

##### Jacker配置

Jackett的配置就更直接但是重复一些，我们首先到Jackett的页面，点击你需要使用的Indexer，点击`Copy Torznab Feed`（或者`Copy Potato Feed`也可）[![23 34 09 2x](https://www.leishi.io/static/48b2791df3afa7248c86b1f01d434923/2bef9/23.34.09%402x.png "23 34 09 2x")](https://www.leishi.io/static/48b2791df3afa7248c86b1f01d434923/bf286/23.34.09%402x.png)

然后来到Radarr，点击Settings（设置）->Indexers->点击加号[![23 29 30 2x](https://www.leishi.io/static/db6b0a9c099f8016570a059537895d7b/2bef9/23.29.30%402x.png "23 29 30 2x")](https://www.leishi.io/static/db6b0a9c099f8016570a059537895d7b/09ede/23.29.30%402x.png)

选择`Torznab`（或者`TorrentPotato`），然后在页面里填上刚才复制的链接，和Jackett的API Key（就在Jackett页面右上角）[![23 31 34 2x](https://www.leishi.io/static/6d7a7fdc7c9d0d20aea35c055b66d933/2bef9/23.31.34%402x.png "23 31 34 2x")](https://www.leishi.io/static/6d7a7fdc7c9d0d20aea35c055b66d933/6f464/23.31.34%402x.png)

然后就是重复工作了，对于每个你想要使用的资源来源，都需要在Radarr/Sonarr这样配置：）如果你的资源来源比较多的，Good Luck！

#### 配置媒体目录

解决了资源和下载问题，我们再来设置我们的媒体库位置，进入到Radarr，点击设置，媒体管理，然后点击添加根目录[![23 39 45 2x](https://www.leishi.io/static/4b7618935647106671656fcde8791b9e/2bef9/23.39.45%402x.png "23 39 45 2x")](https://www.leishi.io/static/4b7618935647106671656fcde8791b9e/d7ceb/23.39.45%402x.png)

因为是Radarr，所以我们把我们之前创建（或者已有）的电影文件夹设为根目录[![23 43 18 2x](https://www.leishi.io/static/40f1d0efdbb1a14b712cc8db9e5712b8/2bef9/23.43.18%402x.png "23 43 18 2x")](https://www.leishi.io/static/40f1d0efdbb1a14b712cc8db9e5712b8/56e36/23.43.18%402x.png)

Sonarr同理，但是要换成剧集文件夹[![23 45 35 2x](https://www.leishi.io/static/4a595b6e2236695a1a781b56252b4539/2bef9/23.45.35%402x.png "23 45 35 2x")](https://www.leishi.io/static/4a595b6e2236695a1a781b56252b4539/56e36/23.45.35%402x.png)

#### 恭喜配置完成

至此，自动化的搜索下载整理工作已经完全可以运行起来了，我们来添加一部电影试试看！![](https://www.leishi.io/public/4364278a4ce501843a1e9f512dda7e88/radarr.gif)

配置Overseerr/Ombi
----------------

配置好了Radarr/Sonarr，我们就可以为他们配上一个好看的前端了。这里我们选择Overseerr（适合Plex）或者Ombi（适合Emby/Plex），你可以根据自己的影视库软件来选择

### 安装Overseerr

> Overseerr依赖TMDB，你可能需要代理才能完整使用Overseerr

同前面一样，我们继续修改`docker-compose.yml`

    ---
    version: "3"
    services:
      # ... 下载器容器
    
      # ... Indexer容器
    
      radarr:
        ...
    
      sonarr:
        ...
    
      overseerr:
        image: lscr.io/linuxserver/overseerr
        container_name: overseerr
        restart: unless-stopped
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
        ports:
          - 5055:5055
        volumes:
          - ./overseerr:/app/config

修改完成后，还是几条老命令，创建配置文件夹，重启所有容器：

    # 移动到你的配置文件夹下
    # cd ~ 
    # 创建需要的配置文件夹
    mkdir -p overseerr
    sudo docker-compose down && sudo docker-compose up -d

完成后浏览器打开，到`NAS_IP:5055`就可以访问Overseerr的页面了

### 配置Overseerr

Overseerr首先需要连接你的Plex，请根据提示连接，在`Plex Libraries`这一步，注意把`Movies`和`TV Shows`选上，然后点击`Start Scan`[![13 30 25](https://www.leishi.io/static/0ce6dbfffd5b03b487f5e1bcec65bc4e/76cea/13.30.25.png "13 30 25")](https://www.leishi.io/static/0ce6dbfffd5b03b487f5e1bcec65bc4e/76cea/13.30.25.png)

下一步，分别添加Radarr/Sonarr服务器[![13 39 28](https://www.leishi.io/static/7059ed35669dbbf6854bfe0d158aed63/898f6/13.39.28.png "13 39 28")](https://www.leishi.io/static/7059ed35669dbbf6854bfe0d158aed63/898f6/13.39.28.png)

> 你可以到Settings（设置）-> General（通用）来找你的API Key

[![13 34 32](https://www.leishi.io/static/1646984cc489e845b54dd5de9e338e0f/01267/13.34.32.png "13 34 32")](https://www.leishi.io/static/1646984cc489e845b54dd5de9e338e0f/01267/13.34.32.png)

全部配置完成后，点击`Finish Setup`即可[![13 43 26](https://www.leishi.io/static/06a32c383c84b5b0003bccab000029f4/73fd0/13.43.26.png "13 43 26")](https://www.leishi.io/static/06a32c383c84b5b0003bccab000029f4/73fd0/13.43.26.png)

### 安装Ombi

我们继续修改`docker-compose.yml`

    ---
    version: "3"
    services:
      # ... 下载器容器
    
      # ... Indexer容器
    
      radarr:
        ...
    
      sonarr:
        ...
    
      ombi:
        image: lscr.io/linuxserver/ombi
        container_name: ombi
        restart: unless-stopped
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
        volumes:
          - ./ombi:/config
        ports:
          - 3579:3579

修改完成后，还是几条老命令，创建配置文件夹，重启所有容器：

    # 移动到你的配置文件夹下
    # cd ~ 
    # 创建需要的配置文件夹
    mkdir -p ombi
    sudo docker-compose down && sudo docker-compose up -d

完成后浏览器打开，到`NAS_IP:3579`就可以访问Ombi的页面了

### 配置Ombi

打开浏览器，访问Ombi的页面，我们可以看到Ombi支持Emby/Plex/Jellyfin，这里我们以Emby为例[![13 53 28](https://www.leishi.io/static/fbce41afb6ec93f886255e3739729207/1ac29/13.53.28.png "13 53 28")](https://www.leishi.io/static/fbce41afb6ec93f886255e3739729207/1ac29/13.53.28.png)

然后再设置个用户名密码，一路下一步就可以了。再次登陆Ombi后，我们首先设置Radarr/Sonarr[![13 56 06](https://www.leishi.io/static/d5bef4f1cb325f57b04d5151ddc58e85/339e3/13.56.06.png "13 56 06")](https://www.leishi.io/static/d5bef4f1cb325f57b04d5151ddc58e85/339e3/13.56.06.png)

### 使用Overseerr/Ombi

Overseerr和Ombi的使用大同小异，都是在搜索框搜索你想看的电影/剧集，点击Request，然后在弹出框选择你要的清晰度，和电影/剧集文件夹即可[![14 04 00](https://www.leishi.io/static/a0cc5aae47ef433a6b0b361fd7dc7bdf/2bef9/14.04.00.png "14 04 00")](https://www.leishi.io/static/a0cc5aae47ef433a6b0b361fd7dc7bdf/76a99/14.04.00.png)[![14 02 17](https://www.leishi.io/static/9638898f13dc781b5fb8b7e7eef4afaa/2bef9/14.02.17.png "14 02 17")](https://www.leishi.io/static/9638898f13dc781b5fb8b7e7eef4afaa/84ee5/14.02.17.png)

安装字幕下载器
-------

至此，资源的自动入库，自动化下载改名刮削已经完全可以运作了，就只差最后一块拼图 -- 中文字幕的获取了。自动字幕下载器我们选择Chinesesubfinder和Bazarr

### 安装Chinesesubfinder

继续编辑`docker-compose.yml`文件

    ---
    version: "3"
    services:
      # ...下载器容器
      # ...Indexer容器
      # ...Radarr
      # ...Sonarr
      # ...Overseerr/Ombi
    
      chinesesubfinder:
        image: allanpk716/chinesesubfinder:latest
        container_name: chinesesubfinder
        restart: unless-stopped
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
        volumes:
          - ./chinesesubfinder:/config
          - /volume1/Data/Movies:/volume1/Data/Movies
          - /volume2/Data2/Series:/volume2/Data/Series

编辑完成后，**重启并马上关闭容器**

    # 移动到你的配置文件夹下
    # cd ~ 
    # 创建需要的配置文件夹
    mkdir -p chinesesubfinder
    sudo docker-compose down && sudo docker-compose up chinesesubfinder

当命令行开始报`ERROR ... MovieFolder not found`的时候，按`Ctrl+C`退出。

### 配置Chinesesubfinder

现在你的chinesesubfinder文件夹下应该会多`config.yml`和`config.yaml.sample`这两个文件[![14 21 12](https://www.leishi.io/static/2879ce816e42adab94f94b29508d5494/fcb94/14.21.12.png "14 21 12")](https://www.leishi.io/static/2879ce816e42adab94f94b29508d5494/fcb94/14.21.12.png)

编辑`config.yaml`，按需添加代理，设置电影和剧集文件即可[![14 22 08](https://www.leishi.io/static/d3703be7d96eb421acbcb9dc448e9dc1/b6a9b/14.22.08.png "14 22 08")](https://www.leishi.io/static/d3703be7d96eb421acbcb9dc448e9dc1/b6a9b/14.22.08.png)

> chinesesubfinder暂时不支持多个电影/剧集文件夹，你需要添加多个容器来支持多个文件夹

编辑完成后，重启所有容器

    # 移动到你的配置文件夹下
    # cd ~ 
    sudo docker-compose up -d

### 安装Bazarr

继续编辑`docker-compose.yml`

    services:
      # ...下载器容器
      # ...Indexer容器
      # ...Radarr
      # ...Sonarr
      # ...Overseerr/Ombi
    
      bazarr:
        image: lscr.io/linuxserver/bazarr
        container_name: bazarr
        restart: unless-stopped
        environment:
          - PUID=1026 # 注意替换
          - PGID=100  # 注意替换
          - TZ=Asia/Shanghai
        volumes:
          - ./bazarr:/config
          - /volume1/Data/Movies:/volume1/Data/Movies     # 替换成你自己的电影文件夹
          - /volume1/Data/Series:/volume1/Data/Series     # 替换成你自己的剧集文件夹
          - /volume2/Data2/Movies:/volume2/Data2/Movies   # 替换成你自己的电影文件夹
          - /volume2/Data2/Series:/volume2/Data2/Series   # 替换成你自己的剧集文件夹
          # - /volume1:/volume1 你也可以选择这种整个映射的形式
          # - /volume2:/volume2
        ports:
          - 6767:6767

编辑完成后，重启所有容器

    # 移动到你的配置文件夹下
    # cd ~ 
    # 创建需要的配置文件夹
    mkdir -p bazarr
    sudo docker-compose down && sudo docker-compose up -d

重启完成后，你可以到`NAS_IP:6767`访问Bazarr

### 配置Bazarr

首先你可以Bazarr的初始页面配置下用户验证和代理，然后点击Settings（设置）-> Providers，添加我们的字幕来源[![14 41 01](https://www.leishi.io/static/c195946d6b0ef4e9de5783abd641f29c/8b69f/14.41.01.png "14 41 01")](https://www.leishi.io/static/c195946d6b0ef4e9de5783abd641f29c/8b69f/14.41.01.png)

然后在Settings（设置）-> Radarr/Sonarr，添加Radarr/Sonarr[![14 43 12](https://www.leishi.io/static/09883441a15f8246552f4172c418ae1d/1ddef/14.43.12.png "14 43 12")](https://www.leishi.io/static/09883441a15f8246552f4172c418ae1d/1ddef/14.43.12.png)

然后我们把中文加上作为我们的字幕语言[![14 47 17](https://www.leishi.io/static/3eb6aab25e3cf4c9e43e378f2e6b20ee/3cd52/14.47.17.png "14 47 17")](https://www.leishi.io/static/3eb6aab25e3cf4c9e43e378f2e6b20ee/3cd52/14.47.17.png)

至此，Bazarr应该就可以自动连接Radarr/Sonarr，并为其中的资源自动匹配字幕了！

*   [Github](https://github.com/leishi1313/)
*   [Linkedin](https://linkedin.com/in/lei-shi-swe/)
*   [Telegram](https://t.me/caomeixigua/)
*   [Email](mailto:me@leishi.io)

© 2023 Lei Shi. All rights reserved.
