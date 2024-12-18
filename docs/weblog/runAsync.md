## JAVA 异步执行

### 背景

因有个统计页面，需要从多个数据源数据，并进行一些逻辑处理，最后汇总计算。之前是用一个sql写完所有的取数汇总逻辑的，因业务越加越多，性能也越来越慢，现在改为用java代码进行异步查询，统计。

###  java代码

````
// 异步调用loadTaCalendar方法
CompletableFuture<Void> loadTask1 = CompletableFuture.runAsync(() -> loadTaCalendar(vo)).handle((result, e) -> {
    if (e != null) {
        throw new RuntimeException("加载TA结转配置异常: " + e.getMessage());
    }
    return null;
});
// 异步调用loadTaInfo方法
CompletableFuture<Void> loadTask2 = CompletableFuture.runAsync(() -> loadTaInfo(vo)).handle((result, e) -> {
    if (e != null) {
    throw new RuntimeException("加载HKTA流水异常: " + e.getMessage());
    }
    return null;
});

 // 组合任务：将多个CompletableFuture组合在一起，形成复杂的异步流程
        CompletableFuture<Void> loadFuture = CompletableFuture.allOf(loadTask1, loadTask2);

 // 在加载任务完成后，执行计算任务
 loadFuture.thenRunAsync(() -> {
            // 创建多个异步任务，用来计算头寸
            // 计算TA头寸项
            CompletableFuture<List<koss_qdii_cash_calc>> calcTask1 = CompletableFuture.supplyAsync(() -> {
                try {
                    return calcTaCash();
                } catch (Exception e) {
                    throw new CompletionException(e);
                }
            });

            // 计算交易头寸项
            CompletableFuture<List<koss_qdii_cash_calc>> calcTask2 = CompletableFuture.supplyAsync(() -> {
                return calcTradeCash();
            });

            // 组合任务：将多个CompletableFuture组合在一起，形成复杂的异步流程
            CompletableFuture<Void> calcFuture = CompletableFuture.allOf(calcTask1, calcTask2);

            // 等待计算任务完成并处理结果
            calcFuture.whenComplete((result, throwable) -> {
                if (throwable != null) {
                    throw new CompletionException(throwable);
                }

                // 获取计算结果
                List<koss_qdii_cash_calc> ret1 = calcTask1.join();
                List<koss_qdii_cash_calc> ret2 = calcTask2.join();
           
                // 将每个任务的计算结果添加到m_listCashCalc中
                m_listCashCalc.addAll(ret1);
                m_listCashCalc.addAll(ret2);
            });

            try {
                // 等待所有计算任务完成
                calcFuture.join();
                // 批量插入
                insertCashCalc(vo);
            } catch (CompletionException e) {
                throw new RuntimeException("头寸计算失败: " + e.getMessage(), e);
            } catch (RuntimeException e) {
                throw new RuntimeException("批量插入失败: " + e.getMessage(), e);
            } catch (Exception e) {
                throw new RuntimeException(e.getMessage(), e);
            }
        });
 
 
```



```java
 /**
 * 计算TA头寸项
 */
 private List<koss_qdii_cash_calc> calcTaCash() throws Exception {
     List<koss_qdii_cash_calc> ret = new ArrayList<>();
     return ret;
 }

    
 /**
 * 计算交易头寸项
 */
 private List<koss_qdii_cash_calc> calcTradeCash() throws Exception {
     List<koss_qdii_cash_calc> ret = new ArrayList<>();
     return ret;
 }
````