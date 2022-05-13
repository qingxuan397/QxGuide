修改属性类型

```
修改表user10中的字段test,不能为空，默认为123
ALTER TABLE user10 MODIFY test CHAR(32) NOT NULL DEFAULT '123';
```

添加索引

语法如下：

1.PRIMARY KEY（主键索引）
    mysql>ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` ) 
2.UNIQUE(唯一索引)
     mysql>ALTER TABLE `table_name` ADD UNIQUE (`column` ) 
3.INDEX(普通索引)
    mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column` )
4.FULLTEXT(全文索引)
    mysql>ALTER TABLE `table_name` ADD FULLTEXT ( `column` )
5.多列索引
    mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )

