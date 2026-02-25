1、MySQL连接池

MySQL连接池是一个管理数据库连接的机制，它可以在应用程序和数据库之间创建和维护数据库连接。当应用程序需要访问数据库时，它可以从连接池中请求一个可用的数据库连接，执行数据库操作，然后将连接放回池中以供其他应用程序使用。

使用连接池的好处是它可以降低数据库的负载，提高应用程序的性能。当每个应用程序在需要时都创建一个数据库连接时，这会导致数据库服务器处理大量的连接请求。但是，使用连接池时，应用程序可以重复使用现有的连接，从而减少了连接的创建和销毁操作，提高了数据库的性能。

1.1、go的gorm库和MySQL连接池

```sql
//GORM 库和 MySQL 连接池
package main
import (
    "fmt"
    "log"
    "time"
    "github.com/go-sql-driver/mysql"
    "github.com/jinzhu/gorm"
)
type User struct {
    ID    uint   `gorm:"primary_key"`
    Name  string `gorm:"type:varchar(50);not null"`
    Email string `gorm:"type:varchar(100);unique_index"`
}
var myDb *gorm.DB
func main() {
    // 创建 MySQL 连接池
    config := mysql.Config{
        User:   "user",
        Passwd: "password",
        Net:    "tcp",
        Addr:   "localhost:3306",
        DBName: "database_name",
    }
   myDb, err := gorm.Open("mysql", config.FormatDSN())
    if err != nil {
        log.Fatal(err)
    }
    defer my myDb.Close()
    // 设置连接池参数
    myDb.DB().SetMaxIdleConns(10)
    myDb.DB().SetMaxOpenConns(100)
    myDb.DB().SetConnMaxLifetime(5 * time.Minute)
    // 自动迁移表结构
    myDb.AutoMigrate(&User{})
    // 创建用户
    user := User{Name: "Alice", Email: "alice@example.com"}
    myDb.Create(&user)
    fmt.Printf("Created user with ID: %d\n", user.ID)
    // 查询用户
    var result User
    myDb.First(&result, "email = ?", "alice@example.com")
    fmt.Printf("Found user with ID: %d\n", result.ID)
    // 更新用户
    myDb.Model(&result).Update("Name", "Alice Smith")
    fmt.Println("Updated user name to: Alice Smith")
    // 删除用户
    myDb.Delete(&result)
    fmt.Println("Deleted user")
}
```

1.2、php的mysqli操作mysql连接池

```
<?php
// 创建 MySQL 连接池
$host = 'localhost';
$user = 'user';
$password = 'password';
$dbname = 'database_name';
$pool = new mysqli_pool(10, 100, 5 * 60 * 1000);
$pool->addResource(new mysqli($host, $user, $password, $dbname));
// 获取连接
$mysqli = $pool->getResource();
if ($mysqli->connect_error) {
    die('Connect Error (' . $mysqli->connect_errno . ') ' . $mysqli->connect_error);
}
// 执行查询
$sql = 'SELECT * FROM users WHERE email = ?';
$stmt = $mysqli->prepare($sql);
$email = 'alice@example.com';
$stmt->bind_param('s', $email);
$stmt->execute();
$result = $stmt->get_result();
// 处理结果
while ($row = $result->fetch_assoc()) {
    printf("ID: %d, Name: %s, Email: %s\n", $row['id'], $row['name'], $row['email']);
}
// 释放连接
$pool->releaseResource($mysqli);
```

```
<?php
// 创建 MySQL 连接池
$host = 'localhost';
$user = 'user';
$password = 'password';
$dbname = 'database_name';
$pool = new PDOConnectionPool(10, 100, 5 * 60 * 1000);
$pool->addResource(new PDO("mysql:host=$host;dbname=$dbname", $user, $password));
// 创建 swoole_server
$server = new Swoole\Server('127.0.0.1', 9501, SWOOLE_BASE);
// 监听连接事件
$server->on('connect', function ($server, $fd) {
    echo "Client $fd connected.\n";
});
// 监听数据接收事件
$server->on('receive', function ($server, $fd, $reactor_id, $data) use ($pool) {
    $data = trim($data);
    if ($data === 'get') {
        // 从连接池获取连接
        $pdo = $pool->getResource();
        if (!$pdo) {
            $server->send($fd, 'Failed to get connection from pool');
        } else {
            // 发送连接的 ID 到客户端
            $connectionId = $pool->getResourceId($pdo);
            $server->send($fd, "Got a connection from pool: $connectionId");
        }
    } elseif (strpos($data, 'release') === 0) {
        // 释放连接到连接池
        $connectionId = substr($data, strlen('release '));
        $pdo = $pool->getResourceById($connectionId);
        if (!$pdo) {
            $server->send($fd, "Failed to find connection $connectionId in pool");
        } else {
            $pool->releaseResource($pdo);
            $server->send($fd, "Released the connection $connectionId to pool");
        }
    } else {
        $server->send($fd, 'Invalid command');
    }
});
// 监听关闭连接事件
$server->on('close', function ($server, $fd) {
    echo "Client $fd closed.\n";
});
// 启动服务器
$server->start();
```