# live 模块

`from bilibili_api import live`

直播弹幕获取，直播间操作，发送弹幕等。

## 通用名词解释

房间id分为两种。

room_display_id：房间显示ID，一般很短，签约主播才会用。就是live.bilibili.com/这里的数字。

room_real_id：真正的房间号。使用 [get_room_play_info](#get_room_play_info) 获取

**如无特殊说明，下列所有方法均需要传入 room_real_id，不再赘述**

## 方法

### get_room_play_info

获取房间信息（真实房间号，封禁情况等）

| 参数名          | 类型 | 必须提供 | 默认 | 释义       |
| --------------- | ---- | -------- | ---- | ---------- |
| room_display_id | int  | True     | -    | 房间显示ID |

### get_chat_conf

获取聊天服务器信息，用于连接弹幕服务器用。下面有 [LiveDanmaku](#LiveDanmaku) 类可以方便连接，不需要用到这个方法。

### get_room_info

获取直播间信息（标题，简介等）

### get_user_info_in_room

获取自己在直播间的信息（粉丝勋章等级，直播用户等级等）

需要登录。

### get_self_info

获取自己直播用户等级信息（相对上一个来说是通用的）

### get_black_list

获取房间封禁列表，需要登录并且是房管或者主播。

### get_dahanhai

获取大航海列表

参照：[循环获取数据参数说明][循环获取数据参数说明]

### get_dahanhai_raw

低层级API，获取大航海列表

| 参数名       | 类型 | 必须提供 | 默认 | 释义                                                         |
| ------------ | ---- | -------- | ---- | ------------------------------------------------------------ |
| room_real_id | int  | True     | -    | 房间真实ID                                                   |
| ruid         | int  | True     | -    | room_uid，从 [get_room_play_info](#get_room_play_info) 可获取 |
| page         | int  | False    | 1    | 页码                                                         |
| page_size    | int  | False    | 29   | 每页数量                                                     |

### get_seven_rank

获取七日榜

### get_fans_medal_rank

获取粉丝勋章排行。

### send_danmaku

发送弹幕

| 参数名  | 类型               | 必须提供 | 默认 | 释义     |
| ------- | ------------------ | -------- | ---- | -------- |
| danmaku | [Danmaku][Danmaku] | True     | -    | 发送弹幕 |

### ban_user

封禁用户，需要是房管或者主播。

| 参数名 | 类型 | 必须提供 | 默认 | 释义          |
| ------ | ---- | -------- | ---- | ------------- |
| uid    | int  | True     | -    | 用户UID       |
| hour   | int  | False    | 1    | 小时数，1~720 |

### unban_user

解封用户

| 参数名   | 类型 | 必须提供 | 默认 | 释义       |
| -------- | ---- | -------- | ---- | ---------- |
| block_id | int  | True     | -    | 封禁事件ID |

注意，block_id可用 [ban_user](#ban_user) 的返回值获取，或者 [get_black_list](#get_black_list) 中获取。


### connect_all_livedanmaku

自动连接多个直播间弹幕

| 参数名   | 类型 | 必须提供 | 默认 | 释义       |
| -------- | ---- | -------- | ---- | ---------- |
| *livedanmaku_classes | [LiveDanmaku](#LiveDanmaku)  | True     | -    | LiveDanmaku类动态参数 |

具体用法参加：[连接多个直播间](#连接多个直播间)


## 类

### LiveDanmaku

连接直播间弹幕服务器。

#### 初始化参数

| 参数名          | 类型 | 必须提供 | 默认  | 释义                          |
| --------------- | ---- | -------- | ----- | ----------------------------- |
| room_display_id | int  | True     | -     | 房间显示ID                    |
| debug           | bool | False    | False | 调试模式，将输出详细的信息    |
| use_wss         | bool | False    | True  | 使用WSS（Websocket over SSL） |
| verify         | bool | False    | None  | verify类，事实上不用提供也行 |

#### 属性

| 属性名 | 类型           | 释义                           |
| ------ | -------------- | ------------------------------ |
| logger | logging.Logger | 日志记录，可以自行设置一些输出 |

#### 方法

##### connect

连接弹幕服务器。

| 参数名          | 类型 | 必须提供 | 默认  | 释义                          |
| --------------- | ---- | -------- | ----- | ----------------------------- |
| return_task | bool  | False     | False     | 返回Task类，供用户自行调用协程 |


##### disconnect

断开弹幕服务器连接。


##### add_event_handler

添加事件处理器

| 参数名          | 类型 | 必须提供 | 默认  | 释义                          |
| --------------- | ---- | -------- | ----- | ----------------------------- |
| event_name | str  | True     | -     | 事件名，参照下面的事件名列表 |
| func | Function  | True     | -     | 事件处理器方法 |


##### has_connected

是否已连接直播间

#### 事件

收到事件时调用用户指定方法，完整例子：

```python
from bilibili_api.live import LiveDanmaku

room = LiveDanmaku(room_display_id=114514)

@room.on("DANMU_MSG")  # 指定事件名
def on_danmu(msg):
    print(msg)
    
if __name__ == "__main__":
    room.connect()
```

请按照以上格式写。

另外还有一种异步写法，只要简单在方法前面加个async就可以了，**推荐异步写法**：

```python
from bilibili_api.live import LiveDanmaku

room = LiveDanmaku(room_display_id=114514)

@room.on("DANMU_MSG")  # 指定事件名
async def on_danmu(msg):
    print(msg)
    
if __name__ == "__main__":
    room.connect()
```

也可以采用 [add_event_handler](#add_event_handler) 来添加事件处理器。


常用事件名：

```
DANMU_MSG: 用户发送弹幕
SEND_GIFT: 礼物
WELCOME: 老爷进入房间
WELCOME_GUARD: 房管进入房间
NOTICE_MSG: 系统通知
PREPARING: 直播准备中
LIVE: 直播开始
ROOM_REAL_TIME_MESSAGE_UPDATE: 粉丝数等更新
ENTRY_EFFECT: 进场特效
ROOM_RANK: 房间排名更新
INTERACT_WORD: 用户进入直播间
ACTIVITY_BANNER_UPDATE_V2: 好像是房间名旁边那个xx小时榜
```

特殊事件名（本模块自定义事件）：
```
ALL: 所有事件均调用回调函数
VIEW: 直播间人气更新
```

回调数据格式：
```json
{
    "room_display_id": "房间展示ID，类型int", 
    "room_real_id": "房间真实ID，类型int", 
    "type": "事件名，类型str", 
    "data": "事件内容，类型根据事件名而定（一般是dict，VIEW事件是int）"
}
```

直播区更新速度快，以实际API为准，可以开debug自己看。

没有把代码写死，所以如果新增了事件应该也可以用。

事件名就是返回的 `cmd` 键对应的值。


### 连接多个直播间

按以下写法即可同时连接多个直播间


```python
from bilibili_api.live import LiveDanmaku, connect_all_livedanmaku

# 初始化不同房间的类
room1 = LiveDanmaku(114514, debug=True)
room2 = LiveDanmaku(1919810, debug=True)
# 可以添加多个，目前还没测试出最多多少


@room1.on('INTERACT_WORD')
async def rm1(event):
    # 房间1信息处理
    print(event)


@room2.on('INTERACT_WORD')
async def rm2(event):
    # 房间2信息处理
    print(event)


if __name__ == '__main__':
    # 使用动态参数传入
    connect_all_livedanmaku(room1, room2)
    # 也可以这样写
    room_list = [room1, room2]
    connect_all_livedanmaku(*room_list)
```


[Danmaku]: /docs/bilibili_api/模块/bilibili_api#Danmaku

[循环获取数据参数说明]: /docs/bilibili_api/通用解释#循环获取数据参数说明