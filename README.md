# A-multi-pricing-engine-based-on-knowledge-graphs
# 基于知识图谱的多元定价引擎

## 一、基本介绍
- **名称**：基于知识图谱的多元定价引擎  
- **用途**：数据集定价  


## 二、安装与配置

### 1、数据处理
代码路径：`Table_Data.zip`  

处理步骤：  
1. 将待处理数据放入名为“AAAD”的文件夹中（需修改路径为实际数据集路径）；  
2. 运行`t1.py`程序：遍历读取文件夹下的数据文件，提取文件名、大小、来源、属性数目、属性名称集合等信息，整合并生成CSV表格`T1.csv`；  
3. 运行`t2.py`程序：使用`difflib`库比对各表属性列相似度，若两表存在2个高相似度属性列，则判定存在关系，以最高相似度属性列名为关系名，生成三元组记录文件`T2.csv`（每条记录含两张表名称及关系）。  


### 2、模型初次训练
代码路径：`model.zip`  

训练步骤：  
1. 补全`T1.csv`数据，字段说明如下：  

| 列名             | 数据类型       | 含义                  | 备注                                  |
|------------------|----------------|-----------------------|---------------------------------------|
| name             | String/float   | 数据集名              | 来自T1                                |
| size             | String/float   | 大小                  | 来自T1                                |
| source           | String/float   | 来源                  | 来自T1                                |
| column_nums      | String/float   | 数据集包含总列数      | 来自T1                                |
| column_name      | String/float   | 各列名称              | 来自T1                                |
| application_value| String/float   | 应用价值              | 计算公式：`application_value = use_nums / column_nums / 2` |
| price            | String/float   | 价格                  | 计算公式：`price = 2.5 * 0.01 * size * attr_nums * app` |
| LastUpdatedTime  | String/float   | 最后一次更新时间      | 根据数据集实际情况输入                |
| DataExpirationTime | String/float | 数据过期时间          | 根据数据集实际情况输入                |
| use_nums         | String/float   | 用户选取列数          | 随机模拟                              |
| sell             | String/float   | 售出情况              | 首次训练随机模拟，后续用实际数据      |

2. 打开数据预处理文件`1.py`，修改读取文件路径，生成`train_value.csv`（预处理后的特征值）；  
3. 打开`GBDT_regression`，修改文件路径，运行程序保存模型，完成训练；  
4. 将生成的`pmml`文件替换至路径：`data_trading\src\main\resources\py\model.pmml`。  


### 3、前端
代码路径：`AAA.zip`  

#### （一）安装步骤
本系统前端基于`nodejs`和`vue`框架，安装步骤如下：  
1. 使用`webstorm`打开项目；  
2. 在控制台运行命令安装依赖：  
   ```bash
   yarn install
   ```  
3. 依赖安装完成后，运行命令在浏览器启动：  
   ```bash
   yarn run serve
   ```  


#### （二）使用方法
1. **注册**：用户输入用户名和密码完成注册；  
2. **登录**：输入用户名和密码登录，进入系统首页；  
3. **查询数据集**：在上方输入框输入查询内容，点击“查询”按钮查看结果，点击“查看”进入数据定价页面；  
4. **数据定价**：在定价页面查看数据名称、来源和大小，选中所需数据列后点击“查询价格”，系统给出定价结果。  


### 4、服务端部署设计
代码路径：`data_trading.zip`  

#### （一）云服务器部署
本系统选用阿里云轻量应用服务器（华东1地域），部署步骤如下（以阿里云为例）：  

1. 进入阿里云官网，在“产品→弹性计算”中选择“轻量应用服务器”；  
2. 配置服务器资源：  
   - 地域：华东1（杭州）；  
   - 操作系统：Linux Ubuntu20.04镜像；  
   - 配置：2核4G，云存储空间60GB，限峰值带宽5Mbps，购买时长1年；  
3. 填写管理员密码，完成购买；  
4. 进入服务器控制台，设置密钥用于后续远程连接；  
5. 使用`Final Shell`工具建立SSH远程连接：输入服务器公网IP、连接名称、私钥`pem`文件；  
6. 配置防火墙安全规则：开启系统连接端口、数据库端口、远程连接端口、缓存服务端口等。  


#### （二）OS的选择与部署
选用Linux主流操作系统`Ubuntu`（轻量、安全、便捷），替代方案可考虑`CentOS`（基于Red Hat源代码编译，适合开发）。  


#### （三）OOPL的选择与部署
1. **C++**：Ubuntu已集成`gcc`与`g++`，无需额外配置，用于修改文件配置时的编译；  

2. **Java**：安装JDK1.8版本：  
   ```bash
   sudo apt-get install openjdk-8-jdk
   ```  
   设置环境变量（作用于所有用户）：  
   ```bash
   vim /etc/profile
   ```  
   在文件最下方添加：  
   ```bash
   export JAVA_HOME=/usr/local/java/jdk1.8
   export JRE_HOME=${JAVA_HOME}/jre
   export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
   export PATH=.:${JAVA_HOME}/bin:$PATH
   ```  
   使配置生效：  
   ```bash
   source /etc/profile
   ```  

3. **Undertow**：JFinal框架支持的高性能服务器（优于Tomcat），配置后用于高并发业务；  

4. **Nginx**：作为HTTP和反向代理web服务器，安装命令：  
   ```bash
   sudo apt-get install nginx
   ```  


#### （四）DBMS的选择与部署
1. **MySQL**：用于维护用户数据，安装命令：  
   ```bash
   sudo apt-get install mysql-server  # 服务端
   sudo apt-get install mysql-client  # 客户端
   sudo apt-get install libmysqlclient-dev  # 编译链接库
   ```  

2. **neo4j**：图数据库，部署步骤：  
   ① 下载资源：  
      ```bash
      wget https://neo4j.com/artifact.php?name=neo4j-community-3.5.11-unix.tar.gz
      ```  
   ② 解压并进入文件夹：  
      ```bash
      tar -xzvf artifact.php\?name\=neo4j-community-3.5.11-unix.tar.gz
      cd neo4j-community-3.5.11/
      ```  
   ③ 修改`neo4j.conf`文件配置。  


#### （五）图数据库实体关系数据导入
配置neo4j后，通过URL访问图形界面，导入步骤：  

1. 将CSV格式数据文件上传至云服务器neo4j目录的`import`模块下；  
2. 使用`Load csv`语句导入：  
   - **实体导入**（T1表）：  
     ```cypher
     Load csv with headers from "file:///usr/neo4j/neo4j-community-3.5.26/import/tzbAAA2.csv" as line 
     create(c:Data{name:line.name,size:line.size,source:line.source,attr_nums:line.attr_nums,attr_names:line.attr_names})
     ```  
   - **关系导入**（T2表）：  
     ```cypher
     Load csv with headers from "file:///usr/neo4j/neo4j-community-3.5.26/import/tzbRelation.csv" as line 
     match (c:Data),(d:Data) where c.name=line.table1 and d.name=line.table2 
     merge (c)-[r:DataRel{name:line.similarity}]->(d)
     ```  


#### （六）运行后端项目
1. 配置`maven`：  
   ① 打开IDEA，进入`File→Settings→Maven`，配置`Maven home path`，取消勾选`work offline`，点击`Apply`；  
   ② 确保网络正常，在右侧maven视图点击“重新加载”引入依赖；  
   ③ 在`App`主类中启动，本地项目启动成功。  


#### （七）开发软件环境
| 类型         | 具体配置                                  |
|--------------|-------------------------------------------|
| 操作系统     | Ubuntu 20.04                              |
| 数据库       | MySQL 8.0.31、neo4j 3.5                   |
| 开发环境     | JRE 1.8、Node.js                          |
| 开发工具包   | JDK 1.8、Maven 3.8.4                      |
| 包管理工具   | NPM、CNPM                                 |
| Web服务器    | Undertow、Nignx 1.9.9                     |
| 浏览器       | Edge、Chrome                              |
| 应用框架     | JFinal、Vue                               |
| 本地编译器   | IDEA、WebStorm、HBuilderx                 |
