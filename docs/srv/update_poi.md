# UpdatePOI

**服务类型**: `map_manager/srv/UpdatePOI`

**服务名**: `/map_manager/update_poi`

## 描述

更新 POI 信息

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| poi | POI | POI 数据 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/update_poi map_manager/srv/UpdatePOI \
  "{}"
```
