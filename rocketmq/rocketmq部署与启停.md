# NameServer 启动
``` bash
nohup $ROCKETMQ_ROOT/bin/mqnamesrv 2>&1 >/dev/null &
```
# Broker 启动

## 单机启动
``` bash
nohup $ROCKETMQ_ROOT/bin/mqbroker -n [[$NAME_SERVER_ADDR:$NAME_SERVER_PORT],] 2>&1 >/dev/null &
```
## 指定配置文件启动（通常用于主从/集群场景）
```bash
nohup $ROCKETMQ_ROOT/bin/mqbroker -c $BROKER_CONFIG_FILE 2>&1 >/dev/null &
```
# 启动参数配置

如果机器的可用内存不足，需要分别修改`bin`目录下的`runserver.sh`和`runbroker.sh`文件，对如下配置进行酌情降配（注意上下文中有个 java 版本判断，需要结合本机 java 版本修改对应的逻辑分支）：
```bash
# runserver.sh
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

# runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g"
```

## 针对`JAVA_VERSION >= 9`的特别配置
[[rocketmq broker 启动报错#broker 启动报错]]

# Broker 停止
``` bash
$ROCKETMQ_ROOT/bin/mqshutdown broker
```
# NameServer 停止
``` bash
$ROCKETMQ_ROOT/bin/mqshutdown namesrv
```