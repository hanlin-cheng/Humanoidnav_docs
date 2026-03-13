# UpdateForbiddenArea

**服务类型**: `map_manager/srv/UpdateForbiddenArea`

**服务名**: `/map_manager/update_forbidden_area`

## 描述

更新禁区边界

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| area | ForbiddenArea | 区域数据 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| wall_ids | int32[] | 虚拟墙 ID 列表 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/update_forbidden_area map_manager/srv/UpdateForbiddenArea \
  "{floor_id: '1F'}"
```
