# Spring Boot AGENTS.md 规范

> 一份可直接复制、按项目裁剪的 Spring Boot 工程规范；也可以作为 skill 装进 AI Agent，让 Codex、Qoder、TRAE、Claude Code、Cursor 在同一个项目里写出一致的代码。

AI Coding Agent 写 Java 时，常见的问题不是「不会写」，而是「像个第一天进项目的人」：

1. 项目已用 Lombok，它仍然手写构造方法、Getter、Setter。
2. 项目采用 Controller / Service / Mapper 分层，它把业务逻辑写进 Controller。
3. 简单的业务逻辑被过度抽象，平白多出接口、工厂类和工具类。
4. 项目有现成的工具类、公共组件，它还是重新实现一套。
5. 动不动引入 Handler、Manager、Factory。
6. 换个 Agent，代码风格又变了。

这份规范就是把这些「团队默认知道、但从没写下来」的约定落成文件，让 Agent 每次动手前先读一遍。

## 用法一：直接复制（30 秒）

1. 把 [AGENTS.md](AGENTS.md) 复制到你的 Spring Boot 项目根目录。
2. 按文件顶部注释改三处：根包名、持久层选型（删除 `mapper` 或 `repository` 中不适用的一段）、删掉项目不存在的（按需）目录行。
3. 打开 [references/acceptance.md](references/acceptance.md)，拿一个真实小需求（比如新增一个查询接口）试一次，对照清单看 Agent 有没有跑偏。

规范只有一份，不在项目里复制粘贴第二个版本。

## 用法二：安装为 skill

支持 skill 的 Agent（Qoder、Claude Code 等）可以直接安装。把下面这段复制给它：

```text
请从这个地址安装 spring-boot-agents-md skill，并阅读 SKILL.md：
https://github.com/Naylor55/spring-boot-agents-md
```

装好后，Agent 在 Spring Boot 项目中会自动读取这套规范；目标项目已有 `AGENTS.md`、`CLAUDE.md`、`.cursor/rules/` 等规则文件时，以项目规则为准，skill 只做补充。

三点要清楚：

- **skill 是快照**。本仓库更新后不会自动同步，需要重装。
- **不覆盖项目规则**。除非你明确要求生成或更新，否则它不会改动目标项目的规则文件。
- **只在 Spring Boot 项目触发**。非 Java、非 Spring Boot 项目不应套用。
- **仓库名建议与 skill 名一致**（`spring-boot-agents-md`）。部分工具会校验 skill 目录名与 frontmatter 中的 `name` 是否相同。

## 仓库内容

| 文件 | 用途 |
|---|---|
| [AGENTS.md](AGENTS.md) | 规范正文，12 节：角色目标、开发前检查、工程结构、分层职责、编码规范、Spring 规范、异常日志、持久层、接口校验、测试、工作流、规则优先级 |
| [SKILL.md](SKILL.md) | skill 入口：触发条件、冲突优先级、三种使用方式（生成 / 编码 / 评审） |
| [references/antipatterns.md](references/antipatterns.md) | 7 类典型翻车场景，每类给出 ❌ 反例和 ✅ 正例代码 |
| [references/acceptance.md](references/acceptance.md) | 验收清单：开发前检查、Agent 自检、Reviewer 验收、规范生效验证 |
| [LICENSE](LICENSE) | 文档 CC BY 4.0，代码示例 MIT |

## 它不是

- **不是规范大全**。刻意不覆盖所有场景；覆盖不到的，宁可让你删，也不写一条 Agent 判断不了的规则。
- **不绑定某个 Agent**。`AGENTS.md` 是跨工具的通用约定；工具不直接读取时，把内容同步到该工具支持的规则文件即可（如 `CLAUDE.md`、`.cursor/rules/`、`.github/copilot-instructions.md`），不要两个文件各写一套。
- **不照搬别人的规则**。不同项目 Java 版本、Spring Boot 版本、持久层选型都不同，整份复制别人的规范，只会让 Agent 按不属于你的方式写代码。

## 参考项目

编写时参考了以下项目的思路，未复制其内容：

| 项目 | 参考方向 |
|---|---|
| [java-development-skill](https://github.com/zander-zyx/java-development-skill) | Java 开发相关实践 |
| [spring-boot-skills](https://github.com/rrezartprebreza/spring-boot-skills) | Spring Boot 开发技能 |
| [java-skills](https://github.com/full-stack-skills/java-skills) | Java 开发规则与实践 |
| [java-spring-best-practices](https://github.com/cosbort/agent-skills/tree/main/java-spring-best-practices) | Java / Spring 最佳实践 |
| [ai-skill-java-spring-boot](https://github.com/dunghoangcy-web/ai-skill-java-spring-boot) | Java Spring Boot 相关 Skill |
| [awesome-copilot](https://github.com/github/awesome-copilot) | 社区提示词与扩展资源 |

另外参考了 Spring 官方的 [Team Practices](https://github.com/spring-projects/spring-boot/wiki/Team-Practices) 与 [spring-javaformat](https://github.com/spring-io/spring-javaformat)。

## 怎么验证规范真的生效

不要问 Agent「你理解规范了吗」，直接给它一个真实小需求，检查输出：

- 代码是否落在正确的业务领域；
- 是否使用项目既有的 DTO、VO 和统一响应；
- 持久层是否按项目选型走 Mapper 或 Repository；
- Lombok 是否复用、有没有重复手写；
- 异常、日志、依赖注入是否遵循约定；
- 有没有顺手重构无关代码。

完整的判定项见 [references/acceptance.md](references/acceptance.md)。

有些规则不该指望 Agent 自觉：代码格式、编译、单测、静态检查交给 Maven 插件、Checkstyle、Spotless、spring-javaformat 或 CI 执行。AI 负责写，工具负责验。

## 维护

规范不是写完就结束的文档：

- code review 里反复出现的问题，补进规则；
- 项目升级 Java / Spring Boot 版本，同步更新约束；
- 某条规则不再适用，及时删除，而不是留着看。

发现某条规则在你的项目里不适用，欢迎提 issue，说明技术栈和反例——这通常意味着模板需要增加一个开关，而不是你说错了。

## License

- 规范文本（Markdown 文档）：[CC BY 4.0](LICENSE)
- 代码示例（文档中的 Java / SQL / 配置片段）：[MIT](LICENSE)

---

## English TL;DR

A copy-and-trim `AGENTS.md` for Spring Boot projects, so Codex, Qoder, TRAE, Claude Code and Cursor follow the same architecture, layering and coding conventions in your repo. It also ships as an installable skill (`SKILL.md`) that generates and applies the template without overwriting a project's own rules.

**Quick start:** copy [`AGENTS.md`](AGENTS.md) to your project root, replace the base package, delete either the `mapper` or `repository` section, drop unused optional directories, then verify with [`references/acceptance.md`](references/acceptance.md) on one real task.

**Install as skill:** tell your agent — `Install the spring-boot-agents-md skill from https://github.com/Naylor55/spring-boot-agents-md and read SKILL.md`.

Scope is intentionally narrow: 12 sections covering structure, layering, Java style, Spring conventions, exceptions/logging, persistence, API validation, testing, workflow and rule precedence. Docs are CC BY 4.0; code samples are MIT.