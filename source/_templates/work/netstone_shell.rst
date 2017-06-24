


上线sql准备
=======================================

数据导入过程
---------------------------------------
1. 将石头网member表导出，存入大平台数据库中，别名为mem_net，将大平台member表导出，存入到石头网数据库中，别名为mem_plat;
2. 利用SQL将石头网比大平台多的用户导出，建立临时表mem_net_more(283K)；
3. 利用SQL将大平台比石头网多的用户导出，建立临时表mem_plat_more;
4. 将大平台和石头网**相同用户名相同手机号码**的条目筛出（我们认为是同一用户），建立临时表mem_net_plat_same（847K）;
5. 将大平台和石头网**相同用户名不同手机号码**的条目筛出（我们认为是不同用户），建立临时表mem_net_plat_diff（1818）;
6. 向大平台member表导入mem_net_more中数据;
7. 向石头网member表导入mem_plat_more中数据;
8. update大平台member表，将mem_net_plat_same中的密码更新到member表中；
9. 将大平台中所有手机号重复的数据找出来，将其所有的手机号都置为空（让其重新绑定）；
10. 将mem_net_plat_same大平台的id导入到石头网的p_member_id中；
11. 手动处理mem_net_plat_diff，需要客服支持。




相关SQL准备
---------------------------------------
1. 石头网比大平台多出的用户

.. code::

   SELECT 
    username, PASSWORD, enabled, account_non_expired, credentials_non_expired,
    reg_time AS create_date, realname AS real_name, phone AS mobile_number,
    qq AS qq_number, email  FROM mem_net n 
    WHERE n.username IN 
    (SELECT username FROM member) ;
    
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
      
3. 石头网和大平台相同用户名相同手机号码用户：

.. code::

    SELECT n.username, n.password, n.phone, m.MOBILE_NUMBER FROM mem_net n
    INNER JOIN member m ON n.username = m.USERNAME AND n.phone = m.MOBILE_NUMBER
    
4. 石头网与大平台用户名相同，手机号不同用户：

.. code::

    SELECT n.username, m.USERNAME, n.password, n.phone, m.MOBILE_NUMBER FROM mem_net n
    INNER JOIN member m ON n.username = m.USERNAME AND n.phone <> m.MOBILE_NUMBER

5. 将石头网比大平台多出来的用户导入到大平台中：

.. code::
      
    INSERT INTO member
    (username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, create_date, REAL_NAME, mobile_number, qq_number, email)
    SELECT username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, reg_time, realname, phone, qq, email
    FROM mem_net
    WHERE NOT EXISTS (SELECT * FROM member
    WHERE member.username=mem_net.username);
    
6. 将大平台多出来的用户导入到石头网中：

.. code::

    INSERT INTO member
    (username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, reg_time, realname, phone, qq, email)
    SELECT username, PASSWORD, enabled, account_non_expired, CREDENTIALS_NON_EXPIRED, create_date, REAL_NAME, mobile_number, qq_number, email
    FROM mem_plat
    WHERE NOT EXISTS (SELECT * FROM member
    WHERE member.username=mem_plat.username);
 
 
7. 将大平台与石头网帐号相同，手机号相同的账户，都改成石头网的密码：

.. code::

    UPDATE member m, mem_net n SET m.PASSWORD = n.password WHERE m.USERNAME = n.username AND m.MOBILE_NUMBER = n.phone
    
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

9. 将所有大平台重复的手机号码置为NULL：

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
      
10. 将大平台的member表的id导入到石头网表的p_member_id中：

.. code::

    UPDATE mem_net n, member m SET n.p_member_id = m.ID WHERE n.username = m.USERNAME;