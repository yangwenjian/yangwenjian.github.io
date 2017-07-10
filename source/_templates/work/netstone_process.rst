

石头网整合大平台详细流程
=================================================

准备工作
-------------------------------------------------
1. 在半夜进行，保证期间没有人进行注册，登录；
2. 时间比较紧，抓紧时间进行


具体流程
-------------------------------------------------
1.暂停线上石头网服务，将线上石头网数据库的member表和authorise表利用如下语句导出为sql文件，之后将member_net表中的`member`全部替换为`member_net`，将之前的两个sql导入到大平台数据库中；（15min）

.. code::
	
	石头网线上服务器中：
	mysqldump -uroot -p stw member > member_net.sql
	mysqldump -uroot -p stw authorise > authorise.sql
	sed -i 's/`member`/`member_net`/' member_net.sql
	gzip member_net.sql
	gzip authorise.sql
	大平台线上服务器中：
	gunzip member_net.sql.gz
	gunzip authorise.sql.gz
	将member_net建表语句第二行替换为`username` varchar(30) COLLATE utf8_bin NOT NULL,
	mysql -uroot platform < member_net.sql 
	mysql -uroot platform < authorise.sql
	为member_net表建立索引：
	


2. 通过投顾线上系统，建立投顾活跃用户表member_plat_effective，放入打平台数据库;（yyw提供，5min）

3. 查询石头网活跃用户，建立石头网活跃用户表member_net_effective;

.. code::

    大平台线上服务器中：
    mysqldump -uroot platform member_net -w "id in(select member_id from authorise where member_id= member_net.id and expire>=SYSDATE())" --lock-all-tables > member_net_effective.sql;
    sed -i 's/`member_net`/`member_net_effective`/' member_net_effective.sql
    mysql -uroot platform < member_net_effective.sql
   
4. 将大平台与石头网重复用户名部分中，是投顾活跃用户的列表筛选出来,备份结果，创建临时表member_stw，这样在登录中首先对其进行认证，如果不匹配再去大平台认证；

.. code::
   
   大平台线上服务器中：
   mysqldump -uroot platform member_net -w "username IN (SELECT username FROM member_plat_effective) AND n.username IN (SELECT username FROM member_net_effective)" --lock-all-tables > member_stw.sql;
   sed -i 's/`member_net`/`member_stw`/' member_stw.sql
   mysql -uroot platform < member_stw.sql
   (参考查询语句）SELECT n.* FROM member_net n INNER JOIN member_plat_effective p ON n.username = p.username INNER JOIN member_net_effective s ON p.username = s.username;
   
5. 对于大平台和石头网重复用户名的部分，大平台中保留所有投顾活跃用户，其余用户名相同的都改成石头网密码（1min）；

.. code::

   大平台上执行：
   UPDATE member m, member_net n SET m.PASSWORD = n.password WHERE m.USERNAME = n.username AND m.username not in (SELECT username from member_plat_effective);
	
6. 将石头网比大平台多的用户导入到大平台中，缺少的字段置默认值(17s)；

.. code::

   INSERT INTO member (username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, create_date, REAL_NAME, mobile_number, qq_number, email) SELECT username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, reg_time, realname, phone, qq, email FROM member_net WHERE NOT EXISTS (SELECT * FROM member WHERE member.username = member_net.username)
	
7. 将大平台比石头网多的用户导入到石头网的临时表中，缺少的字段置默认值；(last_logintime,role_id,note, mark加入默认值)

.. code::

   提前检查下用户名重复，因为大平台是大小写敏感的，但石头网member表不是：
   (参考查询语句）SELECT * FROM member m WHERE EXISTS (SELECT * FROM member m2 WHERE LOWER(m.username) = m2.username AND m.id != m2.id);
   INSERT INTO member_net(username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, reg_time, realname, phone, qq, email, last_logintime, role_id, note, mark) SELECT username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, create_date, REAL_NAME, mobile_number, qq_number, email, '2015-01-01', 1 , '', '' FROM  member WHERE id IN (SELECT mm.id FROM member mm WHERE mm.id NOT IN (SELECT m.id FROM member m LEFT JOIN member_net mn ON m.username = mn.username WHERE mn.username IS NOT NULL) );
   检查语句：
   select count(*) from member;
   select count(*) from member_net;
   （参考语句，慢）INSERT INTO member_net(username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, reg_time, realname, phone, qq, email, last_logintime, role_id, note, mark) SELECT username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, create_date, REAL_NAME, mobile_number, qq_number, email, '2015-01-01', 1 , '', '' FROM member WHERE NOT EXISTS (SELECT * FROM member_net WHERE member_net.username=member.username);
   
8. 将大平台的member表的id导入到石头网表的p_member_id中，并将member_net表导回石头网数据库；

.. code::

    UPDATE member_net n, member m SET n.p_member_id = m.ID WHERE n.username = m.USERNAME;
    mysqldump -uroot platform member_net > member_back_net.sql;
    sed -i 's/`member_net`/`member`/' member_back_net.sql
    gzip member_back_net.sql;
    石头网线上服务器中：
    gunzip member_back_net.sql.gz
    mysql -uroot -pLL1shitou71 stw < member_back_net.sql 

9. 将大平台手机号重复的置空：

.. code::
  
    UPDATE member m SET m.MOBILE_NUMBER = NULL WHERE m.ID IN (SELECT m1.id  FROM	(SELECT * FROM member WHERE MOBILE_NUMBER IS NOT NULL  AND MOBILE_NUMBER != '') m1 
    INNER JOIN  (SELECT  * FROM  member  WHERE MOBILE_NUMBER IS NOT NULL  AND MOBILE_NUMBER != '') m2  ON m1.MOBILE_NUMBER = m2.MOBILE_NUMBER   AND m1.USERNAME != m2.USERNAME  AND m1.MOBILE_NUMBER IS NOT NULL  AND m1.MOBILE_NUMBER != ''  AND m2.MOBILE_NUMBER IS NOT NULL  AND m2.MOBILE_NUMBER != '' GROUP BY m1.username)
    将无效手机号置空;
    UPDATE member SET mobile_number = NULL WHERE LENGTH(mobile_number) <>11 AND LENGTH(mobile_number) > 0;
    将所有的nickname为空的置为用户名
    UPDATE member m SET m.nickname = m.USERNAME WHERE nickname IS NULL;
	
善后工作
-------------------------------------------------
1. 检查几种类型的用户数据是否完整，大平台member表，石头网member表，投顾用户表；
2. 检查集中类型的用户是否能正常登录；

参考资料：
http://yangwenjian.github.io/build/html/_templates/work/netstone_shell.html