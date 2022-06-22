找出网站中前五最受欢迎的网页

cat /var/xx.log | 

​	awk '{print $7}' |

​	sort |

​	uniq -c |

​	sort -r -n | 

​	head -n 5





本地测试学习：

```sql
cat 3月班级系数部分老师修改Mysql库.sql |  awk '{print $14}' | sort -r -n | head -n 5
```



