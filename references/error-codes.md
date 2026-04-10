# Error Codes / 异常代码

## Inbound Exceptions / 入库异常

| Code | Description | Solution |
|------|-------------|----------|
| E101 | BCR scan exception | Conveyor rollback → exception port |
| E102 | Over dimension | Conveyor rollback → exception port |
| E103 | Business exception | Conveyor rollback → exception port |
| E104 | Duplicate inbound | Reject, notify Prime |
| E105 | No location available | Exception port → manual handling |
| E106 | Item family not allowed | Transfer to BCP location |
| E107 | No available location | Transfer to floor location |
| E108 | Duplicate pallet in inventory | Transfer to floor, investigate |
| E109 | Qty exceeds footprint | Transfer to floor, update footprint |

## Outbound Exceptions / 出库异常

| Code | Description | Solution |
|------|-------------|----------|
| E201 | Quality status change | Continue → return move_error |
| E202 | Cancel failed | Continue → return move_error |
| E203 | Pallet exception | Exception port → manual |
| E204 | No inventory found | Return "no inventory" to Prime |
| E205 | No matching inventory | Check Prime-WES inventory match |

## Task Status / 任务状态

| Status | Cancel | Edit Priority | Force Complete |
|--------|--------|---------------|-----------------|
| New / 新建 | ✅ | ✅ | ✅ |
| Accepted / 受理 | ❌ | ❌ | ✅ |
| Executing / 执行 | ❌ | ❌ | ✅ |
| Completed / 完成 | ❌ | ❌ | ❌ |
| Failed / 失败 | ❌ | ❌ | ❌ |

## System Errors / 系统异常

| Code | Description |
|------|-------------|
| E301 | Prime connection failed |
| E302 | IAC connection failed |
| E303 | WCS connection failed |
| E304 | PLC communication error |
| E305 | Database error |
| E306 | BCR equipment error |

## Virtual Container / 虚拟容器

| Code | Description |
|------|-------------|
| E401 | Location group has active task |
| E402 | Location group occupied |
| E403 | Virtual pallet name conflict |
