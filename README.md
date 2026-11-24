# IFCServer — 商场智能跟随购物车 · 后端

 NestJS 后端：登录鉴权、车辆绑定、指令下发与审计、告警规则、历史查询。
 同时是 MQTT 消息的中枢（订阅车端 status/track，向前端推送，代理下发 cmd）。

- 技术栈（规划）：NestJS + Prisma + zod，全 TypeScript strict
- 协议合同：`docs/protocol.md`（**权威副本**，Web 与小程序仓库里的 protocol.md 以本文件为准）
- 硬约束：指令统一由后端下发，Web / 小程序前端不允许直接 publish `car/{id}/cmd`

## 目录

```
docs/               # 协议合同 + 功能清单
docker-compose.yml  # 本地/服务器：MySQL 8 + Redis 7 + EMQX（未启动过，首次使用需验证）
```

> 代码骨架（NestJS）尚未创建，待 Web 控制页假数据版完成后再动手，接口形状以页面需求为准。

## 相关仓库

- Web 管理端：`SmartFollowingShoppingCart-WebAdmin-`
- 小程序用户端：`SmartFollowingShoppingCart-MiniApp`
