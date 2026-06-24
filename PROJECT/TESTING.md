# TESTING.md

> 测试框架标准 - 统一测试规范

## 0. 核心原则（CRITICAL）

### 0.1 符合项目统一大标准（强制）

**所有测试框架必须遵守 PROJECT.md 中定义的统一大标准。**

**必须满足：**
- [ ] 遵守仓库结构规范（PROJECT.md §3）
- [ ] 遵守文档要求（PROJECT.md §4）
- [ ] 遵守 Git 工作流规范（PROJECT.md §5）
- [ ] Git Push 前确保 AGENTS submodule 最新（PROJECT.md §0.1）
- [ ] 遵守 GitHub 工作流最佳实践（PROJECT.md §9.1）

---

### 0.2 Submodule 管理规范（强制）

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
├── Architecture.md           # 架构文档（必需）
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

1. ✅ **被测项目必须在 `projects/` 目录下**
2. ✅ **每个被测项目独立一个 submodule**
3. ❌ **禁止直接复制或嵌套被测项目代码**
4. ❌ **禁止修改 submodule 内的 .git 目录**

---

### 0.3 测试通过后的 Issue 通知机制（强制）

**测试框架在测试成功后，必须在被测仓库创建带有特定前缀的 Issue，以触发被测仓库的构建和发布 CI。**

#### Issue 通知规则

**前缀标准：**
- Java/Spark 项目：`[可发布]`
- Web 前端项目：`[可发布]`
- 其他项目：根据项目类型定义

#### 自动化实现（GitHub Actions）

```yaml
# .github/workflows/test.yml
name: Test Projects

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
    
    - name: Set up environment
      run: |
        # 根据项目类型设置环境
        # Java: setup-java
        # Node: setup-node
    
    - name: Run tests
      id: test
      run: |
        # 执行测试脚本
        ./scripts/run-tests.sh
    
    - name: Create release issue on success
      if: success()
      uses: actions/github-script@v6
      with:
        github-token: ${{ secrets.PAT_TOKEN }}
        script: |
          const projects = ['ProjectA', 'ProjectB'];
          
          for (const project of projects) {
            // 检查项目测试是否通过
            const testPassed = true; // 根据实际测试结果判断
            
            if (testPassed) {
              // 在被测仓库创建 Issue
              await github.rest.issues.create({
                owner: 'BigData2026QDU',
                repo: project,
                title: '[可发布] 测试通过 - 可以触发构建',
                body: `## 测试报告
                
测试框架：${{ github.repository }}
测试时间：${{ github.event.head_commit.timestamp }}
提交 SHA：${{ github.sha }}
测试结果：✅ 通过

## 详细信息
- 测试覆盖率：${coverage}%
- 测试用例数：${testCount}
- 执行时长：${duration}s

## 下一步
此 Issue 将触发被测仓库的构建和发布 CI。

---
🤖 由测试框架自动创建
                `,
                labels: ['release', 'ci', 'auto-generated']
              });
            }
          }
```

#### 被测仓库的响应 CI

**被测仓库的 `.github/workflows/publish.yml`：**

```yaml
name: Build and Publish

on:
  issues:
    types: [opened]

jobs:
  check-trigger:
    runs-on: ubuntu-latest
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
    
    - name: Build and publish
      run: |
        # 构建和发布逻辑
        mvn clean package deploy
    
    - name: Close issue on success
      if: success()
      uses: actions/github-script@v6
      with:
        script: |
          await github.rest.issues.update({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            state: 'closed'
          });
          
          await github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: '✅ 构建和发布成功！'
          });
```

#### 为什么使用 Issue 触发

**优势：**
- ✅ 可追溯的触发历史
- ✅ 自动记录测试报告
- ✅ 便于审查和回溯
- ✅ 避免跨仓库的直接 workflow 调用
- ✅ 提供人工干预的窗口

**注意事项：**
- Issue 会自动关闭，不会堆积
- 需要配置 PAT_TOKEN（Personal Access Token）
- PAT 需要 `repo` 和 `issues` 权限

---

### 0.4 最小化代码侵入原则（强制）

**尽量不要改动被测仓库代码，如果必须改动，不得影响核心业务逻辑。**

#### 允许的改动

✅ **配置文件调整**
- 测试配置（`application-test.properties`）
- 依赖版本（仅用于测试环境）
- 日志级别（测试时降低输出）

✅ **测试辅助代码**
- 添加测试接口（标注 `@VisibleForTesting`）
- Mock 数据生成器
- 测试工具类

✅ **构建脚本**
- 添加测试相关的 Maven/Gradle 插件
- CI/CD 配置文件

#### 禁止的改动

❌ **核心业务逻辑**
- 修改业务代码以通过测试
- 删除或注释核心功能
- 降低代码质量以绕过测试

❌ **破坏性变更**
- 修改公共 API
- 删除已有功能
- 改变数据结构

❌ **规避测试**
- 添加 `@Ignore` 跳过失败的测试
- 修改测试条件使其必然通过
- 降低测试覆盖率要求

#### 改动审查流程

**如果必须改动被测仓库代码：**

1. **在测试框架中创建 Issue 说明原因**
   ```markdown
   ## 需要改动的代码
   仓库：ProjectName
   文件：src/main/java/...
   
   ## 改动原因
   为什么必须改动？现有方式无法测试的具体原因。
   
   ## 改动内容
   具体改动什么？
   
   ## 影响评估
   - 是否影响核心逻辑：否
   - 是否影响 API：否
   - 是否需要文档更新：是/否
   ```

2. **在被测仓库创建 PR**
   - 明确标注 `[测试需求]`
   - 关联测试框架的 Issue
   - 等待至少 1 人 Review

3. **优先考虑替代方案**
   - 使用反射访问私有方法
   - 使用 Mock 框架模拟依赖
   - 调整测试策略而非修改代码

---

### 0.5 自动问题报告机制（强制）

**测试框架在发现问题时，必须自动在被测仓库创建 Issue，附带完整的问题报告。**

#### 触发条件

以下情况必须自动创建 Issue：

1. **测试失败**
   - 单元测试失败
   - 集成测试失败
   - E2E 测试失败

2. **覆盖率不达标**
   - 总体覆盖率 < 要求阈值
   - 核心模块覆盖率不足

3. **代码规范违规**
   - 包命名错误
   - 使用了保留字
   - 目录结构不符合规范

4. **性能问题**
   - 测试执行时间超过阈值
   - 内存使用异常
   - 响应时间过长

5. **安全漏洞**
   - 依赖项安全漏洞
   - 代码安全扫描发现问题

#### Issue 标题格式

```
[测试报告] 问题类型 - 简短描述
```

**示例：**
- `[测试报告] 测试失败 - UserServiceTest.testLogin 失败`
- `[测试报告] 覆盖率不达标 - 总体覆盖率 65% (要求 ≥70%)`
- `[测试报告] 代码规范 - 包名使用了大写字母`
- `[测试报告] 性能问题 - 数据处理耗时 5.2s (阈值 3s)`
- `[测试报告] 安全漏洞 - 依赖项存在高危漏洞`

#### 完整问题报告模板

```markdown
## 📋 问题概述

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

### 性能数据（如适用）
- 执行时长：5.2s（阈值：3s）⚠️ 超时
- 内存使用：512MB（峰值：1024MB）
- GC 次数：15 次

### 安全扫描结果（如适用）
| 依赖项 | 版本 | 漏洞 | CVE 编号 | 严重级别 |
|--------|------|------|----------|----------|
| log4j-core | 2.14.1 | 远程代码执行 | CVE-2021-44228 | 🔴 严重 |

---

## 🛠️ 复现步骤

1. 克隆被测仓库：`git clone ...`
2. 切换到问题分支/提交：`git checkout <commit-sha>`
3. 运行测试命令：`mvn test` / `npm test`
4. 观察错误输出

---

## 💡 建议修复方案

### 方案一（推荐）
[描述推荐的修复方案]

**优点：**
- [优点 1]
- [优点 2]

**实施步骤：**
1. [步骤 1]
2. [步骤 2]

### 方案二（备选）
[描述备选方案]

---

## 📎 相关资源

- 测试报告完整日志：[链接]
- 覆盖率报告：[链接]
- CI 运行日志：[链接]
- 相关文档：[链接]

---

## ✅ 验收标准

问题修复后需满足以下条件：

- [ ] 所有测试通过
- [ ] 覆盖率达标（≥70%）
- [ ] 代码规范检查通过
- [ ] 性能指标正常
- [ ] 安全扫描无高危漏洞
- [ ] 文档已更新

---

## 🏷️ 标签

`test-report` `auto-generated` `[问题类型]` `[严重程度]`

---

🤖 **此 Issue 由测试框架自动生成**  
📅 **生成时间：** 2026-06-16 14:30:45  
🔗 **测试运行：** https://github.com/BigData2026QDU/JavaTestSkeleton/actions/runs/12345
```

#### GitHub Actions 自动化实现

```yaml
# .github/workflows/test.yml
name: Run Tests and Report Issues

on:
  push:
    branches: [ master, main ]
  pull_request:
    branches: [ master, main ]
  schedule:
    - cron: '0 2 * * *'

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
      with:
        submodules: recursive
    
    - name: Set up environment
      run: |
        # 配置测试环境
    
    - name: Run tests
      id: test
      continue-on-error: true
      run: |
        ./scripts/run-all-tests.sh > test-output.log 2>&1
        echo "test_exit_code=$?" >> $GITHUB_OUTPUT
    
    - name: Generate test report
      if: always()
      run: |
        ./scripts/generate-test-report.sh
    
    - name: Parse test results
      if: always()
      id: parse
      run: |
        # 解析测试结果，生成 JSON
        python scripts/parse-results.py > results.json
    
    - name: Create issue for test failures
      if: steps.test.outputs.test_exit_code != '0'
      uses: actions/github-script@v6
      with:
        github-token: ${{ secrets.PAT_TOKEN }}
        script: |
          const fs = require('fs');
          const results = JSON.parse(fs.readFileSync('results.json', 'utf8'));
          
          for (const project of results.projects) {
            if (project.hasFailed) {
              // 获取仓库所有者
              const repo = await github.rest.repos.get({
                owner: 'BigData2026QDU',
                repo: project.name
              });
              const repoOwner = repo.data.owner.login;
              
              // 获取最近的 milestone
              const milestones = await github.rest.issues.listMilestones({
                owner: 'BigData2026QDU',
                repo: project.name,
                state: 'open',
                sort: 'due_on',
                direction: 'asc'
              });
              const nearestMilestone = milestones.data.length > 0 ? milestones.data[0].number : null;
              
              const issueBody = `## 📋 问题概述

**问题类型：** ${project.problemType}
**严重程度：** ${project.severity}
**发现时间：** ${new Date().toISOString()}
**测试框架：** ${{ github.repository }}
**测试运行 ID：** #${{ github.run_id }}

---

## 🔍 详细信息

### 问题描述
${project.description}

### 影响范围
- 影响的文件：${project.affectedFiles.join(', ')}
- 是否阻塞发布：${project.blocksRelease ? '是' : '否'}

---

## 📊 测试数据

${project.testData}

---

## 🛠️ 复现步骤

1. 克隆被测仓库：\`git clone https://github.com/BigData2026QDU/${project.name}.git\`
2. 切换到提交：\`git checkout ${context.sha}\`
3. 运行测试：\`${project.testCommand}\`
4. 观察错误输出

---

## 💡 建议修复方案

${project.fixSuggestions}

---

## 📎 相关资源

- 测试报告：https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }}
- 覆盖率报告：${project.coverageUrl}

---

## ✅ 验收标准

- [ ] 所有测试通过
- [ ] 覆盖率达标
- [ ] 代码规范检查通过

---

🤖 **此 Issue 由测试框架自动生成**
📅 **生成时间：** ${new Date().toLocaleString('zh-CN')}
🔗 **测试运行：** https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }}
`;

              // 创建 Issue
              const issue = await github.rest.issues.create({
                owner: 'BigData2026QDU',
                repo: project.name,
                title: `[测试报告] ${project.problemType} - ${project.summary}`,
                body: issueBody,
                assignees: [repoOwner],  // 指派给仓库所有者
                labels: ['test-report', 'auto-generated', project.problemType, project.severity],
                milestone: nearestMilestone  // 关联最近的 milestone
              });
              
              // 添加到组织 Project
              // 注意：需要使用 GraphQL API 或 Projects (beta) API
              try {
                // 使用 GraphQL 添加到 Project
                await github.graphql(`
                  mutation($projectId: ID!, $contentId: ID!) {
                    addProjectV2ItemById(input: {
                      projectId: $projectId
                      contentId: $contentId
                    }) {
                      item {
                        id
                      }
                    }
                  }
                `, {
                  projectId: 'PVT_kwDOABcDhM4ApqXW',  // Project ID for https://github.com/orgs/BigData2026QDU/projects/3
                  contentId: issue.data.node_id
                });
              } catch (error) {
                console.log('无法添加到 Project:', error.message);
              }
            }
          }
    
    - name: Create success issue if all passed
      if: steps.test.outputs.test_exit_code == '0'
      uses: actions/github-script@v6
      with:
        github-token: ${{ secrets.PAT_TOKEN }}
        script: |
          // 创建 [可发布] Issue（参见 0.3 节）
```

#### Issue 配置要求（强制）

**自动创建的 Issue 必须包含以下配置：**

1. **Assignees（指派人）**
   - 自动指派给仓库所有者
   - 获取方式：通过 GitHub API 获取 `repo.data.owner.login`

2. **Labels（标签）**
   - 必须包含：`test-report`、`auto-generated`
   - 根据问题类型添加：
     - `test-failure` - 测试失败
     - `coverage-insufficient` - 覆盖率不足
     - `code-violation` - 代码规范违规
     - `performance-issue` - 性能问题
     - `security-vulnerability` - 安全漏洞
   - 根据严重程度添加：
     - `priority: critical` (🔴)
     - `priority: high` (🟠)
     - `priority: medium` (🟡)
     - `priority: low` (🔵)

3. **Project（项目看板）**
   - 组织 Project：https://github.com/orgs/BigData2026QDU/projects/3
   - 名称：BiD2026QDU's KanBan
   - 通过 GraphQL API 添加 Issue 到 Project
   - Project ID: `PVT_kwDOABcDhM4ApqXW`

4. **Milestone（里程碑）**
   - 自动关联到最近的 open milestone
   - 获取方式：
     ```javascript
     const milestones = await github.rest.issues.listMilestones({
       owner: 'BigData2026QDU',
       repo: project.name,
       state: 'open',
       sort: 'due_on',      // 按到期日期排序
       direction: 'asc'     // 升序，最近的在前
     });
     const nearestMilestone = milestones.data[0].number;
     ```
   - 如果没有 open milestone，则不关联

#### 获取 Project ID

**方法一：通过 GraphQL 查询**

```bash
# 使用 gh CLI
gh api graphql -f query='
  query {
    organization(login: "BigData2026QDU") {
      projectV2(number: 3) {
        id
        title
      }
    }
  }
'
```

**方法二：通过浏览器开发者工具**
1. 打开 Project 页面
2. 打开浏览器开发者工具（F12）
3. 在 Network 标签中查找 GraphQL 请求
4. 找到 `projectId` 字段

**方法三：使用 Projects (beta) API**

```javascript
const projects = await github.rest.projects.listForOrg({
  org: 'BigData2026QDU'
});
// 注意：这个 API 返回的是旧版 Projects，新版 Projects 需要用 GraphQL
```

#### 问题分类和严重程度定义

**问题类型：**
- `test-failure` - 测试失败
- `coverage-insufficient` - 覆盖率不足
- `code-violation` - 代码规范违规
- `performance-issue` - 性能问题
- `security-vulnerability` - 安全漏洞

**严重程度：**
- 🔴 **严重 (Critical)**: 阻塞发布，必须立即修复
  - 测试大面积失败
  - 核心功能无法工作
  - 严重安全漏洞
  
- 🟠 **重要 (High)**: 影响主要功能，需尽快修复
  - 重要测试失败
  - 覆盖率严重不达标（<60%）
  - 性能严重下降
  
- 🟡 **一般 (Medium)**: 影响次要功能，正常排期
  - 少量测试失败
  - 覆盖率略低（60-70%）
  - 轻微性能问题
  
- 🔵 **提示 (Low)**: 建议改进，不影响发布
  - 代码风格问题
  - 文档缺失
  - 优化建议

#### 自动关闭机制

**问题修复后自动关闭：**

```yaml
# 被测仓库的 .github/workflows/close-test-issues.yml
name: Close Fixed Test Issues

on:
  push:
    branches: [ master, main ]

jobs:
  check-and-close:
    runs-on: ubuntu-latest
    steps:
    - name: Get open test issues
      uses: actions/github-script@v6
      with:
        script: |
          const issues = await github.rest.issues.listForRepo({
            owner: context.repo.owner,
            repo: context.repo.repo,
            labels: 'test-report',
            state: 'open'
          });
          
          // 触发测试验证，如果通过则关闭对应 Issue
```

#### 通知机制

**重要问题通知相关人员：**

```yaml
- name: Notify on critical issues
  if: contains(steps.parse.outputs.severity, 'critical')
  uses: actions/github-script@v6
  with:
    script: |
      // 发送通知到 Slack/邮件/Discord
      // 或在 Issue 中 @mention 相关负责人
```

#### 最佳实践

1. **避免 Issue 泛滥**
   - 相同问题不重复创建（检查现有 Issue）
   - 问题修复后自动关闭
   - 定期清理过期 Issue

2. **提供可操作的信息**
   - 包含完整的错误信息和堆栈
   - 提供复现步骤
   - 给出修复建议

3. **分类清晰**
   - 使用统一的标签
   - 明确严重程度
   - 关联相关 PR/Commit

4. **保持更新**
   - 问题状态变化时更新 Issue
   - 添加修复进度评论
   - 关联相关讨论

---

## 1. 测试框架类型

### 1.1 Java 测试框架

**适用于：** Java 项目、Spark 项目

**测试框架仓库：** https://github.com/BigData2026QDU/JavaTestSkeleton

**技术栈：**
- JUnit 5
- Mockito
- Maven Surefire
- JaCoCo（覆盖率）
- PIT（变异测试）

**参考规范：** JavaTestSkeleton/README.md

---

### 1.2 前端测试框架

**适用于：** Web 前端项目

**测试框架仓库：** https://github.com/BigData2026QDU/FrontendTestSkeleton

**技术栈：**
- Mocha / Jest（单元测试）
- Playwright / Cypress（E2E 测试）
- ESLint（代码检查）
- 自定义规范检查脚本

**测试内容：**
- 目录结构验证（css/js/html）
- 技术栈白名单检查
- 模块实现验证（network.js, theme.js）
- 交互式设计检查

---

### 1.3 Python 测试框架（规划中）

**适用于：** Python 数据分析、脚本项目

**技术栈：**
- pytest
- coverage.py
- pylint / flake8

---

## 2. 测试标准

### 2.1 测试覆盖率要求

**最低要求：**
| 项目类型 | 单元测试覆盖率 | 集成测试覆盖率 |
|---------|---------------|---------------|
| Java 后端 | ≥ 70% | ≥ 50% |
| Web 前端 | ≥ 60% | - |
| 数据处理 | ≥ 70% | ≥ 60% |

**核心业务逻辑要求：**
- 核心业务逻辑覆盖率 ≥ 90%
- 关键算法覆盖率 = 100%

---

### 2.2 测试质量要求

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

### 2.3 测试数据管理

**测试数据原则：**
- 使用独立的测试数据库
- 测试前初始化数据
- 测试后清理数据
- 不依赖生产数据

**测试数据组织：**
```
TestFramework/
└── test-data/
    ├── sql/
    │   ├── schema.sql        # 数据库结构
    │   └── test-data.sql     # 测试数据
    ├── json/
    │   └── mock-api.json     # API Mock 数据
    └── csv/
        └── sample-data.csv   # 样本数据
```

---

## 3. CI/CD 集成

### 3.1 测试框架的 CI 配置

**`.github/workflows/test.yml`：**

```yaml
name: Run Tests

on:
  push:
    branches: [ master, main ]
  pull_request:
    branches: [ master, main ]
  schedule:
    - cron: '0 2 * * *'  # 每天凌晨 2 点运行
  workflow_dispatch:      # 手动触发

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      with:
        submodules: recursive
        token: ${{ secrets.PAT_TOKEN }}
    
    - name: Update AGENTS to latest
      run: |
        cd AGENTS
        git pull origin master
        cd ..
    
    - name: Set up test environment
      run: |
        # 根据项目类型配置环境
    
    - name: Run tests
      run: |
        ./scripts/run-all-tests.sh
    
    - name: Generate reports
      run: |
        ./scripts/generate-report.sh
    
    - name: Upload test results
      uses: actions/upload-artifact@v3
      with:
        name: test-reports
        path: reports/
    
    - name: Notify on success
      if: success()
      run: |
        ./scripts/create-release-issues.sh
    
    - name: Notify on failure
      if: failure()
      uses: actions/github-script@v6
      with:
        script: |
          // 通知相关人员测试失败
```

---

### 3.2 测试报告

**测试报告应包含：**
- 测试概览（通过/失败/跳过）
- 覆盖率报告（行覆盖率、分支覆盖率）
- 失败用例详情
- 性能指标
- 趋势图表

**报告存储：**
- GitHub Actions Artifacts
- GitHub Pages（静态报告）
- 独立的报告服务器

---

## 4. 最佳实践

### 4.1 持续集成

- **频繁运行：** 每次 push 都运行测试
- **快速反馈：** 测试应在 10 分钟内完成
- **并行执行：** 利用 GitHub Actions 矩阵策略并行测试
- **缓存依赖：** 缓存 Maven/npm 依赖加速构建

### 4.2 测试维护

- **定期更新：** 随被测项目演进更新测试
- **清理废弃测试：** 删除不再有效的测试
- **重构测试代码：** 保持测试代码质量
- **文档同步：** 测试文档与代码保持一致

### 4.3 团队协作

- **测试优先：** 新功能先写测试
- **代码审查：** 测试代码也需要 Review
- **知识共享：** 定期分享测试经验
- **问题追踪：** 使用 Issue 跟踪测试问题

---

## 5. 故障排查

### 5.1 常见问题

**Submodule 更新失败：**
```bash
# 解决方案
git submodule update --init --recursive --remote
```

**测试环境不一致：**
- 检查 JDK/Node.js 版本
- 检查依赖版本
- 使用 Docker 容器统一环境

**覆盖率不达标：**
- 检查是否有未测试的代码路径
- 增加边界条件测试
- 检查 Mock 是否正确

---

## 6. 附录

### 6.1 参考资料

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)

### 6.2 检查清单

**测试框架提交前检查：**
- [ ] 所有被测项目都是 submodule
- [ ] AGENTS submodule 已更新到最新
- [ ] 测试全部通过
- [ ] 覆盖率达标
- [ ] CI 配置正确
- [ ] 文档已更新
- [ ] Issue 通知机制配置完成

---

**文档版本：** 1.0  
**创建日期：** 2026-06-16  
**维护者：** xty  
**适用范围：** 所有测试框架仓库
