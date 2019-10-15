
```
root@utuntu:/home/ubuntu# docker logs gitlab-runner
WARNING: Checking for jobs... failed                runner=2K7oPJtf status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: dial tcp 192.168.168.137:80: getsockopt: connection refused
WARNING: Checking for jobs... failed                runner=VE3S_jfp status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: dial tcp 192.168.168.137:80: getsockopt: connection refused
WARNING: Checking for jobs... failed                runner=2K7oPJtf status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: dial tcp 192.168.168.137:80: getsockopt: connection refused
WARNING: Checking for jobs... failed                runner=VE3S_jfp status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: dial tcp 192.168.168.137:80: getsockopt: connection refused
WARNING: Checking for jobs... failed                runner=2K7oPJtf status=couldn't execute POST against http://192.168.168.137/api/v4/jobs/request: Post http://192.168.168.137/api/v4/jobs/request: dial tcp 192.168.168.137:80: getsockopt: connection refused
```

说明，gitlab-ce 本身未启动。

启动就可以了

```
ubuntu@utuntu:~$ sudo docker run --detach   --restart always   --hostname 192.168.168.137   --publish 12443:443 --publish 80:80 --publish 22:22   --name gitlab-ce-11.9.1-2   --volume /srv/gitlab9.1/config:/etc/gitlab   --volume /srv/gitlab9.1/logs:/var/log/gitlab   --volume /srv/gitlab9.1/data:/var/opt/gitlab   gitlab/gitlab-ce:11.9.1-ce.0
[sudo] password for ubuntu: 
52a10ef83006bda072a5c9df5b3d548f05dd548ae887bc64d3277e7b3280b199
ubuntu@utuntu:~$ docker ps
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS                           PORTS                                                            NAMES
52a10ef83006        gitlab/gitlab-ce:11.9.1-ce.0           "/assets/wrapper"        2 seconds ago       Up 1 second (health: starting)   0.0.0.0:22->22/tcp, 0.0.0.0:80->80/tcp, 0.0.0.0:12443->443/tcp   gitlab-ce-11.9.1-2
44014ef848a5        codercom/code-server                   "dumb-init code-serv…"   2 months ago        Up 48 minutes                    0.0.0.0:8443->8443/tcp                                           happy_ramanujan
8303dc0b8742        swaggerapi/swagger-ui:20190808         "sh /usr/share/nginx…"   2 months ago        Up 48 minutes                    80/tcp, 0.0.0.0:3080->8080/tcp                                   swagger-ui
8bc56d42ee67        swaggerapi/swagger-editor:20190808     "sh /usr/share/nginx…"   2 months ago        Up 48 minutes                    0.0.0.0:3081->8080/tcp                                           swagger-editor
086bdb8baa32        mysql03-save:20190525                  "docker-entrypoint.s…"   3 months ago        Up 48 minutes                    0.0.0.0:3306->3306/tcp                                           mysql03-20190525
bb97a9e27f4a        yapi                                   "/api/docker-entrypo…"   4 months ago        Up 48 minutes                    0.0.0.0:3001->3001/tcp                                           yapi
cfbaa6436964        mongo:4                                "docker-entrypoint.s…"   4 months ago        Up 48 minutes                    0.0.0.0:27017->27017/tcp                                         mongod
65fa431b0ec1        gitlab/gitlab-runner:ubuntu-v11.11.1   "/usr/bin/dumb-init …"   4 months ago        Up 48 minutes                                                                                     gitlab-runner-shell
374e972d1fac        some-redis-save                        "docker-entrypoint.s…"   4 months ago        Up 48 minutes                    0.0.0.0:6379->6379/tcp                                           some-redis
be6755a2746c        gitlab/gitlab-runner:ubuntu-v11.11.1   "/usr/bin/dumb-init …"   4 months ago        Up 48 minutes                                                                                     gitlab-runner
ubuntu@utuntu:~$ 
```
