## fourier_msgs/srv/CancelCurrentAction

### 文件

fourier_msgs/srv/CancelCurrentAction.srv

### 描述

取消当前正在执行的导航动作。

### 请求

无（空请求）

### 响应

| 元素 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 取消操作是否成功 |
| message | string | 结果描述信息 |

### 适用场景

- 紧急停止导航任务
- 用户手动中断机器人移动
- 切换到新的导航目标前取消当前任务

### 注意事项

- 取消动作后，机器人会停止当前运动
- 如果没有正在执行的动作，服务仍会返回成功
