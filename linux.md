##### 查看进程

ps aux

ps -ef | grep nginx

pgrep redis-server	只显示PID



tar -xvf redis.tar         解开一个tar





##### kill -HUP PID 重启服务

##### 后台启动jar包

nohup java -jar  XXXX.jar  &

##### 占用内存前10

ps -aux | sort -k4nr | head -n 10

##### 查看端口，关闭进程

netstat -tln | grep 9080

sudo lsof -i:9080

```bash
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    5062 root   31u  IPv6 137945      0t0  TCP *:glrpc (LISTEN)
java    5062 root   41u  IPv6 136993      0t0  TCP yikeduo-dev:glrpc->yikeduo-dev:56466 (CLOSE_WAIT)
java    5062 root   42u  IPv6 138378      0t0  TCP yikeduo-dev:glrpc->192.168.10.112:47848 (CLOSE_WAIT)
```

sudo kill -9 5062



docker 密码

192.168.10.202 root  dianvo123/*-+



> 413 Request Entity Too Large

nginx **client_max_body_siz**设置太小

**nginx -t**命令测试配置文件修改后的语法是否正确

**nginx -s reload** 重启让配置文件生效

##### 当前时间

date

##### Git

git reset HEAD^ 是将咱暂存区和HEAD的提交保持一致

git reset **--hard** HEAD^ 是将**工作区**、暂存区和HEAD保持一致







18181096189

1234567a