## elasticsearch

### docker启动elasticsearch

```bash
docker run -p 9200:9200 -p 9300:9300 --name elasticsearch -e "discovery.type=single-node" \
-v /run/elasticsearch/logs:/usr/share/elasticsearch/logs \
-v /run/elasticsearch/data:/usr/share/elasticsearch/data \
-v /run/elasticsearch/temp:/run/elasticsearch/temp \
-v /run/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
docker.elastic.co/elasticsearch/elasticsearch:7.6.1
```

> -v 代表挂在本地的插件，例如中文分词插件

### linux中不能使用root启动es

启动es报错：

```bash
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:174) ~[elasticsearch-7.6.1.jar:7.6.1]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:161) ~[elasticsearch-7.6.1.jar:7.6.1]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-7.6.1.jar:7.6.1]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:125) ~[elasticsearch-cli-7.6.1.jar:7.6.1]
        at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-7.6.1.jar:7.6.1]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:126) ~[elasticsearch-7.6.1.jar:7.6.1]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92) ~[elasticsearch-7.6.1.jar:7.6.1]
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:105) ~[elasticsearch-7.6.1.jar:7.6.1]
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:172) ~[elasticsearch-7.6.1.jar:7.6.1]
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:349) ~[elasticsearch-7.6.1.jar:7.6.1]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:170) ~[elasticsearch-7.6.1.jar:7.6.1]
        ... 6 more
uncaught exception in thread [main]
java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:105)
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:172)
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:349)
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:170)
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:161)
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86)
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:125)
        at org.elasticsearch.cli.Command.main(Command.java:90)
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:126)
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92)
For complete error details, refer to the log at /usr/local/elasticsearch-7.6.1/logs/elasticsearch.log
```

es为了安全不允许用root启动，所以我们添加一个es用户

1. 创建用户：elasticsearch

   ```bash
   adduser elasticsearch
   ```

2. 创建用户密码

   ```bash
   passwd elasticsearch
   ```

3. 将对应的文件夹权限赋给该用户

   ```bash
   chown -R elasticsearch elasticsearch-7.6.1
   ```

4. 切换至elasticsearch用户

   ```bash
   su elasticsearch
   ```

5. 进入启动目录启动 /usr/local/elasticsearch-7.6.1/bin  使用后台启动方式：./elasticsearch -d

   ```bash
   ./elasticsearch -d
   ```


### elasticsearch.yml

```yaml
# 集群名
cluster.name: elasticsearch
# 节点名
node.name: opera-node-1
# 指定该节点是否有资格被选举成为master结点，默认是true
node.master: true
# 指定该节点是否存储索引数据，默认为true
node.data: true
# 主结点数量的最少值，此值的公式为:(master_eligible_nodes / 2) + 1，比如:有3个符合要求的主结点，那么这里要设置为2。
discovery.zen.minimum_master_nodes: 1
# 设置为true可以锁住ES使用的内存，避免内存与swap分区交换数据
bootstrap.memory_lock: false
# 单机允许的最大存储结点数，通常单机启动一个结点建议设置为1，开发环境如果单机启动多个节点可设置大于1
node.max_local_storage_nodes: 1
# 初始默认的主节点
cluster.initial_master_nodes: ["opera-node-1"]

# 数据存储位置(单个目录设置) 
# path.data: /path/to/data 
# 多个数据存储位置,有利于性能提升 
# path.data: /path/to/data1,/path/to/data2 

# 临时文件的路径 
# path.work: /path/to/work 

# 日志文件的路径 
# path.logs: /path/to/logs 

# 插件安装路径 
# path.plugins: /path/to/plugins

network.host: 0.0.0.0
# 设置对外服务的http端口，默认为9200
http.port: 9200
# 设置节点间交互的tcp端口，默认是9300
transport.tcp.port: 9300

# 设置是否压缩tcp传输时的数据，默认为false，不压缩
transport.tcp.compress: true
# 设置内容的最大容量，默认100mb
http.max_content_length: 100mb

# 允许跨域
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### 配置外部访问

1. max file descriptors

   修改/etc/security/limits.conf，添加或者修改如下（切换root用户）

   ```bash
   *  hard  nofile  65536
   *  soft  nofile  65536
   ```

2. max virtual memory

   修改/etc/sysctl.conf，添加如下（切换root用户）

   ```bash
   vm.max_map_count=2621441
   ```

   保存后执行sudo sysctl -p /etc/sysctl.conf 使之生效

3. 重启elasticsearch

### elasticsearch-head

#### 自己部署head插件

`es7.x好像并不完全支持head插件，在head中很多操作会报错，几乎就是不能用，这一点head插件仓库的作者也说了`

> for Elasticsearch 5.x, 6.x, and 7.x: site plugins are not supported.

```bash
docker run -p 9100:9100 --name elasticsearch-head mobz/elasticsearch-head:5
```

#### 安装Google浏览器版

建议：如果不想用自己部署的，可以去Google应用商店安装浏览器版本的elasticsearch-head

### elasticsearch设置跨域

在elasticsearch.yml中添加如下配置，完成后重启es。

```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### elasticsearch创建索引

`PUT http://localhost:9200/opera`

> opera就是要创建的索引名

```json
{
    "settings": {
        "index": {
            "number_of_shards": 1,
            "number_of_replicas": 0
        }
    }
}
```

### elasticsearch创建索引并创建自定义分词器

> 注意：如果想要创建自定义分词器不能在原有的索引上创建，因该和索引同时创建。

`PUT http://localhost:9200/opera`

```json
{
    "settings": {
        "index": {
            "number_of_shards": 1,
            "number_of_replicas": 0
        },
        "analysis": {
            "analyzer": {
                "comma_analyzer": {
                    "type": "pattern",
                    "pattern": ","
                }
            }
        }
    }
}
```

> comma_analyzer是我创建的自定义分词器，按逗号分词。

#### 查看索引信息

`GET http://localhost:9200/opera`

返回

```json
{
    "opera": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "number_of_shards": "1",
                "provided_name": "opera",
                "creation_date": "1583896818084",
                "analysis": {
                    "analyzer": {
                        "comma_analyzer": {
                            "pattern": ",",
                            "type": "pattern"
                        }
                    }
                },
                "number_of_replicas": "0",
                "uuid": "A56MgPPkTZOEYzsN4cB7Jg",
                "version": {
                    "created": "7060199"
                }
            }
        }
    }
}
```

#### 测试自定义逗号分词器

`POST http://localhost:9200/opera/_analyze`

```json
{
  "analyzer": "comma_analyzer",
  "text": "1-2-3,1-3-2,1-2,3-1"
}
```

返回

```json
{
    "tokens": [
        {
            "token": "1-2-3",
            "start_offset": 0,
            "end_offset": 5,
            "type": "word",
            "position": 0
        },
        {
            "token": "1-3-2",
            "start_offset": 6,
            "end_offset": 11,
            "type": "word",
            "position": 1
        },
        {
            "token": "1-2",
            "start_offset": 12,
            "end_offset": 15,
            "type": "word",
            "position": 2
        },
        {
            "token": "3-1",
            "start_offset": 16,
            "end_offset": 19,
            "type": "word",
            "position": 3
        }
    ]
}
```

### 配置中文分词器

1. 下载与es相同的版本：https://github.com/medcl/elasticsearch-analysis-ik/releases

2. 进入elasticsearch安装目录下的plugins目录，创建ik目录

   ```bash
   cd /usr/local/elasticsearch7.6.1/plugins
   mkdir ik
   ```

3. 将下载好的zip包放到ik文件夹中并解压

   ```bash
   unzip elasticsearch-analysis-ik-7.6.1.zip
   ```

4. 重启elasticsearch

   `我的es是放在docker中的，所以需要挂载外部插件；如果不是docker中的，直接解压到es安装目录中的plugins目录下`

#### 测试中文分词器

> 索引时使用`ik_max_word`将搜索内容进行细粒度分词，搜索时使用`ik_smart`提高搜索精确性。

`POST http://localhost:9200/opera/_analyze`

**ik_max_word**

```json
{
  "analyzer": "ik_max_word",
  "text": "配置中文分词器"
}
```

返回

```json
{
    "tokens": [
        {
            "token": "配置",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "中文",
            "start_offset": 2,
            "end_offset": 4,
            "type": "CN_WORD",
            "position": 1
        },
        {
            "token": "分词器",
            "start_offset": 4,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 2
        },
        {
            "token": "分词",
            "start_offset": 4,
            "end_offset": 6,
            "type": "CN_WORD",
            "position": 3
        },
        {
            "token": "器",
            "start_offset": 6,
            "end_offset": 7,
            "type": "CN_CHAR",
            "position": 4
        }
    ]
}
```

**ik_smart**

```json
{
  "analyzer": "ik_smart",
  "text": "配置中文分词器"
}
```

返回

```json
{
    "tokens": [
        {
            "token": "配置",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "中文",
            "start_offset": 2,
            "end_offset": 4,
            "type": "CN_WORD",
            "position": 1
        },
        {
            "token": "分词器",
            "start_offset": 4,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 2
        }
    ]
}
```

### es常见的坑

#### 插入数据报错cluster_block_exception

在往ES中插入数据的时候，报错403，这是由于ES新节点的数据目录data存储空间不足，导致从master主节点接收同步数据的时候失败，此时ES集群为了保护数据，会自动把索引分片index置为只读read-only

解决方法：（一般情况下用第二种方法就可以解决）

1. 磁盘扩容，可在配置文件中修改ES数据存储目录，重启ES

2. 放开索引只读设置，在服务器上通过curl工具发起PUT请求

  `PUT http://localhost:9200/opera/_settings`

  ```json
  {
      "index": {
          "blocks": {
              "read_only_allow_delete": "false"
          }
      }
  }
  ```

#### 插入数据时如果字段值为null，ES会忽略该字段

插入数据是我用的是fastjson将数据对象转换成json字符串，我们可以在JSON.toJSONString方法参数中加一个过滤器，可以让value为null的字段都输出空字符串。

```java
// fastjson过滤器，对于null值字段设置值为""
ValueFilter filter = (Object object, String name, Object v) -> {
    if (v == null) {
        return "";
    }
    return v;
};
```

---

## logstash

### 下载docker镜像

```bash
docker pull docker.elastic.co/logstash/logstash:7.6.1
```

### docker启动

```bash
docker run -p 5044:5044 --name logstash  \
-v /datas/logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml \
-v /datas/logstash/config/log4j2.properties:/usr/share/logstash/config/log4j2.properties \
-v /datas/logstash/pipeline/:/usr/share/logstash/pipeline/ \
-v /datas/logstash/data/:/usr/share/logstash/data/ \
-v /datas/logstash/logs/:/usr/share/logstash/logs/ \
-v /datas/logstash/jar/:/usr/share/logstash/jar/ \
-d docker.elastic.co/logstash/logstash:7.6.1
```

### 修改logstash.yml

```yaml
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://123.59.197.176:9200" ]
```

### 修改pipelines.yml

```yaml
- pipeline.id: main
  path.config: "/usr/share/logstash/pipeline/logstash.conf"
```

### 自定义配置文件（全量拉取）

```bash
input {
    jdbc {
        jdbc_connection_string => "jdbc:mysql://123.59.199.68:33066/youth_database"
        jdbc_user => "root"
        jdbc_password => "youth_0420"
        jdbc_driver_library => "/usr/share/logstash/jar/mysql-connector-java-8.0.19.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        codec => plain { charset => "UTF-8"}
        jdbc_paging_enabled => "true"
        jdbc_page_size => "50000"
        statement => "select * from activity"
        schedule => "* * * * *"
    }
}

output {
    elasticsearch {
        hosts => ["http://123.59.197.176:9200"]
        index => "youth-activity"
        document_id => "%{id}"
    }
    stdout {
    	codec => json_lines
    }
}
```

### 自定义配置文件（增量更新）

```bash
input {
    jdbc {
    	# mysql 数据库链接
        jdbc_connection_string => "jdbc:mysql://123.59.199.68:33066/youth_database"
        # 用户名
        jdbc_user => "root"
        # 密码
        jdbc_password => "youth_0420"
        # 驱动包
        jdbc_driver_library => "/usr/share/logstash/jar/mysql-connector-java-8.0.19.jar"
        # 驱动类
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        # 编码
        codec => plain { charset => "UTF-8"}
        # 是否转换成小写
        lowercase_column_names => "false"
        # 是否分页
        jdbc_paging_enabled => "true"
        # 分页大小
        jdbc_page_size => "50000"
        # sql语句
        statement => "select * from activity where timestamp > :sql_last_value"
        # 定时执行
        schedule => "* * * * *"
        # 是否使用列元素
        use_column_value => true
        # 列元素类型
        tracking_column_type => "timestamp"
        # 追踪的元素名，对应保存到es上面的字段名而不是数据库字段名
        tracking_column => "timestamp"
        # 是否记录最后一次运行内容
        record_last_run => true
        # 设置记录的路径
        last_run_metadata_path => "/usr/share/logstash/config/youth_activity_timestamp"
        # 每次运行是否清除
        clean_run => false
    }
}

filter {
    date {
      timezone => "Asia/Chongqing"
    }
}

output {
    elasticsearch {
    	# es地址
        hosts => ["http://123.59.197.176:9200"]
        # 索引名称
        index => "youth-activity"
        # ID
        document_id => "%{id}"
    }
    stdout {
    	codec => rubydebug
    }
}
```

### log4j2.properties

```properties
status = error 
dest = err
name = PropertiesConfig

property.filename = /usr/share/logstash/logs

appender.rolling.type = RollingFile
appender.rolling.name = RollingFile
appender.rolling.fileName = ${filename}/logstash.log
appender.rolling.filePattern = ${filename}/logstash-%d{yyyy-MM-dd}-%i.log.gz
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = %d %p %C{1.} [%t] %m%n
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy
appender.rolling.policies.time.interval = 1
appender.rolling.policies.time.modulate = true
appender.rolling.policies.size.type = SizeBasedTriggeringPolicy
appender.rolling.policies.size.size = 50MB
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.max = 5

appender.console.type = Console
appender.console.name = plain_console
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = [%d{ISO8601}][%-5p][%-25c] %m%n

appender.json_console.type = Console
appender.json_console.name = json_console
appender.json_console.layout.type = JSONLayout
appender.json_console.layout.compact = true
appender.json_console.layout.eventEol = true

rootLogger.level = debug
rootLogger.appenderRef.rolling.ref = RollingFile
rootLogger.appenderRef.console.ref = ${sys:ls.log.format}_console
```



---

## kibana

### 下载docker镜像

```bash
docker pull docker.elastic.co/kibana/kibana:7.6.1
```

### docker启动

```bash
docker run -p 5601:5601 --link elasticsearch:elasticsearch \
--name kibana -d docker.elastic.co/kibana/kibana:7.6.1
```

---

