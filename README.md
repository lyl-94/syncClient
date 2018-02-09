### **syncClient**

>   syncClient，数据实时同步中间件（阿里canal到kafka、redis）！

 本项目是打通canal、kafka的桥梁；  
 基本原理：  
 canal解析binlog的数据，由syncClient订阅，然后实时推送到kafka或者redis；如果kafka、redis服务异常，syncClient会回滚操作；canal、kafka、redis异常退出，都不会影响数据的传输；


---

**目录：**  
bin：已编译二进制项目，可以直接使用；  
src：源代码；  

---

**配置说明：**

#common  
system_debug=1          # 是否开始调试：1未开启，0为关闭（线上运行请关闭）  

#canal  
canal_ip=127.0.0.1      # canal 服务端 ip;  
canal_port=11111        # canal 服务端 端口：默认11111;  
canal_destination=one   # canal 服务端项目，多个用逗号分隔，如：one,two;  
canal_username=         # canal 用户名：默认为空;   
canal_password=         # canal 密码：默认为空;  
canal_filter=           # canal 同步表设置，默认空使用canal配置;  

#kafka or redis  
target_type=kafka       # 同步插件类型 kafka or redis  
target_ip=              # kafka 服务端 ip;   
target_port=            # kafka 端口：默认9092;    

---

**使用场景(基于日志增量订阅&消费支持的业务)：**

数据库镜像  
数据库实时备份  
多级索引 (分库索引)  
search build  
业务cache刷新  
数据变化等重要业务消息  

**Kafka：**

Topic规则：数据库的每个表有单独的topic，如数据库admin的user表，对应的kafka主题名为：sync_admin_user  
Topic数据字段：

    {
        "head": {
            "binlog_pos": 53036,
            "type": "UPDATE",
            "binlog_file": "mysql-bin.000173",
            "db": "sdsw",
            "table": "sys_log"
        },
        "before": [
            {
                "name": "log_id",
                "update": false,
                "value": "1"
            },
            {
                "name": "log_ip",
                "update": false,
                "value": "27.17.47.202"
            },
            {
                "name": "log_addtime",
                "update": false,
                "value": "1494204717"
            }
        ]
    }

head.type 类型：INSERT（插入）、UPDATE（修改）、DELETE（删除）； 

head.db 数据库； 

head.table 数据库表；

head.binlog_pos  日志位置； 

head.binlog_file 日志文件；  

before： INSERT（插入）、UPDATE（修改）、DELETE（删除）操作下的数据；  

after：  UPDATE（修改）操作下的数据；  


**Redis：**

List规则：数据库的每个表有单独的list，如数据库admin的user表，对应的redis list名为：sync_admin_user  
