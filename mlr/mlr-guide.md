# Miller (mlr) 使用指南

## 名称

Miller -- 类似于 awk、sed、cut、join 和 sort 的工具，专门用于处理命名索引数据，如 CSV 和表格化 JSON。

## 概要

**使用方法：** `mlr [标志] {动词} [动词相关选项 ...] {零个或多个文件名}`

如果未提供文件名，则从标准输入读取数据，例如：
```bash
mlr --csv sort -f shape example.csv
```

可以使用 "then" 将一个动词的输出链接为另一个动词的输入，例如：
```bash
mlr --csv stats1 -a min,mean,max -f quantity then sort -f color example.csv
```

请参阅 `mlr help topics` 获取更多信息。另请参阅：https://miller.readthedocs.io

## 描述

Miller 操作键值对数据，而熟悉的 Unix 工具操作整数索引字段：如果后者的自然数据结构是数组，那么 Miller 的自然数据结构就是插入有序的哈希映射。这涵盖了各种数据格式，包括但不限于熟悉的 CSV、TSV 和 JSON。（Miller 可以将位置索引数据作为特殊情况处理。）本手册页记录了 mlr 6.14.0 版本。

## 基本示例

```bash
# CSV 转换为美观的表格格式
mlr --icsv --opprint cat example.csv

# 按 shape 字段排序
mlr --icsv --opprint sort -f shape example.csv

# 按 shape 字段排序，同时按 index 字段数字降序排序
mlr --icsv --opprint sort -f shape -nr index example.csv

# 仅保留 flag 和 shape 字段
mlr --icsv --opprint cut -f flag,shape example.csv

# 过滤出 color 字段为 "red" 的记录
mlr --csv filter '$color == "red"' example.csv

# 计算比率并输出为 JSON
mlr --icsv --ojson put '$ratio = $quantity / $rate' example.csv

# 组合操作：先按 index 降序排序，然后仅保留 shape 和 quantity 字段
mlr --icsv --opprint --from example.csv sort -nr index then cut -f shape,quantity
```

## 支持的文件格式

### CSV/CSV-lite：逗号分隔值，带单独的头行
### TSV：制表符分隔值
```
+---------------------+
| apple,bat,cog       |
| 1,2,3               | 记录 1: "apple":"1", "bat":"2", "cog":"3"
| 4,5,6               | 记录 2: "apple":"4", "bat":"5", "cog":"6"
+---------------------+
```

### JSON（对象数组）
```json
[
  {
    "apple": 1,         | 记录 1: "apple":"1", "bat":"2", "cog":"3"
    "bat": 2,
    "cog": 3
  },
  {
    "dish": {           | 记录 2: "dish.egg":"7",
      "egg": 7,         | "dish.flint":"8", "garlic":""
      "flint": 8
    },
    "garlic": ""
  }
]
```

### JSON Lines（一行一个对象的序列）
```
{"apple": 1, "bat": 2, "cog": 3}
{"dish": {"egg": 7, "flint": 8}, "garlic": ""}
```
记录 1: "apple":"1", "bat":"2", "cog":"3"
记录 2: "dish:egg":"7", "dish:flint":"8", "garlic":""

### PPRINT：美观的表格格式
```
+---------------------+
| apple bat cog       |
| 1     2   3         | 记录 1: "apple":"1", "bat":"2", "cog":"3"
| 4     5   6         | 记录 2: "apple":"4", "bat":"5", "cog":"6"
+---------------------+
```

### Markdown 表格格式
```
| apple | bat | cog |
| ---   | --- | --- |
| 1     | 2   | 3   | 记录 1: "apple":"1", "bat":"2", "cog":"3"
| 4     | 5   | 6   | 记录 2: "apple":"4", "bat":"5", "cog":"6"
```

### XTAB：美观的转置表格格式
```
+---------------------+
| apple 1             | 记录 1: "apple":"1", "bat":"2", "cog":"3"
| bat   2             |
| cog   3             |
|                     |
| dish 7              | 记录 2: "dish":"7", "egg":"8"
| egg  8              |
+---------------------+
```

### DKVP：分隔的键值对（Miller 默认格式）
```
+---------------------+
| apple=1,bat=2,cog=3 | 记录 1: "apple":"1", "bat":"2", "cog":"3"
| dish=7,egg=8,flint  | 记录 2: "dish":"7", "egg":"8", "3":"flint"
+---------------------+
```

### NIDX：隐式数字索引（Unix 工具风格）
```
+---------------------+
| the quick brown     | 记录 1: "1":"the", "2":"quick", "3":"brown"
| fox jumped          | 记录 2: "1":"fox", "2":"jumped"
+---------------------+
```

## 帮助选项

使用 `mlr help {主题}` 可以获取以下任何主题的帮助：

### 基础主题
- `mlr help topics` - 所有帮助主题
- `mlr help basic-examples` - 基本示例
- `mlr help file-formats` - 文件格式

### 标志选项
- `mlr help flags` - 所有标志
- `mlr help flag` - 特定标志
- `mlr help list-separator-aliases` - 分隔符别名列表
- `mlr help list-separator-regex-aliases` - 分隔符正则表达式别名列表
- `mlr help comments-in-data-flags` - 数据中注释的标志
- `mlr help compressed-data-flags` - 压缩数据标志
- `mlr help csv/tsv-only-flags` - CSV/TSV 专用标志
- `mlr help file-format-flags` - 文件格式标志
- `mlr help flatten-unflatten-flags` - 展开/收缩标志
- `mlr help format-conversion-keystroke-saver-flags` - 格式转换快捷标志
- `mlr help json-only-flags` - JSON 专用标志
- `mlr help legacy-flags` - 遗留标志
- `mlr help miscellaneous-flags` - 杂项标志
- `mlr help output-colorization-flags` - 输出颜色化标志
- `mlr help pprint-only-flags` - PPRINT 专用标志
- `mlr help profiling-flags` - 性能分析标志
- `mlr help separator-flags` - 分隔符标志

### 动词
- `mlr help list-verbs` - 动词列表
- `mlr help usage-verbs` - 动词使用方法
- `mlr help verb` - 特定动词

### 函数
- `mlr help list-functions` - 函数列表
- `mlr help list-function-classes` - 函数类别列表
- `mlr help list-functions-in-class` - 按类别列出函数
- `mlr help usage-functions` - 函数使用方法
- `mlr help usage-functions-by-class` - 按类别的函数使用方法
- `mlr help function` - 特定函数

### 关键字
- `mlr help list-keywords` - 关键字列表
- `mlr help usage-keywords` - 关键字使用方法
- `mlr help keyword` - 特定关键字

### 其他
- `mlr help auxents` - 辅助命令
- `mlr help terminals` - 终端相关
- `mlr help mlrrc` - 配置文件
- `mlr help output-colorization` - 输出颜色化
- `mlr help type-arithmetic-info` - 类型算术信息
- `mlr help type-arithmetic-info-extended` - 扩展类型算术信息

### 快捷方式
- `mlr -g` = `mlr help flags`
- `mlr -l` = `mlr help list-verbs`
- `mlr -L` = `mlr help usage-verbs`
- `mlr -f` = `mlr help list-functions`
- `mlr -F` = `mlr help usage-functions`
- `mlr -k` = `mlr help list-keywords`
- `mlr -K` = `mlr help usage-keywords`

**注意：** `mlr help ...` 将使用 flag、verb、function 和 keyword 的来源搜索您的确切文本 "..."。使用 `mlr help find ...` 进行近似（子字符串）匹配，例如 `mlr help find map` 查找所有名称中包含 "map" 的内容。

## 动词列表

Miller 提供以下动词（操作）：

**数据操作动词：**
- `altkv` - 交替键值处理
- `bar` - 创建条形图
- `bootstrap` - 引导抽样
- `case` - 大小写转换
- `cat` - 连接和显示记录
- `check` - 检查数据格式
- `clean-whitespace` - 清理空白字符
- `count-distinct` - 计算不同值的数量
- `count` - 计数记录
- `count-similar` - 计算相似记录

**数据选择和过滤：**
- `cut` - 选择字段
- `decimate` - 抽取记录
- `fill-down` - 向下填充
- `fill-empty` - 填充空值
- `filter` - 过滤记录
- `grep` - 模式匹配
- `having-fields` - 基于字段名的条件过滤
- `head` - 显示前 N 条记录
- `tail` - 显示后 N 条记录

**数据转换：**
- `flatten` - 展开嵌套结构
- `format-values` - 格式化值
- `json-parse` - 解析 JSON
- `json-stringify` - 序列化为 JSON
- `put` - 计算和添加字段
- `rename` - 重命名字段
- `reorder` - 重新排序字段

**数据分组和聚合：**
- `fraction` - 计算比例
- `gap` - 插入间隔记录
- `group-by` - 按字段分组
- `group-like` - 按相似性分组
- `histogram` - 创建直方图
- `join` - 连接数据
- `nest` - 嵌套/解嵌套数据
- `stats1` - 单变量统计
- `stats2` - 双变量统计
- `step` - 步进计算

**数据排序和采样：**
- `sort` - 排序记录
- `sort-within-records` - 记录内排序
- `shuffle` - 随机排列
- `sample` - 抽样
- `top` - 获取最高值
- `bootstrap` - 有放回抽样

**数据重构：**
- `reshape` - 重塑数据（宽表转长表）
- `split` - 分割数据
- `unflatten` - 还原展开的结构
- `uniq` - 去重
- `regularize` - 规范化

**字符串操作：**
- `gsub` - 全局替换
- `ssub` - 简单替换
- `sub` - 替换
- `latin1-to-utf8` - 编码转换
- `utf8-to-latin1` - 编码转换

**时间处理：**
- `sec2gmtdate` - 秒转换为 GMT 日期
- `sec2gmt` - 秒转换为 GMT 时间

**其他工具：**
- `label` - 标记字段
- `merge-fields` - 合并字段
- `most-frequent` - 最频繁值
- `least-frequent` - 最不频繁值
- `nothing` - 空操作
- `remove-empty-columns` - 删除空列
- `repeat` - 重复记录
- `seqgen` - 生成序列
- `skip-trivial-records` - 跳过琐碎记录
- `sparsify` - 稀疏化
- `summary` - 数据摘要
- `surv` - 生存分析
- `tac` - 反向显示
- `tee` - 分支输出
- `template` - 模板化
- `unspace` - 删除空格
- `unsparsify` - 反稀疏化

## 函数列表

Miller 提供丰富的内置函数，按功能分类：

### 数学函数
**基础数学：** `abs`, `ceil`, `floor`, `round`, `roundm`, `sgn`, `truncate`
**三角函数：** `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2`, `sinh`, `cosh`, `tanh`, `asinh`, `acosh`, `atanh`
**指数和对数：** `exp`, `expm1`, `log`, `log10`, `log1p`, `pow`, `sqrt`, `cbrt`
**统计函数：** `mean`, `median`, `mode`, `antimode`, `min`, `max`, `sum`, `count`, `stddev`, `variance`, `skewness`, `kurtosis`

### 字符串函数
**字符串操作：** `concat`, `contains`, `length`, `substr`, `substr0`, `substr1`
**大小写转换：** `tolower`, `toupper`, `capitalize`
**填充和修剪：** `leftpad`, `rightpad`, `lstrip`, `rstrip`, `strip`
**模式匹配：** `gsub`, `ssub`, `sub`, `regextract`, `strmatch`
**格式化：** `format`, `sprintf`, `fmtnum`, `fmtifnum`

### 类型检查和转换
**类型检查：** `typeof`, `is_array`, `is_bool`, `is_empty`, `is_error`, `is_float`, `is_int`, `is_map`, `is_null`, `is_numeric`, `is_present`, `is_string`
**类型转换：** `boolean`, `float`, `int`, `string`
**类型断言：** `asserting_*` 系列函数

### 时间和日期函数
**时间转换：** `sec2gmt`, `sec2gmtdate`, `gmt2sec`, `localtime2gmt`, `gmt2localtime`
**时间格式：** `strftime`, `strptime`, `dhms2sec`, `sec2dhms`, `hms2sec`, `sec2hms`
**系统时间：** `systime`, `uptime`

### 数组和映射函数
**数组操作：** `arrayify`, `append`, `apply`, `every`, `sort_collection`
**映射操作：** `get_keys`, `get_values`, `haskey`, `mapsum`, `mapdiff`, `mapselect`, `mapexcept`
**JSON 处理：** `json_parse`, `json_stringify`

### 随机数和哈希
**随机数：** `urand`, `urand32`, `urandint`, `urandrange`, `urandelement`
**哈希函数：** `md5`, `sha1`, `sha256`, `sha512`

### 系统和环境函数
**系统信息：** `hostname`, `os`
**执行：** `exec`, `system`

## 标志选项详解

### 数据中注释的标志

Miller 允许您在数据中加入注释，例如：

```
# 这是 CSV 文件的注释
a,b,c
1,2,3
4,5,6
```

**注意事项：**
- 注释只在行的开头生效
- 如果没有使用下面的四个选项，注释将被视为普通文本数据（注释功能是可选的）
- 使用 `--pass-comments` 时，注释行会在读取时立即写入标准输出，不是记录流的一部分

**相关选项：**

- `--pass-comments` - 立即打印输入中的注释行（以 `#` 开头）
- `--pass-comments-with {字符串}` - 立即打印输入中的注释行，使用指定的前缀
- `--skip-comments` - 忽略输入中的注释行（以 `#` 开头）
- `--skip-comments-with {字符串}` - 忽略输入中带有指定前缀的注释行

**示例：**
```bash
# 保留注释并传递到输出
mlr --csv --pass-comments cat data.csv

# 忽略以 # 开头的注释行
mlr --csv --skip-comments cat data.csv

# 忽略以 // 开头的注释行
mlr --csv --skip-comments-with "//" cat data.csv
```

### 压缩数据标志

Miller 提供几种不同的方式来处理压缩数据文件：

**内置解压缩（在 Miller 进程内）：**
- `--bz2in` - 解压 bzip2 格式
- `--gzin` - 解压 gzip 格式
- `--zin` - 解压 zlib 格式
- `--zstdin` - 解压 zstd 格式

**外部解压缩（在 Miller 进程外）：**
- `--prepipe` - 指定预处理管道命令
- `--prepipex` - 类似 prepipe，但不在命令和文件名之间插入 `<`

**详细选项说明：**

- `--bz2in` - 在 Miller 进程内解压 bzip2 文件。如果文件以 `.bz2` 结尾，会自动启用
- `--gzin` - 在 Miller 进程内解压 gzip 文件。如果文件以 `.gz` 结尾，会自动启用
- `--zin` - 在 Miller 进程内解压 zlib 文件。如果文件以 `.z` 结尾，会自动启用
- `--zstdin` - 在 Miller 进程内解压 zstd 文件。如果文件以 `.zstd` 结尾，会自动启用

**预处理管道选项：**
- `--prepipe {解压命令}` - 对每个输入文件执行指定的操作。命令必须能从标准输入读取，调用格式为 `{命令} < {文件名}`
- `--prepipe-bz2` - 等同于 `--prepipe bz2`，但允许在 `.mlrrc` 中使用
- `--prepipe-gunzip` - 等同于 `--prepipe gunzip`，但允许在 `.mlrrc` 中使用
- `--prepipe-zcat` - 等同于 `--prepipe zcat`，但允许在 `.mlrrc` 中使用
- `--prepipe-zstdcat` - 等同于 `--prepipe zstdcat`，但允许在 `.mlrrc` 中使用
- `--prepipex {解压命令}` - 类似 `--prepipe`，但不在命令和文件名之间插入 `<`，适用于 `unzip -qc` 等不读取标准输入的命令

**示例：**
```bash
# 使用内置解压缩
mlr --gzin --csv cat data.csv.gz

# 使用外部解压缩
mlr --prepipe gunzip --csv cat data.csv.gz
mlr --prepipe "zcat -cf" --csv cat data.csv.gz
mlr --prepipe "xz -cd" --csv cat data.csv.xz

# 输出压缩，需要使用管道
mlr --csv cat data.csv | gzip > output.csv.gz
```

### CSV/TSV 专用标志

这些标志专门适用于 CSV 和 TSV 格式：

**数据处理选项：**
- `--allow-ragged-csv-input` 或 `--ragged` 或 `--allow-ragged-tsv-input` - 如果数据行的字段数少于头行，用空字符串填充剩余的键；如果数据行的字段数多于头行，使用整数字段标签
- `--csv-trim-leading-space` - 修剪 CSV 数据中的前导空格，用于处理像 `"foo", "bar"` 这样不符合 RFC-4180 但常见的数据
- `--lazy-quotes` - 接受出现在未引号字段中的引号，以及出现在引号字段中的非双引号

**头部处理选项：**
- `--headerless-csv-output` 或 `--ho` 或 `--headerless-tsv-output` - 只打印 CSV/TSV 数据行，不打印头行
- `--implicit-csv-header` 或 `--headerless-csv-input` 或 `--hi` 或 `--implicit-tsv-header` - 使用 1,2,3,... 作为字段标签，而不是从输入文件的第一行读取。提示：结合 `label` 命令重新创建缺失的头部
- `--no-implicit-csv-header` 或 `--no-implicit-tsv-header` - 与 `--implicit-csv-header` 相反，这是默认设置

**输出格式选项：**
- `--quote-all` - 强制对所有 CSV 字段使用双引号
- `--no-auto-unsparsify` - 对于 CSV/TSV 输出：如果记录键从一行变为另一行，则发出空行和新的头行（不符合 RFC 4180 但对异构数据有帮助）
- `-N` - `--implicit-csv-header --headerless-csv-output` 的快捷方式

**示例：**
```bash
# 处理没有头部的 CSV 文件
mlr --csv --implicit-csv-header cat data.csv

# 输出时不包含头部
mlr --csv --headerless-csv-output cat data.csv

# 处理不规整的 CSV 数据
mlr --csv --allow-ragged-csv-input cat messy_data.csv

# 修剪前导空格
mlr --csv --csv-trim-leading-space cat spaced_data.csv

# 强制引号所有字段
mlr --csv --quote-all cat data.csv

# 快捷方式：无头输入和输出
mlr --csv -N cat data.csv
```

### 文件格式标志

这些标志控制 Miller 如何读取和写入不同的数据格式。

**重要提示：** 请使用 `--iformat1 --oformat2` 而不是 `--format1 --oformat2`。后者会为 `format1` 设置输入和输出标志，当设置输出格式为 `format2` 时，不是所有情况下都会被覆盖。

**输入输出格式选项：**
- `--asv` 或 `--asvlite` - 输入和输出使用 ASV 格式
- `--csv` 或 `-c` 或 `--c2c` - 输入和输出使用 CSV 格式
- `--csvlite` - 输入和输出使用 CSV-lite 格式
- `--dkvp` 或 `--d2d` - 输入和输出使用 DKVP 格式
- `--json` 或 `-j` 或 `--j2j` - 输入和输出使用 JSON 格式
- `--jsonl` 或 `--l2l` - 输入和输出使用 JSON Lines 格式
- `--nidx` 或 `--n2n` - 输入和输出使用 NIDX 格式
- `--pprint` 或 `--p2p` - 输入和输出使用 PPRINT 格式
- `--tsv` 或 `-t` 或 `--t2t` - 输入和输出使用 TSV 格式
- `--tsvlite` - 输入和输出使用 TSV-lite 格式
- `--usv` 或 `--usvlite` - 输入和输出使用 USV 格式
- `--xtab` 或 `--x2x` - 输入和输出使用 XTAB 格式

**仅输入格式选项：**
- `--iasv` 或 `--iasvlite` - 输入使用 ASV 格式
- `--icsv` - 输入使用 CSV 格式
- `--icsvlite` - 输入使用 CSV-lite 格式
- `--idkvp` - 输入使用 DKVP 格式
- `--ijson` - 输入使用 JSON 格式
- `--ijsonl` - 输入使用 JSON Lines 格式
- `--imd` 或 `--imarkdown` - 输入使用 Markdown 表格格式
- `--inidx` - 输入使用 NIDX 格式
- `--ipprint` - 输入使用 PPRINT 格式
- `--itsv` - 输入使用 TSV 格式
- `--itsvlite` - 输入使用 TSV-lite 格式
- `--iusv` 或 `--iusvlite` - 输入使用 USV 格式
- `--ixtab` - 输入使用 XTAB 格式

**仅输出格式选项：**
- `--oasv` 或 `--oasvlite` - 输出使用 ASV 格式
- `--ocsv` - 输出使用 CSV 格式
- `--ocsvlite` - 输出使用 CSV-lite 格式
- `--odkvp` - 输出使用 DKVP 格式
- `--ojson` - 输出使用 JSON 格式
- `--ojsonl` - 输出使用 JSON Lines 格式
- `--omd` 或 `--omarkdown` - 输出使用 Markdown 表格格式
- `--onidx` - 输出使用 NIDX 格式
- `--opprint` - 输出使用 PPRINT 格式
- `--otsv` - 输出使用 TSV 格式
- `--otsvlite` - 输出使用 TSV-lite 格式
- `--ousv` 或 `--ousvlite` - 输出使用 USV 格式
- `--oxtab` - 输出使用 XTAB 格式

**通用格式选项：**
- `--io {格式名}` - 输入和输出使用指定格式。例如：`--io csv` 等同于 `--csv`
- `-i {格式名}` - 输入使用指定格式。例如：`-i csv` 等同于 `--icsv`
- `-o {格式名}` - 输出使用指定格式。例如：`-o csv` 等同于 `--ocsv`

**特殊选项：**
- `--xvright` - XTAB 格式右对齐值
- `--igen` - 忽略输入文件，生成顺序数字输入，使用 --gen-* 参数控制
- `--gen-field-name` - 指定 --igen 的字段名，默认为 "i"
- `--gen-start` - 指定 --igen 的起始值，默认为 1
- `--gen-step` - 指定 --igen 的步长值，默认为 1
- `--gen-stop` - 指定 --igen 的停止值，默认为 100

**示例：**
```bash
# 将 CSV 转换为 JSON
mlr --icsv --ojson cat data.csv

# 将 JSON 转换为美观的表格格式
mlr --ijson --opprint cat data.json

# 生成序列数据
mlr --igen --gen-start 10 --gen-stop 20 --gen-step 2 cat

# 使用简写格式
mlr -i csv -o json cat data.csv
```

### 展开/收缩标志

这些标志控制 Miller 如何转换映射或数组类型的记录值，当输入是 JSON 而输出是非 JSON（展开）时，或输入是非 JSON 而输出是 JSON（收缩）时。

**相关选项：**

- `--flatsep` 或 `--jflatsep {字符串}` - 用于展开多级 JSON 键的分隔符，例如 `{"a":{"b":3}}` 对于非 JSON 格式变成 `a:b => 3`。默认为 `.`

- `--no-auto-flatten` - 当输出是非 JSON 时，抑制默认的自动展开行为。默认情况下：如果 `$y = [7,8,9]`，这会展开为 `y.1=7,y.2=8,y.3=9`，映射也类似。使用 `--no-auto-flatten` 时，我们得到 `$y=[1, 2, 3]`

- `--no-auto-unflatten` - 当输入是非 JSON 而输出是 JSON 时，抑制默认的自动收缩行为。默认情况下：如果输入有 `y.1=7,y.2=8,y.3=9`，这会收缩为 `$y=[7,8,9]`。使用 `--no-auto-unflatten` 时，我们得到 `${y.1}=7,${y.2}=8,${y.3}=9`

**示例：**
```bash
# 使用自定义分隔符展开 JSON
mlr --ijson --ocsv --jflatsep ":" cat nested.json

# 禁用自动展开
mlr --ijson --ocsv --no-auto-flatten cat data.json

# 禁用自动收缩
mlr --icsv --ojson --no-auto-unflatten cat flat_data.csv
```

## 重要动词详解与示例

### cat - 数据连接和显示

`cat` 动词是最基础的动词，主要用于格式转换和数据连接。

**基本用法：**
```bash
# 基本的格式转换
mlr --icsv --opprint cat data.csv

# 添加记录编号
mlr --csv cat -n data.csv

# 自定义记录编号字段名
mlr --csv cat -N record_id data.csv

# 按分组添加编号
mlr --csv cat -n -g category data.csv

# 添加文件名信息
mlr --csv cat --filename data.csv

# 添加文件编号
mlr --csv cat --filenum file1.csv file2.csv
```

**实际应用示例：**
```bash
# 合并多个 CSV 文件并添加来源信息
mlr --csv cat --filename *.csv > merged_data.csv

# 为每个分类的记录添加序号
mlr --csv cat -N seq_in_group -g product_category sales.csv

# 转换格式同时添加处理时间戳
mlr --icsv --ojson cat data.csv | \
  mlr --ijson put '$processed_at = systime()' 
```

### cut - 字段选择

`cut` 动词用于选择或排除特定字段，类似于 Unix 的 `cut` 命令但功能更强大。

**基本用法：**
```bash
# 选择特定字段
mlr --csv cut -f name,age,city data.csv

# 排除特定字段
mlr --csv cut -x -f temp_field,debug_info data.csv

# 保持指定的字段顺序
mlr --csv cut -o -f city,name,age data.csv

# 使用正则表达式选择字段
mlr --csv cut -r -f '^sales_.*,^revenue_.*' data.csv

# 大小写不敏感的正则匹配
mlr --csv cut -r -f '"user.*"i' data.csv
```

**实际应用示例：**
```bash
# 选择用户基本信息字段
mlr --csv cut -f user_id,username,email,created_at users.csv

# 排除调试和临时字段
mlr --csv cut -x -f 'debug_.*,temp_.*,test_.*' data.csv

# 选择所有以日期结尾的字段
mlr --csv cut -r -f '.*_date$,.*_time$' events.csv

# 重新排列字段顺序
mlr --csv cut -o -f id,name,description,price,category products.csv
```

### filter - 数据过滤

`filter` 动词使用 Miller DSL 进行条件过滤，功能极其强大。

**基本条件过滤：**
```bash
# 数值条件
mlr --csv filter '$age >= 18 && $age <= 65' users.csv

# 字符串条件
mlr --csv filter '$status == "active" && $role != "guest"' users.csv

# 正则表达式匹配
mlr --csv filter '$email =~ ".*@company\.com$"' contacts.csv

# 空值检查
mlr --csv filter '$phone != null && $phone != ""' contacts.csv

# 范围检查
mlr --csv filter '$score >= 80 && $score <= 100' students.csv
```

**复杂过滤逻辑：**
```bash
# 组合条件
mlr --csv filter '
  ($department == "Sales" && $performance_score > 85) ||
  ($department == "Engineering" && $years_experience >= 5)
' employees.csv

# 使用函数的过滤
mlr --csv filter 'length($name) >= 3 && is_numeric($salary)' data.csv

# 基于数组或映射的过滤
mlr --ijson filter 'length($skills) >= 3 && haskey($metadata, "verified")' profiles.json

# 时间范围过滤
mlr --csv filter '
  strptime($date, "%Y-%m-%d") >= strptime("2023-01-01", "%Y-%m-%d")
' events.csv
```

**实际应用示例：**
```bash
# 客户数据筛选
mlr --csv filter '
  $age >= 25 && $age <= 55 &&
  $annual_income >= 50000 &&
  $credit_score >= 650 &&
  $account_status == "active"
' customers.csv

# 日志异常检测
mlr --ijsonl filter '
  $level == "ERROR" || 
  $response_time > 5000 ||
  $status_code >= 500
' application.log

# 电商订单筛选
mlr --csv filter '
  $order_status == "completed" &&
  $total_amount >= 100 &&
  strftime(strptime($order_date, "%Y-%m-%d"), "%Y") == "2023"
' orders.csv
```

### put - 数据计算和转换

`put` 动词用于计算新字段、修改现有字段，是 Miller 最强大的功能之一。

**基本字段计算：**
```bash
# 简单算术计算
mlr --csv put '$total = $quantity * $price' sales.csv

# 字符串操作
mlr --csv put '$full_name = $first_name . " " . $last_name' users.csv

# 条件计算
mlr --csv put '$discount = $amount > 1000 ? 0.1 : 0.05' orders.csv

# 类型转换
mlr --csv put '$age_int = int($age_string)' data.csv
```

**复杂数据转换：**
```bash
# 多步骤计算
mlr --csv put '
  $subtotal = $quantity * $unit_price;
  $tax = $subtotal * 0.08;
  $total = $subtotal + $tax;
  $profit_margin = ($total - $cost) / $total
' sales.csv

# 字符串处理和验证
mlr --csv put '
  $name_clean = strip(toupper($name));
  $email_valid = $email =~ "^[^@]+@[^@]+\.[^@]+$";
  $phone_formatted = gsub($phone, "[^0-9]", "");
  $phone_formatted = "(" . substr($phone_formatted, 0, 3) . ") " .
                     substr($phone_formatted, 3, 3) . "-" .
                     substr($phone_formatted, 6, 4)
' contacts.csv

# 时间和日期处理
mlr --csv put '
  $timestamp = systime();
  $date_parsed = strptime($date_string, "%Y-%m-%d");
  $age_days = ($timestamp - $date_parsed) / (24 * 3600);
  $age_years = int($age_days / 365.25);
  $day_of_week = strftime($date_parsed, "%A")
' events.csv
```

**聚合和统计计算：**
```bash
# 累计统计
mlr --csv put '
  begin { @total = 0; @count = 0 }
  @total += $amount;
  @count += 1;
  $running_total = @total;
  $running_average = @total / @count;
  $record_number = @count
' transactions.csv

# 分组聚合
mlr --csv put '
  begin { @category_totals = {} }
  @category_totals[$category] += $sales;
  $category_total = @category_totals[$category];
  end {
    emit @category_totals, "category_summary"
  }
' sales.csv
```

### sort - 数据排序

`sort` 动词提供了强大的多字段、多类型排序功能。

**基本排序：**
```bash
# 字典序升序
mlr --csv sort -f name data.csv

# 字典序降序
mlr --csv sort -r name data.csv

# 数值排序
mlr --csv sort -n age data.csv
mlr --csv sort -nr salary data.csv

# 多字段排序
mlr --csv sort -f department -nr salary data.csv

# 自然排序（处理数字序列）
mlr --csv sort -t filename data.csv
```

**高级排序示例：**
```bash
# 复杂多字段排序
mlr --csv sort -f region -f category -nr sales -f product_name sales.csv

# 大小写不敏感排序
mlr --csv sort -c name data.csv

# 混合排序类型
mlr --csv sort -f status -n priority -r created_date tasks.csv

# 排序后添加排名
mlr --csv sort -nr score then put '$rank = NR' leaderboard.csv
```

### stats1 - 单变量统计

详见前面创建的统计功能延伸指南。

**快速示例：**
```bash
# 基本统计
mlr --csv stats1 -a count,mean,stddev,min,max -f sales data.csv

# 分组统计
mlr --csv stats1 -a mean,median,p95 -f response_time -g service,region logs.csv

# 百分位数分析
mlr --csv stats1 -a p1,p5,p25,p50,p75,p95,p99 -f latency performance.csv
```

### join - 数据连接

`join` 动词类似于 SQL 的 JOIN 操作，用于连接多个数据源。

**基本连接：**
```bash
# 内连接
mlr --csv join -j user_id -f users.csv transactions.csv

# 左连接（包含未匹配的左表记录）
mlr --csv join -j product_id --ul -f products.csv sales.csv

# 右连接（包含未匹配的右表记录）
mlr --csv join -j customer_id --ur -f customers.csv orders.csv

# 外连接（包含所有未匹配记录）
mlr --csv join -j id --ul --ur -f table1.csv table2.csv
```

**高级连接示例：**
```bash
# 不同字段名连接
mlr --csv join -l user_id -r customer_id -f users.csv orders.csv

# 多字段连接
mlr --csv join -j region,category -f regions.csv sales.csv

# 添加前缀避免字段名冲突
mlr --csv join -j id --lp "user_" --rp "profile_" -f users.csv profiles.csv

# 只保留特定字段
mlr --csv join -j user_id --lk "name,email" -f users.csv orders.csv
```

这些核心动词构成了 Miller 数据处理的基础，掌握它们的用法对于高效使用 Miller 至关重要。 

## 快捷方式和节省击键

Miller 提供了一些快捷方式来减少击键，提高效率。

### 短格式指定符
- `--c2p` 等同于 `--icsv --opprint`（CSV 输入，美观的表格输出）
- `--c2j` 等同于 `--icsv --ojson`（CSV 输入，JSON 输出）
- 类似的有 `--c2t`、`--c2d`、`--c2n`、`--c2x`、`--j2c`、`--j2t` 等，用于常见格式转换。

示例：
```bash
mlr --c2p head -n 2 example.csv
```

### 文件名置前
- 使用 `--from {文件名}` 将文件名置于命令开头，便于命令历史编辑。
- 对于多个文件，使用 `--mfrom {文件1} {文件2} --`。

示例：
```bash
mlr --c2p --from example.csv sort -nr index then head -n 3
```

### 最短标志
- `-c` 等同于 `--csv`
- `-t` 等同于 `--tsv`
- `-j` 等同于 `--json`

### .mlrrc 文件
- 在 `~/.mlrrc` 中设置默认选项，如 `--csv`，则后续命令默认使用 CSV 格式。
- 示例：将 `--csv` 写入 `~/.mlrrc`，则 `mlr cat example.csv` 默认处理 CSV。

### 脚本和 mlr -s
- 使用 `-s {脚本文件}` 运行脚本文件中的 Miller 命令。
- 脚本文件可包含多行命令，便于重复使用复杂操作。

## Miller 编程语言（DSL）

Miller 的 `put` 和 `filter` 动词使用嵌入式领域特定语言（DSL），允许复杂数据转换。

### 记录和字段
- 使用 `$字段名` 引用输入字段，如 `$cost = $quantity * $rate`。
- 新字段添加到记录末尾，现有字段就地更新。

### 多行语句
- 使用分号分隔语句，可跨多行。
- 注释使用 `#`。

示例：
```bash
mlr --c2p put '
  $cost = $quantity * $rate;  # 计算成本
  $index *= 100
' example.csv
```

### Begin/End 块
- `begin { ... }` 在处理第一条记录前执行。
- `end { ... }` 在处理最后一条记录后执行。
- 用于初始化变量或输出总结。

示例：
```bash
mlr --c2p put '
  begin { @total = 0 }
  @total += $quantity;
  $running_total = @total;
  end { emit @total, "grand_total" }
' example.csv
```

### 变量
- 使用 `@变量名` 定义跨记录变量，如计数器或总和。
- 支持数组和映射：`@counts[$color] += 1`。

### 用户定义函数
- 使用 `func` 定义函数。
- 示例：
```bash
func square(x) { return x * x }
$area = square($side)
```

### Emit 和 Dump
- `emit` 输出变量，如 `emit @counts`。
- `dump` 输出所有变量，用于调试。

### 其他 DSL 特性
- 支持 if-else、for/while 循环、模式匹配、递归函数等。
- 内置函数丰富，包括数学、字符串、时间等（详见函数列表）。

## 更多动词示例

### top - 显示最高值
显示每个指定字段的前 N 个最高值。

示例：
```bash
mlr --c2p top -f quantity -n 3 example.csv
```

### histogram - 创建直方图
按字段值分桶计数。

示例：
```bash
mlr --c2p histogram -f quantity --lo 0 --hi 100 --nbins 10 example.csv
```

### bar - 创建条形图
使用字符创建条形图。

示例：
```bash
mlr --c2p bar -f quantity -c "#" example.csv
```

### bootstrap - 引导抽样
有放回随机抽样。

示例：
```bash
mlr --csv bootstrap -n 100 data.csv
```

### decimate - 抽取记录
每 N 条记录保留一条。

示例：
```bash
mlr --csv decimate -b 5 data.csv
```

这些特性扩展了 Miller 的功能，允许更复杂的处理和分析。 