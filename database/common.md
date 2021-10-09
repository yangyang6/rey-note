### 拷贝数据

##### 不同的数据库通过语法的拼接 concat

##### create table a as select * from    	    （只创建表的结构，而不会将原表的默认值一起创建）





##### Mysql循环插入语句代码

```mysql
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=100000)do
    insert into t values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();
```





innodb 每个数据页的大小默认是16kb

Mysql5.6引入了索引下推优化







### Mongo更新语句

```mongo
db.getCollection('202105_person_time').update({"courseType":"精品定制课","createTime":{$gt:"2021-08-24"}},{$set:{oldOrNewBoutique:'NEW_BOUTIQUE'}},{multi:true});
```



