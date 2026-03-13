## fourier_msgs/srv/GetCurrentAction

### 文件

fourier_msgs/srv/GetCurrentAction.srv

### 描述

获取当前正在执行的导航动作信息。

### 请求

无（空请求）

### 响应

| 元素 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 是否成功获取信息 |
| action_name | string | 当前动作名称，例如：<br/>`"navigate_to_pose"`<br/>`"follow_path"`<br/>`"spin"`<br/>`"backup"` |
| goal_status | action_msgs/GoalStatus | ROS2标准目标状态，包含：<br/>- goal_info（目标ID和时间戳）<br/>- status（状态码） |
| status_description | string | 人类可读的状态描述 |

### 目标状态码

| 状态码 | 名称 | 描述 |
| ------ | ---- | ---- |
| 0 | STATUS_UNKNOWN | 未知状态 |
| 1 | STATUS_ACCEPTED | 已接受 |
| 2 | STATUS_EXECUTING | 执行中 |
| 3 | STATUS_CANCELING | 取消中 |
| 4 | STATUS_SUCCEEDED | 成功完成 |
| 5 | STATUS_CANCELED | 已取消 |
| 6 | STATUS_ABORTED | 已中止 |
