# 个人用 lnmp 环境


> docker-compose up -d php nginx mysql redis mongo

> ###supervisor 的使用###

```
//进入php容器内
docker-compose exec php bash
//启动某个进程(program_name=你配置中写的程序名称)
supervisorctl start program_name
//查看正在守候的进程
supervisorctl
//停止某一进程 (program_name=你配置中写的程序名称)
supervisorctl stop program_name
//重启某一进程 (program_name=你配置中写的程序名称)
supervisorctl restart program_name
//停止全部进程
supervisorctl stop all
```