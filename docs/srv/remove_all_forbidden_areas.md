# RemoveAllForbiddenAreas

**服务类型**: `map_manager/srv/RemoveAllForbiddenAreas`

**服务名**: `${MM}/remove_all_forbidden_areas`

## 描述

无条件删除**所有楼层**的全部禁区。请求无字段。

与 [RemoveForbiddenAreas](remove_forbidden_areas.md) 的区别：
- `RemoveAllForbiddenAreas`：无字段，固定清空所有楼层
- `RemoveForbiddenAreas`：可按 `floor_id` 过滤（空串也表示所有楼层）

## 请求 (Request)

无字段。

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| count | int32 | 删除的禁区总数 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/remove_all_forbidden_areas map_manager/srv/RemoveAllForbiddenAreas "{}"
```
