# chatu-builder-skill

ChatU App Builder 构建应用时使用的 Agent Skills（Claude Code SKILL 格式）。**本仓库是 builder SKILL 的单一事实源**：builder 沙箱只加载这里的 SKILL，不加载 chatuse 镜像里的默认通用 SKILL。

## 结构

每个 SKILL 一个目录，内含 `SKILL.md`（frontmatter：`name` + `description`），可附带资产文件（如 recipes）。

### 平台能力类（与 @chatu-ai/app-sdk 模块一一对应）

| SKILL | 说明 |
| --- | --- |
| `chatu-kv` | 平台 KV 键值存储与接口限流（app-sdk 的 kv、ratelimit） |
| `chatu-db` | 平台托管文档集合（db）；用户明确要求时可切本地 SQLite（不推荐，先讲缺点） |
| `chatu-storage` | 平台对象存储（storage） |
| `chatu-auth` | 应用的登录用户体系（auth）：应用自建用户 / 渠道账号两种模式；微信扫码 / 公众号 H5 / GitHub / Gitee / QQ 三方登录与 ```chatu-env 环境变量引导；附 `references/login-app.md`、`login-channel.md`、`login-oauth.md` |
| `chatu-ai` | 平台 LLM 中继（ai）：对话/结构化输出/图片理解/工具调用/向量检索/OCR，禁止直连第三方模型；附 `references/rag.md`（知识库检索完整流程） |
| `chatu-validation` | 输入校验规范（zod） |

### 流程类

| SKILL | 说明 |
| --- | --- |
| `chatu-quickstart` | 创建应用标准流程（快速启动为主）；附 `recipes/` 预置代码：list-detail（增删改查）、form-collect（表单收集）、landing（落地页），复制即可编译预览 |
| `chatu-ui` | 界面与样式规范（shadcn@base-ui + Tailwind v4，三态、响应式、中文排版） |
| `chatu-debug` | 排错指南（排查路径、常见错误→修法对照、回退策略） |
| `chatu-verify` | 运行时自检（请求页面确认没有白屏/500，交付前必走） |
| `chatu-deploy` | 发布与部署（EdgeOne / 函数 / Git / 导出，部署前自检与失败对照） |

## 约定

1. **不得包含敏感信息**：密钥、内部域名、真实账号、员工信息一律不进仓库；示例里的 key 用占位符。
2. **SDK 更新必须联动更新 SKILL**：`@chatu-ai/app-sdk` 的 API 有任何变化（新增/改名/参数变更/行为变化），对应能力 SKILL 必须在**同一批改动**里更新并提交到本仓库——SKILL 是 agent 的 API 文档，落后一个版本就会让 agent 写出跑不通的代码。反向同理：SKILL 里描述的 API 必须真实存在于当前 SDK 版本。
3. **改动及时提交**：SKILL 内容有更新（含在 chatu-builder-sdk / chatuse 联动修改时）必须同步提交推送到本仓库，保持单一事实源。同时保留一份拷贝在 `chatu-builder-sdk/packages/app-sdk/skills/`（沙箱的最后一级回退源，随 npm 包发布）。
4. **recipes 与模板对齐**：`chatu-quickstart/recipes/` 只允许使用 next-shadcn 模板已装依赖（shadcn@base-ui 组件、zod、lucide-react、sonner），保证复制后零安装即编译；模板依赖或组件 API 变更时（如 base-ui 的 `render` 属性替代 `asChild`），recipes 必须回归测试（拷入模板跑 `tsc --noEmit`）。
5. SKILL 面向"生成应用的 agent"编写：告诉它平台能力怎么用、什么禁止装（第三方 auth/db/ai 库），而不是面向人类的教程。

## 发布（推 `main` 即生效，不用重建 chatuse 镜像）

推送到 `main` 会触发 `.github/workflows/publish.yml`：把所有含 `SKILL.md` 的目录打成 zip，POST 到 ChatU 服务端的 `builder/skills/publish`，服务端解到共享存储（CFS）；Builder 沙箱 Pod 把该目录只读挂载为 `/app/builder-skills`，runtime 在 **Pod 启动 / 快照恢复** 时优先从这里同步到 workspace 的 `.claude/skills/`。

- 生效范围：发布后**新建、唤醒、重启**的沙箱；正在运行的 Pod 不变。
- 回退顺序（runtime）：CFS 发布目录 → chatuse 镜像烘焙副本（构建时从本仓库 `main` 取）→ `node_modules/@chatu-ai/app-sdk/skills`。
- 仓库 Secrets（在 GitHub 仓库设置里配置，**不要写进任何文件**）：`CHATU_SKILL_PUBLISH_URL`（服务端地址 + `builder/skills/publish`）、`CHATU_SKILL_PUBLISH_TOKEN`（与服务端配置一致的口令）。
- 手动触发：Actions 页面 `Publish skills` → Run workflow。
- 核对：`GET builder/skills/publish`（同口令）返回当前已发布的 git ref、时间与 skill 清单；沙箱侧 `/api/runtime/meta` 的 `skillsSource` 显示实际同步来源。

