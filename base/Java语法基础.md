日期时间

```
LocalDate today = LocalDate.now();
LocalDate.parse(today.toString(),DateTimeFormatter.ofPattern("yyyy-MM-dd"));
```





```java
LocalDateTime today = LocalDateTime.now();
today = today.minusMinutes(5);
DateTimeFormatter dtf2 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String strDate2 = dtf2.format(today);
```

