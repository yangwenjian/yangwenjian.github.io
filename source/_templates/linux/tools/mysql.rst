


=================================================
MySQL Exercise
=================================================
Mysql作为免费的最流行数据库，当然收到程序员的关注。

MySQL配置
-------------------------------------------------
以下是基本配置：

::

    [client]
    port            = 3306
    socket          = /var/run/mysqld/mysqld.sock
    default-character-set = utf8

    # This was formally known as [safe_mysqld]. Both versions are currently parsed.
    [mysqld_safe]
    socket          = /var/run/mysqld/mysqld.sock
    nice            = 0

    [mysqld]
    # * Basic Settings
    default-storage-engine=INNODB
    max_allowed_packet=32M
    sql_mode = NO_AUTO_VALUE_ON_ZERO
    character-set-server=utf8
    collation-server=utf8_bin
    user            = mysql
    pid-file        = /var/run/mysqld/mysqld.pid
    socket          = /var/run/mysqld/mysqld.sock
    port            = 3306
    basedir         = /usr
    datadir         = /var/data/mysql
    tmpdir          = /tmp
    lc-messages-dir = /usr/share/mysql
    skip-external-locking
    # Instead of skip-networking the default is now to listen only on
    # localhost which is more compatible and is not less secure.
    #bind-address            = 192.168.2.84

    # * Fine Tuning
    #key_buffer             = 16M
    key_buffer_size         = 16M
    max_allowed_packet      = 16M
    thread_stack            = 192K
    thread_cache_size       = 8
    # This replaces the startup script and checks MyISAM tables if needed
    # the first time they are touched
    myisam-recover         = BACKUP
    max_connections        = 100
    #table_cache            = 64
    #thread_concurrency     = 10
    # * Query Cache Configuration
    query_cache_limit       = 1M
    query_cache_size        = 16M

    # * Logging and Replication
    # Both location gets rotated by the cronjob.
    # Be aware that this log type is a performance killer.
    # As of 5.1 you can enable the log at runtime!
    general_log_file        = /var/log/mysql/mysql.log
    log-error=/var/log/mysql.err
    #general_log             = 1
    #
    # Error logging goes to syslog due to /etc/mysql/conf.d/mysqld_safe_syslog.cnf.
    #
    # Here you can see queries with especially long duration
    #log_slow_queries       = /var/log/mysql/mysql-slow.log
    #long_query_time = 2
    #log-queries-not-using-indexes
    #
    # The following can be used as easy to replay backup logs or for replication.
    # note: if you are setting up a replication slave, see README.Debian about
    #       other settings you may need to change.
    #开启log-bin，后面的名称可以任意起
    log-bin=mysql-bin
    server-id = 1
    log_bin                 = /var/log/mysql/mysql-bin.log
    expire_logs_days        = 10
    max_binlog_size         = 100M    
    binlog_format           = row
    #binlog_do_db           = include_database_name
    #binlog_ignore_db       = include_database_name
    #
    # * InnoDB
    #
    # InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
    # Read the manual for more InnoDB related options. There are many!
    #
    # * Security Features
    #
    # Read the manual, too, if you want chroot!
    # chroot = /var/lib/mysql/
    #
    # For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
    #
    # ssl-ca=/etc/mysql/cacert.pem
    # ssl-cert=/etc/mysql/server-cert.pem
    # ssl-key=/etc/mysql/server-key.pem
    
    [mysqldump]
    quick
    quote-names
    max_allowed_packet      = 16M
    
    [mysql]
    #no-auto-rehash # faster start of mysql but no tab completition
    
    [isamchk]
    key_buffer              = 16M
    
    #
    # * IMPORTANT: Additional settings that can override those from this file!
    #   The files must end with '.cnf', otherwise they'll be ignored.
    #
    !includedir /etc/mysql/conf.d/

binlog
````````````````````````````````````````````````
binlog是mysql带的二进制日志文件，可以通过binlog来进行mysql的备份和恢复。

binlog_format支持三种格式：
1. statement 根据sql语句进行记录，这样每条语句都会生成一条记录，但也需要记录上下文信息；
2. row 根据每个条目的变化进行记录，这样在update和alter table的时候会产生大量记录；
3. mixed两种方法的结合，根据语句的内容自动选择使用哪种binlog方式，是一种优化的记录方式；
