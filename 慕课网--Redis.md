# 整体介绍

## 系统架构

![image-20210327102303220](image/image-20210327102303220.png)

## Redis故障

![image-20210327102532631](image/image-20210327102532631.png)

![image-20210327102617229](image/image-20210327102617229.png)

![image-20210327102718839](image/image-20210327102718839.png)

![image-20210327102752853](image/image-20210327102752853.png)

![image-20210327102836204](image/image-20210327102836204.png)

![image-20210327102856807](image/image-20210327102856807.png)

## 深层内容

![image-20210327103003431](image/image-20210327103003431.png)

![image-20210327103050104](image/image-20210327103050104.png)

# 基础

![image-20210327103316517](image/image-20210327103316517.png)

![image-20210327103529755](image/image-20210327103529755.png)

![image-20210327103625593](image/image-20210327103625593.png)

![image-20210327105955542](image/image-20210327105955542.png)

持多种数据类型，主从复制与哨兵机制，持久化操作等

![image-20210327110204799](image/image-20210327110204799.png)

![image-20210327110329824](image/image-20210327110329824.png)

![image-20210327110411298](image/image-20210327110411298.png)

![image-20210327110506644](image/image-20210327110506644.png)

![image-20210327110536992](image/image-20210327110536992.png)



![image-20210327110858349](image/image-20210327110858349.png)

![image-20210327111030663](image/image-20210327111030663.png)

![image-20210327110930654](image/image-20210327110930654.png)

***

![image-20210327124225670](image/image-20210327124225670.png)

常用修改配置文件redis.conf	

+ bind 0.0.0.0(实际开发中不能这么设置，需设置指定的IP)
+ requirepass 123456（设置密码）
+ logfile "redis.log" (设置日志文件)



![image-20210327214736430](image/image-20210327214736430.png)

### 基本使用

#### 启动

前台启动

​			windowos下：redis-server redis.windows.conf

​			Linux下：进入redis安装路径（/usr/local/redis/Red…）   ./src/redis-server redis.conf

守护进程方式启动 (linux下）

​			修改配置文件redis.conf  将daemonize no 改为 yes

​			启动服务器： redis-server redis.conf

​					查看运行状态 netstat -tulpn(看端口号6379）

​			启动客户端：./src/redis-cli [-h ip] [-p 端口号] -a 密码 

​			退出：exit

#### redis层级存储（对应mysql中的表）

![image-20210327224932640](image/image-20210327224932640.png)                       ![image-20210327224956097](image/image-20210327224956097.png)

#### 常用命令

info   显示信息

FLASHALL  清除所有Key

#### 数据类型

![image-20210329205141247](image/image-20210329205141247.png)

![image-20210329205226130](image/image-20210329205226130.png)

![image-20210329205249334](image/image-20210329205249334.png)

![image-20210329205323673](image/image-20210329205323673.png)

![image-20210329205347772](image/image-20210329205347772.png)

![image-20210329205502295](image/image-20210329205502295.png)







## SpringBoot集成Redis

![image-20210327230830965](image/image-20210327230830965.png)

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
           <!-- 使用lettuce连接池时导入-->
<!--        <dependency>-->
<!--            <groupId>org.apache.commons</groupId>-->
<!--            <artifactId>commons-pool2</artifactId>-->
<!--        </dependency>-->
```

```
  # Redis
  redis:
    port: 6379
    host: 192.168.10.101
    timeout: 3000
    password: 123456
    #配置jedis连接池
#    jedis:
#      pool:
#        max-active: 8
#        max-idle: 8
#        min-idle: 0
#        max-wait: 1000
    #配置lettuce连接池
#    lettuce:
#      pool:
#        max-active: 8
#        max-idle: 8
#        min-idle: 0
#        max-wait: 1000
```

## 项目1

![image-20210327235356581](image/image-20210327235356581.png)

![image-20210327235423859](image/image-20210327235423859.png)

![image-20210327235536859](image/image-20210327235536859.png)

![image-20210327235631275](image/image-20210327235631275.png)

![image-20210327235740668](image/image-20210327235740668.png)

### 登录模块

![image-20210329205641923](image/image-20210329205641923.png)

### 认证授权

![image-20210330055916806](image/image-20210330055916806.png)



![image-20210330062550149](image/image-20210330062550149.png)

## 源码