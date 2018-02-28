### 说明
首先确保已安装JDK环境  
ElasticSearch,LogStash,Kibana安装的版本要保持一致  
由于elk-6.x + x-pack 需要较高的CPU内存配置启动时卡死所以并未使用此版本  
如果使用阿里云,远程访问服务要增加安全配置开放端口  

### 下载
https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.8.tar.gz  
https://artifacts.elastic.co/downloads/logstash/logstash-5.6.8.tar.gz  
https://artifacts.elastic.co/downloads/kibana/kibana-5.6.8-linux-x86_64.tar.gz  

### elasticsearch安装
mkdir /usr/local/elk-5.6.8  
cd /usr/local/elk-5.6.8  
tar zxvf elasticsearch-5.6.8.tar.gz  
mv elasticsearch-5.6.8 elasticsearch  
groupadd elasticsearch  
useradd elasticsearch -g elasticsearch  
chown -R elasticsearch:elasticsearch elasticsearch  
mkdir -p /usr/local/elk-5.6.8/data/es/data  
mkdir -p /usr/local/elk-5.6.8/data/es/logs  
chown -R elasticsearch:elasticsearch /usr/local/elk-5.6.8/data/es   
cd elasticsearch  
vim config/elasticsearch.yml  
```
path.data: /usr/local/elk-5.6.8/data/es/data
path.logs: /usr/local/elk-5.6.8/data/es/logs
bootstrap.memory_lock: false
network.host: 0.0.0.0
```

由于jvm默认分配内存是1G太大调低内存：  
vim config/jvm.options  
```
-Xms256m
-Xmx256m
```
 
如果这时启动会报错：max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]  
vim /etc/sysctl.conf  
```
vm.max_map_count=655360  
```  

vim /etc/security/limits.conf  
```
elasticsearch soft nproc 4096  
elasticsearch hard nproc 4096    
elasticsearch soft nofile 65536  
elasticsearch hard nofile 65536  
```

sysctl -p (重新加载配置)

su - elasticsearch -c "/usr/local/elk-5.6.8/elasticsearch/bin/elasticsearch -d"   (-d表示后台运行,注意不要用root运行)  

如果没什么问题的话：  
curl 'http://localhost:9200' 会输出一段json内容,安装成功

### logstash安装

cd /usr/local/elk-5.6.8/  
tar zxvf logstash-5.6.8.tar.gz  
mv logstash-5.6.8 logstash  
cd logstash  
vim config/accesslog2es.conf  
``` 
input{
    file{
        path =>"/data1/logs/flowphp.access.log"
        start_position=>"beginning"
    }
}

output {
    elasticsearch {
        hosts => ["localhost:9200"]
        #user => "elastic"
        #password => "changeme"
    }
    stdout { codec => rubydebug}
}

```

vim config/jvm.options  
```
-Xms256m
-Xmx256m
```

启动logstash  
./bin/logstash -f config/accesslog2es.conf

### kibana安装

tar zxvf kibana-5.6.8-linux-x86_64.tar.gz  
mv kibana-5.6.8 kibana  
cd kibana    
vim config/kibana.yml  
```
server.host: "0.0.0.0"
elasticsearch.url: "http://localhost:9200"
#elasticsearch.username: "elastic"
#elasticsearch.password: "changeme"
```

启动 ./bin/kibana  
后台 nohup /usr/local/elk-5.6.8/kibana/bin/kibana > /usr/local/elk-5.6.8/data/kibana.out 2>&1 &  
访问 http://localhost:5601/app/kibana  


### 参考

https://www.elastic.co/guide/en/x-pack/current/installing-xpack.html
https://www.cnblogs.com/lizichao1991/p/7809156.html  
http://blog.csdn.net/buqutianya/article/details/72019264?utm_source=itdadao&ut  
http://blog.csdn.net/u013066244/article/details/78699353  
http://blog.csdn.net/choelea/article/details/53841218  
 

