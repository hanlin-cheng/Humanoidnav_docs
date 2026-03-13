# ListRooms

**服务类型**: `map_manager/srv/ListRooms`

**服务名**: `/map_manager/list_rooms`

## 描述

列出房间（可按楼层过滤）

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| rooms | Room[] | 房间列表 |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/list_rooms map_manager/srv/ListRooms \
  "{floor_id: '1F'}"
```
