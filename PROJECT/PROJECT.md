# PROJECT.md

> 学期项目工程规范 - Spark 相关大数据项目

## 1. 项目概述

本项目为学期项目的工程规范文档，定义了统一的仓库结构、文档标准、开发流程和代码规范。

**目标：** 确保项目一致性、可维护性和可协作性。

---

## 2. 技术栈

### 2.1 核心技术栈

| 技术 | 版本 | 状态 | 说明 |
|------|------|------|------|
| Java | 17 (JDK 17) | 必需 | 主要开发语言 |
| Apache Spark | TBD | 规划中 | 大数据处理框架 |
| Python | TBD | 规划中 | 数据分析和脚本 |

### 2.2 技术栈确认流程

**关键原则：** 在编写任何代码前，必须确认技术栈版本与项目要求一致。

1. 查阅本文档技术栈章节
2. 确认 JDK 版本为 17
3. 确认其他依赖的版本号
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

## 9. 附录

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
| 1.0 | 2026-06-16 | 初始版本，整合 AGENTS.md 和 CLAUDE.md | xty |

---

**本文档是项目的核心规范，所有开发者必须遵守。**
