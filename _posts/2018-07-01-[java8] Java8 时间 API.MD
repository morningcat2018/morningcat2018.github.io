---
layout:     post
title:      JDK8 新的时间日期 API
subtitle:   
date:       2019-11-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_637.jpg
catalog: true
tags:
    - java8
---


参考：https://docs.oracle.com/javase/8/docs/api/index.html

## 概念

#### 基本概念

- 时刻：所有计算机系统内部都用一个整数表示`时刻`，这个整数是距离格林尼治标准时间`1970年1月1日0时0分0秒`的毫秒数，可以理解时刻就是绝对时间，它与时区无关，不同时区对同一时刻的解读，即年月日时分秒是不一样的；
- 时区：同一时刻，世界上各个地区的时间可能是不一样的，具体时间与时区有关，一共有24个时区，北京是东八区；

#### 新的时间类

- Instant ：表示时刻，不直接对应年月日信息，需要通过时区转换
- LocalDateTime : 表示与时区无关的日期和时间信息，不直接对应时刻，需要通过时区转换
- LocalDate ：表示与时区无关的日期，与LocalDateTime相比，只有日期信息，没有时间信息
- LocalTime ：表示与时区无关的时间，与LocalDateTime相比，只有时间信息，没有日期信息
- ZonedDateTime ： 表示特定时区的日期和时间
- ZoneId / ZoneOffset ：表示时区

#### 时间 API 所在包

- java.time.*
- java.time.chrono.*
- java.time.format.*
- java.time.temporal.*
- java.time.zone.*

## 新的时间 API

#### 1. 人类识别时间

- LocalDateTime
- LocalDate
- LocalTime

以上类的实例均为`不可变的对象`，使用 ISO-8601 日历系统时间标准。

---

LocalDateTime 构造

```java
        LocalDateTime.parse("2018-11-28T11:15:07.142");
        LocalDateTime.parse("2018-12-10 16:30", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm"));

        LocalDateTime.of(LocalDate.now(), LocalTime.now());
        LocalDateTime.of(2018, 8, 20, 10, 8, 2);
        LocalDateTime.of(2018, Month.AUGUST, 20, 10, 8, 2);

        // ofInstant(Instant instant, ZoneId zone)
        LocalDateTime.ofInstant(Instant.now(), ZoneOffset.ofHours(8));
        // 东八区
        LocalDateTime.ofInstant(Instant.ofEpochMilli(System.currentTimeMillis()), ZoneOffset.ofHours(8));
        LocalDateTime.ofEpochSecond(System.currentTimeMillis() / 1000L, 0, ZoneOffset.ofHours(8));

        LocalDateTime.now();
        LocalDateTime.now(Clock.systemDefaultZone());
        LocalDateTime.now(ZoneOffset.systemDefault());
```

LocalDateTime 常用方法

```java
        LocalDateTime localDateTime = LocalDateTime.now();

        System.out.println(localDateTime.getYear());
        System.out.println(localDateTime.getMonthValue());
        System.out.println(localDateTime.getDayOfMonth());
        System.out.println(localDateTime.getHour());
        System.out.println(localDateTime.getMinute());
        System.out.println(localDateTime.getSecond());
        System.out.println(localDateTime.getNano());

        System.out.println(localDateTime.toLocalDate());
        System.out.println(localDateTime.toLocalTime());
        System.out.println(localDateTime.toInstant(ZoneOffset.ofHours(8)));
        // second
        System.out.println(localDateTime.toEpochSecond(ZoneOffset.ofHours(8)));

        // format
        System.out.println(localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));

        // 增加日期
        System.out.println(localDateTime.plusYears(1L));
        System.out.println(localDateTime.plusMonths(1L));
        System.out.println(localDateTime.plusDays(1L));
        System.out.println(localDateTime.plusWeeks(2L));
        System.out.println(localDateTime.plusHours(1L));

        // 减去日期
        System.out.println(localDateTime.minusDays(2));

        // 对比
        System.out.println(localDateTime.isAfter(LocalDateTime.parse("2018-11-28T11:15:07.142")));
        System.out.println(localDateTime.isBefore(LocalDateTime.parse("2050-01-01T00:00:00.000")));

        // 指定位置更换
        System.out.println(localDateTime.withYear(2021));
        System.out.println(localDateTime.withMonth(1));
        System.out.println(localDateTime.withDayOfMonth(5));
        System.out.println(localDateTime.withHour(8));
```


#### 2. 机器识别时间(时间戳，自 1970-01-01T00:00:00 到当前时刻的值)

- Instant 与时区无关

```java
        Instant.now(); // 默认获取 UTC 时区
        Instant.now(Clock.systemDefaultZone());
        Instant.ofEpochMilli(System.currentTimeMillis()); // ms
        Instant.ofEpochSecond(System.currentTimeMillis() / 1000); // s
        Instant.ofEpochSecond(1568892506L); // 时间戳


        Instant instant = Instant.now();
        System.out.println(instant.getEpochSecond()); // 秒
        System.out.println(instant.getNano()); // 只包含纳秒部分

        // OffsetDateTime atOffset(ZoneOffset offset)
        OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8)); // UTC+8 东八区

        // ZonedDateTime atZone(ZoneId zone) 
        ZonedDateTime zonedDateTime = instant.atZone(ZoneOffset.ofHours(8));
```

#### 3. 时间间隔

- TemporalAmount
- Period    日期间隔 (TemporalAmount 的子类)
- Duration  时间间隔 (TemporalAmount 的子类)

```java
        Period.between(LocalDate.parse("2018-01-01"), LocalDate.parse("2019-11-28")); // P1Y10M27D
        Period.of(1, 2, 3);
        Period.parse("P1Y2M3D"); // 1 year 2 month 3 day
        Period.ofYears(1);
        Period.ofMonths(2);
        Period.ofDays(3);

        Period period = Period.parse("P1Y2M3D");
        System.out.println(period);
        System.out.println(period.getYears());
        System.out.println(period.getMonths());
        System.out.println(period.getDays());
        System.out.println(period.get(ChronoUnit.MONTHS));
```

```java
        Duration.between(Instant.MIN, Instant.now());
        Duration.between(LocalDateTime.MIN, LocalDateTime.now());
        Duration.of(3, ChronoUnit.SECONDS);
        Duration.ofDays(3);
        Duration.ofHours(4);
        Duration.ofMinutes(5); //min
        Duration.ofSeconds(10); //s
        Duration.ofMillis(234); //ms
        Duration.ofNanos(200); //ns
```

#### 4. 时间校正器

- TemporalAdjuster
- TemporalAdjusters 工具类

```java
        // 下一个周日
        TemporalAdjuster temporalAdjuster = TemporalAdjusters.next(DayOfWeek.SUNDAY);
        LocalDateTime localDateTime = LocalDateTime.now().with(temporalAdjuster); 

        // 自定义校正器
        TemporalAdjuster t2 = temporal -> {
            return temporal;
        };
```

#### 5. 格式化

- DateTimeFormat

```java
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

        String dateStr = LocalDateTime.now().format(formatter);

        LocalDateTime localDateTime = LocalDateTime.parse("2018-08-10 10:30:09", formatter);
```


#### 6. 时区

- ZonedDate
- ZonedTime
- ZonedDateTime
- ZoneId

```java
        Set<String> zoneSet = ZoneId.getAvailableZoneIds();
        zoneSet.forEach(System.out::println);
```

指定时区的时间

```java
        LocalDateTime ldt = LocalDateTime.now(ZoneId.of("Asia/Shanghai"));
        // 2019-09-19T19:41:47.990

        LocalDateTime ldt2 = LocalDateTime.now();
        ZonedDateTime zdt = ldt2.atZone(ZoneId.of("Asia/Shanghai"));
        // 2019-09-19T19:41:47.995+08:00[Asia/Shanghai]

        System.out.println(ZoneId.systemDefault());
        // Asia/Shanghai
```

#### 其他

- Clock 与时区有关


## 时间转换

```java
import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.Date;
 
public class DateUtils {
 
    public static Date asDate(LocalDate localDate) {
        return Date.from(localDate.atStartOfDay().atZone(ZoneId.systemDefault()).toInstant());
    }
 
    public static Date asDate(LocalDateTime localDateTime) {
        return Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
    }
 
    public static LocalDate asLocalDate(Date date) {
        return Instant.ofEpochMilli(date.getTime()).atZone(ZoneId.systemDefault()).toLocalDate();
    }
 
    public static LocalDateTime asLocalDateTime(Date date) {
        return Instant.ofEpochMilli(date.getTime()).atZone(ZoneId.systemDefault()).toLocalDateTime();
    }
}
```