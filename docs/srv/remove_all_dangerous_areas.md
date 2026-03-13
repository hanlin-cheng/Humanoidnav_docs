# RemoveAllDangerousAreas

**服务类型**: `map_manager/srv/RemoveAllDangerousAreas`

**服务名**: `/map_manager/remove_all_dangerous_areas`

## 描述

删除所有限速区（可按楼层过滤）

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| count | int32 | 操作影响的项目数量 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/remove_all_dangerous_areas map_manager/srv/RemoveAllDangerousAreas \
  "{floor_id: '1F'}"
```
