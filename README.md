# [新手坑] 02.Vue开发环境和生产环境样式不一致的问题

> 上次提到作用域问题, 导致样是不生效, 这次讲的是我之前遇到的一个小坑, 稍不留神就完蛋.

前阵子做的一个小项目, 引入了Vant的UI库, 外加自己写的很多样式, 在开发环境下测试完美, 直接就build出来上正式环境, 发现竟然有多处样式未生效的问题! 还好是新项目, 尚未推广, 因此除了内部同事测试发现, 没有造成恶劣影响, 不过以后还是要注意下, 开发环境看着没问题, 但是生产环境一定还是要再过一遍.

那么为什么会出现这个问题呢? 我下面来做些小的测试观察一下.

## 问题现象

在开发环境下, 每个不同块的`style`都会被单独提取插入到页面的`head`区域, 而生产出来的的文件是会被合并成一个文件, 在开发环境下, 这些`style`块的顺序又和生产环境编译出来的css文件内的顺序有差别, 导致我们在开发环境中, 使用了相同的优先级, 覆盖原Vant的UI样式看起来正常, 而在生产后, 顺序错误导致失效了!

为了更加方便测试, 我在[vue-cli-mobile-study](https://github.com/whidy/vue-cli-mobile-study)项目创建了一个分支[`02-build-css-order`](https://github.com/whidy/vue-cli-mobile-study/tree/02-build-css-order), 有兴趣可以看看~

> 本来想在不同块的css中添加注释以便于更明显的观察顺序变化, 结果发现生产环境中的注释被自动忽略了, 尝试去掉`cssnano`插件执行, 发现还是有部分注释没有展示出来, 因为不是很重要, 所以没有去纠结这块.

开发环境中的`head`区域有效的`style`有5个. 分别是

1. `index.html`中的css样式
1. vant的`base.css`内容
1. 路径为`./vue-cli-mobile-study/src/assets/styles/uireset.css`内容
1. `App.vue`的css内容
1. `HelloWorld.vue`的css内容

而生产出来的却和这个不同, 因为被合并为1个css文件了, 因此我们观察单个css文件的上面4块的顺序

1. `index.html`中的css样式
1. `App.vue`的css内容
1. `HelloWorld.vue`的css内容
1. vant的`base.css`内容
1. 路径为`./vue-cli-mobile-study/src/assets/styles/uireset.css`内容

当然, 其实在有作用域的组件中所包含的样式顺序对项目是没有影响的, 所以我们需要关注的是**全局引入的样式**顺序, 从上面的现象中可以看出, 除了核心文件`index.html`, **开发环境**下, Vant样式默认在最前面(Vant实际上是Babel那边导入了), 而其他样式则似乎根据`main.js`入口的顺序, 以及渲染顺序来添加到html头部的; 而`生产环境`下, 相对诡异.

## 问题原因

我有在GitHub查阅过相关Issues, 也找过StackOverflow相关内容, 不过没有什么收获, 外加Webpack的高级配置方面也不是很熟悉, 因此也就没有研究出来, 如果有大神能指点一二欢迎留言.

## 解决方案

在需要覆盖第三方组件的默认样式是, 尽量使用高于第三方组件优先级的css样式, 以免出现开发环境和生产环境效果不同的情况. 在自己的组件样式编写中, 除了有公共的组件在不同页面的样式控制下可能需要全局样式外, 尽量写上作用域!

下次来记录一点什么坑呢, 我还没想好 - -.