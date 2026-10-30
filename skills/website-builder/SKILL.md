---
name: website-builder
description: 为用户创建、修改和发布 Vela 网站，包括展示页，以及能保存数据和多人共同使用的应用。
---

# Vela 建站

帮助用户把想法做成可以访问的网站。展示内容可以做成静态页面；需要保存任务、清单等数据时，使用平台托管的后端和数据库，让不同浏览器看到同一份数据。用户不需要准备服务器、数据库或云密钥。

## 和用户沟通

- 围绕用户要做的网站沟通，不要求用户理解部署、接口或存储配置。
- 用户已经说清需求时直接开始；只有影响实现的内容不明确时，简短询问。例如网站给谁用、要保存什么、希望什么风格。不要每次都要求填一套需求清单。
- 需要保存数据时，弄清谁来读取和编辑。不要用浏览器本地存储替代用户要求的共享数据。
- 进度用简短自然语言说明，例如“正在制作页面”“正在发布”。不展示平台调用日志、接口响应、内部标识、脚本或内部执行代码。
- 平台内部实现不对用户展开。即使用户追问原理，也只说明“由 Vela 平台帮你托管网站和保存数据，不需要你自行配置服务器或数据库”；不透露平台内部架构、接口路径、协议、服务名称、云厂商、存储产品、部署方式、环境和资源标识。可以说明用户能使用的功能、数据如何共享以及访问权限，不能借解释功能披露上述实现细节。凭据在任何情况下都不得展示。
- 遇到真实限制或错误，说明它对用户有什么影响。不让用户选择平台内部的协议、版本或部署开关，也不把可用的后端描述成尚未上线。

## 1. 确定网站内容和实现方式

先阅读 本文下方的完整接口调用说明，取得平台预置凭据，查询当前账号的建站权益和已有站点。此步骤由 Agent 完成，不要求用户配置。

- 只展示内容：生成静态网站。
- 需要保存、修改或跨浏览器共享数据：阅读 本文下方的后端与持久 SQL 说明，生成前端和后端。
- 用户要修改已有网站：复用原站点和地址，保留已有数据及上一成功制品。没有新建席位时，说明情况并让用户决定是否更新已有站点，不擅自替换或删除其他网站。

调用时确认账号实际支持所需功能；无法使用时按真实返回说明原因，不生成假接口，也不静默改成单机版。只有需要改变用户需求时才询问替代方案。

## 2. 制作网站

从用途和内容选择布局、字体和配色，做出符合用户风格的页面；除非用户要求，不套用紫色渐变、居中单卡片或 emoji 图标的通用模板。需要图标时可用内联 SVG。

共同要求：

- 首页为发布根目录下的 `index.html`，资源使用相对路径。
- 多页面使用实际 `.html` 文件或 hash 路由（例如 `#/about`）。平台没有 History 路由的自动首页回退，不能依赖 `/about` 刷新后仍返回首页。
- 可提供根目录 `404.html`。
- 页面、公开文件和上传包中不得包含平台凭据、私钥或 `.env`。

带后端的网站：

- 前端放 `public/`，后端放 `backend/app.mjs`，后端源码不得成为公开静态资源。
- 数据写入平台提供的本站数据库；刷新、换浏览器和更新网站继续使用同一份数据。
- 页面明确区分加载中、空数据、读取失败和保存失败。失败时不要显示“保存成功”或清空现有列表。
- 更新网站时兼容已有数据；代码更新不意味着可以删除或重建数据库。
- 具体请求约定、SQL 用法、打包方式及运行限制见 本文下方的后端与持久 SQL 说明。

## 3. 发布

按 本文下方的完整接口调用说明 完成打包、创建发布、上传和状态查询。

- 新网站的同一次创建任务使用稳定幂等键；重试不重复新建站点。
- 更新使用原 `site_id`。保留上一成功发布及制品，不覆盖历史压缩包。
- 只有部署操作为 `success`、站点为 `active` 且目标版本正确时，才报告发布成功。创建站点成功或上传成功都不能替代部署完成。
- 网络超时先查询已有操作，不自动重复提交。失败保留旧站；同站点连续发布失败两次后停止自动发布，说明原因。

## 4. 检查访问入口并交付

从站点详情读取平台返回的 `PublicURL`，不自行拼接或替换域名。仅用 curl 检查：

```bash
curl --silent --show-error --location --max-time 30 \
  --output /dev/null --write-out '%{http_code} redirects=%{num_redirects}\n' \
  "$PUBLIC_URL"
```

保留 TLS 校验。最终 200 表示访问入口可达；跳转到登录或分享码页后的 200 不代表业务功能已测试。首次 404 可等 10 秒重查一次，其余错误如实报告。不要为发布检查启动浏览器、登录、截图、写入业务数据或重置分享码，也不声称已经完成这些额外验收。

交付时给出网站地址，简短说明做成了什么，以及如何访问和分享。

## 访问与分享

先按实例的 `VELA_API_BASE`、站点归属和实际访问策略区分国内站与国际站，不套用同一套分享指引；网站地址始终使用平台返回的 `PublicURL`。

**国际站公开网站**（生产建站 API 为 `https://api.lightvela.ai`）：

> 网站已发布，直接把网站链接发给朋友即可。对方无需登录或输入分享码，就能使用报名、留言等网站功能，大家看到的是同一份后端数据。

- 国际站公开访问也包含本站后端业务接口；不要额外在页面或后端要求平台登录、分享码或所有者身份才能提交表单。
- 网站管理、发布、删除及平台凭据仍只属于创建者，不交给访客。根据用户需求说明哪些业务数据会公开给参与者。

**国内站受保护网站**（生产建站 API 为 `https://api.lightvela.com`）：

尚未分享时告知：

> 网站已发布，默认只有创建者可以访问。如果需要分享给别人，请到工作台点击“分享”，把生成的分享码发给对方；对方输入有效分享码后即可访问，带后端的网站也支持共同编辑数据。

已经开启有效分享时告知：

> 你可以登录访问，其他人可以通过分享码进入；带后端的网站可以共同编辑同一份数据。

- 国内站发布不会自动开启分享。访客无需登录创建者账号；有效分享码只能访问网站和编辑应用数据，不能管理、重新发布或删除网站。
- 国内站关闭分享、分享过期或重置分享码后，旧分享访问失效。仅在用户要求分享时开启，不为检查网站擅自改变分享设置。
- 两站都不能把登录 Cookie、平台 Token 或实例凭据当分享方式。

## 需求边界

网站适合展示内容和持久数据应用。国际站支持公开、免登录的留言、报名、投票等；国内站由创建者和有效分享码访客共同使用。以下需求不能按当前平台能力直接实现：
- 网站自建注册、登录、个人中心或多用户权限系统。
- 在线支付、库存交易，或依赖任意外部网络、外部数据库、定时任务、WebSocket、服务器文件系统的服务。

遇到这些需求时，只解释相关限制，并让用户选择可行方案；不要把所有表单、清单或数据应用一概说成不支持。不能擅自用 localStorage、sessionStorage、IndexedDB 或 cookie 冒充后端数据。

用户要求查看站点、历史版本、回退或删除时，使用 本文下方的完整接口调用说明 中对应流程；删除会清除网站及持久数据，应准确说明后果。套餐到期会暂停访问、保留内容，恢复有效套餐后可恢复。


---

# 完整接口与执行说明（仅供 Agent 使用，不向用户展示）

以下说明直接保留在本文件中，执行建站时按需查阅；所有认证、请求参数、响应字段和调用顺序均不得省略或猜测。

## 认证与 API 地址

<!-- vela-website-credential-fallback:v1 -->

每个实例拥有一个由平台预置的永久凭据（`VELA_WEBSITE_KEY`）。
凭据格式为 `vik_<key_id>_<secret>`，权限范围绑定该实例及其所属用户，
只能调用建站相关接口，不暴露任何账号级权限；凭据不得复制或外泄。

**取值方式**（按顺序取第一个非空值）：

1. 当前执行环境中的 `VELA_WEBSITE_KEY`。
2. 环境变量缺失或为空时，在**同一次 terminal 命令的子 Shell** 中加载平台预置的 `/etc/vela/website-credential.env`，并 export `VELA_WEBSITE_KEY` 和 `VELA_API_BASE`。
3. 只有环境变量和文件兜底都不可用（文件缺失、不可读、加载失败或 Key 仍为空），才停下来提示用户需要通过 Vela 平台初始化实例，**不要继续调用 API**。

**环境变量为空不等于实例未初始化。必须先尝试凭据文件兜底，不要求用户重启 Agent、gateway 或实例。**

API 地址也优先使用当前环境中的非空 `VELA_API_BASE`，缺失时尝试同一凭据文件；
加载文件不得覆盖已有的非空环境变量。如果地址仍为空，按站点归属取生产默认值：
国内 `https://api.lightvela.com`、国际 `https://api.lightvela.ai`。
**只有 API 地址有默认值，凭据没有默认值。**

**加载凭据和使用凭据必须放在同一次终端执行中。** 将 API 请求或子脚本放在下面子 Shell 的结尾，
不要先单独执行加载命令，再在另一次 terminal 调用中使用凭据；每次认证命令都要重复加载步骤。
子 Shell 内 export 的变量可供同一命令启动的 Python/Node/其他子脚本读取，不会修改外层会话。
禁止输出 Key、凭据文件内容或含凭据的原始响应，禁止写入聊天、日志、项目代码或上传文件；
凭据处理及后续 API 请求期间保持 `set +x`，不要开启 shell tracing，不需要重启。

读取示例（在同一个子 Shell 中继续执行建站 API 请求或脚本）：

```bash
(
  set +x
  _vela_env_key="${VELA_WEBSITE_KEY:-}"
  _vela_env_base="${VELA_API_BASE:-}"
  if [ -z "$_vela_env_key" ] || [ -z "$_vela_env_base" ]; then
    if [ -f /etc/vela/website-credential.env ] && [ -r /etc/vela/website-credential.env ]; then
      # 文件仅由平台维护；抑制加载输出，失败时丢弃部分加载的值。
      if ! . /etc/vela/website-credential.env >/dev/null 2>&1; then
        VELA_WEBSITE_KEY=
        VELA_API_BASE=
      fi
    fi
  fi
  VELA_WEBSITE_KEY="${_vela_env_key:-${VELA_WEBSITE_KEY:-}}"
  VELA_API_BASE="${_vela_env_base:-${VELA_API_BASE:-}}"
  unset _vela_env_key _vela_env_base
  if [ -z "$VELA_WEBSITE_KEY" ]; then
    printf '%s\n' 'website credential unavailable' >&2
    exit 1
  fi
  VELA_API_BASE="${VELA_API_BASE:-https://api.lightvela.com}"
  export VELA_WEBSITE_KEY VELA_API_BASE
  KEY="$VELA_WEBSITE_KEY"
  API_BASE="$VELA_API_BASE"

  # 在这里执行建站 API 请求或子脚本；不要打印 KEY，不要重新开启 set -x。
)
```

请求头：

```
Authorization: Vela-Instance-Key <key>
Content-Type: application/json
```

### 认证请求与签名地址的执行方式

下面的请求示例用于解释协议。实际执行时通过读取环境变量的子脚本发送认证请求及签名上传，不把 Key、签名 URL 或敏感请求头放入进程命令参数。签名地址作为 JSON 字符串处理，不把 API 响应写成 shell env 文件 source，也不拼成未引用的 shell 命令。临时文件确有需要时仅保存必要字段、设为 0600，使用后清理；不输出原始响应、签名地址或凭据。

## 响应格式（所有接口统一）

所有成功响应的 HTTP 状态码为 200 或 201，响应体统一包裹为：

```json
{
  "code": "OK",
  "message": "success",
  "data": { ... },
  "request_id": "req-xxx"
}
```

**始终从 `data` 字段提取业务数据。** 错误响应的 `code` 为错误码字符串（见「错误处理」一节），`data` 可能为 null。

## 内部 API 参考（不向用户展示）

所有路径以 `/v1/websites` 为前缀，**不含** `/instance`。

| 方法 | 路径 | 用途 |
|------|------|------|
| GET | /v1/websites/capability | 查询建站权益和用量 |
| POST | /v1/websites | 创建站点 |
| GET | /v1/websites | 列出已有站点 |
| GET | /v1/websites/{siteId} | 查询站点详情 |
| POST | /v1/websites/{siteId}/releases | 创建发布 |
| GET | /v1/websites/{siteId}/releases | 列出发布历史 |
| GET | /v1/websites/{siteId}/operations/{opId} | 查询操作状态 |
| POST | /v1/websites/{siteId}/rollback | 回退到保留的成功发布 |
| DELETE | /v1/websites/{siteId} | 删除站点 |

## 接口调用流程（内部）

### 第一步：查询权益（内部）

调用能力查询接口，确认当前开放的建站能力，然后按用户需求制作并发布网站。

```
GET {API_BASE}/v1/websites/capability
```

始终从响应的 `data` 读取实际能力；遇到错误时按服务端返回的原因处理，不展示原始 JSON。

### 后端与数据能力检查（内部）

- 检查完整响应中的 `data.backend.enabled`（后端执行）和 `data.data.enabled`（服务端数据）。
- **字段缺失、null、未知值或不是布尔 true，一律按 false / 不可用处理。** 旧服务未返回这些字段时继续静态流程，不猜测能力。
- 根据本次查询结果判断接口是否兼容。仅当 `data.backend.enabled` 为布尔 true 且 `data.backend.contract` 严格等于 `fetch-v1` 时，允许执行下面的 带后端发布流程。其他版本一律不可猜测接口。`data.data.enabled` 仍须独立检查。
- 静态 release 请求声明 `"kind": "static"`；旧客户端省略 kind 仍按静态处理。`fullstack` 必须携带 `backend_sha256` 并通过能力检查；未知 kind 为参数错误。
- 能力未开放时，先说明限制，不生成假接口或用 localStorage 冒充共享数据。`data.data.enabled` 为 false 时仅支持无状态 API，访问遵循本站国内／国际策略；为 true 且 contract 为 `rivet-sql-v1` 时支持按网站隔离的持久 SQL。国际公开网站允许匿名访客调用业务 API；国内站有效分享码会话默认可以编辑，实际写入仍由网关验证分享状态、有效期和当前分享码代次，显式关闭共享编辑时会拒绝访客写入。用户要求未开放或超出范围的能力时必须说明具体限制，不能静默缩减需求。得到用户确认后才改做静态/无状态替代方案。
- 收到 `WEBSITE_BACKEND_UNAVAILABLE`：这是能力未开放，**不要重试**；不要删除 kind、改成 static 或降级上传原来的服务端代码。向用户说明后端能力当前不可用。

### 带后端网站：开发、打包与发布

1. **先确认范围匹配。** 本版为 `fetch(Request, env, ctx) -> Response`。先独立检查数据能力：只有 `data.data.enabled === true` 且 `data.data.contract === "rivet-sql-v1"` 才有下面的持久 SQL；否则仍为无状态。JS 模块内存不保证保留。国际公开网站的匿名访客可以使用同一网站的数据；国内站所有者登录或有效分享码会话默认可以编辑同一网站的数据。有效分享码不授予创建发布、重置分享码或停止分享等所有者管理权限。国内站分享过期、停止或重置后，旧分享会话不能继续写入。国际公开网站分享本站链接即可，国内站分享给别人时沿用平台分享码设置及链接；不能把所有者登录 Cookie、实例凭据或平台 Token 给别人。没有任意出网、定时任务、WebSocket、任意 Node 进程、文件系统或 Secret 绑定。需求不匹配时停止并解释，不静默降级。
2. 目录必须分离：`public/` 放静态文件，`backend/app.mjs` 放单个预打包 ES module。不要把后台源码、`.env`、私钥、平台凭据或后端 source map 放进 public。后端形态：

后端必须是单个 ESM 文件，默认导出含 `async fetch(request, env, ctx)` 方法的对象，返回标准 `Response`。
根据用户需求实现 `/api` 范围内的路由、HTTP 方法、输入校验和错误响应；未知路由返回 404，未知方法返回 405。
平台网关已按国际公开／国内受保护策略授权业务访问，并校验写请求的真实 Origin 和发布版本。传入用户后端的 Request 仅保留 `Content-Type`、`Accept` 请求头；`X-Website-Release`、Authorization、Cookie、Origin 及内部控制头不会传入。后端不要因这些头缺失再次拒绝请求，也不要根据请求体中的身份字段替代平台鉴权；只处理业务路由、方法、输入和持久数据。前端仍须按下文发送发布版本头。编辑权限如需提前探测，可使用不执行 SQL、不修改业务数据的 POST 路由，只有符合本站访问策略的请求能进入；前端同时检查成功状态和返回的权限布尔值，不能仅凭 HTTP 200 宣称可写。
不要预置固定业务示例，不依赖 Node.js 模块、实例文件系统或 JS 全局变量保存业务状态。

#### 持久 SQL 调用约定

- 使用 `await env.RIVET_STORAGE.sql.exec(sql, bindings)`，返回 `{ rows, columnNames, rowsRead, rowsWritten }`。`rows` 是以列名为键的对象数组，按 `row.title`、`row.id` 或 `row[columnName]` 读取；不要把每行当数组按 `row[0]` 读取，也不要使用列名配合数字索引重新映射。`RETURNING id` 和 `SELECT ... AS m` 同样按 `.id` 和 `.m` 读取。使用参数绑定，不能拼接用户输入到 SQL。`await env.RIVET_STORAGE.sql.databaseSize()` 返回数据库字节数。
- 数据库绑定网站身份，而不是 release、浏览器或 JS 全局变量；同一网站的重新部署、回滚和运行时实例替换应保留数据。网站删除会终止其持久对象，不能重新创建旧身份。
- 当前上限：单站数据库 256 MiB，SQL 64 KiB，256 个绑定且共 1 MiB，结果最多 1000 行且共 1 MiB。超额显示保存失败，不能用 localStorage 作兜底。
- 国际公开访问和国内分享会话的 GET/HEAD 仅允许一个 SELECT 读取；经网关批准的 POST/PUT/PATCH/DELETE 编辑请求可以执行持久写入。不要在 GET 内初始化数据库或写业务数据；在已授权的写请求中初始化 schema。仅首次建站且业务表尚未创建时，GET 可返回空列表；存储超时、权限、配额或其他 SQL 错误必须明确报错，不得吞掉错误伪装成空数据。前端应保留读取失败状态，不能因后续权限探测或重绘将错误覆盖成空列表。
- 按用户实际需求设计数据实体、字段、接口和页面，不复制预设业务应用。持久化代码写在 `backend/app.mjs`，只通过平台提供的 SQL 接口访问本站数据；不要预置与需求无关的表、路由或业务字段。
- 每个创建意图生成一次稳定的业务标识或幂等键；同一次意图重复提交时不得重复写入或更换标识。校验输入并使用参数绑定，以数据库约束和原子 SQL 保证一致性。写请求失败后先确认结果，不自动重放或偷偷新建记录。
- 重新部署的 schema 变更必须向后兼容，不删除已有业务数据；读请求不能承担迁移或初始化。代码回滚不等于数据 schema 回滚，旧版本必须仍能读取升级后的数据。
- 共享数据必须通过上述服务端 SQL 实现，并保留同一网站的数据；国际公开网站不要求分享会话；国内站没有有效分享会话、分享码过期或停止分享后的访客写入由平台拒绝。建站交付只做后文的 curl 可达性检查，不额外执行浏览器或业务数据验收，也不宣称已经完成这些功能测试。

3. `public/index.html` 在应用 JS 执行之前声明版本标记，deployer 只在私有校验成功后把完整引号包裹的标记值替换为本次 release ID，保留同名 JavaScript 变量。标记必须使用普通单引号或双引号，不能只声明同名变量而不赋值：

```html
<script>globalThis.__LIGHTVELA_RELEASE_ID__="__LIGHTVELA_RELEASE_ID__";</script>
```

所有前端 API 请求必须同源并带 `X-Website-Release`；不要在浏览器里加入平台 Authorization 或实例密钥。写请求由浏览器自动提供真实 Origin，不要通过伪造 Origin 绕过验证：

按实际需求确定 API 路径、HTTP 方法和请求数据；所有请求共用以下调用约束，不预置业务路由或字段：

- 请求路径限定为本站 `/api` 或 `/api/*`，不能指向外部地址。
- 使用 `credentials: "same-origin"`，并传入 `"X-Website-Release": globalThis.__LIGHTVELA_RELEASE_ID__`。
- 有 JSON 请求体时才添加 `Content-Type: application/json`，对实际业务数据执行 `JSON.stringify`；GET/HEAD 不发送请求体。
- 401/403：先区分站点策略和实际错误。国内站认证失败时引导所有者登录或访客输入有效分享码；国际公开网站不要提示登录或分享码，也不能仅凭状态码断言“匿名写入不受支持”或“不是 bug”，应保留表单并如实报告请求失败，排查站点业务校验或平台访问异常。不要自动重放写请求。
- 409：页面版本已过期，提示重新加载；不要用旧页面重试写操作。
- 429/502/503：显示暂时不可用，不把失败提示成已保存成功。


4. 使用当前 Profile 的 Skill 随附打包器。打包器不上传、不运行后端；制品策略由服务端校验。输出仅包含可用于 CreateRelease 的非敏感元数据：

```bash
python3 "${HERMES_HOME:-$HOME/.hermes}/skills/website-builder/scripts/package_fullstack.py" \
  --static-dir ./public --backend ./backend/app.mjs \
  --output ./artifacts/site-v1.zip
```

脚本缺失时停止并说明 Skill 交付不完整，不自行改成静态 zip。产物布局为根目录静态文件与私有 `__backend/app.mjs`；deployer 不会把后者写入静态 COS 路径。不要覆盖历史 zip，保留上一成功 release ID 与制品以供回退。

5. 创建 release 时使用打包器原样输出的 `kind=fullstack`、`backend_sha256`、`artifact_hash`、`artifact_size`、`file_count`。沿用后文签名上传流程，`data.upload_headers` 逐项原样提交；不得输出 `upload_url`、`VELA_WEBSITE_KEY` 或包含凭据的原始响应。后端 API 位于同站点 `/api`、`/api/*`；其它路径仍为静态资源。
6. 轮询 operation 到真实 success，并核对站点和 release。后台未就绪、摘要错误、配额拒绝或发布失败时保持旧站，不能报告成功。不要把 operation 创建成功/HTTP 200 当成已经发布。
7. 按「第六步」仅使用 curl 检查站点访问入口。不调用 browser use 或其它浏览器工具，不登录、截图、操作页面，也不额外执行增删改查、分享访客、重新部署或回滚验收。
8. 回滚使用 `POST {API_BASE}/v1/websites/{siteId}/rollback`，正文 `{ "release_id": "<previous-successful-release>" }`，继续轮询返回的 operation。仅可回到仍保留的静态/后端版本；制品不在时会失败，应使用保留的旧 zip 重新发布，不能宣称回滚成功。回滚完成后按「第六步」做 curl 可达性检查，不额外启动浏览器或业务验收。停用后 API 必须拒绝访问。

**运行限制：** 后端 JS 不超过 1 MiB，请求体不超过 1 MiB，响应体不超过 4 MiB；执行有 5 秒墙钟限制。首版不转发应用 cookie、任意身份头或跨域头，不支持后端重定向。不要编写依赖这些能力的应用。

### 第四步：创建站点（内部）

创建站点记录，拿到 `site_id` 和创建操作 ID（响应中的 `data.operation_id`）。
这个操作 ID 只代表站点记录的创建过程，记为 `create_operation_id`；
**它不是部署操作 ID，不要用它判断网站是否发布成功。**

**幂等键**：创建请求可带 `idempotency_key`。同一用户 + 同一幂等键最多只会建出一个站点，
由服务端唯一索引保证。**同一次建站任务的重试必须复用同一个 key**，
否则网络超时后的重试会重复建站并重复占用席位。
建议取值：本次建站任务的稳定标识（例如会话 ID + 站点名的哈希），不要每次随机生成。
不传该字段则不做去重。

**请求：**

```
POST {API_BASE}/v1/websites
```

请求体（JSON，字段名使用 snake_case）：

```json
{
  "name": "我的个人主页",
  "idempotency_key": "session-abc-我的个人主页"
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 站点名称，最长 100 字符 |
| idempotency_key | string | 否 | 幂等键，同用户+同键只建一个站点 |

**响应（HTTP 201）：**

```json
{
  "code": "OK",
  "message": "success",
  "data": {
    "site_id": "ws-a1b2c3d4e5f6",
    "operation_id": "wo-x1y2z3w4"
  },
  "request_id": "req-xxx"
}
```

`data.site_id` 是站点的唯一标识，后续发布、查询、删除都用它。
`data.operation_id` 是站点创建操作 ID，记为 `create_operation_id`，**不要用它轮询部署状态**。

### 第五步：创建发布并上传（内部）

将静态文件打包为 zip，计算 sha256 哈希值和文件大小，创建发布并从响应中获取
`data.release_id`、`data.upload_url`、`data.upload_headers`、`data.upload_method` 和发布操作 ID
（`data.operation_id`，记为
`deploy_operation_id`），然后按 `data.upload_method` 上传 zip 到 `data.upload_url`。

`data.upload_url`、`VELA_WEBSITE_KEY` 和原始响应都属于敏感的内部操作数据，
不得输出、打印或写入日志。只从响应解析所需字段，不保留完整响应。

**后续只轮询 `deploy_operation_id`。** 创建站点和创建发布会各返回一个
`operation_id`，混用会把旧创建操作的结果误判为部署结果。

#### 5.1 打包并计算哈希

```bash
# 打包为 zip（确保 index.html 在根目录）
cd /tmp/website-build && zip -r site.zip . -x '*.DS_Store'

# 计算 sha256（64 位小写十六进制）
ARTIFACT_HASH=$(sha256sum site.zip | cut -d' ' -f1)

# 文件大小（字节数）
ARTIFACT_SIZE=$(stat -c%s site.zip)   # Linux
# ARTIFACT_SIZE=$(stat -f%z site.zip)  # macOS

# 文件数
FILE_COUNT=$(python3 -c 'import zipfile; print(sum(not entry.is_dir() for entry in zipfile.ZipFile("site.zip").infolist()))')
```

#### 5.2 创建发布

**请求：**

```
POST {API_BASE}/v1/websites/{siteId}/releases
```

请求体（JSON，字段名使用 snake_case，**artifact_hash、artifact_size、file_count 三个字段必填**）：

```json
{
  "kind": "static",
  "artifact_hash": "a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456",
  "artifact_size": 45678,
  "file_count": 3
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| kind | string | 否 | 静态填写 static；省略兼容旧静态客户端；fetch-v1 能力通过时可填 fullstack |
| backend_sha256 | string | fullstack 必填 | 私有 app.mjs 的小写 SHA-256，静态发布不传 |
| artifact_hash | string | 是 | zip 文件的 sha256 哈希值，64 位小写十六进制 |
| artifact_size | int64 | 是 | zip 文件大小（字节数） |
| file_count | int | 是 | zip 内文件总数 |

**响应（HTTP 201）：**

```json
{
  "code": "OK",
  "message": "success",
  "data": {
    "release_id": "wr-a1b2c3d4",
    "operation_id": "wo-v9w8x7y6",
    "version": 1,
    "upload_url": "https://cos-bucket.cos.ap-shanghai.myqcloud.com/sites/ws-xxx/wr-xxx.zip?sign=xxx&expires=xxx",
    "upload_method": "PUT",
    "upload_headers": {
      "Content-Length": "45678",
      "x-cos-meta-artifact-sha256": "a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456"
    }
  },
  "request_id": "req-xxx"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| release_id | string | 发布 ID |
| operation_id | string | 发布操作 ID，记为 `deploy_operation_id`，用于轮询部署状态 |
| version | int | 发布版本号，从 1 递增 |
| upload_url | string | 文件上传地址，敏感数据，不得输出 |
| upload_method | string | 上传 HTTP 方法（通常为 `PUT`） |
| upload_headers | map<string,string> | 服务端签名的上传请求头，必须原样使用 |

#### 5.3 上传 zip 文件

使用 `data.upload_method`（通常为 `PUT`）向 `data.upload_url` 发送请求：

```bash
curl -X PUT \
  -H "Content-Length: ${ARTIFACT_SIZE}" \
  -H "x-cos-meta-artifact-sha256: ${ARTIFACT_HASH}" \
  -H "Content-Type: application/zip" \
  --data-binary @site.zip \
  "${UPLOAD_URL}"
```

上传注意：
- `data.upload_headers` 是服务端签名的上传契约，必须同时包含
  `Content-Length` 和 `x-cos-meta-artifact-sha256`
- 校验 `Content-Length` 等于本地 zip 的实际字节数，校验
  `x-cos-meta-artifact-sha256` 等于本地计算出的 64 位小写 sha256
- PUT 时把 `data.upload_headers` 中每一项逐项原样作为请求头发送，再加
  `Content-Type: application/zip`；不要自行推导、改名、规范化或丢弃服务端返回的值
- `data.upload_headers` 缺失、上述任一必需头缺失或出现未知头时，按协议异常立即停止发布；
  不要尝试上传，也不要重新创建站点或发布

**上传成功后 COS 返回 HTTP 200，响应体为空。** 非 200 状态码视为上传失败。

#### 5.4 轮询部署状态

使用第五步拿到的 `deploy_operation_id`（不是创建站点的 `create_operation_id`）轮询：

**请求：**

```
GET {API_BASE}/v1/websites/{siteId}/operations/{deploy_operation_id}
```

无请求体。

**响应 `data` 字段：**

```json
{
  "operation_id": "wo-v9w8x7y6",
  "status": "success",
  "error_message": ""
}
```

| status 值 | 含义 | 动作 |
|-----------|------|------|
| created | 操作已创建，等待执行 | 继续轮询 |
| running | 正在执行 | 继续轮询 |
| success | 执行成功 | 部署完成，告知用户站点地址 |
| failed | 执行失败 | 停止，查看 error_message 并告知用户 |
| dead | 超过最大重试次数，终止 | 停止，告知用户部署失败 |

**轮询间隔：每 3 秒一次。** 等待较久时简短告知用户仍在处理中，继续查询同一个 operation，直到终态；等待本身不算失败，不能据此新建站点或重复提交发布。

#### 5.5 上传时机

- **创建发布后要尽快上传**：拿到 `upload_url` 后立即 PUT；失败时读取服务端返回，不假定发布成功。
- 上传完成后不需要额外通知服务端，deployer 会自动检测文件并继续部署。

### 第六步：验证站点可访问（内部）

部署成功后，查询站点详情获取访问地址：

```
GET {API_BASE}/v1/websites/{siteId}
```

从响应 `data.PublicURL` 字段（PascalCase）取出站点地址。
**创建站点的响应不包含此字段**——创建只返回 `site_id` 和 `operation_id`。

发布后的检查到此为止：仅用 curl 访问平台返回的 `PublicURL`，确认最终 HTTP 状态码为 200，不额外使用 browser use 或其它浏览器工具，不登录、截图或做业务功能验收。
保留 TLS 证书校验，不使用 `-k`，不猜测或替换站点域名：

```bash
curl --silent --show-error --location --max-time 30 \
  --output /dev/null --write-out '%{http_code} redirects=%{num_redirects}\n' \
  "$PUBLIC_URL"
```

- 最终返回 200：访问入口可达。若经过登录或分享码页面的跳转，只说明访问入口正常，并按实际访问策略告知用户如何访问，不把授权页面的 200 当成网站业务功能已验证。
- DNS、证书或请求错误，以及非 200 结果：如实说明当前无法确认可达；不要绕过 TLS、换域名、启动浏览器或报告验证成功。
- 首次 404 可能是 DNS / CDN 刷新延迟，可等待 10 秒后重试一次；仍失败时保留实际失败结论。
- 部署仍须先确认 operation 为 success、站点为 active、目标 release 正确；curl 的 200 不能替代部署状态查询。

### 部署完成后向用户输出

用自然语言告知平台返回的站点地址，并按「访问与分享」分别说明国际站直接分享链接、国内站创建者登录及工作台“分享”入口，不要输出技术细节：

> ✅ 网站已上线！访问地址：https://ws-xxx.lightvela.site/

## 列出已有站点（内部）

**请求：**

```
GET {API_BASE}/v1/websites?page=1&page_size=20
```

**响应 `data` 字段：**

```json
{
  "sites": [
    {
      "SiteID": "ws-a1b2c3d4",
      "Name": "我的个人主页",
      "Status": "active",
      "PublicURL": "https://ws-xxx.lightvela.site/",
      "CloudPath": "sites/ws-xxx/",
      "StorageBytes": 45678
    }
  ],
  "total": 1,
  "page": 1,
  "page_size": 20
}
```

**注意：`sites` 数组中的站点对象字段名为 PascalCase**（Go 默认序列化），与请求体和显式结果结构体的 snake_case 不同。主要字段：`SiteID`、`Name`、`Status`、`PublicURL`、`CloudPath`、`StorageBytes`。

## 查询站点详情（内部）

```
GET {API_BASE}/v1/websites/{siteId}
```

**响应 `data` 字段**（PascalCase，同上）：

```json
{
  "SiteID": "ws-a1b2c3d4",
  "Name": "我的个人主页",
  "Status": "active",
  "PublicURL": "https://ws-xxx.lightvela.site/",
  "CloudPath": "sites/ws-xxx/",
  "StorageBytes": 45678
}
```

站点 `Status` 可能的值：`active`（正常）、`suspended`（暂停，套餐过期）、`deleting`（删除中）。

## 列出发布历史（内部）

```
GET {API_BASE}/v1/websites/{siteId}/releases?page=1&page_size=10
```

**响应 `data` 字段：**

```json
{
  "releases": [
    {
      "ReleaseID": "wr-a1b2c3d4",
      "SiteID": "ws-xxx",
      "Version": 1,
      "Status": "active",
      "ArtifactHash": "a1b2...",
      "ArtifactSize": 45678,
      "FileCount": 3
    }
  ],
  "total": 1,
  "page": 1,
  "page_size": 10
}
```

**注意：`releases` 数组中的字段名同样是 PascalCase。**

## 回退网站版本（内部）

用户要求回退时，先通过发布历史确认目标 release 是成功且仍保留的版本，并记录当前成功 release 与制品。

**请求：**

```
POST {API_BASE}/v1/websites/{siteId}/rollback
```

认证方式与其他建站接口一致，正文为 JSON：

```json
{
  "release_id": "<previous-successful-release>"
}
```

`release_id` 必填，使用本站仍保留的成功发布 ID，不使用版本数字、站点创建操作 ID 或其他站点的 release。

从响应的 `data.operation_id` 获取本次回退操作，按发布同样的 `created` / `running` / `success` / `failed` / `dead` 契约轮询。查询超时继续查同一操作，不重复提交回退。success 后查询本站详情与发布历史，确认实际激活目标版本，再按第六步仅做 curl 可达性检查。

制品已不保留时回退会失败；需要重新使用保存的旧 ZIP 发布，不能把回退请求已接受说成已恢复。代码回退不删除业务数据、不回退数据库 schema；保留旧代码读取升级后数据的兼容性。回退后不额外启动浏览器或业务验收。

## 删除网站流程

用户要求删除网站时，调用删除接口，然后确认删除成功。

**请求：**

```
DELETE {API_BASE}/v1/websites/{siteId}
```

无请求体。

**响应（HTTP 200）：**

```json
{
  "code": "OK",
  "message": "success",
  "data": {
    "operation_id": "wo-d1e2f3g4"
  },
  "request_id": "req-xxx"
}
```

`data.operation_id` 是删除操作 ID。按上面的同一套状态契约轮询：
`created` / `running` 继续，`success` 完成，`failed` / `dead` 停止并报告失败；
超时只表示删除仍在处理中，不要再次调用删除接口，也不要声称已经删除。

删除会清除站点的全部静态文件，不可恢复。页面内容重要时提醒用户先自行留一份。

**输出示例：**

> 🗑️ 网站已删除，相关资源已清理。

## 错误处理（翻译为用户语言）

错误响应的 HTTP 状态码为 4xx 或 5xx，响应体中的 `code` 字段为错误码字符串：

```json
{
  "code": "WEBSITE_QUOTA_EXCEEDED",
  "message": "quota exceeded",
  "data": null,
  "request_id": "req-xxx"
}
```

| 内部错误码 | 对用户说的话 |
|------------|-------------|
| WEBSITE_QUOTA_EXCEEDED | 根据服务端返回的具体原因说明当前无法发布 |
| WEBSITE_ARTIFACT_TOO_LARGE | 根据服务端返回的 message 和 details 说明原因及可行的调整 |
| WEBSITE_FILE_COUNT_EXCEEDED | 根据服务端返回的 message 和 details 说明原因及可行的调整 |
| WEBSITE_PASS_INACTIVE | 你的套餐已过期，续费后可以继续使用 |
| WEBSITE_NO_QUOTA | 你当前的套餐不包含建站功能，升级套餐即可使用 |
| WEBSITE_NOT_ACTIVE | 这个网站当前不可用，可能已暂停或正在删除 |
| WEBSITE_NOT_FOUND | 没有找到这个网站，可能已经被删除了 |
| WEBSITE_BACKEND_UNAVAILABLE | 当前还不能发布带后端的网站，可以先做纯静态版本 |
| WEBSITE_INVALID_PARAM | 网站配置有点问题，我调整一下重试 |

**WEBSITE_INVALID_PARAM 特别说明**：服务端会在响应顶层 `details.reason` 附上具体原因，
例如 `"name is required"`、`"artifact_hash is required"` 等。遇到此错误时检查请求体
是否缺少必填字段或字段名拼写错误（字段名必须是 snake_case），修正后重试。

### 重试上限（必须遵守）

- **任何情况下都不要为了重试而新建站点。** 重新发布用现有 site_id，保留原站点地址。
- **首次访问 404 不算失败**，先按上文等 10 秒再自检，不要触发重试。
- 同一个站点**发布失败 2 次后停止重试**，转为告诉用户具体原因并给出可执行的建议
  不要继续自动重发。不变的失败制品不要盲重试，也不要擅自删减用户内容。
- 同一个原因连续出现 3 次，停止逐个打补丁，直接向用户说明根因。

## 注意事项

- 用户不需要也不应该接触任何云厂商密钥、环境标识或底层资源标识
- 站点访问地址由平台发布后返回，且**在站点生命周期内固定不变**；只使用接口返回值，不要自行拼接
- 站点 ID 为随机生成，不可被枚举
- **套餐到期后站点会真正下线**（访问地址不可用），网站内容被保留；续费后平台自动恢复，地址不变。
  用户问"过期了网站还在吗"时可以回答"内容会保留，续费后自动恢复访问"
- 套餐到期或销毁后站点会下线，但网站内容会保留；重新获得有效套餐后可以恢复访问
- 建站服务是否接受本次发布，以服务端实际返回为准。异步发布失败时读取 operation 的 `error_message`，说明原因并保留旧站。
- 所有技术过程（API 调用、文件上传、状态轮询）对用户透明，AI 用"正在部署…"等自然语言描述进度即可
- 部署过程中不要输出代码块、脚本内容或工具日志给用户
- **字段名大小写**：请求体（JSON）使用 snake_case（如 `artifact_hash`、`artifact_size`、`file_count`、`idempotency_key`）；响应中显式结果结构体也使用 snake_case（如 `site_id`、`operation_id`、`upload_url`）；但站点和发布实体对象使用 PascalCase（如 `SiteID`、`Name`、`Status`、`PublicURL`、`ReleaseID`、`Version`）。调用时注意区分，不要混用。
