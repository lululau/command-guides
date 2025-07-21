# Miller 统计和聚合功能详解

## 概述

Miller 提供了强大的统计和聚合功能，可以对数据进行单变量、双变量分析，以及时间序列分析。主要通过 `stats1`、`stats2`、`step`、`histogram` 等动词实现。

## 单变量统计（stats1）

### 基本统计量

`stats1` 动词提供了丰富的单变量统计函数：

#### 基础统计
- `count` - 计数
- `sum` - 求和
- `mean` - 平均值
- `min` - 最小值
- `max` - 最大值

#### 分布统计
- `median` - 中位数（等同于 p50）
- `mode` - 众数（最频繁出现的值）
- `antimode` - 反众数（最不频繁出现的值）
- `stddev` - 标准差
- `var` - 方差
- `skewness` - 偏度
- `kurtosis` - 峰度

#### 百分位数
- `p10`, `p25`, `p50`, `p75`, `p90`, `p95`, `p99` - 各种百分位数
- 支持任意百分位数，如 `p25.5`, `p99.9`

#### 字符串统计
- `minlen` - 最短字符串长度
- `maxlen` - 最长字符串长度
- `null_count` - 空值计数
- `distinct_count` - 不同值计数

### 基本用法示例

```bash
# 计算单个字段的基本统计
mlr --csv stats1 -a count,sum,mean,stddev -f sales data.csv

# 计算多个字段的统计
mlr --csv stats1 -a mean,median,max -f price,quantity,revenue data.csv

# 按分组计算统计
mlr --csv stats1 -a mean,count -f revenue -g region,category data.csv

# 计算百分位数
mlr --csv stats1 -a p25,p50,p75,p95 -f response_time data.csv
```

### 高级分组统计

```bash
# 多级分组
mlr --csv stats1 -a mean,count,stddev -f sales -g year,quarter,region data.csv

# 正则表达式字段选择
mlr --csv stats1 -a mean --fr '^sales_.*' --gr '^region_.*' data.csv

# 排除特定字段
mlr --csv stats1 -a sum --fx 'temp_.*' data.csv

# 组合字段和分组的正则表达式
mlr --csv stats1 -a mean --grfx '^(region|category)_.*' data.csv
```

### 实际应用示例

```bash
# 电商数据分析
mlr --csv stats1 \
  -a count,sum,mean,median,stddev,p95 \
  -f order_amount \
  -g customer_segment,product_category \
  ecommerce_data.csv

# 网站性能分析
mlr --csv stats1 \
  -a count,mean,median,p95,p99,max \
  -f response_time \
  -g endpoint,method \
  web_logs.csv

# 销售漏斗分析
mlr --csv stats1 \
  -a count,sum,mean \
  -f conversion_rate \
  -g traffic_source,campaign_type \
  marketing_data.csv
```

## 双变量统计（stats2）

### 支持的分析类型

#### 回归分析
- `linreg-ols` - 普通最小二乘法线性回归
- `linreg-pca` - 主成分分析线性回归
- `logireg` - 逻辑回归
- `r2` - 决定系数（需要与 linreg-ols 配合使用）

#### 相关性分析
- `corr` - 皮尔逊相关系数
- `cov` - 协方差
- `covx` - 协方差矩阵

### 基本用法

```bash
# 计算两个变量的相关性
mlr --csv stats2 -a corr -f price,sales data.csv

# 线性回归分析
mlr --csv stats2 -a linreg-ols,r2 -f advertising_spend,revenue data.csv

# 主成分回归
mlr --csv stats2 -a linreg-pca -f x,y data.csv

# 逻辑回归（适用于二分类问题）
mlr --csv stats2 -a logireg -f feature,target data.csv

# 分组回归分析
mlr --csv stats2 -a linreg-ols,r2 -f price,demand -g product_category data.csv
```

### 高级分析示例

```bash
# 多个变量对的相关性分析
mlr --csv stats2 -a corr -f x1,y1,x2,y2,x3,y3 data.csv

# 拟合回归模型并应用到数据
mlr --csv stats2 -a linreg-ols --fit -f advertising,sales data.csv

# 协方差矩阵分析
mlr --csv stats2 -a covx -f feature1,feature2,feature3,feature4 data.csv
```

### 实际应用场景

```bash
# 价格敏感性分析
mlr --csv stats2 \
  -a linreg-ols,r2 \
  -f price,sales_volume \
  -g product_category \
  sales_data.csv

# 广告效果分析
mlr --csv stats2 \
  -a corr,linreg-ols \
  -f ad_spend,conversions \
  -g channel,campaign_type \
  marketing_data.csv

# 用户行为相关性分析
mlr --csv stats2 \
  -a corr \
  -f page_views,session_duration,bounce_rate,conversion_rate \
  user_analytics.csv
```

## 时间序列分析（step）

### 支持的步进计算

#### 基础步进函数
- `counter` - 计数器
- `delta` - 相邻记录差值
- `ratio` - 相邻记录比率
- `from-first` - 与第一条记录的差值
- `rsum` - 累计求和
- `rprod` - 累计乘积

#### 移动平均
- `shift` 或 `shift_lag` - 包含前一条记录的值
- `shift_lead` - 包含后一条记录的值
- `ewma` - 指数加权移动平均
- `slwin` - 滑动窗口平均

### 基本用法

```bash
# 计算累计和
mlr --csv step -a rsum -f sales data.csv

# 计算相邻记录的差值
mlr --csv step -a delta -f price data.csv

# 计算与第一条记录的差值
mlr --csv step -a from-first -f stock_price data.csv

# 按分组计算步进统计
mlr --csv step -a rsum,delta -f revenue -g product_id data.csv
```

### 移动平均和滑动窗口

```bash
# 指数加权移动平均
mlr --csv step -a ewma -d 0.1,0.5,0.9 -f stock_price data.csv

# 自定义 EWMA 输出字段名
mlr --csv step -a ewma -d 0.1,0.9 -o smooth,rough -f price data.csv

# 滑动窗口平均（7个历史值，2个未来值）
mlr --csv step -a slwin_7_2 -f sales data.csv

# 向前和向后滑动窗口
mlr --csv step -a slwin_9_0,slwin_0_9 -f value data.csv
```

### 实际时间序列分析

```bash
# 股价技术分析
mlr --csv step \
  -a rsum,delta,ratio,ewma \
  -d 0.1,0.3 \
  -f close_price \
  -g stock_symbol \
  stock_data.csv

# 网站流量趋势分析
mlr --csv step \
  -a delta,rsum,slwin_7_0 \
  -f daily_visits \
  web_analytics.csv

# 销售业绩跟踪
mlr --csv step \
  -a rsum,delta,from-first \
  -f monthly_sales \
  -g sales_rep,region \
  sales_performance.csv
```

## 直方图分析（histogram）

### 基本用法

```bash
# 基本直方图
mlr --csv histogram -f age --lo 0 --hi 100 --nbins 20 data.csv

# 自动计算范围
mlr --csv histogram -f income --auto --nbins 50 data.csv

# 自定义输出前缀
mlr --csv histogram -f score --lo 0 --hi 100 -o grade_ data.csv
```

### 实际应用

```bash
# 年龄分布分析
mlr --csv histogram \
  -f age \
  --auto \
  --nbins 25 \
  -o age_dist_ \
  customer_data.csv

# 收入分布分析
mlr --csv histogram \
  -f annual_income \
  --lo 20000 \
  --hi 200000 \
  --nbins 30 \
  salary_survey.csv

# 响应时间分布
mlr --csv histogram \
  -f response_time_ms \
  --auto \
  --nbins 40 \
  performance_logs.csv
```

## 高级聚合模式

### 使用 put 和 begin/end 进行自定义聚合

```bash
# 自定义聚合逻辑
mlr --csv put '
  begin {
    @total_revenue = 0;
    @customer_count = 0;
    @product_performance = {};
    @monthly_trends = {}
  }
  
  # 累计指标
  @total_revenue += $revenue;
  @customer_count += 1;
  @product_performance[$product] += $revenue;
  
  # 时间趋势分析
  $month = strftime(strptime($date, "%Y-%m-%d"), "%Y-%m");
  @monthly_trends[$month] += $revenue;
  
  # 计算衍生指标
  $customer_value = $revenue / $quantity;
  $profit_margin = ($revenue - $cost) / $revenue;
  
  end {
    # 输出汇总指标
    emit {
      "total_revenue": @total_revenue,
      "avg_revenue_per_customer": @total_revenue / @customer_count,
      "customer_count": @customer_count
    }, "summary";
    
    # 输出产品业绩排名
    emit @product_performance, "product_performance";
    
    # 输出月度趋势
    emit @monthly_trends, "monthly_trends"
  }
' -q sales_data.csv
```

### 复杂分组聚合

```bash
# 多维度聚合分析
mlr --csv put '
  begin {
    @metrics = {};
  }
  
  # 构建多维键
  $key = $region . "|" . $category . "|" . $quarter;
  
  # 聚合多个指标
  @metrics[$key]["revenue"] += $revenue;
  @metrics[$key]["units"] += $units_sold;
  @metrics[$key]["customers"] += 1;
  
  end {
    # 输出结构化结果
    for (key, data in @metrics) {
      $parts = split(key, "|");
      emit {
        "region": $parts[1],
        "category": $parts[2], 
        "quarter": $parts[3],
        "total_revenue": data["revenue"],
        "total_units": data["units"],
        "customer_count": data["customers"],
        "avg_revenue_per_customer": data["revenue"] / data["customers"]
      }, "aggregated_metrics"
    }
  }
' -q business_data.csv
```

## 百分位数和分位数分析

### 详细分位数分析

```bash
# 完整的分位数分析
mlr --csv stats1 \
  -a count,min,p5,p10,p25,p50,p75,p90,p95,p99,max \
  -f response_time \
  -g service_name \
  performance_data.csv

# 自定义百分位数
mlr --csv stats1 \
  -a p1,p5,p10,p25,p50,p75,p90,p95,p99,p99.9 \
  -f latency_ms \
  api_logs.csv
```

### 性能基准分析

```bash
# SLA 性能分析
mlr --csv put '
  # 计算 SLA 指标
  $sla_target_95 = 500;  # 95% 请求应在 500ms 内
  $sla_target_99 = 1000; # 99% 请求应在 1000ms 内
' then stats1 \
  -a count,p95,p99,max \
  -f response_time \
  -g endpoint \
then put '
  # 判断 SLA 达成情况
  $sla_95_met = $response_time_p95 <= 500;
  $sla_99_met = $response_time_p99 <= 1000;
  $sla_status = $sla_95_met && $sla_99_met ? "PASS" : "FAIL"
' api_performance.csv
```

## 统计分析最佳实践

### 1. 数据清理

```bash
# 在统计分析前清理数据
mlr --csv \
  filter '$value != null && $value != ""' then \
  put '$value = float($value)' then \
  filter '$value >= 0' then \
  stats1 -a mean,median,stddev -f value \
  data.csv
```

### 2. 异常值检测

```bash
# 使用 IQR 方法检测异常值
mlr --csv stats1 -a p25,p75 -f value then put '
  $iqr = $value_p75 - $value_p25;
  $lower_bound = $value_p25 - 1.5 * $iqr;
  $upper_bound = $value_p75 + 1.5 * $iqr
' then filter '$value >= $lower_bound && $value <= $upper_bound' data.csv
```

### 3. 分层采样

```bash
# 按分组进行分层统计
mlr --csv stats1 \
  -a count,mean,stddev \
  -f metric \
  -g stratum \
then put '
  # 计算权重和置信区间
  $weight = $metric_count / $total_count;
  $margin_error = 1.96 * $metric_stddev / sqrt($metric_count);
  $ci_lower = $metric_mean - $margin_error;
  $ci_upper = $metric_mean + $margin_error
' survey_data.csv
```

### 4. 趋势分析

```bash
# 时间序列趋势分析
mlr --csv put '$timestamp = strptime($date, "%Y-%m-%d")' then \
  sort -f timestamp then \
  step -a delta,rsum,slwin_7_0 -f value then \
  put '
    # 计算趋势指标
    $trend = $value_delta > 0 ? "up" : $value_delta < 0 ? "down" : "flat";
    $volatility = abs($value_delta);
    $growth_rate = $value_delta / ($value - $value_delta) * 100
  ' time_series_data.csv
```

这些统计和聚合功能使得 Miller 成为强大的数据分析工具，能够处理从简单的描述性统计到复杂的时间序列分析等各种分析需求。 