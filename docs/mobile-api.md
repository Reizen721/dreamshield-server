# DreamShield 手机 HTTPS 接口

本约定对应服务端 v0.1.6 的手机网关协议 1。启动脚本默认将电脑控制台绑定在回环地址的 8016 端口，手机 HTTPS 网关使用 8746 端口。手机配对使用电脑局域网地址，配对和请求均遵循网关提供的地址和证书指纹。

## 发现、配对和会话

| 方法和路径 | 作用 | 请求与响应约定 |
|---|---|---|
| 电脑本地 GET /api/mobile/setup | 为本机配对页面提供信息 | 配对信息包含 version、base_url、certificate_sha256、pairing_code。电脑页面为 /mobile-setup。 |
| HTTPS GET /api/mobile/health | 检查手机网关 | 响应包含 ok、server_name、protocol_version。 |
| HTTPS POST /api/mobile/pair | 一次性配对 | 请求键为 pairing_code、device_name。成功响应包含 ok、token、expires_at、persistent、device_name、server_name。 |

配对码有效期为 300 秒，成功配对后用过的码失效。后续受保护请求使用 Authorization: Bearer TOKEN。协议保留 ISO 格式的 expires_at 字段供旧客户端兼容。服务器可从电脑的配对页面撤销会话。POST 请求使用 application/json。

## 已配对手机的读写接口

| 方法与路径 | 用途 | 参数或响应要点 |
|---|---|---|
| GET /api/app/status | 读取应用状态 | 供两款客户端刷新设备、环境和任务状态。 |
| GET /api/device/status、/api/device/snapshot | 设备状态与实时画面 | 快照允许数字 t 或 _ 查询参数，画面为 JPEG。 |
| GET /api/environment/current | 当前温湿度 | Android 使用。iOS 0.1.1 从应用状态读取环境。 |
| GET /api/environment/history、/api/captures | 环境和抓拍记录 | 支持数字 limit。 |
| GET /api/automatic-ai/history | 读取 AI 观察记录 | 记录文件按服务器返回的路径访问。 |
| GET /captures/…、/automatic-ai-results/… | 读取记录文件 | 抓拍可读 JPEG 和 JSON，AI 记录文件为 JSON。 |
| POST /api/config | 更新基础参数 | 可提交 capture_interval_seconds、schedule_enabled、environment_recording_enabled、environment_interval_seconds。 |
| POST /api/capture | 对当前配置的设备抓拍 | 请求 JSON 为空对象。 |
| POST /api/automatic-ai | 更新原有自动 AI 设置 | enabled 必填。开启时 cloud_upload_consent 必须为 true。可设置调用间隔和预算。 |
| GET、POST /api/policy-settings | 读取或修改温控预设 | GET 返回当前预设、策略、默认值和限制。POST 可提交 preset、policy、environment_interval_seconds、sensor_sample_target_seconds。 |
| POST /api/sensor-settings | 应用 SHT31 采样周期 | 请求字段 sample_interval_seconds 为 5～30 的整数。设备确认后记录当前值。 |

预设名为 light、balanced、efficient、custom。编辑参数时使用 custom。保存策略会暂停自动任务，设备采样目标需通过采样接口另行应用。策略设置与采样入口在 Android 0.6.1 可用。

热像时序闭环、设备配网和直接空调控制由电脑端操作。网关按允许路由转发，凭据缺失或失效返回 401，允许范围外的路径返回 403，无效参数返回 400。实现以 [mobile_gateway.py](../backend/app/mobile_gateway.py) 和 [server.py](../backend/app/server.py) 为准。
