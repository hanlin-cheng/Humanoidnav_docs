# ListFloors

**服务类型**: `map_manager/srv/ListFloors`

**服务名**: `/map_manager/list_floors`

## 描述

列出所有楼层信息

## 请求 (Request)

*无请求参数*

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floors | Floor[] | 楼层列表 |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/list_floors map_manager/srv/ListFloors
```
