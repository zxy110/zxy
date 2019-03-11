---
title: LevelDB
date: 2019-03-11 11:29:26
categories:
  - DB
tags: [LevelDB]
---

## 简介

LevelDB是一种kv数据库，是一种单进程的服务，性能非常高。目前的版本支持billion级别的数据量，在一台4核Q6600的CPU机器上，每秒钟写数据超过40w，随机读每秒超过10w（完全命中内存的速度，若不命中，速度大大下降）。

LevelDB是一个C/C++编程语言的库，不包含网络服务封装，无法像一般意义的存储服务器（如MySQL）用客户端连接，使用者需要自己封装自己的网络服务器。

一次只允许一个进程访问一个特定的数据库，没有内置的C/S架构，但开发者可以使用LevelDB库自己封装一个server，不具备“分布式”集群架构能力。不支持索引和事务。

应用场景：内存占用大的服务，写多于读

## Java Demo

原生leveldb是基于C++开发，java语言无法直接使用；iq80对leveldb使用JAVA 语言进行了“逐句”重开发，经过很多大型项目的验证（比如ActiveMQ），iq80开发的JAVA版leveldb在性能上损失极少（10%）。对于JAVA开发人员来说，我们直接使用即可，无需额外的安装其他lib。

pox.xml

```
<dependencies>
        <dependency>
            <groupId>org.iq80.leveldb</groupId>
            <artifactId>leveldb</artifactId>
            <version>0.7</version>
        </dependency>
        <dependency>
            <groupId>org.iq80.leveldb</groupId>
            <artifactId>leveldb-api</artifactId>
            <version>0.7</version>
        </dependency>
    </dependencies>
```

LevelDB.java

```
import org.iq80.leveldb.DB;
import org.iq80.leveldb.Options;
import org.iq80.leveldb.impl.Iq80DBFactory;

import java.io.File;

public class LevelDb {
    private static DB db;
    private static String charsetName = "UTF-8";

    public LevelDb(String filename){
        connectLeveldb(filename);
    }
    /*
     * Open a connection to leveldb
     * You need to give filename where .db files saves
     */
    public static void connectLeveldb(String filename){
        File file=new File(filename);
        try{
            db = Iq80DBFactory.factory.open(new File(file,"db"),new Options().createIfMissing(true));
        }catch(Exception e){
            e.printStackTrace();
        }
    }

    /*
     * Add an element to leveldb
     */
    public static void put(String key, String value){
        try {
            db.put(key.getBytes(charsetName), value.getBytes(charsetName));
        }catch(Exception e){
            e.printStackTrace();
        }
    }

    /*
     * Find an element
     */
    public static byte[] get(String key){
        try {
            return db.get(key.getBytes(charsetName));
        }catch(Exception e){
            e.printStackTrace();
            return null;
        }
    }

    /*
     * Delete an element
     */
    public static void delete(String key){
        try {
            db.delete(key.getBytes(charsetName));
        }catch(Exception e){
            e.printStackTrace();
        }
    }

}
```



## 参考文献：

https://blog.csdn.net/future_1024/article/details/82155680