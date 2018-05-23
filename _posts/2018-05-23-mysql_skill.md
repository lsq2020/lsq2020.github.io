---
layout: post
title: 'mysql技巧'
subtitle: 'mysql获取汉字首字母'
date: 2018-05-23
categories: 技术
tags: mysql 开发
---

## mysql获取汉字首字母



DELIMITER $$

USE `test`$$

DROP FUNCTION IF EXISTS `getFirstHanZiCode`$$

CREATE DEFINER=`hjd`@`%` FUNCTION `getFirstHanZiCode`(in_string VARCHAR(100))
 RETURNS VARCHAR(100) CHARSET utf8
BEGIN
#定义临时字符串变量，用于接收函数中传递进来的字符串值，这里是in_string
DECLARE tmp_str VARCHAR(100) CHARSET gbk DEFAULT '' ;
#定义临时字符串变量，用于存放函数中传递进来的字符串值的第一个字符
DECLARE tmp_char VARCHAR(2) CHARSET gbk DEFAULT '';
#tmp_str的长度
DECLARE tmp_loc SMALLINT DEFAULT 0;
#初始化，将in_string赋给tmp_str
SET tmp_str = in_string;
#获取tmp_str最左端的首个字符，注意这里是获取首个字符，该字符可能是汉字，也可能不是。
SET tmp_char = LEFT(tmp_str,1);
#获取字符的编码范围的位置，为了确认汉字拼音首字母是那一个
SET tmp_loc=INTERVAL(CONV(HEX(tmp_char),16,10),
0xB0A1,0xB0C5,0xB2C1,0xB4EE,0xB6EA,0xB7A2,0xB8C1,0xB9FE,0xBBF7,0xBFA6,0xC0AC,
0xC2E8,0xC4C3,0xC5B6,0xC5BE,0xC6DA,0xC8BB,0xC8F6,0xCBFA,0xCDDA ,0xCEF4,0xD1B9,
0xD4D1);
#判断左端首个字符是多字节还是单字节字符，要是多字节则认为是汉字且作以下拼音获取
，要是单字节则不处理。如果是多字节字符但是不在对应的编码范围之内
，即对应的不是大写字母则也不做处理，这样数字或者特殊字符就保持原样了
IF (LENGTH(tmp_char)>1 AND tmp_loc>0 AND tmp_loc<24) THEN
SELECT ELT(tmp_loc,'A','B','C','D','E','F','G','H','J','K','L','M','N','O'
,'P','Q','R','S','T','W','X','Y','Z') INTO tmp_char; #获得汉字拼音首字符
END IF;
RETURN tmp_char;#返回汉字拼音首字符
END$$

DELIMITER ;

