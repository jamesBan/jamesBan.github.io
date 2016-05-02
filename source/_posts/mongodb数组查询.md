title: mongodb数组查询
date: 2014-12-17 19:06:15
tags: mongodb
---

- 插入测试数据

```shell
>db.food.insert({"_id":1, "fruit":["apple","banana","peach"]})
>db.food.insert({"_id":2, "fruit":["apple","kumquat","orange"]})
>db.food.insert({"_id":3, "fruit":["cherry","banana","apple"]})
```

- $all(数组元素必须包含)

```shell
>db.food.find({"fruit":{$all:["apple", "banana"]}}) // 返回_id 为1,3的记录
```

- $in (数组元素为其中任意一个)

```shell
>db.food.find({"fruit":{$in:["apple,", "banana"]}}) // 1,2,3
```

单个$in 可以省略$in操作符 一下两条查询结果相同
```
>db.food.find({"fruit":"apple"}); //1, 2,3
>db.food.find({"fruit":{$in:["apple"]}});//1,2,3
```

- $size (数组长度)
```shell
>db.food.find({"fruit":{$size:3}}) // 1, 2,3
```
也可是使用 $where 来代替 
```
>db.food.find({$where:"this.fruit.length == 3"}); // 1, 2, 3
```

- $slice(对数组进行分片)
```
>db.food.find({}, {"fruit":{$slice:[1,1]}}) //只返回数组第一个元素
```

- 查询第一个元素为apple的记录
```shell
>db.food.find({"fruit.0": "apple"});
>db.food.find({$where:"this.fruit[0] == 'apple'"});
```

