

# 错误1

使用 GitHubdesktop 在fetch是突然报错，如下：

错误信息：schannel: failed to receive handshake, [SSL](https://so.csdn.net/so/search?q=SSL&spm=1001.2101.3001.7020)/TLS connection failed



解决方式：

在C盘个人用户中找到.gitconfig文件，设置或修改以下内容，然后解决。



```
[http]
    sslbackend = openssl
```

