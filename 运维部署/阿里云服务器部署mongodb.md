在阿里云上买了个服务器，部署 mongodb 遇到一些坑，解决办法也是从网上搜集而来，把零零碎碎的整理记录一下。

服务器是：Alibaba Cloud Linux

## 下载安装

mongodb 官网下载实在是太慢，可以从阿里镜像安装：[阿里 MongoDb 镜像](https://developer.aliyun.com/mirror/mongodb?spm=a2c6h.13651102.0.0.25581b115JRT4F)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c98d66ee7aea4a34938fe6d9a31fbb9d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

**使用 yum 安装**

- 在/etc/yum.repos.d 目录下添加 mongodb-org.repo 文件
  ```bash
  cd /etc/yum.repos.d
  vim mongodb-org.repo

  [mogodb-org]
  name=MongoDB Repository
  baseurl=http://mirrors.aliyun.com/mongodb/yum/redhat/7Server/mongodb-org/4.0/x86_64/
  gpgcheck=0
  enabled=1
  ```
  vim 命令：是编辑（新建）文件的命令，退出编辑的时候，按`esc`,然后输入 `:wq`退出报存。如果对 linux 命令不熟，用 Xftp 等工具直接上传也可以。
  baseurl：在阿里镜像中，点击`下载地址`后，选择的 mongodb 的版本的链接，根据选择的版本不同而不同，其他的不用变。
- 用 yum 安装
  ```bash
  yum -y install mongodb-org
  ```
  yum：linux 下载包的命令，从上面添加的 .repo 文件中的 baseurl 地址开始下载。`-y`是为了免去安装的确认操作。

## 配置

安装完成了过后，找到配置 mongodb 的配置文件

```bash
rpm -qla | grep mongod.conf
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/1aa908590e6b48da83e21e3495c220b9.png)

- 修改配置文件
  ```bash
  vim /etc/mongod.conf

  # mongod.conf
  systemLog:
    destination: file
    logAppend: true
    path: /var/log/mongodb/mongod.log # 日志文件目录

  # Where and how to store data.
  storage:
    dbPath: /var/lib/mongo # 数据目录
    journal:
      enabled: true
  #  engine:
  #  mmapv1:
  #  wiredTiger:

  # how the process runs
  processManagement:
    fork: true  # fork and run in background
    pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile
    timeZoneInfo: /usr/share/zoneinfo

  # network interfaces
  net:
    port: 27017 #端口
    # 修改ip
    bindIp: 0.0.0.0  # 这里默认是127.0.0.1，要改成0.0.0.0
  ```
  主要修改点：bindIp 值改为 0.0.0.0,这样可以外网访问
  如果对 linux 的指令不熟悉的，可以直接在 Xftp 的工具里面选中文件，右键有编辑操作

## 运行

- 在`/etc/init.d`文件夹中添加开机启动脚本`mongod`
  ```bash
  cd /etc/init.d
  vim mongod

  EXEC=/usr/bin/mongod
  CONF=/etc/mongod.conf
  LOCKFILE=/var/lock/subsys/mongod
  RETVAL=0
  case "$1" in
      start)
          echo -n $"Starting mongod: "
          $EXEC -f $CONF
          RETVAL=$?
          echo
          [ $RETVAL -eq 0 ] && touch $LOCKFILE
          ;;
      stop)
          echo -n $"Stopping mongod: "
          $EXEC -f $CONF --shutdown
          RETVAL=$?
          echo
          [ $RETVAL -eq 0 ] && rm -f $LOCKFILE
          ;;
      restart)
          ${0} stop
          ${0} start
          ;;
      *)
          echo "Usage: /etc/init.d/mongod {start|stop|restart}" >&2
          exit 1
  esac
  ```
- 运行权限

  ```bash
  # 获取文件权限
  chmod +x /etc/init.d/mongodb
  ```

- 启动
  ```bash
  service mongod start
  ```
- 停止
  ```bash
  service mongod stop
  ```
- 重启
  ```bash
  service mongod restart
  ```
- 卸载
  ```bash
  # 停止服务
  service mongod stop
  # 删除安装的包
  yum erase $(rpm -qa | grep mongodb-org)
  # 删除数据和日志
  rm -rf /var/log/mongodb
  rm -rf /var/lib/mongo
  ```

## 验证

- 开启端口
  mongodb 默认端口为：27017，在配置文件`/etc/mongod.conf`里可以查看
  在阿里云后台，防火墙打开端口 ![在这里插入图片描述](https://img-blog.csdnimg.cn/0740a75415134822940166b3c2b6d846.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
- 浏览器访问
  ip:27017 访问
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/39b27f942db044feb80e35063ac802c7.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

## 问题排查

- 异常关闭时，手动关闭进程
  如果启动失败，可能是由于系统异常关闭，可能会有进程没有退出

  ```bash
  #查看mongodb的进程
  ps aux | grep mongod

  #根据查询的结果pid，关闭进程
  kill -9 pid
  ```

  mongod.lock 和 diagnostic.data 文件删掉

- 自启动失效
  如果 mongodb 自启动，查看配置文件`/etc/mongod.conf`中 fork 配置是否有开启：![在这里插入图片描述](https://img-blog.csdnimg.cn/3d4002e9405e4fa69e5c44ca6e2766cc.png)

**参考：**
[CentOS7 使用阿里镜像安装 mongodb4.0](https://blog.csdn.net/qq_35433926/article/details/89680721)
[mongodb 启动异常](https://blog.csdn.net/weixin_43842451/article/details/106584027)
