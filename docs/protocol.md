# 软硬件通信协议 v1.0

> 软硬件双方共同维护。改动需双方确认并升版本号（v1.1、v1.2 …）。
> 真车由队友开发；软件端可用模拟车脚本按本协议模拟车端，前后端开发不等硬件。

## 1. 通信方式

| 端 | 方式 | 说明 |
|---|---|---|
| 车端 | WiFi 模块，MQTT over TCP 接入 EMQX | 队友实现 |
| 小程序 / Web | MQTT over WSS | 只订阅和展示 |
| 后端 | MQTT over TCP | 指令**统一由后端下发**，前端不许直发 |
| 消息格式 | JSON，UTF-8 | Topic 中 `{id}` 为车辆编号，如 `car001` |

## 2. Topic 定义

| Topic | 方向 | QoS | 用途 |
|---|---|---|---|
| `car/{id}/status` | 车 → 云 | 0 | 1~2 秒周期状态上报 |
| `car/{id}/cmd` | 云 → 车 | 1 | 后端下发控制指令 |
| `car/{id}/ack` | 车 → 云 | 1 | 指令回执 |
| `car/{id}/event` | 车 → 云 | 1 | 事件上报 |
| `car/{id}/track` | 车 → 云 | 0 | 轨迹点，5 秒一个 |

## 3. 消息格式

### status 状态上报

```json
{ "soc": 78, "speed": 0.6, "mode": "follow", "dist": 1.2, "ang": 15, "ts": 1763798400 }
```

`soc` 电量(%) · `speed` 速度(m/s) · `mode` 模式(follow / pause / idle) · `dist` UWB 距离(m) · `ang` 方位角(°) · `ts` 时间戳(秒)

### cmd 指令下发

```json
{ "cmdId": "c123", "cmd": "estop" }
```

`cmd` 取值：`estop` 急停 · `recall` 一键召回 · `follow_start` 启动跟随 · `follow_stop` 暂停跟随 · `set_param` 参数设置

### ack 指令回执

```json
{ "cmdId": "c123", "result": "ok" }
```

`result` 取值：`ok` 成功 · `error` 参数错误 · `busy` 忙碌拒绝

### event 事件上报

```json
{ "type": "bind_success", "ts": 1763798400 }
```

`type` 取值：`bind_success` 绑定成功 · `target_lost` 目标丢失 · `estop` 急停触发 · `low_battery` 低电量 · `offline` 离线

### track 轨迹点

```json
{ "ts": 1763798400, "x": 12.5, "y": 3.2, "yaw": 90, "speed": 0.6, "soc": 78 }
```

`x` / `y` 为场地坐标(米)，`yaw` 朝向(°)；无定位条件时上报相对起点坐标。

## 4. 时序与异常约定

- 下发 `cmd` 后车端须在 **3 秒**内回 `ack`；超时视为失败，服务端重试 1 次，仍失败则前端提示网络异常
- 车端超过 **10 秒**未上报 `status`，判定离线，自动产生 `offline` 事件
- `estop` 急停：车端收到后**无条件立即执行**，不等待回执流程
- **安全兜底（车端本地，不依赖云端）**：云端连接断开超过 5 秒自动刹停；K230 检测到障碍立即停止
- 轨迹按 5 秒抽稀，避免数据量过大
