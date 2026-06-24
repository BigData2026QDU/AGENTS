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
