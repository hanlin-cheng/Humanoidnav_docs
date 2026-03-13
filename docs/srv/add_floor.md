# AddFloor

**服务类型**: `map_manager/srv/AddFloor`

**服务名**: `/map_manager/add_floor`

## 描述

添加新楼层到复合地图

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor | Floor | 楼层信息 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/add_floor map_manager/srv/AddFloor \
  "{}"
```
