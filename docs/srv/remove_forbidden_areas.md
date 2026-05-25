# RemoveForbiddenAreas

**服务类型**: `map_manager/srv/RemoveForbiddenAreas`

**服务名**: `${MM}/remove_forbidden_areas`

## 描述

按楼层批量删除禁区。`floor_id` 为空串时，删除所有楼层的禁区。

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识；空串表示所有楼层 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| count | int32 | 删除的禁区数量 |

## 使用示例

### 命令行调用

```bash
ros2 service call ${MM}/remove_forbidden_areas map_manager/srv/RemoveForbiddenAreas \
  "{floor_id: '1F'}"

```
