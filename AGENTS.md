# AGENTS.md - 学期项目规范

## 技术栈

- **Java 17** (JDK 17)
- 后续可能添加：
  - Apache Spark 版本
  - Python 版本
  - 其他技术栈

**注意：** 编程助手在开发前必须确认当前技术栈，确保代码符合版本要求。

---

## 仓库结构要求

每个仓库的最上层目录结构必须为：

```
repository-name/
├── repository-name/          # 与仓库同名的源代码文件夹
├── Architecture.md           # 中文架构说明文档
├── README.md                 # 中文项目简介及使用说明
├── File_Index.md             # 中文文件索引
├── .git/                     # Git 管理目录
├── .gitignore                # Git 忽略文件
└── 其他 Git 管理文件
```

**关键规则：**
1. 仅允许一个与仓库同名的源代码文件夹
2. 三份文档必须放在仓库根目录
3. 其他文件只能是 Git 管理相关文件

---

## 文档要求

### 1. Architecture.md (架构说明)

**格式：** Markdown (.md)
**语言：** 中文
**内容要求：**
- 项目整体架构图或描述
- 各个模块/组件的职责说明
- 模块之间的关系和依赖
- 数据流向说明
- 关键技术点说明

### 2. README.md (项目简介)

**语言：** 中文
**内容要求：**
- 项目简介和目标
- 主要功能特性
- 环境要求（Java 版本等）
- 快速开始指南
- 项目结构说明
- 如何构建和运行
- 如何贡献

### 3. File_Index.md (文件索引)

**语言：** 中文
**内容要求：**
- 每个文件的路径（相对于仓库根目录）
- 文件的主要作用
- 文件内容的简要介绍
- 按目录结构组织

---

## 开发流程

1. **确认技术栈：** 开发前必须确认使用的语言版本（当前为 Java 17）
2. **创建仓库结构：** 按照上述结构创建目录和文档
3. **编写代码：** 在对应的技术栈文件夹内开发
4. **更新文档：** 代码变更后同步更新三份文档
5. **提交代码：** 按照规范提交到 Git

---

## Java 包命名规范（必须遵守）

### 规则

1. **所有 Java 文件必须声明 `package` 语句**，且包名必须与文件所在目录结构一致
2. **包名只使用小写字母**，禁止使用大写字母、下划线、连字符
3. **包名使用反向域名格式**，例如 `org.example`、`com.bigdata.tool`
4. **禁止使用 Java 保留字作为包名或类名**，包括但不限于：
   - `value`、`test`、`class`、`new`、`import`、`package`、`public`、`private`、`protected`
5. **测试类的包结构必须与生产代码一致**，例如：
   ```
   src/main/java/org/example/Tool/HibernateUtil.java  → package org.example.Tool;
   src/test/java/org/example/Tool/HibernateUtilTest.java  → package org.example.Tool;
   ```

### 为什么重要

- **Maven Surefire/Failsafe 插件**依赖包名匹配来发现测试类
- **PIT 变异测试**需要正确的包路径来定位要变异的代码
- **JaCoCo 覆盖率**按包名统计，包结构错误会导致覆盖率为 0%
- **IDE 和 CI 工具**可能无法识别非标准包结构的代码

### 常见错误示例

```java
// ❌ 错误：文件在 Tool/ 目录，但包名是 tool（大小写不匹配）
package org.example.tool;  // 文件在 org/example/Tool/

// ❌ 错误：使用保留字 value 作为类名
public class value { }  // H2 数据库中 value 是保留字

// ❌ 错误：测试类在默认包（无 package 语句）
// src/test/java/MyTest.java 没有 package 语句

// ✅ 正确：包名与目录结构完全匹配
package org.example.Tool;  // 文件在 src/main/java/org/example/Tool/
```

---

## 测试项目规范

### Submodule 管理

**所有被测试的 Java 代码仓库必须作为 Git Submodule 加入到测试项目中。**

```
TestProject/
├── AGENTS/                    # 规范文档
├── TestProject/               # 测试代码
│   ├── pom.xml
│   └── src/test/java/...
├── projects/                  # 被测项目（全部为 submodule）
│   ├── ProjectA/
│   └── ProjectB/
```

**规则：**
1. 被测项目放在 `projects/` 目录下，每个项目独立一个 submodule
2. 测试项目通过 `build-helper-maven-plugin` 引入 submodule 源码
3. 测试项目的 `pom.xml` 中必须配置 submodule 的源码路径
4. 禁止直接复制或嵌套其他项目代码，必须使用 submodule

**为什么必须使用 Submodule：**
- 保持被测项目的独立版本管理
- 测试项目始终引用最新的被测代码
- CI/CD 自动拉取所有依赖的被测项目
- 避免代码重复和版本不一致

### pom.xml 配置示例

```xml
<!-- 引入 submodule 源码到编译路径 -->
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>add-project-sources</id>
            <phase>generate-sources</phase>
            <goals><goal>add-source</goal></goals>
            <configuration>
                <sources>
                    <source>../projects/被测项目/src/main/java</source>
                </sources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

---

## 版本信息

- 文档版本：1.2
- 创建日期：2026-06-09
- 更新日期：2026-06-09
- 维护者：xty
