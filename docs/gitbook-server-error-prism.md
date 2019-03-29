

当使用 `gitbook serve` 时会报出下面的错误：

```
Failed to load prism syntax: dockerfile
{ Error: Cannot find module 'prismjs/components/prism-dockerfile.js'
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:581:15)
    at Function.Module._load (internal/modules/cjs/loader.js:507:25)
    at Module.require (internal/modules/cjs/loader.js:637:17)
    at require (internal/modules/cjs/helpers.js:22:18)
    at requireSyntax (/mnt/f/tom/blog/blog/node_modules/gitbook-plugin-prism/index.js:31:3)
    at Object.code (/mnt/f/tom/blog/blog/node_modules/gitbook-plugin-prism/index.js:103:11)
    at Record.TemplateBlock.applyBlock (/root/.gitbook/versions/3.2.3/lib/models/templateBlock.js:205:23)
    at /root/.gitbook/versions/3.2.3/lib/output/getModifiers.js:56:33
    at /root/.gitbook/versions/3.2.3/lib/output/modifiers/highlightCode.js:47:24
    at /root/.gitbook/versions/3.2.3/lib/output/modifiers/editHTMLElement.js:11:16 code: 'MODULE_NOT_FOUND' }
```

是 prismjs 不能理解 "```dockerfile" , 要换成 prismjs 能理解的 "```docker" 


## 参考

- https://github.com/rootsongjc/kubernetes-handbook/issues/221
- https://github.com/rootsongjc/kubernetes-handbook/commit/15eb4dd0323150ced8946b114bf05a46cadd9ccd

