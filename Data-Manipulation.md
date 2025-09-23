Data Manipulation
================
Chuqi
2025-09-18

使用dplyr动词和管道清理和整理数据。

示例： 在这个例子中，我将为我为Data Wrangling
I主题启动的repo/项目启动一个新的R
Markdown文件；这将使使用我在数据导入中编写的代码轻松加载示例数据集。

我们将再次使用tidyverse，所以我们一开始就会加载它。我们将查看大量输出，因此默认情况下，我将只打印每个提布尔的三行。最后，我们将关注FAS_litters.csv和FAS_pups.csv中的数据，因此我们将加载这些数据，并使用我们在数据导入中学到的内容清理列名。

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

import dataset

``` r
options(tibble.print_min = 3)

litters_df = 
    read_csv("./data/FAS_litters.csv", na = c("NA", ".", ""))
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
litters_df = 
    janitor::clean_names(litters_df)

pups_df = 
    read_csv("./data/FAS_pups.csv", na = c("NA", "."), skip = 3) #跳过前三行
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
pups_df = 
    janitor::clean_names(pups_df)
```

## select

对于给定的分析，您可能只需要数据表中列的子集；仅提取您需要的内容可以有助于整理杂乱，特别是当您有大型数据集时。使用select选择列。

您可以通过命名所有列来指定要保留的列

``` r
select(litters_df, group, litter_number, gd0_weight, pups_born_alive)
```

    ## # A tibble: 49 × 4
    ##   group litter_number gd0_weight pups_born_alive
    ##   <chr> <chr>              <dbl>           <dbl>
    ## 1 Con7  #85                 19.7               3
    ## 2 Con7  #1/2/95/2           27                 8
    ## 3 Con7  #5/5/3/83/3-3       26                 6
    ## # ℹ 46 more rows

您可以指定要保留的列范围 : 哪列到哪列

``` r
select(litters_df, group:gd_of_birth)
```

    ## # A tibble: 49 × 5
    ##   group litter_number gd0_weight gd18_weight gd_of_birth
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>
    ## 1 Con7  #85                 19.7        34.7          20
    ## 2 Con7  #1/2/95/2           27          42            19
    ## 3 Con7  #5/5/3/83/3-3       26          41.4          19
    ## # ℹ 46 more rows

## 您还可以指定要删除的列

删除

``` r
select(litters_df, -pups_survive)
```

    ## # A tibble: 49 × 7
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 Con7  #85                 19.7        34.7          20               3
    ## 2 Con7  #1/2/95/2           27          42            19               8
    ## 3 Con7  #5/5/3/83/3-3       26          41.4          19               6
    ## # ℹ 46 more rows
    ## # ℹ 1 more variable: pups_dead_birth <dbl>

您可以将变量重新命名为此过程的一部分

``` r
select(litters_df, GROUP = group, LiTtEr_NuMbEr = litter_number)
```

    ## # A tibble: 49 × 2
    ##   GROUP LiTtEr_NuMbEr
    ##   <chr> <chr>        
    ## 1 Con7  #85          
    ## 2 Con7  #1/2/95/2    
    ## 3 Con7  #5/5/3/83/3-3
    ## # ℹ 46 more rows

如果您只想重命名某物，您可以使用rename而不是select。这将重命名您关心的变量，并保留其他所有内容
select by rename

``` r
rename(litters_df, GROUP = group, LiTtEr_NuMbEr = litter_number)
```

    ## # A tibble: 49 × 8
    ##   GROUP LiTtEr_NuMbEr gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 Con7  #85                 19.7        34.7          20               3
    ## 2 Con7  #1/2/95/2           27          42            19               8
    ## 3 Con7  #5/5/3/83/3-3       26          41.4          19               6
    ## # ℹ 46 more rows
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

有一些方便的帮助函数进行select；阅读所有使用?select_helpers。我经常使用starts_with()ends_with()和contains()），特别是当变量用后缀或其他标准模式命名时
经常用！！！

``` r
select(litters_df, group, starts_with("gd"))
```

    ## # A tibble: 49 × 4
    ##   group gd0_weight gd18_weight gd_of_birth
    ##   <chr>      <dbl>       <dbl>       <dbl>
    ## 1 Con7        19.7        34.7          20
    ## 2 Con7        27          42            19
    ## 3 Con7        26          41.4          19
    ## # ℹ 46 more rows

rearange

``` r
select(litters_df, starts_with("gd"), group)
```

    ## # A tibble: 49 × 4
    ##   gd0_weight gd18_weight gd_of_birth group
    ##        <dbl>       <dbl>       <dbl> <chr>
    ## 1       19.7        34.7          20 Con7 
    ## 2       27          42            19 Con7 
    ## 3       26          41.4          19 Con7 
    ## # ℹ 46 more rows

``` r
select(litters_df, litter_number, everything())
```

    ## # A tibble: 49 × 8
    ##   litter_number group gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr>         <chr>      <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 #85           Con7        19.7        34.7          20               3
    ## 2 #1/2/95/2     Con7        27          42            19               8
    ## 3 #5/5/3/83/3-3 Con7        26          41.4          19               6
    ## # ℹ 46 more rows
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

我还经常使用iseverythingeverything()这对于在不丢弃任何东西的情况下重新组织列很方便：

``` r
select(litters_df, litter_number, pups_survive, everything())
```

    ## # A tibble: 49 × 8
    ##   litter_number pups_survive group gd0_weight gd18_weight gd_of_birth
    ##   <chr>                <dbl> <chr>      <dbl>       <dbl>       <dbl>
    ## 1 #85                      3 Con7        19.7        34.7          20
    ## 2 #1/2/95/2                7 Con7        27          42            19
    ## 3 #5/5/3/83/3-3            5 Con7        26          41.4          19
    ## # ℹ 46 more rows
    ## # ℹ 2 more variables: pups_born_alive <dbl>, pups_dead_birth <dbl>

relocate做类似的事情（有点像rename，因为它很方便，但并不重要）：

``` r
relocate(litters_df, litter_number, pups_survive)
```

    ## # A tibble: 49 × 8
    ##   litter_number pups_survive group gd0_weight gd18_weight gd_of_birth
    ##   <chr>                <dbl> <chr>      <dbl>       <dbl>       <dbl>
    ## 1 #85                      3 Con7        19.7        34.7          20
    ## 2 #1/2/95/2                7 Con7        27          42            19
    ## 3 #5/5/3/83/3-3            5 Con7        26          41.4          19
    ## # ℹ 46 more rows
    ## # ℹ 2 more variables: pups_born_alive <dbl>, pups_dead_birth <dbl>

在更大的数据集中，

最后，与dplyr中的其他函数一样，即使您只选择一列，select也会导出数据框架。大多数情况下，这很好，但有时您希望将向量存储在列中。要拉动单个变量，请使用pull。

只想要一个变量？

\##学习评估：在小狗数据中，选择包含猫砂数量、性别和PD耳朵的列。

``` r
select(pups_df, litter_number, sex, pd_ears)
```

    ## # A tibble: 313 × 3
    ##   litter_number   sex pd_ears
    ##   <chr>         <dbl>   <dbl>
    ## 1 #85               1       4
    ## 2 #85               1       4
    ## 3 #1/2/95/2         1       5
    ## # ℹ 310 more rows

``` r
filter(litters_df, pups_born_alive ==6)
```

    ## # A tibble: 6 × 8
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 Con7  #5/5/3/83/3-3       26          41.4          19               6
    ## 2 Con7  #4/2/95/3-3         NA          NA            20               6
    ## 3 Con7  #2/2/95/3-2         NA          NA            20               6
    ## 4 Mod7  #1/82/3-2           NA          NA            19               6
    ## 5 Low7  #112                23.9        40.5          19               6
    ## 6 Low8  #99                 23.5        39            20               6
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

``` r
filter(litters_df, pups_born_alive > 5)
```

    ## # A tibble: 41 × 8
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 Con7  #1/2/95/2             27        42            19               8
    ## 2 Con7  #5/5/3/83/3-3         26        41.4          19               6
    ## 3 Con7  #4/2/95/3-3           NA        NA            20               6
    ## # ℹ 38 more rows
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

``` r
filter(litters_df, pups_born_alive >= 5)
```

    ## # A tibble: 46 × 8
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 Con7  #1/2/95/2           27          42            19               8
    ## 2 Con7  #5/5/3/83/3-3       26          41.4          19               6
    ## 3 Con7  #5/4/2/95/2         28.5        44.1          19               5
    ## # ℹ 43 more rows
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

``` r
filter(litters_df, pups_born_alive <= 5)
```

    ## # A tibble: 8 × 8
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 Con7  #85                 19.7        34.7          20               3
    ## 2 Con7  #5/4/2/95/2         28.5        44.1          19               5
    ## 3 Con8  #2/2/95/2           NA          NA            19               5
    ## 4 Mod7  #3/82/3-2           28          45.9          20               5
    ## 5 Mod7  #5/3/83/5-2         22.6        37            19               5
    ## 6 Mod7  #106                21.7        37.8          20               5
    ## 7 Low7  #111                25.5        44.6          20               3
    ## 8 Low8  #4/84               21.8        35.2          20               4
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

``` r
filter(pups_df, sex == 1)
```

    ## # A tibble: 155 × 6
    ##   litter_number   sex pd_ears pd_eyes pd_pivot pd_walk
    ##   <chr>         <dbl>   <dbl>   <dbl>    <dbl>   <dbl>
    ## 1 #85               1       4      13        7      11
    ## 2 #85               1       4      13        7      12
    ## 3 #1/2/95/2         1       5      13        7       9
    ## # ℹ 152 more rows

``` r
filter(pups_df, pd_walk < 1, sex == 2)
```

    ## # A tibble: 0 × 6
    ## # ℹ 6 variables: litter_number <chr>, sex <dbl>, pd_ears <dbl>, pd_eyes <dbl>,
    ## #   pd_pivot <dbl>, pd_walk <dbl>

## mutate

mutate

有时您需要选择列；有时您需要更改它们或创建新的列。你可以使用mutate来做到这一点。

下面的示例创建了一个测量gd18_weight和gd0_weight之间的差异的新变量，并修改了现有的

``` r
mutate(
  litters_df,
  wt_gain = gd18_weight - gd0_weight,
  group = str_to_lower(group)
)
```

    ## # A tibble: 49 × 9
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 con7  #85                 19.7        34.7          20               3
    ## 2 con7  #1/2/95/2           27          42            19               8
    ## 3 con7  #5/5/3/83/3-3       26          41.4          19               6
    ## # ℹ 46 more rows
    ## # ℹ 3 more variables: pups_dead_birth <dbl>, pups_survive <dbl>, wt_gain <dbl>

这个例子中的几件事值得一点：

你的新变量可以是旧变量的函数 新变量按照创建顺序出现在数据集的末尾
你可以覆盖旧的变量 您可以创建一个新变量，并立即引用（或更改）它
创建一个完全符合你需求的新变量可能是一个挑战；你了解的功能越多，这就越容易。

\###学习评估：在小狗数据中： 创建一个从PD枢轴中减去7的变量
创建一个变量，即所有PD变量的总和 解决方案

\#arrange
与前述相比，安排非常简单。您可以根据一个或多个列中的值排列数据中的行：

``` r
arrange(litters_df, pups_born_alive)
```

    ## # A tibble: 49 × 8
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 Con7  #85                 19.7        34.7          20               3
    ## 2 Low7  #111                25.5        44.6          20               3
    ## 3 Low8  #4/84               21.8        35.2          20               4
    ## # ℹ 46 more rows
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

``` r
arrange(litters_df, desc(group), pups_born_alive)
```

    ## # A tibble: 49 × 8
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 Mod8  #7/82-3-2           26.9        43.2          20               7
    ## 2 Mod8  #97                 24.5        42.8          20               8
    ## 3 Mod8  #5/93/2             NA          NA            19               8
    ## # ℹ 46 more rows
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

``` r
arrange(litters_df, gd0_weight)
```

    ## # A tibble: 49 × 8
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 Mod7  #59                 17          33.4          19               8
    ## 2 Mod7  #62                 19.5        35.9          19               7
    ## 3 Con7  #85                 19.7        34.7          20               3
    ## # ℹ 46 more rows
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

我们看到了您将经常用于数据操作和清理的几个命令。您很少会单独使用它们。例如，假设您想加载数据，清理列名，删除pups_survive，并创建wt_gain。这种多步骤数据操作有几个选项：
·定义中间数据集（或在每个阶段覆盖数据） ·巢函数调用
以下是第一个选项的示例： do not do this！！！

``` r
litters_df_clean = 
  drop_na(
    mutate(
      select(
        janitor::clean_names(
          read_csv("./data/FAS_litters.csv", na = c("NA", ".", ""))
          ), 
      -pups_survive
      ),
    wt_gain = gd18_weight - gd0_weight,
    group = str_to_lower(group)
    ),
  wt_gain
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
litters_df_clean
```

    ## # A tibble: 31 × 8
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>
    ## 1 con7  #85                 19.7        34.7          20               3
    ## 2 con7  #1/2/95/2           27          42            19               8
    ## 3 con7  #5/5/3/83/3-3       26          41.4          19               6
    ## # ℹ 28 more rows
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, wt_gain <dbl>

## PIPES

下面，我们尝试第二个选项：

``` r
litters_df = 
  read_csv("./data/FAS_litters.csv", na = c("NA", ".", "")) |> 
  janitor::clean_names() |> 
  select(-pups_survive) |> 
  mutate(
    wt_gain = gd18_weight - gd0_weight,
    group = str_to_lower(group)) |> 
  drop_na(wt_gain)
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

所有三种方法都产生相同的数据集，但到目前为止，管道命令是最直接的。阅读\|\>的最简单方法是“then”；键盘快捷键是
Cmd + Shift +
M（Mac）。请注意，默认情况下，RStudio将插入“传统”管道%\>%，您可以通过全局首选项\>代码\>使用本机管道运算符更新到管道中的原生。以下有关\|\>和%\>%之间的差异的更多信息。

dplyr（以及大部分tidyverse）中的函数旨在与管道操作员顺利工作。默认情况下，管道将取一个函数调用的结果，并将其用作下一个函数调用的第一个参数；根据设计，dplyr中的函数将以tibble为输入，并返回一个tibble作为结果。因此，dplyr中的函数很容易在数据清理链中连接。

在大多数情况下（以及 tidyverse
的任何地方），你可以相信第一个论点是正确的，并对生活感到满意，但在某些情况下，你正在管道的东西并没有进入第一个论点。在这里，有必要使用占位符_来指示被管道的对象应该去哪里。例如，要在pups_born_alive上回归wt_gain，您可以使用：

``` r
litters_df |>
  lm(wt_gain ~ pups_born_alive, data = _) |>
  broom::tidy()
```

    ## # A tibble: 2 × 5
    ##   term            estimate std.error statistic  p.value
    ##   <chr>              <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)       13.1       1.27      10.3  3.39e-11
    ## 2 pups_born_alive    0.605     0.173      3.49 1.55e- 3

管道有限制。你不应该有太长的序列；没有处理多个输入和输出的好方法；而且不是每个人都会知道\|\>意味着什么或做什么。也就是说，与R用户只有前两个选项的日子相比，生活要好得多！

\##学习评估：编写一串命令： 加载小狗数据 清理变量名称
过滤数据，只包括有性1的小狗 去除PD耳朵变量
创建一个变量，表示PD枢轴是7天还是更长时间 解决方案

``` r
pups_df = 
  read_csv("./data/FAS_pups.csv", skip=3, na = c("NA", ".", "")) |> 
  janitor::clean_names() |> 
  filter(sex == 1)|> 
  select(-pd_ears)|>
  mutate(pd_pivot_ge7 = pd_pivot >=7)
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
pups_df
```

    ## # A tibble: 155 × 6
    ##   litter_number   sex pd_eyes pd_pivot pd_walk pd_pivot_ge7
    ##   <chr>         <dbl>   <dbl>    <dbl>   <dbl> <lgl>       
    ## 1 #85               1      13        7      11 TRUE        
    ## 2 #85               1      13        7      12 TRUE        
    ## 3 #1/2/95/2         1      13        7       9 TRUE        
    ## # ℹ 152 more rows

\|\>对比%\>%

\|\>和%\>%之间存在一些细微的差异，阅读这些差异是有帮助的，但在绝大多数情况下，这些差异实际上是可以互换的。虽然相似性和差异性可能很重要，但有几个点可能会更频繁地出现：

您将在网上看到很多%\>%。它首先被引入，在\|\>出现之前，在tidyverse代码中非常普遍。
原生管道\|\>是在R
4.1.0中引入的，它在4.2.0和4.3.0中发生了一些变化；它现在非常稳定，主要功能不太可能改变，但它不会让每个人都熟悉，也根本无法在旧版本的R中使用。
