---
title: 一些配置
date: 2021-03-29 20:45:53
tags: mysql
categories: mysql
---

mysql命令行工具: myscli(https://www.mycli.net/)

mysql共有三个密码侧率:
- LOW 密码长度>=8
- MEDIUM Length >= 8, 包含数字, 大小写字母和特殊符号
- STRONG Length >= 8, 处理MEDIUM还需要字典文件

mysql查看密码验证策略(默认可能是MEDIUM):
``` sql
SHOW VARIABLES LIKE 'validate_password%';
```
输出可能是以下:
```text
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password.check_user_name    | ON    |
| validate_password.dictionary_file    |       |
| validate_password.length             | 8     |
| validate_password.mixed_case_count   | 1     |
| validate_password.number_count       | 1     |
| validate_password.policy             | LOW   |
| validate_password.special_char_count | 1     |
+--------------------------------------+-------+
```
修改/关闭验证策略:

``` sql
// 修改为低等级
SET GLOBAL validate_password.policy=LOW;
// 修改为中等
SET GLOBAL validate_password.policy=MEDIUM;
// 关闭验证策略暂时
UNINSTALL COMPONENT "file://component_validate_password";
```
