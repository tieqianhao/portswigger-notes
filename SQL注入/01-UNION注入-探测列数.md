# Lab：SQL injection UNION attack — determining the number of columns returned by the query

> 难度：PRACTITIONER ｜ 分类：SQL 注入 ｜ 状态：📖 解析（待本人实操验证）
> 官方页面：https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns

## 一、漏洞原理

商品筛选参数被直接拼接进 SQL 的字符串上下文：

```sql
SELECT * FROM products WHERE category = '<用户输入>' AND released = 1
```

闭合单引号后可以在原查询上追加自己的查询。`UNION` 有两个硬约束：前后两段查询**列数必须相同**、对应列的**数据类型必须兼容**。所以 UNION 注入的第一步不是取数据，是探列数——列数不对，后面所有 payload 都只会报错或返回空结果。

## 二、标准利用链

官方给了两种探列数的方法，任选其一。

**方法 A：ORDER BY 递增**

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

`ORDER BY n` 按结果集第 n 列排序，不需要知道列名。当 n 超过实际列数时数据库报错（如 `The ORDER BY position number 3 is out of range...`）；应用可能回显原始错误、通用错误页，或只是不返回结果——**只要响应出现可观察的差异**就能推断列数。

**方法 B：UNION SELECT NULL 递增**

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

列数不匹配时报错（如 `All queries combined using a UNION ... must have an equal number of expressions`）；匹配时结果集多出一行 NULL。用 `NULL` 是因为它可隐式转换为几乎所有常见类型，能先把"列数"这个变量单独测出来，避免同时撞上类型不兼容。

**注释符与数据库差异**：`--` 在 MySQL 后面必须跟空格，也可用 `#`（URL 里要写 `%23`）；Oracle 上每个 SELECT 都要带 `FROM`，用内置表 `dual`：`' UNION SELECT NULL FROM DUAL--`。

**探完列数后**：逐列把 NULL 换成字符串 `'a'`，哪一列不报错且回显里出现了 `a`，哪一列就能承载文本数据：

```
' UNION SELECT 'a',NULL,NULL--
' UNION SELECT NULL,'a',NULL--
```

## 三、我的利用步骤（实操后填写）

```
（此处贴 Burp 里实际发出的请求行与响应差异）
```

- 实测列数：`（填实测值）`
- 判断依据：`（填你观察到的报错 / 响应差异）`
- 卡住的地方与解法：`（填）`

## 四、防御视角

**参数化查询（prepared statement）是唯一彻底的解法**：SQL 结构与数据分离，输入永远被当作值而不是 SQL 代码片段。

几种常见的无效修法：

- 黑名单过滤 `'` / `UNION` —— 大小写、`UN/**/ION` 内联注释、URL 编码都能绕过；
- 转义单引号 —— 数字型注入（`WHERE id = <input>`）根本不需要引号；
- 只在 WAF 层拦 —— 参数在业务里被二次拼接照样出事。

`ORDER BY` 这类无法参数化的位置，正确做法是**白名单映射**（前端传 `sort=price`，后端映射到固定列名），而不是过滤。

## 五、一句话复盘

"UNION 注入的第一步是探列数，因为 UNION 要求两侧列数一致且类型兼容；我用 ORDER BY 递增或 UNION SELECT NULL 递增探出来，再用 NULL 逐列换字符串找能承载文本的列。修复上必须参数化，白名单只用于 ORDER BY 这类无法参数化的位置。"
