---
name: spring-boot-agents-md
description: 在 Spring Boot 项目中开发、修改、重构或评审 Java 代码时使用。提供工程结构、分层职责、Lombok、构造器注入、异常日志、持久层、接口校验与验收清单等工程约定；当目标项目缺少 AGENTS.md 时，按本 skill 的模板生成并裁剪。
---

# Spring Boot 工程规范

## 何时使用

满足任一条时使用本 skill：

- 目标项目是 Spring Boot 项目，能检测到 `pom.xml` 或 `build.gradle` 中的 `spring-boot` 依赖；
- 用户要求「按团队规范写」「统一代码风格」「给项目立规矩」「生成 AGENTS.md」；
- 用户要求评审 Spring Boot 代码是否符合工程约定。

以下情况不使用：

- 非 Java 或非 Spring Boot 项目；
- 只做文案、纯 SQL 脚本、前端样式等与后端工程结构无关的改动。

## 冲突优先级

1. 用户当前任务的明确要求。
2. 目标项目已有的规则文件：`AGENTS.md`、`CLAUDE.md`、`.cursor/rules/`、`.github/copilot-instructions.md` 等。
3. 本 skill 的规范。
4. Java / Spring 官方推荐实践。
5. 你自己的默认习惯。

目标项目已有规则且与本文冲突时，以目标项目为准，在回答中说明差异即可，不要擅自「纠正」。

除非用户明确要求生成或更新，否则不要修改目标项目的规则文件。

## 使用方式

### A. 项目没有规范文件，需要生成

1. 读 `pom.xml` 或 `build.gradle`：JDK 版本、Spring Boot 版本、持久层依赖（MyBatis / MyBatis-Plus / Spring Data JPA）、是否引入 Lombok、统一返回与参数校验依赖。
2. 看包结构：按业务领域还是按技术分层，`controller` 下是否已有领域子包，Service 是否接口与实现分离。
3. 找一个同类业务的 Controller + Service + Mapper / Repository，作为风格基准。
4. 以本 skill 根目录的 `AGENTS.md` 为模板，在目标项目根目录生成同名文件，并完成三处裁剪：
   - 把 `com.example.project` 替换为实际根包名；
   - 删除 `mapper` 与 `repository` 中不适用的一段；
   - 删除项目不存在的（按需）目录行（`aspect`、`job`、`mq`、`oss` 等）。
5. 不要写入与项目事实不符的章节。例如项目未使用 Lombok，就删掉 Lombok 小节，而不是留着让后续 Agent 误判。
6. 生成后列出「依据哪些事实、做了哪些裁剪」，让用户能逐条核对。

### B. 项目已有规则，正在写代码

按目标项目的规则执行。仅在项目规则未覆盖的部分，参考本 skill 的 `AGENTS.md` 与 `references/antipatterns.md`。

### C. 代码评审

对照 `references/acceptance.md` 的「Reviewer 验收」逐条检查。问题用 `references/antipatterns.md` 的 ❌ / ✅ 形式给出，只覆盖本次改动范围，不附带无关重构建议。

## 文件

- `AGENTS.md`：规范正文，12 节，也是生成用的模板。
- `references/antipatterns.md`：7 类常见反例与正例代码。
- `references/acceptance.md`：开发前检查、提交前自检、Reviewer 验收、规范生效验证。

## 不要做的事

- 不要把本 skill 的规则当作目标项目的规则强行覆盖。
- 不要在非 Spring Boot 项目中套用本规范。
- 不要为了「更规范」而大范围重构存量代码。
- 不要在目标项目里创建 `references/`、`docs/`、`checklists/` 这类本 skill 自身的目录。