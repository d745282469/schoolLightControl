# 校园灯控MQTT订阅

| 版本号 |                           修改内容                           |  修改人   |
| :----: | :----------------------------------------------------------: | :-------: |
|  v1.0  |                         确定基础逻辑                         |   潘东    |
|  V2.1  | 调整单独控制灯开关的数据格式；分开客户端发的指令和设备端发的指令；调整订阅主题逻辑；增加读取设备状态指令； | 潘东/Qurn |
|  V2.2  |                   调整设备端上报状态指令；                   | 潘东/Qurn |
|  V2.3  |                             新增                             |           |



#### 使用前必读

所有主题均以`schoolLight/`作为开头，后根据不同的需求再细分不同的主题。

假设学校A的topic为`A-ID-1234`，教学楼A栋topic为`A-BUILDING-1234`，A栋1楼topic为`A-FLOOR-1234`，501室的topic为`A-ROOM-1234`，其中有1台主控板控制4个区域共20盏灯，主控板topic为`A-CONTROL-1234`，区域划分为`A区、B区、C区、D区`，区域下的灯根据索引进行控制。

下面所有订阅主题例子，均以上述环境作为基础。**注意：所有topic信息请以服务端返回为准，长度最长为22个字符。**

### 客户端发

#### 控制灯开关

**主题：**`schoolLight/学校topic/建筑topic/楼层topic/房间topic/设备topic`

**服务质量：**`qos1`

**消息内容：**

|       key       |  type  | 是否必须 |            备注            |
| :-------------: | :----: | :------: | :------------------------: |
|       cmd       | string |    是    | 操作命令，此处固定为switch |
|      data       | array  |    是    |        数据json数组        |
|  data.location  | string |    是    |          控制区域          |
| data.lightIndex |  int   |    是    |      区域内灯的索引值      |
|   data.status   |  int   |    是    |        0：开，1：关        |

**示例：**

**打开**学校A的教学楼A栋1楼501室**A区的第0盏灯**，**B区的第1盏灯**。

主题：schoolLight/A-ID-1234/A-BUILDING-1234/A-FLOOR-1234/A-ROOM-1234/A-CONTROL-1234

```json
{
    "cmd":"switch",
    "data":[
        {
            "location":"A",
            "lightIndex":0,
            "status":0
        },
        {
            "location":"B",
            "lightIndex":1,
            "status":0
        }
    ]
}
```



#### 控制区域灯开关

**主题：**`schoolLight/学校topic/建筑topic/楼层topic/房间topic/设备topic`

**服务质量：**`qos1`

**消息内容：**

|   key    |  type  | 是否必须 |            备注            |
| :------: | :----: | :------: | :------------------------: |
|   cmd    | string |    是    | 操作命令，此处固定为switch |
| location | string |    是    |          区域名称          |
|  status  |  int   |    是    |    灯状态，0：开，1：关    |

**示例：**

**打开**学校A的教学楼A栋1楼501室**A区的第0盏灯**，**B区的全部灯**。

主题：schoolLight/A-ID-1234/A-BUILDING-1234/A-FLOOR-1234/A-ROOM-1234/A-CONTROL-1234

```json
{
    "cmd":"switch",
    "location":"A",
    "lightIndex":0
}
```



#### 控制设备指定索引的灯开关

**主题：**`schoolLight/学校topic/建筑topic/楼层topic/房间topic/设备topic`

**服务质量：**`qos1`

**消息内容：**

|    key     |  type  | 是否必须 |            备注            |
| :--------: | :----: | :------: | :------------------------: |
|    cmd     | string |    是    | 操作命令，此处固定为switch |
| lightIndex |  int   |    是    |           索引号           |
|   status   |  int   |    是    |    灯状态，0：开，1：关    |

**示例：**

**打开**学校A的教学楼A栋1楼501室**A区的第0盏灯**，**索引号为0的全部灯**。

主题：schoolLight/A-ID-1234/A-BUILDING-1234/A-FLOOR-1234/A-ROOM-1234/A-CONTROL-1234

```json
{
    "cmd":"switch",
    "lightIndex":0,
    "status":0
}
```



#### 控制某个房间所有灯开关

**主题：**``schoolLight/学校topic/建筑topic/楼层topic/房间topic``

**服务质量：**``qos1``

**消息内容：**

|  key   |  type  | 是否必须 |            备注            |
| :----: | :----: | :------: | :------------------------: |
|  cmd   | string |    是    | 操作命令，此处固定为switch |
| status |  int   |    是    |        0：开，1：关        |

**示例：**

**关闭**学校A的教学楼A栋1楼501室的全部灯。

主题：schoolLight/A-ID-1234/A-BUILDING-1234/A-FLOOR-1234/A-ROOM-1234

```json
{
    "cmd":"switch",
    "status":1
}
```



#### 控制某层楼所有灯开关

**主题：**``schoolLight/学校topic/建筑topic/楼层topic``

**服务质量：**``qos1``

**消息内容：**

|  key   |  type  | 是否必须 |            备注            |
| :----: | :----: | :------: | :------------------------: |
|  cmd   | string |    是    | 操作命令，此处固定为switch |
| status |  int   |    是    |        0：开，1：关        |

**示例：**

**关闭**学校A的教学楼A栋1楼的全部灯。

主题：schoolLight/A-ID-1234/A-BUILDING-1234/A-FLOOR-1234

```json
{
    "cmd":"switch",
    "status":1
}
```



#### 控制某栋楼所有灯开关

**主题：**``schoolLight/学校topic/建筑topic``

**服务质量：**``qos1``

**消息内容：**

|  key   |  type  | 是否必须 |            备注            |
| :----: | :----: | :------: | :------------------------: |
|  cmd   | string |    是    | 操作命令，此处固定为switch |
| status |  int   |    是    |        0：开，1：关        |

**示例：**

**关闭**学校A的教学楼A栋的全部灯。

主题：schoolLight/A-ID-1234/A-BUILDING-1234

```json
{
    "cmd":"switch",
    "status":1
}
```



#### 控制某个学校所有灯开关

**主题：**``schoolLight/学校topic``

**服务质量：**``qos1``

**消息内容：**

|  key   |  type  | 是否必须 |            备注            |
| :----: | :----: | :------: | :------------------------: |
|  cmd   | string |    是    | 操作命令，此处固定为switch |
| status |  int   |    是    |        0：开，1：关        |

**示例：**

**关闭**学校A的全部灯。

主题：schoolLight/A-ID-1234

```json
{
    "cmd":"switch",
    "status":1
}
```



#### 读取设备当前状态

**主题：**``schoolLight/学校topic/建筑topic/楼层topic/房间topic/设备topic``

**服务质量：**``qos1``

**消息内容：**

| key  |  type  | 是否必须 |              备注              |
| :--: | :----: | :------: | :----------------------------: |
| cmd  | string |    是    | 操作命令，此处固定为get_status |

**示例：**

读取学校A，教学楼A栋1楼501室的设备状态。

主题：schoolLight/A-ID-1234/A-BUILDING-1234/A-FLOOR-1234/A-ROOM-1234/A-CONTROL-1234

```json
{
    "cmd":"get_status"
}
```



### 设备端发

#### 上报设备当前状态

**主题：**``lightGateway``

**服务质量：**``qos1``

**消息内容：**

|          key          |   type    | 是否必须 |                  备注                   |
| :-------------------: | :-------: | :------: | :-------------------------------------: |
|          cmd          |  string   |    是    | 操作命令，此处固定为report_gateway_info |
|       client_id       |  string   |    是    |                 设备id                  |
|         light         | jsonArray |    是    |               灯状态数组                |
|    light.location     |  string   |    是    |                区域名称                 |
|   light.lightIndex    |    int    |    是    |             灯的区域内索引              |
|     light.status      |    int    |    是    |         灯的状态，0：开，1：关          |
|      brightness       | jsonArray |    是    |              区域亮度数组               |
|  brightness.location  |  string   |    是    |                区域名称                 |
| brightness.brightness |    int    |    是    |              区域内的亮度               |

**示例：**

上报学校A，教学楼A栋1楼501室的设备状态。

主题：lightGateway

```json
{
    "cmd":"report_gateway_info",
    "client_id":"bf5365a6-e4a2-11ed-9cd1-00163e02d664",
    "light":[
        {
            "location":"A",
            "lightIndex":0,
            "status":0
        },
        {
            "location":"A",
            "lightIndex":1,
            "status":1
        }
    ],
    "brightness":[
        {
            "location":"A",
            "brightness":50
        }
    ]
}
```

