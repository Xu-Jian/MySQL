🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  架构 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

⦿ 服务器    : GCE、VPS1、VPS2.
⦿ 操作系统  : CentOS7_x64
⦿ MySQL版本 : 5.5.56


⦿ GCE :  mysql-master、 wha
⦿ VPS1:  mysql-slaver、 wha、 proxySQL、 keepalived
⦿ VPS2:  mysql-slaver、 wha、 proxySQL、 keepalived  


|                        +-----------------+
|                        | SS 翻墙客户端   |
|                        +-----------------+
|                                 ↓
|            ↓-------------------------------------------↓
|        ProxySQL_1                                   ProxySQL_2
|        Keepalived                                   Keepalived        
|    +----------------------+                   +------------------------+
|    | VPS1: 23.105.192.96  |                   |  VPS2: 104.224.139.45  |
|    +----------------------+                   +------------------------+   
|                            \                   /
|                             \                 /
|                              \               /
|                               \             /
|                                 MySQL-Master
|                                     MHA 
|                        +-------------------------+    
|                        |    GCE: 35.194.128.92   |
|                        +-------------------------+    
|                       /                           \
|                      /                             \
|                     /                               \
|               MySQL-Slave_1                     MySQL-Slave_2
|                   MHA                               MHA
|        +----------------------+          +------------------------+
|        | VPS1: 23.105.192.96  |          |  VPS2: 104.224.139.45  |
|        +----------------------+          +------------------------+   









🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 本文简介 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸
🔸 环境要求
						❗️❗️ 主从服务器的操作系统和MySQL 版本必须一致.❗️❗️
						❗️❗️ 主从服务器的操作系统和MySQL 版本必须一致.❗️❗️

				• 确保两个vps 数据库版本一致; 我们这都是  mysql 5.5.56, 
				• 确保两个vps 系统架构一致;   Linux系统要么都是x86,要么都是x64
				• 最好系统也一样. 我这里两个都是Centos7_x64 .
				• 新手别跨版本折腾, 也别跨平台折腾.. 很多很多问题你解决不了的.

		❗️❗️我的三个服务器都是 centos7_x64; 数据库版本(mysql -V)都是 5.5.56❗️❗️


🔸 数据库备份: 主从热备 

		主从是为了数据备份.
		数据库是必须备份的. 手工备份很麻烦. 用主从热备可以让你自动实时备份.
		
		有了主从备份
		如果你的主服务器挂了,虽然你的整个网站还是会挂. 
		但是只要你在主数据库手动恢复恢复下数据. 
		或者先不管主数据库直接把数据库指向从数据库的IP,能非常快速的恢复服务.
		
		仅仅设置主从热备是不够的. 
		因为切换服务器.或者数据恢复过程都要花费时间的.这段时间内肯定无法正常提供服务.
		虽然有了备份. 但是可用性不高. 性能也不高.


🔸 数据库高性能: 读写分离

		增加性能. 也一定程度增加了可用性.
		就算主服务器挂了. 只是不能写入数据.但是还可以从从服务器读取数据.
		能一定程度的 保证项目运行.就像网站一样 
		用户一般都是从服务器读取网页数据. 很少写入数据到服务器.

				
🔸 故障切换(failover):   	Keepalived	

		当主服务器挂了后. 自动把从服务器变成主服务器.
		这样就可以读写了. 可用性非常高.
		这个是IP层面的 高可用.
		多个 服务器 争强一个IP.
		如果服务器都正常 那么就是负载均衡.
		如果某个服务器挂了. 那个服务器自然就抢不到IP了.不影响项目运行.


🔸 数据库高可用 MHA

		主从数据库. 主数据库是核心. 不能挂的, 万一主数据库挂了... 整个项目也就挂了.
		其实从数据库 和 主数据库的内容是一模一样的.
		MHA 技术可以在主数据库遇到故障的时候 自动把从服务器变成主服务器保证项目运行.




🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 主从简介 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

❗️❗️ MySQL进行实时数据同步，本质上是将mysql动作同步到Slave服务器，而不是对实质的数据进行同步。所以同步开始前，两端的数据要保持一致。 ️❗️
❗️❗️ MySQL进行实时数据同步，本质上是将mysql动作同步到Slave服务器，而不是对实质的数据进行同步。所以同步开始前，两端的数据要保持一致。 ️❗️
❗️❗️ MySQL进行实时数据同步，本质上是将mysql动作同步到Slave服务器，而不是对实质的数据进行同步。所以同步开始前，两端的数据要保持一致。 ️❗️

⦿ 简介
		主从设计可以让数据从一台服务器（Master）自动复制到多台独立的服务器（Slaves）
		主从结构有很多好处，这已经成为后端标配的架构，
		在MYSQL中实现这一功能的术语叫 - Replication 。


⦿ 主从架构原理
		主服务器 上的任何修改都会保存到 BinaryLog日志 里面.
		从服务器 开启一个进程 连接到主服务器并读取主服务器的 BinaryLog日志.
		从服务器 把读取到的主服务器日志 写到本地yy日志里面.
		从服务器 定时检查 本地yy日志, 
		从服务器 一旦发现yy日志有变化 就立刻用日志里面的命令更新从服务器数据库.

⦿ 复制方法
		mysql 就是执行一些语句, 然后数据库中的数据就变化了.
		比如我工资表中一万个用户. 我把每个用户工资加100 可以只用一条mysql命令搞定,
		实际数据库中 有10000 行的数据发生了变化.

		我们要同步两个数据库.  
		可以选择同步这个一个命令.          >只要同步一个命令.
		还能选择同步发生变化的10000行数据. >要同步10000行数据.
		这就是两种复制方法. mysql会自动选择一种. 两种方法肯定是只同步一个命令来的快.
		但是两种方法各有优缺点.这里不细说.


⦿ BinaryLog 作用: 
		• 主从复制
		• 备份恢复

⦿ 主从架构优点
		• 水平扩展，读写分离
				所有的增/删/改操作在Master上执行，
				所有的读操作在Slaves上执行，这样可以把并行压力分担到多个从库
		• 数据安全 - 从库可以随时停下来备份数据，而不必考虑服务不可用的问题。
		• 数据分析 - 在从库上分析数据，不会影响主库的性能
		• 远程数据分配 - 可以通过从库创建数据提供给远端的网站使用，而不必暴露主库


    ⦿ 冷备份
        数据库停止运行时候. 直接通过拷贝数据文件/目录 来进行备份!

    ⦿ 热备份
        数据库运行过程中 进行备份. 比较麻烦,主要分3步

        1. 把数据库设置成只读状态.
            mysql>flush tables with read lock;

            语句意思: 把只读锁应用到所有表.
            语句作用：关闭所有打开的表，并锁定所有的表，直到执行UNLOCK TABLES为止。

            备份数据库的时候. 数据库文件是不能有变化的,
            所以. 要暂时禁止数据库的写入功能, 只保留数据库读取功能.
            等数据备份好后 解锁数据库 恢复数据库写入功能..

            注意：在备份完成之前当前会话的连接不可退出，否则自动解锁。


        2. 数据文件导出(可以用 mysqldump命令, 也可以直接复制数据文件)
            最简单的导出就是：mysqldump db_name > db_back.db

        3. 取消数据库只读状态
            mysql>unlock tables;





🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 Mysql 日志简介 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸
🔸 日志

		日志文件是MySQL数据库的重要组成部分。
		mysql有几种不同的日志文件，通常包括错误日志文件，二进制日志等等。
		这些日志可以帮助我们定位mysqld内部发生的事件，数据库性能故障，记录数据的变更历史，用户恢复数据库等等。
		❗️❗️ 二进制日志，也叫binary log，是MySQL Server中最为重要的日志. ❗️❗️
		❗️❗️ 二进制日志，也叫binary log，是MySQL Server中最为重要的日志. ❗️❗️

		binlog 记录了所有的DDL和DML(除了数据查询语句)语句.
		也就是除了读取数据库不会记录.写入和更新mysql 的语句都会被记录到这个文件.
		所以从数据库可以从主数据库获取这个文件.然后根据日志里面的 mysql 语句再执行一遍就实现同步了.


🔸 查看 mysql 是否开启 binlog 日志

		mysql> show variables like 'log_%';
		+---------------------------------+------------------------------+
		| Variable_name                   | Value                        |
		+---------------------------------+------------------------------+
		| log_bin                         | ON       ➜ ➜ ON 就是开启了 |
		| log_bin_trust_function_creators | OFF                          |
		| log_error                       | /usr/local/mysql/var/gce.err |
		| log_output                      | FILE                         |
		| log_queries_not_using_indexes   | OFF                          |
		| log_slave_updates               | OFF                          |
		| log_slow_queries                | OFF                          |
		| log_warnings                    | 1                            |
		+---------------------------------+------------------------------+



🔸 查看 binlog 日志
		mysql> show master logs;
		+------------------+-----------+
		| Log_name         | File_size |
		+------------------+-----------+
		| mysql-bin.000015 |  14895107 |
		| mysql-bin.000016 |     23659 |
		| mysql-bin.000017 |   4993170 |
		| mysql-bin.000018 |   1931715 |
		| mysql-bin.000019 |    270456 |
		| mysql-bin.000020 |     52187 |
		| mysql-bin.000021 |     93211 |
		| mysql-bin.000022 |      9631 |
		| mysql-bin.000023 |     55917 |
		| mysql-bin.000024 |     35242 |
		| mysql-bin.000025 |    846340 |
		+------------------+-----------+



🔸 查看 master 状态，
		即最后(最新)一个binlog日志的编号名称，及其最后一个操作事件pos结束点(Position)值

		mysql> show master status;
		+------------------+----------+--------------+------------------+
		| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
		+------------------+----------+--------------+------------------+
		| mysql-bin.000025 |   854545 | ss-vps1      |                  |
		+------------------+----------+--------------+------------------+



🔸 刷新 log 日志，
      mysql> flush logs;		自此刻开始产生一个新编号的binlog日志文件
      注：每当mysqld服务重启时，会自动执行此命令，刷新binlog日志

			mysql> flush logs;
			Query OK, 0 rows affected (0.01 sec)

			mysql> show master status;
			+------------------+----------+--------------+------------------+
			| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
			+------------------+----------+--------------+------------------+
			| mysql-bin.000026 |      107 | ss-vps1      |                  |
			+------------------+----------+--------------+------------------+
			1 row in set (0.00 sec)


🔸 重置(清空)所有binlog日志
      mysql> reset master;



🔸 binlog 日志文件路径

		默认日志文件路径 cd /usr/local/mysql/var
		-rw-rw----  1 mysql mysql    23659 Jul 20 01:02 mysql-bin.000016
		-rw-rw----  1 mysql mysql  4993170 Jul 21 13:09 mysql-bin.000017
		-rw-rw----  1 mysql mysql  1931715 Jul 22 13:13 mysql-bin.000018
		-rw-rw----  1 mysql mysql   270456 Jul 22 16:37 mysql-bin.000019
		-rw-rw----  1 mysql mysql    52187 Jul 22 17:30 mysql-bin.000020
		-rw-rw----  1 mysql mysql    93211 Jul 22 18:48 mysql-bin.000021
		-rw-rw----  1 mysql mysql     9631 Jul 22 21:50 mysql-bin.000022

		❗️❗️ 当mysql停止或重启时，服务器会把日志文件记入下一个日志文件，mysql会在重启时生成一个新的日志文件，文件序号依次递增 ❗️❗️
		❗️❗️ 当mysql停止或重启时，服务器会把日志文件记入下一个日志文件，mysql会在重启时生成一个新的日志文件，文件序号依次递增 ❗️❗️



🔸 binlog 日志文件详细内容 一

		⦿ 语法:  mysql> show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];
               IN 'log_name'   指定要查询的binlog文件名(不指定就是第一个binlog文件)
               FROM pos        指定从哪个pos起始点开始查起(不指定就是从整个文件首个pos点开始算)
               LIMIT [offset,] 偏移量(不指定就是0)
               row_count       查询总条数(不指定就是所有行)

		⦿ 实例:
					SHOW BINLOG EVENTS in 'mysql-bin.000025' \G;
					*************************** 9108. row ***************************
						Log_name: mysql-bin.000025
									Pos: 855929
					Event_type: Query
						Server_id: 1
					End_log_pos: 856001
								Info: COMMIT
					*************************** 9109. row ***************************
						Log_name: mysql-bin.000025
									Pos: 856001
					Event_type: Query
						Server_id: 1
					End_log_pos: 856072
								Info: BEGIN
					*************************** 9110. row ***************************

						• Pos: 855929----------------> pos起始点:
						• Event_type: Query----------> 事件类型：Query
						• Server_id: 1 --------------> 标识是由哪台服务器执行的
						• End_log_pos: 856001 -------> pos结束点:856001(即：下行的pos起始点)



		⦿ 其他:

				A.查询第一个(最早)的binlog日志：
					mysql> show binlog events\G; 
			
				B.指定查询 mysql-bin.000021 这个文件：
					mysql> show binlog events in 'mysql-bin.000021'\G;

				C.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起：
					mysql> show binlog events in 'mysql-bin.000021' from 8224\G;

				D.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起，查询10条
					mysql> show binlog events in 'mysql-bin.000021' from 8224 limit 10\G;

				E.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起，偏移2行，查询10条
					mysql> show binlog events in 'mysql-bin.000021' from 8224 limit 2,10\G;



🔸 binlog 日志文件详细内容 二
		binlog是二进制文件，普通文件查看器cat more vi等都无法打开，必须使用自带的 mysqlbinlog 命令查看

		/usr/local/mysql/bin/mysqlbinlog /usr/local/mysql/var/mysql-bin.000025







🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 my.cnf 配置文件详解 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 配置文件路径查找方法
		• locate my.cnf
		• whereis my.cnf
		• sudo find / -name my.cnf
		• sudo mysql --help | grep my.cnf



🔸 my.cnf 优先级
		配置文件可能有好几个的!
		✘✘∙𝒗2 /usr ➜ sudo mysql --help | grep my.cnf
										order of preference, my.cnf, $MYSQL_TCP_PORT,
				• /etc/my.cnf 
				• /etc/mysql/my.cnf 
				• /usr/local/mysql/etc/my.cnf 
				• ~/.my.cnf
		虽然都叫my.cnf 但是文件路径不一样,代表文件优先级不一样!
		/etc/my.cnf 优先级最高... ~/.my.cnf 优先级最低
		所以我们 修改优先级最高的 /etc/my.cnf



🔸 my.cnf 文件结构
		my.cnf 分成好几部分(模块)如: [client]，[mysqld], [mysql]...
		一般只要修改 [mysqld] 模块里的参数就可以了. 



🔸 server_id 
		server_id=1 就是主节点 .   其他值都是从节点.  一个集群中.这个值绝对不能有重复.

		主节点:  server_id=1
		从节点1: server_id=1
		从节点2: server_id=1
		......


🔸 log_bin = log-bin
		❗️只要log_bin 没被注释掉. 那么 mysql的二进制日志文件就是开启的. 该参数值无所谓.只是生成日志文件名的前缀而已.❗️
		log_bin= log-bin 、 log_bin= mysql-bin 都可以.


🔸 binlog-do-db & replicate_do_db
		设置要同步的数据库!  如果不设置 默认是同步整个 mysql 所有数据库的!


		👁‍🗨 主从架构默认是同步所有数据库的!!!
			我们只同步其中的几个数据库:Django 和 DB2
			binlog-do-db=DB2           如果添加上这行 就说明只同步DB2数据库
			binlog-do-db=Django        如果要同步多个数据库 多添加几行就可以

			binlog_ignore_db = mysql    表示不同步某个数据库. 
																	一般设置需要同步的数据库就好了
																	不需要设置不需要同步的数据库

		要设置同步某个数据库的话.需要 主从配置文件里都进行设置.
		主服务器 和 从服务器的 设置参数稍微有区别.
		主配置文件用 binlog-do-db=数据库名
		从配置文件用 replicate_do_db=数据库名

		如: 我们只要同步 ss-vps1 这个数据库. 只要 
				• 主 my.cnf 配置文件 [mysqld] 模块下添加  binlog-do-db = ss-vps1
				• 从 my.cnf 配置文件 [mysqld] 模块下添加  replicate_do_db = ss-vps1



🔸 skip_name_resolve=ON
		作为MySQL调优的一部分，很多人都推荐开启skip_name_resolve。
		不开也无所谓. 大概就是禁止域名解析的 我们只是初级教程. 不开了. 详情自己谷歌去..


🔸 innodb_file_per_table=ON
		性能调优. 避免麻烦.不开算了


🔸 slave-skip-errors=all
		❗️❗️ mysql主从复制，经常会遇到错误而导致slave端复制中断，这个时候一般就需要人工干预，跳过错误才能继续 ❗️❗️
		❗️❗️ mysql主从复制，经常会遇到错误而导致slave端复制中断，这个时候一般就需要人工干预，跳过错误才能继续 ❗️❗️


⦿ 查看有多少事务组
		mysql> SHOW BINLOG EVENTS in 'mysql-bin.000003' from 9508\G
		一个row代表一个事务组


mysql>slave stop;

mysql>SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1        跳过一个事务 (pos)

mysql>slave start



在mysql中，对于sql的 binary log 他实际上是由一连串的event组成的一个组，即事务组。
我们在master上可以通过
SHOW BINLOG EVENTS 来查看一个sql里有多少个event。

SHOW BINLOG EVENTS in 'mysql-bin.000025' \G;



		跳过错误有两种方式：
		1.跳过指定数量的事务：
		mysql>slave stop;
		mysql>SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1        #跳过一个事务
		mysql>slave start

		2.修改mysql的配置文件，通过slave_skip_errors参数来跳所有错误或指定类型的错误
		vi /etc/my.cnf
		[mysqld]
		#slave-skip-errors=1062,1053,1146 #跳过指定error no类型的错误
		#slave-skip-errors=all #跳过所有错误


		跳过所有错误
		同步过程中总会有很多错误.. 
		可以选择跳过错误...

		如: 只对mydb记录了binlog,当在mydb库操作其它数据库的表，但该表在slave上又不存在时就出错了。













🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 Mysql 问题排除 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 问题排除
    mysql 启动不了.
    首先看你用的什么命令, 启动和重启是不一样的!  
    你用service mysql start   只会报一个错误
    你用service mysql restart 就会报两个错误,因为你没有启动.所以就没有停止
    说什么 PID file 没有找到.  这个pid file是数据库启动之后才会生成的.
    你没启动自然就没有 pid file了. 所以问题排错 千万不要走错路!
    一旦能启动后. 自然就有了 pid file .也就不会报这个错误了.


    我们来看看启动命令会报什么错误.
    👹 Starting MySQL...... ERROR! Manager of pid-file quit without updating file.

    排错前 首先想想  你干了什么导致起不来.
    比如你修改了什么配置文件.  那么改回去 再来启动看看 说不定就可以了...


    👹 The server quit without updating PID file (/usr/local/mysql/var/mail.0214.help.pid).
    删除 /usr/local/mysql/var/mail.0214.help.pid 这个文件就可以了.
		没这个文件怎么办....

		删除 /usr/local/mysql/var/ 文件夹下 任何带有 pid 的文件??

		sudo rm /usr/local/mysql/var/localhost.localdomain.pid


		👹 ✘✘∙𝒗1 ~ ➜ service mysql start
		Starting MySQL.Logging to '/usr/local/mysql/var/mail.err'.
		ERROR! The server quit without updating PID file (/usr/local/mysql/var/mail.pid).

		详细错误记录在/usr/local/mysql/var/mail.err这个文件中!!!
		cat /usr/local/mysql/var/mail.err


    👹 MySQL is not running, but lock file (/var/lock/subsys/mysql[FAILED]
    所以应该是my.cnf文件中配置出错了!


    👹 ERROR 1115 (42000) at line 2224: Unknown character set: 'utf8mb4'
        字符集问题.  当前数据库版本 可能不支持 导出的数据库文件里的字符集..

        当成创建数据库的时候 你指定了一个字符集. 当时你mysql 版本不支持那么个字符集.
        utf8mb4 是个 utf8 的 超集

        原来 vps1 的mysql 数据库字符格式是 utf8mn4的. 
        但是 vps2 的mysql 数据库字符格式是 utf8的 

        这样你要把vps1 里的某个数据库 导出到vps2上 ..就出问题了.



🔸 主从步骤简介

    最简单的复制模式就是一主一从的复制模式了，只需要三个步骤即可完成：
    （1）设置主服务器: 开启binlog，设置服务器id；

     (2) 如果主服务器已经有数据存在那么需要导出数据给从服务器.
         导出主服务器数据 > 上传给从服务器 > 数据导入进从服务器

    （3）建立一个从节点，设置服务器id；
    （4）将从节点连接到主节点上。










⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵  MySQL 主从热备实例  🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔸 主从目的

		GCE 上的 ss-vps1 数据库是三个ssr后端的总数据库.  
		万一GCE 挂了 那么三个SSR 都挂了. 不安全. 
		所以我们要给 数据库做备份. 
		以GCE为主数据库. vps1 和 vps2 为从数据库.


🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  主从热备流程简介  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸
修改主从的 my.cnf 配置文件.

server_id=1、

server_id=2、

server_id=3、


主锁表 
➜ 另开ssh 进行主数据库备份 
➜ 主节点的备份数据库上传给从节点 
➜ 从节点恢复主上传过来的数据库 
➜ 


主: 修改 my.cnf > 添加
server-id=1

主: 创建mysql帐号. 好让从连接到主的数据库.获取日志文件.







🔸 准备工作

		⦿ 所有服务器 系统备份
				折腾之前 必须备份三台服务器. 万一你搞坏了还能还原重新折腾.

		⦿ 所有服务器 mysql 配置文件备份
				cp /etc/my.cnf /etc/my.cnf.Newback && ll /etc | grep my.cnf

		⦿ 还原my.cnf配置 (可选 my.cnf折腾坏了 恢复用.)
				rm /etc/my.cnf && cp /etc/my.cnf.Newback /etc/my.cnf && ll /etc | grep my.cnf



🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 主服务器 GCE 设置 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 修改数据库配置文件 my.cnf 的 [mysqld]模块
		🔅 vi /etc/my.cnf
		binlog-do-db = ss-vps1    # 添加: 
		server_id = 1		          # 修改: 


🔸 重启数据库:   service mysql restart

⦿ 确认logbin是否开启
	从服务器就是通过 logbin 这个文件来实现同步的
		🔅 mysql -u root -pxujian0219
		mysql> reset master;                    ➜ 清空所有日志.
		mysql> FLUSH TABLES WITH READ LOCK;  ➜ 锁定数据表(禁止数据写入)
		mysql> show master status;
					+------------------+----------+--------------+------------------+
					| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
					+------------------+----------+--------------+------------------+
					| mysql-bin.000001 |      107 | ss-vps1      |                  |
					+------------------+----------+--------------+------------------+


				这个master 信息是一直在变的!!!!!  
				只有你锁住了数据库. 数据库不能写入数据的时候. master信息才是不变的.

					这里会显示主服务器的配置. 
					如我们现在要同步的数据库: ss-vps1
					File 和 Position 要记住. 这两个值 会在从服务器里面用到
					如果启用了 binarylog 那么File下有个文件. 没启用这里是空的

					❗️❗️这里必须先锁住数据库. 防止数据写入! 直到主从搭建好才能解锁主数据库.❗️❗️
					❗️❗️这里必须先锁住数据库. 防止数据写入! 直到主从搭建好才能解锁主数据库.❗️❗️
					❗️❗️这里必须先锁住数据库. 防止数据写入! 直到主从搭建好才能解锁主数据库.❗️❗️
							
					确保你导出数据库 到 配置好主从服务器 之间的过程中主服务器数据库没有数据写入.
					不然 就是你搭建好主从 也会导致两边数据库不一致.

					❗️ 这里不能退出 mysql . 另外开ssh窗口进行数据库导出.退出就会取消数据库锁定了!
					❗️ 这里不能退出 mysql . 另外开ssh窗口进行数据库导出.退出就会取消数据库锁定了!
					❗️ 这里不能退出 mysql . 另外开ssh窗口进行数据库导出.退出就会取消数据库锁定了!



🔸 新开ssh窗口 导出某数据库
		如果数据库不是空的. 那么需要先导出数据库到从服务器.
		然后再进行主从同步, 因为主从数据库只会同步之后的数据.
		在你设置主从数据库之前就存在的数据是不会被同步到从数据库的
		所以你必须先锁定主服务器的数据库. 
		然后导出数据库. 然后在从服务器进行数据库恢复.
		最好你才能 进行 主从配置. 配置好主从后再解锁主数据库的锁定.
		这样两个数据库的数据 才能一模一样.

			mysqldump db_name [tbl_name ...]  ➜ 导出某个数据库中的某些表
			mysqldump --databases db_name ... ➜ 导出某个数据库
			mysqldump --all-databases         ➜ 导出所有数据库

			❗️ 导出主从结构的数据库. 必须加上 --master-data 这个参数
			❗️ 导出主从结构的数据库. 必须加上 --master-data 这个参数
			❗️ 导出主从结构的数据库. 必须加上 --master-data 这个参数

			• 导出所有数据库: (我们只同步一个数据库.没必要导出所有的 )
					mysqldump -u root -p --all-databases --master-data > /root/dbdump.db

			• 导出某个数据库: (我们选择这个)	
						 cd && rm dbdump.db   可选. 不需要删除旧的数据.如果遇到同名的会直接覆盖的.
					🔅 mysqldump -u root -pxujian0219 --databases ss-vps1 --master-data > /root/dbdump.db && ls
						❗️❗️ 另外备份文件中会记录当前的binlog文件和position
						好像.现在就可以解除锁定表.
						其实在主服务器生成的日志文件是按顺序的.
						-rw-rw----  1 mysql mysql 14895107 Jul 20 00:42 mysql-bin.000015
						-rw-rw----  1 mysql mysql    23659 Jul 20 01:02 mysql-bin.000016
						-rw-rw----  1 mysql mysql  4993170 Jul 21 13:09 mysql-bin.000017
						-rw-rw----  1 mysql mysql  1931715 Jul 22 13:13 mysql-bin.000018
						-rw-rw----  1 mysql mysql   270456 Jul 22 16:37 mysql-bin.000019
						-rw-rw----  1 mysql mysql    52187 Jul 22 17:30 mysql-bin.000020
						-rw-rw----  1 mysql mysql    93211 Jul 22 18:48 mysql-bin.000021
						-rw-rw----  1 mysql mysql     9631 Jul 22 21:50 mysql-bin.000022
						-rw-rw----  1 mysql mysql    12848 Jul 22 22:02 mysql-bin.000023
						每当进行一次操作 .就会生成一个 新的 mysql-bin.xxxxxx 日志文件
						备份导出的时候 备份文件中有记录导出时候的日志文件的 mysql-bin.000019 
						主数据库导出后 .如果主数据库有新的写入. 那么就会有新的日志文件.
						当从服务器数据库恢复的时候.是恢复到 mysql-bin.000019 这这个状态的.
						如果你不设置主从 那么无所谓.
						如果你设置了主从.  当从数据库恢复好时候. 主数据库已经产生很多写入了.
						这些写入. 会自动同步到从服务器的!  因为从服务器的同步是从 mysql-bin.000019  开始的.






⦿ 导出之前要先锁表! 避免导出过程中有新数据写入数据库. 
		也就是之前 这个命令的作用 FLUSH TABLES WITH READ LOCK;

		导出完成后还不能解锁. 必须等到 主从配置好后 就可以用下面命令解锁了.
		mysql> UNLOCK TABLES; 
		或者直接用 exit; 命令退出mysql 命令行也可以达到解锁数据库的目的.


⦿ 数据库上传(备份/还原)
		上传给vps1、vps2   
		🔅  
		scp -P 2222 /root/dbdump.db root@23.105.192.96:/root/ && scp -P 2222 /root/dbdump.db root@104.224.139.45:/root/




🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 从服务器 vps1 设置 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

⦿ vps1 防火墙.
	 查看防火墙: iptables -L
	 清空防火墙: iptables -F
	 停止防火墙  systemctl disable firewalld
	 添加允许防火墙
		sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
		sudo firewall-cmd --reload


⦿ 修改 vi /etc/my.cnf 下的 [mysqld]模块
		replicate_do_db = ss-vps1    # 添加:  
		server_id = 3		          # 修改: 

⦿ 重启数据库.  service mysql restart

⦿ 停止 slave  (可选. 下一步导入不报错就不用执行)
		mysql -uroot -pxujian0219 
		stop slave;

⦿ 导入数据到从服务器

		🔅 mysql -u root -pxujian0219 < /root/dbdump.db

				这里会自动建立 ss-vps1数据库.不需要手动建立..

				因为只有就导出了一个数据库. 所以这里导入进从服务器的数据库也只有一个.
				如果你选择的是导出所有数据库,那么这里导入进从服务器的就有很多数据库了.

⦿ 导入测试
		现在你去 navicast 连vps1 的myslq数据库. 正常的话就多了一个ss-vps1数据库了.
		导入导出 和 主从架构完全没关系的.就是数据库备份还原而已
		下面我们来弄主从架构.


⦿ 进入vps1 mysql 命令行
		
		mysql -u root -pxujian0219
		mysql> stop slave;
				根据主服务器的信息 在从服务器上配置连接主服务器的各种信息
				+------------------+----------+--------------+------------------+
				| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
				+------------------+----------+--------------+------------------+
				| mysql-bin.000001 |      107 | ss-vps1      |                  |
				+------------------+----------+--------------+------------------+
		🔅
		mysql> change master to master_host='35.194.128.92',master_user='root',master_password='xujian0219',master_log_file="mysql-bin.000001",master_log_pos=107;

				👁‍🗨 下面字段都要修改成你自己的 
						master_host='35.194.128.92'        ➜ 主服务器 IP
						master_user='root'
						master_password='xujian0219'
						master_log_file="mysql-bin.000017" ➜ 主服务器的 log_file 名称
						master_log_pos=1465759;            ➜ 主服务器的 log_file 储存位置


		mysql> start slave;           ➜ 启动slave
		mysql> show slave status\G;   ➜ 检查slave状态
				在出现信息中找到Slave_IO_Running/Slave_SQL_Running,都为yes则成功了
				现在 一主一从架构就搭建好了!!!  
		mysql> exit;

		重启数据库.service mysql restart


⦿ 先不要测试
		现在还不可以在主服务器停止数据库锁定. 等vps2 也配置好后才能停止主服务器的锁定.
		一起测试!!! 

🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 从服务器 vps2 设置 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

⦿ 修改 vi /etc/my.cnf 下的 [mysqld]模块
		replicate_do_db = ss-vps1    # 添加
		server_id = 2		          # 修改: 

⦿ 重启数据库.  service mysql restart

⦿ 导入数据到从服务器
		🔅 mysql -u root -pxujian0219 < /root/dbdump.db

				这里会自动建立 ss-vps1数据库.不需要手动建立..

				因为只有就导出了一个数据库. 所以这里导入进从服务器的数据库也只有一个.
				如果你选择的是导出所有数据库,那么这里导入进从服务器的就有很多数据库了.

⦿ 导入测试
		现在你去 navicast 连vps2 的myslq数据库. 正常的话就多了一个ss-vps1数据库了.
		导入导出 和 主从架构完全没关系的.就是数据库备份还原而已
		下面我们来弄主从架构.

⦿ 从服务器数据库配置
		
		mysql -u root -pxujian0219
				stop slave;
				change master to master_host='35.194.128.92',master_user='root',master_password='xujian0219',master_log_file="mysql-bin.000001",master_log_pos=107;
				start slave;
				show slave status\G;   		确保Slave_IO_Running/Slave_SQL_Running,都为yes则成功了
		    exit;
		
		service mysql restart




🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 主从同步测试 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸
		 ❗️❗️ 两个从服务器主从设置好之后. 主服务器才能用 exit 命令退出锁定表.❗️❗️


🔸 主节点测试: GCE
		主服务器查看从节点:
		mysql -uroot -pxujian0219
				mysql> show slave hosts;
						+-----------+------+------+-----------+
						| Server_id | Host | Port | Master_id |
						+-----------+------+------+-----------+
						|         3 |      | 3306 |         1 |
						|         2 |      | 3306 |         1 |
						+-----------+------+------+-----------+
				mysql>exit;



🔸 从节点1测试: vps1  
		mysql -uroot -pxujian0219
				show slave status\G;
				❗️ ❗️ 没有错误才可以. 有错误的话. 需要跳过错误.❗️❗️
				❗️ ❗️ 没有错误才可以. 有错误的话. 需要跳过错误.❗️❗️
				我们这里出现了 Last_SQL_Errno: 1062 这个错误. 我们选择跳过所有的 1062类型的错误.

		从节点配置文件的添加.
		slave-skip-errors=1062   #跳过指定error类型的错误
		然后重启从节点的mysql.
		再进去看 slave 的状态. 没有error 就说明正常开启了.




🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 主从问题排除 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 同步失败

		第一次设置 可能从服务器状态两个都是 yes. 按理说 主从设置肯定正确的. 
		其实这时候还没开始同步.只是同步设置成功了而已. 还不算真正的完成主从设置.
		其实 在真正进行同步的时候会出很多错误的!
		主从同步.出错一般都是从节点出错.主节点很少出错.
		❗️❗️必须要任何时候. 进从服务器 看show slave status\G; 没有报错才算真正的主从设置完成 ❗️❗️


🔸 主服务器 状态
		• 同步日志文件
				我们先来看同步的关键文件. 主数据库的日志文件.
				这个文件的路径在哪呢. /etc/my.cnf 里有定义的.
				datadir = /usr/local/mysql/var
				这里确实有很多文件. 包括  mysql-bin.000018 这些从服务器要用到的日志文件. 

		• 主节点信息  show master status;
		• 查看从节点  show slave hosts;



🔸 从服务器 状态.
		mysql -u root -pxujian0219
		start slave;
		show slave status\G;

			👹 现在就有错误信息了. 
				🔸 跳过错误操作.
						主从同步错误是难免的!!!
						我们直接一般问题都是出在从节点上.
						我们直接修改从服务器的 my.cnf .直接跳过某些类型的错误

						slave-skip-errors=1062 
						跳过指定error no类型的错误

						然后重启数据库.
						再进入从服务器的 mysql 看状态 现在就 yesyes了.
						show slave status\G;


🔸 主从同步测试.

		停止主服务器的数据库锁定 (退出 mysql 命令行就可以.)

		然后去主数据库 随便找个表 新加个数据. 马上就同步到 从服务器了!!!

		下面重启 主服务器.  重启从服务器. 
		再进数据库新建数据看看 能不能同步..居然也可以....  done ....  ✔︎ 




⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵 主主热备 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

🔸 主从 & 主主
		MySQL主从模式:  对主的读写会实时同步到从节点. 对从的写入不会同步到主.
		MySQL主主模式:  对主从的读写. 都会在所有主从节点数据同步 .一般用作高容灾方案。


🔸 为什么要主主. 
		三个翻墙服务器. 
		翻墙是由 帐号验证+后端 实现翻墙的.
		最好么 验证和后端在一台服务器上. 速度块啊.


🔸 怎么设置主主.
		主主其实就是双向主从.
		我们在上面主从的基础上 进行主主设置.下面是大概操作.
		进行主主配置前.确保主从正确.能真正进行数据同步.

		⦿ 主从:
				主配置加 binlog-do-db = ss-vps1    (导出改数据库的操作到日志文件.)
				从配置加 replicate_do_db = ss-vps1 (从主的日志文件中. 同步该数据库部分的日志文件)

		⦿ 主主: 
				主配置加 binlog-do-db = ss-vps1   
								replicate_do_db = ss-vps1

				从配置加 replicate_do_db = ss-vps1
								binlog-do-db = ss-vps1



🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  vps1 设置  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 从节点 vps1 设置.

		⦿ 废话
			由于设置了主从. 现在所有数据库的数据都是一直的.
			所以主从设置里面的备份还原步骤就可以跳过了.
			我们只要开启 从服务器的 日志文件!然后设置要导出的数据库.
			再去主服务器 设置同步从服务器的数据库就可以了.

		
		⦿ 修改 vi /etc/my.cnf 下的 [mysqld]模块
				添加  binlog-do-db = ss-vps1
				意思就是把从节点的 ss-vps1 的数据库操作也写进日志文件中
				这样主服务器就可以获取从服务器的日志文件.来同步从节点的数据到主节点了.

		⦿ 重启数据库 service mysql restart


		🔅 mysql -u root -pxujian0219
		mysql> reset master;                    ➜ 清空所有日志.
		mysql> FLUSH TABLES WITH READ LOCK;     ➜ 锁定数据表(禁止数据写入)
		mysql> show master status;
					+------------------+----------+--------------+------------------+
					| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
					+------------------+----------+--------------+------------------+
					| mysql-bin.000001 |      107 | ss-vps1      |                  |
					+------------------+----------+--------------+------------------+




🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  GCE 操作  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 主节点 GCE 操作.
		⦿ 修改 vi /etc/my.cnf 下的 [mysqld]模块
			添加replicate_do_db = ss-vps1

		⦿ 重启数据库 service mysql restart


		⦿
		mysql -u root -pxujian0219
		mysql> stop slave;
		mysql> change master to master_host='23.105.192.96',master_user='root',master_password='xujian0219',master_log_file="mysql-bin.000001",master_log_pos=107;
						这里的 master_host 写 vps1 服务器的

		mysql> start slave;           ➜ 启动slave
		mysql> show slave status\G;   ➜ 检查slave状态

		如果这里没报错.
		那么我们会到 vps1 上. 去查看从节点主机.应该能看到一个的.
		mysql> show slave hosts;
		+-----------+------+------+-----------+
		| Server_id | Host | Port | Master_id |
		+-----------+------+------+-----------+
		|         1 |      | 3306 |         3 |
		+-----------+------+------+-----------+


		然后vps1 写入数据. 看看 gce 上会不会更新数据...
		不行啊... 
		继续排除..  从节点. 现在是 gce了..
				mysql> show slave status\G;   ➜ 检查slave状态
		操 果然又出现了 1062 错误!!!
		那么 我们去 gce 的 my.cnf 添加一行.
		slave-skip-errors=1062

		然后重启数据库 
		然后再去刷新 gce 的数据库. 就发现 vps1 里写入的数据.居然同步到gce上了.
		vps2 怎么办呢..
		虽然最初搭建主从. gce 有变化 会应用到 vps1 和vps2 .
		这分两种情况.
		手动修改gce 的数据 肯定会同步到 vps1 和vps2 
		但是如果 gce 是数据是 从vps1 里面同步过来的. 
		那么这vps1 同步过来的数据只会影响到 gce. 不会同步到vps2!!!
		所以如果我们想要实现 三个数据库 任何一个写入都能同步到别的数据库.
		这个就要求 一个从服务器 连接两个主服务器了. 不知道能不能实现.
		估计麻烦..我们还是算了. 只说两个的主主. 
		这里的从是不完整的. 这个从只能同步gce的写入数据. 不能同步vps2 的写入数据.

		MySQL 5.7.6 开始，添加了一个新特性：多源复制 Multi-Source Replication
		可以让你同时从多个master中并行复制，也就是形成了一种新的主从复制结构 一从多主
		我们用的是 mysql 5.5  就不折腾这个了!!!
		我们就到此为止. 两个主  + 一个半残废的从.
		用mysql 5.7 应该能实现 两个主 + 一个完整的从.

		其实我们残废的从对我们影响不大.
		我们ssr 翻墙注册账户都是在 gce 的mysql的.
		这个 gce的数据是可以同时同步到 vps1 和vps2的.


⦿ Next
		下面就开始读写分离设置.
		读写分离需要主从从结构.
		我们现在是 主主从结构.
		gce  是主、 vps1 是主、vps2 是gce的从
		所以读写分离这里我们就忽略 vps1 了. 拿 gce 和 vps2 来测试.
		因为读写分离. 主复制写. 从负责读(进制写) vps1 是主很多有数据写如 所以不能设置读写分离 























⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵 MySQL 读写分离实例 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔸 参考:

		★★★★ http://www.weare.net.cn/article/a501146c9be7fb7c7f80b253f2a5a8dc.html
		★★★★  http://www.cnblogs.com/zhoujinyi/p/6697141.html
		★★★★ http://seanlook.com/2017/04/10/mysql-proxysql-install-config/







🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  废话  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 从节点只读设置
		要把从服务器改成只读模式,
		也就是从服务器不能手动写入数据.只能通过主服务器同步来修改数据.
		之前的主从模式 从是可以写数据的. 只是这些数据不会同步到主而已.
		现在读写分离 需要禁止从写入数据.
		
		vps2: 
			vi /etc/my.cnf ➜ [mysqld] 模块下添加 ➜ read_only=ON
			然后重启数据库. service mysql restart



🔸 读写分离简介
		MySQL读写分离基本原理是让master数据库处理写操作，slave数据库处理读操作。
		master将写操作的变更通过之前设置的主从同步到各个slave节点。
		减轻主服务器的压力. 毕竟硬盘性能实在有限.还能增加冗余，提高可用性。
		设置读写分离 首先要设置主从配置. 主从是基础.
		读写方法实现方法有很多中. 主要就是通过中间件.


🔸 读写分离中间件
		如今最大的难处在于中间件太多了，不知道用哪个! 我们选择 proxySQL

		1. proxySQL
		2. maxscale 
		3. 阿里云的RDS（中间层支持读写自动分离）。
		上面这些是第一梯队的作品，其他的不用看
		• MySQL官方提供的 MySQL Proxy (不更新好久了...)
		• mycat

		⦿ ProxySQL
		　ProxySQL是个高性能代理
			ProxySQL 实际上是在客户端请求与MySQLServer之间建立了一个连接池。
			所有客户端请求都是发向proxySQL 然后经由MySQLProxy进行相应的分析，
			判断出是读操作还是写操作，分发至对应的MySQLServer上。
			对于多节点Slave集群，也可以起做到负载均衡的效果.

			•	2017-07-21-10 最新版本 1.3.7    官网: http://www.proxysql.com/




🔸 ProxySQL 相关文件

			• /etc/proxysql.cnf                ➜ 配置文件,一些启动选项，sqlite的数据目录等等
			• /var/lib/proxysql/ProxySQL.db    ➜ SQLITE的数据文件
			• /var/lib/proxysql/ProxySQL.log   ➜ 日志文件，排查问题好地方。
			• /var/lib/proxysql/ProxySQL.pid   ➜ pid文件不多说了。











🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  proxysql 准备   🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸


🔸 ProxySQL 安装 & 配置
		这是一个中转软件.  你可以安装在单独的服务器.
		也可以安装在主从的某个节点中.
		我们安装在. vps1 中 因为这个服务器的延迟最低..速度最快..

		⦿ ProxySQL 安装: 1.3.7 版本
				yum install https://www.percona.com/redir/downloads/percona-release/redhat/latest/percona-release-0.1-4.noarch.rpm -y && yum install proxysql -y
		⦿ ProxySQL 版本: 				     proxysql -V
		⦿ 新建 ProxySQL 配置文件		 vi /etc/proxysql.cnf




🔸 ProxySQL 监控用户和业务用户
		需要在所有的mysql 数据库上创建帐号: gce vps1 vps2 

		ProxySQL 需要登录到所有的数据库.进行数据库的监控和操作. 需要要用到两个帐号.
		• 监控是一个帐号: ProxySQL ProxySQLPass
		• 业务是一个帐号: sbuser   sbpass

		CREATE USER 'ProxySQL'@'%' IDENTIFIED BY 'ProxySQLPass';
		GRANT USAGE ON  *.* TO 'ProxySQL'@'%';

		CREATE USER 'sbuser'@'%' IDENTIFIED BY 'sbpass';
		GRANT ALL ON * . * TO 'sbuser'@'%';

		FLUSH PRIVILEGES;





🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  proxysql.cnf 内容   🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 vi /etc/proxysql.cnf


datadir="/var/lib/proxysql"
#❗️❗️ ProxySQL 管理接口 ❗️❗️
admin_variables=
{
    admin_credentials="admin:admin"
				# ❗️ ProxySQL 数据库登录用户和密码 ❗️
    mysql_ifaces="127.0.0.1:6032;/tmp/proxysql_admin.sock"
				# ❗️ ProxySQL 数据库IP和端口.      ❗️️
				#  SQLite 不是C/S 架构. 不支持远程登录!!!!!
}
mysql_variables=
{
    threads=4
    max_connections=2048
    default_query_delay=0
    default_query_timeout=36000000
    have_compress=true
    poll_timeout=2000
    interfaces="0.0.0.0:6033;/tmp/mysql.sock"
		#❗️ 这个必须是 6033 而不是 3306 
		# 如果你设置错了.最好是卸载 proxysql 重新安装.
		# 要修改的话. 不仅要改这个配置文件. 好像还要改 proxysql 的数据库.
    default_schema="information_schema"
    stacksize=1048576
    server_version="5.5.30"
    connect_timeout_server=3000
    monitor_history=600000
    monitor_connect_interval=60000
    monitor_ping_interval=10000
    monitor_read_only_interval=1500
    monitor_read_only_timeout=500
    ping_interval_server=120000
    ping_timeout_server=500
    commands_stats=true
    sessions_sort=true
    connect_retries_on_failure=10
}
mysql_servers =
(
		# ❗️ 主服务器 gce 设置; 
    {
        address = "35.194.128.92"
        port = 3306
        hostgroup = 0
        status = "ONLINE"  
        weight = 1   
        compression = 0 
    },

		# ❗️ 从服务器2 vps2 设置 
    {
        address = "104.224.139.45" 
        port = 3306           
        hostgroup = 1            
        status = "ONLINE"    
        weight = 1           
        compression = 0      
    }
)

# ❗️ 设置所有服务器的通用登录帐号密码 
mysql_users:
(
        {
                username = "root"
                password = "xujian0219"
                default_hostgroup = 0
                max_connections=1000
                default_schema="ss-vps1"
                active = 1
        }

)
    mysql_query_rules:
(
)
    scheduler=
(
)
mysql_replication_hostgroups=
(
    {
        writer_hostgroup=0
        reader_hostgroup=1
    }
)











🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸    ProxySQL 登录&简介  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 登录 

				ProxySQL 是一个代理(类似快递中转站). 也需要登录的.需要管理的.
					• 无中间件: 客户端 ➜ mysql
					• 有中间件: 客户端 ➜ proxySQL ➜ mysql.
						有中间件后我们的客户端不再是直接连 mysql 来读写数据. 
						而是连接到 proxysql .proxysql 会根据客户端的请求是读还是写.
						是写就把客户端的写入请求转发给 主服务器.
						是读就把客户端的读取轻松转发给 某个从服务器
						
				为了实现通过 proxysql 来操作 ss-vps1 数据库 我们需要进行额外设置.		

				ProxySQL 需要知道 
					• 有哪些主从服务器. 
					• 哪个是主数据库服务器, 哪个是从数据库服务器.
					• 主从服务器的IP地址. 端口. 
					• 还要一个能登录所有主从数据库的用户和密码


				ProxySQL也是有管理接口和客户端接口，
				作者为了方便我们使用,用Mysql 命令行来操作 管理接口和客户端接口.
					•	管理接口:   登录后可以控制 ProxySQL 各种设置的.
					• 客户端接口: 登录后可以操作 所有的主从数据库.


				⦿ ProxySQL 管理接口
					管理接口需要登录的. 
					和 mysql 一样需要帐号密码.端口.
					只是这里的帐号密码不是mysql 的账户密码.
					而是 ProxySQL配置文件/etc/proxysql.cnf	
					admin_variables= {.....} 模块里设置的帐号密码
					废话说了这么多 让我们登录 proxysql 看看吧.


	🔅 mysql -u admin -padmin -h 127.0.0.1 -P6032  ➜ 登录 proxy 的数据库.

				很多同学可能已经蒙逼，这不是进入了MySQL数据库了吗？哈哈。
				笔者第一次玩这个的时候也陷入了这种尴尬的局面，以为特么的搞错了. 其实这事ProxySQL作者的一番用意，
				他使用MySQL客户端作为用户和ProxySQL之间的交换工具，
				让MySQL的DBA有一种亲切感和熟悉感，我们做DBA的用起来上手更快，不用花时间学习，熟悉，适应，直接上手用。

				既然使用MySQL客户端登录，
				❗️❗️ mysql 的命令在这里也能用,只是显示的东西有些不一样!❗️❗️
				❗️❗️ mysql 的命令在这里也能用,只是显示的东西有些不一样!❗️❗️
				因为现在连的是 /var/lib/proxysql/ProxySQL.db  这个SQLite 数据库
				而且端口是proxysql的 6032 端口. 不是mysql的3306端口


	🔅	mysql> show databases;
				+-----+---------+-------------------------------+
				| seq | name    | file                          |
				+-----+---------+-------------------------------+
				| 0   | main    |                               |
				| 2   | disk    | /var/lib/proxysql/proxysql.db |
				| 3   | stats   |                               |
				| 4   | monitor |                               |
				+-----+---------+-------------------------------+
				• main 		内存配置数据库，表里存放后端db实例、用户验证、路由规则等信息。
				• disk 		持久化到硬盘的配置，sqlite数据文件。
				• stats 	proxysql运行抓取的统计信息，包括到后端各命令的执行次数、流量
				• monitor 库存储 monitor 模块收集的信息，主要是对后端db的健康/延迟检查。



		mysql> show tables;   ➜ main库内容
				+--------------------------------------+
				| tables                               |
				+--------------------------------------+
				| global_variables                ❗️设置变量，包括监听的端口、管理账号等 
				| mysql_collations               
				| mysql_query_rules              
				| mysql_replication_hostgroups   
				 监视指定主机组中所有服务器的read_only值，并且根据read_only的值将服务器分配给写入器或读取器主机组
				| mysql_servers                    ❗️ 设置后端MySQL的表
				| mysql_users                      ❗️ 配置后端数据库的账号和监控的账号
				| runtime_global_variables             |
				| runtime_mysql_query_rules            |
				| runtime_mysql_replication_hostgroups |
				| runtime_mysql_servers                |
				| runtime_mysql_users                  |
				| runtime_scheduler                    |
				| scheduler                            |
				+--------------------------------------+

					表名以 runtime_开头的表示proxysql当前运行的配置内容，
					不能通过dml语句修改，只能修改对应的不以 runtime_ 开头的（在内存）里的表，
					然后 LOAD 使其生效， SAVE 使其存到硬盘以供下次重启加载。





🔸 ProxySQL 数据库简介
		将数据库和数据库相关用户配置进ProxySQL,这些命令是在ProxySQL中执行的。

		⦿ 有无 runtime 前缀的区别.
				命令行只能修改 没有runtime 前缀的数据表!!!!
				要修改 有runtime 的数据包.只需要先修改没runtime 的对应的表.
				然后保存就可以了.

		⦿ 查看数据
			如果这两个数据库中 runtime_mysql_servers;  runtime_mysql_users;  已有数据.
			就不需要添加下面的节点.和帐号信息.
			因为上面的配置文件中你已经写进去恶劣.

				proxysql 设置里无法两个表
				• mysql_servers  设置数据库主从节点信息 
				• mysql_users    设置登录主从数据库的账户信息

				select * from mysql_servers\G;  
				select * from runtime_mysql_servers\G;  
				这里有2行数据就可以了. 每行一个数据库.
						👹 这里的状态..status 不应该是 online么 SHUNNED 是回避的意思啊...
						回避 就是服务器有问题的意思啊.... 怎么会...
						问题估计出在 mysql 的 my.cnf 配置文件中.



				select * from mysql_users;  
				select * from runtime_mysql_users;  
				这里必须要有我们之前创建的 监控账户 和 业务账户.



🔸 添加数据库节点

		这个添加可以在配置文件中设置. 也可以在proxysql 管理接口中这种.
		最终要去 proxysql 管理接口.看看设置有没有生效

		⦿ 添加主数据库节点: 
				>insert into mysql_servers(hostgroup_id,hostname,port) values(0,'35.194.128.92',3306);

		⦿ 添加从数据库节点:
				>insert into mysql_servers(hostgroup_id,hostname,port) values(1,'104.224.139.45',3306);
				>insert into mysql_servers(hostgroup_id,hostname,port) values(1,'23.105.192.96',3306);

				❗️❗️主节点的 hostgroup_id 的值是0  从节点的hostgroup_id值是1 ❗️❗️
						这个是在/etc/proxysql.cnf 下面模块里设置的.
						mysql_replication_hostgroups=
						(
								{
										writer_hostgroup=0
										reader_hostgroup=1
								}
						)



🔸 修改表数据(删除某行)	
		如果添加错了.就需要修改	
		mysql>delete from  mysql_servers where hostname='35.194.128.92';
		mysql>delete from  mysql_servers where hostname='23.105.192.96';
																									ip 必须用单引号!!!

🔸 配置业务帐号
		INSERT INTO MySQL_users(username,password,default_hostgroup) VALUES ('sbuser','sbpass',1);


🔸 配置监控帐号信息
		UPDATE global_variables SET variable_value='ProxySQL' WHERE variable_name='MySQL-monitor_username';
		UPDATE global_variables SET variable_value='ProxySQLPass' WHERE variable_name='MySQL-monitor_password';

🔸 配置生效
		上面的数据操作都是内存中操作的. 没有保存到硬盘.
		要保存到硬盘 还需要下面的操作

		LOAD MYSQL SERVERS TO RUNTIME;
		LOAD MYSQL USERS TO RUNTIME;
		SAVE MYSQL SERVERS TO DISK;
		SAVE MYSQL USERS TO DISK;



🔸 (VPS1 )通过ProxySQL登录 vps2/gce的 MySQL

		mysql -u sbuser -psbpass -h 127.0.0.1 -P 6033
		这个就登录 proxysql 了 .
		现在! proxysql 里显示的就是 mysql 数据库了!!!! 不再是proxysql 的数据库了.
		因为你登录的用户不一样啊. 之前是 admin .现在是 sbuser.
		那么现在连的到底是那个服务器上的数据库呢... 应该是gce .不太确定
		不管了 只要我们的 ss-vps1 数据库 正常就可以了.

		mysql> show databases;
		......
		mysql> use ss-vps1;
		Database changed
		mysql> show tables;
		......
		mysql> select * from user_token;
		....


		现在就成功的在 vps1 上登录了 vps2/gce 的数据库了.
		现在 vps1 上有两个数据库了.
		一个是 mysql 的3306端口.
		一个是 proxysql 的6033 端口.
		客户端 连接到 6033端口就可以读写 vps2/gce 上的数据库了.
		达到了我们的 读写分离目的.

















🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  mysql_servers 表  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 mysql_servers：设置后端Mysql的表
				mysql> select * from mysql_servers\G;
				*************************** 1. row ***************************
							hostgroup_id: 0
									hostname: 35.194.128.92
											port: 3306
										status: ONLINE
										weight: 1
								compression: 0
						max_connections: 1000
				max_replication_lag: 0
										use_ssl: 0
						max_latency_ms: 0
										comment:
				*************************** 2. row ***************************
							hostgroup_id: 1
									hostname: 104.224.139.45
											port: 3306
										status: ONLINE
										weight: 1
								compression: 0
						max_connections: 1000
				max_replication_lag: 0
										use_ssl: 0
						max_latency_ms: 0
										comment:
				2 rows in set (0.00 sec)



		⦿ hostgroup_id： 
				Proxysql通过 hostgroup (下称HG) 的形式组织后端db实例。
				一个 HG 代表同属于一个角色: 要么是读. 要么就是写.
				一个 HG 可以有多个实例，即多个从库，可以通过 weight 分配权重。
				可以看到一个主机 可以在多个hostgroup里面，也就是说一台主机 既可以读也可以写.

				❗️❗️ hostgroup_id 0 是一个特殊的HG，路由查询的时候，没有匹配到规则则默认选择 HG 0 ❗️❗️
				❗️❗️ hostgroup_id 0 是一个特殊的HG，路由查询的时候，没有匹配到规则则默认选择 HG 0 ❗️❗️
				❗️❗️ hostgroup_id 0 是一个特殊的HG，路由查询的时候，没有匹配到规则则默认选择 HG 0 ❗️❗️

		⦿ status：
				ONLINE：             当前后端实例状态正常
				SHUNNED：             临时被剔除，可能因为后端 too many connections error，或者超过了可容忍延迟阀值 max_replication_lag
				OFFLINE_SOFT：        “软离线”状态，不再接受新的连接，但已建立的连接会等待活跃事务完成。
				OFFLINE_HARD：        “硬离线”状态，不再接受新的连接，已建立的连接或被强制中断。当后端实例宕机或网络不可达，会出现。

				max_connections：     允许连接到该后端实例的最大连接数。不要大于Mysql设置的 max_connections，如果后端实例 hostname：port 在多个 hostgroup 里，以较大者为准，而不是各自独立允许的最大连接数。
				max_replication_lag： 允许的最大延迟，主库不受这个影响，默认0。如果 > 0， monitor 模块监控主从延迟大于阀值时，会临时把它变为 SHUNNED
				max_latency_ms：      mysql_ping 响应时间，大于这个阀值会把它从连接池剔除（即使是ONLINE），默认0。
				comment： 备注。

				❗️ mysql 正常.但是 proxysql 管理接口
				select * from runtime_mysql_servers\G;
				显示 shunned . 
				你需要在 proxysql 的配置文件中.
				在相应的服务器模块中添加  max_replication_lag = 500
				延迟时间自己设置. 然后重启 proxysql .
				再进去看  select * from runtime_mysql_servers\G;
				应该就正常了.









🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  mysql_users 表  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 mysql_users：配置后端数据库的账号和监控的账号
		mysql> select * from mysql_users\G;
		*************************** 1. row ***************************
									username: root
									password: xujian0219
										active: 1
									use_ssl: 0
				default_hostgroup: 0
						default_schema: ss-vps1
						schema_locked: 0
		transaction_persistent: 0
							fast_forward: 0
									backend: 1
									frontend: 1
					max_connections: 1000
		*************************** 2. row ***************************
									username: sbuser
									password: sbpass
										active: 1
									use_ssl: 0
				default_hostgroup: 1
						default_schema: NULL
						schema_locked: 0
		transaction_persistent: 0
							fast_forward: 0
									backend: 1
									frontend: 1
					max_connections: 10000


				• username, password： 连接后端db的用户密码。
				• active：             是否生效该用户。
				• transaction_persistent： 
						如果设置为1，连接上Proxysql的会话后，
						如果在一个hostgroup上开启了事务，那么后续的sql都继续维持在这个hostgroup上，不伦是否会匹配上其它路由规则，直到事务结束。虽然默认是0。

				• frontend, backend： 
						目前版本这两个都需要使用默认的1，
						将来有可能会把 Client -> Proxysql (frontend) 与 Proxysql -> BackendDB (backend)的认证分开。
						从 runtime_mysql_users 表内容看到，记录数比 mysql_users 多了一倍，就是把前端认证与后端认证独立出来的结果。
				
				• fast_forward： 
						忽略查询重写/缓存层，直接把这个用户的请求透传到后端DB。相当于只用它的连接池功能，一般不用，路由规则 .* 就行了。

				• default_hostgroup：  这个用户的请求没有匹配到规则时，默认发到这个 hostgroup，默认0
						❗️❗️❗️ 我们的问题肯定出在这里!  路由规则设置错误. 
						❗️❗️❗️ 我们的问题肯定出在这里!  路由规则设置错误. 
						❗️❗️❗️ 我们的问题肯定出在这里!  路由规则设置错误. 

						而且我们用的是 sbuser 连接的proxysql 数据库接口
						所以匹配不到规则. 默认就所有操作都在这个HG中执行了. 难怪.
						我们用 root 账户登录 proxysql 的数据库接口. 进行读写.
						然后看看管理接口中的读写状态是不是变成0那个HG了.

								数据库接口
								✘✘∙𝒗1 ~ ➜ mysql -uroot -pxujian0219 -h 127.0.0.1 -P 6033
								use ss-vps1;
								select * from user_token;
								insert into user_token (id,token,user_id,create_time,expire_time) VALUES (77,1,2,3,4);
								select * from user_token;

								管理接口 
								✘✘∙𝒗1 ~ ➜ mysql -u admin -padmin -h 127.0.0.1 -P6032
								select * from stats_mysql_query_digest; 
								现在就能看到 0 这个HG了.  
								好高兴啊.... 终于找到问题了. 路由规则不对. 坑爹的教程..






🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 ProxySQL 后端节点详情  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 作用 
		每当你在 proxysql 的数据库接口执行一个命令.
		你都可以去 proxysql 的管理接口.查看到底是那个节点执行了该命令.
		执行命令成功一次 connok 值就会加1 . 非常方便判断到底是主还是从执行了这个命令.


🔸 stats_mysql_connection_pool
		mysql -u admin -padmin -h 127.0.0.1 -P6032
				mysql> select * from stats_mysql_connection_pool\G;
				*************************** 1. row ***************************
							hostgroup: 0
							srv_host: 35.194.128.92
							srv_port: 3306
								status: SHUNNED
							ConnUsed: 0    Proxysql当前使用多少个连接来向后端服务器发送查询。
							ConnFree: 0    目前有多少个连接是空闲。
								ConnOK: 1    成功建立了多少个连接。
								ConnERR: 0    没有成功建立多少个连接。
								Queries: 1    路由到此特定后端服务器的查询数。
				Bytes_data_sent: 85   发送到后端的数据量。
				Bytes_data_recv: 0    从后端接收的数据量。
						Latency_us: 0    从Monitor报告的当前ping以毫秒为单位的延迟时间。
				*************************** 2. row ***************************
							hostgroup: 1
							srv_host: 104.224.139.45
							srv_port: 3306
								status: SHUNNED
							ConnUsed: 0
							ConnFree: 0
								ConnOK: 53
								ConnERR: 0
								Queries: 204
				Bytes_data_sent: 20140
				Bytes_data_recv: 1110411
						Latency_us: 0
				2 rows in set (0.00 sec)













🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  Proxysql 路由规则 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 添加路由规则

		⦿ 目的: 除select * from tb for update的select全部发送到slave，其他的的语句发送到master。

		⦿ 设置Slave 的规则.  把select语句都发送到slave 执行
				INSERT INTO mysql_query_rules(active,match_pattern,destination_hostgroup,apply) VALUES(1,'^SELECT',1,1);

		⦿ 设置Master 的规则.  把所有select之外的语句都给master执行.
				INSERT INTO mysql_query_rules(active,match_pattern,destination_hostgroup,apply) VALUES(1,'^SELECT.*FOR UPDATE$',0,1);

						• active                   表示是否启用这个sql路由项
						• match_pattern            就是我们正则匹配项，
						• destination_hostgroup    表示我们要将该类sql转发到哪些mysql上面去. 值参考你的 proxy配置里面的值
						• apply                    为1表示该正则匹配后，将不再接受其他匹配，直接转发。


		⦿ 保存配置到硬盘
				load mysql query rules to runtime;
				save mysql query rules to disk;


		⦿ ^SELECT
				SELECT 开头的. 给read 群组. 也就是1.

		⦿ ^SELECT.*FOR UPDATE$
				SELECT开头 而且. FOR UPDATE 结尾的 给写群组 也就是0 


				. ➜ 匹配换行符以外的单个字符
				* ➜ 匹配 0或多个前导字符
				我们来看mysql 的 select 用法
				一般来说 我们用 select * from table 就可以获取数据了.
				但是在大量数据存储的情况下 　select 命令不安全.
				比如淘宝某个商品要下订单 必须先看有没有库存.
				如果有库存那么 下订单的同时需要减去库存.
				而不能 先下订单 然后减库存. 因为这两步之间有时间差.
				比如某个商品只有最后一件库存.
				必须下订单和减库存同时进行.要实现这个功能 就需要.
				SELECT FOR UPDATE;  语法.
				SELECT quantity FROM products WHERE id=3 FOR UPDATE; 
				这个就是更新数据. 也就得写数据库了.
				只有这个语法的 才会到 主服务器进行.
				不然就到从服务器进行...
				


🔸 查看路由规则.

		⦿ proxysql 管理接口查看规则

				mysql -u admin -padmin -h 127.0.0.1 -P6032
				mysql> select * from mysql_query_rules\G;
				*************************** 1. row ***************************
											rule_id: 1       ➜ 几条规则就几个序号.
											active: 1        ➜ 只有值为1 的 才参与规则匹配
										username: NULL     ➜ 如果非 NULL，只有连接用户是 username 的值才会匹配
									schemaname: NULL
											flagIN: 0
									client_addr: NULL    ➜ 匹配客户端来源IP
									proxy_addr: NULL
									proxy_port: NULL
											digest: NULL     ➜ 精确匹配一类查询。
								match_digest: NULL     ➜ 正则匹配一类查询。
								match_pattern: ^SELECT ➜ 正则匹配查询
				negate_match_pattern: 0       
											flagOUT: NULL    ➜ ➜ ➜ ➜ ➜  ➜ 上面都是匹配规则，下面是匹配后的行为：
							replace_pattern: NULL    查询重写，默认为空，不rewrite。
				destination_hostgroup: 1       路由查询到这个 hostgroup。
										cache_ttl: NULL    查询结果缓存的毫秒数。
										reconnect: NULL
											timeout: NULL   这一类查询执行的最大时间（毫秒），超时则自动kill。这是对后端DB的保护机制，默认给的是10h。
											retries: NULL   语句在执行时失败时，重试次数
												delay: NULL   查询延迟执行，这是Proxysql提供的限流机制，会让其它的查询优先执行。
							mirror_flagOUT: NULL
						mirror_hostgroup: NULL
										error_msg: NULL
													log: NULL    是否记录查询日志。
												apply: 1
											comment: NULL
				*************************** 2. row ***************************
											rule_id: 2
											active: 1
										username: NULL
									schemaname: NULL
											flagIN: 0
									client_addr: NULL
									proxy_addr: NULL
									proxy_port: NULL
											digest: NULL
								match_digest: NULL
								match_pattern: ^SELECT.*FOR UPDATE$
				negate_match_pattern: 0
											flagOUT: NULL
							replace_pattern: NULL
				destination_hostgroup: 0
										cache_ttl: NULL
										reconnect: NULL
											timeout: NULL
											retries: NULL
												delay: NULL
							mirror_flagOUT: NULL
						mirror_hostgroup: NULL
										error_msg: NULL
													log: NULL
												apply: 1
											comment: NULL
				2 rows in set (0.00 sec)




🔸 实际路由规则
			proxysql 实际的路由规则由好几部分组成的.
			1. 路由规则.
			2. 用户默认操作数据库.

			首先任何一个sql 语句.都会先匹配路由规则.
			如果匹配到 那么就按照匹配设置分配主从节点

			❗️❗️如果没匹配到! 那么 不是默认写就是给主的.读就是给从的.❗️❗️
			而是按照你的 proxysql 数据库接口的 登录用户的 默认数据库来的.
			所以用户的默认数据库设置很重要.一般都是设置主数据库!!!而不是从数据库.






🔸 定义路由规则


⦿ 先清空stats_mysql_query_digest统计表：
		select * from stats_mysql_query_digest_reset;
		注意这个命令不是查看.比查看多个 _reset!!		select * from stats_mysql_query_digest;


⦿ 写入测试数据： 
		现在是在 proxysql 的数据库接口执行的. 不是在管理接口执行的
		insert into user_token (id,token,user_id,create_time,expire_time) VALUES (9998,1,2,3,4);

⦿ 读取数据: 
		看看有没有 9998 多出来. 有就代表写入成功的.
		select * from user_token;


⦿ 查看统计信息：
	select * from stats_mysql_query_digest;
	发现 hostgroup 还是1. 啊....fuck....






🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 👹 MySQL server has gone away  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

⦿ 管理接口 问题排除
		select * from mysql_servers\G;  
		select * from runtime_mysql_servers\G;  
				👹 这里gce的状态..status 不应该是 online么 SHUNNED 是回避的意思啊...
				回避 就是服务器有问题的意思啊.... 怎么会.....
				又要问题排除了... 问题在哪 我也不知道啊.
				我Mac连 gce的数据库正常的啊. 没有防火墙.....
				到底哪里错了...  要看 proxysql的日志了么.


⦿ 查看当前 后端的mysql 运行状态
		mysql> select hostgroup_id,hostname,port,status from runtime_mysql_servers;
		+--------------+----------------+------+---------+
		| hostgroup_id | hostname       | port | status  |
		+--------------+----------------+------+---------+
		| 0            | 35.194.128.92  | 3306 | SHUNNED |
		| 1            | 104.224.139.45 | 3306 | SHUNNED |
		+--------------+----------------+------+---------+
		shunned 都是挂了的意思啊...


⦿ 我们去 proxysql 的数据库接口. 查询下数据是能查询到的啊!!! 说明没挂啊!!
		你看 是重新连接了... 说明之前是被防火墙啥啥的断开了?
		还是mysql 服务端自动断开的客户端...

		mysql> show databases;
		ERROR 2006 (HY000): MySQL server has gone away
		No connection. Trying to reconnect...
		Connection id:    46
		Current database: ss-vps1
		....


⦿ 原因
		增加gce/vps2服务器 my.cnf 配置文件的 max_allowed_packet and wait_timeout  值就可以了.
		如果服务器正常的. 那么问题一般出在下面两个原因中.
		一个是 单个包太大. 被丢弃.
		一个是 服务端的 my.cnf 配置文件中的 wait_timeout 值太小


🔸 单条记录最大值
		mysql> show global variables like 'max_allowed_packet';
		一般是1M. 
		编辑/etc/my.cnf，将 max_allowed_packet = 1M ➜ 1G
		然后重启数据库

		然后再用 		mysql> show global variables like 'max_allowed_packet';
		确保设置生效.


🔸 客户端 TCP/IP 超时断开.

		如果程序使用的是长连接，则这种情况的可能性会比较大。
		即，某个长连接很久没有新的请求发起，达到了server端的timeout，被server强行关闭。
		此后再通过这个connection发起查询的时候，就会报错server has gone away

		去GCE 看超时设置.
		mysql> show global variables like '%timeout';
		+----------------------------+----------+
		| Variable_name              | Value    |
		+----------------------------+----------+
		| connect_timeout            | 10       |
		| delayed_insert_timeout     | 300      |
		| innodb_lock_wait_timeout   | 50       |
		| innodb_rollback_on_timeout | OFF      |
		| interactive_timeout        | 28800    |
		| lock_wait_timeout          | 31536000 |
		| net_read_timeout           | 30       |
		| net_write_timeout          | 60       |
		| slave_net_timeout          | 3600     |
		| wait_timeout               | 28800    |
		+----------------------------+----------+


		mysql> SET SESSION wait_timeout=5;
		Query OK, 0 rows affected (0.00 sec)

		mysql> SELECT NOW();
		ERROR 2006 (HY000): MySQL server has gone away
		No connection. Trying to reconnect...
		Connection id:    20324
		Current database: ss-vps1

		+---------------------+
		| NOW()               |
		+---------------------+
		| 2017-07-24 19:08:59 |
		+---------------------+
		1 row in set (0.00 sec)


		把超时设为5秒.
		然后你等10秒钟. 在进行任何操作.就会报错!
		现在这个报错 和你在proxysql 里面遇到的报错一模一样了.
		一般来说都是  wait_timeout 太短了. 设置长一点!!

		mysql> SET SESSION wait_timeout=28800; 是8小时啊 不短了...!
		估计是 slave_net_timeout 这个参数 3600秒 一小时..也还好啊..
		那么是 net_read_timeout  和 net_write_timeout 应该就是了.
		这两个参数只有 60秒 也就是一分钟...



		#skip-networking
		max_connections = 500
		max_connect_errors = 100
		open_files_limit = 65535
		下添加 
		wait_timeout = 6000000
		然后重启数据库


		再去 proxysql 的管理接口查看 服务器信息.
				select * from mysql_servers\G;
				确保这里两个都是 online.
				然后保存设置
				LOAD MYSQL SERVERS TO RUNTIME;
				SAVE MYSQL SERVERS TO DISK;

		重启 proxysql service proxysql restart
		再进入:       mysql -u admin -padmin -h 127.0.0.1 -P6032
		再查看...








🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  其他  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 mysql_replication_hostgroups：
		监视指定主机组中所有服务器的read_only值，并且根据read_only的值将服务器分配给写入器或读取器主机组

		定义 hostgroup 的主从关系。
		Proxysql monitor 模块会监控 HG 后端所有servers 的 read_only 变量，
		如果发现从库的 read_only 变为0、主库变为1，则认为角色互换了，
		自动改写 mysql_servers 表里面 hostgroup 关系，达到自动 Failover 效果。


🔸 stats_mysql_commands_counters
		统计各种sql类型的执行次数和时间，

		mysql> select * from stats_mysql_commands_counters;



🔸 查看 proxysql 的统计信息.

		⦿ 登录 proxysql 管理接口
				通过proxysql接口(6033端口)正常操作数据，从管理接口(6032端口)看看ProxySQL的统计信息：
				mysql -u admin -padmin -h 127.0.0.1 -P6032

		⦿ 查看各类命令的执行情况, 看不懂啊 没关系我也不懂....
				mysql> select Command,Total_Time_us,Total_cnt from stats_mysql_commands_counters where Total_cnt >0;
						+---------+---------------+-----------+
						| Command | Total_Time_us | Total_cnt |
						+---------+---------------+-----------+
						| COMMIT  | 0             | 227       |
						| INSERT  | 5494768       | 230       |
						| SELECT  | 3473350       | 237       |
						| SET     | 0             | 567       |
						| SHOW    | 163228        | 4         |
						+---------+---------------+-----------+
						5 rows in set (0.01 sec)


		⦿ 查看各类SQL的执行情况
				select * from stats_mysql_query_digest;
						会发现 所有在 proxysql 上读写的操作 都是到 hostgroup=1 这个服务器进行的.
						我们在 /etc/proxysql.cnf  的配置是下面这样的.
						writer_hostgroup=0 、 reader_hostgroup=1

						我们现在根本没有用到主库. 读写都到从库去了.!
						主要原因就是ProxySQL的核心mysql_query_rules路由表没有配置。
						proxysql是通过自定义sql路由规则就可以实现读写分离。











































🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  常用命令  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 MySQL 
		• 配置:    vi /etc/my.cnf
		• 重启:    service mysql restart


🔸 Proxysql 
		• 配置:  vi /etc/proxysql.cnf
		• 启动   service proxysql start
		• 停止   service proxysql stop
		• 重启   service proxysql restart
		• 状态   service proxysql status
		• 验证   nmap -p 6032,6033 127.0.0.1   ➜ 6032,6033 端口必须同时开启.	


🔸 proxysql 管理接口
			mysql -u admin -padmin -h 127.0.0.1 -P6032
					show tables;
					
					select * from mysql_servers\G;                    
					select * from runtime_mysql_servers\G; ➜ 查看集群状态
					select * from mysql_users\G;  
					select * from runtime_mysql_users\G;   ➜ 查看用户状态

					LOAD MYSQL SERVERS TO RUNTIME;
					LOAD MYSQL USERS TO RUNTIME;
					SAVE MYSQL SERVERS TO DISK;
					SAVE MYSQL USERS TO DISK;

					select * from mysql_query_rules;

					INSERT INTO mysql_query_rules(active,match_pattern,destination_hostgroup,apply) VALUES(1,'^SELECT',1,1);
					INSERT INTO mysql_query_rules(active,match_pattern,destination_hostgroup,apply) VALUES(1,'^SELECT.*FOR UPDATE$',0,1);
					load mysql query rules to runtime;
					save mysql query rules to disk;

					select * from stats_mysql_query_digest\G;           ➜ 查看读写统计数据.



🔸 proxysql 数据库接口
		不同用户有不同的默认操作数据库. 没匹配到正则式.就会然默认数据库执行命令.

		mysql -u sbuser -psbpass -h 127.0.0.1 -P 6033
		mysql -u root -pxujian0219 -h 127.0.0.1 -P 6033


				use ss-vps1;
				show tables;

				select * from user_token;

				insert into user_token (id,token,user_id,create_time,expire_time) VALUES (18,1,2,3,4);
				insert into user_token (id,token,user_id,create_time,expire_time) VALUES (43,1,2,3,4);
				insert into user_token (id,token,user_id,create_time,expire_time) VALUES (07,1,2,3,4);
				insert into user_token (id,token,user_id,create_time,expire_time) VALUES (97,1,2,3,4);

				show global variables like '%timeout';    ➜ 查看超时设置.












		













🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  读写分离测试  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 proxysql 读写功能测试

	上面设置了两台的 proxysql_1、proxysql_2
	下面我们来测试 vps2 上的 proxysql_2 读写分离是否成功设置.
	👁‍🗨 proxysql 也是一个mysql数据库.  你可以远程登录的.

	⦿ GCE ssr 前端配置
			你用了 proxysql .
			那么程序 就不能直接连mysql. 而是要连 proxysql 里面的数据库.
			帐号 密码 端口都用 proxysql 的设置.
			前端配置: vi /home/wwwroot/ss.0214.help/.env

			db_host = 'localhost'      ➜ 104.224.139.45
			db_port = '3306'           ➜ 6033
			db_database = 'ss-vps1'  
			db_username = 'root'       ➜ sbuser
			db_password = 'xujian0219' ➜ sbpass

			然后 ss.0214.help 能正常登录登出. 说明 proxysql 的客户端接口正常.
			至于到底是不是读写分离的,应该是不会错的.


	⦿ GCE ssr 后端配置
			vi /root/shadowsocksr/usermysql.json

					"host": "104.224.139.45",
					"port": 6033,
					"user": "sbuser",
					"password": "sbpass",
					"db": "ss-vps1",


			然后重启后端
			停止运行 sh /root/shadowsocksr/stop.sh
			开始运行 sh /root/shadowsocksr/run.sh

			现在gce 能正常翻墙.说明 proxysql_2 确实是正常了!!!
			我们可以用 ssr 来判断 proxysql 是否配置成功.
			也可以直接登录 proxysql 来看看能不能真正对数据库进行读写



🔸 proxysql 写入测试：
		首先在 vps1 上登录 6033 端口的 proxysql .

		✘✘∙𝒗1 ~ ➜ mysql -u sbuser -psbpass -h 127.0.0.1 -P 6033

		mysql> use ss-vps1;
		Database changed
		mysql> show tables;
		mysql> select * from user_token;
					+-------+-------+---------+-------------+-------------+
					| id    | token | user_id | create_time | expire_time |
					+-------+-------+---------+-------------+-------------+
					|    12 |       |       0 |           0 |           0 |

		mysql> insert into user_token (id,token,user_id,create_time,expire_time) VALUES (18,1,2,3,4);
		ERROR 2006 (HY000): MySQL server has gone away
		No connection. Trying to reconnect...
		Connection id:    10
		Current database: ss-vps1
		Query OK, 1 row affected (0.05 sec)


				我们往 user_token 表里插入写数据.		这个mysql命令肯定是对的. 
				这里出问题了!!!  MySQL server has gone away 虽然后来重新连接进去了.但是第一次是连接失败的!
				wait_timeout

				然后查看数据库.... 忽然发现.. 新加的数据写到了 vps2里. 而不是 gce里面啊.
				这就搞大了...   难道是因为 vps2 的my.cnf 没有加入只读么...
				这个读写分配应该是  proxysql 的事情啊.
				现在就要进行问题排除了. 这个需要登录 proxysql 的管理接口. 而不是现在的数据库接口.



🔸 超时设置.
在Mysql的默认设置中，如果一个数据库连接超过8小时没有使用（闲置8小时，即
28800s），mysql server将主动断开这条连接，后续在该连接上进行的查询操作都将失败，将
出现：error 2006 (MySQL server has gone away)!。

但是mysql 配置文件中 有两个超时参数.
wait_timeout 和 interactive_timeout

如果修改interactive_timeout的话wait_timeout也会跟着变，而只修改wait_timeout是不生效的。
所以保险起见 修改 interactive_timeout  而不是 wait_timeout



设置成一年 
mysqld 模块下添加下面两个. 

wait_timeout=31536000
interactive_timeout=31536000









⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵  sysbench 性能测试 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸   sysbench 性能测试  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸


🔸 sysbench 简介.
		sysbench是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。


🔸 sysbench yum安装(推荐):
		yum list installed | grep sysbench查看已安装的sysbench
		yum remove -y sysbench.x86_64 移除已安装的sysbench
		只能查看/移除 用 yum安装的 sysbench. 

		yum list | grep sysbench查看可安装的sysbench包
		yum -y install sysbench 安装sysbench


🔸 sysbench 编译安装(不推荐):

		1.下载软件
				cd && git clone https://github.com/akopytov/sysbench.git && cd sysbench

		2.安装依赖包
				yum -y install  make automake libtool pkgconfig libaio-devel vim-common

		3.安装sysbench
				./autogen.sh
				./configure --prefix=/usr/local/sysbench
				make && make install

		4. 设置环境路径
	 		  echo "export PATH=/usr/local/sysbench/bin:\$PATH" >> ~/.zshrc
				echo "export LD_LIBRARY_PATH=/usr/local/MySQL/lib:\$PATH" >> ~/.zshrc
				source ~/.zshrc

				不设置环境路径只能 /usr/local/sysbench/bin/sysbench -V 来使用

		5. 查看版本:  
				sysbench -V



🔸 测试流程
		数据库测试肯定会有 读取和写入.
		测试肯定是新件个测试数据库来测试的.
		为了安全不能用现有的数据库来测试.

		测试分三个阶段.  你只需要创建数据库和mysql 帐号密码
		其他的 sysbench 会搞定.

		prepare 准备工作，eg：创建文件、填充数据库等，
		run 运行内置选项或者lua脚本实际的test, 
		cleanup  在run之后， 清除临时的数据,



🔸 命令解析

	sysbench --test=/usr/share/sysbench/oltp_read_only.lua --db-driver=mysql --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root --mysql-password=xujian0219 --mysql-db=testdb --tables=10 --table-size=1000 --report-interval=10 --threads=12 --time=120 prepare

		--test=/usr/share/sysbench/oltp_read_only.lua 表示调用脚本进行模式测试

		--mysql-host=23.105.192.96 
		--mysql-port=3306
		--mysql-user=root 
		--mysql-password=xujian0219

		--tables	    10	  测试中会自动创建10个表来给你测试.
		--table_size	1000	每个表的数据量
		--threads	    12	  测试线程数
		--time	      120	  测试时间，单位为秒
				
		





🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  测试实例  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 准备工作
	⦿ 在vps1 上创建名为 testdb 的数据库
	⦿ 在vps1 上创建数据库用户: root 密码 xujian0219 
	⦿ 在vps1 上安装 sysbench
	⦿ cd 到/usr/share/sysbench目录下 需要用到里面的测试脚本.

🔸 测试: 
	测试分.3步骤.
		准备阶段: prepare 创建测试数据
			sysbench ./oltp_read_only.lua --db-driver=mysql --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root --mysql-password=xujian0219 --mysql-db=testdb --tables=10 --table-size=1000 --report-interval=10 --threads=12 --time=120 prepare

		测试阶段: run     真正测试.
			sysbench ./oltp_read_only.lua --db-driver=mysql --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root --mysql-password=xujian0219 --mysql-db=testdb --tables=10 --table-size=1000 --report-interval=10 --threads=12 --time=120 run

		收尾阶段: cleanup  清除第一步创建的测试数据
			sysbench ./oltp_read_only.lua --db-driver=mysql --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root --mysql-password=xujian0219 --mysql-db=testdb --tables=10 --table-size=1000 --report-interval=10 --threads=12 --time=120 cleanup



🔸 测试结果

		SQL statistics:
				queries performed:
						read:                            676872
						write:                           0
						other:                           96696
						total:                           773568
				transactions:                        48348  (402.80 per sec.)
				queries:                             773568 (6444.79 per sec.)
				ignored errors:                      0      (0.00 per sec.)
				reconnects:                          0      (0.00 per sec.)

		General statistics:
				total time:                          120.0300s
				total number of events:              48348

		Latency (ms):
						min:                                  1.34
						avg:                                 29.78
						max:                                209.68
						95th percentile:                     71.83
						sum:                            1440031.79

		Threads fairness:
				events (avg/stddev):           4029.0000/36.06
				execution time (avg/stddev):   120.0026/0.01









🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  测试实例 二:  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸
看到读写分离已经成功。为了更直观一点，使用sysbench测试读写分离是否正常：
我们直接在 ss-vsp1 这个数据库测试吧... 先备份数据库..


⦿ 先清空stats_mysql_query_digest统计表：
		mysql> select * from stats_mysql_query_digest_reset;

⦿ 查看 stats_mysql_query_digest统计表：
		mysql> select * from stats_mysql_query_digest;

⦿ sysbench 准备测试数据		
		sysbench --test=/usr/share/sysbench/oltp_read_write.lua --mysql-host=23.105.192.96 --mysql-port=6033 --mysql-user=root --mysql-password=xujian0219 --mysql-db=ss-vps1  --report-interval=10  --max-requests=0 --time=30 --threads=4 --tables=1  --table-size=500 --skip-trx=on --db-ps-mode=disable --mysql-ignore-errors=1062 prepare

⦿ sysbench 开始测试		
		sysbench --test=/usr/share/sysbench/oltp_read_write.lua --mysql-host=23.105.192.96 --mysql-port=6033 --mysql-user=root --mysql-password=xujian0219 --mysql-db=ss-vps1  --report-interval=10  --max-requests=0 --time=30 --threads=4 --tables=1  --table-size=500 --skip-trx=on --db-ps-mode=disable --mysql-ignore-errors=1062 run

⦿ 测试完成后看下统计表：
		✘✘∙𝒗1 ~ ➜ mysql -u admin -padmin -h 127.0.0.1 -P6032
		mysql> select * from stats_mysql_query_digest;
				这里你会看到 hostgroup 有0 和1 了. 
				说明有些操作是在 hostgroup 0: 主节点 gce上执行的.
				有些操作是在 hostgroup 1: 从节点 vps2上执行的
				到这里 我们的读写分离就完成了!!!

				从上面的结果可知，路由规则已经生效，select语句均到从库上执行了。

⦿ sysbench 测试收尾.
		测试过程会在 ss-vps1 数据库中创建 sbtest.... 等表.
		你可以手动删除. 最好是用下面命令自动删除..
		sysbench --test=/usr/share/sysbench/oltp_read_write.lua --mysql-host=23.105.192.96 --mysql-port=6033 --mysql-user=root --mysql-password=xujian0219 --mysql-db=ss-vps1  --report-interval=10  --max-requests=0 --time=30 --threads=4 --tables=1  --table-size=500 --skip-trx=on --db-ps-mode=disable --mysql-ignore-errors=1062 clean






















⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵  LVS  🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️


英文文档.
★★★★★★★ http://www.austintek.com/LVS/LVS-HOWTO/HOWTO/LVS-HOWTO.LVS-Tun.html






★★★★★ 三种模式图文讲解.   http://www.cswlyw.com/a/yunweijishu/2017/0723/261.html

TUN 模式的DS 需要双网卡 + 双外网IP??????
;　




IPVS比对数据包请求的服务是否为集群服务，



lvs/tun
优点：

		不需要调度应答报文，性能高
		服务器和调度器可以不再一个VLAN
		支持广域负载均衡
		缺点：

		所有服务器必须支持“IP Tunneling”协议，要安装内核模块（比如IPIP等），配置复杂
		有建立IP隧道的开销
		服务器上直接绑定虚拟IP，风险很大
		服务器需要联通外网
		不支持端口映射




keepalived + proxysql + mysql .
怎么没有lvs  keepalived 功能 比lvs强大. 但是 keepalived 要求服务器必须有两个IP. 
如果服务器只有一个IP. 那么就要用 lvs..







                        ________
                       |        |
                       | client |
                       |________|
                       CIP=192.168.1.254
                           |
             CIP->VIP |    |   ^  
                      v    |   | VIP->CIP
                           |
       VIP=192.168.1.110   |
       (eth0:1, arps)      |
         __________        |
        |          |       |
        | director |-------
        |__________|       |
       DIP=192.168.1.1     |
       (eth0)              |
                           |
   DIP->RIP(CIP->VIP) |    |
                      v    
          -------------------------------------
          |                |                  |
          |                |                  |
   RIP1=192.168.1.2  RIP2=192.168.1.3  RIP3=192.168.1.4 (eth0)
   VIP=192.168.1.110 VIP=192.168.1.110 VIP=192.168.1.110 (all tunl0,non-arping)
    _____________     _____________     _____________
   |             |   |             |   |             |
   | realserver  |   | realserver  |   | realserver  |
   |_____________|   |_____________|   |_____________|





🔸 LVS 相关术语

		• CIP：Client IP，客户端(用户)的外网IP地址。

		• DS ：Director Server。接收客户端的请求然后分配给集群中某个服务器执行.
				• VIP：向外部直接面向用户请求，作为用户请求的目标的IP地址。
						👁‍🗨 必选: VIP 必须是外网IP.
				• DIP：Director Server IP，主要用于和内部主机通讯的IP地址。
						👁‍🗨 可选: DIP 如果机器是多网卡. 可以设置DIP.毕竟内网速度块.

		• RS：Real Server。后端真实的工作服务器。
				• RIP：Real Server IP，后端服务器的IP地址。
						👁‍🗨 必选: 内外网IP都可以.



🔸 LVS 三层架构.

		⦿ 最前面: 负载调度器（load balancer） DS 负载均衡层面
				(相对于路由器,设定集群的路由表.) ➜ 肯定有一个公网IP. 客户端要连这个IP的.

		⦿ 中间的: 服务器池（server pool） RS 服务器集群组
			➜ 这里一般都是内网IP!!! 当然也可以是 公网IP.

		⦿ 底下的: 共享存储（shared storage）  数据层.
				集群需要访问数据.  这些数据需要一致.
				由于是集群. 必然会有数据的读取和写入.
				要保证所有集群读取到正确的内容.



🔸 LVS负载均衡技术简介

		指标	                VS/NAT	    VS/TUN	        VS/DR
		服务器操作系统	       任意	       支持隧道	       多数(支持Non-arp)
		服务器网络	           私有网络	   局域网/广域网	 局域网
		服务器数目(100M网络)	 10~20	     100	           大于100
		服务器网关	           负载均衡器	自己的路由	     自己的路由
		效率	                一般	      高	             最高


		⦿ NAT 模式: 		
				CIP ➜ DS: VIP(WAN) ➜ RS: RIP (LAN) ➜ DS: VIP (WAN) ➜ CIP
						• 1) DS 要把客户端发起的请求分配给某个 RS
						• 2) RS 处理客户端请求的结果返回给 DS
						• 3) DS 把就RS得到的结果回复给CIP

				网卡/IP 要求: 
						DS 必须有双网卡.  
						接收客户端请求肯定需要公网IP.				
						DS 和 RS 之间的交流是通过 LAN的. 所以DS 和RS 必须要有内网IP.

				优缺点			
						客户端的请求和客户端的响应都由 DS 处理. DS 容易变成瓶颈.

		⦿ TUN 模式:  
				CIP ➜ DS: VIP(WAN) ➜ RS: RIP(WAN) ➜ CIP
						• 1) DS 把客户端发起的请求分配给某个 RS
						• 2) RS 处理客户端请求的结果直接返回给 CIP

				网卡/IP 要求:
						DS 和 RS 有一个网卡就可以. 但是两个都必须是公网IP

				系统要求: 
						这里的 DS 要把 WAN 把客户端请求转发给 RS. 
						需要操作系统支持 IP隧道 技术. Linux 是支持的.Win 就不一定了.

				优缺点			
						相比 NAT模式. DS 要处理的内容少的多. 
						DS 可以处理更多的客户端请求.使得DS更加不容易成为系统瓶颈
						但是这里有个小问题. DS 和 RS 都是公网IP. 延迟大,速度慢.


		⦿ DR 模式:

				CIP ➜ DS: VIP(WAN) ➜ RS: RIP (LAN) ➜ RS: RIP (WAN) ➜ CIP

				网卡/IP 要求:
						DS: 必须双网卡. 一个外网(接收客户端的请求) + 一个内网(转发客户端请求给RS)
						RS: 必须双网卡. 一个外网(响应客户端的请求) + 一个内网(接收客户端请求从DS)

				优缺点			
						相比 TUN 模式. DS 和 RS 之间是通过内网交流的. 速度非常块. 
						在三种模式中性能最高.缺点就RS需要有外网IP.




🔸 IPVS 是什么？
IPVS 是 lvs的第一部分负载调度器（load balancer）的实现 ，也就是最核心的部分，用来进行负载均衡,
ipvs主要实现了  三种IP负载均衡技术  + 多种论调算法 （轮叫，加权、最小链接、加权最小链接等等）来实现lvs的负载均衡功能。。




🔸 Heartbeat 是什么？

		heartbeat提供了2个核心的功能正式  lvs所需要的，心跳监测部分和资源接管，
		心跳监测可以通过网络链路和串口进行，而且支持冗余链路，
		安装了 Heartbeat 的两台机器会通过心跳检测互相检测对方的状态，
		当检测到对方失效的时候会调用资源接管来做接管服务器，保证高可靠性。

		在一个高可靠的lvs集群中，负载调度 IPVS部分一般由2台服务器组成,一台负责调度，一台负责备用，
		当负责调度的服务器出现问题的时候迅速切换到备用机器上，
		而heartbeat 就是负责检测，负载调度 IPVS 的可用性，并在出现问题的时候切换到备用 IPVS 上面...



🔸 ldirectord是什么？ ldirectord的作用
             ldirectord是专门为LVS 监控而编写的。是用来监控 lvs架构中 服务器池（server pool） 的服务器状态的..
             ldirectord 运行在 IPVS 节点上， 
						 ldirectord作为一个守护进程启动后会对服务器池中的每个真是服务器发送请求进行监控,
						 如果 服务器没有响应 ldirectord 的请求，那么ldirectord 认为该服务器不可用 
						 ldirectord 会运行 ipvsadm  对 IPVS表中该服务器进行删除，如果等下次再次检测有相应则通过ipvsadm 进行添加


🔸 Keepalived 是什么？有什么作用？
			Keepalived在这里主要用作RealServer的健康状态检查以及LoadBalance主机和BackUP主机之间failover的实现 。
			IPVS通常与keepalived配合使用，后者也是LVS项目的子项目之一，用于检测服务器的状态。
			在lvs体系中，Keepalived主要有如下2个功能
			1、用来检测 2台 负载调度器 IPVS的状态加测，出现问题及时切换
			2、检测服务器池 各个节点的状态，动态的移除故障节点，动态添加故障解除节点
			也就是说 Keepalived 实现了  heartbeat + ldirectord 的功能...



🔸 LVS的两种实现方法（heartbeat与KeepAlived） 
lvs分化成了2种集群实现方式
       
			 第一种集群方式 ： LVS+heartbeat+ldirectord实现集群负载：
                    ldirectord+heartbeat的完美组合
                    ldirectord+heartbeat方案前端负载调度器采用双机热备份方式，双机均安装双网卡，一个网卡用于连接集群系统，另一个作为冗余心跳线路连接双机。采用串口线 ＋ 以太网口做为冗余心跳线路，以确保双机热备份的可靠性，消除由于主负载调度器或心跳线故障带来的集群单点故障。
                    在负载调度器的主机和备机上都安装上 ldirectord，heartbeat，并且全部启动，主机和备机就会相互监听状态，当备机检测到主机发生了故障，备机就会调用shell脚本来完整虚拟ip故障转移和调用 ipvsadm 启动 LVS及ldirectord，完成整个负载均衡从主机到备机的切换...

                   备机检测到主机又恢复正常之后，会再次进行虚拟ip故障切换到主机，停止自身的lvs 和 ldirectord...
                                       
            
        第二种集群方式：LVS+KeepAlived集群负载(这种目前是比较优秀的方案了)



🔸 IP Tunel 原理.

		我们的目的: 客户端访问 gce 然平均分配请求到 vps1 和 vps2 .
		要实现这个功能. 需要所有的客户端都连 gce.
		然后 gce 通过IP隧道技术把客户端的请求 平均转发给 vps1 或 vps2

		⦿ IP隧道 简介
				又叫IPIP，是一种将一个IP封装到另外一个IP中进行传输的技术。
				将两个无法直接通信的IP，封装在两个能够直接通信的IP，借助IP Tunnel 进行传输、通信。
				我们这里就是利用IP隧道技术.实现 CIP 和 某个RIP 之间进行通信.

		⦿ IP隧道 实例
				其实我们的VPN 就是一个例子.
				我们不用VPN 就上不了谷歌. 我们用了VPN就可以上谷歌网站.
				不能直接上谷歌是因为国内电脑是不能上谷歌的. 国外电脑是可以上谷歌的.
				国内电脑想上谷歌.只能通过国外电脑来实现! 国外电脑就像一个中转站.
				我们电脑连上VPN后, 其实是把网页请求先添加个封包转发给发给国外的电脑. 
				然后国外的电脑进行解包,再把这个网页请求发给谷歌.然后谷歌回复网页内容到国外电脑.
				最后国外电脑把回复内容转发到你电脑.

				VPN 和 IP隧道还是有点区别的.  
				VPN 是基于 PPTP/L2TP 等协议的. 
				IP 隧道是基于Linux的内核模块的.



		⦿ IP隧道 要求
				需要两个部件：封装部件和解包部件，两端各需要一个IP地址，且两个IP地址能够直接通信。
				也就是说. DS 和 RS 要么LAN 要么 都是WAN.
				如果DS、RS 都是WAN, 那么DS、RS 只要一个网卡就够了.
				如果DS、RS 都是LAN, 那么RS可以是单网卡. DS 必须得双网卡DS需要一个Wan 来接收用户的客户端请求.

		⦿ 数据封包/解包

				• 封包
						DS 首先把客户端请求进行封包.这样DS才能把客户端的请求转发给RS.
						数据封装需要在DS上配置 ipvsadm (负载均衡功能.对客户端的请求进行分配) 
								比如对DS服务器那个端口收到的数据进行封包.
								还要配置封包的目标服务器: 把封好的包转发给哪个RS服务器的哪个端口.

				• 解包
						RS 收到DS封装后的数据包.必须进行解包操作,对数据报进行还原!!
						不解包的话就是 RS 和 DS之间的交流了.会把结果返回给DS. 我们的目的是RS 和 CIP 交流.
						解包是需要预先 在 RS 服务器上配置的!!!  
						也就是在RS 上添加一个 Tunel 网卡. 里面设置DIP的IP.

   当一个数据包到达real server，
	 real server首先需要解包，
	 解包后发现此包的来源地址为 DIP.的某个端口.
	 由于我们在RS上配置了 tunel 网卡
	 那么tunel将处理此请求此后将响应结果直接返回给用户终端。
	 其中最重要的一步就是将所有的real server的“tunnel”网络接口增加VIP地址的配置。
	 比如负载均衡器的DIP地址为“202.103.106.5”，那么在real server的“tunnel”网卡需要增加此DIP




🔸 创建隧道

隧道一旦建立，数据就可以通过隧道发送。隧道客户端和服务器使用隧道数据传输协议准备传输数据。
例如，当隧道客户端向服务器端发送数据时，
客户端首先给负载数据加上一个隧道数据传送协议包头，然后把封装的数据通过互联网络发送，
并由互联网络将数据路由到隧道的服务器端。
隧道服务器端收到数据包之后，去除隧道数据传输协议包头，然后将负载数据转发到目标网络。




在 Linux 中 IP Tunnel 的实现也分为两个部件：封装部件和解封部件，分别司职发送和接收。但这两个部分是在不同的层次以不同的方式实现的。

我们知道，
每一个 IP 数据包均交由 ip_rcv 函数处理，在进行一些必要的判断后， ip_rcv 对于发送给本机的数据包将交给上层处理程序。
对于 IPIP 包来说，其处理函数是 ipip_rcv （就如 TCP 包的处理函数是 tcp_rcv 一样， IP 层不加区分）。
也就是说，当一个目的地址为本机的封包到达后， ip_rcv 函数进行一些基本检查并除去 IP 头，然后交由 ipip_rcv 解封。
ipip_rcv 所做的工作就是去掉封包头，还原数据包，然后把还原后的数据包放入相应的接收队列（ netif_rx() ）。 

 从以上 IP Tunnel 实现的思想来看，思路十分清晰，但由于 IP Tunnel 的特殊性，其实现的层次并不单纯。实际上，它的封装和解封部件不能简单地象上面所说的那样分层。 tunl 设备虽应算进链路层，但其发送程序中做了更多的工作，如制作 IPIP 头及新的 IP 头（这些一般认为是传输层或网络层的工作），调用 ip_forward 转发新包也不是一个网络设备应当做的事。可以说， tunl 借网络设备之名，一把抓干了不少工作，真是 ‘ 高效 ’ 。而解封部件宏观上看在网络层之上，解出 IPIP 头，恢复原数据包是它分内的事，但在它解出数据包（即原完整的协议数据包）后，它把这个包放入相应的协议接收队列。这种事可不是一个上层协议干的，这是网络设备中断接收程序的义务。看到了，在这点上，它好象到了数据链路层。 








🔸 配置要求.


                          |
                          |
                      eth0|192.168.0.30
                    +----------+
--------------------|    LVS   |----------------------
                    +-----+----+
                      eth1|10.0.0.30
                          |
+------------+            |             +------------+
|  Backend01 |10.0.0.51   |    10.0.0.52|  Backend02 |
| Web Server +------------+-------------+ Web Server |
|            |eth0                  eth0|            |
+------------+                          +------------+








🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 负载均衡 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸


🔸 LVS 负载均衡

		负载均衡集群是 load balance 集群的简写，翻译成中文就是负载均衡集群。
		常用的负载均衡开源软件有nginx、lvs、haproxy，商业的硬件负载均衡设备F5、Netscale。

		LB集群的架构和原理很简单，就是当用户的请求过来时，会直接分发到Director Server上，
		然后它把用户的请求根据设置好的调度算法，智能均衡地分发到后端真正服务器(real server)上。
		为了避免不同机器上用户请求得到的数据不一样，需要用到了共享存储，这样保证所有用户请求的数据是一样的。

		LVS是 Linux Virtual Server 的简称，也就是Linux虚拟服务器。
		现在 LVS 已经是 Linux 内核标准的一部分。
		使用 LVS 可以达到的技术目标是：
		通过 LVS 达到的负载均衡技术和 Linux 操作系统实现一个高性能高可用的 Linux 服务器集群，它具有良好的可靠性、可扩展性和可操作性。从而以低廉的成本实现最优的性能。
		LVS 是一个实现负载均衡集群的开源软件项目，LVS架构从逻辑上可分为调度层、Server集群层和共享存储。



🔸 LVS的基本工作原理

		1. 当用户向负载均衡调度器（Director Server）发起请求，调度器将请求发往至内核空间
		2. PREROUTING链首先会接收到用户请求，判断目标IP确定是本机IP，将数据包发往INPUT链
		3. IPVS是工作在INPUT链上的，当用户请求到达INPUT时，IPVS会将用户请求和自己已定义好的集群服务进行比对，如果用户请求的就是定义的集群服务，那么此时IPVS会强行修改数据包里的目标IP地址及端口，并将新的数据包发往POSTROUTING链
		4. POSTROUTING链接收数据包后发现目标IP地址刚好是自己的后端服务器，那么此时通过选路，将数据包最终发送给后端的服务器


🔸 LVS的组成

		LVS 由2部分程序组成，包括 ipvs 和 ipvsadm。

		1. ipvs(ip virtual server)：一段代码工作在内核空间，叫ipvs，是真正生效实现调度的代码。
		2. ipvsadm：另外一段是工作在用户空间，叫ipvsadm，负责为ipvs内核框架编写规则，定义谁是集群服务，而谁是后端真实的服务器(Real Server)




🔸 HTTP重定向负载均衡
		当用户发来请求的时候，
		Web服务器通过修改HTTP响应头中的Location标记来返回一个新的url，
		然后浏览器再继续请求这个新url，实际上就是页面重定向。通过重定向，来达到“负载均衡”的目标。
		例如，我们在下载PHP源码包的时候，点击下载链接时，
		为了解决不同国家和地域下载速度的问题，它会返回一个离我们近的下载地址。重定向的HTTP返回码是302。


🔸 DNS域名解析负载均衡
		一个域名是可以配置成对应多个IP的。因此，DNS也就可以作为负载均衡服务。


🔸 反向代理负载均衡


🔸 IP 负载均衡

		客户端肯定是只能连一个IP的. 
		连proxysql-server1 的 IP1 或者 proxysql-server-2 的 IP2中的任何一个都不好.
		最好的办法是连一个额外的IP: IP3.
		这个IP3 不是绑定proxysql1. 也不是绑定proxysql2. 而是动态绑定的.
		如果proxysql1 和 proxysql2 都正常那么就随便绑定一个.
		如果某个proxysql 挂了. 那么IP3 就会绑定到 没挂的那个proxysql上.
		这样就实现了高可用!!
		高可用有很多种方法: LVS 和 keepalived 都可以.







🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸 LVS 🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 why lvs
		主从 和 读写分离我们都弄好了. 下面我们要考虑.. 高性能了.
		读写分离 用的 proxysql 是一个入口. 
		这个入口往往是性能瓶颈!! 因为这只是一台服务器.而不是集群.		
		而且如果这个proxysql入口坏了 整个项目也就挂了.
		我们先来解决性能问题. 然后再解决. proxysql的 高可用.

		要解决性能问题.那么我们就要搭建多个 proxysql,来实现负载均衡.多个 proxysql 就有多个IP地址.
		mysql 的负载均衡可以用proxysql读写分离来实现.
		proxysql 的负载均衡就得用 LVS 来实现了!!! 
		lvs 的角色和 proxysql 差不多. 
		LVS可以实现负载均衡，但是不能够进行健康检查，
		如果某个 proxysql 挂了. LVS 是判断不出来的.  这时候就要用到 keepalived技术了.

		keepalive 可以进行健康检查，而且能同时实现 LVS 的高可用性，解决 LVS 单点故障的问题，其实 keepalive 就是为 LVS 而生的。
		keepalive 可以进行健康检查，而且能同时实现 LVS 的高可用性，解决 LVS 单点故障的问题，其实 keepalive 就是为 LVS 而生的。
		keepalive 可以进行健康检查，而且能同时实现 LVS 的高可用性，解决 LVS 单点故障的问题，其实 keepalive 就是为 LVS 而生的。



🔸 LVS 要求

		❗️❗️❗️只有 TUN 技术可以用在 局域网和广域网,其他两种只能在 局域网中.❗️❗️❗️
		❗️❗️❗️只有 TUN 技术可以用在 局域网和广域网,其他两种只能在 局域网中.❗️❗️❗️
		❗️❗️❗️只有 TUN 技术可以用在 局域网和广域网,其他两种只能在 局域网中.❗️❗️❗️


		⦿ IP 要求
				服务器IP 肯定需要两个的.
				至于是两个公网IP 还是两个内网IP 还是一公一内都可以.

				一般来说. 集群是安装在内网的! 一个机房里面很多服务器.
				lvs 管理机器 需要有个外网IP. 这样用户才能连到集群中.


		⦿ 网卡要求
				两个IP 不代表必须两个网卡.  一个网卡也可以设置双IP的 



🔸 IP隧道

		IP隧道(IP tunneling)是将一个IP报文封装在另一个IP报文的技术，
		这可以使得目标为一个IP地址的数据报文能被封装和转发到另一个IP地址。
		IP隧道技术亦称为IP封装技术(IP encapsulation)。
		IP隧道主要用于移动主机和虚拟私有网络(Virtual Private Network)，
		在其中隧道都是静态建立的，隧道一端有一个IP地址，另一端也有唯一的IP地址。
		它的连接调度和管理与VS/NAT中的一样，只是它的报文转发方法不同。
		调度器根据各个服务器的负载情况，动态地选择一台服务器，将请求报文封装在另一个IP报文中，再将封装后的IP报文转发给选出的服务器;服务器收到报文后，先将报文解封获得原来目标地址为 VIP 的报文，服务器发现VIP地址被配置在本地的IP隧道设备上，所以就处理这个请求，然后根据路由表将响应报文直接返回给客户。






🔸 LVS/Tun 模式

		⦿ 环境:  
			• 客户端       ip: CIP
			• lvs 节点     ip: DIP
			• proxyssql_1  ip: RIP1
			• proxysql_2   ip: RIP2


		⦿ 流程
				客户端 ➜ lvs 节点 ➜ 某proxysql 节点.



		客户端连到 lvs.. lvs 再分配给 proxysql...
		但是 lvs 节点也会挂的啊.. 挂了还是挂了啊...


		用户请求到达 lvs, 这里用户请求数据包的 来源IP是用户的IP. 目的IP是lvs 的IP.
		然后.. lvs 在这数据包之外再加一层. 来源IP还是用户的IP. 目的IP是某台proxysql 的IP


TUN模式服务概述：
     IP Tunneling(IP隧道) --可以在不同地域，不同网段
     Director分配请求到不同的real server。real server处理请求后直接回应给用户，这样director负载均衡器仅处理客户机与服务器的一半连接。IP Tunneling技术极大地提高了director的调度处理能力，同时也极大地提高了系统能容纳的最大节点数，可以超过100个节点。real server可以在任何LAN或WAN上运行，这意味着允许地理上的分布，这在灾难恢复中有重要意丿。服务器必须拥有正式的公网IP地址用于不客户机直接通信，并且所有服务器必须支持IP隧道协议。





lvs-tun是在请求报文又封装了一层ip报文用于隧道传输，工作方法如下：

客户端发送请求到lvs负载均衡器（源ip：CIP，目标ip：VIP）–> 通过调度算法选定一台公网上的RS，然后在请求报文外再封装一层ip首部，源ip为DIP，目标IP为RIP –> RS接收到报文，拆包得到原始请求报文（源ip：CIP，目标ip：VIP），然后用本机的VIP发送响应报文到CIP完成通信。




🔸 轮叫调度 rr
		这种算法是最简单的，就是按依次循环的方式将请求调度到不同的服务器上，该算法最大的特点就是简单。
		轮询算法假设所有的服务器处理请求的能力都是一样的，
		调度器会将所有的请求平均分配给每个真实服务器，不管后端 RS 配置和处理能力，非常均衡地分发下去。






🔸 Ipvsadm命令：


  管理集群服务
               添加：-A -t|u|fservice-address [-s scheduler]
                        -t:TCP协议的集群
                        -u:UDP协议的集群
                                 service-address:     IP:PORT
                        -f:FWM: 防火墙标记
                                 service-address:Mark Number
               修改：-E
               删除：-D -t|u|fservice-address

               #ipvsadm -A -t 172.16.100.1:80 -s rr




  管理集群服务中的RS
               添加：-a -t|u|fservice-address -r server-address [-g|i|m] [-w weight]
                          -t|u|f service-address：事先定义好的某集群服务
                          -r server-address: 某RS的地址，在NAT模型中，可使用IP：PORT实现端口映射；
                          [-g|i|m]: LVS类型      
                                 -g:DR
                                 -i:TUN
                                 -m:NAT
                        [-wweight]: 定义服务器权重
               修改：-e
               删除：-d -t|u|fservice-address -r server-address

               #ipvsadm -a -t 172.16.100.1:80 -r 192.168.10.8 –g
               #ipvsadm -a -t 172.16.100.1:80 -r 192.168.10.9 -g
     查看
               -L|l
                        -n:数字格式显示主机地址和端口
                        --stats：统计数据
                        --rate:速率
                        --timeout:显示tcp、tcpfin和udp的会话超时时长
                        -c:显示当前的ipvs连接状况

     删除所有集群服务
               -C：清空ipvs规则
     保存规则
               -S
               #ipvsadm -S &gt; /path/to/somefile
     载入此前的规则：
               -R
               # ipvsadm -R &lt;/path/form/somefile




🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  LVS/Tun 实例  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

🔸 拓扑: 
		客户端 ➜ lvs 负载均衡器(gce) ➜ vps1: proxysql_1 / vps2 proxysql_2


		Directory：
				Real IP 35.194.128.92
				Virtual IP (VIP): 4.3.2.1 (an add-on IP assigned to your server)

		RealServer:
				Real IP 23.105.192.96
		RealServer:
				Real IP 104.224.139.45  


🔸 实验目标
		1：正确理解TUN的工作原理  
		2：使用LVS+TUN搭建集群实现负载均衡
		3：使用webbench测试LVS-TUN集群性能



🔸 开启 ipv4 转发
		vi /etc/sysctl.conf
		net.ipv4.ip_forward = 1
		sysctl -p


🔸 ipvsadm 安装:    yum -y install ipvsadm -y





🔸 配置网卡文件 绑定VIP.

root@web01:~# ifconfig tunl0 192.168.10.254 broadcast 192.168.10.254 netmask 255.255.255.255
 ?????

🔸 激VIP.

🔸 编辑/etc/sysctl.conf 文件，启用系统转发功能：



5、配置LVS成TUN 模式：



⦿ 查看负载平衡 ipvsadm

🔸 添加配置.

ipvsadm -A -t 35.194.128.92:6033 -s rr
ipvsadm -a -t 35.194.128.92:6033 -r 23.105.192.96 -i
ipvsadm -a -t 35.194.128.92:6033 -r 104.224.139.45 -i

我们mysqlproxy 用的是 6033端口.
		在终端里输入下面三个命令 就可以配置 ipvsadm了.

			• rr 调度方法. round-robin (rr)
			•	35.194.128.92  换成你自己VIP.  load balancer VIP, 
			•	23.105.192.96  换成你第一个proxysql 的IP.
			•	104.224.139.45 换成你第一个proxysql 的IP.




客户都就不要设置 ipvsadm 的么...
....
要添加网卡?? 添加虚拟IP??


Real Servcer1--172.16.2.24主机网卡配置：
# cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
HWADDR=00:0C:29:7F:32:0F
TYPE=Ethernet
UUID=2d67590f-694e-4491-9e8c-d7757ca7e5c0
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=172.16.2.24
PREFIX=24
GATEWAY=172.16.2.31





🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸  ipvsadm 使用  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸
⦿ 查看当前配置.   ipvsadm
⦿ 清空所有配置.   ipvsadm -C
⦿ 保存所有配置.   ipvsadm -S

⦿ 集群配置.
		ipvsadm -A -t 35.194.128.92:http -s rr
				-A 添加集群配置.
				-t tcp集群
				-u udp集群.
				35.194.128.92:http IP:port
				-s 模式

⦿ 节点配置.
		ipvsadm -a -t 35.194.128.92:http -r 23.105.192.96 -i
				-a 添加节点配置.
				-t 上一步事先定义好的集群配置 IP+端口
				-r 从节点的地址.
				-i LVS类型. TUN类型.
		
		ipvsadm -a -t 35.194.128.92:6033 -r 104.224.139.45 -i






⦿ 查看负载平衡 ipvsadm

		✘✘∙GCE ~ ➜ ipvsadm
		IP Virtual Server version 1.2.1 (size=4096)
		Prot LocalAddress:Port Scheduler Flags
			-> RemoteAddress:Port           Forward Weight ActiveConn InActConn
		TCP  92.128.194.35.bc.googleuserc rr
			-> mail.0214.help:http          Tunnel  1      0          0
			-> 104.224.139.45.16clouds.com: Tunnel  1      0          0

				大概就是: 到lvs的http请求包. 都会用 ip tune 技术 转发到别的服务器.
		下面就是从节点的设置了.
		每个从节点都需要一个 tunel 接口 来和lvs 交流.
		大概意思是双向连接. 然后.. 就可以了....




🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸   vps1 设置  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸

TUN 模式不需要设置客户端吧. 





⦿ 新建 tunlo 接口
		vi /etc/sysconfig/network-scripts/ifcfg-tunl0

DEVICE=tunl0
BOOTPROTO=static
IPADDR=35.194.128.92
NETMASK=255.255.255.255
ONBOOT=yes
HWADDR=00:0c:29:a3:3e:16


ipaddr     ➜  替换成vip 的IP就可以了.



⦿ 重启网络   service network restart



⦿ 启用 tunl0 接口 		ifup tunl0

👹 /etc/sysconfig/network-scripts/ifup-eth] Device tunl0 does not seem to be present, delaying initialization
估计是哪里没设置对.
难道少了什么参数.
BROADCAST=192.168.1.255
HWADDR=00:0c:29:a3:3e:16

IPADDR=192.168.1.75
NETMASK=255.255.255.0

NETWORK=192.168.1.0

ONBOOT=yes
GATEWAY=192.168.1.1



或许是少了 mac地址. 要复制个别的mac地址么..
重启vps　试试.. 





⦿ 查看网卡配置   ifconfig
		看起来应该类似下面的. 
			tunl0     Link encap:IPIP Tunnel  HWaddr   
								inet addr:4.3.2.1  Mask:255.255.255.255
								UP RUNNING NOARP  MTU:1480  Metric:1
								RX packets:2 errors:0 dropped:0 overruns:0 frame:0
								TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
								collisions:0 txqueuelen:1 
								RX bytes:80 (80.0 B)  TX bytes:0 (0.0 B)




🔸 反向路由过滤设置(rp_filter)

⦿ 简介
		Linux的rp_filter用于实现反向过滤技术，也即uRPF，
		它验证反向数据包的流向，以避免伪装IP攻击，但是它和Linux的策略路由却很容易发生冲突，
		其本质原因在于，uRPF技术强制规定了一个反向包的“方向”，而实际的路由是没有方向的。
		策略路由并没有错，错就错在uRPF增加了一个路由概念本身并没有且从不考虑的约束。

		假定网络条件为
		eth0  MAC0, IP0，直连网络为 NET0，网关 GW0 
		eth1  MAC1, IP1，直连网路为 NET1，网关 GW1

		rp_filter = 1 意味着开启 strict mode 反向路径过滤（主要是防 IP 地址欺骗）。
		这时如果这台主机从 eth0 收到来自 NET1 的数据包，会丢弃，因为查找路由表发现 NET1 的数据包应该从 eth1 收到而不是 eth0。
		我们这里 需要关闭这个功能...


⦿ 关闭反向路由过滤功能

		vi /etc/sysctl.conf   添加.

		net.ipv4.conf.all.rp_filter = 0
		net.ipv4.conf.eth0.rp_filter = 0
		net.ipv4.conf.tunl0.rp_filter = 0



sysctl -p






🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸   vps2 设置  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸









🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸   测试.  🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸🔸
既然只设置了gce端. 那么..来测试吧.

之前我们的 proxysql 是安装在vps1中的. 
我们可以通过 		mysql -u sbuser -psbpass -h 23.105.192.96 -P 6033 登录vps1的 musql.
现在我们的 gce 开启了数据转发.
我们应该可以在任何电脑 
mysql -u sbuser -psbpass -h 35.194.128.92 -P 6033
也能登录 msql.. ???? 因为到gce 6033 端口的数据会自动转发到vps1中啊.
为啥登录不了呢..
也不对.. 估计没转发成功...














































⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵  ProxySQL Keepalived 高可用❌  🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

👁‍🗨 keepalived 需要服务器有双网卡. 双IP . 
	这个双IP那么都是内网IP, 要么都是外网IP.
	所以就算谷歌的GCE有双网卡.但是一个是内网IP,一个是外网IP. 也不行的...
	所以... 折腾不了.



读写分离好设置好了. 现在这个架构还远远不够完美.

现在 vps2 节点是连到 vps1 的proxysql_1 的.
现在 gce  节点是连到 vps2 的proxysql_2 的.

虽然搭建了两个 proxysql. 但是只要有任何一个 proxysql 挂掉.必然会有一个节点不能翻墙.
有没有更好的解决方案呢.
keepalived 可以帮你实现.当然也可以用 HeartBeat、RoseHA 但是这些配置麻烦.


⦿ Keepalived
		Keepalived是Linux下一个轻量级的高可用解决方案
		Keepalived主要是通过虚拟路由冗余来实现高可用功能，
		Keepalived部署和使用非常简单，所有配置只需一个配置文件即可完成。
		Keepalived作为一个高性能集群软件，它还能实现对集群中服务器运行状态的监控及故障隔离。

⦿ keepalived 三大模块
		• core 模块 核心模块，负责主进程的启动、维护以及全局配置文件的加载和解析。
		• check模块 负责健康检查，包括常见的各种检查方式。
		• vrrp 模块 实现VRRP协议的。


⦿ keepalived 功能
		• 将IP地址飘移到其他节点上，
		• 在另一个主机上生成ipvs规则
		• 健康状况检查

⦿ keepalived 安装
		
		yum install keepalived -y

		我们的目的是让客户端连接 proxysql_1 proxysql_2 中的任何一个
		如果某个 proxysql 挂了. 就自动连到正常的那个proxysql
		这样就实现了高可用.

		我们需要在安装有 proxysql 的服务器安装 keepalived.
		也就是 vps1 和 vps2 都要安装 keepalived




🔸 VPS1 keepalived 配置   vi /etc/keepalived/keepalived.conf


! Configuration File for keepalived

global_defs {
   notification_email {
     root@localhost
   }
   notification_email_from keepalived@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
	 # ❗️ 这里两个文件不一样. 一个是 node0 一个是node1
   router_id node0
   vrrp_mcast_group4 224.1.101.23

}

#存在文件时，检测成功，即执行降级；否则不存在,全部退出；实现服务器切换
vrrp_script chk_down{
    script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
    interval 1
    weight -10
    fall 1
    rize 1
}


#脚本，健康状态检测，检测proxysql是否存活
vrrp_script chk_proxysql {   
    script "killall -0  proxysql && exit 0 || exit 1"
    interval 1
    weight -10
    fall 1
    rise 1
}

vrrp_instance sr1 {
		#❗️ 下面也不一样. 一个是Master 一个是backup
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
		#❗️ 这个也不一样. 应该数字越大. 优先级越高.
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass rEiszbuO
    }
    virtual_ipaddress {
        192.168.42.182/24 dev ens33 label ens33:0
    }

    #脚本调用
    track_script {
        chk_down
        chk_proxysql
    }

    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"

}









🔸 vps2 的 keepalived 配置   vi /etc/keepalived/keepalived.conf

! Configuration File for keepalived

global_defs {
   notification_email {
     root@localhost
   }
   notification_email_from keepalived@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id node1
   vrrp_mcast_group4 224.1.101.23

}

#存在文件时，检测成功，即执行降级；否则不存在,全部退出；实现服务器切换
vrrp_script chk_down{
    script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
    interval 1
    weight -10
    fall 1
    rize 1
}


#脚本，健康状态检测，检测proxysql是否存活
vrrp_script chk_proxysql {   
    script "killall -0 proxysql && exit 0 || exit 1"
    interval 1
    weight -10
    fall 1
    rise 1
}

vrrp_instance sr1 {
	  #❗️ 下面也不一样. 一个是Master 一个是backup
    state BACKUP
    interface ens33
    virtual_router_id 51
		#❗️ 这个也不一样. 应该数字越大. 优先级越高.
    priority 96
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass rEiszbuO
    }
    virtual_ipaddress {
        192.168.42.182/24 dev ens33 label ens33:0
    }

    #脚本调用
    track_script {
        chk_down
        chk_proxysql
    }

    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"

}








🔸 notify.sh脚本
这个脚本 手动创建 放到 vps1 和vps2 的 /etc/keepalived/目录下.

vi /etc/keepalived/notify.sh


#!/bin/bash
#
contact='root@localhost'
notify() {
     mailsubject="vrrp:$(hostname) to be $1"
     mailbody="$(hostname) to be $1,vrrp transition, $(date)."
     echo "$mailbody" | mail -s "$mailsubject" $contact
}
    case $1 in
    master)
      notify master
      service proxysql  start
      ;;
    backup)
      notify backup
      service proxysql  start
      ;;
    fault)
      notify fault
      service proxysql  stop
      ;;
    *)
      echo "Usage: $(basename $0) {master|backup|fault}"
      exit 1
      ;;
esac







🔸 keepalived 公网IP 👹 👹

内网参考   http://www.178linux.com/80462
公网    找不到参考...



(5).因为keepalived是引用漂移ip地址,所以,我们上面配置的proxysql.conf的IP绑定需要修改
记得是node1和node2都要修改哦!

这个IP是什么鬼....

keepalived 原理是创建一个虚拟IP: vip 
然后让vps1 vps2 去抢这个vip.
客户端是连到这个vip的. 
哪个proxysql服务器抢到这个vip. 那个proxysql服务器就会处理客户端请求.

关键问题是 两台服务器是公网IP. 每台服务器只有一个网卡!
如果你的 集群都是在内网的 那么好说.. 安装个网卡. 分配一个空闲的内网IP.

不管怎么说. 都需要第三个公网IP.
客户端连这第三个公网IP. 然后 再分配给 vps1 或者vps2..



VIP 是 这个KEEPALIVED组对外暴露的IP，外部访问的时候访问的是VIP，当一个机子挂掉，会自动漂移到另外一台机子上

V IP 是单独的虚拟IP，对外暴露的，假如你的 VIP是内网的IP 那你的有个固定的IP需要映射到内网的这个VIP上




比如 vip 是gce的ip.
让



❗️❗️ 解决方案思路:  lvs+keepalived群集配置

❗️❗️ 解决方案思路:  lvs+keepalived群集配置
❗️❗️ 解决方案思路:  lvs+keepalived群集配置
❗️❗️ 解决方案思路:  lvs+keepalived群集配置
http://www.jianshu.com/p/e146a7a14b4b















⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵  MHA 高可用 ❌ 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

参考文章 http://www.178linux.com/80462

http://yangxikun.com/mysql/2016/05/02/mysql-mha.html




🔸 mysql 高可用.

		现在 读写分离差不多了 . 下面我们来弄 mysql 节点的高可用.
		上面的架构中. 主节点是不能挂的. 挂了就不能写入数据了. 
		这时候我们可以用MHA技术,将从节点自动提升为主节点. 实现高可用.
		如果主节点挂了. 那么就把从节点单成主节点! 来实现高可用. 

		MHA（Master HA） 在监控到 master 节点故障时，会提升其中拥有最新 数据的 slave 节点成为新的 master 节点，
		在此期间，MHA 会通过于其它从节点获取额外信 息来避免一致性方面的问题。
		MHA 还提供了 master 节点的在线切换功能，即按需切换 master/slave 节点。

		MHA 服务有两种角色，MHA Manager(管理节点)和 MHA Node(数据节点)： 
		MHA Manager：通常单独部署在一台独立机器上管理多个 master/slave 集群，每个 master/slave 集群称作一个 application； 
		MHA node：   运行在每台 MySQL 服务器上(master/slave/manager)，它通过监控具备解析 和清理 logs 功能的脚本来加快故障转移。

		❗️❗️目前MHA主要支持一主多从的架构，要搭建MHA,要求一个复制集群中必须最少有三台数据库服务器，❗️❗️
		一主二从，即一台充当master，一台充当备用master，另外一台充当从库，
		因为至少需要三台服务器，出于机器成本的考虑，淘宝也在该基础上进行了改造，目前淘宝TMHA已经支持一主一从。


🔸 MHA 安装位置.
		MHA Manage 一般是安装在单独的服务器中的. 不是安装在 mysql 集群中的.
		因为如果你的manage 安装在 mysql 集群中的某个服务器中. 
		当那个服务器死机的时候. MHA manger 也就挂了. 也就无效了


		我们用 GCE  开个临时的服务器: gce2 带. MHA Manger 
		Manager 需要后台连接 所有主从节点. 所以需要配置 密钥.
		让gce2 这个 MHA manager 可以连接所有 主从节点.
		也就需要把 gce2 的公钥放到 所有主从节点的 authorized_keys 中


🔸 Manager 节点密钥设置.
		任何一个节点 生成一对密钥.
		把这对密钥放到别的服务器的.ssh 文件夹下就可以 互相 ssh 登录了.
		最终实现 

		vps1 可以 ssh vps2、gce、gce2
		vps2 可以 ssh vps1、gce、gce2
		gce  可以 ssh vps1、vps2、gce2
		gce2 可以 ssh vps1、vps2、gce


		首先清空所有服务器上的原有的密钥+公钥.
		vps1 节点创建密钥,然后把密钥复制给集群里的所有其他节点: gce gce2 vps2

⦿ vps1 
		cd .ssh && ls      ➜ 先看看之前有没有生成过密钥. 有的话可以直接用.
		ssh-keygen         ➜ 没有的话. 一路回车 就生成了
		cd .ssh

		把公钥添加到 vps1 的authorized_keys文件上 
				ssh-copy-id  -i id_rsa.pub -p 2222 root@23.105.192.96
				添加后就可以免密码登录了vps1. 
				ssh -p '2222' 'root@23.105.192.96'

		把公钥添加到 vps2 的authorized_keys文件上 
				ssh-copy-id  -i id_rsa.pub -p 2222 root@104.224.139.45
				添加后就可以免密码登录了vps1. 
				ssh -p '2222' 'root@104.224.139.45'

		把公钥添加到 GCE 的authorized_keys文件上 
				ssh-copy-id  -i id_rsa.pub -p 2222 root@35.194.128.92
				添加后就可以免密码登录了vps2. 
				ssh -p '2222' 'root@35.194.128.92'

		把公钥添加到 GCE2 的authorized_keys文件上 
				ssh-copy-id  -i id_rsa.pub -p 2222 root@104.199.220.60
				添加后就可以免密码登录了gce2. 
				ssh -p '2222' 'root@104.199.220.60'

		上面的操作 只能让 vps1 可以ssh 其他服务器. 其他服务器是不能ssh vps1的
		要让所有服务器可以互相ssh 需要所有放服务器用相同的私钥.也就是vps1 生成的密钥对
		我们直接把私钥和公钥 全部上传给别的服务器.就可以了.


		vps1 密钥上传给 vps2 
				scp -P 2222  id_rsa id_rsa.pub root@104.224.139.45:~/.ssh/

		vps1 密钥上传给 gce   
				scp -P 2222  id_rsa id_rsa.pub root@35.194.128.92:~/.ssh/

		vps1 密钥上传给 gce2   
				scp -P 2222  id_rsa id_rsa.pub root@104.199.220.60:~/.ssh/








🔸 Manager 安装

		⦿ MHA Manager: gce2
				需要安装 mha4mysql-manager  mha4mysql-node
				先安装依赖 ➜ 再安装node  ➜ 最后安装 manager


				• 安装 node 所需依赖
				yum install perl-DBD-MySQL perl-Module-Install cpan perl-DBI -y

				• 安装manager所需依赖
				yum   install -y perl perl-Config-Tiny perl-Email-Date-Forma perl-Log-Dispatch perl-MIME-Liteperl-MIME-Types perl-Mail-Sender perl-Mail-Sendmail perl-MailTools perl-Parallel-ForkManagerperl-Params-Validate perl-Time-HiRes perl-TimeDate

				• 安装 MHA node 
				curl http://www.mysql.gr.jp/frame/modules/bwiki/index.php\?plugin\=attach\&pcmd\=open\&file\=mha4mysql-node-0.56-0.el6.noarch.rpm\&refer\=matsunobu -o mha4mysql-node-0.56-0.el6.noarch.rpm
				yum localinstall -y mha4mysql-node-0.56-0.el6.noarch.rpm

				• 安装 MHA manager 
				curl http://www.mysql.gr.jp/frame/modules/bwiki/index.php\?plugin\=attach\&pcmd\=open\&file\=mha4mysql-manager-0.56-0.el6.noarch.rpm\&refer\=matsunobu -o mha4mysql-manager-0.56-0.el6.noarch.rpm
				yum localinstall -y mha4mysql-manager-0.56-0.el6.noarch.rpm



		⦿ MHA node:  vps1、vps2、gce 
				只需安装 mha4mysql-node

				• 安装 node 所需依赖
				yum install perl-DBD-MySQL perl-Module-Install cpan perl-DBI -y

				• 安装 MHA node 
				curl http://www.mysql.gr.jp/frame/modules/bwiki/index.php\?plugin\=attach\&pcmd\=open\&file\=mha4mysql-node-0.56-0.el6.noarch.rpm\&refer\=matsunobu -o mha4mysql-node-0.56-0.el6.noarch.rpm
				yum localinstall -y mha4mysql-node-0.56-0.el6.noarch.rpm

			



🔸 Manger 配置 GCE2 
		Manger 节点需要为每个监控的 master/slave 集群提供一个专用的配置文件， 而所有的 master/slave 集群也可共享全局配置。
		全局配置文件默认为/etc/masterha_default.cnf，其为可 选配置。
		如果仅监控一组 master/slave 集群，也可直接通过 application 的配置来提供各服务 器的默认配置信息。而每个 application 的配置文件路径为自定义，



		创建配置文件  mkdir /etc/masterha && vim /etc/masterha/app1.cnf

		# ❗️配置文件详细参数用法及解释 http://wubx.net/mha-parameters/
		# ❗️配置文件详细参数用法及解释 http://wubx.net/mha-parameters/

[server default]
user=root     
password=xujian0219  
#mysql 连接的用户和密码, 需要权限较高.一般设置root.

manager_workdir=/data/masterha/app1 
# 用于指定mha manager产生相关状态文件全路径。
manager_log=/data/masterha/app1/manager.log 
# 指定mha manager的绝对路径的文件名日志文件。

remote_workdir=/data/masterha/app1
# MHA node上工作目录的全路径名。
# 如果不存在，MHA node会自动创建，如果不允许创建，MHA Node自动异常退出。

ssh_user=root 
#ssh的用户名设置

repl_user=root
# MySQL用于复制的用户，也是用于生成CHANGE MASTER TO 每个slave使用的用户。
repl_password=xujian0219
# MySQL中repl_user用户的密码。

ping_interval=1
#设置MHA Manager多长时间去ping一下master(执行一些SQL语句）. 
#当失去和master三次偿试，MHA Manager会认为MySQL Master死掉了。
#也就是说，最大的故障切换时间是４次ping_interval的时间，默认是3秒。

[server1] 
hostname=vps1
# mysql 服务器备注名. 无所谓.区分用
ip=23.105.192.96
# mysql 服务器的IP地址.必须准确
port = 3306 
# mysql 数据库的端口
ssh_port=2222
# mysql 服务器的 ssh 端口.
candidate_master=1
# 优先级.当主挂掉.那么该值为1的机器 会首先提升为主服务器.(默认值是0)
# 如果多个服务器的该值都是1, 那么优先级安装模块名来排列.
# [server1] > [server1] > [server1] 

[server2] 
hostname=vps2
ip=104.224.139.45
ssh_port=2222
candidate_master=1

[server3] 
hostname=gce
ip=35.194.128.92
ssh_port=2222
candidate_master=1






🔸 SSH 配置测试
		masterha_check_ssh --conf=/etc/masterha/app1.cnf 

		输出信息最后一行类似如下信息，表示其通过检测。 
		[info] All SSH connection tests passed successfully.





🔸 修改主节点和从节点的mysql配置 vi /etc/my.cnf


配置文件几乎都是一模一样的. 除了[mysqld]模块下面几个地方.

     binlog-do-db = ss-vps1
     replicate-do-db= ss-vps1
		 所有数据库有要有上面两行.

server_id   主是1  其他是  2 3 

从比主多一行 relay_log_purge=0
从比主多一行 read_only=ON






👹 Checking recovery script configurations on gce(35.194.128.92:3306)..
Sat Jul 22 17:43:06 2017 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/var/lib/mysql,/var/log/mysql --output_file=/data/masterha/app1/save_binary_logs_test --manager_version=0.56 --start_file=mysql-bin.000021
Sat Jul 22 17:43:06 2017 - [info]   Connecting to root@35.194.128.92(gce:2222)..
Failed to save binary log: Binlog not found from /var/lib/mysql,/var/log/mysql! If you got this error at MHA Manager, please set "master_binlog_dir=/path/to/binlog_directory_of_the_master" correctly in the MHA Manager's configuration file and try again.
 at /usr/bin/save_binary_logs line 123.


主服务器上确实没有找到 ... bin文件... 
主从同步的日志文件啊 .. 去哪了... fuck...










		⦿ 主节点 GCE  整个文件改成下面的

[client]
#password   = your_password
port        = 3306
socket      = /tmp/mysql.sock

[mysqld]
server_id = 1

binlog-do-db = ss-vps1
replicate-do-db= ss-vps1

skip_name_resolve=ON
relay-log=relay-bin

innodb_file_per_table=ON
log_bin = log-bin


port        = 3306
socket      = /tmp/mysql.sock
datadir = /usr/local/mysql/var
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
thread_cache_size = 8
query_cache_size = 8M
tmp_table_size = 16M

#skip-networking
max_connections = 500
max_connect_errors = 100
open_files_limit = 65535

binlog_format=mixed
expire_logs_days = 10

default_storage_engine = InnoDB
innodb_data_file_path = ibdata1:10M:autoextend
innodb_log_group_home_dir = /usr/local/mysql/var
innodb_buffer_pool_size = 16M
innodb_additional_mem_pool_size = 2M
innodb_log_file_size = 5M
innodb_log_buffer_size = 8M
innodb_flush_log_at_trx_commit = 1
innodb_lock_wait_timeout = 50

[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash

[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout















		⦿ 从节点 VPS1 整个文件改成  vi /etc/my.cnf



[client]
#password   = your_password
port        = 3306
socket      = /tmp/mysql.sock

[mysqld]
server_id = 11   # ❗️ 区别


binlog-do-db = ss-vps1
replicate-do-db= ss-vps1

skip_name_resolve=ON
relay-log=relay-bin
relay_log_purge=0    # ❗️ 区别. 多一行.

innodb_file_per_table=ON
log_bin = log-bin


port        = 3306
socket      = /tmp/mysql.sock
datadir = /usr/local/mysql/var
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
thread_cache_size = 8
query_cache_size = 8M
tmp_table_size = 16M

#skip-networking
max_connections = 500
max_connect_errors = 100
open_files_limit = 65535

binlog_format=mixed
expire_logs_days = 10

default_storage_engine = InnoDB
innodb_data_file_path = ibdata1:10M:autoextend
innodb_log_group_home_dir = /usr/local/mysql/var
innodb_buffer_pool_size = 16M
innodb_additional_mem_pool_size = 2M
innodb_log_file_size = 5M
innodb_log_buffer_size = 8M
innodb_flush_log_at_trx_commit = 1
innodb_lock_wait_timeout = 50

[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash

[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
















		service mysql restart























🔸 mysql 同步检查 ❗️❗️❗️
检查管理的 MySQL 复制集群的连接配置参数是否 OK：
masterha_check_repl --conf=/etc/masterha/app1.cnf


👹 There is no alive slave. We can't do failover
这里必须要有3个mysql主从服务器. 一个master. 一个masterback 一个slaver.
如果你在上面的配置文件只设置了两个 server .那么肯定有这个报错的.
去增加个服务器吧.




SQL Thread(线程) is stopped(error) on vps1(23.105.192.96:3306)! 
Errno:1062, Error:Error 'Duplicate entry '159843' for key 'PRIMARY'' on query. Default database: 'ss-vps1'. Query: 'INSERT INTO `ss_node_info_log` (`id`, `node_id`, `uptime`, `load`, `log_time`) VALUES (NULL, '2', '167514.369278', '0.00 0.00 0.00
', unix_timestamp())'

SQL Thread is stopped(error) on vps2(104.224.139.45:3306)! Errno:1062, Error:Error 'Duplicate entry '158631' for key 'PRIMARY'' on query. Default database: 'ss-vps1'. Query: 'INSERT INTO `ss_node_info_log` (`id`, `node_id`, `uptime`, `load`, `log_time`) VALUES (NULL, '2', '132207.489794', '0.00 0.00 0.00
', unix_timestamp())'

There is no alive slave. We can't do failover
GCE 上没有从数据库??? 












































































































@: 2002-03-08-MySQL
---
layout: post
title: MySQL tags: 数据库
categories: IT-Admin
---


## MySQL

MySQL 重要目录　 
数据库文件 /var/lib/mysql/ 配置文件　 | /usr/share/mysql  |（mysql.server命令及配置文件）
相关命令   | /usr/bin          | (mysqladmin mysqldump等命令) 启动脚本   | /etc/rc.d/init.d/ |（启动脚本文件mysql的目录）


修改登录密码 - 命令 usr/bin/mysqladmin -u root password 'new-password' - 格式：mysqladmin -u用户名 -p旧密码 password 新密码 
'' - 例:   给root加密码123456  
[root@test1 local]\# /usr/bin/mysqladmin -u root password 123456 注：因为开始时root没有密码，所以-p可以省略。 
'' - 测试是否修改成功  
'' [root@test1 local]()\# mysql -u root -p   
'' Enter password: (输入修改后的密码123456)


## MySQL 常用操作
'' **注意：MySQL中每个命令后都要以分号；结尾。 ** 　　　　
### 显示数据库 
''     mysql> show databases;  
> - Mysql 自带有几个数据库
- Mysql库非常重要，它里面有MySQL的系统信息 我们改密码和新增用户，实际上就是在这个库中进行操作。

### 显示数据库表 
'' mysql\> use mysql; （先选择要使用的数据库）  
'' mysql\> show tables; (显示数据库中的表)
　　
### 显示数据表内容

'' 显示表内容:  select * from 表名;  
'' 
''     例：显示 mysql库 中 user表 的内容。(MySQL用户都在此表中)  
'' 
''     Select * from user; 　

### 新建数据库 
''     例：创建一个名字xujian 的数据库.
'' 
''         create database xujian; 


### 新建表结构.
- 只是帮你建一个表格框架,没有具体内容.
- 比如 明珠医院 护士联系表
	- 只建立 工号 姓名 性别 学位  数据类型/长度 和默认值. 
	- 不会有哪个护士的具体工号名字 什么的. 


- 命令格式 :
- create table 表名 (字段名 类型 数据宽度 是否为空 是否主键 自动增加 默认值)     实例:
	'' create table mzyy (  
	'' >id int(4) not null primary key auto_increment,
	'' >name char(20) not null,
	'' >sex int(4) not null default '0',
	'' >degree double(16,2));
	'' 
	'' 以上例子不能直接复制运行. 
	'' 原因是这个命令实在太长了,为了书写好看 把它分成好几行了.
	'' 终端里你 可以用 \+回车键 来另起一行 而不打断输入命令.

### 修改表结构 ( 增加,删除,修改字段 )
- 添加字段:
	'' 格式: alter table 表名 add 字段 类型 其他;  
	'' 
	'' 例: alter table mzyy add phone int(4) not null;
- 删除字段:
	'' 格式: alter table 表名 drop index phone;
  
- 修改字段:     格式: alter table 表名 chande 原名 新名 类型;
	'' 例子

### 查看表结构
'' mysql\> describe mzyy;
'' 
'' 
'' +-------+---------------+-----+---------+----------------+
'' | Field |       Type    | Null | Key | Default | Extra　　　|
'' +-------+---------------+-----+---------+----------------+
'' | id　　 | int(4)　      |　NO　| PRI | NULL　　| auto_increment |
'' | name　 | char(20)     |  NO　|　　 | NULL　　 |　　　　　　　　|
'' | sex　  | int(4)       |  NO　|　　 | 0    　　|　　　　　　　　|
'' | degree | double(16,2) | YES　|　　 | NULL　　 |　　　　　　　　|
'' +-------+---------------+-----+---------+----------------+
　
　
#### 表格内容
- 新加内容: insert into 表格名 values('字段1','字段2','字段3','字段4');

- insert into mzyy values('219','xujian','男','1971-10-01');





- 删除内容: delete from table 表格名 where id=1;









## 数据类型:

整数运算速度快 占内存少. 比较实用.  比如 123

包含小数点  要用 single double currency



| 数据类型 | 大小 ~<br>~ (字节) | 精度范围 |
|:---:|:---:|:---:|
|varchar||汉字用这个|
| Byte ~<br>~ 字节型 | 1 | 0-255 |
| Boolean ~<br>~ 布尔型/逻辑型 | 2 | True 或 False |
| Integer ~<br>~ 整数型 | 2 | -32,768 到 32767 |
| Long ~<br>~ 长整型 | 4 | -2,147,483,648 \~ 2,147,483,647 |
| Single ~<br>~ 单精度浮点型 | 4 | 负数范围:-3.402823E38 \~ -1.401298E-45 ~<br>~ 正数范围:1.401298E-45 \~ 3.402823E38 | | Double ~<br>~ 双精度浮点型 | 8 |负数范围: ~<br>~ -1.797,693,134,862,32E308 \~-4.940,656,458,412,47E-324~<br>~正数范围: ~<br>~ 4.940,656,458,412,47E-324 \~1.797,693,134,862,32E308  |
| Currency ~<br>~ 货币类型 | 8 |  -922,337,203,685,477.5808 \~ 922,337,203,685,477.5807|
| Date ~<br>~ 日期型 | 8 | 100年1月1日\~9999年12月31日 |















　
　　可用select命令来验证结果。
　　mysql\> select \* from name;
　　+----+------+------+------------+
　　| id | xm　 | xb　 | csny　　　 |
　　+----+------+------+------------+
　　|　1 | 张三 | 男　 | 1971-10-01 |
　　|　2 | 白云 | 女　 | 1972-05-20 |
　　+----+------+------+------------+
　　8、修改纪录
　　例如：将张三的出生年月改为1971-01-10
　　mysql\> update name set csny='1971-01-10' where xm='张三';
　　9、删除纪录
　　例如：删除张三的纪录。
　　mysql\> delete from name where xm='张三';
　　10、删库和删表 
　　drop database 库名; 
　　drop table 表名；
　　九、增加MySQL用户
　　格式：grant select on 数据库.\* to 用户名@登录主机 identified by "密码" 
例1、增加一个用户user\_1密码为123，让他可以在任何主机上登录，并对所有数据库有查询、插入、修改、删除的权限。首先用以root用户连入MySQL，然后键入以下命令：
　　mysql\> grant select,insert,update,delete on *.* to user\_1@"%" Identified by "123"; 
例1增加的用户是十分危险的，如果知道了user\_1的密码，那么他就可以在网上的任何一台电脑上登录你的MySQL数据库并对你的数据为所欲为了，解决办法见例2。
　　例2、增加一个用户user\_2密码为123,让此用户只可以在localhost上登录，并可以对数据库aaa进行查询、插入、修改、删除的 操作（localhost指本地主机，即MySQL数据库所在的那台主机），这样用户即使用知道user\_2的密码，他也无法从网上直接访问数据库，只能 通过MYSQL主机来操作aaa库。
　　mysql\>grant select,insert,update,delete on aaa.\* to user\_2@localhost identified by "123";
　　用新增的用户如果登录不了MySQL，在登录时用如下命令：
　　mysql -u user\_1 -p　-h 192.168.113.50　（-h后跟的是要登录主机的ip地址）
我们知道，在ms sql server中或access中， 若要查询前10条记录，使用top 10即可， 但在mysql中 不支持这个写法，它用limit 10。    
我们可以利用MySQL中 SELECT支持的一个子句——LIMIT——来完成这项功能。  LIMIT可以实现top N查询，也可以实现M至N（某一段）的记录查询，具体语法如下：  SELECT * FROM MYTABLE ORDER BY AFIELD  LIMIT offset, recnum 其中offset为从第几条（M+1）记录开始，recnum为返回的记录条数。例：  select * from mytable order by afield  limit 2, 5  即意为从第3条记录开始的5条记录。
　　
　　
十一. 常见问题：
1."Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' "错误
解决方法：
这是没启动mysql的守护进程，执行service mysqld start就行了
 


Mysql Tool:  phpmyadmin, navicast ....

Mysql 连接方式: 
- tcp/ip (通过ip/ssh 连接)
- socket (通过一个文件连接) 
	> 数据库工具和mysql 服务器在同一电脑时可用,速度比tcp 快
	> socket 文件路径: 终端:`mysql_config` 一般在 /tmp/mysql.sock


## 数据库管理



### Mysql 字段类型:

- 整数: tinyint smallint `int` bigint
- 小数: float double `decimal(m,d)` m 整个位长.d小数位长
- 字符: char `varchar` 弹性大小  最长255字符. 再长就用 备注型.
- 日期: 
	- datetime 日期+时间 2008 0918 12 00
	- date  只有日期 无时间 2008 0918
	- `timestamp`  日期转为数字.精度高.

- 备注: tinytext `text` longtext



### Mysql 基本操作:


建库: `create database xx;`
删库: `drop database xx;`


*汉字变问好:基本是 编码问题. 选 utf8 就可以了*

\`\`语句里面 必须用''  而不是 \`\`\`




改字段: rename table `原字段` to `新字段`
删字段: alert table `表` drop `原字段`
插字段: alert table `表` add `新字段` .. after `原字段.`




不能建空表. 必须写上一个字段 才能建表成功.
只有一个字段的话. 这个字段必须是主键: primary key
*主键字段下面的值是不能重复的*


`CREATE TABLE port (`
`  id int(10) not null AUTO_INCREMENT,`
`PRIMARY KEY (id)`
`)`

这个就是创建表的 最简单命令.
*auto—increment 就是自动增加字段*




`select * form calss` 
查询 class 表的 所有信息.

`select id,uid from calss`
查询 class 表里面 id 和 uid 的信息.

`selece yourClassname as name from class.`
别名.  名字太长了 浏览器显示不下 就要左右拖动 很麻烦.
这时候可以 把那个很长的名字.暂时变成很短的名字.方便浏览.



*查询 某几行 的数据. 而不是整个数据表.*

`select * form calss where id=2` 
只会出现 第二行信息.

等于 =
不等 \<\>
包含 in
不含 not in
匹配: like
范围内: between
范围外: not between
and or ()




`select * form calss where id in (1,2,4)`
只显示 124 行. 不显示第三行.
`select * form calss where id not in (1,2,4)`
显示不包含 124 行的结果

`select * form calss where 'uid' like '%王%'`
显示 uid 中 含有王字的 结果.
*`%` 前后匹配*

`select * form calss where id between 1 and 3`
显示1-3的内容
`select * form calss where id not between 1 and 3`
显示1-3之外的内容

`select * form calss where id=1 and 'name'='徐'`
同时满足 两个条件的 结果.




*排序语句:order by 正序(默认值 ASC )/倒序 desc*
这个肯定在 limit 之前. 也就是倒数第二的位置.

`select * form calss order by 'regdate' desc`
一般是按照id 排序. 可以自定义.
这个 就是按照 regdate 进行倒序排列.

`select * form calss order by 'regdate' desc, id desc`
多个参数. 多重排列. 同时满足....



*指针 Limit: 永远在语句最后面.* 显示多少条. 范围:
`select * form calss limit 3`
显示前3条. 默认是第一天开始. 
`select * form calss limit 0,3`
显示前3条.

`select * form calss limit 2,3`
显示2-3 条.



*分组语句: group by*
`select * form calss group by 'remark'`
按照 remark 字段 来分组: 发现只有 
学生 和工人 .




select
排序 分组 指针查询 计算
insert
update 修改/更新
delete





## Mysql 函数

count()   统计函数 有多少条信息.
max()   最大值 数字和日期有效.
min()  最小值
avg() 平均值
sum() 累计值

`select count(*) form calss`
这个表有多少条信息.

`select max(id) form calss`
id 中的 最大值.





*插入语句 注意字段类型.免得乱码*
insert into 表名(字段,字段2...)values(值,值2....)

id 字段累加字段 可以不写具体数字. 但是 要留''.

*值: now() 可以自动插入当前时间*


*更新语句:*
update class set id=3 where id=2
把id 2   改成3 .

*删除语句*
delete from class where id=1











## phpMyAdimn:
新建数据表. 输入名字. 和 字段数(几行). 就可以了.

具体小数字段:
类型: decimal 长度 8,3  就是总长8为. 3位是小数点.





@: 2002-03-08-MySQL Cluster 数据库集群
---
layout: post
title: MySQL Cluster 数据库集群 tags: 数据库
categories: IT-Admin
---

## MySQL Cluster 数据库集群


高性能 可扩展性 集群化数据库产品

研发的初衷 就是满足许多行业里 最严酷的应用要求.... 要求数据库可靠性 99.999%


是基于无共享的可由多台服务器组成的、同时对外提供数据管理服务的分布式集群系统。
通过合理的配置，可以将服务请求在多台物理机上分发实现负载均衡 ；
同时内部实现了冗余机制，在部分服务器宕机的情况下，整个集群对外提供的服务不受影响，从而能达到99.999%以上的高可用性。 

MySQL Cluster设计之初出于性能考虑，将数据完全存放在内存当中，因此MySQL Cluster可以当作一种分布式的内存数据库。

建议：
1.若是双主复制的模式，不用做数据拆分，那么就可以选择MHA或 Keepalive 或 heartbeat
2.若是双主复制，还做了数据的拆分，则可以考虑采用Cobar；
3.若是双主复制+Slave，还做了数据的拆分，需要读写分类，可以考虑Amoeba；

先了解一下你是否应该用MySQL集群。
减少数据中心结点压力和大数据量处理，采用把MySQL分布，一个或多个application对应一个MySQL数据库。把几个MySQL数据库公用的数据做出共享数据，例如购物车，用户对象等等，存在数据结点里面。其他不共享的数据还维持在各自分布的MySQL数据库本身中。



Management Node(管理节点)：mysql集群全局的管理者，整个集群只有一个management node,负责管理集群架构中的其他Nodes(节点)，包括控制其他节点的启停，查看节点状态等。另外还维护了集群的全局配置信息，因此在整个集群环境中应该优先于所有节点启动。管理节点启动命令为ndb\_mgmd.
Data Node(数据节点)：数据的存储节点，集群中会有多个Data Node，每个节点存储数据多个数据副本。虽然mysql集群可以设置副本数量为1，即集群没有数据冗余，所有节点的只存储各自数据片段的副本，但是这样的集群也就失去了意义，所以为了保证高可用性，建议最好是设定副本数量至少为2，或者更大，一份用于存储，一份作为冗余。系统默认值为2. 冗余副本数量越大，需要的节点数就会越多。Data node 节点启动命令为ndbd.关于存储副本的概念参考图-02（来自官网）也许有助于理解副本、冗余的含义和作用。


如果单MySQL的优化始终还是顶不住压力时，
这个时候我们就必须考虑MySQL的高可用架构(很多同学也爱说成是MySQL集群)了，
目前可行的方案有： 
一、MySQL Cluster 优势：可用性非常高，性能非常好。每份数据至少可在不同主机存一份拷贝，且冗余数据拷贝实时同步。但它的维护非常复杂，存在部分Bug，目前还不适合比较核心的线上系统，所以这个我不推荐。 
二、DRBD磁盘网络镜像方案 优势：软件功能强大，数据可在底层快设备级别跨物理主机镜像，且可根据性能和可靠性要求配置不同级别的同步。IO操作保持顺序，可满足数据库对数据一致性的苛刻要求。但非分布式文件系统环境无法支持镜像数据同时可见，性能和可靠性两者相互矛盾，无法适用于性能和可靠性要求都比较苛刻的环境，维护成本高于MySQL Replication。另外，DRBD也是官方推荐的可用于MySQL高可用方案之一，所以这个大家可根据实际环境来考虑是否部署。 
三、MySQL Replication 在实际应用场景中，MySQL Replication是使用最为广泛的一种提高系统扩展性的设计手段。众多的MySQL使用者通过Replication功能提升系统的扩展性后，通过简单的增加价格低廉的硬件设备成倍 甚至成数量级地提高了原有系统的性能，是广大MySQL中低端使用者非常喜欢的功能之一，也是许多MySQL使用者选择MySQL最为重要的原因。
比较常规的MySQL Replication架构也有好几种，这里分别简单说明下
MySQL Replication架构一：
             常规复制架构--Master-slaves，是由一个Master复制到一个或多个Salve的架构模式，主要用于读压力大的应用数据库端廉价扩展解决方案，读写分离，Master主要负责写方面的压力。 
MySQL Replication架构二：
             级联复制架构，即Master-Slaves-Slaves,这个也是为了防止Slaves的读压力过大，而配置一层二级 Slaves，很容易解决Master端因为附属slave太多而成为瓶劲的风险。 MySQL Replication架构三：
             Dual Master与级联复制结合架构，即Master-Master-Slaves，最大的好处是既可以避免主Master的写操作受到Slave集群的复制带来的影响，而且保证了主Master的单点故障。 以上就是比较常见的MySQL replication架构方案，大家可根据自己公司的具体环境来设计 ，Mysql 负载均衡可考虑用LVS或Haproxy来做，高可用HA软件我推荐Heartbeat。
 
MySQL Replication的不足：
          如果Master主机硬件故障无法恢复，则可能造成部分未传送到slave端的数据丢失。所以大家应该根据自己目前的网络规划，选择自己合理的Mysql架构方案，跟自己的MySQL DBA和程序员多沟涌，多备份(备份我至少会做到本地和异地双备份)，多测试，数据的事是最大的事，出不得半点差错，切记切记。







