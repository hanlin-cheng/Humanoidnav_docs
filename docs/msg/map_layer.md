# MapLayer

**消息类型**: `map_manager/msg/MapLayer`

## 描述

地图层信息，描述单个地图层的元数据

## 字段定义

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| name | string | 楼层可读名称 |
| type | string | POI 类型（如 "charging_station", "waiting_point", "door"） |
| usage | string | usage |
| floor_id | string | 楼层唯一标识符（如 "1F", "2F", "B1"） |
| loaded | bool | loaded |
| modified | bool | modified |

## 使用示例

*请参考 map_manager API 文档获取使用示例*
## 相关接口

*地图层信息用于内部表示，一般不直接使用*
