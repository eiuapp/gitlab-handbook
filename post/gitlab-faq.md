
## 取消Default to Auto DevOps pipeline for all projects的选择框

https://blog.csdn.net/weixin_44723434/article/details/88425094

## mkdir: cannot create directory ‘/home/ubuntu/gitlabRunner/builds’: Permission denied

在一次机器重启后, 发现, 重启gitlab-runner后, 需要一个 `gitlabRunner/builds` 的文件夹, 所以我就手动创建了.

`sudo mkdir -p /home/ubuntu/gitlabRunner/builds`

然后,在运行 gitlab-runner 时,日志中会有如下错误:

```
Running with gitlab-runner 10.5.0 (10.5.0)
  on api-deploy-runner obhhKvSY
Using Shell executor...
Running on utuntu...
mkdir: cannot create directory ‘/home/ubuntu/gitlabRunner/builds’: Permission denied
ERROR: Job failed: exit status 1
```

这时, `sudo cat /etc/passwd` 会发现 gitlab-runner 用户, 所以, 这里要修改 文件夹的 用户属性.

`sudo chown -R gitlab-runner:gitlab-runner /home/ubuntu/gitlabRunner/`

问题解决了.
