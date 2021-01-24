# performace_monitor
## 介绍
#### 已完成如下功能<br>
1、监控整个服务器的CPU使用率、io wait、内存使用、磁盘IO、网络带宽和TCP连接数<br>
2、监控指定端口的CPU使用率、上下文切换、内存占用大小、磁盘读写和TCP连接数<br>
3、针对java应用，可以监控jvm大小和垃圾回收情况；当Full GC频率过高时，可发送邮件提醒<br>
4、系统CPU使用率过高，或者剩余内存过低时，可发送邮件提醒；可设置自动清理缓存<br>
5、可随时启动/停止监控指定端口<br>
6、当端口重启后，可自动重新监控<br>
7、支持运维监控，端口停止后，可发送邮件提醒<br>
8、可直接在服务端停止客户端<br>
9、可按照指定时间段可视化监控结果<br>
10、自动按照百分位数计算出CPU、磁盘IO和带宽的数据<br>
11、数据采样频率最高可达约1次/s，可设置任意采样频率<br>
12、可直接在页面查看服务器当前资源（CPU、内存、磁盘）使用情况<br>
13、可同时管理监控多台服务器<br>

#### 实现过程
1、使用基于协程的http框架`aiohttp`<br>
2、服务端前端使用`jinjia2`模板渲染<br>
3、数据可视化采用`echarts`，可降低服务端压力<br>
4、采用线程池+队列的方式实现同时监控多个端口<br>
5、客户端每隔8s向服务端注册本机IP和端口<br>
6、服务端每隔10s会查询所有已注册的客户端的状态<br>
7、使用influxDB数据库存储监控数据；数据可设置自动过期时间<br>
8、为保证监控结果准确性，直接使用Linux系统命令获取数据，且可视化时未做任何曲线拟合处理<br>

#### 项目分支
| 分支 | 框架 | 特点 |
| :----: | :----:| :---- |
| [master分支](https://github.com/leeyoshinari/performance_monitor) | aiohttp | 1、可同时监控多台服务器<br> 2、可视化时使用echarts画图，可降低服务端的压力<br> 3、可视化的图形可交互 |
| [matplotlib分支](https://github.com/leeyoshinari/performance_monitor/tree/matplotlib) | aiohttp | 1、可同时监控多台服务器<br> 2、可视化时使用matplotlib画图，可降低客户端（浏览器）的压力 |
| [single分支](https://github.com/leeyoshinari/performance_monitor/tree/single) | flask | 1、单机监控<br> 2、监控数据存放在日志中<br> 3、使用matplotlib画图 |

## 使用
1. 克隆 performance_monitor
   ```shell
   git clone https://github.com/leeyoshinari/performance_monitor.git
   ```
   server文件夹是服务端，只需部署一个即可；agent文件夹是客户端，部署在需要监控的服务器上<br>

2. 分别修改server和agent文件夹里的配置文件 `config.ini`

3. 部署InfluxDB数据库。CentOS安装过程如下：<br>
    （1）下载并安装<br>
        `wget https://dl.influxdata.com/influxdb/releases/influxdb-1.8.3.x86_64.rpm` <br>
        `yum localinstall influxdb-1.8.3.x86_64.rpm` <br>
    （2）启动<br>
        `systemctl enable influxdb` <br>
        `systemctl start influxdb` <br>
    （3）修改配置<br>
         `vim /etc/influxdb/influxdb.conf` <br>
         第256行左右，修改端口：`bind-address = ":8086"` <br>
         第265行左右，不打印日志：`log-enabled = false` <br>
         重启 <br>
    （4）创建数据库<br>
        `create database test` <br>
        `use test` <br>
        `create user root with password '123456'` 创建用户和设置密码 <br>
        `grant all privileges on test to root` 授权数据库给指定用户 <br>
   
4. 分别运行server和agent文件夹中的`server.py`
   ```shell
   nohup python3 server.py &
   ```

5. 页面访问<br>
   （1）从机（客户端）启动后，输入`http://ip:port`可以看到页面显示服务器的CPU核数、总内存和磁盘号<br>
   ![slave home](https://github.com/leeyoshinari/performance_monitor/blob/master/master/static/slave.jpg)
   
   （2）主机（服务端）启动后，输入`http://ip:port/上下文`可以看到首页，页面展示已经注册的从机（客户端）的IP和注册时间<br>
   ![master home](https://github.com/leeyoshinari/performance_monitor/blob/master/master/static/home.jpg)
   
   （3）点击开始监控，可以看到监控页面；点击开始监控按钮，即可在指定的服务器上开始监控指定的端口；点击停止监控按钮，即可在指定的服务器上停止监控指定的端口；点击获取监控列表按钮，可以查看当前已经监控的端口<br>
   ![startMonitor](https://github.com/leeyoshinari/performance_monitor/blob/master/master/static/monitor.jpg)
   
   （4）点击结果可视化，可以看到可视化页面；点击画图按钮，即可将指定服务器上的指定端口的监控数据可视化<br>
   ![Visualize](https://github.com/leeyoshinari/performance_monitor/blob/master/master/static/visual.jpg)
   
## 打包
pyinstaller既可以将python脚本打包成Windows环境下的可执行文件，也可以打包成Linux环境下的可执行文件。打包完成后，可快速在其他环境上部署该监控服务，而不需要安装python3.7+环境和第三方包。<br>

pyinstaller安装过程自行百度，下面直接进行打包：<br>

1. 打包server<br>
    (1)安装好python环境，安装第三方包，确保程序可以正常运行；<br>
    (2)进入server文件夹，开始打包：<br>
    ```shell
    pyinstaller -F server.py -p draw_performance.py -p config.py -p Email.py -p logger.py -p process.py -p request.py -p __init__.py --hidden-import draw_performance --hidden-import config --hidden-import logger --hidden-import Email --hidden-import process --hidden-import request
    ```
    `打包过程可能提示缺少一些模块，请按照提示安装对应的模块`<br>
    (3)打包完成后，在当前路径下会生成dist文件夹，进入`dist`即可找到可执行文件`server`;<br>
    (4)将配置文件`config.ini`拷贝到`dist`文件夹下，并修改配置文件；<br>
    (5)将模板文件`templates`和静态文件`static`拷贝到`dist`文件夹下；<br>
    (6)将`dist`整个文件夹拷贝到其他环境，启动server
    ```shell
    nohup ./server &
    ```

2. 打包agent<br>
    (1)安装好python环境，安装第三方包，确保程序可以正常运行；<br>
    (2)进入agent文件夹，开始打包：<br>
    ```shell
    pyinstaller -F server.py -p performance_monitor.py -p logger.py -p config.py -p common.py -p __init__.py --hidden-import logger --hidden-import performance_monitor --hidden-import common --hidden-import config
    ```
    (3)打包完成后，在当前路径下会生成dist文件夹，进入`dist`即可找到可执行文件`server`;<br>
    (4)将配置文件`config.ini`拷贝到`dist`文件夹下，并修改配置文件；<br>
    (5)将`dist`整个文件夹拷贝到其他环境，启动server
    ```shell
    nohup ./server &
    ```
   `由于agent需要在待监控的服务器上运行，在CentOS系统X86架构的服务器上打包完成的可执行文件，只能运行在CentOS系统X86架构的服务器上；其他系统和架构的服务器需要重新打包。`<br>

## 注意
1. 服务器必须支持以下命令：`jstat`、`iostat`、`netstat`，如不支持，请安装。

2. sysstat的版本必须是12+，目前测试过12的版本，其他版本未测试过，使用老版本可能会导致数据异常；最新版本下载地址[请点我](http://sebastien.godard.pagesperso-orange.fr/download.html)

3. 如果你不知道怎么在Linux服务器上安装好Python3.7+，[请点我](https://github.com/leeyoshinari/performance_monitor/wiki/Python-3.7.x-%E5%AE%89%E8%A3%85)。

4. 如需查看最新的操作文档，可在运行程序后，查看教程即可。

5. 统计监控数据时，对监控数据进行排序，使用js排序，默认使用自带的排序算法（冒泡排序）排序，如果觉得慢，可以使用快速排序算法，可在`plot_port.js`和`plot_system.js`中按需修改；快速排序算法可能会导致堆栈溢出。

6. 当前程序几乎可以运行在任何可以运行python的linux系统上，已测试过的系统`CentOS`、`Ubuntu`、`中标麒麟`、`银河麒麟`，支持`X86`和`ARM`架构。
## Requirements
1. aiohttp>=3.6.2
2. aiohttp_jinja2>=1.2.0
3. jinja2>=2.10.1
4. influxdb>=5.2.3
5. requests
6. Python 3.7+
