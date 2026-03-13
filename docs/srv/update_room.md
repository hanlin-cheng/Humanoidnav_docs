# UpdateRoom

**服务类型**: `map_manager/srv/UpdateRoom`

**服务名**: `/map_manager/update_room`

## 描述

更新房间信息

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

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/update_room map_manager/srv/UpdateRoom \
  "{floor_id: '1F'}"
```
