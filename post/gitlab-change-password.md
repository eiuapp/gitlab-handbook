gitlab修改root用户密码

https://www.cnblogs.com/rhca/p/10646641.html

# step
1.使用root权限登录到服务器。
2.使用以下命令启动控制台:
   gitlab-rails console production
3.有多种方法可以查找您的用户。您可以搜索电子邮件或用户名。
   user = User.where(id: 1).first
或者
   user = User.find_by(email: 'admin@local.host')
4.现在, 您可以更改密码:
   user.password = 'secret_pass'
   user.password_confirmation = 'secret_pass'
5.不要忘记保存：
   user.save!
6.退出控制台, 然后尝试使用新密码登录。

# example

```bash
ubuntu@utuntu:~$ docker exec -it gitlab-ce-11.9.1-2 bash
root@192:/# sudo gitlab-rails console production
bash: sudo: command not found
root@192:/#  gitlab-rails console production

-------------------------------------------------------------------------------------
 GitLab:       11.9.1 (86f0b5d)
 GitLab Shell: 8.7.1
 postgresql:   9.6.11
-------------------------------------------------------------------------------------
Loading production environment (Rails 5.0.7.1)
irb(main):001:0>
irb(main):002:0> user = User.where(id: 1).first
=> #<User id:1 @root>
irb(main):003:0> user = User.where(id: 2).first
=> #<User id:2 @zengyunlong>
irb(main):004:0> user.password = 'secret_pass'
=> "secret_pass"
irb(main):005:0> user.password_confirmation = 'secret_pass'
=> "secret_pass"
irb(main):006:0>  user.save!
Enqueued ActionMailer::DeliveryJob (Job ID: 8138f360-ccc2-4904-b013-ac050e5723d1) to Sidekiq(mailers) with arguments: "DeviseMailer", "password_change", "deliver_now", #<GlobalID:0x00007f462c2db440 @uri=#<URI::GID gid://gitlab/User/2>>
=> true
irb(main):007:0> exit
root@192:/#
```
