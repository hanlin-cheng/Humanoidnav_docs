## fourier_msgs/msg/EventsInfo

### 文件

fourier_msgs/msg/EventsInfo.msg

### 描述

聚合后的系统事件消息，由 `events_node` 发布，包含所有组件的事件汇总。

### 定义

| 元素 | 类型 | 描述 |
| ---- | ---- | ---- |
| events | BaseEventInfo[] | 聚合后的所有事件 |

### 话题信息

- **话题名**: `/Humanoid_nav/events`
- **发布节点**: `events_node`
- **发布频率**: 10 Hz（由参数 `events_pub_period` 控制，默认 0.1s；可通过 launch 参数覆盖）
- **发布行为**: 每个发布周期都会发布一次（**即使 `events` 数组为空**），相当于心跳。下游可据此判断 events_node 是否在线

### 相关消息

- [BaseEventInfo](base_event_info.md) - 基础事件信息
