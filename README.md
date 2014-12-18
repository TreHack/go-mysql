# go-mysql

A pure go library to handle MySQL network protocol and replication.

## Replication

Replication package handles MySQL replication protocol like [python-mysql-replication](https://github.com/noplay/python-mysql-replication).

You can use it acting like a MySQL slave to sync binlog from master then do somethings, like updating cache, etc...

### Example

```go
import (
    "github.com/siddontang/go-mysql/replication"
    "os"
)
// Create a binlog syncer with a unique server id, the server id must be different from other MySQL's. 
syncer := replication.NewBinlogSyncer(100)

// Register slave, the MySQL master is at 127.0.0.1:3306, with user root and an empty password
syncer.RegisterSlave("127.0.0.1", 3306, "root", "")

// Start sync with sepcified binlog file and position
streamer, _ := syncer.StartSync(binlogFile, binlogPos)

for {
    ev, _ := streamer.GetEvent()
    // Dump event
    ev.Dump(os.Stdout)
}
```

The output looks:

```
=== RotateEvent ===
Date: 1970-01-01 08:00:00
Log position: 0
Event size: 43
Position: 12188
Next log name: mysql.000001

=== FormatDescriptionEvent ===
Date: 2014-12-17 16:39:52
Log position: 0
Event size: 116
Version: 4
Server version: 5.6.19-log
Create date: 1970-01-01 08:00:00
```

## Client

Client package supports a simple MySQL connection driver which you can use it to communicate with MySQL server. 

### Example

```go
import (
    "github.com/siddontang/go-mysql/client"
)

// Connect MySQL at 127.0.0.1:3306, with user root, an empty passowrd and database test
conn, _ := client.Connect("127.0.0.1:3306", "root", "", "test")

conn.Ping()

// Insert
r, _ := conn.Execute(`insert into table (id, name) values (1, "abc")`)

// Get last insert id
println(r.InsertId)

// Select
r, _ := conn.Execute(`select id, name from table where id = 1`)

// Handle resultset
v, _ := r.GetInt(0, 0)
v, _ = r.GetIntByName(0, "id") 
```

## Server

Server package supplies a framework to implement a simple MySQL server which can handle the packets from the MySQL client. 
You can use it to build your own MySQL proxy. 

### Example

```go
import (
    "github.com/siddontang/go-mysql/server"
    "net"
)

l, _ := net.Listen("127.0.0.1:4000")

c, _ := l.Accept()

// Create a connection with user root and an empty passowrd
// We only an empty handler to handle command too
conn, _ := server.NewConn(c, "root", "", server.EmptyHandler{})

for {
    conn.HandleCommand()
}
```

Another shell

```
mysql -h127.0.0.1 -P4000 -uroot -p 
//Becuase empty handler does nothing, so here the MySQL client can only connect the proxy server. :-) 
```

## Feedback

go-mysql is still in development, your feedback is very welcome. 


Gmail: siddontang@gmail.com