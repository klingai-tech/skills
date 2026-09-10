# 可灵 MCP 工具协议与示例

CLI 是可灵后端 MCP server 的薄客户端：业务调用由 CLI 统一封装并发往后端 MCP 工具（传输与鉴权细节由 CLI 处理，无需关心）。本文档是工具协议速查；字段速查见 [`reference.md`](./reference.md)。

## 工具清单

| # | 工具名 | 分组 | 同步性 | 触达下游 | 一句话说明 |
|---|--------|------|--------|----------|------------|
| 1 | `who_am_i` | 能力发现 | 同步 | 否 | 身份 + 各生成工具可用模型与参数规格；**新会话先调** |
| 2 | `text_to_image` | 生成 | **异步** | 是 | 文生图，返回 `generationId`，需轮询 `query_tasks` |
| 3 | `image_to_image` | 生成 | **异步** | 是 | 参考图 + prompt 生图（需 inputs），返回 `generationId` |
| 4 | `text_to_video` | 生成 | **异步** | 是 | 文生视频，返回 `generationId`，需轮询 `query_tasks` |
| 5 | `image_to_video` | 生成 | **异步** | 是 | 图生视频（需 inputs），返回 `generationId` |
| 6 | `query_tasks` | 任务查询 | 同步 | 是 | 按 `generationId` 查询生成状态与最终资源 URL |
| 7 | `file_upload` | 文件上传 | 同步 | 是 | 申请一次性上传票据；文件字节由调用方自行上传（两步式，见下） |
| 8 | `element_create` / `element_list` / `element_get` / `element_update` / `element_delete` | 主体素材 | 同步 | 是 | 可复用 Element 的完整生命周期 |
| 9 | `motion_library_list` | 动作素材 | 同步 | 是 | 列出当前用户保存的动作素材 |
| 10 | `motion_control` | 生成 | **异步** | 是 | 主体图 + 动作视频或 motionId 生成视频 |
| 11 | `feedback` | 反馈 | 同步 | 是 | 上报卡住、计费异常或意外结果 |
| 12 | `query_membership_and_credits` | 商业化 | 同步 | 是 | 查询会员身份与可用灵感值（身份取自 JWT，无参数） |

> `kling <command> --help` 对上述 MCP-backed 命令会尽量实时读取该工具的 `tools/list` 声明（工具说明 + inputSchema）；完整模型清单与参数规格仍以 `who_am_i` 为准。示例：`kling image_to_image --help`。

客户端入口另有 `login`、`tool_list` 和 `account`（映射 `query_membership_and_credits`）；`logout` 调用同名 MCP 工具，成功后清除本地凭据。完整 19 个 CLI 命令见 [SKILL.md](./SKILL.md)。以下 JSON 是协议说明，不应绕过 CLI 直接发送请求。

## who_am_i

请求参数：无（身份取自 JWT）。返回示例：

```json
{
  "user": { "userId": 10000001 },
  "availableModels": {
    "text_to_image": [
      {
        "model": "kling-image-v3_0",
        "arguments": [
          { "name": "prompt", "required": true, "description": "Text prompt describing the image to generate" },
          { "name": "kolors_version", "required": false, "default": "3.0" },
          { "name": "img_resolution", "required": false, "default": "2k" },
          { "name": "aspect_ratio", "required": false, "default": "3:4" },
          { "name": "imageCount", "required": false, "default": "1", "description": "Number of images, max 9" }
        ],
        "inputs": []
      }
    ]
  },
  "authMode": "oauth"
}
```

- 模型名、参数、默认值均以**运行时返回**为准（服务端配置）。
- `arguments[]`：`required` 必填恒无默认值；`allowedValues` 不出现表示不限制；选填缺省时服务端回填 `default`。

## 生成类工具通用协议

5 个生成工具（含 `motion_control`）共用同一套入参信封与返回结构：

```json
{
  "model": "kling-image-v3_0",
  "arguments": [
    { "name": "prompt", "value": "two kids singing while running" },
    { "name": "imageCount", "value": "1" }
  ],
  "inputs": [
    { "name": "input", "inputType": "URL", "url": "https://cdn.example.com/ref.png" }
  ]
}
```

- `model` 必填，必须来自 who_am_i 该工具的清单。
- `arguments[].value` **一律为字符串**；省略选填项由服务端回填默认值。
- `inputs[].inputType` 当前仅 `"URL"`；`url` 须公网可访问：外部公网 URL（CDN / 外链、此前任务返回的 `works[].url`）直接使用，本地文件先 `file_upload` 换取 URL。`text_to_*` 通常无 inputs。
- 可选追踪参数（纯埋点、不参与校验、不透传下游；以 `tool_list` 的 inputSchema 声明为准）：`taskTraceId`（全部工具；CLI 全局 flag `--task-trace-id`，同一任务链复用同一 ID，不传时 CLI 静默生成 32 位字母数字 ID）、`rationale`（5 个生成工具；CLI flag `--rationale`，说明创作意图与参数理由，不传时 CLI 自动传空串）。

服务端在转发下游（扣费）前做本地校验，任一不过即报参数错误（**聚合列出所有问题项**）：model 在清单内、argument 名非空/不重复/已声明、必填不缺、值域命中、inputs 同理。

返回（GenerationSubmitResult）：

```json
{
  "generationId": "Qk1Zb3VyT3BhcXVlR2VuZXJhdGlvbklkRXhhbXBsZQ",
  "status": "submitted",
  "creditsConsumed": 10,
  "message": "Generation submitted. Poll query_tasks with this generationId to get the result."
}
```

## query_tasks

请求：`{ "generationId": "<generationId 原值>" }`。返回（已完成，实测状态为大写）：

```json
{
  "generationId": "AIUbOyYx...",
  "status": "COMPLETED",
  "createTime": 1781164986117,
  "finishTime": 1781165014373,
  "works": [
    {
      "status": "COMPLETED",
      "contentType": "image",
      "url": "https://cdn.example.com/.../out.png",
      "urlWithoutWatermark": "https://.../out_clean.png",
      "coverUrl": "https://.../cover.jpg",
      "coverUrlWithoutWatermark": "https://.../cover_clean.jpg"
    }
  ]
}
```

- `status` 为下游透传字符串，**按大小写不敏感处理**；中间态 `QUEUING`/`RUNNING`/`submitted`/`processing`，成功 `COMPLETED`/`PARTIAL_COMPLETED`/`succeed`。
- `generationId` 非法或非本人 → `Generation not found. Please verify the generationId.`

## file_upload（两步式）

第一步（MCP 工具，参数均选填但建议提供）：

```json
{ "filename": "photo.png", "contentType": "image/png", "size": 102400 }
```

返回 `{ "ticket": "...", "uploadUrl": "...", "expireAt": 1733900000 }`。

第二步（调用方自行执行，CLI 已封装）：向 `uploadUrl` 发 `multipart/form-data` POST，字段 `ticket`（票据）+ `file`（文件字节）。上传响应含文件 URL，可作为 `inputs[].url`。票据单次有效、过期作废。

## Element 主体

图片 Element 创建示例（本地路径由 CLI 先自动上传）：标签从当前区域 `element_create` 的实时工具 description 中选择并原样传入，不按对话语言翻译。以下示例假设目录包含 `Characters`；执行时将 CLI 与 payload 中的标签替换为实际目录中的角色标签。

```bash
kling element_create --name "Alice" --description "红发侦探" --tag Characters \
  --cover ./front.png --secondary ./side.png
```

对应 MCP payload：

```json
{
  "name": "Alice",
  "description": "红发侦探",
  "resource": {
    "cover": "https://cdn.example/front.png",
    "secondary": [
      { "inputType": "URL", "name": "secondary_1", "url": "https://cdn.example/side.png" }
    ]
  },
  "tags": ["Characters"]
}
```

视频 Element 将 `resource` 换为 `{ "video": "https://..." }`，可选 `voice`。`element_list` 无参数；`element_get` / `element_delete` 传 `{ "id": "..." }`。CLI 的 `element_update` 接受可选变更字段，内部先调用 `element_get` 并合成完整 payload；图片 Element 始终使用 `element_get` 返回的原 `cover`，用户不传 `--cover`。

## 动作库与动作控制

```bash
kling motion_library_list
kling motion_control --model <who_am_i 返回的模型> --image ./subject.png --motionId <动作库返回的ID> \
  --motionDirection image_direction --poll 300
```

`motion_control` 信封和其他生成工具相同。`inputs` 第一项为主体图片；使用动作视频时第二项为动作视频，具体 input 名称由 `who_am_i` 的模型声明决定；使用动作库时在 `arguments` 中传 `motionId`。两种动作来源必须二选一。

## feedback

```bash
kling feedback --summary "Generation completed with an empty works list" \
  --category EMPTY_OR_PARTIAL_RESULT --triggerMode user_initiated \
  --tool image_to_video --tool query_tasks --generationId <id>
```

先取得用户发送反馈的明确授权；上述 `user_initiated` 仅用于用户主动提出发送反馈的情形，Agent 先建议则使用 `agent_initiated`。`--category` 以实时工具声明为准。只发送脱敏摘要；可重复传 `--modelVersion` 和 `--relatedTaskTraceId` 关联模型与历史任务。该工具只上报反馈，不重试、不退款、不修复原任务；同一问题不要循环上报。

## query_membership_and_credits

请求参数：无（身份取自 JWT）。返回示例：

```json
{ "userId": 10000001, "membershipType": "NORMAL", "availableRemainCredits": 0 }
```

`membershipType`：`NORMAL` / `VIP` / `SVIP` / `SSVIP` / `SSSVIP`；`availableRemainCredits` 为用户可见的灵感值，无需换算。

> CLI 的 `account` 命令即此工具的直通调用。

## 工具发现（CLI `tool_list`）

用于查看后端 MCP server 当前实际暴露的工具元数据（`tools/list`），便于排障、确认灰度能力或读取工具 description 中的动态说明。

```bash
kling tool_list
```

返回结构示意（inputSchema 已简化；实际字段、必填项与值域以实时结果为准）：

```json
{
  "ok": true,
  "status": 200,
  "body": {
    "tools": [
      {
        "name": "text_to_image",
        "description": "Submit text-to-image generation.",
        "inputSchema": { "type": "object", "properties": { "model": { "type": "string" }, "arguments": { "type": "array", "items": { "type": "object" } }, "inputs": { "type": "array", "items": { "type": "object" } } } }
      }
    ]
  }
}
```

## 鉴权

鉴权全程由 `kling login`（浏览器 OAuth）封装，token 由 CLI 自动管理与续期。获取凭据**只有 `kling login` 这一种合法方式**，Agent 无需也不应手工构造任何鉴权请求或读取凭据内容。
