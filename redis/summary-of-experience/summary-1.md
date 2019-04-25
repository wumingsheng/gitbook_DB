# redis-log


## redis日志

1. 默认redis是没有日志输出文件的，在配置文件中，日志的配置文件是空字符串，也就是:`logfile ""`，空字符串，默认redis将日志输出到标准输出standard output
2. 注意，如果日志是输出到标准输出里了，并且redis配置了后台进程启动（daemonize yes），日志将被发送到黑洞/dev/null
3. 因此，我们需要手动配置redis的日志输出文件，例如`logfile "/redis/log/redis.log"`,注意，日志所在的文件目录必须存在，不存在手动创建，否则redis无法启动