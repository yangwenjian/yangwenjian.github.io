uthorise表复制到大平台中，将member表改为member_net别名；

2. 通过投顾系统建立投顾活跃用户表member_plat_effective;

3. 查询石头网活跃用户，建立石头网活跃用户表member_net_effective;
   SELECT n.username, n.password,n.phone FROM member_net n INNER JOIN authorise a ON a.member_id = n.id GROUP BY n.id;
   
4. 将大平台与石头网重复用户名部分中，是投顾活跃用户的列表筛选出来；
   SELECT n.* FROM member_net n INNER JOIN member_plat_effective p ON n.username = p.username INNER JOIN member_net_effective s ON p.username = s.username;
   
5. 这部分石头网用户单独提出放入临时表member_stw，将上一条的结果导出为SQL脚本，修改表名执行，这样在登录中首先对其进行认证，如果不匹配再去大平台认证；

6. 对于大平台和石头网重复用户名的部分，大平台中保留所有投顾活跃用户，其余用户名相同的都改成石头网密码；
   UPDATE member m, member_net n SET m.PASSWORD = n.password WHERE m.USERNAME = n.username AND m.username not in (SELECT username from member_plat_effective);
    
7. 将石头网比大平台多的用户导入到大平台中，缺少的字段置默认值；
   INSERT INTO member (username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, create_date, REAL_NAME, mobile_number, qq_number, email) SELECT username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, reg_time, realname, phone, qq, email FROM member_net WHERE NOT EXISTS (SELECT * FROM member WHERE member.username = member_net.username)

8. 将大平台比石头网多的用户导入到石头网的临时表中，缺少的字段置默认值；
   INSERT INTO member_net(username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, reg_time, realname, phone, qq, email) SELECT username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, create_date, REAL_NAME, mobile_number, qq_number, email FROM member WHERE NOT EXISTS (SELECT * FROM member_net
   WHERE member_net.username=member.username);

9. 将大平台手机号重复的置空；
   UPDATE member m SET  m.MOBILE_NUMBER = NULL  WHERE m.ID IN  (SELECT  m1.id  FROM (SELECT  * FROM  member WHERE MOBILE_NUMBER IS NOT NULL  AND MOBILE_NUMBER != '') m1 
   INNER JOIN  (SELECT  * FROM  member  WHERE MOBILE_NUMBER IS NOT NULL  AND MOBILE_NUMBER != '') m2  ON m1.MOBILE_NUMBER = m2.MOBILE_NUMBER   AND m1.USERNAME != m2.USERNAME  
   AND m1.MOBILE_NUMBER IS NOT NULL  AND m1.MOBILE_NUMBER != ''  AND m2.MOBILE_NUMBER IS NOT NULL  AND m2.MOBILE_NUMBER != '' GROUP BY m1.username)

10. 将大平台的member表的id导入到石头网表的p_member_id中；
    UPDATE member_net n, member m SET n.p_member_id = m.ID WHERE n.username = m.USERNAME;

11. 将member_net表导回到石头网中；
    
12. 将无效手机号置空;
    UPDATE member SET mobile_number = NULL WHERE LENGTH(mobile_number) <>11 AND LENGTH(mobile_number) > 0;
    
13. 将所有的nickname为空的置为用户名
    UPDATE member m SET m.nickname = m.USERNAME WHERE nickname IS NULL；
    
善后工作
-------------------------------------------------
1. 检查几种类型的用户数据是否完整，大平台member表，石头网member表，投顾用户表；
2. 检查集中类型的用户是否能正常登录；

参考资料：
http://yangwenjian.github.io/build/html/_templates/work/netstone_shell.html
