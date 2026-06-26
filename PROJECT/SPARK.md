# SPARK.md

> Spark 流水线开发规范 - 数据处理工作流

## 1. 概述

本文档定义 Spark 大数据项目的流水线规范，包括目录结构、脚本要求和工作流程。

## 2. 技术栈版本

| 技术 | 版本 | 用途 |
|------|------|------|
| Scala | 2.12 | 主要开发语言 |
| Apache Spark | 3.5.0 | 大数据处理框架 |
| sbt | 1.9+ | Scala 构建工具 |
| Python | 3.8+ | 数据清洗脚本 |
| Kafka | 3.6+ | 消息队列（实时流处理） |
| MySQL | 8.0+ | 数据导出（JDBC） |

## 3. 目录结构规范

```
项目仓库/
├── SparkMain/                    # 源代码目录
│   ├── src/main/scala/org/example/
│   │   ├── Main.scala            # 主入口
│   │   ├── streaming/            # Spark Streaming 处理器
│   │   │   ├── RatingStreamProcessor.scala
│   │   │   └── RatingProducer.scala
│   │   ├── analysis/             # 分析任务
│   │   │   ├── AnalyzeRatings.scala
│   │   │   └── AnalyzeGenres.scala
│   │   └── utils/                # 工具类
│   │       └── MySQLExporter.scala
│   ├── build.sbt                 # sbt 构建配置
│   └── test/                     # 测试代码
├── dataset/                      # 原始数据目录（CSV 文件）
├── dataset_test/                 # 测试数据目录（轻量级）
├── cleanPy/                      # Python 清洗脚本
├── cleanPy_test/                 # 测试清洗脚本
├── output/                       # 分析结果输出
├── config/                       # 配置文件
├── main_pipeline_new.sh          # 流水线脚本
├── main_pipeline_test.sh         # 测试流水线脚本
├── start_streaming.sh            # 启动 Kafka + Spark Streaming
└── README.md
```

## 4. 流水线执行流程

```
步骤1: 运行 truncate_file.py（截断大文件）
    ↓
步骤2: 运行 cleanPy/ 下的清洗脚本
    ↓
步骤3: 运行 Spark 分析任务（spark-submit）
    ↓
步骤4: 结果输出到 output/（Parquet/CSV）
    ↓
步骤5: 导出到 MySQL（通过 JDBC）
```

## 5. 脚本规范

### 5.1 main_pipeline_new.sh

**必需功能：**
- 环境检查（spark-submit、python 命令）
- 目录检查（dataset、cleanPy）
- 错误处理（任何步骤失败立即退出）
- 进度输出（每步骤打印标题和状态）

**配置项：**
```bash
PYTHON_CMD="python3"            # Python 命令
SPARK_MASTER="local[*]"         # Spark Master
JAR_PATH="SparkMain/target/scala-2.12/sparkmain_2.12-1.0.jar"  # JAR 路径
```

### 5.2 truncate_file.py

**功能：** 将大 CSV 文件截断到指定大小

**用法：**
```bash
# 无参数：处理 dataset/ 目录下所有 CSV
python truncate_file.py

# 指定文件
python truncate_file.py input.csv -o output.csv --size 300
```

### 5.3 清洗脚本（cleanPy/）

**规范：**
- 文件名：`*.py`
- 按字母顺序执行
- 输入：`truncatedDataset/` 或 `dataset/`
- 输出：`cleanedDataset/`

### 5.4 Scala 分析任务

**目录：** `SparkMain/src/main/scala/org/example/analysis/`

**规范：**
- 文件名：`Analyze任务名称.scala`
- 必须包含 `main` 方法
- 支持命令行参数（输入路径、输出路径）
- 输出格式：Parquet 或 CSV
- 支持 MySQL 导出（通过环境变量配置）

**模板：**
```scala
package org.example.analysis

import org.apache.spark.sql.{SparkSession, SaveMode}
import org.example.utils.MySQLExporter

object AnalyzeXxx {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
      .appName("AnalyzeXxx")
      .getOrCreate()

    // 读取数据
    val data = spark.read.parquet("output/ratings_streaming")

    // 分析逻辑
    val result = data.groupBy("column").agg(...)

    // 输出结果
    result.write.mode("overwrite").parquet("output/xxx_stats")

    // 导出到 MySQL（可选）
    val jdbcUrl = sys.env.getOrElse("MYSQL_JDBC_URL", "")
    if (jdbcUrl.nonEmpty) {
      val props = MySQLExporter.createProperties(
        sys.env.getOrElse("MYSQL_USER", ""),
        sys.env.getOrElse("MYSQL_PASSWORD", "")
      )
      MySQLExporter.exportToMySQL(result, "xxx_stats", jdbcUrl, props)
    }

    spark.stop()
  }
}
```

## 6. MySQL 导出规范

### 6.1 环境变量配置

```bash
export MYSQL_JDBC_URL="jdbc:mysql://localhost:3306/bigdata_ana"
export MYSQL_USER="root"
export MYSQL_PASSWORD="password"
```

### 6.2 JDBC URL 格式

```
jdbc:mysql://host:port/database?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
```

### 6.3 表命名规范

- 分析结果表：`任务名_stats`
- 示例：`movie_stats`, `genre_stats`, `user_stats`

## 7. 开发工作流

### 7.1 新建分析任务

1. 在 `SparkMain/src/main/scala/org/example/analysis/` 创建 Scala 文件
2. 遵循命名规范：`Analyze任务名称.scala`
3. 实现 `main` 方法
4. 在 `Main.scala` 中注册任务
5. 本地测试通过后提交

### 7.2 运行流水线

```bash
# 构建项目
cd SparkMain
sbt package

# 运行流水线
chmod +x main_pipeline_new.sh
./main_pipeline_new.sh
```

### 7.3 运行单个任务

```bash
spark-submit \
  --class org.example.analysis.AnalyzeRatings \
  --master local[*] \
  SparkMain/target/scala-2.12/sparkmain_2.12-1.0.jar
```

## 8. CI/CD 规范

### 8.1 CI 流程

1. **静态检查：** `sbt compile`
2. **单元测试：** `sbt test`
3. **构建 JAR：** `sbt package`
4. **集成测试：** 使用测试数据运行流水线
5. **发布：** 测试通过后发布到 Releases

### 8.2 发布产物

- **Releases：** 完整项目压缩包（含 JAR，不含源码）
- **GitHub Packages：** 任务 JAR 文件

## 9. 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| `sbt: command not found` | sbt 未安装 | 安装 sbt 并配置环境变量 |
| `spark-submit: command not found` | Spark 未安装 | 安装 Spark 并配置环境变量 |
| `python: command not found` | Python 未安装 | 安装 Python 3.8+ |
| MySQL 连接失败 | JDBC 配置错误 | 检查环境变量配置 |
| JAR 文件不存在 | 未构建项目 | 运行 `sbt package` |

## 10. 参考

- [Apache Spark 官方文档](https://spark.apache.org/docs/3.5.0/)
- [Scala 官方文档](https://www.scala-lang.org/documentation/)
- [sbt 官方文档](https://www.scala-sbt.org/1.x/docs/)
- [PROJECT.md](PROJECT.md) - 全项目通用规范

---

**文档版本：** 2.0  
**更新日期：** 2026-06-26  
**维护者：** yiyangchen609-web
