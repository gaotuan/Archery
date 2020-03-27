
title: event 故障整理 
date: 2020-03-13 10:35:21

#### 原因：
权限调整的需要，删除某用户，导致该用户下的procedure，和event失效
#### 分析：
##### procedure：
- 问题解决：
方法一：修改存储过程的definer：
　　` update mysql.proc set definer='root@localhost' where db='db_name'; `
方法二：修改sql security：
　　sql secuirty的值决定了调用存储过程的方式，取值 ：definer或者invoker
　　definer:在执行存储过程前验证definer对应的用户如：root@%是否存在，以及是否具有执行存储过程的权限，若没有则报错
　　invoker:在执行存储过程时判断inovker即调用该存储过程的用户是否有相应权限，若没有则报错。
　　修改语法：`alter procedure pro_name sql security invoker;`

##### event：
- definer：指明该event的用户，服务器在执行该事件时，使用该用户来检查权限
- 默认用户为当前用户，即definer = current_user；
　　如果明确指明了definer，则必须遵循如下规则：
　　　　1.如果没有super权限，唯一允许的值就是自己当前用户，而不能设置为其他用户。
　　　　2.如果具有super权限，则可以指定任意存在的用户；如果指定的用户不存在，则事件在执行时会报错。
- 解决办法：` update mysql.EVENTS set definer='billing@%';`


#### 参考文章：
https://dev.mysql.com/doc/refman/8.0/en/stored-objects-security.html 　

> MySQL uses the following rules to control which accounts a user can specify in an object DEFINER attribute:
> > - if you have the SET_USER_ID or SUPER privilege, you can specify any account as the DEFINER value, although a warning is generated if the account does not exist. Additionally, as of MySQL 8.0.16, to set the DEFINER attribute for a stored object to an account that has the SYSTEM_USER privilege, you must have the SYSTEM_USER privilege.
> > - Otherwise, the only permitted account is your own, either specified literally or as CURRENT_USER or CURRENT_USER(). You cannot set the definer to some other account.　　

> Creating a stored object with a nonexistent DEFINER account may have negative consequences:
> > - For a stored routine, an error occurs at routine execution time if the SQL SECURITY value is DEFINER but the definer account does not exist.
> > - For a trigger, it is not a good idea for trigger activation to occur until the account actually does exist. Otherwise, the behavior with respect to privilege checking is undefined.
> > - For an event, an error occurs at event execution time if the account does not exist.
> > - For a view, an error occurs when the view is referenced if the SQL SECURITY value is DEFINER but the definer account does not exist.

> The SQL SECURITY Characteristic:
> > - Definitions for stored routines (procedures and functions) and views can include an SQL SECURITY characteristic with a value of DEFINER or INVOKER to specify whether the object executes in definer or invoker context. If a definition omits the SQL SECURITY characteristic, the default is definer context.
> > - Triggers and events have no SQL SECURITY characteristic and always execute in definer context. The server invokes these objects automatically as necessary, so there is no invoking user.

> > - Definer and invoker security contexts differ as follows:
> > > - A stored object that executes in definer security context executes with the privileges of the account named by its DEFINER attribute. These privileges may be entirely different from those of the invoking user. The invoker must have appropriate privileges to reference the object (for example, EXECUTE to call a stored procedure or SELECT to select from a view), but during object execution, the invoker's privileges are ignored and only the DEFINER account privileges matter. If the DEFINER account has few privileges, the object is correspondingly limited in the operations it can perform. If the DEFINER account is highly privileged (such as a root account), the object can perform powerful operations no matter who invokes it.
> > > - A stored routine or view that executes in invoker security context can perform only operations for which the invoker has privileges. The DEFINER attribute has no effect during object execution.
