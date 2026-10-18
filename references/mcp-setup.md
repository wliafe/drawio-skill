# Next AI Draw.io MCP：检查与安装

本参考只处理 [`@next-ai-drawio/mcp-server`](https://github.com/DayuanJiang/next-ai-draw-io/tree/main/packages/mcp-server)。它通过 stdio 连接 Agent，并在浏览器显示可共同编辑的 draw.io 画布。不要把它与 `@drawio/mcp`、托管的 draw.io MCP 或其他同名服务混用。

## 先确认当前客户端与现状

先检查本次会话提供的 MCP 工具及当前客户端的有效配置，再决定是否需要安装。结合启动命令、参数中的包名、插件来源和服务器说明判断身份；`drawio` 只是常见注册名，工具命名空间也可能不同。

| 检查结果 | 应采取的动作 |
| --- | --- |
| 已有本服务的工具 | 调用其只读 `get_drawing_guide`，参数为 `{}`，验证响应。 |
| 同一包已用其他名称配置 | 复用已有别名，后续检查和调用使用该名称，不重复注册。 |
| 配置存在，但当前会话没有工具 | 核对启用状态、客户端重载与工具加载情况；不能据此判定未安装。 |
| 服务已禁用、被拒绝或受管理策略限制 | 说明具体状态；不自动启用、覆盖或另起别名绕过限制。 |
| `drawio` 已指向其他包或服务 | 说明冲突并保留原项；已有新增授权时可选择未占用的别名，例如 `next-ai-drawio`，并告知实际名称。用户要求必须使用原名称时，才确认替换方式。 |
| 确认当前客户端没有本服务 | 检查前置条件，再走该客户端的注册分支。 |

区分“配置存在”“客户端尚未加载”“只读工具已响应”三种证据。只有最后一种可报告 MCP 工具可用；浏览器预览及导出是否正常还需在实际绘图任务中验证。不要用会创建或切换画布的 `start_session` 做安装健康检查。

仅为当前已确认的客户端配置。若客户端、执行主机或用户级/项目级范围无法从任务与环境确定，一次针对性询问这些缺项；不要因其他客户端已经安装，就把它当作本次目标。当前任务已有明确安装授权且条件齐备时，执行最后的注册步骤，不重复询问相同授权。仅要求画图或触发技能不构成安装或扩大范围的授权；尚无授权时，说明缺项和准备执行的注册方式，一次询问安装意愿。

## 检查前置条件

在 MCP 实际运行的主机和客户端环境中检查，不能用 Agent 自带但客户端无法找到的 Node 代替：

```sh
node --version
npm --version
npx --version
```

- Node.js 需满足当前包及其依赖的运行要求。推荐使用 [Node.js 官方下载页](https://nodejs.org/en/download)提供的受支持 LTS；官方安装器通常同时提供 npm，npx 随 npm 提供。
- 通过 npx 启动不需要先全局安装本 MCP。首次下载或更新未缓存的包需要访问所用 npm registry；`@latest` 会随上游发布变化，已有可用固定版本不因本技能自动升级。
- 需要普通浏览器打开本机预览。当前上游包内含 draw.io 编辑器，不要求额外安装 draw.io Desktop、Docker 或配置绘图模型 API 密钥。截图以及 PNG、SVG、`.drawio.svg` 导出依赖浏览器预览页渲染；不能因导出失败自动安装或切换其他导出器。
- 只有目标是 Codex 时才要求 Codex CLI；Claude Code、Cursor、VS Code 不需要安装 Codex CLI。

**版本说明（2026-10-18 核对）：** npm 发布包 `0.4.6` 自身声明 Node `>=18`，但其 `open ^11.0.0` 依赖声明 Node `>=20`。不能只凭包自身的 `>=18` 就认定 Node 18/19 满足条件。这是当日版本事实，不是永久下限；安装前读取当前发布包元数据，并按相关依赖的当前要求判断：

```sh
npm view @next-ai-drawio/mcp-server@latest version engines dependencies --json
```

元数据查询失败时区分网络、registry 与运行时错误，不把失败解释成“包不存在”。可对照[上游 package.json](https://github.com/DayuanJiang/next-ai-draw-io/blob/main/packages/mcp-server/package.json)和[已核对的 open 11.0.0 要求](https://github.com/sindresorhus/open/blob/v11.0.0/package.json)，但仓库分支不一定与 npm 已发布版本相同。

缺少前置软件时，说明缺项并提供官方安装方法，不代为安装 Node、npm、npx、客户端或包管理器。Node/npm/npx 缺失优先引导使用 Node 官方 LTS 安装器，安装后重新打开终端与客户端再检查。目标为 Codex 且 CLI 缺失时，可提供[官方 CLI 安装文档](https://learn.chatgpt.com/docs/codex/cli)中的 `npm install -g @openai/codex@latest` 方法；目标为其他客户端时，按该客户端官方安装文档处理。

## 按当前客户端注册

以下示例只用于本服务缺失、名称无冲突、前置条件满足且当前任务已授权安装的情况。使用其他别名时，将示例中的服务名 `drawio` 一致替换；包名保持不变。先读取已有配置，只增加本服务条目，保留其他服务、环境变量、输入项和审批设置；不要把整个配置文件替换成示例。权限不足或企业策略阻止时，报告实际原因，不更改策略绕过限制。

### Codex

先确认本机命令支持情况，再检查有效配置：

```sh
codex --version
codex mcp --help
codex mcp list --json
codex mcp get drawio --json
```

`get` 找不到 `drawio` 时，仍需检查 `list` 中是否存在同一包的其他别名；命令不可用或配置读取失败不能当作服务缺失。已有本服务配置时走重载或故障诊断，禁用与同名冲突按前述规则处理。

确认用户级注册符合已有授权后执行：

```sh
codex mcp add drawio -- npx -y @next-ai-drawio/mcp-server@latest
codex mcp get drawio --json
codex mcp list --json
```

此命令使用 Codex 当前配置根；默认用户配置位于 `~/.codex/config.toml`。不要编造 `codex mcp add --scope` 选项。若只授权项目级配置，按[Codex 官方 MCP 文档](https://learn.chatgpt.com/docs/extend/mcp?surface=cli)在受信任项目的 `.codex/config.toml` 最小合并：

```toml
[mcp_servers.drawio]
command = "npx"
args = ["-y", "@next-ai-drawio/mcp-server@latest"]
```

写入后按当前 Codex 客户端提示重启或重新加载；可在 `/mcp` 或 MCP 设置中查看连接状态，再实际调用 `get_drawing_guide`。添加命令成功只证明注册步骤完成。

### Claude Code

先用 `claude mcp list`、`claude mcp get drawio` 查看当前项目的有效配置、来源与连接状态；同样检查别名。注册时使用已有授权的范围，只执行以下匹配的一条：

```sh
# 当前项目个人使用，不写项目共享配置：本地范围
claude mcp add --transport stdio --scope local drawio -- npx -y @next-ai-drawio/mcp-server@latest

# 仅当已授权所有项目可用时：用户级
claude mcp add --transport stdio --scope user drawio -- npx -y @next-ai-drawio/mcp-server@latest

# 仅当已授权写入当前项目共享配置时：项目级
claude mcp add --transport stdio --scope project drawio -- npx -y @next-ai-drawio/mcp-server@latest
```

`local` 是当前项目的个人配置，不写项目共享文件；`user` 跨项目可用；`project` 写入项目根目录 `.mcp.json`，可能随仓库共享。用户明确要求“当前项目个人使用”时直接选 `local`，不扩大到 `user` 或 `project`，也不重复确认已经明确的范围。添加后读取 `get/list`，按客户端提示重连或重启，在 `/mcp` 中核对状态并验证只读工具响应。范围和审批行为以[Claude Code 官方 MCP 文档](https://code.claude.com/docs/en/mcp)为准。

### Cursor

检查当前有效的项目 `.cursor/mcp.json`、用户 `~/.cursor/mcp.json` 及客户端 MCP 设置。仅在已授权的范围最小合并；[Cursor 官方文档](https://cursor.com/docs/mcp)使用顶层 `mcpServers`：

```json
{
  "mcpServers": {
    "drawio": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@next-ai-drawio/mcp-server@latest"]
    }
  }
}
```

### VS Code

先通过 MCP 设置或 `MCP: List Servers` 确定当前配置来源、范围与启用状态。VS Code 专用 `.vscode/mcp.json`（或通过 `MCP: Open User Configuration` 打开的用户配置）使用顶层 `servers`：

```json
{
  "servers": {
    "drawio": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@next-ai-drawio/mcp-server@latest"]
    }
  }
}
```

截至 2026-10-18，[VS Code 官方文档](https://code.visualstudio.com/docs/agent-customization/mcp-servers)还支持并推荐部分场景使用可移植的项目根 `.mcp.json`，其顶层是 `mcpServers`，可复用上面的 Cursor JSON 结构。先按当前版本和既有配置来源选定已授权的文件，不为同一服务同时写入多种位置，也不要把两种顶层键混用。

Cursor 与 VS Code 保存后，按当前客户端提供的重载/重启服务方式使配置生效，并检查工具列表与只读响应。未知客户端参照[上游安装说明](https://github.com/DayuanJiang/next-ai-draw-io/tree/main/packages/mcp-server)和该客户端官方文档，只注册当前目标客户端。

## 已有连接异常时诊断

- 工具缺失：检查配置是否生效、服务是否被禁用、工具筛选、客户端是否需要重载，以及当前执行主机是否正确。保留用户的禁用与审批决定。
- 启动失败：查看该服务日志，核对客户端可见的 Node/npx 路径、运行版本、首次下载网络与 npm registry。不要用删除再添加代替诊断。
- 只读响应失败：保留实际报错，区分进程启动、协议连接和调用错误；注册成功不能消除这些问题。
- 只读调用成功但预览异常：在已授权绘图时使用 `start_session` 返回的完整 URL；使用普通浏览器，截图或导出超时时恢复预览并置前后重试一次。默认本机预览端口为 6002，占用时上游会尝试后续端口。部分能力也受预览模式限制，例如 `get_selection` 不可用时，应按页、标签和位置定位，不当作服务安装失败。
- 提示体积过大、413 或插入大素材后导出超时：按 [图片预处理与容量控制](images-and-limits.md)区分请求、完整图同步和导出回传问题；先修正容量再有限重试，不盲目重装。

不修改 `default_tools_approval_mode`、自动批准列表或其他审批策略。没有终端、配置编辑或 MCP 调用能力时，给出适合当前客户端的可复制命令/配置与重载步骤，并明确哪些步骤尚未执行、哪些状态尚未验证；不要声称已经安装或可用。
