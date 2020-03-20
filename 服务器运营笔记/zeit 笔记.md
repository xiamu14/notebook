# Zeit 使用笔记

## 将 koa 项目部署到 zeit

`index.js` 文件内容与正常的 Koa.js 项目代码无异，唯一的区别是最终项目没有直接调 `app.listen()` 方法进行监听，而是使用 `module.exports = app.callback()` 将最终的 callback 方法进行了返回。我们知道 [app.callback()](https://github.com/koajs/koa/blob/ed84ee50da8ae3cd08056f944d061e00d06ed87f/lib/application.js#L141-L152) 方法返回的是接受 `request` 和 `response` 对象作为参数的函数，这就回到了文章最开始的示例了。

我们再来看看 `now.json` 的内容。该 JSON 文件用于告诉 now 服务 `index.js` 文件需要使用 `@now/node` 运行时执行，而所有的请求需要转发到 `index.js` 文件上。听起来是不是非常像 Nginx 上的内容？

```json
{
  "version": 2,
  "builds": [
    { "src": "index.js", "use": "@now/node" }
  ],
  "routes": [
    { "src": "/(.*)", "dest": "/index.js" }
  ]
}
```

最后，git commit 保存好修改文件，再接着使用 ` now` 命令 发布到 zeit 服务上即可。

参考这个[demo](https://github.com/lizheming/now-koa-demo)。