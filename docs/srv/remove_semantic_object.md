# RemoveSemanticObject

**服务类型**: `map_manager/srv/RemoveSemanticObject`

**服务名**: `/map_manager/remove_semantic_object`

## 描述

删除指定语义对象

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| object_id | string | 对象 ID |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/remove_semantic_object map_manager/srv/RemoveSemanticObject \
  "{floor_id: '1F', object_id: ''}"
```
