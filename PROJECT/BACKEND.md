# BACKEND.md

> 后端开发规范 - Java & Spark 大数据项目

## 0. 关键约束（CRITICAL）

### 0.1 Java 包命名规范（必须遵守）

**所有 Java 代码必须遵守以下包命名规范，否则测试工具将无法正常工作。**

#### 必须满足的条件

1. **每个 .java 文件必须有 `package` 声明**，且包名必须精确匹配文件目录路径
2. **包名全小写**，例如 `org.example.tool`，不能有大写字母
3. **包名使用反向域名格式**，例如 `org.example`、`com.bigdata.tool`
4. **测试类必须与生产代码在相同包结构下**：
   ```
   src/main/java/org/example/tool/Foo.java     → package org.example.tool;
   src/test/java/org/example/tool/FooTest.java → package org.example.tool;
   ```
5. **禁止使用 Java 保留字作为包名或类名**，包括但不限于：
   - `value`、`test`、`class`、`new`、`import`、`package`、`public`、`private`、`protected`、`void`、`int`、`string` 等

#### 为什么重要

- **Maven Surefire 插件**通过包名匹配发现测试类
- **PIT 变异测试**按包名定位要变异的生产代码
- **JaCoCo 覆盖率**按包名统计，包结构错误会导致覆盖率为 0%
- **H2 数据库**中 `value` 是 SQL 保留字，会导致 DDL 错误
- **IDE 和 CI 工具**可能无法识别非标准包结构的代码

#### 推荐包结构

```
org.example.{modulename}     → 例如 org.example.tool, org.example.service
```

#### 常见错误示例

```java
// ❌ 错误：文件在 Tool/ 目录，但包名是 tool（大小写不匹配）
package org.example.tool;  // 文件在 org/example/Tool/

// ❌ 错误：使用保留字 value 作为类名
public class value { }  // H2 数据库中 value 是保留字

// ❌ 错误：测试类在默认包（无 package 语句）
// src/test/java/MyTest.java 没有 package 语句

// ✅ 正确：包名与目录结构完全匹配，全小写
package org.example.tool;  // 文件在 src/main/java/org/example/tool/
```

---

### 0.2 Java 测试仓库规范（CRITICAL）

**所有 Java 代码必须作为 Git Submodule 加入到统一的 Java 测试仓库进行测试。**

#### Java 测试仓库

**仓库地址：** https://github.com/BigData2026QDU/JavaTestSkeleton

**测试框架规范：** https://github.com/BigData2026QDU/JavaTestSkeleton/blob/main/README.md

#### 标准测试仓库结构

```
JavaTestSkeleton/
├── AGENTS/                    # 规范文档 (submodule)
├── JavaTestSkeleton/          # 测试框架代码
│   ├── pom.xml               # Maven 配置
│   └── src/test/java/...     # 测试代码
├── projects/                  # 被测 Java 项目（全部为 submodule）
│   ├── ProjectA/
│   └── ProjectB/
└── README.md
```

#### 必须满足的条件

1. **所有 Java 项目必须作为 submodule 加入 `projects/` 目录**
2. **测试框架通过 `build-helper-maven-plugin` 引入 submodule 源码**
3. **测试项目的 `pom.xml` 必须配置 submodule 的源码路径**
4. **禁止直接复制或嵌套其他项目代码，必须使用 submodule**
5. **测试框架必须遵守 JavaTestSkeleton/README.md 中的规范**

#### 为什么

- 统一测试环境和测试框架
- 保持被测项目的独立版本管理
- 测试项目始终引用最新的被测代码
- CI/CD 自动拉取所有依赖的被测项目
- 避免代码重复和版本不一致
- 确保所有项目符合相同的质量标准

#### pom.xml 配置示例

```xml
<!-- 引入 submodule 源码到编译路径 -->
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <version>3.3.0</version>
    <executions>
        <execution>
            <id>add-project-sources</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>add-source</goal>
            </goals>
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

## 1. 技术栈

### 1.1 核心技术

| 技术 | 版本 | 用途 |
|------|------|------|
| Java | 17 (LTS) | 主开发语言 |
| Apache Spark | TBD | 分布式计算引擎 |
| Maven/Gradle | Latest | 构建工具 |

### 1.2 开发工具

- **IDE：** IntelliJ IDEA (推荐) / Eclipse
- **JDK：** OpenJDK 17 或 Oracle JDK 17
- **代码格式化：** Google Java Format

---

## 2. 代码规范

### 2.1 命名规范

```java
// 类名：PascalCase
public class DataProcessor { }

// 接口：PascalCase，可选 I 前缀
public interface IDataService { }

// 方法名：camelCase，动词开头
public void processData() { }
public boolean isValid() { }
public Data getData() { }

// 变量名：camelCase
private String userName;
private int recordCount;

// 常量：UPPER_SNAKE_CASE
public static final int MAX_RETRY_COUNT = 3;
public static final String DEFAULT_ENCODING = "UTF-8";

// 包名：全小写，点分隔
package com.bigdata.processor;
```

### 2.2 注释规范

**类注释：**
```java
/**
 * 数据处理器
 * <p>
 * 负责清洗、转换和聚合原始数据。
 * 支持批处理和流处理两种模式。
 * </p>
 * 
 * @author xty
 * @version 1.0
 * @since 2026-06-16
 */
public class DataProcessor {
}
```

**方法注释：**
```java
/**
 * 处理原始数据
 * 
 * @param rawData 原始数据集
 * @param options 处理选项
 * @return 处理后的数据集
 * @throws DataException 当数据格式不正确时
 * @throws IllegalArgumentException 当参数为空时
 */
public Dataset<Row> processData(Dataset<Row> rawData, ProcessOptions options) 
    throws DataException {
    // 实现逻辑
}
```

**行内注释：**
```java
// 过滤掉空值记录
Dataset<Row> filtered = rawData.filter(col("value").isNotNull());

// TODO: 优化性能 - 考虑使用分区策略
// FIXME: 修复边界情况处理
// NOTE: 这里使用了缓存以提高重复访问性能
```

### 2.3 代码结构

**标准类结构顺序：**
```java
public class Example {
    // 1. 常量
    private static final int MAX_SIZE = 100;
    
    // 2. 静态变量
    private static int instanceCount = 0;
    
    // 3. 实例变量
    private String name;
    private int value;
    
    // 4. 构造方法
    public Example() { }
    public Example(String name) { }
    
    // 5. 静态方法
    public static Example create() { }
    
    // 6. 公共方法
    public void process() { }
    
    // 7. 包私有方法
    void internalProcess() { }
    
    // 8. 保护方法
    protected void templateMethod() { }
    
    // 9. 私有方法
    private void helper() { }
    
    // 10. Getter/Setter
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    // 11. 内部类
    private static class Helper { }
}
```

---

## 3. Spark 开发规范

### 3.1 SparkSession 管理

```java
/**
 * Spark 会话单例
 */
public class SparkSessionManager {
    private static volatile SparkSession instance;
    
    private SparkSessionManager() { }
    
    /**
     * 获取 SparkSession 实例
     * 
     * @return SparkSession 单例
     */
    public static SparkSession getInstance() {
        if (instance == null) {
            synchronized (SparkSessionManager.class) {
                if (instance == null) {
                    instance = SparkSession.builder()
                        .appName("BigDataProject")
                        .master("local[*]") // 开发环境
                        .config("spark.sql.warehouse.dir", "/tmp/spark-warehouse")
                        .getOrCreate();
                }
            }
        }
        return instance;
    }
    
    /**
     * 关闭 SparkSession
     */
    public static void close() {
        if (instance != null) {
            instance.close();
            instance = null;
        }
    }
}
```

### 3.2 DataFrame/Dataset 操作规范

```java
// ✅ 好的实践：链式调用，每个操作一行
Dataset<Row> result = spark.read()
    .format("csv")
    .option("header", "true")
    .load("data/input.csv")
    .filter(col("age").gt(18))
    .groupBy("city")
    .agg(avg("salary").as("avg_salary"))
    .orderBy(desc("avg_salary"))
    .limit(10);

// ✅ 复杂转换：提取为独立方法
Dataset<Row> processed = processData(rawData);

// ✅ 使用缓存避免重复计算
Dataset<Row> cached = expensiveComputation.cache();
cached.count(); // 触发缓存
cached.show();  // 使用缓存

// ❌ 避免：过长的单行链式调用
Dataset<Row> bad = data.filter(...).groupBy(...).agg(...).join(...).select(...);
```

### 3.3 性能优化

**分区策略：**
```java
// 数据倾斜处理
Dataset<Row> repartitioned = data
    .repartition(200, col("key"))  // 按 key 重新分区
    .cache();

// 减少分区数（合并小文件）
Dataset<Row> coalesced = data.coalesce(10);
```

**广播变量：**
```java
// 小表广播 Join
Broadcast<Map<String, String>> broadcastMap = 
    spark.sparkContext().broadcast(lookupMap, scala.reflect.ClassTag$.MODULE$.apply(Map.class));

Dataset<Row> joined = largeDataset.map(
    (MapFunction<Row, Row>) row -> {
        String key = row.getString(0);
        String value = broadcastMap.value().get(key);
        return RowFactory.create(key, value);
    },
    RowEncoder.apply(schema)
);
```

---

## 4. 异常处理

### 4.1 异常层次

```java
// 自定义业务异常基类
public class DataProcessException extends Exception {
    public DataProcessException(String message) {
        super(message);
    }
    
    public DataProcessException(String message, Throwable cause) {
        super(message, cause);
    }
}

// 具体异常
public class DataValidationException extends DataProcessException { }
public class DataTransformException extends DataProcessException { }
```

### 4.2 异常处理规范

```java
// ✅ 好的实践：具体异常，清晰消息
public void processFile(String filePath) throws DataProcessException {
    try {
        Dataset<Row> data = spark.read().csv(filePath);
        validate(data);
    } catch (AnalysisException e) {
        throw new DataProcessException(
            String.format("无法读取文件: %s", filePath), e);
    } catch (IllegalArgumentException e) {
        throw new DataValidationException(
            String.format("数据验证失败: %s", e.getMessage()), e);
    }
}

// ✅ 资源管理：使用 try-with-resources
try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
    String line;
    while ((line = reader.readLine()) != null) {
        process(line);
    }
} catch (IOException e) {
    logger.error("文件读取失败", e);
    throw new DataProcessException("处理失败", e);
}

// ❌ 避免：吞掉异常
try {
    riskyOperation();
} catch (Exception e) {
    // 什么都不做 - 不好！
}

// ❌ 避免：泛型异常
public void process() throws Exception { } // 太宽泛
```

---

## 5. 日志规范

### 5.1 日志级别

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class DataProcessor {
    private static final Logger logger = LoggerFactory.getLogger(DataProcessor.class);
    
    public void process(Dataset<Row> data) {
        // TRACE: 最详细的信息
        logger.trace("开始处理数据，分区数: {}", data.rdd().getNumPartitions());
        
        // DEBUG: 调试信息
        logger.debug("数据行数: {}", data.count());
        
        // INFO: 关键流程节点
        logger.info("数据处理完成，输出记录数: {}", result.count());
        
        // WARN: 警告（不影响运行）
        logger.warn("发现 {} 条重复记录", duplicateCount);
        
        // ERROR: 错误（需要关注）
        logger.error("数据验证失败: {}", e.getMessage(), e);
    }
}
```

### 5.2 日志消息规范

```java
// ✅ 好的实践：使用占位符，避免字符串拼接
logger.info("处理文件 {} 完成，耗时 {} ms", fileName, duration);

// ✅ 记录异常堆栈
logger.error("处理失败", exception);

// ✅ 结构化信息
logger.info("任务统计 - 总数: {}, 成功: {}, 失败: {}", 
    total, success, failure);

// ❌ 避免：字符串拼接（影响性能）
logger.info("处理文件 " + fileName + " 完成"); // 不好

// ❌ 避免：日志信息过于简单
logger.error("失败"); // 太简略
```

---

## 6. 测试规范

### 6.1 单元测试

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.AfterEach;
import static org.junit.jupiter.api.Assertions.*;

/**
 * 数据处理器测试
 */
class DataProcessorTest {
    private DataProcessor processor;
    private SparkSession spark;
    
    @BeforeEach
    void setUp() {
        // 创建测试用 SparkSession
        spark = SparkSession.builder()
            .appName("test")
            .master("local[2]")
            .getOrCreate();
        processor = new DataProcessor(spark);
    }
    
    @AfterEach
    void tearDown() {
        if (spark != null) {
            spark.close();
        }
    }
    
    @Test
    void testProcessData_正常情况() {
        // Given: 准备测试数据
        Dataset<Row> input = createTestData();
        
        // When: 执行操作
        Dataset<Row> result = processor.process(input);
        
        // Then: 验证结果
        assertEquals(10, result.count(), "应该有 10 条记录");
        assertTrue(result.columns().length > 0, "应该有列");
    }
    
    @Test
    void testProcessData_空数据() {
        // Given
        Dataset<Row> empty = spark.emptyDataFrame();
        
        // When & Then
        assertThrows(DataValidationException.class, () -> {
            processor.process(empty);
        }, "空数据应该抛出验证异常");
    }
    
    private Dataset<Row> createTestData() {
        // 创建测试数据集
        return spark.read().json("src/test/resources/test-data.json");
    }
}
```

### 6.2 测试覆盖率要求

- 单元测试覆盖率：**≥ 70%**
- 核心业务逻辑：**≥ 90%**
- 所有公共 API 必须有测试

---

## 7. 依赖管理

### 7.1 Maven 配置示例

```xml
<project>
    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <spark.version>3.5.0</spark.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    
    <dependencies>
        <!-- Spark Core -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>${spark.version}</version>
        </dependency>
        
        <!-- Spark SQL -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>${spark.version}</version>
        </dependency>
        
        <!-- 日志 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>2.0.9</version>
        </dependency>
        
        <!-- 测试 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

---

## 8. 项目结构

### 8.1 标准后端项目结构

```
backend/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/bigdata/project/
│   │   │       ├── config/           # 配置类
│   │   │       ├── model/            # 数据模型
│   │   │       ├── processor/        # 数据处理
│   │   │       ├── service/          # 业务逻辑
│   │   │       ├── util/             # 工具类
│   │   │       └── Application.java  # 主入口
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── log4j2.xml
│   │       └── data/                 # 测试数据
│   └── test/
│       ├── java/
│       │   └── com/bigdata/project/
│       │       └── processor/
│       │           └── DataProcessorTest.java
│       └── resources/
│           └── test-data.json
├── pom.xml
└── README.md
```

---

## 9. 性能优化清单

- [ ] 避免使用 `collect()` 操作大数据集
- [ ] 合理使用 `cache()` 和 `persist()`
- [ ] 选择合适的分区策略
- [ ] 使用广播变量优化 Join
- [ ] 避免数据倾斜
- [ ] 使用列式存储格式（Parquet）
- [ ] 配置合理的并行度
- [ ] 监控 Spark UI 性能指标

---

## 10. 代码审查清单

**提交前自查：**
- [ ] 代码编译通过，无警告
- [ ] 所有测试通过
- [ ] 测试覆盖率达标
- [ ] 代码格式化（Google Java Format）
- [ ] 类和公共方法有 Javadoc
- [ ] 异常处理完整
- [ ] 日志记录合理
- [ ] 无硬编码配置
- [ ] 资源正确关闭
- [ ] 文档已同步更新

---

**文档版本：** 1.0  
**更新日期：** 2026-06-16  
**维护者：** xty
