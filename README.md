

### Linux启动ELK步骤：
1.启动elasticsearch
    elasticsearch目录下（master）
```
    ./bin/elasticsearch
```
2.启动elasticsearch-head-master
    elasticsearch-head-master目录下
```
    grunt server
```
3.启动kibana  
    kibana目录下
```
    ./bin/kibana
```    
4.启动logstash（选择性是否启动）
    logstash目录下
```
    bin/logstash -e 'input {stdin {} } output{ stdout {}}'
```       

bin/logstash -e 'input {stdin {} } output{ stdout {}}'




### Windows启动步骤：
1.启动elasticsearch
```
启动服务中的elasticsearch
```
2.启动elasticsearch-head-master
``` shell
cd D:\elasticsearch-head-master
grunt server
```
3.启动kibana  
```
打开：D:\kibana-6.8.7-windows-x86_64\bin
双击 kibana.bat
```


