# Business Rules / 业务规则

## Putaway Strategy / 上架策略

### Priority / 优先级

1. Same SKU + Same Batch + Same Location Group (同SKU同批次同库位组)
2. Empty Location Group - Deepest Position (空库位组最深层)
3. Exception - Transfer to Exception Station (异常 → 异常站)

### Special Rules / 特殊规则

- **退料 / Returns**: 优先单深位 / Single-deep preferred
- **非退料 / Non-returns**: 优先多深位 / Multi-deep preferred
- **物料类型 / Item Family**: 配置是否允许入四穿库

### Location Lock / 货位锁定

- 创建虚拟容器时锁闭整个库位组
- 四穿车顶起托盘时自动解锁

---

## Allocation Strategy / 分配策略

### Outbound Priority / 出库优先级

1. **Prime指定托盘号**: 严格按照托盘号分配
2. **指定Batch + Footprint Code**: 按MANDTE从小到大
3. **仅指定Footprint Code**: 按MANDTE排序，找最早批次

### Quality Status / 质量状态

- Status = U: 正常出库，回传pick_confirm
- Status ≠ U: 标记异常，回传move_error

---

## Inventory Status / 库存状态

### Inbound / 入库

1. WES接收 → 库存至上架暂存区
2. 四穿到达终点 → 库存移至密集存储区

### Outbound / 出库

1. WCS顶起托盘 → 库存移至下架暂存区
2. 到达出库工作站前端 → 任务完结
3. 光电感应为空 → 扣减库存

---

## Batch Rules / 批次规则

- 同一库位组只存放相同SKU相同批次
- 超水位提示不允许入库
- 预留空位用于倒库

### Batch Attributes / 批次属性

- LOTNUM (批号)
- FTPCOD (Footprint Code)
- MANDTE (生产日期)
- INVSTS (库存状态)
- EXPIRE_DTE (过期日期)
- 货主 (pc04-pc09)

---

## Task Priority / 任务优先级

- 数值越低，优先级越高
- 新建状态可修改优先级
- 范围: 1-50

### Priority Upgrade (IAC) / 优先级升级

- Range: 1-50
- Start: 50, End: 1
- Step: 每10分钟降2级
