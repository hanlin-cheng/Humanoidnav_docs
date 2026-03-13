# AddRoom

**服务类型**: `map_manager/srv/AddRoom`

**服务名**: `/map_manager/add_room`

## 描述

添加房间定义

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| room | Room | 房间数据 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| room_id | string | 房间 ID |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/add_room map_manager/srv/AddRoom \
  "{floor_id: '1F'}"
```
