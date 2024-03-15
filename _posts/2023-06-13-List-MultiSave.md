---
title: 表单一键保存列表
date:  2023-06-13 15:30:00 +0800
category: Original
tags: Java
excerpt: 一键保存带有列表的表单，满足表单内列表项的批量增删改
---

### 背景

工作上正在实现渠道方式管理的功能，产品原型画的表单里有一个属性是物流商列表，用户可以随意添加删除修改物流商列表数据，在全部操作完成最终点击确定后才会把这个列表持久化到数据库。这时候有两种实现方案：

- 一种比较简单的做法是把这个属性存成json

  每次保存都会将新的列表转成json覆盖掉旧的列表，读取也是将json解析成原来的列表。但是这样做有一个弊端是：这项数据在需要查询，想要建立索引的时候会束手无策。因此这种做法只适用于单项数据的保存与展示。

- 另一种比较复杂的做法，也就是把这个列表存于一张新的数据库表：物流商表

  这种做法显然不能简单粗暴地使用json格式保存，而是需要将此列表单拎出来处理（存在另一张新表），每一行查询出来的已保存过的数据必须带有标识（ID），然后使用新列表对比原列表：

  - 没有ID标识的：新增
  - 有ID标识的：更新
  - 原来有ID标识，新列表没有的：删除

### 过程

为了解决我这种场景，第二种方案会更适合，我就很好奇这种不友好的代码怎么能够优雅地写出来，🤔思考......，一个原列表、一个新列表、新旧对比逻辑.....💡有了。废话不多说，直接上代码：

```java
    /**
     * 一键保存带有列表的表单，满足表单内列表数据的批量增删改
     *
     * @param list    旧列表
     * @param newList 新列表
     * @param column  标识列
     * @param add     新增动作
     * @param update  更新动作
     * @param delete  删除动作
     * @param <T>     列表数据
     * @param <R>     标识列类型
     */
    public static <T, R> void saveData(List<T> list,
                                       List<T> newList,
                                       Function<T, R> column,
                                       Consumer<List<T>> add,
                                       Consumer<List<T>> update,
                                       Consumer<List<R>> delete) {
        // 用List去存储需要操作的数据
        List<T> addList = new ArrayList<T>();
        List<T> updateList = new ArrayList<T>();
        // 获取列表的标识集合
        Set<R> ids = list.stream().map(column).collect(Collectors.toSet());
        Set<R> newIds = new HashSet<>();
        // 分类新增、更新
        for (T data : newList) {
            R id = column.apply(data);
            if (id == null) {
                addList.add(data);
            } else {
                newIds.add(id);
                if (ids.contains(id)) {
                    updateList.add(data);
                }
            }
        }
        // 分类删除
        List<R> deleteList = ids.stream()
                .filter(id -> !newIds.contains(id))
                .collect(Collectors.toList());
        // 执行增删改操作
        add.accept(addList);
        update.accept(updateList);
        delete.accept(deleteList);
    }
```

---

随机生成的简单物流商数据表示例：

| logistics\_id | logistics\_name | transport\_area | service\_rating |
| :------------ | :-------------- | :-------------- | :-------------- |
| 1             | 物流快递        | 北京            | 4.5             |
| 2             | 速运物流        | 上海            | 4.2             |
| 3             | 精准物流        | 广州            | 4.7             |
| 4             | 顺丰快递        | 深圳            | 4.8             |
| 5             | 邮政快递        | 成都            | 3.9             |

---

测试代码：

```java
  @Test
    public void save() {
        List<Logistics> list = logisticsService.list();
        // new list
        List<Logistics> newList = logisticsService.list();
        // 删掉ID为1的数据
        newList.stream()
                .filter(logistics -> Objects.equals(logistics.getLogisticsId(), 1))
                .findFirst().ifPresent(newList::remove);
        // 更新ID为2的评分为4.3
        newList.stream()
                .filter(logistics -> Objects.equals(logistics.getLogisticsId(), 2))
                .findFirst().ifPresent(logistics -> logistics.setServiceRating(BigDecimal.valueOf(4.3)));
        // 新增一行数据
        Logistics logistics = new Logistics();
        logistics.setLogisticsName("我的物流");
        logistics.setTransportArea("清远");
        logistics.setServiceRating(BigDecimal.valueOf(4.1));
        newList.add(logistics);
        // 保存
        saveData(list,
                newList,
                Logistics::getLogisticsId,
                logisticsService::saveOrUpdateBatch,
                logisticsService::saveOrUpdateBatch,
                logisticsService::removeByIds);
    }
```

分析：

- list = {ArrayList@16396}  size = 5
   0 = {Logistics@22184} "Logistics(logisticsId=1, logisticsName=物流快递, transportArea=北京, serviceRating=4.5)"
   1 = {Logistics@22185} "Logistics(logisticsId=2, logisticsName=速运物流, transportArea=上海, serviceRating=4.2)"
   2 = {Logistics@22186} "Logistics(logisticsId=3, logisticsName=精准物流, transportArea=广州, serviceRating=4.7)"
   3 = {Logistics@22187} "Logistics(logisticsId=4, logisticsName=顺丰快递, transportArea=深圳, serviceRating=4.8)"
   4 = {Logistics@22188} "Logistics(logisticsId=5, logisticsName=邮政快递, transportArea=成都, serviceRating=3.9)"
- newList = {ArrayList@16397}  size = 5
   0 = {Logistics@22177} "Logistics(logisticsId=2, logisticsName=速运物流, transportArea=上海, serviceRating=4.3)"
   1 = {Logistics@22178} "Logistics(logisticsId=3, logisticsName=精准物流, transportArea=广州, serviceRating=4.7)"
   2 = {Logistics@22179} "Logistics(logisticsId=4, logisticsName=顺丰快递, transportArea=深圳, serviceRating=4.8)"
   3 = {Logistics@22180} "Logistics(logisticsId=5, logisticsName=邮政快递, transportArea=成都, serviceRating=3.9)"
   4 = {Logistics@16398} "Logistics(logisticsId=null, logisticsName=我的物流, transportArea=清远, serviceRating=4.1)"

---

保存后的简单物流商数据表：

| logistics\_id | logistics\_name | transport\_area | service\_rating |
| :------------ | :-------------- | :-------------- | :-------------- |
| 2             | 速运物流        | 上海            | 4.3             |
| 3             | 精准物流        | 广州            | 4.7             |
| 4             | 顺丰快递        | 深圳            | 4.8             |
| 5             | 邮政快递        | 成都            | 3.9             |
| 6             | 我的物流        | 清远            | 4.1             |

### 总结

抽象后的工具类能够满足一次批量保存列表数据的任何场景，实际业务需要根据情况使用，其实如果表单里的列表不属于当前数据库表（而是存储在一张新的数据表），合理的做法应该是单拎出来，放在一个新的配置页面去操作数据行。
