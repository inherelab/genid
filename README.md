# GenId

Global ID generator base on MySQL

> Inspired the projects [flike/idgo](https://github.com/flike/idgo) 

`genId` is a sequential id generator which can generate batch ids through MySQL transcation way. Its features as follows:

- The ID generated by genId is by order.
- generate batch ids through MySQL transcation way, dose not increase the load of MySQL.
- When process crashed, admin can restart server, does not worry about generating repetitive ids.
- GenId talks with clients using redis protocol. developer can connect the server using redis sdk.
- The server also supports Http protocol to provide services

Someone knows the resolution of generating id with MySQL:

```
REPLACE INTO Tickets64 (stub) VALUES ('a');
SELECT LAST_INSERT_ID();
```

The disadvantage of this resolution is that generates one id need query MySQL once. When generating id too quickly, leading MySQL overload. This is why I build this project to generating ids.

## [中文说明](README.zh-CN.md)


## 2. The architecture

GenId talks with clients using redis protocol, developer can connect to genid using redis sdk. Every Key will mapped to a table storing in MySQL, and the table only has one row data to record the id.

GenId only supports four commands of redis as follows:

- `SET key value`, set the initial value of id generator.
- `GET key`, get the value of key.
- `EXISTS key`, check the key if exist.
- `DEL key`, delete the key from server.
- `SELECT index`, just a mock select command, prevent the select command error.

## 3. Install

Install following these steps:

1. Install Go environment, the version of Go is greater than `1.12`.
3. `git clone https://github.com/inherelab/genid`
4. `cd genid`
5. `go mod tidy`
7. set the config file.
8. run genid: `./bin/genid -config=config/config.toml`

Set the config file(`config/config.toml`):

```ini
# the address of genid server
addr="127.0.0.1:6389"
# log_path: /Users/inhere/go/dev 
log_level="debug"

[db]
mysql_host="127.0.0.1"
mysql_port=3306
db_name="idgen_test"
user="root"
password=""
max_idle_conns=64
```

## Quick start

Examples:

```
# start server
➜  genid git:(master) ✗ ./bin/genid redis -config=config/config.toml

#start a redis client to connecting genid server, and set/get the key.
➜  ~  redis-cli -p 6389
redis 127.0.0.1:6389> get abc
(integer) 0
redis 127.0.0.1:6389> set abc 100
OK
redis 127.0.0.1:6389> get abc
(integer) 101
redis 127.0.0.1:6389> get abc
(integer) 102
redis 127.0.0.1:6389> get abc
(integer) 103
```

## 4. HA

When the server crashed, you can restart `genid` and reset the key by increasing a fixed offset.

## 5. License

MIT 
