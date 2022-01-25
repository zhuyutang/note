服务器：
阿里云 Alibaba Cloud Linux

## 下载

- 进入到预计存放 nginx 的目录，比如：`/usr/local/`
- 下载 nginx 压缩包，并解压
  ```bash
  cd /usr/local
  wget http://nginx.org/download/nginx-1.18.0.tar.gz
  tar -zxvf nginx-1.18.0.tar.gz
  ```

## 安装

- 进入到解压的文件夹并安装
  ```bash
  cd /usr/local/nginx-1.18.0
  ./configure
  make -j2
  make install
  ```
  安装完成后，会有`nginx`文件夹

## 配置

配置文件位置：`/usr/local/nginx/conf/nginx.conf`
其他的不用变，主要看`server`：

```javascript
  server {
        listen       80; # 端口
        server_name  localhost;
        index  index.html index.htm; # 默认文件，会在索引文件夹下，寻找index配置的文件

		##	alias 不包含url的路径
		## 比如url：http://ip/aaa
		## 服务器索引的文件是 /home/www/dist/index.html

        location /aaa/ {
            alias   /home/www/dist/;
        }

		##	root 包含url的路径
		## 比如url:http://ip/aaa
		## 服务器索引的文件是 /home/www/dist/aaa/index.html

        location /aaa/ {
            root /home/www/love/dist/;
        }
    }
```

## 运行

- 进入 nginx 命令目录：`/usr/local/nginx/sbin/`
- 运行

  ```bash
  cd /usr/local/nginx/sbin/
  ./nginx
  ```

- 重启
  ```bash
  cd /usr/local/nginx/sbin/
  ./nginx -s reload
  ```
- 查看 nginx 进程
  `bash ps -ef|grep nginx `
  如果显示如下，则表示启动成功：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/c6c181a0ff614f8e9e5cce3589f89d3f.png)

## 问题排查

- 查看日志文件
  打开日志的配置：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20117d5f633544d882afe5a033ea5b7f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_15,color_FFFFFF,t_70,g_se,x_16)
  日志文件位置：`/usr/local/nginx/logs`
  其中`nginx.pid`文件记录的是 nginx 进程的 pid，不要删除，否则`nginx -s reload`等需要杀进程的命令会报错。

- 资源文件路径 assets 找不到
  默认情况下，打包后的代码中，index.html 引入的文件是绝对路径：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/f58a67cd2a494e038386a8e2958c5839.png)
  如果 nginx 没有配置 assets 的 location，是找不到资源的。
  就算配了 location，如果一个服务器上部署了多个前端项目，就会存在多个 assets，那就要把不同项目的资源文件夹名称都改成不一样的，再把每一个文件路径配上 location。这样麻烦，干脆把把这里的绝对路径改为相对路径：
  webpack 打包：修改 `assetPublicPath：'./'`
  vite 打包：修改`base:'./'`
