# chatu-builder-skill

ChatU App Builder 构建应用时使用的 Agent Skills（Claude Code SKILL 格式）。**本仓库是 builder SKILL 的单一事实源**：builder 沙箱只加载这里的 SKILL，不加载 chatuse 镜像里的默认通用 SKILL。

## 结构

每个 SKILL 一个目录，内含 `SKILL.md`（frontmatter：`name` + `description`）：

| SKILL | 说明 |
| --- | --- |
| `chatu-kv` | 平台 KV 键值存储（@chatu-ai/app-sdk 的 kv） |
| `chatu-db` | 平台托管文档集合（db） |
| `chatu-storage` | 平台对象存储（storage） |
| `chatu-auth` | 应用自己的登录用户体系（auth） |
| `chatu-ai` | 平台 LLM 中继（ai），禁止直连第三方模型 |
| `chatu-validation` | 输入校验规范（zod） |

## 约定

1. **不得包含敏感信息**：密钥、内部域名、真实账号、员工信息一律不进仓库；示例里的 key 用占位符。
2. **改动及时提交**：SKILL 内容有更新（含在 chatu-builder-sdk / chatuse 联动修改时）必须同步提交到本仓库，保持单一事实源。
3. SKILL 面向"生成应用的 agent"编写：告诉它平台能力怎么用、什么禁止装（第三方 auth/db/ai 库），而不是面向人类的教程。
