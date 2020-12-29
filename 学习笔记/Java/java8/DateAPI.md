# DateAPI 日期与时间

------

### LocalDate

![](.\img\LocalDate.png)

`包含年月日信息`

------

### LocalTime

![](.\img\LocalTime.png)

`包含时分秒信息`

------

### LocalDateTime

![](.\img\LocalDateTime.png)

`LocalDate+LocalTime = LocalDateTime`

### 替换Date

![](.\img\Date.PNG)

````java
Date now = new Date();
//废弃方法太多
//获取的时刻...
System.out.println(now);//Wed Oct 28 15:53:26 CST 2020
System.out.println(now.getYear());//120
System.out.println(now.getMonth());//9
//创建时间也是...
Date date = new Date(2020,10,1);

LocalDateTime now = LocalDateTime.now();
        System.out.println(now);//2020-10-28T16:00:58.361
//创建一个日期
LocalDateTime dateTime = LocalDateTime.of(2020, 10, 1, 16, 0, 0);
        System.out.println(dateTime);//2020-10-01T16:00
//修改时间--->生成全新的日期！
System.out.println(now.plusMonths(2));//2020-12-28T16:07:30.409
System.out.println(now.minusYears(1));//2019-10-28T16:07:30.409
System.out.println(now.withYear(2019));//2019-10-28T16:07:30.409
System.out.println(now.withHour(17));//2020-10-28T17:07:30.409
//格式化日期
System.out.println(now.format(DateTimeFormatter.BASIC_ISO_DATE));//20201028
System.out.println(now.format(DateTimeFormatter.ISO_DATE));//2020-10-28
System.out.println(now.format(DateTimeFormatter.ISO_DATE_TIME));//2020-10-28T16:13:33.668
System.out.println(now.format(DateTimeFormatter.ofPattern("yyyy/MM/dd")));//2020/10/28
//str->日期
System.out.println(LocalDateTime.parse("2020--10--01 12:31", DateTimeFormatter.ofPattern("yyyy--MM--dd HH:mm")));
//两个时间差
System.out.println(dateTime.until(now, ChronoUnit.DAYS));//27天
//用起来怎样也比Date方便吧
````

