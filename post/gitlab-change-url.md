
gitlab域名修改

https://juejin.im/post/5b7f724c51882542f5046ef1

```bash
root@gitlab:/# cd /opt/gitlab/embedded/service/gitlab-rails/
root@gitlab:/opt/gitlab/embedded/service/gitlab-rails# ls
CHANGELOG.md           GITLAB_PAGES_VERSION      Gemfile       MAINTENANCE.md  README.md  VERSION  builds      config.ru  doc_styleguide.md   fixtures             locale        public   scripts  tmp            yarn.lock
CONTRIBUTING.md        GITLAB_SHELL_VERSION      Gemfile.lock  PROCESS.md      REVISION   app      changelogs  db         docker              generator_templates  log           qa       shared   tsconfig.json
GITALY_SERVER_VERSION  GITLAB_WORKHORSE_VERSION  LICENSE       Procfile        Rakefile   bin      config      doc        docker-compose.yml  lib                  package.json  rubocop  symbol   vendor
root@gitlab:/opt/gitlab/embedded/service/gitlab-rails# vi config
root@gitlab:/opt/gitlab/embedded/service/gitlab-rails# vi /var/opt/gitlab/gitlab-rails/etc/gitlab.yml
root@gitlab:/opt/gitlab/embedded/service/gitlab-rails# which gitlab-ctl
/opt/gitlab/bin/gitlab-ctl
root@gitlab:/opt/gitlab/embedded/service/gitlab-rails# gitlab-ctl restart
ok: run: gitaly: (pid 929) 1s
ok: run: gitlab-monitor: (pid 940) 0s
ok: run: gitlab-workhorse: (pid 950) 1s
ok: run: logrotate: (pid 961) 0s
ok: run: nginx: (pid 970) 1s
ok: run: node-exporter: (pid 979) 0s
ok: run: postgres-exporter: (pid 985) 0s
ok: run: postgresql: (pid 995) 1s
ok: run: prometheus: (pid 1003) 0s
ok: run: redis: (pid 1012) 1s
ok: run: redis-exporter: (pid 1017) 0s
ok: run: sidekiq: (pid 1059) 0s
ok: run: sshd: (pid 1062) 0s
ok: run: unicorn: (pid 1066) 1s
root@gitlab:/opt/gitlab/embedded/service/gitlab-rails#
```


## ref
- https://segmentfault.com/q/1010000000689162
- https://juejin.im/post/5b7f724c51882542f5046ef1
