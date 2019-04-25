# 运行状态判断

## redis启动状态可以通过一下方式快速判断

1. `pidof redis-server`，如果显示进程，redis服务端正在运行
2. 如果知道端口，可以通过`ss -lunt`判断
3. 通过进程`ps -ef |grep redis`
4. 通过redis-cli连接