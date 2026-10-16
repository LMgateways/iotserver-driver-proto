# iotserver-driver-proto

iotserver 物联网平台的 **驱动 gRPC 协议定义**。
平台核心与协议驱动之间通过 gRPC 双向通信，本仓库是双方共同遵守的接口契约。

## 双向通信模型

```
┌──────────────┐          ┌──────────────────────────────┐
│ 驱动（容器内） │  gRPC   │ 平台核心（iotserver-core）      │
│              │ ◄══════► │                               │
│  server 端   │  client │  client 端        server 端     │
│ (平台调驱动)  │ (驱动调平台)                              │
└──────────────┘          └──────────────────────────────┘
```

### 驱动实现（平台调驱动）

| 服务 | 方法 | 触发时机 |
| --- | --- | --- |
| `Common` | `Ping` / `Version` | 平台健康检查 |
| `DeviceCallBackService` | `Create` / `Update` / `Delete` | 设备增删改、绑定/解绑驱动 |
| `ProductCallBackService` | `Create` / `Update` / `Delete` | 产品增删改 |
| `ThingModelDownService` | `MsgIssue` | 属性设置/查询、服务调用 |
| `CloudInstanceCallBackService` | `StatueCallback` | 云实例状态变更 |

### 平台提供（驱动调平台）

| 服务 | 方法 | 用途 |
| --- | --- | --- |
| `ThingModelUpService` | `ThingModelMsgReport` | **上报设备数据**（属性/事件/批量） |
| `RpcDevice` | `QueryDeviceList` 等 | 查询本驱动关联的设备 |
| `RpcProduct` | `QueryProductById` 等 | 查询设备物模型 |
| `RpcGateway` | `GetGatewayInfo` | 网关信息 |
| `RpcCustomMqttMessage` | 自定义 MQTT 主题收发 | 高级用法 |
| `DriverStorage` | 键值存储 | 驱动持久化自身状态 |

## 目录

```
drivercommon/           公共类型（BaseRequestMessage / CommonResponse / IotPlatform 枚举）
thingmodel/             物模型上报与下发（核心）
driverdevice/           设备 CRUD 与连接状态
driverproduct/          产品查询
devicecallback/         设备变更回调（平台 → 驱动）
productcallback/        产品变更回调（平台 → 驱动）
cloudinstance/          云实例管理
cloudinstancecallback/  云实例状态回调
gateway/                网关信息
custommqttmessage/      自定义 MQTT 消息
driverstorge/           驱动侧键值存储
```

## 重新生成 Go 代码

改了 `.proto` 后（需要 `protoc` + `protoc-gen-go` + `protoc-gen-go-grpc`）：

```bash
make gen-all
```

## 使用

```go
import (
    "github.com/LMgateways/iotserver-driver-proto/thingmodel"
    "github.com/LMgateways/iotserver-driver-proto/drivercommon"
)
```

## 与上游的差异

fork 自 `github.com/winc-link/edge-driver-proto`，改动仅限工程层面：

- 模块路径改为 `github.com/LMgateways/iotserver-driver-proto`；
- 补上原仓库缺失的 `go.mod`（原仓库依赖 Go 自动合成）；
- 各 `.proto` 的 `go_package` 选项同步更新。

**协议本身（消息、方法、字段）与上游完全一致**，因此基于本仓库构建的驱动与上游平台仍可互通。