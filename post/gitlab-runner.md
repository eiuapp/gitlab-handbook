gitlab-runner安装与使用

## env ##

- env-1 tl-lan-server-a

## step ##

```bash
sudo docker run -d --name gitlab-runner --restart always -v /var/run/docker.sock:/var/run/docker.sock -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner:ubuntu-v11.11.1
ubuntu@utuntu:~/lcnx/aliyun/tomcat-data$ docker ps
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS                  PORTS                                                               NAMES
640f0e0c3642        gitlab/gitlab-runner:ubuntu-v11.11.1   "/usr/bin/dumb-init …"   27 hours ago        Up 27 hours                                                                                 gitlab-runner
b4c4ccdd3baa        gitlab/gitlab-ce:11.9.1-ce.0           "/assets/wrapper"        33 hours ago        Up 33 hours (healthy)   0.0.0.0:22->22/tcp, 0.0.0.0:12280->80/tcp, 0.0.0.0:12443->443/tcp   gitlab-ce-11.9.1-2
```

- http://walterinsh.github.io/2016/04/18/using-gitlab-ci.html
- https://www.cnblogs.com/xishuai/p/ubuntu-install-gitlab-runner-with-docker.html

```bash
ubuntu@utuntu:~/lcnx/aliyun/tomcat-data$ docker exec -it gitlab-runner gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=121 revision=5a147c92 version=11.11.1
Running in system-mode.

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://192.168.168.137:12280/
Please enter the gitlab-ci token for this runner:
5nnXzbP4cYe3qPUvgJ_z
Please enter the gitlab-ci description for this runner:
[640f0e0c3642]: my-runner
Please enter the gitlab-ci tags for this runner (comma separated):
my-tag
Registering runner... succeeded                     runner=5nnXzbP4
Please enter the executor: docker-windows, docker-ssh, shell, virtualbox, docker-ssh+machine, docker, ssh, docker+machine, kubernetes, parallels:
docker
Please enter the default Docker image (e.g. ruby:2.1):
node:11.15.0
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
ubuntu@utuntu:~/lcnx/aliyun/tomcat-data$
```


## 验证 ##

chrome: http://192.168.168.137:12280/admin/runners 应该能看到 gitlab runners 就是成功了.

## 查看 version, status ##

```bash
ubuntu@utuntu:~/lcnx/local$ docker exec -it gitlab-runner gitlab-runner --version
Version:      11.11.1
Git revision: 5a147c92
Git branch:   11-11-stable
GO version:   go1.8.7
Built:        2019-05-24T12:32:58+0000
OS/Arch:      linux/amd64
ubuntu@utuntu:~/lcnx/local$ docker exec -it gitlab-runner gitlab-runner status
Runtime platform                                    arch=amd64 os=linux pid=165 revision=5a147c92 version=11.11.1
gitlab-runner: Service is not running.
ubuntu@utuntu:~/lcnx/local$
```

## 使用 ##

### error: `port 80: Connection refused` ###

报错如下:

```
Running with gitlab-runner 11.11.1 (5a147c92)
  on my-runner kj6xC-zB
Using Docker executor with image node:latest ...
Pulling docker image node:latest ...
Using docker image sha256:6be2fabd419658c51f44d7c2a1a4a6fd172125c11e718a7eb989e74b8b2f9e5a for node:latest ...
Running on runner-kj6xC-zB-project-2-concurrent-0 via 640f0e0c3642...
Reinitialized existing Git repository in /builds/FED/lvchuang-web/.git/
Fetching changes...
fatal: unable to access 'http://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@192.168.168.137/FED/lvchuang-web.git/': Failed to connect to 192.168.168.137 port 80: Connection refused
ERROR: Job failed: exit code 1
```

https://gitlab.com/gitlab-org/gitlab-runner/issues/3091

因为这里,我们的 gitlab url 是 192.168.168.137:12080, 但是这里报的是 80: Connection refused 所以先尝试解决这个 port 不一致的问题.

我这里,先简单处理, 把gitlab dokcer 直接使用 80 端口.

然后,发现,可以使用了.

### pending ###

```
This job is stuck because you don't have any active runners online with any of these tags assigned to them: my-tag
Go to Runners page
```

因为我的配置与官网是相同的,还是要回到官网来看一下日志

https://docs.gitlab.com/runner/install/docker.html#reading-gitlab-runner-logs

For example, if GitLab Runner was started with the following command:

```bash
docker run -d --name gitlab-runner --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
```
you may get the logs with:

```bash
docker logs gitlab-runner
```

输出信息:

```
WARNING: Checking for jobs... failed                runner=VE3S_jfp status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: dial tcp 192.168.168.137:80: i/o timeout
WARNING: Checking for jobs... failed                runner=VE3S_jfp status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: dial tcp 192.168.168.137:80: i/o timeout
WARNING: Checking for jobs... failed                runner=VE3S_jfp status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: dial tcp 192.168.168.137:80: i/o timeout
```
说明是 container 到达不了 192.168.168.137:80, 那么, 先检查一下 防火墙. 果然,就是这个问题.

```bash
sudo ufw allow from 172.17.0.1/16  to any port 80
```

重试.
成功.

### gitlab runner permission deny ###

可能是:
- 电脑重启了
- docker 的数据盘放在了其它硬盘

导致:

```shell
root@utuntu:/home/ubuntu/fangyuan/local/fs.delete.20191230# docker ps | grep gitlab-runner
65fa431b0ec1        gitlab/gitlab-runner:ubuntu-v11.11.1   "/usr/bin/dumb-init …"   6 months ago        Up 6 hours                                                                              gitlab-runner-shell
be6755a2746c        gitlab/gitlab-runner:ubuntu-v11.11.1   "/usr/bin/dumb-init …"   7 months ago        Up 6 hours                                                                              gitlab-runner
root@utuntu:/home/ubuntu/fangyuan/local/fs.delete.20191230# docker logs -f gitlab-runner
...
WARNING: Checking for jobs... failed                runner=VE3S_jfp status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: read tcp 172.17.0.3:59338->192.168.168.137:80: read: connection reset by peer
WARNING: Checking for jobs... failed                runner=2K7oPJtf status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: read tcp 172.17.0.3:59346->192.168.168.137:80: read: connection reset by peer
WARNING: Checking for jobs... failed                runner=VE3S_jfp status=502 Bad Gateway
WARNING: Checking for jobs... failed                runner=2K7oPJtf status=502 Bad Gateway
```

jobs 中会显示对 某个文件夹 `permission deny`, 如:

```
Running with gitlab-runner 10.5.0 (10.5.0)
  on api-deploy-runner obhhKvSY
Using Shell executor...
Running on utuntu...
Fetching changes...
HEAD is now at 8bd0543 fix pms_grant_option_stage_detail saveNewBulk
From http://192.168.168.137/fangyuan/fangyuan-server
   8bd0543..749e544  dev        -> origin/dev
Checking out 749e5449 as dev...
Skipping Git submodules setup
$ echo "hello fangyuan-server"
hello fangyuan-server
$ echo "deploy_test"
deploy_test
$ echo $PWD && ls ~
/home/ubuntu/gitlabRunner/builds/obhhKvSY/0/fangyuan/fangyuan-server
logs
$ bash /home/ubuntu/lcnx/local/scripts/fangyuan-server/deploy/deploy-test.sh
Mon Dec 30 15:08:24 CST 2019
20191230150824
fatal: Unable to create '/home/ubuntu/fangyuan/local/fangyuan-server-run-start/.git/index.lock': Permission denied
Cannot save the current index state
error: cannot open .git/FETCH_HEAD: Permission denied
cp: cannot create regular file 'config/config.prod.js': Permission denied
...
...
...
npm ERR! A complete log of this run can be found in:
npm ERR!     /var/lib/gitlab-runner/.npm/_logs/2019-12-30T07_08_29_657Z-debug.log
docker
Job succeeded
```

但是,后来,又时不时 没有这个 `permission denied`.

##

## 说明 ##

### install example ###

#### install gitlab-ci

```bash
sudo docker run --detach \
  --restart always \
  --hostname 192.168.168.137 \
  --publish 12443:443 --publish 80:80 --publish 22:22 \
  --name gitlab-ce-11.9.1-2 \
  --volume /srv/gitlab9.1/config:/etc/gitlab \
  --volume /srv/gitlab9.1/logs:/var/log/gitlab \
  --volume /srv/gitlab9.1/data:/var/opt/gitlab \
  gitlab/gitlab-ce:11.9.1-ce.0
```

#### gitlab-runner-docker ####

```bash
sudo docker run -d --name gitlab-runner --restart always -v /var/run/docker.sock:/var/run/docker.sock -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner:ubuntu-v11.11.1
```

#### gitlab-runner-shell ####

```bash
sudo docker run -d --name gitlab-runner-shell --restart always -v /var/run/docker.sock:/var/run/docker.sock -v /srv/gitlab-runner-ubuntu-v11/shell/config:/etc/gitlab-runner gitlab/gitlab-runner:ubuntu-v11.11.1
```

##### runner token
启动后看一下

```bash
root@utuntu:/srv/gitlab-runner-ubuntu-v11/shell/config# cat config.toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "shell-runner"
  url = "http://192.168.168.137/"
  token = "S-j8EMnmzTbxRNPyCpad"
  executor = "shell"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
root@utuntu:/srv/gitlab-runner-ubuntu-v11/shell/config#
```

上面的这个token值`S-j8EMnmzTbxRNPyCpad`会显示在`docker logs -f gitlab-runner-shell`的输出中.
