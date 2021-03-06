## OnceDB

OnceDB基于Redis开发。将redis从一个键值对内存数据库，改造成支持查询和关系的NoSQL数据库。

- [OnceDB on Linux](https://github.com/OnceDoc/OnceDB)
- [OnceDB on Windows](https://github.com/OnceDoc/OnceDB.win)


## 安装

Linux 

    git clone https://github.com/OnceDoc/OnceDB.git
    cd OnceDB
    make

Windows

请到 [Release](https://github.com/OnceDoc/OnceDB.win/releases) 下载二进制可执行文件


## Redis参考

- [Redis指令参考](https://redis.io)
- [Redis on Linux](https://github.com/antirez/redis)
- [Redis on windows](https://github.com/MSOpenTech/Redis)



## 搜索指令说明

### search [key pattern] [operator] [value]

搜索字符串类型的健值

准备测试数据

    127.0.0.1:6379> set test1 'this is testing'
    OK
    127.0.0.1:6379> set test2 'this is article'
    OK
    127.0.0.1:6379> set test3 kris
    OK
    127.0.0.1:6379> set test4 10
    OK
    127.0.0.1:6379> set test5 100
    OK

= 完全匹配搜索: 在指定pattern的key中搜索含有指定值的key和值

    127.0.0.1:6379> search test* = kris
    1) "test3"
    2) "kris"

~ 模糊搜索: 搜索含有指定值的key及其值

    127.0.0.1:6379> search test* ~ is
    1) "test2"
    2) "this is article"
    3) "test1"
    4) "this is testing"

~| 包含并截取: 搜索含有指定值的key及其值，如果值过长则截取其中一段值打印，如果文字较短则会不截取

    127.0.0.1:6379> search test* ~| is
    1) "test2"
    2) "this is article"
    3) "test1"
    4) "this is testing"

<、>、<=、>= 比较搜索: 找到符合条件的数字

    127.0.0.1:6379> search test* > 20
    1) "test5"
    2) "100"



### hsearch [key pattern] [field] [operator] [value]

搜索hash类型的健值，operator同search，支持：=、~、~|、>、<、>=、<=

准备测试数据

    127.0.0.1:6379> hmset user:001 name kris email c2u@live.cn gender male
    OK
    127.0.0.1:6379> hmset user:002 name sunny age 24
    OK

搜索age大于18的key和搜索到的值，并打印name字段

    127.0.0.1:6379> hsearch user:* age > 18 name ~ ''
    1) "user:002"
    2) "24"
    3) "sunny"

### hselect [num of fields] field1 field2 ... key1 key2 key3 ...

批量打印出key的指定字段值，不含对应字段的显示null

    127.0.0.1:6379> hselect 3 name email age user:001 user:002
    1) "user:001"
    2) "kris"
    3) "c2u@live.cn"
    4) (nil)
    5) "user:002"
    6) "sunny"
    7) (nil)
    8) "24"

### hmgetall key1 key2 ...

批量打印出指定key的全部字段和值，以两个null分隔

    127.0.0.1:6379> hmgetall user:001 user:002
     1) "user:001"
     2) "name"
     3) "kris"
     4) "email"
     5) "c2u@live.cn"
     6) "gender"
     7) "male"
     8) (nil)
     9) (nil)
    10) "user:002"
    11) "name"
    12) "sunny"
    13) "age"
    14) "24"
    15) (nil)
    16) (nil)



## 有序数列

一种特殊列表，只存储数字，分为升序和降序两种。

### xrange key from to

从一个有序列表中查找出范围内的值

    localhost:0>lrange list_bigtosmall 0 100000
    1) 1111
    2) 111
    3) 31
    4) 21
    5) 11
    6) 9
    7) 7
    8) 5
    9) 3
    10) 1

    localhost:0>xrange list_bigtosmall 2 9
    1) 9
    2) 7
    3) 5
    4) 3
    localhost:0>

### xsearch key value

从一个有序列表中查找某值位置

    xsearch list_bigtosmall 111
    1


### xinsert key value

创建一个小到大序列，返回新位置信息

    xinsert list2 111
    11

从小到大序列，返回新位置信息

    xinsert list_smalltobig 111
    11



### xinsertdesc key value

将数字插入从小到大序列

插入成功，返回新位置

    localhost:0>xinsertdesc list_bigtosmall 2
    9

插入重复项目，-1表示添加失败

    localhost:0>xinsertdesc list_bigtosmall 111
    -1

插入排序类型不符从大到小的key

    localhost:0>xinsertdesc list_smalltobiglocalhost:0> 111
    ERR Wrong list sort