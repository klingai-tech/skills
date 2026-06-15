# 可灵 MCP 工具协议与示例

CLI 是可灵后端 MCP server 的薄客户端：所有业务调用都是 MCP `tools/call`（Streamable HTTP，`{baseUrl}/mcp`，`Authorization: Bearer`）。本文档是工具协议速查；字段速查见 [`reference.md`](./reference.md)。

## 工具清单

| # | 工具名 | 分组 | 同步性 | 触达下游 | 一句话说明 |
|---|--------|------|--------|----------|------------|
| 1 | `who_am_i` | 能力发现 | 同步 | 否 | 身份 + 各生成工具可用模型与参数规格；**新会话先调** |
| 2 | `text_to_image` | 生成 | **异步** | 是 | 文生图，返回 `generation_id`，需轮询 `query_tasks` |
| 3 | `image_to_image` | 生成 | **异步** | 是 | 参考图 + prompt 生图（需 inputs），返回 `generation_id` |
| 4 | `text_to_video` | 生成 | **异步** | 是 | 文生视频，返回 `generation_id`，需轮询 `query_tasks` |
| 5 | `image_to_video` | 生成 | **异步** | 是 | 图生视频（需 inputs），返回 `generation_id` |
| 6 | `query_tasks` | 任务查询 | 同步 | 是 | 按 `generation_id` 查询生成状态与最终资源 URL |
| 7 | `file_upload` | 文件上传 | 同步 | 是 | 申请一次性上传票据；文件字节由调用方自行上传（两步式，见下） |
| 8 | `query_membership_and_points` | 商业化 | 同步 | 是 | 查询会员身份与可用灵感值（身份取自 JWT，无参数） |

## who_am_i

请求参数：无（身份取自 JWT）。返回示例：

```json
{
  "user": { "user_id": 10000001 },
  "available_models": {
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
  "auth_mode": "oauth"
}
```

- 模型名、参数、默认值均以**运行时返回**为准（服务端配置）。
- `arguments[]`：`required` 必填恒无默认值；`allowed_values` 不出现表示不限制；选填缺省时服务端回填 `default`。

## 生成类工具通用协议

4 个生成工具共用同一套入参信封与返回结构：

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
- `inputs[].inputType` 当前仅 `"URL"`；`url` 须公网可访问（本地文件先 `file_upload`）。`text_to_*` 通常无 inputs。

服务端在转发下游（扣费）前做本地校验，任一不过即报参数错误（**聚合列出所有问题项**）：model 在清单内、argument 名非空/不重复/已声明、必填不缺、值域命中、inputs 同理。

返回（GenerationSubmitResult）：

```json
{
  "generation_id": "Qk1Zb3VyT3BhcXVlR2VuZXJhdGlvbklkRXhhbXBsZQ",
  "status": "submitted",
  "credits_consumed": 10,
  "message": "Generation submitted. Poll query_tasks with this generation_id to get the result."
}
```

## query_tasks

请求：`{ "generationId": "<generation_id 原值>" }`。返回（已完成，实测状态为大写）：

```json
{
  "generation_id": "AIUbOyYx...",
  "status": "COMPLETED",
  "create_time": 1781164986117,
  "finish_time": 1781165014373,
  "works": [
    {
      "status": "COMPLETED",
      "content_type": "image",
      "url": "https://cdn.example.com/.../out.png",
      "url_without_watermark": "https://.../out_clean.png",
      "cover_url": "https://.../cover.jpg",
      "cover_url_without_watermark": "https://.../cover_clean.jpg"
    }
  ]
}
```

- `status` 为下游透传字符串，**按大小写不敏感处理**；中间态 `QUEUING`/`RUNNING`/`submitted`/`processing`，成功 `COMPLETED`/`PARTIAL_COMPLETED`/`succeed`。
- `generation_id` 非法或非本人 → `Generation not found. Please verify the generation_id.`

## file_upload（两步式）

第一步（MCP 工具，参数均选填但建议提供）：

```json
{ "filename": "photo.png", "contentType": "image/png", "size": 102400 }
```

返回 `{ "ticket": "...", "upload_url": "...", "expire_at": 1733900000 }`。

第二步（调用方自行执行，CLI 已封装）：向 `upload_url` 发 `multipart/form-data` POST，字段 `ticket`（票据）+ `file`（文件字节）。上传响应含文件 URL，可作为 `inputs[].url`。票据单次有效、过期作废。

## query_membership_and_points

请求参数：无（身份取自 JWT）。返回示例：

```json
{ "userId": 10000001, "membershipType": "NORMAL", "availablePoints": 0.0 }
```

`membershipType`：`NORMAL` / `VIP` / `SVIP` / `SSVIP` / `SSSVIP`；`availablePoints` 为用户可见的灵感值，无需换算。

> CLI 的 `account` 命令即此工具的直通调用。

## 鉴权（OAuth，CLI `login` 已封装）

1. `GET {base}/.well-known/oauth-protected-resource` → `resource` + `authorization_servers[0]`（issuer）。
2. `GET {origin}/.well-known/oauth-authorization-server{issuerPath}`（RFC 8414 path-insert）→ 端点 + `scopes_supported`。
3. DCR `POST /auth/register`（公共 native client，`token_endpoint_auth_method: none`）。
4. 浏览器 `GET /auth/authorize`（PKCE S256 + `resource` 参数）→ 回调 127.0.0.1 拿 code。
5. `POST /auth/token`（form-urlencoded，**必须带与 authorize 一致的 `resource`**，否则 `invalid_target`）→ access/refresh token。
6. 刷新：`grant_type=refresh_token`（同样带 `resource`）。
