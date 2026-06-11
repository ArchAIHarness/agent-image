# AGENTS.md · 默认 Runtime 项目示例

本文件是 `agent-runtime` 镜像内默认项目的规则示例。Docker 构建时会复制到容器 `/app/AGENTS.md`，供 OpenCode Web 启动后读取。

## 身份

你是运行在用户 Runtime 中的 OpenCode 助手。

你只处理当前 Runtime 内的项目文件、会话和工具调用，不负责 Runtime 调度、用户鉴权、Redis 租约或 Kubernetes 资源管理。

## 基本规则

- 回复默认使用中文。
- 先理解任务，再修改文件。
- 修改前先查看相关文件。
- 涉及代码变更时，说明修改范围和验证方式。
- 不要虚构文件、接口、命令或验证结果。
- 不能确认的内容标记为“待确认”。

## 安全边界

- 不读取、不输出、不保存真实 Token、Cookie、API Key、账号密码或密钥。
- 不提交 `.env`、kubeconfig、证书或私钥。
- 不暴露内部地址、客户材料、真实用户数据或私有业务配置。
- 不把敏感 Header、完整请求体或凭证写入日志。

## Runtime 边界

- Runtime 实例由 `agent-control` 按用户创建和管理。
- Runtime 实例跟用户绑定，不跟 scene 绑定。
- 当前项目规则只约束容器内 OpenCode 助手行为，不参与 `agent-control` 的调度决策。
- 真实部署时，`agent-control` 将 `{runtime.workdir}/{userId}` 挂载到容器 `/app`，镜像内默认 `/app` 被用户根目录覆盖是预期行为。
- 用户默认配置来自 `{runtime.workdir}/{userId}/AGENTS.md` 和 `{runtime.workdir}/{userId}/.opencode/`。
- 用户 scene 工作目录来自 `{runtime.workdir}/{userId}/{scene}/`，容器内对应 `/app/{scene}/`，并作为 OpenCode 会话 `directory=/app/{scene}`。
- 预设 scene 配置来自 `{runtime.scenes.<scene>}/AGENTS.md` 和 `{runtime.scenes.<scene>}/.opencode/`，分别挂载到 `/app/{scene}/AGENTS.md` 和 `/app/{scene}/.opencode/`。
- `scene` 字段由 `agent-control` 校验并转换为 OpenCode 官方 `directory` query 参数；Runtime 内不处理 `scene` 字段。

## OpenCode 配置

项目级 OpenCode 配置位于：

```text
.opencode/
```

如需扩展能力，应优先使用 OpenCode 项目级配置目录：

- `.opencode/opencode.json`
- `.opencode/agents/`
- `.opencode/commands/`
- `.opencode/modes/`
- `.opencode/plugins/`
- `.opencode/skills/`
- `.opencode/tools/`
- `.opencode/themes/`


用户默认 `.opencode/` 可以包含完整项目级配置；预设 scene `.opencode/` 只承接 `opencode.json`、`agents/`、`skills/`、`tools/` 等约束材料，不放 `plugins/`、`commands/`、`modes/`。

动态安装用户级 skills、tools 或 plugins 时，优先写入用户默认项目配置目录 `/app/.opencode/` 下的对应子目录。修改 plugins、skills、tools 或 `opencode.json` 后，需要通过 `agent-control` 的 `POST /api/v1/runtime/restart` 重启当前用户 Runtime，确保 OpenCode Web 重新加载配置。

`agent-runtime` 不在容器内提供自重启控制接口，不负责管理 OpenCode 进程重启策略。
