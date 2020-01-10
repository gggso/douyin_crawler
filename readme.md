## 抖音分布式爬虫使用文档

#### 2019.12.20

#### 核心功能和优势

1. 现在抖音升级特别频繁，老版本的限制越来越多，以前的签名算法，要么已经过期，要么就不返回数据，要么就是经常封 IP，造成接口和代码要经常改动，维护代理 IP 的成本也高
2. 现在的方案是通过最新的分布式采集系统实现数据抓取，一个任务同时在后台，使用多个 IP 不同的协议和签名实现，保证返回数据，减少单点系统的瓶颈和成本
3. 目前可以实现每天上千万次抓取，接口简单，效率极高
4. 采用云方案，不需要部署采集服务器，不需要购买代理 IP，不需要处理升级

#### 在线演示地址

> https://yundou.me/

#### 采集任务投放地址

> https://service.yundou.me/

#### 只接受 JSON 消息格式

```shell
curl -s -m 5 -d '{}'
```

#### 采集成功后，会回调到客户的 http 接口

> 回调接口处理成功，必须返回 `{"code":200}` 的字符串
> 回调接口请求超时时间为 10 秒，响应超时视为失败
> 回调失败最多只重试一次
> 任务超过5分钟视为失败，可重新发起任务


#### 支持的接口列表

- crawler_search_user: 根据抖音号搜索 UID
- crawler_search_video: 搜索视频
- crawler_user_info: 根据 UID 返回用户详情
- crawler_user_following: 根据 UID 返回用户关注列表
- crawler_user_follower: 根据 UID 返回用户粉丝列表
- crawler_user_post: 根据 UID 返回用户作品列表
- crawler_nearby_feed: 根据城市返回用户作品列表
- crawler_comment_list: 根据视频 ID 返回评论列表

#### 接受的参数列表

```json
{
    "token": "",
    "num": 20,
    "type": "crawler_search_user",
    "uid": "85635793",
    "vid": "6763872129701124",
    "sec_uid": "MS4wLjABAAAA6FJbgV0BY17eGBY",
    "city_id": "510100",
    "search": "abcdefg",
    "task":"your_uniq_id",
    "result": []
}

> 不同的任务，需要传递的参数不一样，详细参数见下表
> `token` 为购买时分配的密钥，请注意续费
> `task` 是预留的字段，只支持16位以内字符串，可以用于标记客户自己的唯一任务
> `result` 为采集的结果
> 该请求参数，也是回调内容

```

#### 响应内容

>
```json
{
    "code": 200
}
```

```json
{
    "code": 500,
    "msg": "异常内容"
}
```

#### 详细接口

```json
{
    "num": 20,
    "type": "crawler_search_user",
    "search": "abc12345678"
}
```

```json
{
    "num": 20,
    "type": "crawler_search_video",
    "search": "abc12345678"
}
```

> 必须包含 uid 和 sec_uid

```json
{
    "id": 0,
    "type": "crawler_user_info",
    "uid": "9338953804",
    "sec_uid": "MS4wLjABAAAAQ4xCNiRbRwIg"
}
```

> 必须包含 uid 和 sec_uid

```json
{
    "id": 0,
    "num": 20,
    "type": "crawler_user_favorite",
    "uid": "632494600",
    "sec_uid": "MS4wLjABAAAAQ4xCNiRbRwIg"
}
```

> 必须包含 uid 和 sec_uid

```json
{
    "id": 0,
    "num": 20,
    "type": "crawler_user_post",
    "uid": "632494600",
    "sec_uid": "MS4wLjABAAAAQ4xCNiRbRwIg"
}
```

> 必须包含 vid

```json
{
    "id": 0,
    "num": 20,
    "type": "crawler_comment_list",
    "vid": "66082937525764932"
}
```

> 必须包含 uid 和 sec_uid

```json
{
    "id": 0,
    "num": 20,
    "type": "crawler_user_follower",
    "uid": "16361944337",
    "sec_uid": "MS4wLjABAAAAQ4xCNiRbRwIg"
}
```

> 必须包含 uid 和 sec_uid

```json
{
    "id": 0,
    "num": 20,
    "type": "crawler_user_following",
    "uid": "163619337",
    "sec_uid": "tWyVTUdvPOg90efQ7E"
}
```

> city_id 和 坐标必须包含其中一项

```json
{
    "id": 0,
    "num": 20,
    "type": "crawler_nearby_feed",
    "city_id": "510900",
    "longitude": "105.389997",
    "latitude": "30.87346",
}

```

#### 限额

1. 1000元/百万次/月
2. 每秒并发不超过 100

#### 联系方式

1. wec.cloud@gmail.com
2. tg: +639288446666
3. QQ: 2390501091 2878261643

#### 注意事项

1. 回调失败会扣除点数，回调只重试一次，超时时间为 10 秒，尽量将业务逻辑放到异步处理
2. 有需要分页的请求默认是20页每页，可以传20的整总数，最多可以传5000条，按20条/页扣除点数
3. 如果分页数据不足按20条/页扣除点数，比如100条抓取到第2页没数据了，也会扣除5个点数
3. 任务不除重，相同的任务会扣除点数，每次扣除1个点数
4. 有些账号设置了粉丝隐私，无法查看粉丝列表，拿不到数据的也会扣除点数，提前通过用户信息页获取设置信息
5. 每个任务会通过不同的IP、不同版本的协议去抓取，最多尝试9次，一共扣除1个点数
6. 任务正常完成的时间在3-4秒，第一次失败的任务会在 2 分钟后开始重试，5分钟未回调成功的视为失败
