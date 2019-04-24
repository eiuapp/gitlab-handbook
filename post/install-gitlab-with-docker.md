
https://docs.gitlab.com/ee/install/docker.html

拉取镜像

```bash
docker pull gitlab/gitlab-ce:11.10.0-ce.0
```

启动

```bash
sudo docker run --detach \
  --hostname gitlab.tianluoteam.com \
  --publish 5443:443 --publish 5080:80 --publish 5022:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  store/gitlab/gitlab-ce:10.2.4-ce.0
  


sudo docker run --detach \
  --hostname 192.168.168.164 \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab-tl \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  store/gitlab/gitlab-ce:10.2.4-ce.0
  
  
sudo docker run --detach \
  --hostname 192.168.168.164 \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab-ce-11.10.0-ce.0 \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:11.10.0-ce.0
  
 
  
sudo docker run --detach \
  --hostname 192.168.168.164 \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab-ce-11.10.0-ce.0 \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:11.9.1-ce.0
  

sudo docker run --detach \
  --hostname 192.168.168.164 \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab-ce-11.9.1-2 \
  --restart always \
  --volume /srv/gitlab9.1/config:/etc/gitlab \
  --volume /srv/gitlab9.1/logs:/var/log/gitlab \
  --volume /srv/gitlab9.1/data:/var/opt/gitlab \
  gitlab/gitlab-ce:11.9.1-ce.0
```

chrome:

http://192.168.168.164

输入密码后，会让修改密码的。

如果第一次不知道输入什么密码，那就走修改密码方式。

