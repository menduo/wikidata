---
categories: postgresql,sql
...

替换指定表的指定列中的指定字串。

```sql
UPDATE 表名 set 列名=REPLACE(列名,'字符串 1','字符串 2');
```


```sql
--- update by replace.
UPDATE table_a SET img_url=REPLACE(img_url,'a.com/','b.cn/'); 
UPDATE table_a SET detail=REPLACE(detail,'a.com/','b.cn/'); 

```
