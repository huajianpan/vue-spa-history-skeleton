目前page-skeleton-webpack-plugin 似乎没怎么维护了，初看一堆的issues 难以下手呀。


不过看了自动生成骨架屏的原理，这个文章
[基于page-skeleton-webpack-plugin分析自动生成骨架屏原理](https://www.colabug.com/goto/aHR0cHM6Ly9qdWVqaW4uaW0vcG9zdC81Yzk4OTAxNjZmYjlhMDcwYjg1MDYzNDE=)

，还是很有必要进行一次DEMO 演习，以证明生成骨架屏的可行性、必要性。

下面记录踩坑步骤：

### 1、拉取 `page-skeleton-webpack-plugin` 代码,或者下载zip包

```
git clone https://github.com/ElemeFE/page-skeleton-webpack-plugin.git
复制代码
```

2、拉好代码之后，切换到 `examples/sale` 案例目录下，进行依赖安装，这里你可能会遇到Puppeteer 安装失败的问题，这里请你上网|cnpm |yarn等，多试几次，直到成功。

### 3、跑 `npm run dev` 这里你可能遇到：

- 报TypeError: Cannot read property ‘properties’ of undefined

  - 升级 `webpack-cli` 到3.1.1 `npm install --save-dev webpack-cli@3.1.1`
  - 去除Demo 工程 `webpack.config.js // 119line` 下的storagies 对象，避免报错

- 报缺style-loader模块

  - 安装 `style-loader` 模块

- 报

  ```
  address already in use :::8989 at Server
  ```

  ,修改成任何端口，依然提示端口被占用的bug。

  - 发现compiler.hooks.entryOption被执了两次，于是：在createServer方法中判断插件的server是否被创建，没创建的时， 正常执行，创建过就什么都不做了。修改文件在 `node_modules/page-skeleton-webpack-plugin-master/src/skeletonPlugin.js` 在 `SkeletonPlugin.prototype.createServer` 里加一个if判断

```
// const server = this.server = new Server(this.options) // eslint-disable-line no-multi-assign
// server.listen().catch(err => server.log.warn(err))
if (!this.server) {
const server = this.server = new Server(this.options) // eslint-disable-line no-multi-assign
server.listen().catch(err => server.log.warn(err))
}
复制代码
```

然后跑dev 应该问题不大了，如何还有问题，依赖删了再重新试一下。很多问题都可以在issues 中找到，只是官方没有对demo进行更新处理。

### 4、接下来，应该如何去更新shell 目录下的骨架样式文件呢？

dev 之后，默认会打开demo 页，然后命令端这里会起一个 `page-skeleton` 的server,我们在浏览器打开这个端口（ip+端口号）的界面：

在这里我们可以预览到骨架屏和样式代码，我们也可以修改对应的样式代码，最终保存到shell文件下的文件中。 整个生成骨架页面的核心也就是在这一步

最终，可以看到，输出的内容都放置到Dom app 节点下。 希望可以帮到。

注意：本文来自[稀土掘金](https://www.colabug.com/author/19902/)。本站无法对本文内容的真实性、完整性、及时性、原创性提供任何保证，请您自行验证核实并承担相关的风险与后果！
CoLaBug.com遵循[[CC BY-SA 4.0](https://www.colabug.com/goto/cclicensing)]分享并保持客观立场，本站不承担此类作品侵权行为的直接责任及连带责任。您有版权、意见、投诉等问题，请通过[[eMail\]](https://www.colabug.com/goto/tousu)联系我们处理，如需商业授权请联系原作者/原网站。