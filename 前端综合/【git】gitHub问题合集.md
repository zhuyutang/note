1. `remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.`

   - 2021.8.13 过后，需要用用户 token 拉取或提交代码
   - token 设置在：`github-setting-developer settings`
   - 可以放在连接中，避免每次都输入： `git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git`
   - 如果没有输入密码选项，输入`git config --system --unset credential.helper`

2. `OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 10054`
   - https 协议需要部署用 SSL 证书，加密网络传输数据。
   - 关闭 ssl 验证：`git config http.sslVerify "false"`
