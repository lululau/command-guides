# Miller DSL 编程语言详解

## 概述

Miller 的 DSL（Domain-Specific Language，领域特定语言）是一个专门为数据处理设计的编程语言。它主要用于 `put` 和 `filter` 动词中，提供了强大的数据操作能力。

## DSL 基础语法

### 变量和字段引用

在 Miller DSL 中，有几种不同类型的变量：

1. **字段变量**：以 `$` 开头，引用当前记录的字段
2. **出流变量**：以 `@` 开头，在记录之间保持状态
3. **内置变量**：系统提供的特殊变量

```bash
# 字段变量示例
mlr put '$total = $quantity * $price' data.csv

# 出流变量示例
mlr put '@sum += $amount; $running_total = @sum' data.csv

# 内置变量示例
mlr put '$record_number = NR; $filename = FILENAME' data.csv
```

### 数据类型

Miller DSL 支持多种数据类型：

- **字符串**：`"hello"`, `'world'`
- **数字**：`42`, `3.14`, `-1.5e10`
- **布尔值**：`true`, `false`
- **数组**：`[1, 2, 3]`, `["a", "b", "c"]`
- **映射**：`{"key": "value", "count": 42}`
- **空值**：`null`

```bash
# 类型示例
mlr put '
  $name = "Alice";
  $age = 25;
  $active = true;
  $scores = [85, 92, 78];
  $metadata = {"created": "2023", "updated": "2024"}
' data.csv
```

## 常用操作符

### 算术操作符

```bash
# 基本算术
mlr put '$result = $a + $b - $c * $d / $e' data.csv

# 幂运算
mlr put '$square = $x ** 2' data.csv

# 取模
mlr put '$remainder = $total % 10' data.csv
```

### 比较操作符

```bash
# 数值比较
mlr filter '$age > 18 && $score <= 100' data.csv

# 字符串比较
mlr filter '$name =~ "^A.*" && $city != "Unknown"' data.csv

# 空值检查
mlr filter '$email != null && $phone != ""' data.csv
```

### 逻辑操作符

```bash
# 与、或、非
mlr filter '$age >= 18 && ($status == "active" || $priority == "high")' data.csv

# 三元操作符
mlr put '$category = $age < 18 ? "minor" : "adult"' data.csv
```

## 控制结构

### 条件语句

```bash
# 简单 if 语句
mlr put 'if ($score >= 90) { $grade = "A" }' data.csv

# if-else 语句
mlr put '
  if ($score >= 90) {
    $grade = "A"
  } else if ($score >= 80) {
    $grade = "B"
  } else if ($score >= 70) {
    $grade = "C"
  } else {
    $grade = "F"
  }
' data.csv

# 复杂条件
mlr put '
  if ($age >= 18 && $income > 30000) {
    $loan_eligible = true;
    $max_loan = $income * 3
  } else {
    $loan_eligible = false;
    $max_loan = 0
  }
' data.csv
```

### 循环语句

```bash
# for 循环遍历数组
mlr put '
  $total_score = 0;
  for (score in $scores) {
    $total_score += score
  }
  $average = $total_score / length($scores)
' data.csv

# for 循环遍历映射
mlr put '
  $summary = "";
  for (key, value in $metadata) {
    $summary = $summary . key . ":" . value . ";"
  }
' data.csv

# while 循环
mlr put '
  $factorial = 1;
  $i = 1;
  while ($i <= $n) {
    $factorial *= $i;
    $i += 1
  }
' data.csv
```

## 内置变量

### 记录相关变量

- `NR`：当前记录在所有输入中的行号
- `FNR`：当前记录在当前文件中的行号
- `NF`：当前记录的字段数
- `FILENAME`：当前输入文件名
- `FILENUM`：当前文件在输入列表中的编号

```bash
# 使用内置变量
mlr put '
  $record_id = FILENAME . ":" . FNR;
  $field_count = NF;
  $is_first_record = (NR == 1);
  $file_number = FILENUM
' data.csv
```

### 时间相关变量

```bash
# 时间戳相关
mlr put '
  $processed_at = systime();
  $process_date = strftime(systime(), "%Y-%m-%d");
  $uptime_seconds = uptime()
' data.csv
```

## 函数使用

### 字符串函数

```bash
# 字符串操作
mlr put '
  $name_upper = toupper($name);
  $name_length = strlen($name);
  $first_char = substr($name, 0, 1);
  $last_name = regextract($full_name, "([A-Z][a-z]+)$");
  $clean_phone = gsub($phone, "[^0-9]", "")
' data.csv
```

### 数学函数

```bash
# 数学计算
mlr put '
  $abs_value = abs($amount);
  $rounded = round($price, 2);
  $sqrt_val = sqrt($area);
  $log_val = log($population);
  $random = urand()
' data.csv
```

### 类型转换函数

```bash
# 类型转换
mlr put '
  $age_int = int($age_string);
  $price_float = float($price_string);
  $status_bool = boolean($active_flag);
  $id_string = string($user_id)
' data.csv
```

### 数组和映射函数

```bash
# 数组操作
mlr put '
  $item_count = length($items);
  $first_item = $items[1];
  $sorted_items = sort($items);
  $has_target = any($items, func(x) { return x == $target })
' data.csv

# 映射操作
mlr put '
  $keys = get_keys($metadata);
  $values = get_values($metadata);
  $has_key = haskey($metadata, "status")
' data.csv
```

## 高级特性

### 用户定义函数

```bash
# 定义和使用函数
mlr put '
  func calculate_bmi(weight, height) {
    return weight / (height * height)
  }
  
  $bmi = calculate_bmi($weight_kg, $height_m);
  $bmi_category = $bmi < 18.5 ? "underweight" : 
                  $bmi < 25 ? "normal" : 
                  $bmi < 30 ? "overweight" : "obese"
' data.csv
```

### begin 和 end 块

```bash
# 初始化和汇总
mlr put '
  begin {
    @total_count = 0;
    @total_amount = 0;
    @categories = {}
  }
  
  @total_count += 1;
  @total_amount += $amount;
  @categories[$category] += 1;
  
  end {
    emit {"total_records": @total_count, "total_amount": @total_amount}, "summary";
    emit @categories, "categories"
  }
' -q data.csv
```

### 正则表达式

```bash
# 正则表达式匹配和提取
mlr put '
  # 检查邮箱格式
  $valid_email = $email =~ "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$";
  
  # 提取电话号码中的区号
  if ($phone =~ "^\(([0-9]{3})\)") {
    $area_code = "\1"
  }
  
  # 替换敏感信息
  $masked_ssn = gsub($ssn, "[0-9]", "*")
' data.csv
```

## 实际应用示例

### 数据清洗

```bash
# 综合数据清洗
mlr put '
  # 规范化名称
  $name = strip(toupper($name));
  
  # 验证和修正年龄
  if ($age < 0 || $age > 150) {
    $age = null;
    $age_valid = false
  } else {
    $age_valid = true
  }
  
  # 格式化电话号码
  $phone_clean = gsub($phone, "[^0-9]", "");
  if (strlen($phone_clean) == 10) {
    $phone_formatted = "(" . substr($phone_clean, 0, 3) . ") " . 
                      substr($phone_clean, 3, 3) . "-" . 
                      substr($phone_clean, 6, 4)
  } else {
    $phone_formatted = null
  }
  
  # 计算衍生字段
  $name_length = strlen($name);
  $is_senior = $age >= 65;
  $generation = $age < 25 ? "Gen Z" : 
                $age < 41 ? "Millennial" : 
                $age < 57 ? "Gen X" : "Boomer"
' data.csv
```

### 数据分析

```bash
# 销售数据分析
mlr put '
  begin {
    @monthly_sales = {};
    @product_performance = {};
    @total_revenue = 0
  }
  
  # 提取月份
  $month = strftime(strptime($date, "%Y-%m-%d"), "%Y-%m");
  
  # 计算收入
  $revenue = $quantity * $unit_price * (1 - $discount);
  
  # 累计统计
  @monthly_sales[$month] += $revenue;
  @product_performance[$product] += $revenue;
  @total_revenue += $revenue;
  
  # 计算指标
  $profit_margin = ($revenue - $cost) / $revenue;
  $is_profitable = $profit_margin > 0.2;
  
  end {
    # 输出汇总报告
    for (month, sales in @monthly_sales) {
      emit {"month": month, "sales": sales, "percentage": sales/@total_revenue*100}, "monthly_summary"
    }
    
    emit {"total_revenue": @total_revenue}, "overall_summary"
  }
' -q sales.csv
```

### 日志分析

```bash
# Web 日志分析
mlr put '
  # 解析时间戳
  $timestamp_parsed = strptime($timestamp, "%d/%b/%Y:%H:%M:%S");
  $hour = strftime($timestamp_parsed, "%H");
  $date = strftime($timestamp_parsed, "%Y-%m-%d");
  
  # 分类请求
  $is_api = $path =~ "^/api/";
  $is_static = $path =~ "\.(css|js|png|jpg|gif)$";
  $is_error = $status >= 400;
  
  # 计算响应时间分类
  $response_time_category = 
    $response_time < 100 ? "fast" :
    $response_time < 500 ? "medium" :
    $response_time < 2000 ? "slow" : "very_slow";
  
  # 检测异常
  $suspicious_user_agent = $user_agent =~ "(bot|spider|crawler)"i;
  $large_response = $bytes > 1000000;
  
  begin {
    @hourly_traffic = {};
    @error_count = 0;
    @total_requests = 0
  }
  
  @hourly_traffic[$hour] += 1;
  @total_requests += 1;
  if ($is_error) { @error_count += 1 }
  
  end {
    emit {"error_rate": @error_count / @total_requests * 100}, "summary"
  }
' -q access.log
```

## 性能优化技巧

### 1. 避免重复计算

```bash
# 错误：重复计算
mlr put '$a = expensive_function($x); $b = expensive_function($x) + 1' 

# 正确：缓存结果
mlr put '$temp = expensive_function($x); $a = $temp; $b = $temp + 1'
```

### 2. 使用合适的数据结构

```bash
# 对于大量查找操作，使用映射而不是数组
mlr put '
  begin { @lookup = {"key1": "value1", "key2": "value2"} }
  $result = @lookup[$key]  # O(1) 查找
'
```

### 3. 早期过滤

```bash
# 在计算之前先过滤
mlr filter '$status == "active"' then put '$complex_calc = ...' data.csv
```

## 调试技巧

### 1. 使用 -v 标志查看 AST

```bash
mlr -v put '$result = $a + $b' data.csv
```

### 2. 分步调试

```bash
# 逐步添加复杂性
mlr put '$debug1 = $input' data.csv
mlr put '$debug1 = $input; $debug2 = process($debug1)' data.csv
```

### 3. 使用 emit 输出中间结果

```bash
mlr put 'emit {"debug": $intermediate_value}, "debug_output"' data.csv
```

Miller DSL 提供了丰富的编程能力，使得复杂的数据处理任务可以通过简洁的表达式来完成。掌握这些语法和技巧，能够大大提高数据处理的效率和准确性。 