gitlab cicd + node project


https://segmentfault.com/a/1190000017174825

https://docs.gitlab.com/ee/ci/ssh_keys/README.html#ssh-keys-when-using-the-docker-executor

## 前端部分的 gitlab-ci.yaml ##

https://docs.gitlab.com/ce/ci/yaml/#before_script-and-after_script

`before_script` 的内容有调整, 因为, 我们只需要在 `deploy_test` 和 `deploy_production` 的时候, 运行就可以了.

然后,因为 `deploy_test` 和 `deploy_production` 的机器不同, 所以,设置了2份`SSH_KNOWN_HOSTS` 和 `SSH_PRIVATE_KEY`

原理: 把一份能够完成免密 scp 或 ssh 的机器密钥与对应的密钥指纹放到 gitlab JOB启动的 gitlab-runner container中, 让 gitlab-runner container 来完成这个事情.

**注意** 这里配置的 `SSH_KNOWN_HOSTS` 和 `SSH_PRIVATE_KEY` 是 gitlab project 的 Maintainer 或 以上权限的所有者可查. 也就是意味着, scp,ssh对应的机器,会有权被其控制.

### `SSH_KNOWN_HOSTS` 的来源 ###

https://liam.page/2018/01/24/ssh-keyscan/

可见, ssh-keyscan 是来获取 密钥指纹的.

`ssh-keyscan IP`

如果是多个IP, 则写进文件里.

```bash
ubuntu@utuntu:~$ cat hosts.txt 
192.168.168.137
192.168.168.138
120.77.***.***
ubuntu@utuntu:~$ ssh-keyscan -f hosts.txt 
```


### `SSH_PRIVATE_KEY` ###

`cat ~/.ssh/id_rsa` 

具体地,我的 .gitlab-ci.yml 文件如下:



## 后端部分 gitlab ci ##

### 后端使用 docker 直接在 host 上启用服务 ###

#### docker 服务单独启动能正常运行 ####

##### Dockerfile #####

这里的 Dockerfile 参考了

- https://pm2.io/doc/en/runtime/integration/docker/
- https://www.jianshu.com/p/ab76ba86eafc

```
FROM keymetrics/pm2:latest-alpine

# Bundle APP files
RUN mkdir -p /home/service
WORKDIR /home/service

COPY . /home/service
# COPY src src/
# COPY package.json .
# COPY ecosystem.config.js .

# Install app dependencies
ENV NPM_CONFIG_LOGLEVEL warn
RUN npm install

# Expose the listening port of your app
EXPOSE 3011

# Show current folder structure in logs
RUN ls -al -R

CMD [ "pm2-runtime", "start", "ecosystem.config.js", "--only", "server-dev" ]
```

##### 制作docker image #####

```bash
docker build -t lvchuang-server-test4 .
```

##### docker run #####

```bash
docker run -d --name lvchuang-server -p 2011:3011 lvchuang-server-test4
```

##### 验证 #####

chrome: http://192.168.168.137:2011/GetStage

#### host 安装 gitlab-ci-multi-runner ####

```bash
sudo apt-get install gitlab-ci-multi-runner
```


#### 把 gitlab-runner 用户加入 docker 组 ####

```bash
sudo usermod -aG docker gitlab-runner
```

#### gitlab-runner register ####

```bash
gitlab-ci-multi-runner register
```

注册过程中,executor 选择 shell. 具体完成后, 可见 config.toml 文件.

```
ubuntu@utuntu:~/lcnx/local/lvchuang-server$ cat ~/.gitlab-runner/config.toml 
concurrent = 1
check_interval = 0

[[runners]]
  name = "api-deploy-runner"
  url = "http://192.168.168.137/"
  token = "obhhKvSY4s5M1pGp49mf"
  executor = "shell"
  [runners.cache]

```

#### gitlab-runner 验证 ####

```bash
gitlab-ci-multi-runner register
gitlab-ci-multi-runner --help
sudo gitlab-ci-multi-runner status
gitlab-ci-multi-runner verify
gitlab-ci-multi-runner list
```

具体效果看一下

```bash
ubuntu@utuntu:~/lcnx/local/lvchuang-server$ !1321
sudo gitlab-ci-multi-runner status
[sudo] password for ubuntu: 
gitlab-runner: Service is running!
ubuntu@utuntu:~/lcnx/local/lvchuang-server$ gitlab-ci-multi-runner verify
WARNING: Running in user-mode.                     
WARNING: The user-mode requires you to manually start builds processing: 
WARNING: $ gitlab-runner run                       
WARNING: Use sudo for system-mode:                 
WARNING: $ sudo gitlab-runner...                   
                                                   
Verifying runner... is alive                        runner=obhhKvSY
Verifying runner... is alive                        runner=k7U2n9Sb
ubuntu@utuntu:~/lcnx/local/lvchuang-server$ gitlab-ci-multi-runner list
Listing configured runners                          ConfigFile=/home/ubuntu/.gitlab-runner/config.toml
api-deploy-runner                                   Executor=shell Token=obhhKvSY4s5M1pGp49mf URL=http://192.168.168.137/
ssh-multi-runner                                    Executor=ssh Token=k7U2n9Sb8ErrXa5nzLDr URL=http://192.168.168.137/
ubuntu@utuntu:~/lcnx/local/lvchuang-server$ 
```

#### .gitlab-ci.yml ####

tag 写之前 gitlab-runner register 时填写的对应.

**注意** 下的script中,写的就是 docker 相关命令

```
image: node:11.15.0

stages:
  - deploy_test

cache:
  key: ${CI_BUILD_REF_NAME}
  paths:
    - node_modules/

before_script:
  - export PATH=/usr/local/bin:$PATH

# 部署测试服务器
deploy_test:
  stage: deploy_test
  tags:
    - node-api-deploy-test
  only:
    - dev
    - master
  script:
    - 'echo "deploy_test"'
    - 'echo $PWD && ls ~'
    - docker cp ./ lvchuang-server:/home/service
    - docker restart lvchuang-server
```

#### git push ####

更新代码, git push 至 repo 

#### 观察 gitalb.com ####

选择好 runner 
从 gitlab page: http://192.168.168.137 中看到, pipeline 下的 job 是 pending 状态的. 也就是意味着, 没启动.

#### gitlab-ci-multi-runner run ####

```
ubuntu@utuntu:~/lcnx/local/lvchuang-server$ gitlab-ci-multi-runner run
Starting multi-runner from /home/ubuntu/.gitlab-runner/config.toml ...  builds=0
WARNING: Running in user-mode.                     
WARNING: Use sudo for system-mode:                 
WARNING: $ sudo gitlab-runner...                   
                                                   
Configuration loaded                                builds=0
Metrics server disabled                            
Checking for jobs... received                       job=302 repo_url=http://192.168.168.137/zengyunlong/test4.git runner=obhhKvSY
Job succeeded                                       job=302 project=12 runner=obhhKvSY
Checking for jobs... received                       job=303 repo_url=http://192.168.168.137/zengyunlong/test4.git runner=obhhKvSY
Job succeeded                                       job=303 project=12 runner=obhhKvSY
^CAll workers stopped. Can exit now                   builds=0
WARNING: Requested service stop: interrupt          builds=0
ubuntu@utuntu:~/lcnx/local/lvchuang-server$ 
```

相应地, 看到 pending 已经开始 running 了.

具体日志如下:

```
Running with gitlab-runner 10.5.0 (10.5.0)
  on api-deploy-runner obhhKvSY
Using Shell executor...
Running on utuntu...
Fetching changes...
HEAD is now at c09fdc9 fix
From http://192.168.168.137/zengyunlong/test4
   c09fdc9..10f12d7  master     -> origin/master
Checking out 10f12d72 as master...
Skipping Git submodules setup
Checking cache for master...
Successfully extracted cache
$ export PATH=/usr/local/bin:$PATH
$ echo "deploy_test"
deploy_test
$ echo $PWD && ls ~
/home/ubuntu/lcnx/local/lvchuang-server/builds/obhhKvSY/0/zengyunlong/test4
default.conf
docker
env
hosts.txt
index.html
lcnx
test-dev
$ docker cp ./ lvchuang-server:/home/service
$ docker restart lvchuang-server
lvchuang-server
Creating cache master...
WARNING: node_modules/: no matching files          
Archive is up to date!                             
Created cache
Job succeeded
```

从日志里看出, 当 executor 为 shell 时, 实际上, gitlab-runner 是在host上执行命令的. 
所以, 在 .gitlab-ci.yml 中运行的命令,实际上就是以 gitlab-runner 这个用户在host上面运行.

#### gitlab-runner 如何自启动 ####

因为上面的运行, 需要, 手动地在命令行下执行 ``, 所以我们需要更进一步,让系统自动完成,这个操作.

参考:

- https://www.jianshu.com/p/2b43151fb92e
- https://www.cnblogs.com/jiukun/p/7481287.html (特别是 gitlab-ci-multi-runner run, install, start, stop, restart 之间的关系介绍) 
- https://blog.csdn.net/xl_lx/article/details/78329019
- https://www.cnblogs.com/cnundefined/p/7095368.html

##### install 包装 #####

```bash
ubuntu@utuntu:~/lcnx/local/test/test4$ sudo gitlab-ci-multi-runner install  -n  "jiukun_self_runner_2"  -d "/home/ubuntu/lcnx/local/test/test4"  -c "/home/ubuntu/.gitlab-runner/config.toml" -u ubuntu
```

执行上述安装命令时，gitlab-ci-multi-runner后台实际是将run命令写入/etc/systemd/system/jiukun_self_runner_2.service文件，使之成为一个单独的服务

##### 启动服务 #####

```bash
ubuntu@utuntu:~/lcnx/local/test/test4$ gitlab-ci-multi-runner start -n jiukun_self_runner_2
FATAL: Please run the commands as root             
ubuntu@utuntu:~/lcnx/local/test/test4$ sudo gitlab-ci-multi-runner start -n jiukun_self_runner_2
ubuntu@utuntu:~/lcnx/local/test/test4$ ls /etc/systemd/system/jiukun*
jiukun_self_runner_2.service  
```

##### 验证是否启动 #####

```bash
ubuntu@utuntu:~/lcnx/local/test/test4$ gitlab-ci-multi-runner status
FATAL: Please run the commands as root             
ubuntu@utuntu:~/lcnx/local/test/test4$ sudo gitlab-ci-multi-runner status
gitlab-runner: Service is running!
ubuntu@utuntu:~/lcnx/local/test/test4$ sudo gitlab-ci-multi-runner status -n jiukun_self_runner_2
jiukun_self_runner_2: Service is running!
ubuntu@utuntu:~/lcnx/local/test/test4$ sudo gitlab-runner list --config /home/ubuntu/.gitlab-runner/config.toml 
Listing configured runners                          ConfigFile=/home/ubuntu/.gitlab-runner/config.toml
api-deploy-runner                                   Executor=shell Token=obhhKvSY4s5M1pGp49mf URL=http://192.168.168.137/
ssh-multi-runner                                    Executor=ssh Token=k7U2n9Sb8ErrXa5nzLDr URL=http://192.168.168.137/
ubuntu@utuntu:~/lcnx/local/test/test4$ sudo gitlab-runner verify --config /home/ubuntu/.gitlab-runner/config.toml 
Running in system-mode.                            
                                                   
Verifying runner... is alive                        runner=obhhKvSY
Verifying runner... is alive                        runner=k7U2n9Sb
ubuntu@utuntu:~/lcnx/local/test/test4$ sudo gitlab-ci-multi-runner status -n jiukun_self_runner_2
jiukun_self_runner_2: Service is running!
ubuntu@utuntu:~/lcnx/local/test/test4$ sudo gitlab-ci-multi-runner status -n jiukun_self_runner
jiukun_self_runner: Service is not running.
```

##### 管理服务 #####

1.注册的runner列表

```
gitlab-runner list \
    --config "/etc/gitlab-runner/config_jiukun_test.toml"
```
2.查看runner连接状态

```
gitlab-runner verify \
    --config "/etc/gitlab-runner/config_jiukun_test.toml"
```
3.取消注册（移除）

```
gitlab-runner unregistry \
    --url http://gitlab.xxxxxx.com/ci \
    --token 9c1bb50065661ba766023016f6ebf2
```
不能直接在project的web端进行remove操作，否则这里会执行失败

4.管理gitlab-runner服务

```
gitlab-runner status \
    -n jiukun_self_runner.service
```

不指定服务名，则默认为gitlab-runner服务

```
gitlab-runner stop \
    -n jiukun_self_runner.service

gitlab-runner restart \
    -n jiukun_self_runner.service

gitlab-runner uninstall \
    -n jiukun_self_runner.service
```

执行uninstall会卸载该服务，与之对应的runner将无法通过该服务运行，请确保对应的CI任务已停止。

### 后端使用 pm2 直接在 host 上启用服务 ###

大体与上面相同,就是在运行命令时,记得加入到 PATH 中.

```
before_script:
  - export PATH=/usr/local/bin:/home/ubuntu/.nvm/versions/node/v10.15.3/bin:/home/ubuntu/.nvm/versions/node/v11.15.0/bin:$PATH
```

## web .gitlab-ci.yml ##

```yml
image: node:11.15.0

stages:
  - install_deps
  # - test
  - build
  - deploy_test
  - deploy_production

cache:
  # key: ${CI_BUILD_REF_NAME}
  paths:
    - node_modules/
    - dist/

install_deps:
  stage: install_deps
  tags:
    - my-tag
  only:
    - dev
    - master
  script:
    - 'echo "install_deps"'
    - npm install -g cnpm
    - cnpm install

# 运行测试用例
# test:
#   stage: test
#   tags:
#     - my-tag
#   only:
#     - dev2
#     - master
#   script:
#     - npm run test

# 编译
build:
  stage: build
  tags:
    - my-tag
  only:
    - dev
    - master
  script:
    - 'echo "build"'
    - npm install -g cnpm
    - cnpm run alpha

# 部署测试服务器
deploy_test:
  stage: deploy_test
  tags:
    - my-tag
  only:
    - dev
  before_script:
    - export PATH=/usr/local/bin:$PATH
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SSH_PRIVATE_KEY")
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - echo "$SSH_KNOWN_HOSTS" > ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - '[[ -f /.dockerenv ]] && echo -e "Host *\\n\\tStrictHostKeyChecking no\\n\\n" > ~/.ssh/config'
  script:
    - 'echo "deploy_test"'
    - scp -P 2222 -r dist/ root@192.168.168.137:/home/ubuntu/lcnx/local/lvchuang-web/

# 部署生产服务器
deploy_production:
  stage: deploy_production
  tags:
    - my-tag
  only:
    - master
  before_script:
    - export PATH=/usr/local/bin:$PATH
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SSH_PRIVATE_KEY_LCNX")
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - echo "$SSH_KNOWN_HOSTS_LCNX" > ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - '[[ -f /.dockerenv ]] && echo -e "Host *\\n\\tStrictHostKeyChecking no\\n\\n" > ~/.ssh/config'
  script:
    # - bash scripts/deploy/deploy.sh
    - 'echo "deploy_test"'
    - scp -r dist/ lcnx@120.77.39.189:/home/lcnx/lcnx_eiss/lvchuang-web/dist/
```


