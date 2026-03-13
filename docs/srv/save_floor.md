# SaveFloor

**服务类型**: `map_manager/srv/SaveFloor`

**服务名**: `/map_manager/save_floor`

## 描述

保存当前楼层数据到磁盘

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| saved_path | string | 实际保存路径 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/save_floor map_manager/srv/SaveFloor \
  "{floor_id: '1F'}"
```
