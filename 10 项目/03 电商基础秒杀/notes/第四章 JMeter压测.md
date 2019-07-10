

# JMeter入门

压测商品列表

测试计划右键添加线程组

![1561878316206](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561878316206.png)

Ramp-up启动10个线程的时间，0为同时启动

线程组->添加->配置元件->Http请求默认值

![1561878535889](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561878535889.png)

线程组->添加->取样器->Http请求

![1561879048869](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561879048869.png)

设置路径

![1561879190971](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561879190971.png)

查看结果，添加聚合报告

![1561879234768](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561879234768.png)

保存jmx到桌面，点击运行![1561879377648](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561879377648.png)

10个请求的结果，qps17.7

![1561879412055](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561879412055.png)

线程数 1000 ，qps84.9

![1561879569228](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561879569228.png)

为什么低，因为访问数据库，性能的瓶颈

TOP命令 监控服务器CPU、内存

# 自定义变量模拟多用户

新建HTTP请求，压测带参数的/user/info

![1561879838300](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561879838300.png)

浏览器F12，立即秒杀查看token，添加到参数值中

![1561879950465](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561879950465.png)

qps达到300

获取用户信息，调用getByToken

![1561880128876](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880128876.png)

![1561880169765](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880169765.png)

因为从redis中获取缓存，所以QPS高

商品的压测不仅读取了缓存，还读了数据库，所以qps低

但是这样的测试不准确，因为用的都是同一个token

csv数据文件设置

![1561880321323](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880321323.png)

![1561880390620](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880390620.png)

常规的压测数据

![1561880420175](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880420175.png)

配置设置

![1561880481484](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880481484.png)

引用配置文件的变量$(userToken)

![1561880549191](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880549191.png)

# Redis压测工具redis-benchmark

![1561880663479](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880663479.png)

ps -ef | grep redis 启动redis

1.6s处理100000个请求，qps60000多

![1561880750657](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880750657.png)

redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000

![1561880838293](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880838293.png)

redis-benchmark -t set,lpush -q -n 100000  //只测试set和lpush

![1561880884501](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880884501.png)

redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')" 只测试这个命令

![1561880999169](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561880999169.png)

# Spring Boot打war包，放到Tomcat运行

添加spring-boot-starter-tomcat的依赖，provided表示运行时不需要

![1561881199366](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561881199366.png)

添加maven-war-lugin插件，在build下

![1561881258863](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561881258863.png)

修改启动类，重写SpringApplicationBuilder方法

![1561881319272](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561881319272.png)

![1561881469314](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561881469314.png)

cmd切换到Location 文件夹下，mvn clean package命令

![1561881557257](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561881557257.png)

打完到target下

![1561881611253](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561881611253.png)

把war包复制到TOMCAT/webapps下，启动Tomcat

![1561881414786](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561881414786.png)

打开浏览器，注意路径加上miaosha

![1561881688676](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561881688676.png)

# JMeter命令行使用

把war包改成jar包，删掉依赖，启动类改回去

![1561882104378](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882104378.png)

![1561882161054](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882161054.png)jar包上传到linux

![1561882256614](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882256614.png)

保存jmx文件，到linux

压测 ：sh jmeter.sh -n -t XXX.jmx -l result.jtl

下载结果 ：sz result.jtl qps600多

![1561882423076](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882423076.png)

查询商品，qps1200多 5000个并发，循环10次

![1561882542546](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882542546.png)

压测秒杀

![1561882616275](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882616275.png)

UserUtil生产5000个用户，并获得用户的token

![1561882753799](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882753799.png)

调用接口 ，拿到token的值，写入到文件中

![1561882780194](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882780194.png)

csv配置文件 ，token.txt  userid,token

![1561882861413](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882861413.png)

![1561882951498](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1561882951498.png)

sh jmeter.sh -n -t miaosha.jmx -l result.jtl

QPS 1306，卖超了3个