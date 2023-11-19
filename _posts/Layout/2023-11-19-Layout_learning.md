---

layout: post
title: 运维工作笔记
category: 技术
tags: 运维
keywords: 运维

---

# Table of Contents

1.  [操作系统](#orgfeb9986)
    1.  [windows](#orgeb240d6)
        1.  [端口](#orgaf60f5b)
    2.  [linux](#orgb431d7d)
        1.  [linux的locale设置](#org8e09a7e)
        2.  [windows和linux之间的编码转换问题](#org51a0bef)
2.  [中间件](#orgf6067f0)
    1.  [jdk](#orgd41fd4d)
        1.  [环境变量设置](#org0d3ba09)
    2.  [tomcat](#org175cd1a)
        1.  [端口](#org91bdabc)
        2.  [内存参数](#org6db98b6)
        3.  [日志($TOMCAT/logs)](#org738f382)
        4.  [访问控制](#orgf28d931)
        5.  [入侵防护](#org3b9f39d)
    3.  [mysql](#org8424434)
        1.  [启动进程](#org32cd1e5)
        2.  [启动原理](#orgecd2cbd)
        3.  [配置文件](#org3010c54)
        4.  [参数详解](#org57c8e6e)
        5.  [mariadb简明安装](#orga0eb3b7)
        6.  [数据库定时备份](#orgb1557a3)
        7.  [Mairadb开启SSL登录并配置证书](#org8355d38)
        8.  [生成服务端和客户端的证书](#org7e71fe2)
        9.  [将证书路径配置到配置文件并重启服务](#orgb20f99d)
        10. [客户端使用证书连接或](#org0185995)
        11. [sql查询练习题](#org33a8162)
    4.  [nginx](#org3bd0ce3)
        1.  [nginx的访问部分资源：503 Service Temporarily Unavailable：](#org6dbeb8a)
    5.  [docker](#orgc3ac75d)
        1.  [镜像分片](#orgd89ab08)
    6.  [nfs](#org99e1028)
        1.  [简介](#org48ab5a2)
        2.  [工作流程](#org136ec42)
        3.  [依赖下载](#org034f0c6)
        4.  [NFS配置](#org4af5945)
3.  [实用命令](#orga5d8ea4)
    1.  [md5sum](#orgad91c99)
        1.  [简介](#orgffb9486)
        2.  [命令选项](#orgb987a5b)
        3.  [生成校检码](#orge7d562a)
        4.  [校检](#orgc173d84)
        5.  [总结](#orga3c1274)
4.  [其他经验](#org9e20295)
    1.  [编辑yml配置文件时，记得不要用打tab，而是用空格键，否则会因为格式问题产生配置格式读取错误，要确保信息正确，格式正确](#org3fbc8d1)
    2.  [博微新配置文件的加解密（需要安装JDK和weakPassword-0.0.1.jar）](#org4357f87)
        1.  [对明文进行加密(执行会得到密文内容)](#orgad282a2)
        2.  [添加"-d"选项对密文解密(生成解密内容)，添加">"指定明文输出到的文件(不加打印到终端)](#orgcd5821b)
    3.  [文件解压缩的规范注意](#org2a0aa9c)
        1.  [问题：在centos解压文件： tar -xvf mysql-5.7.27-aarch64.tar.gz -C *usr/local*](#orgbd6e5df)
        2.  [解决办法：](#orga24d56a)
    4.  [Navicat运行sql文件时需要双击数据库将信息项给展开，否则“运行sql文件”是灰色的。](#orgd9d0b4a)
    5.  [Linux更换jar包calss文件：](#orgacfddf9)
        1.  [要替换的文件在jar包的二级及以下目录下，则需要以下步骤(比如说要将：gzdwpg-biz-1.0.0.jar包中的ProjectBatchCreateServiceImpl.class替换新的class文件)：](#org30c0666)
        2.  [jar命令参数学习：https://www.cnblogs.com/liyanbin/p/6088458.html](#org1841a93)
    6.  [DOP文件预览服务：](#org3c551b6)
    7.  [字体问题：](#org19b0d4d)


<a id="orgfeb9986"></a>

# 操作系统


<a id="orgeb240d6"></a>

## windows


<a id="orgaf60f5b"></a>

### 端口

1.  端口查看

    netstat -ano
    netstat -aon|findstr "8081

2.  找到端口对应PID（最后一栏）

3.  根据PID查找进程

        tasklist|findstr "9088"  

4.  结束进程

        # (强制（/F参数）杀死 pid 为 9088 的所有进程包括子进程（/T参数）)
        taskkill /T /F /PID 9088  

5.  端口启用和禁用：

    windows防火墙->高级设置->出站规则->端口->允许连接和禁用对应开启和关闭

6.  windos系统常用端口辨析

    1.  135、137、138、139和445端口，这几个端口都是与文件共享和打印机共享有关的端口，445端口是局域网文件共享及打印机共享端口(易遭永恒之蓝的病毒入侵), 在这几个端口上经常爆发很严重的漏洞。


<a id="orgb431d7d"></a>

## linux


<a id="org8e09a7e"></a>

### linux的locale设置


<a id="org51a0bef"></a>

### windows和linux之间的编码转换问题

1.  问题：根本原因是由于windows使用GBK编码，而linux使用UTF-8编码，所以需要做相应转换后传输解决乱码问题。实际是如果在windows中有中文名的文件或文件夹，通过XFTP传到linux，名字显示是乱码的，仅仅是可观的那层名字是乱码的，里面的那些以及文件内容如果有中文名，在解压后也会受到影响。

    1.  解决方法：(将乱码的中文名恢复正常)
    
        1.  直接将源文件进行编码转换
        
            yum install convmv
            convmv  &#x2013;nosmart -f gbk -t utf8 -r xxx.zip &#x2013;notest
        
        2.  convmv参数详解：
        
            -f from从什么编码
            -t to改成什么编码
            -r 包含所有子目录
            &#x2013;nosmart 如果已经是utf－8 忽略
            &#x2013;notest 不加表示只列出有什么需要转换的，不做实际转换，所以一定要加
        
        3.  通过文件的解压添加解压为utf-8的参数
        
            若在windows传过去的zip包带中文字符，传到windows中会乱码，通过unzip -O utf-8 xxx.zip -d xxx解决


<a id="orgf6067f0"></a>

# 中间件


<a id="orgd41fd4d"></a>

## jdk


<a id="org0d3ba09"></a>

### 环境变量设置

root用户

    export JDK_HOME=/usr/java/jdk1.8.0_311
    export JAVA_HOME=/usr/java/jdk1.8.0_311
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export PATH=$JAVA_HOME/bin:$PATH

普通用户自行管理

    export JDK_HOME=/home/app/java/jdk1.8.0_311
    export JAVA_HOME=/home/app/java/jdk1.8.0_311
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export PATH=$JAVA_HOME/bin:$PATH

普通用户需共享root的java环境

    chown 755 /usr/java/jdk1.8.0_311


<a id="org175cd1a"></a>

## tomcat


<a id="org91bdabc"></a>

### 端口

应用端口和shutdown端口采用5位数，且多个服务时不产生占用冲突

    <Server port="18005" shutdown="SHUTDOWN">
    
    <!-- ... -->
    
    <Connector port="18080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"/>
    
    </Server>


<a id="org6db98b6"></a>

### 内存参数

使用$TOMCAT/bin/setenv.sh，方便迁移 或 添加至$TOMCAT/bin/catalina.sh首行


<a id="org738f382"></a>

### 日志($TOMCAT/logs)

<div class="org-center">
<p>
catalina.out：核心日志，无论是正确信息还是错误信息，tomcat服务还是运行的应用的信息都会记录到该日志
manager/host-manager：管理台的日志。
localhost<sub>access</sub><sub>log</sub>：访问日志，如配置了nginx代理tomcat，看nginx日志即可。
catalina.xxxx-xx-xx.log：记录的信息与catalina.out重复。
localhost.xxxx-xx-xx.log：应用初始化未处理的异常最后被tomcat捕获而输出的日志。
</p>
</div>

1.  切分脚本(log<sub>cut.sh</sub>)

        #!/bin/bash
        tomcat_home=/bea/apache-tomcat-8.5.86
        d=`date +%Y-%m-%d`
        d7=`date -d'7 day ago' +%Y-%m-%d`
        
        cd $tomcat_home/logs
        
        cp catalina.out catalina.out.${d}
        cat /dev/null > catalina.out 
        rm -rf  catalina.out.${d7}
        rm -rf  localhost_access_log.${d7}.txt
        rm -rf  localhost.${d7}.log
        rm -rf  host-manager.${d7}.log

2.  定时任务脚本(log<sub>cut</sub><sub>cron.sh</sub>)

        59 23 * * * /bin/sh /bea/log_cut.sh

3.  开启定时任务

    crontab log<sub>cut</sub><sub>c</sub> && crontab -l


<a id="orgf28d931"></a>

### 访问控制

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left">中危漏洞 ｜ 中间件 ｜ 中间件使用root用户运行tomcat ｜ 建议中间件不以root用户运行tomcat ｜</td>
</tr>
</tbody>
</table>

    chown -R user:usergp $TOMCAT
    chown -R user:usergp /bwreleaseopt && chmod o+x /bwreleaseopt
    chown -R user:usergp /data/logs && chmod o+x /data && chmod o+x /data/logs


<a id="org3b9f39d"></a>

### 入侵防护

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left">中危漏洞</td>
<td class="org-left">中间件</td>
<td class="org-left">中间件未修改版本文件信息，未能防止版本信息泄露</td>
<td class="org-left">编辑TOMCAT/lib/catalina.jar中的ServerInfo.properties注释server.into server.number并重启tomcat</td>
</tr>


<tr>
<td class="org-left">中危漏洞</td>
<td class="org-left">中间件</td>
<td class="org-left">中间件未自定义错误页面</td>
<td class="org-left">建议中间件自定义400、403、404、500错误文件，防止信息泄露，在web.xml文件中配置jsp文件</td>
</tr>
</tbody>
</table>

编辑$TOMCAT/conf/web.xml，添加配置并html文件：webapps/error/error.html(内容：NOT FOUND)

    <error-page>
      <!--  error-code表示错误类型      -->
      <error-code>500</error-code>
      <!--   location表示出现上面500类型的错误要去跳转到的页面        -->
      <location>/error/error.html</location>
    </error-page>
    
    <!-- error-page 标签配置，服务器出错之后，自动跳转的页面 -->
    <error-page>
      <!--  error-code表示错误类型      -->
      <error-code>404</error-code>
      <!--   location表示出现上面404类型的错误要去跳转到的页面        -->
      <location>/error/error.html</location>
    </error-page>


<a id="org8424434"></a>

## mysql


<a id="org32cd1e5"></a>

### 启动进程


<a id="orgecd2cbd"></a>

### 启动原理

默认的mysql的服务启动程序是mysql.server，mysql.server程序主要是会用到两个程序和一个函数，分别是my<sub>print</sub><sub>defaults</sub>、myslqd<sub>safe和parse</sub><sub>server</sub><sub>arguments</sub>。

my<sub>print</sub><sub>defaults</sub>: 读取my.cnf配置文件，输出参数传递给parse<sub>server</sub><sub>arguments</sub>，该程序只读my.cnf中[mysqld]中的参数。

parse<sub>server</sub><sub>arguments</sub>: 该函数处理my<sub>print</sub><sub>defaults传递过来的参数赋值给</sub>&#x2013;basedir、&#x2013;datadir、&#x2013;pid-file、&#x2013;server-startup-timeout

myslqd<sub>safe</sub>: mysqld<sub>safe程序调用mysqld程序来启动mysql服务</sub>,[mysqld<sub>safe</sub>]会覆盖mysqld部分中的参数

mysqld<sub>multi</sub>: 会读取配置文件中的[mysqld<sub>muti</sub>],[mysqldN]下面的参数，N需要时一个整数，建议用端口号表示，该部分的配置会覆盖[mysqld]部分中的配置

在mysqld进程挂掉的时候，mysqld<sub>safe进程会监测到并重新</sub>


<a id="org3010c54"></a>

### 配置文件

1.  配置文件my.cnf启动加载读取顺序

    读取顺序：/etc/my.cnf > *etc/mysql/my.cnf > /usr/etc/my.cnf > ~*.my.cnf
    
        mysql --verbose --help | grep my.cnf

2.  配置项详解

        [client]
        port = 3306
        socket = /dev/shm/mysql.sock
        [mysqld] # 服务器端配置
        !includedir /etc/mysql/conf.d/xxx.cnf #包含指定目录的cnf配置文件，若与包含文件若有冲突则会覆盖此文件设置
        port = 3306 # 监听端口
        bind-address = 0.0.0.0 # 监听的IP地址
        socket = /dev/shm/mysql.sock # socket通信设置
        basedir = /usr/local/mysql # mysql的程序路径
        datadir = /usr/local/mysql/data # 数据目录
        pid-file = /usr/local/mysql/data/mysql.pid # pid文件路径
        user = mysql
        server-id = 1 # mysql的服务ID
        init-connect = 'SET NAMES utf8mb4'
        character-set-server = utf8mb4
        #skip-name-resolve
        #禁止 MySQL 对外部连接进行 DNS 解析，使用这一选项可以消除 MySQL 进行 DNS 解析的时间。
        #但需要注意，如果开启该选项，则所有远程主机连接授权都要使用 IP 地址方式，否则 MySQL 将无法正常处理连接请求！
        #skip-networking
        #开启该选项可以彻底关闭 MySQL 的 TCP/IP 连接方式，
        #如果 WEB 服务器是以远程连接的方式访问 MySQL 数据库服务器则不要开启该选项！
        #否则将无法正常连接！ 如果所有的进程都是在同一台服务器连接到本地的 mysqld, 这样设置将是增强安全的方法
        back_log = 300
        max_connections = 1000
        max_connect_errors = 6000
        open_files_limit = 65535
        table_open_cache = 128
        max_allowed_packet = 4M
        binlog_cache_size = 1M
        max_heap_table_size = 8M
        tmp_table_size = 16M
        read_buffer_size = 2M
        read_rnd_buffer_size = 8M
        sort_buffer_size = 8M
        join_buffer_size = 8M
        key_buffer_size = 4M # 指定索引的缓冲区大小。InnoDB引擎8M足够
        thread_cache_size = 8
        query_cache_type = 1
        query_cache_size = 8M
        query_cache_limit = 2M
        ft_min_word_len = 4
        log_bin = mysql-bin
        binlog_format = mixed
        expire_logs_days = 30
        log_error = /usr/local/mysql/logs/mysql-error.log
        slow_query_log = 1
        long_query_time = 1
        slow_query_log_file = /usr/local/mysql/logs/mysql-slow.log
        performance_schema = 0
        explicit_defaults_for_timestamp
        #lower_case_table_names = 1
        skip-external-locking
        default_storage_engine = InnoDB
        #default-storage-engine = MyISAM
        innodb_file_per_table = 1
        innodb_open_files = 500
        innodb_buffer_pool_size = 64M
        innodb_write_io_threads = 4
        innodb_read_io_threads = 4
        innodb_thread_concurrency = 0
        innodb_purge_threads = 1
        innodb_flush_log_at_trx_commit = 2
        innodb_log_buffer_size = 2M
        innodb_log_file_size = 32M
        innodb_log_files_in_group = 3
        innodb_max_dirty_pages_pct = 90
        innodb_lock_wait_timeout = 120
        bulk_insert_buffer_size = 8M
        myisam_sort_buffer_size = 8M
        myisam_max_sort_file_size = 10G
        myisam_repair_threads = 1
        interactive_timeout = 28800
        wait_timeout = 28800
        [mysqldump]
        quick
        max_allowed_packet = 16M
        [myisamchk]
        key_buffer_size = 8M
        sort_buffer_size = 8M
        read_buffer = 4M
        write_buffer = 4M

3.  配置文件样例

        [mysqld]
        # 设置3306端口
        port=3306
        # 设置mysql的安装目录
        basedir=/usr/local/mysql
        # 设置mysql数据库的数据的存放目录
        datadir=/usr/local/mysql/mysqldb
        # 允许最大连接数
        max_connections=10000
        # 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
        max_connect_errors=10
        # 服务端使用的字符集默认为UTF8
        character-set-server=utf8
        # 创建新表时将使用的默认存储引擎
        default-storage-engine=INNODB
        # 默认使用“mysql_native_password”插件认证
        default_authentication_plugin=mysql_native_password
        [mysql]
        # 设置mysql客户端默认字符集
        default-character-set=utf8
        [client]
        # 设置mysql客户端连接服务端时默认使用的端口
        port=3306
        default-character-set=utf8


<a id="org57c8e6e"></a>

### TODO 参数详解

1.  错误日志(log<sub>error参数</sub>)

2.  错误日志（Error Log）是 MySQL 中最常用的一种日志，主要记录 MySQL 服务器启动和停止过程中的信息、服务器在运行过程中发生的故障和异常情况等。

    作为初学者，要学会利用错误日志来定位问题。下面介绍如何操作查看错误日志。
    启动和设置错误日志
    在 MySQL 数据库中，默认开启错误日志功能。一般情况下，错误日志存储在 MySQL 数据库的数据文件夹下，通常名称为 hostname.err。其中，hostname 表示 MySQL 服务器的主机名。
    在 MySQL 配置文件中，错误日志所记录的信息可以通过 log-error 和 log-warnings 来定义，其中，log-err 定义是否启用错误日志功能和错误日志的存储位置，log-warnings 定义是否将警告信息也记录到错误日志中。
    
    将 log<sub>error</sub> 选项加入到 MySQL 配置文件的 [mysqld] 组中，形式如下：
    
        [mysqld]
        log-error=dir/{filename}
    
    查看：
     mysql> show variables like 'log<sub>error</sub>';
    
    <!-- This HTML table template is generated by emacs 29.1 -->
    <table border="1">
      <tr>
        <td align="left" valign="top">
          &nbsp;Variable_name&nbsp;
        </td>
        <td align="left" valign="top">
          &nbsp;Value&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        </td>
      </tr>
      <tr>
        <td align="left" valign="top">
          &nbsp;log_error&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        </td>
        <td align="left" valign="top">
          &nbsp;./localhost.err&nbsp;
        </td>
      </tr>
    </table>
    
    其中，dir 参数指定错误日志的存储路径；filename 参数指定错误日志的文件名；省略参数时文件名默认为主机名，存放在 Data 目录中。
    
    重启 MySQL 服务后，参数开始生效，可以在指定路径下看到 filename.err 的文件，如果没有指定 filename，那么错误日志将直接默认为 hostname.err。
    注意：错误日志中记录的并非全是错误信息，例如 MySQL 如何启动 InnoDB 的表空间文件、如何初始化自己的存储引擎等，这些也记录在错误日志文件中。

3.  查看错误日志

    错误日志中记录着开启和关闭 MySQL 服务的时间，以及服务运行过程中出现哪些异常等信息。如果 MySQL 服务出现异常，可以到错误日志中查找原因。
    在 MySQL 中，通过 SHOW 命令可以查看错误日志文件所在的目录及文件名信息。
    mysql> SHOW VARIABLES LIKE 'log<sub>error</sub>';
    
    <!-- This HTML table template is generated by emacs 29.1 -->
    <table border="1">
      <tr>
        <td align="left" valign="top">
          &nbsp;Variable_name&nbsp;
        </td>
        <td align="left" valign="top">
          &nbsp;Value&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        </td>
      </tr>
      <tr>
        <td align="left" valign="top">
          &nbsp;log_error&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        </td>
        <td align="left" valign="top">
          &nbsp;C:\ProgramData\MySQL\MySQL&nbsp;Server&nbsp;5.7\Data\LAPTOP-UHQ6V8KP.err&nbsp;
        </td>
      </tr>
    </table>
    
    1 row in set, 1 warning (0.04 sec)
    错误日志以文本文件的形式存储，直接使用普通文本工具就可以查看。这里通过记事本打开，从上面可以知道错误日志的文件名。该文件在默认的数据路径“C:\ProgramData\MySQL\MySQL Server 5.7\Data\\”下，打开 LAPTOP-UHQ6V8KP.err 文件，部分内容如下：
    190906 22:06:45 InnoDB: Completed initialization of buffer pool
    190906 22:06:45 InnoDB: highest supported file format is Barracuda.
    190906 22:06:45  InnoDB: Waiting for the background threads to start
    190906 22:06:46 InnoDB: 5.7.29 started; log sequence number 1605345
    190906 22:06:47 [Note] Server hostname (bind-address): '0.0.0.0'; port: 3306
    190906 22:06:47 [Note]   - '0.0.0.0' resolves to '0.0.0.0';
    190906 22:06:47 [Note] Server socket created on IP: '0.0.0.0'.
    190906 22:06:47 [Note] Event Scheduler: Loaded 0 events
    190906 22:06:47 [Note] /usr/sbin/mysqld: ready for connections.
    Version: '5.7.29-log'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server (GPL)
    
    以上是错误日志文件的一部分，主要记载了系统的一些运行错误。

4.  删除错误日志

    在 MySQL 中，可以使用 mysqladmin 命令来开启新的错误日志，以保证 MySQL 服务器上的硬盘空间。mysqladmin 命令的语法如下：
    mysqladmin -uroot -p flush-logs
    执行该命令后，MySQL 服务器首先会自动创建一个新的错误日志，然后将旧的错误日志更名为 filename.err-old。
    MySQL 服务器发生异常时，管理员可以在错误日志中找到发生异常的时间、原因，然后根据这些信息来解决异常。对于很久之前的错误日志，查看的可能性不大，可以直接将这些错误日志删除。

5.  log<sub>bin参数</sub>


<a id="orga0eb3b7"></a>

### mariadb简明安装

1.  下载地址: <https://downloads.mariadb.org/>

2.  卸载系统自带的mysql

3.  解压包到/usr/local/mariadb目录

4.  创建user及group

        groupadd mysql
        useradd -g mysql mysql
        chown -R mysql:mysql /usr/local/mariadb/

5.  添加配置文件(/etc/my.cnf)

        [mysql]
        #设置mysql客户端默认字符集
        default-character-set=utf8
        
        [mysqld]
        skip-name-resolve
        #设置3306端口
        port = 13360 
        #设置mysql的安装目录
        basedir=/usr/local/mariadb
        #设置mysql数据库的数据的存放目录
        datadir=/usr/local/mariadb/data
        #允许最大连接数
        max_connections=2000
        #服务端使用的字符集默认为8比特编码的latin1字符集
        character-set-server=utf8
        #创建新表时将使用的默认存储引擎
        default-storage-engine=INNODB 
        lower_case_table_names=1
        max_allowed_packet=16M
        innodb_buffer_pool_size=1024M

6.  安装及初始化

        cd /usr/local/mariadb
        ./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mariadb --datadir=/usr/local/mariadb/data
    
    执行成功后如下：

7.  添加服务并设置自启

        cp ./support-files/mysql.server /etc/init.d/mysqld && ll /etc/init.d/mysqld
        chkconfig --add mysqld
        chkconfig --list mysqld
        service mysqld restart

8.  环境变量配置

        #set mariadb environment
        export MARIADB_HOME=/usr/local/mariadb
        export PATH=$PATH:${MARIADB_HOME}/bin
    
        source /etc/profile

9.  配置mariadb

        # 需要在mariadb目录下执行，不然会有找不到：my_print_defaults这个文件的问题
        cd /usr/local/mariadb
        
        ./bin/mariadb-secure-installation --basedir=/usr/local/mariadb

10. 授权远程连接

        mysql -u root -p
    
        grant all privileges on *.* to root@"%" identified by "Booway128_r" with grant option;
        flush privileges;

11. 设置本地需要正确密码登录

        alter user 'root'@'localhost' IDENTIFIED BY 'Booway128_r';  

12. 以utf-8编码创建数据库

        CREATE DATABASE `test` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;


<a id="orgb1557a3"></a>

### 数据库定时备份

1.  编写备份脚本：mariadb<sub>backup.sh</sub>

        #!/bin/bash
        
        #保存备份个数，备份7天数据
        number=15
        #备份保存路径
        backup_dir=/root/mysqlbackup
        #日期
        dd=`date +%Y-%m-%d-%H-%M-%S`
        #备份工具
        tool=mysqldump
        #用户名
        username=root
        #密码
        password=Booway128_r
        #将要备份的数据库
        database_name=gzznsjps_prod
        
        
        #如果文件夹不存在则创建
        if [ ! -d $backup_dir ];
        then     
            mkdir -p $backup_dir;
        fi
        
        #简单写法 mysqldump -u root -p123456 users > /root/mysqlbackup/users-$filename.sql
        $tool -u $username -p$password $database_name > $backup_dir/$database_name-$dd.sql
        
        #写创建备份日志
        echo "create $backup_dir/$database_name-$dd.dupm" >> $backup_dir/log.txt
        
        #找出需要删除的备份
        delfile=`ls -l -crt $backup_dir/*.sql | awk '{print $9 }' | head -1`
        
        #判断现在的备份数量是否大于$number
        count=`ls -l -crt $backup_dir/*.sql | awk '{print $9 }' | wc -l`
        
        if [ $count -gt $number ]
        then
          #删除最早生成的备份，只保留number数量的备份
          rm $delfile
          #写删除文件日志
          echo "delete $delfile" >> $backup_dir/log.txt
        fi  

2.  添加定时任务：mariadb<sub>backup</sub><sub>cron.sh</sub>

        55 23 * * * source /etc/profile && xxx/mariadb_backup.sh

3.  开启定时任务：crontab -l mariadb<sub>backup</sub><sub>cron.sh</sub>


<a id="org8355d38"></a>

### Mairadb开启SSL登录并配置证书


<a id="org7e71fe2"></a>

### 生成服务端和客户端的证书


<a id="orgb20f99d"></a>

### 将证书路径配置到配置文件并重启服务


<a id="org0185995"></a>

### 客户端使用证书连接或


<a id="org33a8162"></a>

### sql查询练习题

grant all privileges on databaes.\* to 'user'@'localhost';

192.168.0.93 3306   shcool
root/Booway128<sub>r</sub>

\#Student表：(sno,sname,sage,sdept,ssex)
\#Cource表：(cno,cname,tid)
\#Score表：(sno,cno,score)
\#Teacher表：(tid,tname)

&#x2013; 1.查询“01”课程比“02”课程成绩高的所有学生的学号

&#x2013; 2.查询平均成绩大于60分的同学的学号和平均成绩
SELECT sno AVG(socre) FROM score GROUP BY sno HAVING AVG(score) > 60;

&#x2013; 3.查询所有同学的学号、姓名、选课数、总成绩
select st.sno,st.sname,count(sc.sno),sum(sc.score) from Student as st left join Score as sc on st.sno = sc.sno group by sno,sname;

&#x2013; 4.查询姓“李”的老师的个数
SELECT COUNT(tid) FROM Teacher WHERE tname LIKE '李%';

&#x2013; 5.查询没学过“张三”老师课的同学的学号、姓名

&#x2013; 6.查询学过“张三”老师所教的课的同学的学号、姓名；

&#x2013; 7.查询学过编号“01”并且也学过编号“02”课程的同学的学号、姓名；

&#x2013; 8.查询学过01但是没有学过02的同学的信息

&#x2013; 9.查询所有课程成绩小于60分的同学的学号、姓名；

&#x2013; 11.查询至少有一门课与学号为“01”的同学所学相同的同学的学号和姓名

&#x2013; 12.查询和"01"号的同学学习的课程完全相同的其他同学的学号和姓名

&#x2013; 14.查询没学过"张三"老师讲授的任一门课程的学生姓名

&#x2013; 15.查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

&#x2013; 16.检索"01"课程分数小于60，按分数降序排列的学生信息

&#x2013; 17.按平均成绩从高到低显示所有学生的平均成绩

&#x2013; 18.查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率

&#x2013; 19.按各科平均成绩从低到高和及格率的百分数从高到低顺序

&#x2013; 20.查询学生的总成绩并进行排名

&#x2013; 21.查询不同老师所教不同课程平均分从高到低显示

&#x2013; 22.查询所有课程的成绩第2名到第3名的学生信息及该课程成绩

&#x2013; 23.统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比

&#x2013; 24.查询学生平均成绩及其名次

&#x2013; 25.查询各科成绩前三名的记录

&#x2013; 26.查询每门课程被选修的学生数

&#x2013; 27.查询出只选修了一门课程的全部学生的学号和姓名

&#x2013; 28.查询男生、女生人数

&#x2013; 29.查询名字中含有"风"字的学生信息

&#x2013; 30.查询同名同性学生名单，并统计同名人数

&#x2013; 31.查询1990年出生的学生名单(注：Student表中Sage列的类型是datetime)

&#x2013; 32.查询每门课程的平均成绩，结果按平均成绩升序排列，平均成绩相同时，按课程号降序排列

&#x2013; 33.查询不及格的课程，并按课程号从大到小排列

&#x2013; 34.查询课程编号为"01"且课程成绩在60分以上的学生的学号和姓名

&#x2013; 35.查询所有学生的课程及分数情况

&#x2013; 36.查询任何一门课程成绩在70分以上的姓名、课程名称和分数

&#x2013; 37.查询课程名称为"数学"，且分数低于60的学生姓名和分数

&#x2013; 38.查询课程编号为03且课程成绩在80分以上的学生的学号和姓名

&#x2013; 39.求每门课程的学生人数

&#x2013; 40.查询选修“张三”老师所授课程的学生中，成绩最高的学生姓名及其成绩

&#x2013; 41.查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

&#x2013; 42.查询每门功课成绩最好的前两名

&#x2013; 43.统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

&#x2013; 44.检索至少选修两门课程的学生学号

&#x2013; 45.查询选修了全部课程的学生信息

&#x2013; 46.查询各学生的年龄

&#x2013;47.查询本周过生日的学生

&#x2013; 48.查询下周过生日的学生

&#x2013; 49.查询本月过生日的学生

&#x2013; 50.查询下月过生日的学生


<a id="org3bd0ce3"></a>

## nginx


<a id="org6dbeb8a"></a>

### nginx的访问部分资源：503 Service Temporarily Unavailable：

1.  问题

2.  解决方式

    <https://blog.csdn.net/fly_captain/article/details/84846634>


<a id="orgc3ac75d"></a>

## docker


<a id="orgd89ab08"></a>

### 镜像分片

1.  需求：交付项目需要压缩镜像大小不在乎导入及压缩的耗时。

2.  比较

        # 普通导出
        docker save 450379344707 > /tmp/1.tar.gz
        # 使用gzip导出
        docker save <myimage>:<tag> | gzip > <myimage>_<tag>.tar.gz
        docker save 450379344707 | gzip >  /tmp/2.tar.gz
        # 结果对此大小差异明显

3.  操作

    1 导出镜像并压缩：docker save <myimage>:<tag> | gzip > <myimage>\_<tag>.tar.gz
    2 把文件打成zip压缩包: zip <myimage>:<tag>.zip <myimage>:<tag>.tar.gz
    3 把文件用zip分片: zip -s 490m <myimage>:<tag>.zip &#x2013;out <myimage>:<tag>
    4 把分片文件合成zip文件: cat <myimage>:<tag>.z\* &#x2013;out <myimage>:<tag>.zip
    5 修复并解压得到tar.gz压缩文件: zip -F <myimage>:<tag>.zip && unzip <myimage>:<tag>.zip
    6 对比文件的md5: md5sum objec1t.tar.gz object2.tar.gz
    7 解压并导入镜像：gunzip -c <myimage>\_<tag>.tar.gz | docker load
    8 使md5sum工具查看是否被更改内容

4.  隐患：

    多个文件在分片进行网络传输时存在被损坏的风险，使用md5sum进行检查
    md5sum XXX-file


<a id="org99e1028"></a>

## nfs


<a id="org48ab5a2"></a>

### 简介


<a id="org136ec42"></a>

### 工作流程


<a id="org034f0c6"></a>

### 依赖下载

离线下载nfs-utils和rpcbind的rpm安装包：
<http://rpmfind.net/linux/rpm2html/search.php?query=nfs-utils>
<http://www.rpmfind.net/linux/rpm2html/search.php?query=rpcbind%28x86-64%29> 


<a id="org4af5945"></a>

### NFS配置

1.配置文件设置（/24去掉给，这个是不需要的东西）
*bwdata/upload* 10.150.141.93(rw,sync,all<sub>squash,fsid</sub>=0) 10.150.141.94(rw,sync,all<sub>squash,fsid</sub>=0)
*home/gzdwpg* 10.150.141.93/24 (rw,sync) 10.150.141.94/24 (rw,sync)

2.平滑重启并检查挂载情况
exportfs -arv

showmount -e 10.150.141.95

验证系统更新后是否存在漏洞
1.opc访问网络安全运营服务系统: <https://10.174.241.100>
2.定位至“任务管理-扫描任务管理”页面针对需扫描的系统新建一条任务扫描一下看看，不能扫描出来的话系统会自动将漏洞转移到已整改,若系统漏洞已整改则无问题，若任然出现在未整改则还是有问题。
3.系统数据验:在服务器上的/bwdata/upload/ 和 *home/gzdwpg* 新建文本文档，看下数据是否正常显示。


<a id="orga5d8ea4"></a>

# 实用命令


<a id="orgad91c99"></a>

## md5sum


<a id="orgffb9486"></a>

### 简介


<a id="orgb987a5b"></a>

### 命令选项


<a id="orge7d562a"></a>

### 生成校检码

单个文件

    md5sum aaa.txt > checksum/aaa.md5

多文件

    md5sum aaa.txt bbb.log aaa_pkg.tgz > checksum/all.md5
    md5sum aaa* > checksum/aaa_all.md5


<a id="orgc173d84"></a>

### 校检

在文件所在目录执行

    md5sum -c ./checksum/aaa.md5
    md5sum -c ./checksum/all.md5

非文件所在目录执行

若校检文件缺失


<a id="orga3c1274"></a>

### 总结

特殊说明
1）md5sum是校验文件内容，与文件名是否相同无关
2）md5sum是逐位校验，所以文件越大，校验时间越长

md5校验，可能极小概率出现不同的文件生成相同的校验和，比md5更安全的校验算法还有SHA\*系列，如sha1sum/sha224sum/sha256sum/sha384sum/sha512sum等等，
基本用法与md5sum命令类似，详情可通过man sha1sum查询。


<a id="org9e20295"></a>

# 其他经验


<a id="org3fbc8d1"></a>

## 编辑yml配置文件时，记得不要用打tab，而是用空格键，否则会因为格式问题产生配置格式读取错误，要确保信息正确，格式正确


<a id="org4357f87"></a>

## 博微新配置文件的加解密（需要安装JDK和weakPassword-0.0.1.jar）


<a id="orgad282a2"></a>

### 对明文进行加密(执行会得到密文内容)

1.对于指定文件加密  java -jar weakPassword-0.0.1.jar -f E:\dcode5\dcode\bwopt\booway\dcode\bootstrap.yml
2.对于指定内容字符串加密  java -jar weakPassword-0.0.1.jar -s Booway@1


<a id="orgcd5821b"></a>

### 添加"-d"选项对密文解密(生成解密内容)，添加">"指定明文输出到的文件(不加打印到终端)

1.对密文字符串解密  java -jar weakPassword-0.0.1.jar -s CA572FEF1A15F348E4A51A6567457A78>ok.txt -d
2.对密文文件解密   java -jar weakPassword-0.0.1.jar -f E:\dcode5\dcode\bwopt\booway\dcode\test.yml>ok.txt -d
3.对于java -jar weakPassword-0.0.1.jar -f ok.txt > ok.txt -d报错，不能解密回同一个文件


<a id="org2a0aa9c"></a>

## 文件解压缩的规范注意


<a id="orgbd6e5df"></a>

### 问题：在centos解压文件： tar -xvf mysql-5.7.27-aarch64.tar.gz -C *usr/local*

  报错：gzip: stdin: not in gzip format
　　    tar: Child returned status 1
　　    tar: Error is not recoverable: exiting now


<a id="orga24d56a"></a>

### 解决办法：

tar包压缩的时候用cvf参数，解压的时候用xvf参数
或压缩的时候用czvf参数，解压的时候用xzvf参数
bz 包遇到了，就把z参数换成相应j参数


<a id="orgd9d0b4a"></a>

## Navicat运行sql文件时需要双击数据库将信息项给展开，否则“运行sql文件”是灰色的。


<a id="orgacfddf9"></a>

## Linux更换jar包calss文件：


<a id="org30c0666"></a>

### 要替换的文件在jar包的二级及以下目录下，则需要以下步骤(比如说要将：gzdwpg-biz-1.0.0.jar包中的ProjectBatchCreateServiceImpl.class替换新的class文件)：

1.  使用jar -tvf jar名称 | grep 目标文件名 查询出目标文件在war包中的目录

        jar -tvf gzdwpg-biz-1.0.0.jar | grep ProjectBatchCreateServiceImpl.class

2.  使用jar -xvf jar名称 目标文件名(copy上面查出的全路径) 将目标class文件及所在war包中的目录解压到当前路径

        jar -xvf gzdwpg-biz-1.0.0.jar com/booway/gzdwpg/project/service/impl/ProjectBatchCreateServiceImpl.class

3.  修改目标文件的内容，或者将要新的目标文件替换掉提取出来的目标文件

4.  使用jar -uvf jar名称 目标文件名(和步骤(2)中的目标文件名相同)将新目标class文件替换到jar包中

        jar -uvf gzdwpg-biz-1.0.0.jar com/booway/gzdwpg/project/service/impl/ProjectBatchCreateServiceImpl.class


<a id="org1841a93"></a>

### jar命令参数学习：<https://www.cnblogs.com/liyanbin/p/6088458.html>


<a id="org3c551b6"></a>

## DOP文件预览服务：


<a id="org19b0d4d"></a>

## 字体问题：

    mkdir /usr/share/fonts/windows/    //创建windows的字体目录
    cp /bea/fonts/*  /usr/share/fonts/windows/*
    cd /usr/share/fonts/ && chmod 755 windows && cd /usr/share/fonts/windows && chmod 644 * 
    
    mkfontscale
    mkfontdir
    
    # 然后在系统上刷新字体缓存，不需要重启Linux，命令来自于：yum install -y fontconfig
    fc-cache -f -v
    
    # 重启dop和应用系统

