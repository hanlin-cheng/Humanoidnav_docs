# ListSemanticObjects

**服务类型**: `map_manager/srv/ListSemanticObjects`

**服务名**: `/map_manager/list_semantic_objects`

## 描述

列出语义对象（可按楼层和类型过滤）

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| object_type | string | 对象类型过滤器 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| objects | SemanticObject[] | 语义对象列表 |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/list_semantic_objects map_manager/srv/ListSemanticObjects \
  "{floor_id: '1F', object_type: ''}"
```
