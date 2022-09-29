## Tair Tutorial

快速入门概览
![](https://raw.githubusercontent.com/yangbodong22011/tair-tutorial/main/pictures/connect-tair.png)

1，准备 Tair 实例，使用训练营提供的实例或者自行[创建实例](https://help.aliyun.com/document_detail/443863.html), 存储介质选择**内存型**可支持所有 Tair 高级数据结构。

![](https://raw.githubusercontent.com/yangbodong22011/tair-tutorial/main/pictures/common-buy.jpg)

2，设置实例[白名单](https://help.aliyun.com/document_detail/443864.html)。
![](https://raw.githubusercontent.com/yangbodong22011/tair-tutorial/main/pictures/white-list.jpg)

3，申请公网访问地址（后续的教程在 aliyun shell 完成，因此需要公网访问 Tair 实例）。
![](https://raw.githubusercontent.com/yangbodong22011/tair-tutorial/main/pictures/direct-link.jpg)

## 连接实例

### 1，通过`redis-cli`连接 Tair 实例

```bash
./redis-cli -h xxx -p 6379 -a xxx --raw
```

### 2，通过`DMS`连接 Tair 实例

在Redis实例管理页面点击`登录数据库`，即可自动跳转到DMS平台。

![](https://raw.githubusercontent.com/yangbodong22011/tair-tutorial/main/pictures/dms-tair.jpg)


## 游戏好友信息关系模型

游戏中，用户信息与好友列表维护是很常见的操作。

![](https://raw.githubusercontent.com/yangbodong22011/tair-tutorial/main/pictures/game-friend.png)
**注：图片仅做示意使用**

### 1，存储用户`tom`和`denny`信息

```bash
JSON.SET user:tom . '{"name":"tom","age":22,"description":"A man with a blue lightsaber","friends":[]}'
```
```bash
JSON.SET user:denny . '{"name":"denny","age":23,"description":"Driver with big truck from Siberia","friends":[]}'
```

### 2，双方互相添加好友

```bash
JSON.ARRAPPEND user:denny .friends '"user:tom"'
```
```bash
JSON.ARRAPPEND user:tom .friends '"user:denny"'
```

### 3，再次查看`tom`和`denny`信息
```bash
JSON.GET user:tom
```
```bash
JSON.GET user:denny
```

## 游戏论坛的搜索

![](https://raw.githubusercontent.com/yangbodong22011/tair-tutorial/main/pictures/game-forum.png)
**注：图片仅做示意使用**

### 1, 创建索引
一个论坛的帖子包括以下几个部分：title，content，time，author，heat。按照相应的类型可以为不同的字段创建不同的索引，Tair 支持多种分词器。

```bash
TFT.CREATEINDEX post_index '{"mappings":{"properties":{"title":{"type":"keyword"},"content":{"type":"text","analyzer":"jieba"},"time":{"type":"long"},"author":{"type":"keyword"},"heat":{"type":"integer"}}}}'
```

### 2，发帖
```bash
TFT.ADDDOC post_index '{"title":"Does not work","content":"It was removed from the beta a while ago. You should have expected it was going to be removed from the stable client as well at some point.","time":1541713787,"author":"cSg|mc","heat":10}'
```
```bash
TFT.ADDDOC post_index '{"title":"paypal no longer launches to purchase","content":"Since the last update, I cannot purchase anything via the app. I just keep getting a screen that says","time":1551476987,"author":"disasterpeac","heat":2}'
```
```bash
TFT.ADDDOC post_index '{"title":"cat not login","content":"Hey! I am trying to login to steam beta client via qr code / steam guard code but both methods does not work for me","time":1664488187,"author":"7xx","heat":100}'
```

### 3，搜索帖子

搜索结果按照`order`降序，并且搜索的关键词是`paypal work code`。
```bash
TFT.SEARCH post_index '{"sort":[{"heat":{"order":"desc"}}],"query":{"match":{"content":"paypal work code"}}}'
```

## 游戏多维排行榜

![](https://raw.githubusercontent.com/yangbodong22011/tair-tutorial/main/pictures/game-leaderboard.jpg)
**注：图片仅做示意使用**

在游戏中，当`等级`相同时候，按照`胜率`排序，否则再按照`积分`排序。

### 1，添加数据
各维度的比分需要用`#`隔开，添加六个 user，各自的`等级`、`胜率`、`积分`如下所示。
```bash
EXZADD leaderboard 60#30#20 user1 60#20#15 user2 80#30#20 user3 100#25#15 user4 90#80#10 user5 30#30#30 user6
```

### 2，获取某个用户排名
注意到，user1 和 user2 的`等级`是相同的，因此需要按照`胜率`对它们再次排序，user2 的胜率小，因此它被排在后面，如果想按照胜率从低到高排序，使用 EXZRANK 即可。
```bash
EXZREVRANK leaderboard user2
```
```bash
EXZREVRANK leaderboard user1
```

### 3，获取 top3 用户
EXZREVRANGE 可以按照从高往低的顺序遍历用户，指定起始点位为 0 和 2 可以获取 top3 的用户。
```bash
EXZREVRANGE leaderboard 0 2
```

### 4，获取所有用户的排名
指定起始点位为 0 和 -1 则可以获取全部用户。
```bash
EXZREVRANGE leaderboard 0 -1
```