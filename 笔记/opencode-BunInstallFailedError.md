#BunInstallFailedError
# 现象
1. windows11 **普通用户**终端运行opencode时报错：
```
$ opencode
INFO  2026-01-26T09:11:20 +105ms service=models.dev file={} refreshing
{
  "name": "BunInstallFailedError",
  "data": {
    "pkg": "opencode-antigravity-auth",
    "version": "latest"
  }
}

```


2. **管理员身份**运行终端却可以正常运行


# 解决办法

打开node.exe所在文件夹允许完全控制以提高普通用户运行权限


![[3.png]]

