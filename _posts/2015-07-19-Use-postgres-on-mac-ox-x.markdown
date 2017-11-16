---
layout: post
title: Mac下Postgres得安装和使用
category: Document
tags: postgres
year: 2015
month: 07
day: 19
published: true
summary: 在Mac下安装和使用Postgres
image: pirates.svg
comment: true
---

## 安装与配置

在 mac 下，可以利用 homebrew 直接安装 PostgreSQL：

```
brew install postgresql -v
```

稍等片刻，PostgreSQL 就安装完成。接下来就是初始数据库，在终端执行一下命令，初始配置 PostgreSQL：

```
initdb /usr/local/var/postgres -E utf8
```

上面指定 "/usr/local/var/postgres" 为 PostgreSQL 的配置数据存放目录，并且设置数据库数据编码是 utf8，更多配置信息可以 "initdb --help" 查看。

设成开机启动 PostgreSQL：

```
ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
```

启动 PostgreSQL：

```
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
```

关闭 PostgreSQL：

```
pg_ctl -D /usr/local/var/postgres stop -s -m fast
```

## 创建一个 PostgreSQL 用户

```
createuser username -P
#Enter password for new role:
#Enter it again:
```

上面的 username 是用户名，回车输入 2 次用户密码后即用户创建完成。更多用户创建信息可以 "createuser --help" 查看。

## 创建数据库

```
createdb dbname -O username -E UTF8 -e
```

上面创建了一个名为 dbname 的数据库，并指定 username 为改数据库的拥有者（owner），数据库的编码（encoding）是 UTF8，参数 "-e" 是指把数据库执行操作的命令显示出来。

更多数据库创建信息可以 "createdb --help" 查看。

## 连接数据库

```
psql -U username -d dbname -h 127.0.0.1
```

## PostgreSQL 数据库操作

显示已创建的数据库：

```
\l
```

在不连接进 PostgreSQL 数据库的情况下，也可以在终端上查看显示已创建的列表：

```
psql -l
```

连接数据库

```
\c dbname
```

显示数据库表

```
\d
```

创建一个名为 test 的表

```
CREATE TABLE test(id int, text VARCHAR(50));
```

插入一条记录

```
INSERT INTO test(id, text) VALUES(1, 'sdfsfsfsdfsdfdf');
```

查询记录

```
SELECT * FROM test WHERE id = 1;
```

更新记录

```
UPDATE test SET text = 'aaaaaaaaaaaaa' WHERE id = 1;
```

删除指定的记录

```
DELETE FROM test WHERE id = 1;
```

删除表

```
DROP TABLE test;
```

删除数据库

```
DROP DATABASE dbname;
```

或者利用 dropdb 指令，在终端上删除数据库

```
dropdb -U user dbname
```

下面是自用的 PostgreSQL 的 php 操作类：

```php
<?php
 
define("HOST", "127.0.0.1");
define("PORT", 5432);
define("DBNAME", "dbname");
define("USER", "user");
define("PASSWORD", "password");
 
class Ext_Pgsql {
     
    //单例
    private static $instance = null;
 
    private $conn = null;
 
    private function __construct()
    {
        $this->conn = pg_connect("host=" . HOST . " port=" . PORT . " dbname=" . DBNAME . " user=" . USER . " password=" . PASSWORD) or die('Connect Failed : '. pg_last_error());
    }
 
    public function __destruct()
    {
        @pg_close($this->conn);
    }
 
    /**
     * 单例模式
     * @param $name
     */
    public static function getInstance()
    {
        if ( ! self::$instance )
        {
            self::$instance = new self();
        }
        return self::$instance;
    }
 
    /**
     * 获取记录
     */
    public function fetchRow($sql)
    {
        $ret = array();
        $rs = pg_query($this->conn, $sql);
        $ret = pg_fetch_all($rs);
        if ( ! is_array($ret) )
        {
            return array();
        }
        return $ret;
    }
 
    /**
     * 执行指令
     * @param string $sql
     */
    public function query($sql)
    {
        $result = pg_query($this->conn, $sql);
        if ( ! $result )
            die("SQL : {$sql} " . pg_last_error());
    }
 
    /**
     * 获取一条记录
     */
    public function fetchOne($sql)
    {
        $ret = array();
        $rs = pg_query($this->conn, $sql);
        $ret = pg_fetch_all($rs);
        if ( ! is_array($ret) )
        {
            return array();
        }
        return current($ret);
    }
     
}
 
?>
```

## 一些问题

### PostgreSQL 9.2 版本升级到 9.3.1 版本后的数据兼容问题

连接 PostgreSQL 时报以下错误：

```
psql: could not connect to server: Connection refused
Is the server running on host "127.0.0.1" and accepting
TCP/IP connections on port 5432?
```

打开 PostgreSQL 的服务日志发现是 PostgreSQL 9.2 版本升级到 9.3.1 版本后的数据兼容问题：

```
tail -f /usr/local/var/postgres/server.log
FATAL:  database files are incompatible with server
DETAIL:  The data directory was initialized by PostgreSQL version 9.2, which is not compatible with this version 9.3.1.
```

对于版本的数据升级问题，PostgreSQL 提供了 pg_upgrade 来做版本后的数据迁移，用法如下：

```
pg_upgrade -b 旧版本的bin目录 -B 新版本的bin目录 -d 旧版本的数据目录 -D 新版本的数据目录 [其他选项...]
```

数据迁移前，记得先关闭 PostgreSQL 的 postmaster 服务，不然会报以下错误：

```
There seems to be a postmaster servicing the new cluster.
Please shutdown that postmaster and try again.
Failure, exiting
```

利用 pg_ctl 关闭 postmaster：

```
pg_ctl -D /usr/local/var/postgres stop
```

Mac 下也可以这样关闭：

```
launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
```

首先备份就版本的数据（默认是在 /usr/local/var/postgres 目录）：

```
mv /usr/local/var/postgres /usr/local/var/postgres.old
```

利用 initdb 命令再初始一个数据库文件：

```
initdb /usr/local/var/postgres -E utf8 --locale=zh_CN.UTF-8
```

_NOTE：_记得加 "--locale=zh_CN.UTF-8" 选项，不然会报以下错误：

```
lc_collate cluster values do not match:  old "zh_CN.UTF-8", new "en_US.UTF-8"
```

最后运行 pg_upgrade 进行数据迁移：

```
pg_upgrade -b /usr/local/Cellar/postgresql/9.2.4/bin/ -B /usr/local/Cellar/postgresql/9.3.1/bin/ -d /usr/local/var/postgres.old -D /usr/local/var/postgres -v
```

> 参见：http://dhq.me/mac-postgresql-install-usage
