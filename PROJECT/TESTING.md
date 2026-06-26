# TESTING.md

> 测试框架标准 - 统一测试规范

**文档版本：** 2.0  
**更新日期：** 2026-06-26  
**维护者：** xty  
**适用范围：** 所有测试框架仓库

---

## 1. 核心原则（CRITICAL）

### 1.1 符合项目统一大标准（强制）

**所有测试框架必须遵守 PROJECT.md 中定义的统一大标准。**

**必须满足：**
- [ ] 遵守仓库结构规范（PROJECT.md §3）
- [ ] 遵守文档要求（PROJECT.md §4）
- [ ] 遵守 Git 工作流规范（PROJECT.md §5）
- [ ] Git Push 前确保 AGENTS submodule 最新（PROJECT.md §0.1）
- [ ] 遵守 GitHub 工作流最佳实践（PROJECT.md §9.1）

---

### 1.2 Submodule 管理规范（强制）

**所有被测试的代码仓库必须作为 Git Submodule 加入到测试框架的 `projects/` 目录下。**

#### 标准测试框架结构

```
TestFramework/
├── AGENTS/                    # 规范文档 (submodule)
├── TestFramework/             # 测试框架代码
│   ├── pom.xml / package.json # 构建配置
│   ├── src/test/             # 测试代码
│   └── scripts/              # 测试脚本
├── projects/                  # 被测项目（全部为 submodule）
│   ├── ProjectA/
│   ├── ProjectB/
│   └── ProjectC/
├── .github/
│   └── workflows/
│       └── test.yml          # 测试 CI
├── Architecture.md           # 架档文档（必需）
├── README.md                 # 项目说明（必需）
└── File_Index.md             # 文件索引（必需）
```

#### 添加被测项目流程

```bash
# 1. 进入测试框架根目录
cd TestFramework

# 2. 添加 submodule
git submodule add https://github.com/BigData2026QDU/ProjectName.git projects/ProjectName

# 3. 提交变更
git add .gitmodules projects/
git commit -m "feat: 添加 ProjectName 到测试框架"
git push

# 4. 其他人更新 submodule
git pull
git submodule update --init --recursive
```

#### 关键约束

| 规则 | 说明 |
|------|------|
| ✅ **强制** | 被测项目必须在 `projects/` 目录下 |
| ✅ **强制** | 每个被测项目独立一个 submodule |
| ❌ **禁止** | 直接复制或嵌套被测项目代码 |
| ❌ **禁止** | 修改 submodule 内的 .git 目录 |

---

## 2. 测试触发与发布流程（CRITICAL）

### 2.1 完整流程概述

```
测试框架推送代码
    ↓
GitHub Actions 触发测试
    ↓
┌─────────────────────────────────────────────────┐
│                   测试失败                       │
│    ↓                                            │
│  在测试框架仓库创建 Issue（指派维护者+提交者）    │
│    ↓                                            │
│  等待修复后重新测试                              │
└─────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────┐
│                   测试通过                       │
│    ↓                                            │
│  在被测仓库创建 [可发布] Issue                   │
│    ↓                                            │
│  被测仓库 publish.yml 监听并触发                 │
│    ↓                                            │
│  构建 + 创建 GitHub Release                     │
└─────────────────────────────────────────────────┘
```

---

### 2.2 测试通过触发发布（强制）

**测试框架在测试成功后，必须在被测仓库创建 `[可发布]` Issue，触发被测仓库的构建和 Release。**

#### Issue 前缀标准

| 项目类型 | 前缀 | 示例 |
|---------|------|------|
| Java/Spark | `[可发布]` | `[可发布] 测试通过 - 可以发布` |
| Web 前端 | `[可发布]` | `[可发布] 测试通过 - 可以发布` |

#### 测试框架创建 Issue（强制实现）

```yaml
# .github/workflows/test.yml
name: Run Tests and Trigger Release

on:
  push:
    branches: [ master, main ]
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
      with:
        submodules: recursive
    
    - name: Run tests
      id: test
      run: |
        ./scripts/run-all-tests.sh
    
    - name: Create release trigger on success
      if: success()
      uses: actions/github-script@v6
      with:
        github-token: ${{ secrets.PAT_TOKEN }}
        script: |
          const projects = ['ProjectA', 'ProjectB'];
          
          for (const project of projects) {
            if (/* 项目测试通过 */) {
              // 在被测仓库创建 [可发布] Issue
              await github.rest.issues.create({
                owner: 'BigData2026QDU',
                repo: project,
                title: '[可发布] 测试通过 - 可以发布',
                body: `## 测试报告\n\n测试框架：${{ github.repository }}\n测试时间：${new Date().toISOString()}\n提交 SHA：${{ github.sha }}\n测试结果：✅ 通过\n\n---\n🤖 此 Issue 由测试框架自动创建，将触发构建和 Release`,
                labels: ['release', 'auto-generated']
              });
            }
          }
```

---

### 2.3 被测仓库响应 CI（强制配置）

**每个被测仓库必须配置 `.github/workflows/release.yml` 来响应 `[可发布]` Issue。**

#### 完整 release.yml 模板

```yaml
# .github/workflows/release.yml
name: Build and Release

on:
  issues:
    types: [opened]

jobs:
  release:
    runs-on: ubuntu-latest
    # 仅在 Issue 标题以 [可发布] 开头时触发
    if: startsWith(github.event.issue.title, '[可发布]')
    
    steps:
    - name: Acknowledge trigger
      uses: actions/github-script@v6
      with:
        script: |
          await github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: '✅ 收到测试通过通知，开始构建和发布流程...'
          });
    
    - uses: actions/checkout@v3
    
    - name: Set up environment
      run: |
        # 根据项目类型设置环境
        # Java: setup-java
        # Node: setup-node
    
    - name: Build
      run: |
        # 构建项目
        # Java: mvn clean package -DskipTests
        # Node: npm run build
    
    - name: Create GitHub Release
      if: success()
      id: create_release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: v${{ github.run_number }}
        release_name: Release v${{ github.run_number }}
        body: |
          ## 发布说明
          
          此 Release 由测试框架自动触发。
          
          **测试框架：** ${{ github.repository }}
          **触发时间：** ${{ github.event.issue.created_at }}
          **测试通过 Issue：** #${{ github.event.issue.number }}
        draft: false
        prerelease: false
    
    - name: Upload Release Asset
      if: success()
      uses: actions/upload-release-asset@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.create_release.outputs.upload_url }}
        asset_path: ./target/*.jar  # 根据项目类型调整
        asset_name: my-artifact.jar
        asset_content_type: application/java-archive
    
    - name: Close trigger issue on success
      if: success()
      uses: actions/github-script@v6
      with:
        script: |
          // 关闭触发 Issue
          await github.rest.issues.update({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            state: 'closed'
          });
          
          // 添加发布成功评论
          await github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: `✅ 构建和发布成功！\n\nRelease: ${{ steps.create_release.outputs.html_url }}`
          });
    
    - name: Comment on failure
      if: failure()
      uses: actions/github-script@v6
      with:
        script: |
          await github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: '❌ 构建失败，请检查日志。'
          });
```

#### 关键约束

| 规则 | 说明 |
|------|------|
| ✅ **强制** | 必须监听 `issues: [opened]` 事件 |
| ✅ **强制** | 必须检查 Issue 标题以 `[可发布]` 开头 |
| ✅ **强制** | 成功后必须创建 GitHub Release |
| ✅ **强制** | 成功后必须关闭触发 Issue |
| ✅ **强制** | 失败时必须在 Issue 中添加失败评论 |
| ❌ **禁止** | 跳过构建直接创建 Release |
| ❌ **禁止** | Release 创建失败后自动关闭 Issue |

---

### 2.4 版本号管理（强制）

**Release 版本号必须遵循语义化版本规范。**

#### 版本号格式

```
v{MAJOR}.{MINOR}.{PATCH}
```

| 类型 | 说明 | 示例 |
|------|------|------|
| MAJOR | 破坏性变更 | v2.0.0 |
| MINOR | 新增功能（向后兼容） | v1.1.0 |
| PATCH | Bug 修复 | v1.0.1 |

#### 版本号获取方式

```yaml
# 方式一：从 git tag 获取最新版本
- name: Get latest tag
  id: get_tag
  run: |
    LATEST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
    echo "latest_tag=$LATEST_TAG" >> $GITHUB_OUTPUT

# 方式二：从 package.json / pom.xml 获取
- name: Get version from package.json
  run: |
    VERSION=$(node -p "require('./package.json').version")
    echo "VERSION=$VERSION" >> $GITHUB_ENV
```

---

## 3. 测试失败问题报告（CRITICAL）

### 3.1 触发条件（强制）

以下情况**必须**自动创建 Issue：

| 条件 | 说明 |
|------|------|
| 测试失败 | 单元/集成/E2E 测试失败 |
| 覆盖率不达标 | 低于要求阈值 |
| 代码规范违规 | 命名错误、结构不符 |
| 性能问题 | 执行超时、内存异常 |
| 安全漏洞 | 依赖项存在高危漏洞 |

---

### 3.2 Issue 标题格式（强制）

```
[测试报告] 被测项目名 - 问题类型 - 简短描述
```

**示例：**
- `[测试报告] UserService - 测试失败 - UserServiceTest.testLogin 失败`
- `[测试报告] DataProcessor - 覆盖率不达标 - 总体覆盖率 65%`
- `[测试报告] AuthModule - 代码规范 - 包名使用了大写字母`
- `[测试报告] QueryEngine - 性能问题 - 数据处理耗时 5.2s`
- `[测试报告] WebAPI - 安全漏洞 - 依赖项存在高危漏洞`

---

### 3.3 Issue 创建位置（强制）

**在测试框架仓库创建 Issue，不是被测仓库。**

| 测试框架 | 仓库地址 |
|---------|---------|
| JavaTestSkeleton | https://github.com/BigData2026QDU/JavaTestSkeleton |
| FrontendTestSkeleton | https://github.com/BigData2026QDU/FrontendTestSkeleton |

---

### 3.4 指派规则（强制）

**Issue 必须同时指派给两人：**

1. **LuckyAnJun**（测试框架维护者）
2. **被测项目最近非 LuckyAnJun 提交者**

```javascript
// 获取被测项目最近提交（获取多个，以便找到非LuckyAnJun的提交者）
const commits = await github.rest.repos.listCommits({
  owner: 'BigData2026QDU',
  repo: project.name,
  per_page: 10  // 获取最近10个提交
});

// 找到第一个不是LuckyAnJun的提交作者
let commitAuthor = null;
for (const commit of commits.data) {
  const author = commit.author?.login || commit.commit.author.name;
  if (author && author !== 'LuckyAnJun') {
    commitAuthor = author;
    break;
  }
}

// 指派给两个人：LuckyAnJun和最近的非LuckyAnJun提交者
assignees: ['LuckyAnJun', commitAuthor].filter(Boolean)
```

---

### 3.5 标签规范（强制）

**Issue 必须包含以下标签：**

| 标签 | 说明 | 必须 |
|------|------|------|
| `test-report` | 测试报告 | ✅ |
| `auto-generated` | 自动生成 | ✅ |
| `needs-investigation` | 需要调查 | ✅ |
| `test-failure` | 测试失败 | 按需 |
| `coverage-insufficient` | 覆盖率不足 | 按需 |
| `code-violation` | 代码规范 | 按需 |
| `performance-issue` | 性能问题 | 按需 |
| `security-vulnerability` | 安全漏洞 | 按需 |
| `priority: critical` | 🔴 严重 | 按需 |
| `priority: high` | 🟠 重要 | 按需 |
| `priority: medium` | 🟡 一般 | 按需 |
| `priority: low` | 🔵 提示 | 按需 |

---

### 3.6 Issue 内容模板（强制）

```markdown
## 📋 问题概述

**被测项目：** [项目名称]  
**被测仓库：** https://github.com/BigData2026QDU/[项目名称]  
**问题类型：** [测试失败 | 覆盖率不达标 | 代码规范违规 | 性能问题 | 安全漏洞]  
**严重程度：** [🔴 严重 | 🟠 重要 | 🟡 一般 | 🔵 提示]  
**发现时间：** YYYY-MM-DD HH:mm:ss  
**测试框架：** JavaTestSkeleton / FrontendTestSkeleton  
**测试运行 ID：** #12345

---

## 🔍 详细信息

### 问题描述
[简洁明确地描述发现的问题]

### 影响范围
- 影响的文件/模块：
- 影响的功能：
- 是否阻塞发布：是/否

---

## 📊 测试数据

### 失败的测试用例（如适用）
```
测试类：org.example.UserServiceTest
测试方法：testLogin
失败原因：Expected <true> but was <false>
堆栈跟踪：
  at org.example.UserServiceTest.testLogin(UserServiceTest.java:45)
  ...
```

### 覆盖率数据（如适用）
| 指标 | 实际值 | 要求值 | 状态 |
|------|--------|--------|------|
| 行覆盖率 | 65% | ≥70% | ❌ |
| 分支覆盖率 | 58% | ≥60% | ❌ |
| 方法覆盖率 | 72% | ≥70% | ✅ |

---

## 🔗 错误上下文

### 被测项目信息
- **提交 SHA：** abc1234def5678
- **提交作者：** @username
- **提交时间：** YYYY-MM-DD HH:mm:ss
- **提交信息：** feat: 添加用户登录功能
- **分支：** master

### 测试环境
- **操作系统：** Ubuntu 22.04
- **Java 版本：** 17 (如适用)
- **Node 版本：** 18.x (如适用)

---

## 🛠️ 复现步骤

1. 克隆测试框架：`git clone https://github.com/BigData2026QDU/JavaTestSkeleton.git`
2. 更新 submodule：`git submodule update --init --recursive`
3. 切换被测项目到问题提交：`cd projects/[项目名] && git checkout abc1234`
4. 运行测试：`mvn test` / `npm test`
5. 观察错误输出

---

## 💡 可能的原因

### 原因一：被测代码问题
[描述被测代码可能存在的问题]

### 原因二：测试框架问题
[描述测试框架可能存在的问题]

### 原因三：环境问题
[描述环境配置可能存在的问题]

---

## 📎 相关资源

- 测试报告完整日志：[链接]
- 覆盖率报告：[链接]
- CI 运行日志：[链接]

---

## ✅ 验收标准

问题修复后需满足以下条件：

- [ ] 确认问题根源（被测代码 / 测试框架 / 环境）
- [ ] 所有测试通过
- [ ] 覆盖率达标（≥70%）
- [ ] 代码规范检查通过
- [ ] 性能指标正常
- [ ] 安全扫描无高危漏洞

---

## 🏷️ 标签

`test-report` `auto-generated` `[问题类型]` `[严重程度]` `needs-investigation`

---

## 👥 相关人员

- **被测项目作者：** @username
- **测试框架维护者：** @LuckyAnJun

---

🤖 **此 Issue 由测试框架自动生成**  
📅 **生成时间：** YYYY-MM-DD HH:mm:ss  
🔗 **测试运行：** https://github.com/BigData2026QDU/JavaTestSkeleton/actions/runs/12345  
⚠️ **注意：** 此问题可能是被测代码问题，也可能是测试框架问题，需要双方协作排查。
```

---

### 3.7 自动创建验收规则（强制）

**测试框架在创建 Issue 前，必须完成以下校验。校验失败时，CI 必须失败。**

| 规则 | 说明 |
|------|------|
| 标题可追踪 | 必须符合标准格式，项目名与 submodule 一致 |
| 正文信息完整 | 必须包含所有必需字段，缺失时写明 N/A |
| 指派准确 | 必须同时指派 LuckyAnJun 和提交者 |
| 标签准确 | 必须包含必需标签和问题类型/严重程度标签 |
| 禁止重复创建 | 创建前搜索同名 open Issue，重复时更新原 Issue |
| 失败即阻断 | 创建失败时 CI 必须标记失败 |
| 关闭条件明确 | 必须提供测试通过链接、修复提交、验证结果 |

---

## 4. 问题讨论和解决流程（强制）

### 4.1 流程步骤

| 步骤 | 说明 |
|------|------|
| 1. Issue 创建 | 被测项目作者和测试框架维护者收到通知 |
| 2. 问题排查 | 双方在 Issue 中讨论，确认问题根源 |
| 3. 问题修复 | 根据问题归属，由对应方修复 |
| 4. 验证关闭 | 重新测试通过后关闭 Issue，提供详实依据 |

---

### 4.2 关闭 Issue 规范（强制）

**关闭 Issue 时必须提供详实的关闭依据。**

**必须包含：**
- 通过的测试运行链接
- 修复提交或被验证提交
- 验证命令和结果摘要

**禁止：**
- 只评论"已修复"、"已完成"等无法追溯的说明
- 未经验证直接关闭

---

## 5. 最小化代码侵入原则（强制）

### 5.1 允许的改动

| 类型 | 说明 |
|------|------|
| 配置文件调整 | 测试配置、依赖版本（仅测试环境）、日志级别 |
| 测试辅助代码 | 测试接口（标注 `@VisibleForTesting`）、Mock 数据、工具类 |
| 构建脚本 | 测试相关插件、CI/CD 配置文件 |

### 5.2 禁止的改动

| 类型 | 说明 |
|------|------|
| 核心业务逻辑 | 修改业务代码以通过测试、删除/注释核心功能 |
| 破坏性变更 | 修改公共 API、删除已有功能、改变数据结构 |
| 规避测试 | 添加 `@Ignore`、修改测试条件、降低覆盖率要求 |

### 5.3 改动审查流程

**如果必须改动被测仓库代码：**

1. 在测试框架中创建 Issue 说明原因
2. 在被测仓库创建 PR，标注 `[测试需求]`
3. 等待至少 1 人 Review
4. 优先考虑替代方案（反射、Mock、调整测试策略）

---

## 6. 测试框架类型

### 6.1 Java 测试框架

**适用于：** Java 项目、Spark 项目

**测试框架仓库：** https://github.com/BigData2026QDU/JavaTestSkeleton

**技术栈：** JUnit 5, Mockito, Maven Surefire, JaCoCo, PIT

---

### 6.2 前端测试框架

**适用于：** Web 前端项目

**测试框架仓库：** https://github.com/BigData2026QDU/FrontendTestSkeleton

**技术栈：** Mocha/Jest, Playwright/Cypress, ESLint

---

### 6.3 Python 测试框架（规划中）

**适用于：** Python 数据分析、脚本项目

**技术栈：** pytest, coverage.py, pylint/flake8

---

## 7. 测试标准

### 7.1 测试覆盖率要求（强制）

| 项目类型 | 单元测试覆盖率 | 集成测试覆盖率 |
|---------|---------------|---------------|
| Java 后端 | ≥ 70% | ≥ 50% |
| Web 前端 | ≥ 60% | - |
| 数据处理 | ≥ 70% | ≥ 60% |

**核心业务逻辑要求：**
- 核心业务逻辑覆盖率 ≥ 90%
- 关键算法覆盖率 = 100%

---

### 7.2 测试质量要求（强制）

**必须包含：**
- [ ] 正向测试用例（正常流程）
- [ ] 负向测试用例（异常流程）
- [ ] 边界测试用例（临界值）
- [ ] 并发测试（如适用）
- [ ] 性能测试（关键路径）

**测试隔离：**
- 测试之间互不依赖
- 测试顺序无关
- 每个测试可独立运行

**测试命名规范：**
```java
// ✅ 好的命名：方法名_场景_期望结果
@Test
void processData_whenInputIsNull_shouldThrowException() { }

@Test
void calculateTotal_withValidItems_shouldReturnCorrectSum() { }

// ❌ 不好的命名
@Test
void test1() { }

@Test
void testProcess() { }
```

---

## 8. 检查清单

### 8.1 测试框架提交前检查

- [ ] 所有被测项目都是 submodule
- [ ] AGENTS submodule 已更新到最新
- [ ] 测试全部通过
- [ ] 覆盖率达标
- [ ] CI 配置正确
- [ ] 文档已更新
- [ ] Issue 通知机制配置完成
- [ ] 被测仓库配置 release.yml

---

**文档版本：** 2.0  
**更新日期：** 2026-06-26  
**维护者：** xty
