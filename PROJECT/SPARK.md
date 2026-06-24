# SPARK.md

> Spark 流水线开发规范 - 数据处理工作流

## 1. 概述

本文档定义 Spark 大数据项目的流水线规范，包括目录结构、脚本要求和工作流程。

## 2. 技术栈版本

| 技术 | 版本 | 用途 |
|------|------|------|
| Java | 17 (JDK 17 LTS) | 主要开发语言 |
| Apache Spark | 3.5.0 | 大数据处理框架 |
| Scala | 2.12 | Spark 依赖二进制版本 |
| Python | 3.8+ | 数据清洗脚本 |
| Hive | 3.1.x | 数据仓库 |
| HDFS | Hadoop 3.x | 分布式存储 |

## 3. 目录结构规范

```
项目仓库/
├── dataset/                  # 原始数据目录（CSV 文件）
├── truncatedDataset/         # 截断后数据（自动生成）
├── cleanedDataset/           # 清洗后数据（自动生成）
├── cleanPy/                  # Python 清洗脚本
│   └── *.py
├── initializeSQL/            # Hive 建表 SQL
│   └── *.sql
├── prepareData/              # 数据准备 SQL（Hive 查询）
│   └── *.sql
├── jobSQL/                   # 分析任务 SQL
│   └── *.sql
├── main_pipeline.sh          # Linux 流水线脚本
├── main_pipeline.bat         # Windows 流水线脚本
├── truncate_file.py          # 数据截断工具
├── print_end.sh              # 结束标记脚本
└── README.md
```

## 4. 流水线执行流程

```
步骤1: 检查/创建 Hive 数据库
    ↓
步骤2: 运行 truncate_file.py（截断大文件）
    ↓
步骤3: 运行 cleanPy/ 下的清洗脚本
    ↓
步骤4: 上传 cleanedDataset/ 到 HDFS
    ↓
步骤5: 检查 Hive 表，按需执行 initializeSQL/
    ↓
步骤6: 运行 prepareData/ 下的 SQL
    ↓
步骤7: 运行 jobSQL/ 下的分析任务
    ↓
步骤8: 打印结束标记
```

## 5. 脚本规范

### 5.1 main_pipeline.sh / main_pipeline.bat

**必需功能：**
- 环境检查（hive、hdfs、python 命令）
- 目录检查（dataset、cleanPy、initializeSQL、jobSQL）
- 错误处理（任何步骤失败立即退出）
- 进度输出（每步骤打印标题和状态）

**配置项：**
```bash
HIVE_DB="bigdata_ana"           # Hive 数据库名
HDFS_BASE_PATH="/user/hive/..." # HDFS 存储路径
PYTHON_CMD="python3"            # Python 命令
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

**参数：**
- `input_file`：输入文件（可选，默认处理 dataset/）
- `-o, --output`：输出文件
- `--size`：目标大小 MB（默认 300）
- `--dataset-dir`：数据集目录
- `--output-dir`：输出目录

### 5.3 清洗脚本（cleanPy/）

**规范：**
- 文件名：`*.py`
- 按字母顺序执行
- 输入：`truncatedDataset/` 或 `dataset/`
- 输出：`cleanedDataset/`

### 5.4 SQL 脚本

**initializeSQL/：**
- 建表语句
- 表名与 CSV 文件名对应（去掉 .csv 后缀）

**prepareData/：**
- 数据准备查询
- 创建中间表或视图

**jobSQL/：**
- 分析任务查询
- 结果输出到 Hive 表

## 6. Spark + Hive 配置规范

### 6.1 执行引擎配置

**所有 SQL 文件必须在开头配置 Hive on Spark：**

```sql
-- 设置 Hive 执行引擎为 Spark
SET hive.execution.engine=spark;
SET spark.master=local[*];
```

### 6.2 配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `hive.execution.engine` | `spark` | 使用 Spark 作为执行引擎 |
| `spark.master` | `local[*]` | 本地模式，使用所有 CPU 核心 |

### 6.3 生产环境配置

**集群环境配置：**
```sql
SET hive.execution.engine=spark;
SET spark.master=yarn;
SET spark.deploy.mode=cluster;
```

### 6.4 执行流程

```
hive -f xxx.sql
    ↓
Hive 解析 SQL
    ↓
提交到 Spark 执行
    ↓
返回结果
```

### 6.5 环境要求

| 组件 | 版本 | 说明 |
|------|------|------|
| Hive | 3.1.x | 数据仓库，支持 Spark 引擎 |
| Spark | 3.5.0 | 大数据处理框架 |
| Hadoop | 3.x | 分布式存储（HDFS） |

### 6.6 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| Spark 引擎未配置 | 缺少 `SET hive.execution.engine=spark` | 在 SQL 文件开头添加配置 |
| Spark 连接失败 | Spark 服务未启动 | 启动 Spark Master 和 Worker |
| 内存不足 | 数据量过大 | 调整 `spark.executor.memory` |

## 7. 开发工作流

### 6.1 新建项目

1. 克隆仓库结构
2. 配置 `.gitignore`（忽略 truncatedDataset/、cleanedDataset/）
3. 将数据文件放入 `dataset/`
4. 编写清洗脚本放入 `cleanPy/`
5. 编写建表 SQL 放入 `initializeSQL/`
6. 编写分析 SQL 放入 `jobSQL/`

### 6.2 运行流水线

**Linux/Mac：**
```bash
chmod +x main_pipeline.sh
./main_pipeline.sh
```

**Windows：**
```cmd
main_pipeline.bat
```

### 6.3 调试

- 检查每个步骤的输出日志
- 验证 HDFS 文件是否上传成功
- 验证 Hive 表是否创建成功

## 7. 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| `hive: command not found` | Hive 未安装或未配置 PATH | 安装 Hive 并配置环境变量 |
| `hdfs: command not found` | Hadoop 未安装 | 安装 Hadoop 并配置环境变量 |
| `python: command not found` | Python 未安装 | 安装 Python 3.8+ |
| CSV 文件为空 | 数据清洗脚本输出异常 | 检查 cleanPy/ 脚本逻辑 |
| Hive 表创建失败 | SQL 语法错误 | 检查 initializeSQL/ 中的 SQL |

## 8. 参考

- [Apache Spark 官方文档](https://spark.apache.org/docs/3.5.0/)
- [Apache Hive 官方文档](https://hive.apache.org/)
- [PROJECT.md](PROJECT.md) - 全项目通用规范
- [BACKEND.md](BACKEND.md) - 后端开发规范

---

**文档版本：** 1.1  
**更新日期：** 2026-06-24  
**维护者：** yiyangchen609-web
