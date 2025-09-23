tidy_data
================
chuqi
2025-09-23

示例
我将继续使用与数据导入和数据处理相同的仓库/项目，但会创建一个新的.Rmd文件用于整理。同时，我将加载一些相关包，并限制tibble中打印的行数。

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.2     ✔ tibble    3.3.0
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.1.0     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(readxl)
library(haven)

options(tibble.print_min = 5)
```

## pivot_longer

pivot_longer(…) 这是 tidyr 包中的一个函数，用于将数据框从“宽格式（wide
format）”转换为“长格式（long format）”。

在数据导入过程中，我们使用haven包从.sas7bdat文件加载了PULSE生物标志物数据集。
现在让我们重新加载这些数据并深入分析：

``` r
pulse_df = 
  haven::read_sas("./data/public_pulse_data.sas7bdat") |>
  janitor::clean_names()

pulse_df
```

    ## # A tibble: 1,087 × 7
    ##      id   age sex   bdi_score_bl bdi_score_01m bdi_score_06m bdi_score_12m
    ##   <dbl> <dbl> <chr>        <dbl>         <dbl>         <dbl>         <dbl>
    ## 1 10003  48.0 male             7             1             2             0
    ## 2 10015  72.5 male             6            NA            NA            NA
    ## 3 10022  58.5 male            14             3             8            NA
    ## 4 10026  72.7 male            20             6            18            16
    ## 5 10035  60.4 male             4             0             1             2
    ## # ℹ 1,082 more rows

基于对整洁数据的新认识，我们很快发现了一个问题：BDI评分分散在四个列中，分别对应四个观察时间点。
我们可以使用pivot_longer函数解决这个问题：

``` r
pulse_tidy_df = 
  pivot_longer(
    pulse_df,                           # 原始数据框
    bdi_score_bl:bdi_score_12m,         # 选取要转换的列，从 bdi_score_bl 到 bdi_score_12m
    names_to = "visit",                 # 新建一列名为 visit，用来存放原来的列名
    values_to = "bdi"                   # 新建一列名为 bdi，用来保存原来的数值（如 12、15 等）
  )

pulse_tidy_df
```

    ## # A tibble: 4,348 × 5
    ##      id   age sex   visit           bdi
    ##   <dbl> <dbl> <chr> <chr>         <dbl>
    ## 1 10003  48.0 male  bdi_score_bl      7
    ## 2 10003  48.0 male  bdi_score_01m     1
    ## 3 10003  48.0 male  bdi_score_06m     2
    ## 4 10003  48.0 male  bdi_score_12m     0
    ## 5 10015  72.5 male  bdi_score_bl      6
    ## # ℹ 4,343 more rows

看起来好多了！然而，现在visit有一个问题。原始列名虽然信息量很大，但我们可能不需要bdi_score_在每种情况下都保留前缀。我将使用一个附加选项来pivot_longer解决这个问题：

``` r
pulse_tidy_df = 
  pivot_longer(
    pulse_df,                              # 原始数据框，包含多个时间点的抑郁评分（bdi）
    bdi_score_bl:bdi_score_12m,            # 要转换的列范围，从基线（bl）到12个月（12m）
    names_to = "visit",                    # 创建新变量 visit，用来保存原来的列名（如 bdi_score_bl）
    names_prefix = "bdi_score_",           # 去掉原列名前缀 "bdi_score_"，只保留后面的部分（如 bl、6m、12m）
    values_to = "bdi"                      # 创建新变量 bdi，用来存放原来各列中的数值
  )

pulse_tidy_df
```

    ## # A tibble: 4,348 × 5
    ##      id   age sex   visit   bdi
    ##   <dbl> <dbl> <chr> <chr> <dbl>
    ## 1 10003  48.0 male  bl        7
    ## 2 10003  48.0 male  01m       1
    ## 3 10003  48.0 male  06m       2
    ## 4 10003  48.0 male  12m       0
    ## 5 10015  72.5 male  bl        6
    ## # ℹ 4,343 more rows

在前面，我保存了中间数据集，以便清晰地展示每个步骤。
虽然这在您尝试代码时可能有所帮助，但通常来说，这不是一个好习惯。
为了完成数据整理过程，还需要进行一些额外的转换，例如，为了在访问中保持一致性，将其更改为，以及转换bl为因子变量。
（您可能希望将其转换为数值变量，这可以通过调用 来完成
。）00mvisitvisitmutate

总之，下面的代码将导入、整理并将 PULSE 数据集转换为可用格式：

``` r
pulse_df = 
  haven::read_sas("./data/public_pulse_data.sas7bdat") |>  # 读取 SAS 格式的数据文件
  janitor::clean_names() |>                                # 清理列名（全部转为小写、下划线格式，更易用）
  
  pivot_longer(                                            # 宽转长，把多个列转为两列（visit 和 bdi）
    bdi_score_bl:bdi_score_12m,                            # 选中从 bdi_score_bl 到 bdi_score_12m 的列
    names_to = "visit",                                    # 将原来的列名存入新列 visit
    names_prefix = "bdi_score_",                           # 去掉列名前缀 "bdi_score_"，只留下 bl、6m、12m
    values_to = "bdi"                                      # 将原列中的数值存入新列 bdi
  ) |>
  
  relocate(id, visit) |>                                   # 调整列顺序，把 id 和 visit 移到最前面
  
  mutate(                                                  # mutate：变出新列、改已有列
    visit = replace(visit, visit == "bl", "00m"),          # 将 visit 中的 "bl"（basicline基线）替换为 "00m"
    visit = factor(visit)                                  # 把 visit 列转换为因子（方便后续排序或分组分析）
  )

print(pulse_df, n = 12)  # 打印 pulse_df 的前 12 行数据
```

    ## # A tibble: 4,348 × 5
    ##       id visit   age sex     bdi
    ##    <dbl> <fct> <dbl> <chr> <dbl>
    ##  1 10003 00m    48.0 male      7
    ##  2 10003 01m    48.0 male      1
    ##  3 10003 06m    48.0 male      2
    ##  4 10003 12m    48.0 male      0
    ##  5 10015 00m    72.5 male      6
    ##  6 10015 01m    72.5 male     NA
    ##  7 10015 06m    72.5 male     NA
    ##  8 10015 12m    72.5 male     NA
    ##  9 10022 00m    58.5 male     14
    ## 10 10022 01m    58.5 male      3
    ## 11 10022 06m    58.5 male      8
    ## 12 10022 12m    58.5 male     NA
    ## # ℹ 4,336 more rows

现在我们的状况还不错:-)。

another example

``` r
litter_df = 
  read_csv(                                                          # 读取 CSV 文件 FAS_litters.csv
    "./data/FAS_litters.csv",                                        # 文件路径
    na = c("NA", ".", "")) |>                                        # 将 "NA"、"."、空字符串识别为缺失值
  janitor::clean_names() |> 
  pivot_longer(                                            # 宽转长，把多个列转为两列（visit 和 bdi）
    gd0_weight:gd18_weight,                            # 选中从 bdi_score_bl 到 bdi_score_12m 的列
    names_to = "gd_time",                                    # 将原来的列名存入新列 visit
    values_to = "weight"                                      # 将原列中的数值存入新列 bdi
  ) |>
  mutate(
    gd_time = case_match(
      gd_time,
      "gd0_weight" ~ 0,
      "gd18_weight" ~ 18
    )
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litter_df
```

    ## # A tibble: 98 × 8
    ##   group litter_number gd_of_birth pups_born_alive pups_dead_birth pups_survive
    ##   <chr> <chr>               <dbl>           <dbl>           <dbl>        <dbl>
    ## 1 Con7  #85                    20               3               4            3
    ## 2 Con7  #85                    20               3               4            3
    ## 3 Con7  #1/2/95/2              19               8               0            7
    ## 4 Con7  #1/2/95/2              19               8               0            7
    ## 5 Con7  #5/5/3/83/3-3          19               6               0            5
    ## # ℹ 93 more rows
    ## # ℹ 2 more variables: gd_time <dbl>, weight <dbl>

\##学习评估：

在窝仔数据中，变量gd0_weight和gd18_weight分别表示母鼠在妊娠第0天和第18天的体重。
编写一个数据清理链，仅保留litter_number和这些列；生成新的变量gd和weight；
并创建和的gd数值变量（对于最后一部分，你可能想使用……）。
这个版本“整洁”吗？018recode

## 解决方案

pivot_wider

我们一直以来都专注于整理数据，但我们也承认，有时不整洁的数据更利于人类阅读。
因此，我们将稍微偏离主题，谈谈如何整理你的整洁数据。

下面的代码创建了一个整洁的数据集，可用于分析。
这是进行进一步分析或可视化的正确格式，但不利于人类读者进行快速比较。

``` r
analysis_result =                         # 创建一个新的数据框（tibble），赋值给变量 analysis_result
  tibble(                                 # 使用 tibble() 函数，生成一个简洁的表格数据结构
    group = c("treatment", "treatment", "placebo", "placebo"),  # 定义一列 group，包含4个元素，表示组别
    time = c("pre", "post", "pre", "post"),                     # 定义一列 time，表示时间点（干预前后）
    mean = c(4, 8, 3.5, 4)                                      # 定义一列 mean，表示对应组别和时间的平均值
  )

analysis_result                          # 输出数据框内容，查看创建的 tibble
```

    ## # A tibble: 4 × 3
    ##   group     time   mean
    ##   <chr>     <chr> <dbl>
    ## 1 treatment pre     4  
    ## 2 treatment post    8  
    ## 3 placebo   pre     3.5
    ## 4 placebo   post    4

相同数据的另一种呈现方式可能是将组放在行中，将时间放在列中，并将平均值放在表格单元格中。这显然不够整洁；
为了实现这一点，我们需要使用 pivot_wider，它是pivot_longer的逆：

``` r
pivot_wider(
  analysis_result,          # 输入数据框，长格式的分析结果
  names_from = "time",      # 将 "time" 列的不同值（如 "pre", "post"）变成新的列名
  values_from = "mean"      # 新列对应的值取自 "mean" 列
)|>
  knitr::kable()
```

| group     | pre | post |
|:----------|----:|-----:|
| treatment | 4.0 |    8 |
| placebo   | 3.5 |    4 |

现在我们已经基本完成了 - 在某些情况下，您可能会使用它
select来重新排序列，并且（取决于您的目标）使用它
knitr::kable()来生成更美观的阅读表格。

## Binding rows(堆叠)

我们研究了单表非整洁数据，但非整洁性通常源于分散在多个表中的相关数据。
在最简单的情况下，这些表基本相同，可以堆叠起来生成一个整洁的数据集。
这就是中的设置LotR_words.xlsx，其中三部曲中每部电影中不同种族和性别的字数分布在不同的数据矩形中
（这些数据基于此示例）。

为了生成所需的整洁数据集，我们首先需要读取每个表并进行一些清理。

``` r
fellowship_ring = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "B3:D6") |>   #range，Excel取值
  mutate(movie = "fellowship_ring")   #创建新的列“movie”

two_towers = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "F3:H6") |>
  mutate(movie = "two_towers")

return_king = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "J3:L6") |>
  mutate(movie = "return_king")
```

这里需要为每个数据框添加一个变量（mutate添加一）来指示电影；这些信息存储在原始电子表格的其他位置。
顺便说一句，上面的三个代码片段除了范围和电影名称之外基本相同——
稍后我们将看到一种更好的方法来处理这种情况，即编写我们自己的函数，但目前这种方法已经可以正常工作。

一旦每个表都准备好了，我们就可以使用它们用bind_rows堆叠起来并整理结果：

``` r
lotr_tidy = 
  bind_rows(fellowship_ring, two_towers, return_king) |>        # 合并三个数据框，按行绑定（堆叠）
  janitor::clean_names() |>                                     # 清理列名，转换为小写加下划线格式
  relocate(movie) |>                                            # 将 movie 列移动到最前面
  pivot_longer(
    female:male,                                                # 将 female 到 male 这两列转换成长格式
    names_to = "gender",                                        # 新建变量 gender，存放原列名（female/male）
    values_to = "words"                                         # 新建变量 words，存放原列对应的数值
  ) |> 
  relocate(movie) |>                                            # 把 movie 列放到最前
  mutate(race = str_to_lower(race))                             # 把 race 列的字符串转换成小写

lotr_tidy   # 查看转换后的整洁数据框
```

    ## # A tibble: 18 × 4
    ##    movie           race   gender words
    ##    <chr>           <chr>  <chr>  <dbl>
    ##  1 fellowship_ring elf    female  1229
    ##  2 fellowship_ring elf    male     971
    ##  3 fellowship_ring hobbit female    14
    ##  4 fellowship_ring hobbit male    3644
    ##  5 fellowship_ring man    female     0
    ##  6 fellowship_ring man    male    1995
    ##  7 two_towers      elf    female   331
    ##  8 two_towers      elf    male     513
    ##  9 two_towers      hobbit female     0
    ## 10 two_towers      hobbit male    2463
    ## 11 two_towers      man    female   401
    ## 12 two_towers      man    male    3589
    ## 13 return_king     elf    female   183
    ## 14 return_king     elf    male     510
    ## 15 return_king     hobbit female     2
    ## 16 return_king     hobbit male    2673
    ## 17 return_king     man    female   268
    ## 18 return_king     man    male    2459

有了这种形式的数据，我们就可以更轻松地对电影之间进行比较、对三部曲中的种族进行汇总以及执行其他分析。

## Joining datasets

数据可能分布在多个相关表中，在这种情况下，需要在分析之前对它们进行合并或连接。
我们将只关注合并两个表的问题；
合并三个或更多表的步骤将使用相同的思路逐步完成。

有四种主要方式连接数据框x和y： 内部：保留同时出现在x和y中的数据（交集）
左：保留出现在x的 右：保留出现在y的
完整：保留出现在x和y的所有数据（并集）

左连接是最常见的，因为它们将较小表y中的数据添加到较大表x中，而不会从x中删除任何内容。
留大加小

举例来说，考虑FAS_pups.csv 和中的数据表FAS_litters.csv，它们通过Litter
Number变量关联。
前者包含每只幼崽独有的数据，后者包含每窝幼崽独有的数据。我们可以使用左连接将窝仔数据与幼崽数据合并；
这样做可以保留每只幼崽的数据，并在新列中添加数据。

（在重新审视这个例子时，请查看group窝仔数据集中的变量：
它编码了剂量和治疗日期！我们将在处理流程中修复这个混乱的部分。
我还将解决我的一个烦恼，即将性别编码为一个模棱两可的数字变量。）

``` r
pup_df = 
  read_csv(                                                          # 读取 CSV 文件 FAS_pups.csv
    "./data/FAS_pups.csv",                                           # 文件路径
    skip = 3,                                                        # 跳过前三行（可能是标题或说明）
    na = c("NA", "", ".")) |>                                        # 将 "NA"、空字符串和 "." 识别为缺失值
  janitor::clean_names() |>                                          # 清理列名，转为小写加下划线格式
  mutate(sex = recode(sex, `1` = "male", `2` = "female"))            # 把 sex 列中数字转换为文字 
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litter_df = 
  read_csv(                                                          # 读取 CSV 文件 FAS_litters.csv
    "./data/FAS_litters.csv",                                        # 文件路径
    na = c("NA", ".", "")) |>                                        # 将 "NA"、"."、空字符串识别为缺失值
  janitor::clean_names() |>                                          # 清理列名，转为小写加下划线格式
  relocate(litter_number) |>                                         # 将 litter_number 列移到最前面
  separate(group, into = c("dose", "day_of_treatment"), sep = 3) |>  # 按第3个字符后拆分group列，Con7拆成Con和7
  mutate(                                                            # 创建或修改列
    wt_gain = gd18_weight - gd0_weight,                              # 计算体重增加：第18天体重减去第0天体重
    dose = str_to_lower(dose)                                        # 将 dose 列转换为小写
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

join them!

``` r
fas_df = 
  left_join(pup_df, litter_df, by = "litter_number") |> # 按 litter_number 列左连接合并pup_df和litter_df两个数据框
  arrange(litter_number) |>                             # 按 litter_number 升序排列行
  relocate(litter_number, dose)                         # 将 litter_number 和 dose 列移到最前面

fas_df                                                  # 查看合并整理后的数据框
```

    ## # A tibble: 313 × 15
    ##   litter_number dose  sex    pd_ears pd_eyes pd_pivot pd_walk day_of_treatment
    ##   <chr>         <chr> <chr>    <dbl>   <dbl>    <dbl>   <dbl> <chr>           
    ## 1 #1/2/95/2     con   male         5      13        7       9 7               
    ## 2 #1/2/95/2     con   male         5      13        8      10 7               
    ## 3 #1/2/95/2     con   female       4      13        7       9 7               
    ## 4 #1/2/95/2     con   female       4      13        7      10 7               
    ## 5 #1/2/95/2     con   female       5      13        8      10 7               
    ## # ℹ 308 more rows
    ## # ℹ 7 more variables: gd0_weight <dbl>, gd18_weight <dbl>, gd_of_birth <dbl>,
    ## #   pups_born_alive <dbl>, pups_dead_birth <dbl>, pups_survive <dbl>,
    ## #   wt_gain <dbl>

我们在连接操作中明确指定了键。 默认情况下，
\*\_join连接操作中的函数dplyr会尝试根据待连接数据集中的变量名来确定键。
这通常足够了，但并非总是如此，额外的步骤来明确键将有助于您和其他人阅读您的代码。

请注意，连接对于操作员来说并不是特别合适，
\|\>因为它从根本上来说是非线性的：两个独立的数据集结合在一起，而不是逐步处理单个数据集。

最后一点，这些\*\_join函数与 SQL
语法非常相关，但强调数据分析中常见的操作。

\##学习评估： 此 zip
文件中的数据集包含本课程往年调查问卷的去识别化回复。两个数据集都包含唯一的学生标识符；第一个数据集包含关于操作系统的问题的回复，第二个数据集包含关于学位课程和
Git
经验的问题的回复。请编写一个代码块，导入并清理这两个数据集，然后将它们连接起来。

\##解决方案 我将两个数据集都放在了data我的 repo /
项目的目录中。下面的代码导入了这两个数据集，清理了变量名，并使用left_join、
inner_join和连接这两个数据集anti_join。

``` r
surv_os = 
  read_csv("data/surv_os.csv") |>                         # 读取 surv_os.csv 文件
  janitor::clean_names() |>                               # 清理列名为小写加下划线格式
  rename(id = what_is_your_uni,                           # 重命名列：将 "what_is_your_uni" 改为 "id"
         os = what_operating_system_do_you_use)           # 重命名列："what_operating_system_do_you_use" 改为 "os"
```

    ## Rows: 173 Columns: 2
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): What is your UNI?, What operating system do you use?
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
surv_pr_git = 
  read_csv("data/surv_program_git.csv") |>                        # 读取 surv_program_git.csv 文件
  janitor::clean_names() |>                                       # 清理列名
  rename(                                                         # 重命名多列
    id = what_is_your_uni,                                        # "what_is_your_uni" 改为 "id"
    prog = what_is_your_degree_program,                           # "what_is_your_degree_program" 改为 "prog"
    git_exp = which_most_accurately_describes_your_experience_with_git)#改为 "git_exp"
```

    ## Rows: 135 Columns: 3
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (3): What is your UNI?, What is your degree program?, Which most accurat...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
left_join(surv_os, surv_pr_git)                             # 左连接，保留 surv_os 所有行，匹配 surv_pr_git 的信息
```

    ## Joining with `by = join_by(id)`

    ## Warning in left_join(surv_os, surv_pr_git): Detected an unexpected many-to-many relationship between `x` and `y`.
    ## ℹ Row 7 of `x` matches multiple rows in `y`.
    ## ℹ Row 66 of `y` matches multiple rows in `x`.
    ## ℹ If a many-to-many relationship is expected, set `relationship =
    ##   "many-to-many"` to silence this warning.

    ## # A tibble: 175 × 4
    ##   id          os         prog  git_exp                                          
    ##   <chr>       <chr>      <chr> <chr>                                            
    ## 1 student_87  <NA>       MS    Pretty smooth: needed some work to connect Git, …
    ## 2 student_106 Windows 10 Other Pretty smooth: needed some work to connect Git, …
    ## 3 student_66  Mac OS X   MPH   Smooth: installation and connection with GitHub …
    ## 4 student_93  Windows 10 MS    Smooth: installation and connection with GitHub …
    ## 5 student_99  Mac OS X   MS    Smooth: installation and connection with GitHub …
    ## # ℹ 170 more rows

``` r
inner_join(surv_os, surv_pr_git)                            # 内连接，只保留两个表都存在的 id 行
```

    ## Joining with `by = join_by(id)`

    ## Warning in inner_join(surv_os, surv_pr_git): Detected an unexpected many-to-many relationship between `x` and `y`.
    ## ℹ Row 7 of `x` matches multiple rows in `y`.
    ## ℹ Row 66 of `y` matches multiple rows in `x`.
    ## ℹ If a many-to-many relationship is expected, set `relationship =
    ##   "many-to-many"` to silence this warning.

    ## # A tibble: 129 × 4
    ##   id          os         prog  git_exp                                          
    ##   <chr>       <chr>      <chr> <chr>                                            
    ## 1 student_87  <NA>       MS    Pretty smooth: needed some work to connect Git, …
    ## 2 student_106 Windows 10 Other Pretty smooth: needed some work to connect Git, …
    ## 3 student_66  Mac OS X   MPH   Smooth: installation and connection with GitHub …
    ## 4 student_93  Windows 10 MS    Smooth: installation and connection with GitHub …
    ## 5 student_99  Mac OS X   MS    Smooth: installation and connection with GitHub …
    ## # ℹ 124 more rows

``` r
anti_join(surv_os, surv_pr_git)                             # 反连接，保留 surv_os 中不在 surv_pr_git 的 id 行
```

    ## Joining with `by = join_by(id)`

    ## # A tibble: 46 × 2
    ##   id          os        
    ##   <chr>       <chr>     
    ## 1 student_86  Mac OS X  
    ## 2 student_91  Windows 10
    ## 3 student_24  Mac OS X  
    ## 4 student_103 Mac OS X  
    ## 5 student_163 Mac OS X  
    ## # ℹ 41 more rows

``` r
anti_join(surv_pr_git, surv_os)                             # 反连接，保留 surv_pr_git 中不在 surv_os 的 id 行
```

    ## Joining with `by = join_by(id)`

    ## # A tibble: 15 × 3
    ##    id         prog  git_exp                                                     
    ##    <chr>      <chr> <chr>                                                       
    ##  1 <NA>       MPH   "Pretty smooth: needed some work to connect Git, GitHub, an…
    ##  2 student_17 PhD   "Pretty smooth: needed some work to connect Git, GitHub, an…
    ##  3 <NA>       MPH   "Pretty smooth: needed some work to connect Git, GitHub, an…
    ##  4 <NA>       MPH   "Pretty smooth: needed some work to connect Git, GitHub, an…
    ##  5 <NA>       MS    "Pretty smooth: needed some work to connect Git, GitHub, an…
    ##  6 student_53 MS    "Pretty smooth: needed some work to connect Git, GitHub, an…
    ##  7 <NA>       MS    "Smooth: installation and connection with GitHub was easy"  
    ##  8 student_80 PhD   "Pretty smooth: needed some work to connect Git, GitHub, an…
    ##  9 student_16 MPH   "Smooth: installation and connection with GitHub was easy"  
    ## 10 student_98 MS    "Smooth: installation and connection with GitHub was easy"  
    ## 11 <NA>       MS    "Pretty smooth: needed some work to connect Git, GitHub, an…
    ## 12 <NA>       MS    "What's \"Git\" ...?"                                       
    ## 13 <NA>       MS    "Smooth: installation and connection with GitHub was easy"  
    ## 14 <NA>       MPH   "Pretty smooth: needed some work to connect Git, GitHub, an…
    ## 15 <NA>       MS    "Pretty smooth: needed some work to connect Git, GitHub, an…

left_join和inner_join都给出了关于“多对多”关系的警告。这表明匹配的变量不唯一，应该对警告进行调查。
在下面的代码中，我们来分析一下“第 7 行中x 与 中的多行匹配y”这个警告。

``` r
surv_os |> slice(7)
```

    ## # A tibble: 1 × 2
    ##   id         os        
    ##   <chr>      <chr>     
    ## 1 student_15 Windows 10

``` r
surv_pr_git |> filter(id == "student_15")
```

    ## # A tibble: 2 × 3
    ##   id         prog  git_exp                                                      
    ##   <chr>      <chr> <chr>                                                        
    ## 1 student_15 MPH   Pretty smooth: needed some work to connect Git, GitHub, and …
    ## 2 student_15 MPH   Pretty smooth: needed some work to connect Git, GitHub, and …

\##关于名字的简要说明 有一段时间，人们使用gatherand
spread而不是pivot_longer and pivot_wider。
新功能的更新是有充分理由的；gatherandspread仍将存在，但随着时间的推移，它们将变得不那么常见，您可能永远不会再看到它们。
