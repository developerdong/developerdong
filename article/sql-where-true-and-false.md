# SQL WHERE语句中的“真”与“假”

本人在工作中遇到了一个WHERE语句条件判断的问题，在探究的过程中又学到了新的知识，觉得有必要把整个过程记录下来。

## 完整数据表

首先，利用W3Schools的[SQL Tryit Editor v1.6](https://www.w3schools.com/sql/trysql.asp?filename=trysql_asc)，可以得到一个Categories数据表：

<table><tbody><tr><th>CategoryID</th><th>CategoryName</th><th>Description</th></tr><tr><td>1</td><td>Beverages</td><td>Soft drinks, coffees, teas, beers, and ales</td></tr><tr><td>2</td><td>Condiments</td><td>Sweet and savory sauces, relishes, spreads, and seasonings</td></tr><tr><td>3</td><td>Confections</td><td>Desserts, candies, and sweet breads</td></tr><tr><td>4</td><td>Dairy Products</td><td>Cheeses</td></tr><tr><td>5</td><td>Grains/Cereals</td><td>Breads, crackers, pasta, and cereal</td></tr><tr><td>6</td><td>Meat/Poultry</td><td>Prepared meats</td></tr><tr><td>7</td><td>Produce</td><td>Dried fruit and bean curd</td></tr><tr><td>8</td><td>Seafood</td><td>Seaweed and fish</td></tr></tbody></table>

## 筛选出CategoryID大于等于3并且小于8的数据行

### 正确的WHERE语句

```sql
SELECT * FROM categories WHERE 3 <= categoryid AND categoryid < 8;
```

正确的WHERE条件很简单，我们想要的数据被筛选了出来：
<table><tbody><tr><th>CategoryID</th><th>CategoryName</th><th>Description</th></tr><tr><td>3</td><td>Confections</td><td>Desserts, candies, and sweet breads</td></tr><tr><td>4</td><td>Dairy Products</td><td>Cheeses</td></tr><tr><td>5</td><td>Grains/Cereals</td><td>Breads, crackers, pasta, and cereal</td></tr><tr><td>6</td><td>Meat/Poultry</td><td>Prepared meats</td></tr><tr><td>7</td><td>Produce</td><td>Dried fruit and bean curd</td></tr></tbody></table>

### 错误的WHERE语句产生了有意思的结果

```sql
SELECT * FROM categories WHERE 3 <= categoryid < 8;
```

<table><tbody><tr><th>CategoryID</th><th>CategoryName</th><th>Description</th></tr><tr><td>1</td><td>Beverages</td><td>Soft drinks, coffees, teas, beers, and ales</td></tr><tr><td>2</td><td>Condiments</td><td>Sweet and savory sauces, relishes, spreads, and seasonings</td></tr><tr><td>3</td><td>Confections</td><td>Desserts, candies, and sweet breads</td></tr><tr><td>4</td><td>Dairy Products</td><td>Cheeses</td></tr><tr><td>5</td><td>Grains/Cereals</td><td>Breads, crackers, pasta, and cereal</td></tr><tr><td>6</td><td>Meat/Poultry</td><td>Prepared meats</td></tr><tr><td>7</td><td>Produce</td><td>Dried fruit and bean curd</td></tr><tr><td>8</td><td>Seafood</td><td>Seaweed and fish</td></tr></tbody></table>

显然这样的WHERE语句是错误的，但其产生了有意思的结果——所有数据行都被筛选了出来。为什么会产生这样的结果，我决定继续探究。

## 为什么错误的WHERE语句筛选出了所有的数据

### 上限设为0

```sql
SELECT * FROM categories WHERE 3 <= categoryid < 0;
```

将上述错误的WHERE语句中的上限修改为0，结果一条数据都没有被筛选出来。

### 上限设为1

```sql
SELECT * FROM categories WHERE 3 <= categoryid < 1;
```

<table><tbody><tr><th>CategoryID</th><th>CategoryName</th><th>Description</th></tr><tr><td>1</td><td>Beverages</td><td>Soft drinks, coffees, teas, beers, and ales</td></tr><tr><td>2</td><td>Condiments</td><td>Sweet and savory sauces, relishes, spreads, and seasonings</td></tr></tbody></table>

可以看到结果只有两条，它们的CategoryID都小于3。实验进行到这里，我心里已经有了一个猜测：SQL像C语言一样，用1表示“真”，用0表示“假”，并且比较运算符的运算顺序从左到右。

上面的语句之所以返回了CategoryID小于3的数据，是因为只有当其小于3时，`3 <= categoryid`才会返回0，`3 <= categoryid < 1`才成立。

### 上限设为2

```sql
SELECT * FROM categories WHERE 3 <= categoryid < 2;
```

<table><tbody><tr><th>CategoryID</th><th>CategoryName</th><th>Description</th></tr><tr><td>1</td><td>Beverages</td><td>Soft drinks, coffees, teas, beers, and ales</td></tr><tr><td>2</td><td>Condiments</td><td>Sweet and savory sauces, relishes, spreads, and seasonings</td></tr><tr><td>3</td><td>Confections</td><td>Desserts, candies, and sweet breads</td></tr><tr><td>4</td><td>Dairy Products</td><td>Cheeses</td></tr><tr><td>5</td><td>Grains/Cereals</td><td>Breads, crackers, pasta, and cereal</td></tr><tr><td>6</td><td>Meat/Poultry</td><td>Prepared meats</td></tr><tr><td>7</td><td>Produce</td><td>Dried fruit and bean curd</td></tr><tr><td>8</td><td>Seafood</td><td>Seaweed and fish</td></tr></tbody></table>

当把上限修改为2时，发现和上限为8时得到的结果一样，都是全部数据。这是因为`3 <= categoryid`成立时返回1，不成立时返回0，但是不管成立或不成立，0和1都是小于2的，也就是说WHERE条件永远为真，所以返回了全部数据。

### 对猜测的验证

为了更准确地验证我的猜想，我执行了如下两个SQL语句：

1. `SELECT * FROM categories WHERE 3 <= categoryid = 0;`
2. `SELECT * FROM categories WHERE 3 <= categoryid = 1;`

产生的结果分别如下所示：

<table><tbody><tr><th>CategoryID</th><th>CategoryName</th><th>Description</th></tr><tr><td>1</td><td>Beverages</td><td>Soft drinks, coffees, teas, beers, and ales</td></tr><tr><td>2</td><td>Condiments</td><td>Sweet and savory sauces, relishes, spreads, and seasonings</td></tr></tbody></table>

<table><tbody><tr><th>CategoryID</th><th>CategoryName</th><th>Description</th></tr><tr><td>3</td><td>Confections</td><td>Desserts, candies, and sweet breads</td></tr><tr><td>4</td><td>Dairy Products</td><td>Cheeses</td></tr><tr><td>5</td><td>Grains/Cereals</td><td>Breads, crackers, pasta, and cereal</td></tr><tr><td>6</td><td>Meat/Poultry</td><td>Prepared meats</td></tr><tr><td>7</td><td>Produce</td><td>Dried fruit and bean curd</td></tr><tr><td>8</td><td>Seafood</td><td>Seaweed and fish</td></tr></tbody></table>

可以看到第一个SQL语句筛选出来的都是`3 <= categoryid`为“假”的数据；第二个SQL语句筛选出来的都是`3 <= categoryid`为真的数据，进一步验证了我的猜测。

如果修改判断条件，比如修改为使用`in`关键字：

1. `SELECT * FROM categories WHERE categoryid in (1, 2) = 1;`
2. `SELECT * FROM categories WHERE categoryid in (1, 2) = 0;`

结果和使用`<=`运算符是一样的。

## 0为“假”，非0为“真”

如上所示，当结果为“真”时，用1表示，当结果为“假”时，用0表示。那么，如果直接在WHERE语句后跟一个数字，是否0会被当成“假”，非0会被当成“真”呢？

执行如下SQL语句：

```sql
SELECT * FROM categories WHERE ?;
```

分别把?替换成不同的数字，结果如下表所示：

| 数字 | 结果 |
| ----: | :----: |
| 0 | 空 |
| -0 | 空 |
| 0.0 | 空 |
| -0.0 | 空 |
| 0.1 | 全部数据 |
| -0.1 | 全部数据 |
| 1 | 全部数据 |
| -1 | 全部数据 |
| 2.1 | 全部数据 |
| -2.1 | 全部数据 |

可以看到所有0值都被当成了“假”，非0值都被当成了“真”。

## 结论

综合以上过程，可以得到结论如下：

1. WHERE条件从左向右运算；
2. “真”被当成1，“假”被当成0；
3. 0被当成“假”，非0被当成“真”。

以上结论在不同的数据库产品中可能有不同，但思路应该是相同的。以后如果在执行SQL语句时返回了意料之外的结果，这次的探究结论也是一种参考。