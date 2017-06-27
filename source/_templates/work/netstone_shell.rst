


NetStone数据整合方案
=======================================
数据导入原则
---------------------------------------
1. 将两边数据进行整合，统一用户管理；
2. 保证各个业务系统的活跃用户不受到影响；
3. 将相关数据进行分析整理（各种情况的数量情况），再进行决策；

各项数据情况
---------------------------------------
* 石头网
    * 总用户数: **1196K+** ；
    * 活跃用户数: **186K+** ;
* 大平台
    * 总用户数: **925K+** ；
    * 活跃用户数: **3815** ;
* 交叉部分
    * 共有用户数（以用户名为标识）: **900K+** ;
    * 相同用户名相同手机号用户数: **847K** ;
    * 相同用户名不同手机号用户数: **1818** ;
    * 石头网活跃（在服务器内）与大平台用户名相同的用户数: **173K+** ;
    * 大平台活跃用户数与石头网用户名相同的用户数: **1947** ;
    * 石头网活跃与大平台活跃并且用户名相同的用户数: **1846** ;
    * 石头网活跃与大平台活跃并且用户名相同、手机号不同的用户数: **179** ;
* 其他部分
    * 大平台手机号重复的用户数: **166K+** ;
    * 上条数据中，其中活跃用户数: **1190** ;
    * 石头网用户中手机号重复的用户数: **265K+** ;
    * 上条数据中，其中活跃用户数: **36996** ;

数据导入过程
---------------------------------------
1. 将石头网member表导出，存入大平台数据库中，别名为mem_net，将authorise表导出存到大平台中;
2. 利用SQL将石头网比大平台多的用户导出，建立临时表mem_net_more(283K)；
3. 利用SQL将大平台比石头网多的用户导出，建立临时表mem_plat_more;
4. 将大平台和石头网相同用户名相同手机号码的条目筛出（我们认为是同一用户），建立临时表mem_net_plat_same（847K）;
5. 将大平台和石头网相同用户名不同手机号码的条目筛出（我们认为是不同用户），建立临时表mem_net_plat_diff（1818）;
6. 向大平台member表导入mem_net_more中数据;
7. 向石头网mem_net表导入mem_plat_more中数据;
8. update大平台member表，将mem_net_plat_same中的密码更新到member表中；
9. 将大平台中所有手机号重复的数据找出来，将其所有的手机号都置为空（让其重新绑定）；
10. 将mem_net_plat_same大平台的id导入到石头网的p_member_id中；
11. 将mem_net导回石头网中，替换原来的member表，此时和大平台用户数相等；
12. 手动处理mem_net_plat_diff，需要客服支持。


对业务造成影响
---------------------------------------
1. 相同用户名的石头网帐号和大平台帐号，有一个密码会失效，这取决于是否是活跃用户，如果两个系统都是，则保留石头网用户的密码；
2. 同时拥有大平台和石头网的不同帐号的用户，两个账户无法自动整合成一个账户，各自帐号携带各自的权限，需要手动处理；
3. 大平台中（整合后）相同手机号的用户将会都被置为NULL，导致丢失手机号信息，用excel备份这部分数据；
4. 用户进行注册时，手机号重复将提示无法注册，如被别人抢注后可能无法注册；
5. 今后开通账户时，与客户沟通时需要引导用户，如在大平台中存在将不需要再次注册，避免重复注册；
6. 大平台的注册方式下一步要改成手机号注册模式与石头网统一。


相关SQL准备
---------------------------------------
1. 石头网相关

.. code::

    石头网比大平台多出的用户数：
    SELECT username, PASSWORD, enabled, account_non_expired, credentials_non_expired, reg_time AS create_date, realname AS real_name, phone AS mobile_number,
    qq AS qq_number, email  FROM mem_net n WHERE n.username IN (SELECT username FROM member) ;
    石头网活跃用户数：

    
2. 大平台比石头网多的用户

.. code::

    SELECT 
      username, PASSWORD, enabled, account_non_expired, credentials_non_expired, 
      create_date AS reg_time, real_name AS realname, mobile_number AS phone,
      qq_number AS qq, email 
    FROM
      member n 
    WHERE n.username IN 
      (SELECT username FROM mem_net) ;
      
3. 石头网和大平台相同用户名相同手机号码用户:

.. code::

    SELECT n.username, n.password, n.phone, m.MOBILE_NUMBER FROM mem_net n
    INNER JOIN member m ON n.username = m.USERNAME AND n.phone = m.MOBILE_NUMBER
    
4. 石头网与大平台用户名相同，手机号不同用户:

.. code::

    SELECT n.username, m.USERNAME, n.password, n.phone, m.MOBILE_NUMBER FROM mem_net n
    INNER JOIN member m ON n.username = m.USERNAME AND n.phone <> m.MOBILE_NUMBER
    查询石头网活跃用户与大平台用户名重复的用户数:
    SELECT COUNT(*) FROM mem_net n INNER JOIN authorise a ON n.id = a.member_id INNER JOIN member m ON n.username = n.`username` WHERE a.expire > NOW();

5. 将石头网比大平台多出来的用户导入到大平台中:

.. code::
      
    INSERT INTO member
    (username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, create_date, REAL_NAME, mobile_number, qq_number, email)
    SELECT username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, reg_time, realname, phone, qq, email
    FROM mem_net
    WHERE NOT EXISTS (SELECT * FROM member
    WHERE member.username=mem_net.username);
    
6. 将大平台多出来的用户导入到石头网中:

.. code::

    INSERT INTO mem_net
    (username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, reg_time, realname, phone, qq, email)
    SELECT username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, create_date, REAL_NAME, mobile_number, qq_number, email
    FROM member
    WHERE NOT EXISTS (SELECT * FROM mem_net
    WHERE mem_net.username=member.username);
 
 
7. 将大平台与石头网帐号相同，手机号相同，石头网中非活跃用户数，大平台中是活跃用户数的账户，都改成石头网的密码:

.. code::

    UPDATE member m, mem_net n SET m.PASSWORD = n.password WHERE m.USERNAME = n.username AND m.MOBILE_NUMBER = n.phone
    石头网与大平台都活跃并且用户名相同的用户：
    SELECT COUNT(*) FROM member_effective m WHERE m.`USERNAME` IN  (SELECT n.`username` FROM mem_net n INNER JOIN authorise a ON n.id = a.member_id WHERE a.expire > NOW()) ;
    
    
8. 查询大平台手机号重复

.. code::

   SELECT COUNT(*)
  FROM
    (SELECT 
      m1.USERNAME, m2.USERNAME AS username2, m1.MOBILE_NUMBER,
      m2.MOBILE_NUMBER AS mobile_number2 
    FROM
      (SELECT 
	* 
      FROM
	member 
      WHERE MOBILE_NUMBER IS NOT NULL 
	AND MOBILE_NUMBER != '') m1 
      INNER JOIN 
	(SELECT 
	  * 
	FROM
	  member 
	WHERE MOBILE_NUMBER IS NOT NULL 
	  AND MOBILE_NUMBER != '') m2 
	ON m1.MOBILE_NUMBER = m2.MOBILE_NUMBER 
	AND m1.USERNAME != m2.USERNAME 
	AND m1.MOBILE_NUMBER IS NOT NULL 
	AND m1.MOBILE_NUMBER != '' 
	AND m2.MOBILE_NUMBER IS NOT NULL 
	AND m2.MOBILE_NUMBER != '' 
    GROUP BY m1.username) temp
    
    或者
    
    SELECT 
      COUNT(*) 
    FROM
      member 
    WHERE EXISTS 
      (SELECT 
	* 
      FROM
	(SELECT 
	  mobile_number,
	  COUNT(*) cnt 
	FROM
	  member 
	WHERE mobile_number IS NOT NULL 
	  AND LENGTH(TRIM(mobile_number)) > 0 
	GROUP BY mobile_number 
	HAVING COUNT(*) > 1) t 
      WHERE t.mobile_number = member.mobile_number) 
    ORDER BY mobile_number 

9. 将所有大平台重复的手机号码置为NULL:

.. code::

    UPDATE 
      member m 
    SET
      m.MOBILE_NUMBER = NULL 
    WHERE m.ID IN 
      (SELECT 
	m1.id 
      FROM
	(SELECT 
	  * 
	FROM
	  member 
	WHERE MOBILE_NUMBER IS NOT NULL 
	  AND MOBILE_NUMBER != '') m1 
	INNER JOIN 
	  (SELECT 
	    * 
	  FROM
	    member 
	  WHERE MOBILE_NUMBER IS NOT NULL 
	    AND MOBILE_NUMBER != '') m2 
	  ON m1.MOBILE_NUMBER = m2.MOBILE_NUMBER 
	  AND m1.USERNAME != m2.USERNAME 
	  AND m1.MOBILE_NUMBER IS NOT NULL 
	  AND m1.MOBILE_NUMBER != '' 
	  AND m2.MOBILE_NUMBER IS NOT NULL 
	  AND m2.MOBILE_NUMBER != '' 
      GROUP BY m1.username)
      
10. 其他细节:

.. code::

    将大平台的member表的id导入到石头网表的p_member_id中:
    UPDATE mem_net n, member m SET n.p_member_id = m.ID WHERE n.username = m.USERNAME;
    将无效手机号置空
    UPDATE member SET mobile_number = NULL WHERE LENGTH(mobile_number) <>11 AND LENGTH(mobile_number) > 0;
    将所有的nickname为空的置为用户名
    UPDATE member m SET m.nickname = m.USERNAME WHERE nickname IS NULL;
