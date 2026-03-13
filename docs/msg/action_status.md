## fourier_msgs/msg/ActionStatus

### 文件

fourier_msgs/msg/ActionStatus.msg

### 描述

当前动作状态消息，由 `robot_monitor_node` 发布，用于报告导航动作的执行状态。

### 定义

| 元素 | 类型 | 描述 |
| ---- | ---- | ---- |
| action_name | string | 动作名称，例如：<br/>`"navigate_to_pose"`<br/>`"follow_path"`<br/>`"spin"`<br/>`"backup"`<br/>`"wait"` |
| status | uint8 | 状态码（见下表） |
| status_description | string | 人类可读的状态描述 |
| stamp | builtin_interfaces/Time | 状态记录的时间戳 |

### 状态码定义

| 状态码 | 常量名 | 描述 |
| ------ | ------ | ---- |
| 0 | STATUS_UNKNOWN | 未知状态 |
| 1 | STATUS_ACCEPTED | 动作已被接受 |
| 2 | STATUS_EXECUTING | 动作执行中 |
| 3 | STATUS_CANCELING | 动作取消中 |
| 4 | STATUS_SUCCEEDED | 动作成功完成 |
| 5 | STATUS_CANCELED | 动作已取消 |
| 6 | STATUS_ABORTED | 动作已中止（失败） |

### 话题信息

- **话题名**: `/action_status`
- **发布节点**: `robot_monitor_node`
- **发布频率**: 1 Hz

### 监控的动作列表

默认监控以下导航动作：
- `navigate_to_pose` - 导航到目标点
- `spin` - 原地旋转
- `backup` - 后退
- `wait` - 等待
