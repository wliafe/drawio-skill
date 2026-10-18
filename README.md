# drawio-skill

操作 **Next AI Draw.io MCP Server** 的通用 Agent Skills 技能，包含服务检查、授权安装、会话与对象操作、图片容量控制及可靠保存导出。

技能执行当前任务已经确定的绘图内容，不提供构图、配色、字体或研究表达方法。适用于调用该 MCP、插入图片、保存手动修改和同步导出等请求。

## 使用条件

- AI 客户端能够加载技能说明或允许手动提供这些说明，并支持调用目标 MCP；有终端或配置访问能力时，可以协助完成安装。
- 运行 MCP 的环境具备合适版本的 Node.js 与 npm/npx；首次获取包需要访问 npm。预览及 PNG、SVG、可编辑 SVG 渲染需要可用的浏览器预览页。
- 技能针对 npm 包 `@next-ai-drawio/mcp-server`。不要求所有客户端安装 Codex CLI，也不依赖固定的 MCP 服务别名或客户端工具前缀。

## 加载技能

将包含 `SKILL.md` 和 `references/` 的完整 `drawio-skill` 目录，放到当前客户端实际支持的技能位置或通过其技能管理方式导入，再按该客户端要求重新加载。

技能遵循 [Agent Skills 规范](https://agentskills.io/specification)。不同客户端的技能发现方式和调用语法不同，不保证所有支持 MCP 的客户端也会自动发现技能。不支持自动加载时，可手动把 `SKILL.md` 及按需引用的说明提供给 AI。

**技能加载与 MCP 安装是两件事。** AI 先检查现有工具与配置；缺少前置软件时提供安装方法，条件齐备且已有明确安装授权时，再为当前已确认的客户端添加 MCP。仅触发技能或要求画图，不自动授权安装；已有授权不重复询问。

具体客户端示例见 [MCP 检查与安装](references/mcp-setup.md)，包括 Codex、Claude Code、Cursor 和 VS Code。只配置当前需要的客户端，不向所有已安装的客户端批量写入配置。

## 使用示例

- “按已给出的节点、关系与布局说明，通过 Draw.io MCP 创建图。”
- “检查 Next AI Draw.io MCP；如果缺少依赖，告诉我安装方法，条件满足时帮当前客户端添加 MCP。”
- “修改这张漏标示意图的指定标签，保存到当前版本并同步四种格式。”
- “插入这张图片前检查容量；过大就按素材类型压缩副本，再通过 MCP 插入和试导出。”
- “保存我在浏览器里刚做的修改，图片也导出。”
- “基于当前图创建 v2，保留 v1。”

若客户端支持按技能名称显式调用，可使用该客户端提供的方式选择 `drawio-skill`；核心说明不依赖某一种调用语法。

## 文件约定

```text
项目文档/
└── 漏标示意图/
    └── v1/
        ├── 漏标示意图.drawio
        ├── 漏标示意图.png
        ├── 漏标示意图.svg
        └── 漏标示意图.drawio.svg
```

主题可以使用中文或英文；目录名和文件主名一致，版本放在 `vN` 目录里。已有图默认原地更新，只有用户明确要求时才升版。现有文件不会为了符合默认约定而被自动改名。

默认交付 `.drawio`、`.png`、`.svg`、`.drawio.svg` 四个同名文件；用户明确指定部分格式时按其要求。可编辑 SVG 包含图源，不能用普通 SVG 改名代替。图的创建、修改和所有导出只通过 MCP，不采用其他工具补导出。

保存前后核对当前内容与同步警告，避免将过时状态或不同时间的导出误报为同步完成。详细规则见 [文件、版本与可靠保存](references/drawing-conventions.md)。

## 图片与容量

插入前检查图片的大小、尺寸、透明通道和预计嵌入开销；照片可适度缩放或有损压缩，文字、细线及透明素材优先保真。处理副本、不覆盖原素材，缺少处理工具时不自动安装。

预算覆盖完整图、Base64、请求与导出回传，不能仅看原始图片或工具折叠后的 XML 长度。插入大素材后及早通过 MCP 试导出，最终验证全部请求格式；当前版本的已知限制不能成为所有版本必然可导出的保证。

可接受清晰度下仍无法导出时，保留成果并说明瓶颈，由用户决定后续处理，不自动拆页、删素材或切换导出器。详见 [图片预处理与容量控制](references/images-and-limits.md)。

## 文件结构

```text
drawio-skill/
├── SKILL.md
├── references/
│   ├── mcp-setup.md
│   ├── drawing-conventions.md
│   └── images-and-limits.md
├── README.md
└── LICENSE
```

## 官方资料

- [Next AI Draw.io 项目](https://github.com/DayuanJiang/next-ai-draw-io)
- [MCP 源码与安装说明](https://github.com/DayuanJiang/next-ai-draw-io/tree/main/packages/mcp-server)
- [npm 包](https://www.npmjs.com/package/@next-ai-drawio/mcp-server)
- [Agent Skills 规范](https://agentskills.io/specification)

## 许可证

本技能采用 [MIT License](LICENSE)。
