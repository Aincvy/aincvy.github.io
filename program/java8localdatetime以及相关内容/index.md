# Java8LocalDateTime以及相关内容


## Java8LocalDateTime以及相关内容

在Java8中，jdk包含了新的 时间API:`LocalTime`,`LocalDate`,`LocalDateTime` 

先看一段代码吧

```java
//         LocalTime,  only time
LocalTime localTime = LocalTime.now();
LocalTime localTime1 = LocalTime.of(10,30,50);    // 参数列表: hour,minute,second

LocalTime nTime = localTime.plusHours(1);
LocalTime hour = localTime.get(ChronoField.HOUR_OF_DAY);
localTime.getHour();
localTime.getMinute();


//          LocalDate, only date
LocalDate localDate = LocalDate.now();
LocalDate localDate1 = LocalDate.of(2020,10,15);      // 参数列表: year, month, dayOfMonth

LocalDate year = localDate.getYear();
var year1 = localDate.get(ChronoField.YEAR);
var date = localDate.getDayOfYear();

LocalDate nDate = localDate.plusYears(1).plusDays(15);


//         LocalDateTime, date and time
LocalDateTime localDateTime = LocalDateTime.now();
LocalDateTime dateTime1 = LocalDateTime.of(2020,10,15,10,30);      // 参数列表: year, month, dayOfMonth, hour, minute
LocalDateTime dateTime2 = LocalDateTime.now(ZoneId.of("Asia/Shanghai"));     // 时区
System.out.println("dateTime2: " + dateTime2);    // dateTime2: 2020-05-21T17:58:24.494799200

LocalDateTime dateTime3 = localDateTime.plusYears(1).plusMonths(1).plusHours(-1).plusSeconds(-1000);
System.out.println("dateTime3: " + dateTime3);    // dateTime3: 2021-06-21T16:41:44.494799200

var localDateX = localDateTime.toLocalDate();
var localTimeX = localDateTime.toLocalTime();


var zoneOffset = OffsetDateTime.now().getOffset();
var instant = localDateTime.toInstant(zoneOffset);
System.out.println("instant: " + instant);    //  instant: 2020-05-21T09:58:24.494799200Z

long timestamp = localDateTime.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
var date1 = new Date(timestamp);
System.out.println("date1: " + date1);      // date1: Thu May 21 17:58:24 CST 2020
```



简单来说， 有下面的几个要点

- `LocalDate` 只表示日期， `LocalTime` 只表示时间， `LocalDateTime` 则表示日期+时间
- 三个类的API都是类似的， 或者说一样的
- 3个类的对象应该都是不可变的， 即 当调用`plusXXX()` 方法的时候， 旧对象并没有改动，返回的新对象是改动过的。 类似`String.subString()` 方法。
- `ZoneId` 表示时区， 用`of()`方法获取一个时区的实例。
- `now()` 方法获取到当前的时间内容。
- 这套API 比`Calendar`,`Date` 方便的很多。
- 可以用`localDateTime.toInstant(zoneOffset)` 语句转换成`Instant` 对象
- 可以用`instant.toEpochMilli()` 语句获取到 Unix时间戳， 然后转换成`Date` 对象
- `plusXXX()` 系列方法用于增加或者减少相应的时间单位
- `withXXX()` 系列方法用于设置单个属性（年，月，小时，分钟等） 



## joda-time 第三方时间库

让我们先看一段代码

```java
DateTime dateTime = DateTime.now();
dateTime.getWeekOfWeekyear();
dateTime.getDayOfMonth();
dateTime.getDayOfYear();

DateTime dateTime2 = dateTime.withWeekOfWeekyear(25).withDayOfMonth(1);
DateTime dateTime3 = dateTime.plusWeeks(1).plusDays(1);

Date date = dateTime.withTimeAtStartOfDay().toDate();
System.out.println(date);            // Thu May 21 00:00:00 CST 2020

var zone = DateTimeZone.forID("Asia/Shanghai");
DateTime dateTime4 = dateTime.withZone(zone).plusYears(1);
System.out.println(dateTime4);       // 2021-05-21T18:37:03.806+08:00

System.out.println(dateTime.toString("E MM/dd/yyyy HH:mm:ss.SSS"));    // 周四 05/21/2020 18:37:03.806
    
```

`joda-time` 和 `LocalDate` 系列类的方法差不多。

下面是`joda-time` 的一些特点

- 方法命名上， 没有采用`getHour()` 这类形式 ，而是使用了`getHourOfDay()` 这类形式，可读性更高了
- 更多的API 。除了常规的时间属性之外， 还有`withTimeAtStartOfDay()` 获取一天开始的时间，获取今天是今年的第几周 等等。  :+1::+1::+1:
- `toDate()` 直接转换成 `java.util.Date`类型 :+1::+1::+1:
- `org.joda.time.DateTimeZone` 用于表示时区。 
- `toString()` 方法可以直接格式化日期字符串
- `joda-time` 的时间也是具有不可变性， 各种方法返回的都是一个新的对象。



## 总结

常规情况下， 使用 `LocalDate` 相关的时间函数就可以满足需求了，但是如果需要更多好用的API的话，使用`joda-time` 也是一个不错的选择。 :joy::joy_cat:



#### 拓展阅读： 

https://www.ibm.com/developerworks/cn/java/j-jodatime.html

https://docs.oracle.com/javase/8/docs/api/java/time/LocalTime.html

https://docs.oracle.com/javase/8/docs/api/java/time/MonthDay.html


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/java8localdatetime%E4%BB%A5%E5%8F%8A%E7%9B%B8%E5%85%B3%E5%86%85%E5%AE%B9/  

