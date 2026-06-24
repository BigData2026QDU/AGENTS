# PROJECT.md

> 学期项目工程规范 - Spark 相关大数据项目

## 0. 硬性要求（CRITICAL）

### 0.1 Git Push 前置检查（强制）

**在执行任何 `git push` 操作前，必须确保：**

1. **AGENTS Submodule 为最新状态**
   ```bash
   # 检查 AGENTS submodule 状态
   cd AGENTS
   git pull origin master
   cd ..
   git add AGENTS
   git commit -m "chore: 更新 AGENTS 规范至最新版本"
   ```

2. **符合最新规范标准**
   - 仓库结构符合 PROJECT.md 要求
   - 文档完整（Architecture.md、README.md、File_Index.md）
   - 代码符合对应的 BACKEND.md 或 FRONTEND.md 规范

3. **为什么**
   - 确保所有项目遵循统一的最新规范
   - 避免使用过时的规范导致不一致
   - 保证 CI/CD 流程正常工作

#### Git Push 检查清单

```bash
# 1. 进入 AGENTS 目录，更新到最新版本
cd AGENTS
git checkout master
git pull origin master

# 2. 返回项目根目录
cd ..

# 3. 如果 AGENTS 有更新，提交更新
git add AGENTS
git commit -m "chore: 更新 AGENTS 规范至最新版本"

# 4. 检查自己的代码是否符合最新规范
# - 阅读 AGENTS/PROJECT/*.md
# - 确认符合所有要求

# 5. 执行 git push
git push
```

---

## 1. 项目概述

本项目为学期项目的工程规范文档，定义了统一的仓库结构、文档标准、开发流程和代码规范。

**目标：** 确保项目一致性、可维护性和可协作性。

---

## 2. 技术栈

### 2.1 核心技术栈

| 技术 | 版本 | 状态 | 说明 |
|------|------|------|------|
| Java | 17 (JDK 17 LTS) | 必需 | 主要开发语言，与 CI/CD 和 JavaTestSkeleton 保持一致 |
| Apache Spark | 3.5.0 | 必需 | 大数据处理框架 |
| Scala | 2.12 | 必需 | Spark 依赖坐标统一使用 `_2.12` 后缀 |
| 数据库 | H2 2.2.224（测试默认）/ 业务库按项目声明 | 按需 | 单元测试默认使用 H2；MySQL、PostgreSQL 等业务库必须在项目 README 中明确版本 |
| Python | 3.8+ | 可选 | 数据分析和脚本，需与 Spark 3.5.x 兼容 |

**版本选择说明：**

- **为什么使用 Java 17：** Java 17 是 LTS 版本，适合学期项目统一环境；Spark 3.5.x 官方支持 Java 8/11/17，且 Spark 后续主版本会以 Java 17 作为最低 Java 版本，因此统一到 Java 17 可以减少后续迁移成本。
- **为什么使用 Spark 3.5.0：** 本仓库后端规范中的 Maven 示例已使用 `spark.version` 3.5.0，统一文档版本可以避免开发、测试、CI 使用不同 Spark 版本。
- **为什么使用 Scala 2.12：** Maven 依赖示例使用 `spark-core_2.12`、`spark-sql_2.12`；Spark 应用必须使用与 Spark 构建包一致的 Scala 二进制版本。
- **数据库版本原则：** 规范仓库只固定测试默认库；业务数据库会影响部署环境，必须由具体项目在 README、配置文件和测试说明中声明。

### 2.2 技术栈确认流程

**关键原则：** 在编写任何代码前，必须确认技术栈版本与项目要求一致。

1. 查阅本文档技术栈章节
2. 确认 JDK 版本为 17
3. 确认 Spark、Scala、数据库等依赖版本号
4. 在项目 README 中明确声明使用的版本

---

## 3. 仓库结构规范

### 3.1 标准目录结构

```
repository-name/
├── repository-name/          # 源代码目录（必须与仓库同名）
│   ├── src/                 # 源代码
│   ├── test/                # 测试代码
│   └── ...                  # 其他代码文件
├── Architecture.md          # 架构文档（必需）
├── README.md                # 项目说明（必需）
├── File_Index.md            # 文件索引（必需）
├── .gitignore               # Git 忽略配置
├── .git/                    # Git 目录
└── 其他构建配置文件          # pom.xml, build.gradle 等
```

### 3.2 结构约束

**强制规则：**
1. ✅ 源代码必须放在与仓库同名的单一目录中
2. ✅ 三份核心文档必须位于仓库根目录
3. ✅ 根目录只允许：代码目录、文档、Git 文件、构建配置
4. ❌ 禁止在根目录散落源代码文件
5. ❌ 禁止多个源代码目录并存

### 3.3 Git 配置

`.gitignore` 应包含：
- IDE 配置文件（.idea/, *.iml, .vscode/）
- 构建产物（target/, build/, *.class, *.jar）
- 日志文件（*.log）
- 操作系统文件（.DS_Store, Thumbs.db）
- 依赖管理工具缓存（.gradle/, .mvn/）

---

## 4. 文档规范

### 4.1 Architecture.md（架构文档）

**用途：** 描述系统设计和技术架构

**必需内容：**
- [ ] 系统架构图（推荐使用 Mermaid 或 ASCII 图）
- [ ] 核心模块/组件列表及职责
- [ ] 模块间依赖关系和通信方式
- [ ] 数据流向和处理链路
- [ ] 关键技术选型和理由
- [ ] 性能和扩展性考虑

**格式要求：**
- 语言：中文
- 格式：Markdown
- 图表：推荐使用代码形式（Mermaid/PlantUML）便于版本控制

**示例大纲：**
```markdown
# 系统架构

## 1. 整体架构
[架构图]

## 2. 核心模块
### 2.1 数据采集模块
### 2.2 数据处理模块
### 2.3 数据存储模块

## 3. 技术选型
## 4. 数据流向
## 5. 部署架构
```

### 4.2 README.md（项目说明）

**用途：** 项目入口文档，帮助用户快速理解和使用项目

**必需内容：**
- [ ] 项目简介（一句话描述 + 详细说明）
- [ ] 核心功能和特性列表
- [ ] 环境要求（JDK、依赖工具）
- [ ] 快速开始指南（5 分钟内能运行）
- [ ] 构建命令（Maven/Gradle）
- [ ] 运行方法和配置说明
- [ ] 项目结构说明
- [ ] 贡献指南
- [ ] 许可证信息

**格式要求：**
- 语言：中文
- 格式：Markdown
- 结构清晰，使用标题层级

**示例大纲：**
```markdown
# 项目名称

## 简介
## 功能特性
## 环境要求
## 快速开始
## 构建和运行
## 项目结构
## 开发指南
## 贡献指南
## 许可证
```

### 4.3 File_Index.md（文件索引）

**用途：** 提供代码库文件清单和快速定位

**必需内容：**
- [ ] 所有源代码文件的相对路径
- [ ] 每个文件的主要职责
- [ ] 文件内容简要说明（类、接口、功能）
- [ ] 按目录结构组织

**格式要求：**
- 语言：中文
- 格式：Markdown 表格或列表
- 按目录层级组织

**示例格式：**
```markdown
# 文件索引

## 源代码目录
### src/main/java/com/example/

| 文件路径 | 作用 | 说明 |
|---------|------|------|
| Main.java | 主入口 | 程序启动类 |
| DataProcessor.java | 数据处理 | 核心业务逻辑 |

### src/test/java/com/example/
...
```

### 4.4 文档同步原则

**关键规则：** 代码即文档，文档即代码

- 代码变更时，同步更新对应文档
- 新增模块时，更新 Architecture.md 和 File_Index.md
- 修改接口时，更新 README.md 使用说明
- 定期 Review 文档与代码的一致性

---

## 5. 开发流程

### 5.1 标准开发流程

```
1. 需求分析
   ↓
2. 技术栈确认 ⚠️ 关键检查点
   ↓
3. 创建仓库结构
   - 创建代码目录
   - 创建三份文档框架
   ↓
4. 架构设计
   - 编写 Architecture.md
   ↓
5. 开发实现
   - 编写代码
   - 编写测试
   - 更新 File_Index.md
   ↓
6. 文档同步
   - 更新 README.md
   - 完善其他文档
   ↓
7. 代码审查
   - 检查代码规范
   - 检查文档完整性
   ↓
8. 提交代码
   - Git commit
   - Git push
```

### 5.2 Git 工作流

**分支策略：**
- `master/main`：主分支，保持稳定
- `develop`：开发分支
- `feature/*`：功能分支
- `hotfix/*`：紧急修复分支

**提交规范：**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type 类型：**
- `feat`: 新功能
- `fix`: 修复 Bug
- `docs`: 文档更新
- `style`: 代码格式调整
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建工具或辅助工具变动

**示例：**
```
feat(data-processor): 添加数据清洗功能

- 实现空值过滤
- 实现异常值检测
- 添加单元测试

Closes #123
```

---

## 6. 代码规范

### 6.1 Java 代码规范

**基本原则：**
- 遵循 [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- 使用中文注释
- 保持代码简洁和可读性

**命名规范：**
- 类名：`PascalCase`（如 `DataProcessor`）
- 方法名：`camelCase`（如 `processData()`）
- 常量：`UPPER_SNAKE_CASE`（如 `MAX_RETRY_COUNT`）
- 包名：`lowercase`（如 `com.example.project`）

**注释规范：**
```java
/**
 * 数据处理器
 * 负责清洗和转换原始数据
 * 
 * @author xty
 * @since 2026-06-16
 */
public class DataProcessor {
    
    /**
     * 处理数据
     * 
     * @param rawData 原始数据
     * @return 处理后的数据
     * @throws DataException 数据异常
     */
    public Data processData(RawData rawData) throws DataException {
        // 实现逻辑
    }
}
```

### 6.2 代码质量要求

**必需实践：**
- [ ] 单元测试覆盖率 > 70%
- [ ] 所有公共 API 必须有 Javadoc
- [ ] 关键业务逻辑必须有注释
- [ ] 无警告编译通过
- [ ] 代码格式化统一

---

## 7. 质量保证

### 7.1 检查清单

**提交前检查：**
- [ ] 代码编译通过
- [ ] 测试全部通过
- [ ] 代码规范检查通过
- [ ] Architecture.md 已更新
- [ ] README.md 已更新
- [ ] File_Index.md 已更新
- [ ] Git commit 信息规范

### 7.2 Code Review 要点

- 代码逻辑正确性
- 异常处理完整性
- 性能考虑
- 安全性检查
- 文档同步检查

---

## 8. 工具和配置

### 8.1 推荐工具

| 工具类型 | 推荐工具 | 用途 |
|---------|---------|------|
| IDE | IntelliJ IDEA | Java 开发 |
| 构建工具 | Maven / Gradle | 依赖管理和构建 |
| 版本控制 | Git | 版本管理 |
| 代码格式化 | Google Java Format | 统一代码风格 |

### 8.2 Maven 配置示例

```xml
<properties>
    <java.version>17</java.version>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

---

## 9. 最佳实践建议

### 9.1 GitHub 工作流（强烈推荐）

**遵从 GitHub 工作流，积极管理 Issue 和 GitHub Projects。**

#### Issue 管理

**使用 Issue 追踪：**
- 功能需求（Feature Request）
- Bug 报告（Bug Report）
- 技术债务（Technical Debt）
- 文档改进（Documentation）

**Issue 最佳实践：**
```markdown
## 标题格式
[类型] 简短描述

## 内容模板
### 问题描述
...

### 复现步骤（Bug）
1. 步骤一
2. 步骤二

### 期望行为
...

### 相关截图
...

### 环境信息
- Java 版本：
- 操作系统：
```

**标签使用：**
- `bug` - 缺陷
- `enhancement` - 功能增强
- `documentation` - 文档相关
- `good first issue` - 新手友好
- `help wanted` - 需要帮助
- `priority: high/medium/low` - 优先级

#### GitHub Projects 管理

**使用 Projects 进行任务规划：**

1. **创建 Project Board（看板）**
   - Todo（待办）
   - In Progress（进行中）
   - Review（审查中）
   - Done（完成）

2. **将 Issue 关联到 Project**
   - 每个 Issue 应该在 Project 中有对应卡片
   - 移动卡片反映工作进度

3. **里程碑（Milestones）规划**
   - 版本发布（v1.0, v2.0）
   - 阶段目标（MVP, Beta, Release）

#### Pull Request 工作流

```
1. 创建 Issue 描述需求/Bug
   ↓
2. 创建分支 (feature/xxx, bugfix/xxx)
   ↓
3. 开发 + 提交
   ↓
4. 创建 PR 并关联 Issue
   ↓
5. Code Review
   ↓
6. 合并到主分支
   ↓
7. 关闭 Issue（提供详实依据）
```

**PR 模板：**
```markdown
## 变更说明
...

## 关联 Issue
Closes #123

## 测试
- [ ] 单元测试通过
- [ ] 手动测试通过

## 检查清单
- [ ] 代码符合规范
- [ ] 文档已更新
- [ ] AGENTS submodule 已更新
```

---

#### Issue 关闭规范（强制）

**关闭 Issue 时必须提供详实的关闭依据。**

##### 必须提供的信息

**通过 PR 关闭：**
```markdown
## 问题已修复

**修复方式：** [描述如何修复的]

**关联 PR：** #456

**验证方式：**
- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] 手动验证通过

**相关提交：**
- abc1234 - fix: 修复登录失败问题
- def5678 - test: 添加登录测试用例
```

**标记为重复：**
```markdown
## 重复 Issue

此问题与 #123 重复。

**相同点：**
- 相同的错误信息
- 相同的复现步骤
- 相同的影响范围

已在 #123 中跟踪和修复。
```

**标记为无法复现：**
```markdown
## 无法复现

**测试环境：**
- Java 版本：17
- 操作系统：Ubuntu 22.04
- 项目版本：v1.2.0

**测试步骤：**
1. [步骤 1]
2. [步骤 2]
3. [步骤 3]

**测试结果：** 未能复现问题

**建议：**
如果问题仍然存在，请提供：
- 详细的错误日志
- 完整的环境信息
- 精确的复现步骤
```

**标记为不会修复：**
```markdown
## 不会修复

**原因：** [说明为什么不修复]

**理由：**
- 影响范围极小
- 修复成本过高
- 存在替代方案

**替代方案：**
[提供替代解决方案]
```

**标记为设计如此：**
```markdown
## 符合设计预期

这是按设计工作的预期行为。

**设计文档：** [链接到相关设计文档]

**原因：**
[解释为什么这样设计]

**如果需要改变此行为：**
请创建新的 Feature Request Issue 讨论需求。
```

##### 关闭 Issue 的完整流程

```bash
# 1. 确认问题已解决
# 2. 在 Issue 中添加关闭说明（使用上述模板）
# 3. 关闭 Issue
gh issue close 123 --comment "## 问题已修复
修复方式：修改了登录逻辑，添加了参数校验
关联 PR：#456
验证方式：单元测试和手动测试均通过"

# 或通过 PR 自动关闭（在 PR 描述中使用 Closes #123）
```

##### 禁止的关闭方式

❌ **直接关闭，不提供任何说明**
- 违规示例：直接点击 Close Issue 按钮，没有任何评论

❌ **提供模糊的说明**
- 违规示例："已修复"、"完成了"、"不需要了"
- 缺少具体信息，无法追溯

❌ **关闭后不更新文档**
- 如果 Issue 涉及文档，关闭时必须确认文档已更新

##### GitHub Actions 自动验证

**可以配置 GitHub Actions 检查关闭时是否有评论：**

```yaml
# .github/workflows/issue-close-check.yml
name: Check Issue Close Reason

on:
  issues:
    types: [closed]

jobs:
  check-close-reason:
    runs-on: ubuntu-latest
    steps:
    - name: Check if close reason provided
      uses: actions/github-script@v6
      with:
        script: |
          const issue = context.payload.issue;
          
          // 获取关闭前的最后评论
          const comments = await github.rest.issues.listComments({
            owner: context.repo.owner,
            repo: context.repo.repo,
            issue_number: issue.number,
            per_page: 10,
            sort: 'created',
            direction: 'desc'
          });
          
          const lastComment = comments.data[0];
          const closedByPR = issue.pull_request !== undefined;
          
          // 如果通过 PR 关闭，或者有关闭说明评论，则通过
          if (closedByPR || (lastComment && lastComment.body.length > 50)) {
            console.log('✅ 关闭依据充分');
          } else {
            // 重新打开 Issue 并要求提供关闭依据
            await github.rest.issues.update({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: issue.number,
              state: 'open'
            });
            
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: issue.number,
              body: '⚠️ **关闭 Issue 时必须提供详实的关闭依据**\n\n请参考 PROJECT.md 中的 Issue 关闭规范，提供以下信息之一：\n- 问题已修复（含 PR 链接和验证方式）\n- 标记为重复（指明重复的 Issue）\n- 无法复现（提供测试环境和步骤）\n- 不会修复（说明原因和替代方案）\n- 符合设计预期（解释设计原因）\n\n提供说明后可以再次关闭此 Issue。'
            });
          }
```

##### 最佳实践

1. **关闭前检查清单**
   - [ ] 问题确实已解决/不需要解决
   - [ ] 已添加关闭说明评论
   - [ ] 相关文档已更新
   - [ ] 测试已通过

2. **使用标签辅助说明**
   - `fixed` - 问题已修复
   - `duplicate` - 重复问题
   - `wontfix` - 不会修复
   - `invalid` - 无效问题
   - `works-as-designed` - 符合设计

3. **引用相关资源**
   - PR 链接
   - 相关 Issue 链接
   - 文档链接
   - 测试报告链接
- [ ] 代码符合规范
- [ ] 文档已更新
- [ ] AGENTS submodule 已更新
```

#### 团队协作建议

- **定期同步：** 每日/每周同步 Issue 和 Project 状态
- **代码审查：** 所有 PR 必须至少一人 Review
- **文档优先：** 重要决策记录在 Issue 或 Wiki 中
- **自动化：** 利用 GitHub Actions 进行 CI/CD

---

### 9.2 GitHub CLI 工具（推荐）

**推荐使用 `gh` CLI 提高 GitHub 工作流效率。**

#### 安装

```bash
# Windows (使用 scoop 或 winget)
scoop install gh
# 或
winget install --id GitHub.cli

# macOS
brew install gh

# Linux
# 参考 https://github.com/cli/cli#installation
```

#### 认证

```bash
gh auth login
```

#### 常用命令

**Issue 管理：**
```bash
# 创建 Issue
gh issue create --title "Bug: 登录失败" --body "描述..."

# 列出 Issue
gh issue list

# 查看 Issue 详情
gh issue view 123

# 关闭 Issue
gh issue close 123
```

**Pull Request 管理：**
```bash
# 创建 PR
gh pr create --title "feat: 添加用户模块" --body "关联 Issue #123"

# 列出 PR
gh pr list

# 查看 PR
gh pr view 456

# 检出 PR 到本地
gh pr checkout 456

# 合并 PR
gh pr merge 456 --squash
```

**仓库管理：**
```bash
# 克隆仓库（自动配置 upstream）
gh repo clone BigData2026QDU/AGENTS

# 查看仓库信息
gh repo view

# Fork 仓库
gh repo fork
```

**工作流查看：**
```bash
# 查看 GitHub Actions 运行状态
gh run list

# 查看具体运行日志
gh run view 789
```

#### 优势

- **快速操作：** 命令行直接管理 Issue/PR，无需打开浏览器
- **脚本集成：** 可集成到 Shell 脚本和 CI/CD 流程
- **提高效率：** 减少上下文切换，提升开发体验

---

## 10. 附录

### 9.1 文档版本

- 版本：1.0
- 创建日期：2026-06-16
- 维护者：xty
- 仓库地址：https://github.com/BigData2026QDU/AGENTS

### 9.2 参考资料

- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)

### 9.3 FAQ

**Q: 为什么必须使用与仓库同名的代码目录？**  
A: 统一结构，便于多仓库管理和自动化工具识别。

**Q: 文档必须用中文吗？**  
A: 是的，考虑到团队协作效率，统一使用中文。

**Q: 如何处理技术栈版本变更？**  
A: 先更新本文档技术栈章节，然后通知所有开发者，再进行迁移。

---

## 10. 变更日志

| 版本 | 日期 | 变更内容 | 作者 |
|------|------|---------|------|
| 1.1 | 2026-06-24 | 明确 Java、Spark、Scala、数据库版本要求及选择原因 | LuckyAnJun |
| 1.0 | 2026-06-16 | 初始版本，整合 AGENTS.md 和 CLAUDE.md | xty |

---

**本文档是项目的核心规范，所有开发者必须遵守。**
