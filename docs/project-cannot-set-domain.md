
https://docs.gitlab.com/ee/user/project/pages/introduction.html#add-a-custom-domain-to-your-pages-website

```
Note: Currently there is support only for custom domains on per-project basis. That means that if you add a custom domain (example.com) for your user website (username.example.io), a project that is served under username.example.io/foo, will not be accessible under example.com/foo
```

也就是说，如果你使用了 qqmm.app 来配置 eiuapp.gitlab.io 项目，则 https://qqmm.app/ 可以访问到，但是，https://qqmm.app/npcs-note/ 并不能访问到 npcs-note 项目。


