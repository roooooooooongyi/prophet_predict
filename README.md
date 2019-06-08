# prophet_predict
prophet模型理论、基本用法与实战

#### make_future_dataframe
---
make_future_dataframe(m, periods, freq = "day", include_history = TRUE)

获取用于预测的DataFrame

---
参数解释：
- 指明预测的期，periods是整数。
- freq可以选择：’day’, ’week’, ’month’, ’quarter’, ’year’, 1(1 sec), 60(1 minute) or 3600(1 hour)。
- include_history指定是否将历史数据显示在预测结果中。

注意：
返回的是一个dataframe


#### cross_validation
---
cross_validation(model, horizon, units, period = NULL, initial = NULL)
时间序列中的交叉验证

---
参数解释：
horizon：值得是从每个cutoff（截断点）开始往后预测的天数
units：horizon的单位，默认有："days","secs"等
period：每个cutoff的时间间隔
initial：第一个训练周期的时间长度，initial取值大小会直接影响**cutoff**的数量

原理解释：
很少有资料解释清楚这里cross_validation的原理，查了一下R语言的源码，有了如下理解：
（1）获得cross_validation的结果，并将结果（dataframe）根据距离cutoff的天数来做相应的变化
```{r}
df_cv = cross_validation(model, horizon =20, units = "days", period = 10, initial = 100)
```
![82684edfe1469a14e632ef8543bf931e.png](en-resource://database/3602:1)
图1
数据变化得
```{r}
df_m = df_cv
df_m$horizon <- df_m$ds - df_m$cutoff
df_m <- df_m[order(df_m$horizon), ]
```
图2中ds字段列出了部分距cutoff点为1的数据，换言之，这些数据的horizon都是1。
![6b45ba1459ee56787fca3cc2b57837e0.png](en-resource://database/3616:1)

图2
图3中列出的是距离cutoff距离为“2days”的数据，这些数据的horizon为2。
![32a51f67d22736086fa2a098b324601b.png](en-resource://database/3618:1)

图3
依次类推，可以按照horizon来将**cross_validation**中得到的数据重组，这里也可以看出每个cutoff的间距就是参数**period**。


#### performance_metrics
---
performance_metrics(df, metrics=NULL, rolling_window = 0.1)

通过交叉验证方法验证模型在各个指标上的表现

---

参数解释

- df：cross_validation返回的结果
- metrics： 可以选择mse， rmse， mae， mape， coverage
- rolling_window： 在每个滑动窗口中用于计算metrics的数据的比例

```{r}
performance_metrics(df = df_cv, metrics = c('rmse', 'mae'), rolling_window = 0.2)
```
执行上面代码可以得到：
![5b1dab0b82491fdc1be8fe06ce300137.png](en-resource://database/3614:1)

这里通过实例来解释3点：（1）为什么从4days开始（2）为什么总共有17条数据（3）具体怎么计算cross_validation
我们希望通过prophet模型来预测每个cutoff向后20天（horizon=20）的数据，而这里**rolling_window**设置为0.2，所以滑动窗口大小为4（20*0.2）。
![50246b5500ccef17b15df486b672f139.png](en-resource://database/3622:1)
所以有，总共可以得到17个滑动窗口（20-4+1），为了反应一个周期的过程，所以从4days开始，准确的说是从**horizon*rolling_window**开始，这就解释了前两点。

![20d7678e81890e2bccf755f29ee8fa14.png](en-resource://database/3624:1)
上图进一步解释了问题（3），也就是prophet中对时间序列预测问题进行交叉验证的方法。

受到这种评估时间序列预测方法的启发，可以将这种方法推广到一般时间序列的情况，见下图：
![cbcafb6f31cc9f5ef8214ca916674576.png](en-resource://database/3626:1)

这种评估方法最大的优势在于，可以**清晰的展现模型在每一个窗口内的预测情况**。
