---
title: 使用dblib连接sybase
date: 2017-02-14 15:17:14
tags: php
---
1. 安装freetds
```bash
yum install freetds freetds-dev
```

2. 安装pdo_dblib扩展
```bash
pecl install pdo_dblib
php --ri pdo_dblib # 验证扩展是否安装
```

3. pdo连接
```php
$dsn = 'dblib:appname=PHP freetds;host=localhost:12345;dbname=abc';
$user = 'username';
$passwd = 'password';
try {
    $dbh = new \PDO($dsn, $user, $passwd);
} catch(\PDOException $e) {
    echo $e ->getMessage();
}
```

*yum源没有freetds  
```bash
wget http://www6.atomicorp.com/channels/atomic/centos/7/x86_64/RPMS/atomic-release*rpm
rpm -Uvh atomic-release*rpm
yum install freetds
```
