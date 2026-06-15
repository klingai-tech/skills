# MCP 工具响应字段速查

完整协议与示例见同目录 **`api-examples.md`**。CLI 返回 `{ ok, status, body }`，`body` 为工具结果（已自动解析 JSON 文本块）。

## who_am_i（能力发现）

- **`user.user_id`**：当前用户 ID（来自 JWT）。
- **`available_models`**：工具名 → 模型能力清单；空 `{}` 表示服务端尚未配置。
  - **`[].model`**：模型名（生成命令 `--model` 的合法取值）。
  - **`[].arguments[]`**：`{ name, required, default?, allowed_values?, description? }`。
  - **`[].inputs[]`**：`{ name, required, description? }`（参考资源声明）。
- **`auth_mode`**：鉴权模式（OAuth 登录态下为 `oauth`）。

## 生成工具提交成功后（text_to_image / image_to_image / text_to_video / image_to_video）

- **`generation_id`**：不透明生成 ID，用于 `query_tasks` 轮询。
- **`status`**：初始状态（下游透传，如 `submitted` / `QUEUING`）。
- **`credits_consumed`**：本次消耗的灵感值（可能不出现）。

## query_tasks

- **`generation_id`**：回显查询 ID。
- **`status`**：任务整体状态（下游透传字符串，**大小写不敏感处理**）：
  - 中间态：`QUEUING` / `RUNNING` / `submitted` / `processing`
  - 成功终态：`COMPLETED` / `PARTIAL_COMPLETED` / `succeed`
  - 失败终态：`FAILED` / `CANCELLED` 等
- **`create_time`** / **`finish_time`**：毫秒时间戳（未完成时 finish_time 为 0）。
- **`works[]`**：产出列表：
  - **`status`**：单个产出状态。
  - **`content_type`**：`image` / `video`。
  - **`url`**：资源 URL（带水印，默认展示）。
  - **`url_without_watermark`**：无水印资源 URL（用户要求时展示）。
  - **`cover_url`** / **`cover_url_without_watermark`**：封面 URL。

## file_upload（两步式）

- 第一步（MCP 工具）返回：**`ticket`**（一次性票据）、**`upload_url`**（上传地址）、**`expire_at`**（过期时间戳）。
- 第二步由 CLI 自动完成：multipart POST（`ticket` + `file`）到 `upload_url`，CLI 会把响应中的文件 URL 规整到 `body.url`。

## account（query_membership_and_points 直通）

- **`userId`**：用户 ID。
- **`membershipType`**：会员身份（`NORMAL` / `VIP` / `SVIP` / `SSVIP` / `SSSVIP`）。
- **`availablePoints`**：可用灵感值（用户可见值，无需换算）。

## CLI 轮询结果（--poll / query_tasks --poll）

- **`body.polled`**：true 表示这是轮询聚合结果。
- **`body.timedOut`**：是否超时（超时后可继续 `query_tasks <generationId>`）。
- **`body.generations[]`**：`{ generationId, status, result }`，`result` 即最后一次 query_tasks 的返回体。
