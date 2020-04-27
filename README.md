

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



## Linux下安装ELK(Elasticsearch、Kibana、logstash)

### 1. 下载elasticsearch（6.8.7版本）  
elastic下载地址：[elasticsearch6.8.7](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-8-7)
![elastic](https://note.youdao.com/yws/api/personal/file/66F0E7CC939040B18785A114E013F9A6?method=download&shareKey=f92883f3755b11712211ad3f5fe5924b)

### 2.下载kibana（6.8.7版本）   
kibana下载地址：[kibana6.8.7](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-8-7)
![kibana](https://note.youdao.com/yws/api/personal/file/17DDC7EEECA1426C84DBB6CB02F39AB7?method=download&shareKey=74db165fd274c6fa99fa9360cb0a6c33)

### 3.下载logstash（6.8.7版本）   
logstash下载地址：[logstash6.8.7](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-8-7)
![logstash](https://note.youdao.com/yws/api/personal/file/550AACCB62B142EBBF09719E6E0ED787?method=download&shareKey=8a109a6200fbeaee10940811d702b28e)

将下载好的文件传输到linux中
![elk](https://note.youdao.com/yws/api/personal/file/B3D77E7E609E4502B2F124689E1D0345?method=download&shareKey=35fa0e9854d640bd2e410a364f44e288)

在安装ES之前需要先安装Java配置环境。这个百度一下吧，我机器上都有装了，就不示范了。

es不能在root用户上运行，需要在普通用户运行，但我已经有用户了就不重新新建了。

### 4.安装Elasticsearch

4.1 解压es文件,并来到bin目录下
```
tar -zxvf elasticsearch-6.8.7.tar.gz -C /home/grid/
ls
cd elstaicsearch-6.8.7/bin/
```
![img](https://note.youdao.com/yws/api/personal/file/7CE4B7C0D0264A4EAEEF7CCFE832A8E1?method=download&shareKey=7ef49437998269db366f95006409a3d8)
![img](https://note.youdao.com/yws/api/personal/file/2E87C7129C9B4A05A623E65CEBB43309?method=download&shareKey=29bc880b7f09eca057e293a8756a4813)

4.2 启动es，在es的bin目录下
```
./elstaicsearch
```
> 后台启动可以    ./elasticsearch -d

![img](https://note.youdao.com/yws/api/personal/file/7B21F7C63A4345A7B60A722D97146AF0?method=download&shareKey=9adbc17e7cd42822e58636edb870c116)
![img](https://note.youdao.com/yws/api/personal/file/4BEE84EA92994C7B85CCA229C5620508?method=download&shareKey=1ce5efd8107ccd084ac9092c68a9bb7f)

4.3 测试连接（另起一个界面），会看到一份JSON数据
```
curl 127.0.0.1:9200
```
![img](https://note.youdao.com/yws/api/personal/file/3381A25859EA45CAAD3DEF3F9C5639FF?method=download&shareKey=65a9ae7434810a48964f86fce371d086)

4.4 实现远程访问，在config/elasticsearch.yml配置
```
vi elasticsearch.yml
```
![img](https://note.youdao.com/yws/api/personal/file/7CCFA59DF069483FB39B2FC3A2BBE932?method=download&shareKey=367f24a6070ef5d4a65bc9c49bd86d03)
需要修改的配置内容，cluster.name(集群名称)，path.data(数据存放位置)，path.logs(日志存放位置)，network.host(集群ip)，http.port(集群端口)
```
cluster.name: my-application
path.data: /home/grid/elasticsearch-6.8.7/data
path.logs: /home/grid/elasticsearch-6.8.7/logs
network.host: 192.168.171.101
http.port: 9200
```
![img](https://note.youdao.com/yws/api/personal/file/20DE0D85E25B40E884632FC3160BFF9F?method=download&shareKey=0fe13f697fe51b8a246788c226d918c6)

4.5 重新启动es
```
./elstaicsearch
```
![img](https://note.youdao.com/yws/api/personal/file/7B21F7C63A4345A7B60A722D97146AF0?method=download&shareKey=9adbc17e7cd42822e58636edb870c116)

出现错误，无法启动
```
ERROR: [3] bootstrap checks failed  
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]  
[2]: max number of threads [3863] for user [grid] is too low, increase to at least [4096]  
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144] 
``` 
![img](https://note.youdao.com/yws/api/personal/file/452657E547A3483EA3ED05C15CF004F9?method=download&shareKey=77fbd45e9864540436d309a72a87ae99)

来root用户修复问题  
修复问题[1]，修改limits.conf文件
```
su root
vi /etc/security/limits.conf 
```
![img](https://note.youdao.com/yws/api/personal/file/26B7D87206CB47E794E3B598D68531E8?method=download&shareKey=5b8fa93e3dfe3a9a6ba9345453000f9e)
```
grid soft nofile 65536
grid hard nofile 65536
grid soft nproc 4096
grid hard nproc 4096
```
![img](https://note.youdao.com/yws/api/personal/file/E995E69D64D742819AD2A1482355D695?method=download&shareKey=b106241abc933252f24b3e63c89909e4)

修复问题[2]，修改20-nproc.conf文件
```
vi /etc/security/limits.d/20-nproc.conf 
```
![img](https://note.youdao.com/yws/api/personal/file/0E73FBBE956341B7A90F1E587A5CE09C?method=download&shareKey=a6ee2b2393a1f0269bb18d1f752649bb)
```
grid soft nproc 4096
```
![img](https://note.youdao.com/yws/api/personal/file/739D2FB8878748E68D651D9D6696B472?method=download&shareKey=aeb19fdf8c2e6cbe5ecea7b20a3ec258)

修复问题[3]，修改sysctl.conf文件
```
vi /etc/sysctl.conf
```
![img](https://note.youdao.com/yws/api/personal/file/49EE3CEFA470486DA7359939F1A78EC5?method=download&shareKey=52ccf113a0baf790a62eac71a111236f)
```
vm.max_map_count=655360
```
![img](https://note.youdao.com/yws/api/personal/file/F679FFA7D2BE41F7B69CC784A65125FA?method=download&shareKey=184165aa6f6f382070f978a68ab61e94)

执行以下命令生效，并且关闭防火墙
```
sysctl -p
systemctl stop firewalld.service
```
![img](https://note.youdao.com/yws/api/personal/file/D2FD37DAA52840D095C79756DDA53489?method=download&shareKey=8139e79ad706a6c3f2bc8e850bd3da4a)

重启虚拟机，并且在grid用户，再次重启启动es
```
./elstaicsearch
```
![img](https://note.youdao.com/yws/api/personal/file/7B21F7C63A4345A7B60A722D97146AF0?method=download&shareKey=9adbc17e7cd42822e58636edb870c116)
![img](https://note.youdao.com/yws/api/personal/file/D2599F3F8FBF4D608A10975C5A82B0C1?method=download&shareKey=b7b4925e815facd94cc6ebad5dbd54b5)
4.7 启动成功，访问网址测试连接
```
http://slave1:9200/
```
![img](https://note.youdao.com/yws/api/personal/file/ECE3F33B907740DA9FA926C8EF87A5BA?method=download&shareKey=94bd665e215db3829c8346ccc8258aae)

### 5.安装head(elasticsearch-head-master)

elasticsearch-head-master是es的集群管理工具，可以用来数据的浏览和查询。

> head是开源软件，托管在github，需要在github下载，地址：git://github.com/mobz/elasticsearch-head.git。   
> head的运行需要用到grunt，而grunt需要npm，所以需要安装nodejs。    
> es5.0之后，head已经不做为插件方在plugins目录，直接拷贝在本地就行。    

#### 5.1 安装nodejs
nodejs下载地址：[nodejs下载地址](https://nodejs.org/en/download/)
![img](https://note.youdao.com/yws/api/personal/file/ED2191C10FB748B6B5217135EB77CEE3?method=download&shareKey=5ab9db5885ea3c4297148369c8a1b460)

5.1.1 将nodejs传输到linux中的/home/grid/software目录

5.1.2 转到root用户，将nodejs解压到/usr目录下
```
su root
tar xf node-v12.16.1-linux-x64.tar.xz -C /usr/
```
![img](https://note.youdao.com/yws/api/personal/file/27BA9D498AB4417BA4C11F061366F0E3?method=download&shareKey=77a85b5a31062e5d61dc609b48ae7511)

5.1.3 重命名nodejs文件夹，并且配置环境变量
```
mv node-v12.16.1-linux-x64 node-v12.16.1
vi /etc/profile
```
![img](https://note.youdao.com/yws/api/personal/file/ED5B999788D542D79D419933B22A3E8E?method=download&shareKey=12338ad5587bee6b4c22d41630fbbcaf)
```
export NODEJS_HOME=/usr/node-v12.16.1
:$NODEJS_HOME/bin
```
![img](https://note.youdao.com/yws/api/personal/file/6D81739B8E8F4C8EAAE19E27A421AB5E?method=download&shareKey=c9d916cfb9d727da8b382f3579fe5d05)

5.1.4 让配置文件生效
```
source /etc/profile
```
![img](https://note.youdao.com/yws/api/personal/file/E11323DB5522425EA11E4F38999FE178?method=download&shareKey=faa11ee8a095c72257362aa1f4c2c8fb)

5.1.5 验证是否安装成功，查看node版本
```
node -v
```
![img](https://note.youdao.com/yws/api/personal/file/B9328FE2005B4D0194A30DE2A99A92FE?method=download&shareKey=9641da3bb4039582f4936975f31ac4b2)

#### 5.2 安装elasticsearch-head-master

5.2.1 下载elasticsearch-head-master

head下载地址：[head下载地址](https://github.com/mobz/elasticsearch-head)
![img](https://note.youdao.com/yws/api/personal/file/E5303F91CD464166A0A19BF75AED25BE?method=download&shareKey=c9d18e709537205306f00378d956ac8a)
将下载好的head传输到linux的/home/grid目录
![img](https://note.youdao.com/yws/api/personal/file/C1B1080116C34A4EBEB999167AB229E0?method=download&shareKey=d5699a6146886839133eab72622fced6)

5.2.2 安装grunt-cli
```
npm install -g grunt-cli
```
![img](https://note.youdao.com/yws/api/personal/file/6AA31CC2E9634F60901CF8B9AA5E835C?method=download&shareKey=420278993509106010289fe23e039075)

5.2.3 来到head目录下安装head的依赖包
```
cd elasticsearch-head-master/
```
![img](https://note.youdao.com/yws/api/personal/file/42F1BF7142D8448C9331E4A69B16F2D3?method=download&shareKey=0beafa904eb11ed6d505add863ff8d8b)
> 注：直接使用npm install下载速度可能会比较慢，可以用阿里云的镜像下载
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
或者
npm install 
```
![img](https://note.youdao.com/yws/api/personal/file/AD7C68532DA947D8A31DB40C789FD9CB?method=download&shareKey=cdf601fb5c8474570be915288e4fe13b)

5.2.4 查看版本
```
grunt -version
```
![img](https://note.youdao.com/yws/api/personal/file/D31E5EF5D67B4AA5ACEF201725AAB15F?method=download&shareKey=6f080b84d55eccbe7f55745a5d2a63cf)

5.2.5 修改Gruntfile.js配置文件
```
vi Gruntfile.js
```
![img](https://note.youdao.com/yws/api/personal/file/E9B91D29470A4CAFAEC5E778FD07EF3B?method=download&shareKey=85332ecf9c43bd3f11bc26d9be69aa5c)
    在connect-->server-->options下面添加：hostname:’*’，允许所有IP可以访问
```
hostname:'*',
```
![img](https://note.youdao.com/yws/api/personal/file/8F78D9D82F3F4AD593070D9F13D5D341?method=download&shareKey=1d9ec6762a8381ce75d900855f96a28d)

5.2.6 修改_site目录下的app.js中head的默认连接地址
```
cd _site
vi app.js
```
![img](https://note.youdao.com/yws/api/personal/file/995AEAB8C0784575BA22401C710158B9?method=download&shareKey=da470a17801dc51ad25af86672879970)
```
"http://localhost:9200"改成"http://192.168.171.101:9200"
```
![img](https://note.youdao.com/yws/api/personal/file/7696CFAF77244C5D9422606D023192E9?method=download&shareKey=fabd339e2df393c32a24a41f126049b4)

5.2.7 配置es允许跨域访问，es目录下的config的elasticsearch.yml
```
cd config
vi elasticsearch.yml
``` 
![img](https://note.youdao.com/yws/api/personal/file/1DA01433852443C08675994A66017279?method=download&shareKey=69377ce4bec42c199ffa2817fe3f6aed)
在文件末尾追加
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```
![img](https://note.youdao.com/yws/api/personal/file/E274FD8FC88D411EA77C82D6417FBA5A?method=download&shareKey=9ba5c1465b25994ec12da3995f9d1d29)

5.2.8 启动head
```
grunt server
```
![img](https://note.youdao.com/yws/api/personal/file/97FC084C4D1640659692465693442F07?method=download&shareKey=a44f6afaccb6a2f299f45f56dd08f99a)

5.2.9 在浏览器访问head
```
http://slave1:9100
```
![img](https://note.youdao.com/yws/api/personal/file/407557B14FDC4FF59204932DA8E5D5D4?method=download&shareKey=8c85732afd284ec60786d9136ebec767)


### 6.安装Kibana

6.1 将已经下载好的Kibana传输到linux的/home/grid/software目录

6.2 解压缩kibana
```
tar -zxvf kibana-6.8.7-linux-x86_64.tar.gz -C /home/grid/
```
![img](https://note.youdao.com/yws/api/personal/file/8D1795C45F3E42C98FB485F6FA02C083?method=download&shareKey=29edd2b0aa47d6f27eb17cc17bf40307)
![img](https://note.youdao.com/yws/api/personal/file/B3FF4120AE5548D29A3038E953521C6A?method=download&shareKey=d3655e610a6813caf97e583299d24468)

6.3 修改配置文件在config下的kibana.yml
```
cd kibana-6.8.7-linux-x86_64/config/
vi kibana.yml
```
![img](https://note.youdao.com/yws/api/personal/file/DC54A2B114DD4D8BA29116889CD21C5F?method=download&shareKey=14cebc072b2b0afeced161cb86cf7eaf)
```
server.port: 5601
server.host: "192.168.171.101"
elasticsearch.hosts: ["http://192.168.171.101:9200"]
```
![img](https://note.youdao.com/yws/api/personal/file/9E141BDF5DDE443DB18F573A9588B7A0?method=download&shareKey=6595de199acaa631ccc33fbe2ed6b1e4)

6.4 启动kibana，在kibana的bin目录下执行
> 启动kibana之前需要先启动elasticsearch

```
./kibana
```
![img](https://note.youdao.com/yws/api/personal/file/9BF3D3A924964296B31903ECB7ED5B13?method=download&shareKey=a3db12f19ceaa0186df175b047071da3)
![img](https://note.youdao.com/yws/api/personal/file/4F4EA3B459DD4CCB8AD0E66AA3DCF949?method=download&shareKey=e753afa28fbdab7b7b594c1e793f9ff7)

6.5 在浏览器访问kibana
```
http://slave1:5601/
```
![img](https://note.youdao.com/yws/api/personal/file/C531F9787B4A42A19C01560527C4957B?method=download&shareKey=ac2a2c0ac0687d0d278b1dcb04437a51)


### 7.安装中文分词器

7.1 下载中文分词器(elasticsearch-analysis-ik-master)
下载地址：[下载地址](https://github.com/medcl/elasticsearch-analysis-ik)
![img](https://note.youdao.com/yws/api/personal/file/4D4819C6EF56401FA166AB80388CBFB7?method=download&shareKey=b67624628daa90e2f65a730acb62fbe8)

7.2 将下载好的ik传输到linux中

7.3 将解压后的ik文件夹移动到es的plugins目录下
![img](https://note.youdao.com/yws/api/personal/file/6508BBB4639A48E58CD38304D769837A?method=download&shareKey=6a9d27123ed014815acb77a0151e9565)


### 8.安装logstash

8.1 将logstash传输到linux下的/home/grid/software目录

8.2 解压缩logstash
```
tar -zxvf logstash-6.8.7.tar.gz  -C /home/grid/
```
![img](https://note.youdao.com/yws/api/personal/file/D706DEB2A7A7416C9BF3144E648D5332?method=download&shareKey=f91a43690265dac21baf0806bd7bb256)
![img](https://note.youdao.com/yws/api/personal/file/86E04BEB595740EF913DC9475F676B84?method=download&shareKey=25efd3823083a33247c645dc0c9ca96e)

8.3 启动logstash小案例
```
./logstash -e 'input { stdin { } } output { stdout {} }'
```
![img](https://note.youdao.com/yws/api/personal/file/BBC9142929DC4328800BFEB129CA1D35?method=download&shareKey=9d4d9d1628cee0191f70e6ae8cf3295e)
启动成功
![img](https://note.youdao.com/yws/api/personal/file/A7C836D7E69B4E8ABDA6C4BB79A92554?method=download&shareKey=3295afe0e78e7fa6d80fa2352c520ed2)

> 一般启动logstach是需要配置input，output   

比如新建一个demo，在demo中新建一个xxx.conf，将input和output配置在conf时启动方式：
```
./logstash -f ../demo/xxx.conf
```

### 9.Linux启动ELK步骤
9.1 启动elasticsearch
    elasticsearch目录下（master）
```
./bin/elasticsearch
```

9.2 启动elasticsearch-head-master
    elasticsearch-head-master目录下
```
grunt server
```

9.3 启动kibana  
    kibana目录下
```
./bin/kibana
```    

9.4 启动logstash（选择性是否启动）
    logstash目录下
```
./bin/logstash -e 'input {stdin {} } output{ stdout {}}'
```    

