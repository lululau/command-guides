# Miller 数据格式详解与应用指南

## 概述

Miller 支持多种数据格式，每种格式都有其特定的用途和优势。理解这些格式的特点和适用场景，对于有效使用 Miller 至关重要。

## 键值数据模型

与传统的基于位置的 Unix 工具不同，Miller 基于**键值对**数据模型工作。这意味着：

- **传统工具**：操作第1列、第2列等（基于位置）
- **Miller**：操作名为 "name"、"age"、"score" 等的字段（基于名称）

这种设计使得 Miller 在处理结构化数据时更加直观和强大。

## 支持的数据格式详解

### 1. CSV（逗号分隔值）

**特点：**
- 最常见的结构化数据格式
- 第一行通常是字段名（头部）
- 数据行包含对应的值

**示例：**
```csv
name,age,city
Alice,25,Beijing
Bob,30,Shanghai
Charlie,35,Guangzhou
```

**使用场景：**
- 电子表格数据导出
- 数据库查询结果
- 数据交换的标准格式

**Miller 使用示例：**
```bash
# 读取 CSV 并转换为美观表格
mlr --csv --opprint cat data.csv

# 过滤年龄大于 30 的记录
mlr --csv filter '$age > 30' data.csv

# 按城市分组计算平均年龄
mlr --csv stats1 -a mean -f age -g city data.csv
```

### 2. JSON（JavaScript Object Notation）

**特点：**
- 支持嵌套结构（对象和数组）
- 类型信息保留（字符串、数字、布尔值、null）
- 层次化数据表示

**示例：**
```json
[
  {
    "name": "Alice",
    "age": 25,
    "address": {
      "city": "Beijing",
      "district": "Chaoyang"
    },
    "hobbies": ["reading", "swimming"]
  },
  {
    "name": "Bob",
    "age": 30,
    "address": {
      "city": "Shanghai",
      "district": "Pudong"
    },
    "hobbies": ["coding", "gaming", "music"]
  }
]
```

**Miller 中的处理：**
- 嵌套对象展开：`address.city`, `address.district`
- 数组索引：`hobbies.1`, `hobbies.2`, `hobbies.3`

**使用示例：**
```bash
# JSON 转 CSV（自动展开嵌套结构）
mlr --ijson --ocsv cat data.json

# 访问嵌套字段
mlr --ijson filter '$address.city == "Beijing"' data.json

# 处理数组字段
mlr --ijson put '$hobby_count = length($hobbies)' data.json
```

### 3. JSON Lines (JSONL)

**特点：**
- 每行一个独立的 JSON 对象
- 流式处理友好
- 日志文件常用格式

**示例：**
```jsonl
{"name": "Alice", "age": 25, "city": "Beijing"}
{"name": "Bob", "age": 30, "city": "Shanghai"}
{"name": "Charlie", "age": 35, "city": "Guangzhou"}
```

**适用场景：**
- 日志文件处理
- 大数据流处理
- API 响应处理

### 4. DKVP（分隔键值对）

**特点：**
- Miller 的默认格式
- 明确显示键值关系
- 易于调试和理解

**示例：**
```
name=Alice,age=25,city=Beijing
name=Bob,age=30,city=Shanghai
name=Charlie,age=35,city=Guangzhou
```

**优势：**
- 自描述性强
- 调试友好
- 字段顺序灵活

### 5. TSV（制表符分隔值）

**特点：**
- 类似 CSV，但使用制表符分隔
- 更适合包含逗号的数据
- Unix 工具友好

**示例：**
```tsv
name	age	city
Alice	25	Beijing
Bob	30	Shanghai
Charlie	35	Guangzhou
```

### 6. PPRINT（美观打印）

**特点：**
- 人类可读的表格格式
- 自动列对齐
- 适合终端显示

**示例：**
```
name    age city
Alice   25  Beijing
Bob     30  Shanghai
Charlie 35  Guangzhou
```

### 7. XTAB（交叉表格）

**特点：**
- 转置的键值对显示
- 适合少量记录的详细查看
- 每个记录垂直显示

**示例：**
```
name  Alice
age   25
city  Beijing

name  Bob
age   30
city  Shanghai
```

**使用场景：**
- 详细记录检查
- 调试数据问题
- 少量记录的深度分析

### 8. NIDX（数字索引）

**特点：**
- 传统 Unix 工具风格
- 基于位置的字段访问
- 与 awk、cut 等工具兼容

**示例：**
```
Alice 25 Beijing
Bob 30 Shanghai
Charlie 35 Guangzhou
```

## 格式转换策略

### 常见转换模式

1. **数据清洗流程：**
   ```bash
   # CSV → 清洗 → JSON
   mlr --icsv --ojson \
       clean-whitespace then \
       filter '$age >= 18' then \
       put '$age_group = $age < 30 ? "young" : "mature"' \
       data.csv
   ```

2. **日志分析流程：**
   ```bash
   # JSONL → 分析 → CSV 报告
   mlr --ijsonl --ocsv \
       filter '$level == "ERROR"' then \
       stats1 -a count -g service,timestamp \
       logs.jsonl
   ```

3. **数据探索流程：**
   ```bash
   # 任意格式 → PPRINT 查看
   mlr --icsv --opprint head -n 5 data.csv
   mlr --ijson --opprint summary data.json
   ```

### 格式选择指南

| 用途 | 推荐格式 | 原因 |
|------|----------|------|
| 数据存储 | CSV/JSON | 标准化、兼容性好 |
| 数据处理 | DKVP | Miller 原生，调试友好 |
| 数据展示 | PPRINT | 人类可读 |
| 数据传输 | JSON/JSONL | 类型保留、结构化 |
| 日志处理 | JSONL | 流式友好 |
| 调试分析 | XTAB | 详细显示 |

## 高级格式处理技巧

### 1. 处理嵌套 JSON

```bash
# 控制展开行为
mlr --ijson --ocsv --jflatsep ":" cat nested.json

# 禁用自动展开
mlr --ijson --ocsv --no-auto-flatten cat complex.json
```

### 2. 处理不规则 CSV

```bash
# 处理参差不齐的 CSV
mlr --csv --allow-ragged-csv-input cat messy.csv

# 清理前导空格
mlr --csv --csv-trim-leading-space cat spaced.csv
```

### 3. 无头部数据处理

```bash
# 为无头部数据添加标签
mlr --csv --implicit-csv-header label name,age,city data.csv

# 生成数字索引标签
mlr --nidx --opprint cat data.txt
```

### 4. 格式组合使用

```bash
# 多格式输入，统一输出
mlr --icsv cat file1.csv --ijson cat file2.json --opprint
```

## 最佳实践

### 1. 格式选择原则
- **输入格式**：根据数据源选择
- **处理格式**：使用 DKVP 或保持原格式
- **输出格式**：根据目标用途选择

### 2. 性能考虑
- JSON 解析较慢，但功能强大
- CSV 处理快速，适合大数据
- DKVP 调试友好，中等性能

### 3. 兼容性考虑
- CSV 兼容性最好
- JSON 现代应用友好
- NIDX 与传统 Unix 工具兼容

### 4. 调试建议
- 使用 PPRINT 查看数据结构
- 使用 XTAB 检查问题记录
- 使用 `--opprint head -n 5` 快速预览

## 实际应用场景

### 场景1：日志分析
```bash
# 分析 nginx 访问日志（JSON 格式）
mlr --ijsonl --opprint \
    filter '$status >= 400' then \
    stats1 -a count,mode -f status -g remote_addr then \
    sort -nr count \
    access.log
```

### 场景2：数据清洗
```bash
# 清洗用户数据
mlr --icsv --ocsv \
    clean-whitespace then \
    filter '$email =~ ".*@.*\\..*"' then \
    put '$age = int($age)' then \
    filter '$age >= 18 && $age <= 120' \
    users.csv > clean_users.csv
```

### 场景3：报告生成
```bash
# 生成销售报告
mlr --icsv --opprint \
    put '$total = $quantity * $price' then \
    stats1 -a sum,mean,count -f total -g region then \
    sort -nr total_sum \
    sales.csv
```

### 场景4：数据转换
```bash
# 将多个 CSV 文件合并为 JSON
mlr --icsv --ojson cat *.csv > combined.json

# 将 JSON API 响应转换为 CSV 报告
curl api.example.com/data | \
mlr --ijsonl --ocsv \
    flatten then \
    cut -f id,name,created_at,stats.total
```

这种灵活的格式支持使得 Miller 成为数据处理的瑞士军刀，能够轻松处理各种数据源和输出需求。 