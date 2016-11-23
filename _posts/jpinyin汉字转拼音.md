---
title: jpinyin汉字转拼音
date: 2016-11-23 11:22:43
categories: java
tags: 汉字转拼音
---
# 需求汉字转拼音需求
mysql方案
中文首字母排序
```
SELECT DISTINCT
    cityName,ELT(INTERVAL(CONV(HEX(left(CONVERT(cityName USING gbk),1)),16,10),
0xB0A1,0xB0C5,0xB2C1,0xB4EE,0xB6EA,0xB7A2,0xB8C1,0xB9FE,0xBBF7,
0xBFA6,0xC0AC,0xC2E8,0xC4C3,0xC5B6,0xC5BE,0xC6DA,0xC8BB,0xC8F6,
0xCBFA,0xCDDA,0xCEF4,0xD1B9,0xD4D1),
'A','B','C','D','E','F','G','H','J','K','L','M','N','O','P',
'Q','R','S','T','W','X','Y','Z') as letter
FROM
   nation
ORDER BY
    CONVERT (cityName USING gbk) COLLATE gbk_chinese_ci ASC
```

<!--more-->
大部分可以转换正确，发现一些缺陷
有缺陷 不能识别城市列表
```
亳州  Z
泸州  Z
漯河  Z
濮阳  Z
衢州  Z

正确字母应该是 
B
L
L
P
Q
```

github 三方类库，JPinyin是一个汉字转拼音的Java开源类库

# maven
```
  <dependency>
       <groupId>com.github.stuxuhai</groupId>
       <artifactId>jpinyin</artifactId>
       <version>1.1.8</version>
    </dependency>
```

# 用法
```
    @org.junit.Test
    public void toPinyinTest() throws Exception {
        String str = "浙江省杭州市";
        System.out.println(PinyinHelper.convertToPinyinString(str," ", PinyinFormat.WITH_TONE_MARK));
        System.out.println(PinyinHelper.convertToPinyinString(str," ", PinyinFormat.WITH_TONE_NUMBER));
        System.out.println(PinyinHelper.convertToPinyinString(str," ", PinyinFormat.WITHOUT_TONE));
        System.out.println(PinyinHelper.getShortPinyin(str));
        System.out.println(PinyinHelper.getShortPinyin(str).toUpperCase());
        System.out.println(PinyinHelper.getShortPinyin(str).toUpperCase().substring(0,1));
    }
//output
zhè jiāng shěng háng zhōu shì
zhe4 jiang1 sheng3 hang2 zhou1 shi4
zhe jiang sheng hang zhou shi
zjshzs
ZJSHZS
Z
```
# 解决上面mysql问题
```
System.out.println(PinyinHelper.getShortPinyin(str).toUpperCase());
System.out.println(PinyinHelper.getShortPinyin("亳州").toUpperCase().substring(0,1));
System.out.println(PinyinHelper.getShortPinyin("泸州").toUpperCase().substring(0,1));
System.out.println(PinyinHelper.getShortPinyin("漯河").toUpperCase().substring(0,1));
System.out.println(PinyinHelper.getShortPinyin("濮阳").toUpperCase().substring(0,1));
System.out.println(PinyinHelper.getShortPinyin("衢州").toUpperCase().substring(0,1));
//ouput
B
L
L
P
Q
```
OK


